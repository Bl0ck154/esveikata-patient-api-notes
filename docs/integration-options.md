# Using these eSveikata research notes

This repository documents **unofficial observations from September 2026** of the Lithuanian eSveikata patient portal, ELAB laboratory application, and IPR scheduling endpoints. It contains no live patient session, SDK, MCP server, or ready-to-run authenticated client.

## What is documented

- Patient-facing request and response flows, where observed in an authenticated browser session.
- The browser-to-ELAB session handoff.
- Observations about public and authenticated IPR APIs.
- Cautions about changing endpoints, authorization rules, and session lifetimes.

## Implementation choices

A developer building a personal integration should use the normal eSveikata authentication flow, limit access to the account holder's own records, and keep session credentials in private, access-restricted local storage.

A browser may be needed for interactive login. Subsequent API reads can sometimes be made using a session-scoped HTTP client that handles permitted cookie updates. Whether this works depends on the portal's current session and authorization behavior. An HTTP 401/403 requires renewed authorization; a keep-alive request cannot override an absolute session expiration or server-side invalidation.

For AI tools, a separate local MCP adapter could expose narrowly scoped **read-only** functions such as a list of laboratory results or prescriptions. This repository does **not** implement such an adapter. Never expose cookies or raw health records through a public/shared MCP endpoint.

IPR appointment discovery and patient medical-record access have different privacy and authorization requirements. Any real booking or cancellation must require explicit approval for the specific slot and action.

## Limits

These are research notes, not official eSveikata documentation. Do not hardcode or publish real cookies, patient identifiers, medical documents, Smart-ID credentials, or bearer tokens in code, tests, logs, issues, or examples. Use synthetic data for public examples and verify observed endpoints against the current portal.
