# Security and scope

This repository contains interoperability notes only. It must not contain live credentials or private health data.

## Never commit

- eSveikata or SSO cookies;
- `/session/token` values;
- Smart-ID/bank credentials or PINs;
- personal codes, ESI numbers, addresses or phone numbers;
- real document, requisition or encounter identifiers from a private account;
- raw API captures containing patient data;
- browser profiles, cookie databases or HAR files from a logged-in session.

## Intended use

The documented flows assume that a user has authenticated through the normal eSveikata login process and is accessing data they are authorized to view. Prefer read-only clients, minimal request frequency and local secret storage.

## Reporting accidental disclosure

If a secret or personal record is accidentally committed, remove it from Git history and rotate/invalidate the relevant session rather than merely deleting it in a later commit.
