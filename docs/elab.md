# ELAB notes

The patient portal route `/pp/elab` embeds a separate application from:

```text
https://specialistas.esveikata.lt/dp/elab/iframe/
```

Observed parent frontend behavior constructs an iframe URL similar to:

```text
.../dp/elab/iframe/#/compositions?token=<session-bridge-token>
```

The ELAB frontend reads the token from its URL and exchanges it through:

```text
GET /dp/elab/iframe/api/auth/authenticate?token=<token>
```

The exchange establishes a cookie-backed ELAB session; subsequent API calls use `credentials: include` and do not attach the bridge token to each request.

## Observed read/list endpoints

Frontend bundle inspection exposed the following list functions:

```text
POST /dp/elab/iframe/api/e200/order/list
POST /dp/elab/iframe/api/e200ats/composition/list
POST /dp/elab/iframe/api/e200ats/diagnosticReport/list
POST /dp/elab/iframe/api/e200ats/diagnosticReport/sensitive-list
POST /dp/elab/iframe/api/e200ats/diagnosticReport/list-entered-in-error
```

The generic list client sends a body shaped approximately as:

```json
{
  "page": 1,
  "limit": 10,
  "filters": {}
}
```

Patient navigation uses the `/compositions` screen. Authorization differs by account type; endpoints visible in the bundle are not necessarily available to a patient account.

## Other observed ELAB routes

```text
GET /dp/elab/iframe/api/e200/order/{requisition}
GET /dp/elab/iframe/api/e200ats/documentReference/{id}
```

The document-reference endpoint is used by the frontend to download a signed PDF.

## Important distinction

The legacy/general patient `documents2` API and ELAB are separate surfaces. A robust client should not assume that every laboratory result lives in only one of them. For interoperability, query the patient-side document list and ELAB composition list independently when both are available.

Never publish a real bridge token, ELAB session cookie, patient identifier or requisition/document identifier captured from a live account.
