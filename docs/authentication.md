# Authentication and session flow

Observed high-level flow:

```text
Normal eSveikata login (bank / Smart-ID / supported identity flow)
        ↓
pacientas.esveikata.lt authenticated browser session
        ↓
HttpOnly cookie-backed patient session
        ↓
GET /api/patient/session/current
```

The patient portal frontend also requests:

```text
GET /api/patient/session/token
```

That response is then used to initialize the newer ELAB iframe. The token should be treated as a credential and should not be logged or committed.

## Session persistence

Browser-observed cookies included session cookies for the patient portal and the SSO domain. Because they are session cookies, browser/process lifetime and server-side timeout policy both matter.

A periodic authenticated request may keep a **sliding idle timeout** alive if the application server refreshes activity on requests. It cannot guarantee survival across:

- an absolute maximum session lifetime;
- explicit server-side invalidation;
- account/security policy changes;
- a browser profile reset;
- a reboot where session cookies are not restored.

For a local personal integration, a practical model is therefore:

1. User authenticates normally in a dedicated browser profile.
2. A local read-only service sends a lightweight authenticated heartbeat at a conservative interval.
3. The service reports session health but never attempts to recreate user authentication credentials.
4. When the server invalidates the session, the user performs an interactive login again.

## Security recommendation

Prefer executing authenticated reads inside the existing browser context or keeping credentials in a local cookie jar that is never exposed externally. Do not publish captured cookies or `/session/token` values.
