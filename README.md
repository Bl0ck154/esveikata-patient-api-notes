# eSveikata Patient Portal API Notes

> **Research snapshot — September 2026.** Unofficial API and interoperability notes, not an actively maintained SDK or an official integration.

Unofficial research notes for Lithuania's **eSveikata patient portal** (`pacientas.esveikata.lt`), its embedded **ELAB laboratory results** application, and related patient-facing API flows.

This repository is for people exploring legitimate interoperability, personal health-data portability, and read-only tooling for their own records. It is **not affiliated with eSveikata or Lithuanian healthcare authorities**.

These notes document browser-observed request flows and frontend bundle behavior as of September 2026. They are intended for interoperability, personal data portability and read-only tooling for a user accessing their own health record.

## What was observed

The patient portal is not purely server-rendered HTML. After an authenticated login it uses structured JSON APIs under:

```text
https://pacientas.esveikata.lt/api/patient/...
```

The newer laboratory subsystem (ELAB) is embedded from:

```text
https://specialistas.esveikata.lt/dp/elab/iframe/
```

The parent patient portal obtains a short-lived session bridge token and passes it to the ELAB iframe, which exchanges it for its own cookie-backed session.

## Scope and maintenance

- **Last observed:** September 2026. The documented requests may no longer match the current portal.
- **Format:** API research notes, endpoint observations and authentication-flow documentation — not a ready-to-run client, SDK or service.
- **Maintenance:** No ongoing compatibility guarantee. Check against the current official portal before building integrations.
- **Access:** Use only your own authorized session and data. No bypass of login, authorization or security controls is provided.

## Integrating these notes

For architecture choices, session-handling limits and a possible local MCP adapter, see [Integration options](docs/integration-options.md). This repository is research documentation, not an MCP server or a live patient-data connector.

## Documents

- [Patient API](docs/patient-api.md)
- [Authentication/session flow](docs/authentication.md)
- [ELAB](docs/elab.md)
- [Android mobile API](docs/mobile-api.md)
- [IPR advance-registration API](docs/ipr.md)
- [Security and scope](SECURITY.md)

## Important caveats

This is **not an official API specification**. Endpoints, request shapes and authorization rules may change without notice. Do not embed captured cookies, tokens, personal codes, document identifiers or real medical data in source code, examples, issues or logs.

The repository deliberately does not contain tooling for bypassing authentication. A valid eSveikata session obtained by the user through the normal login flow is assumed.
