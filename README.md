# Posthog Reverse Proxy

This project provides a template and guide for setting up a Netlify reverse proxy for PostHog.

## Why Use a Reverse Proxy for PostHog?

- **Bypass Ad-Blockers:** Many ad blockers detect analytics tools like Google Analytics, PostHog, etc., by the network requests your site sends to specific third-party domains. These requests can be blocked, leading to incomplete data. By routing your analytics traffic through a subdomain on your own website (e.g., `hog.yourdomain.com`), it appears as a first-party request, making it much less likely to be blocked. As the user put it: "If we route our analytics stuff via a sneaky subdomain on our own website like hog.example.com, no one will suspect a thing 🕵️."

- **Custom Domain & Branding:** Using your own domain for analytics enhances your site's professional appearance and can contribute to more reliable data collection by avoiding third-party cookie restrictions that might affect direct integrations.

- **Cost Savings:** PostHog offers a managed reverse proxy, but it may be part of a higher-tier plan (e.g., the user mentioned it requiring a "$450.00/month Teams Add on"). Setting up your own reverse proxy on a platform like Netlify can be a cost-effective alternative. While it's important to monitor Netlify's bandwidth and request limits on their free/lower tiers, for many use cases, this self-managed approach can be significantly cheaper.

## Setup and Configuration Steps

Follow these steps to configure the reverse proxy:

### 1. Prepare Your Project
- Ensure your project has an `index.html` file (can be a simple placeholder).
- Create a `netlify.toml` file in the root of your project for the redirect configurations.

### 2. Create a Site in Netlify and Deploy
- Create a new site in Netlify.
- Connect your GitHub (or other Git provider) repository and deploy your site. Netlify will build and deploy your site when you push changes.

### 3. Configure Custom Domain (Recommended)
- The PostHog documentation notes that the proxy configuration works best with custom domains and might not work correctly with the default `.netlify.app` domain.
- In your DNS provider, add a CNAME record for your chosen subdomain (e.g., `hog.yourdomain.com`) to point to your Netlify site URL (e.g., `your-netlify-site-name.netlify.app`).
- Add the custom domain in your Netlify site settings under "Domain management".

### 4. Configure Netlify Redirects
Add the following redirect rules to your `netlify.toml` file. These rules tell Netlify to proxy requests from your domain's `/ingest` path to PostHog's servers.

**Important:**
- Adjust the `to` URLs if you are using PostHog's EU cloud (e.g., `eu.i.posthog.com` and `eu-assets.i.posthog.com`). The example below uses the US endpoints.
- The `host` property in the redirect rules, as shown in some PostHog documentation examples, might not be necessary or could cause issues in some Netlify setups. Test thoroughly with and without it if you encounter problems. The configuration provided in this `README.md` (and in the project's `netlify.toml`) omits the `host` property for broader compatibility.

```toml
# /netlify.toml

# Proxy for PostHog static assets
[[redirects]]
  from = "/ingest/static/*"
  to = "https://us-assets.i.posthog.com/static/:splat" # Use eu-assets.i.posthog.com for EU cloud
  status = 200
  force = true

# Proxy for PostHog API requests
[[redirects]]
  from = "/ingest/*"
  to = "https://us.i.posthog.com/:splat" # Use eu.i.posthog.com for EU cloud
  status = 200
  force = true

# Optional but Recommended: Force HTTPS
[[redirects]]
  from = "http://*"
  to = "https://:splat" # :splat represents the rest of the matched path
  status = 301
  force = true
```

**Alternative for SvelteKit:** If deploying SvelteKit on Netlify, `netlify.toml` redirects might not be supported. Use a `_redirects` file in your project root instead:
```
# /_redirects
/ingest/static/*  https://us-assets.i.posthog.com/static/:splat  200!
/ingest/*         https://us.i.posthog.com/:splat         200!
```
(Adjust for EU cloud if necessary.)

### 5. Verify HTTPS
- Ensure HTTPS is active for your custom domain in Netlify. Netlify usually provisions SSL certificates automatically for custom domains.
- The redirect rule `http://*` to `https://:splat` in `netlify.toml` helps enforce HTTPS. Verify this is working after deployment.

### 6. Update PostHog Snippet in Your Website
Modify the PostHog JavaScript snippet in your website's code to use your proxy URL as the `api_host`.

Replace `'YOUR_PROJECT_API_KEY'` with your actual PostHog Project API Key.
Replace `https://your-proxy-domain.com` with your actual domain where Netlify is serving the proxy (e.g., `https://hog.yourdomain.com` or `https://your-netlify-site-name.netlify.app` if not using a custom domain).

```javascript
posthog.init('YOUR_PROJECT_API_KEY', {
  api_host: 'https://your-proxy-domain.com/ingest', // Your Netlify site URL + /ingest
  ui_host: 'https://us.posthog.com', // Or 'https://eu.posthog.com' for EU
  // person_profiles: 'identified_only' // Optional: or 'always'
});
```

### 7. Test and Verify

**a. Test the `/decide` Endpoint:**
Visit a URL like the following in your browser, replacing placeholders with your actual proxy domain and Project API Key:
`https://your-proxy-domain.com/ingest/decide/?v=2&ip=1&_=YOUR_TIMESTAMP&ver=LIBRARY_VERSION&key=YOUR_PROJECT_API_KEY&capture_mode=prefix&compression=gzip-js`
(You can get a full example URL from your browser's network tab when PostHog makes a `/decide` call on a page where it's initialized).

A simpler version for a quick check (might vary based on PostHog version):
`https://your-proxy-domain.com/ingest/decide/?v=3&key=YOUR_PROJECT_API_KEY`

You should see a JSON response, for example:
```json
{
  "config": {
    "enable_collect_everything": true
  },
  "toolbarParams": {},
  "isAuthenticated": false,
  "supportedCompression": [
    "gzip",
    "gzip-js"
  ],
  "featureFlags": [],
  "sessionRecording": false 
}
```
*(The exact content of the JSON response may vary.)*

**b. Send a Test Event:**
After initializing PostHog on a test page, capture a custom event:
```javascript
posthog.capture('reverse-proxy-test', { value: 'up-and-running' });
```

**c. Verify in PostHog:**
- Check your browser's developer tools (Network tab) to ensure requests are going to `https://your-proxy-domain.com/ingest/...` and are returning `200 OK`.
- Go to your PostHog project, navigate to the "Activity" or "Events" tab, and look for the `reverse-proxy-test` event. (e.g., `https://us.posthog.com/project/YOUR_PROJECT_ID/activity/explore`)

---

For more background, refer to the [official PostHog documentation on Netlify reverse proxies](https://posthog.com/docs/advanced/proxy/netlify).

