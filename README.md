# Open Obsidian

A simple redirector for Obsidian URI schemes.

## Why use this?

Many applications (like Telegram, Slack, or some Todo apps) do not correctly recognize or allow clicking on custom URI schemes like `obsidian://`. 

This tool provides a standard `https://` link that redirects to your Obsidian vault, allowing you to use Obsidian links in any app that supports web links.

## Usage

Append `?url=` followed by an encoded Obsidian URI to the base URL.

### Example

**Base URL:**
`https://open-obsidian.stefanhoth.com/`

**Obsidian URI:**
`obsidian://open?vault=my-vault&file=Daily%20Notes/2026-03-10`

**Redirect Link:**
`https://open-obsidian.stefanhoth.com/?url=obsidian://open?vault=my-vault%26file=Daily%2520Notes/2026-03-10`

*(Note: The `&` in the Obsidian URI should be encoded as `%26` to ensure it's captured as part of the `url` parameter.)*

## Generator

If you visit the base URL without any parameters, you can use the built-in generator to create these links easily.

## Deployment

Hosted on Cloudflare Workers (static assets from `public/`) at [open-obsidian.stefanhoth.com](https://open-obsidian.stefanhoth.com).

- **Push to `main`** → `wrangler deploy` to production.
- **Pull request** → preview version uploaded under a `pr-<N>` alias on workers.dev (URL in the job summary).

Required repository secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`.

The root `index.html` is kept for the old GitHub Pages URL (`stefanhoth.github.io/open-obsidian/`) and forwards existing links, including their `?url=` parameter, to the new domain.
