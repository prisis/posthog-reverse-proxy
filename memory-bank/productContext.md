# Product Context

## Why This Project Exists

This project exists to set up a reverse proxy for PostHog. This is often done to overcome ad-blockers or to use a custom domain for PostHog analytics, improving data collection reliability and branding.

## Problems It Solves

- Prevents ad-blockers from blocking PostHog tracking scripts by serving them from a custom domain.
- Allows the use of a custom, branded domain for PostHog analytics.

## How It Should Work

- Requests to a specific path on the Netlify site (e.g., `/ingest`) should be forwarded to the PostHog instance.
- The `index.html` page will serve as a placeholder or simple landing page for the root domain, advertising anolilab.com.

## User Experience Goals

- For end-users of anolilab.com, the experience is a simple webpage.
- For the project owner, the PostHog integration should be seamless and reliable. 