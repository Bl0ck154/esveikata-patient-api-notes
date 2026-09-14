# eSveikata Patient Portal API Notes

Unofficial reverse-engineering notes for the Lithuanian eSveikata patient portal (`pacientas.esveikata.lt`) and the embedded ELAB application.

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

## Documents

- [Patient API](docs/patient-api.md)
- [Authentication/session flow](docs/authentication.md)
- [ELAB](docs/elab.md)
- [Security and scope](SECURITY.md)

## Important caveats

This is **not an official API specification**. Endpoints, request shapes and authorization rules may change without notice. Do not embed captured cookies, tokens, personal codes, document identifiers or real medical data in source code, examples, issues or logs.

The repository deliberately does not contain tooling for bypassing authentication. A valid eSveikata session obtained by the user through the normal login flow is assumed.
