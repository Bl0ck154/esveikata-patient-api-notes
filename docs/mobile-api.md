# eSveikata Android mobile API notes

Unofficial reverse-engineering notes for the official eSveikata Android application version 1.4.8. These notes are intended for interoperability, personal data portability and read-only tooling for a user accessing their own account.

## Separate mobile backend

The Android application is not just a wrapper around the patient web portal. Static analysis identified a separate production backend:

```text
https://mobileapp.esveikata.lt
```

Android package:

```text
lt.registrucentras.esveikata_app
```

Observed custom callback URI:

```text
lt.registrucentras.esveikata://callback
```

## Authentication model

The application contains `flutter_appauth` / Android AppAuth and strong evidence of OAuth/OIDC Authorization Code + PKCE:

```text
authorizationCode
codeVerifier
clientId
redirectUrl
discoveryUrl
issuer
openid
```

Token-related names found in the compiled application include:

```text
kc_access_token
kc_refresh_token
kc_id_token
accessToken
refreshToken
idToken
accessTokenExpirationTime
```

Relevant mobile login endpoints found statically:

```text
/api/login/gate
/api/login/ipas
/api/login/oauth
/api/login/refresh
/api/getUser
```

Other observed auth/request metadata includes `Authorization: Bearer`, `x-auth-provider`, `x-device-info`, `x-app`, `x-app-timeout`, `x-fcm-token` and force-logout headers.

The APK includes `flutter_secure_storage` and Android Keystore/encrypted-storage implementations, which is consistent with long-lived refresh/session material being kept in protected local storage rather than ordinary plaintext preferences.

## Live anonymous backend checks

From a normal Linux VPS, the mobile backend was directly reachable. The health endpoint returned HTTP 200, so there is no obvious network-level requirement that requests originate from Android.

The mobile backend also sets its own HTTP cookies (including a session cookie) on anonymous requests. A compatible client should therefore be prepared to maintain both bearer-token state and a normal cookie jar.

`GET /api/login/ipas` redirected to the iPasas flow with the production application identifier `espbi_mobile_prod`, confirming that the mobile application has its own production authentication registration.

## Read-oriented API surface found in the APK

Examples relevant to a read-only integration include:

```text
/api/getUser
/api/patient/checkPatient
/api/patient/getPatientProfile
/api/patient/getPatientContacts
/api/patient/getPatientMail
/api/patient/getOrganization
/api/patient/getPractitioner
/api/healthHistory/getPatientEncounters
/api/healthHistory/getEncounterTypes
/api/calendar/getPatientCalendarEvents
/api/calendar/getEncounterDocuments
/api/documents/getPatientDocumentData
/api/documents/getPatientDocumentHistory
/api/documents/getPatientDocumentPDF
/api/appPrescriptions/getPrescriptionsTypes
/api/appPrescriptions/getPrescriptionsByType
/api/refferals/getPatientRefTypes
/api/refferals/getPatientRefByTypes
/api/appCertificates/getPatientCertificates
/api/notification/getPatientMessages
/api/representees/getPatientRepresentatives
/api/representees/getPatientRepresentees
```

The APK also exposes many write-oriented routes for appointments, profile changes, representatives and notifications. A personal integration should explicitly allowlist only the read operations it needs.

## Relationship to the web patient API

A persistent Chrome process is not required for the web patient API either: an authorized user can bootstrap the normal web session once, store the resulting session cookies in a private cookie jar and continue authenticated HTTPS requests browserlessly until the server invalidates the session.

This changes the role of the Android findings. The mobile API is no longer required merely to eliminate a 24/7 browser process. It remains interesting because:

- the refresh-token model may survive longer than the web session and reduce how often interactive login is needed;
- it provides an independent API surface and useful fallback if the patient web API changes;
- it exposes mobile-specific functionality such as calendar/visit, referral and notification routes;
- it may provide cleaner long-lived authentication if off-device token refresh is accepted by the production backend.

## Remaining unknowns

Static analysis proves the overall token-based architecture but does not prove that a refresh token is portable off-device. The main items still requiring one operator-authorized runtime trace are:

1. exact request/response schema for `/api/login/oauth` and `/api/login/refresh`;
2. whether the refresh token rotates;
3. exact values/format of `x-auth-provider`, `x-app` and `x-device-info`;
4. whether `x-device-info` is merely telemetry or required server-side binding;
5. whether bearer tokens must be accompanied by the mobile session cookie;
6. production OIDC issuer/client configuration if delivered dynamically;
7. which mobile endpoints expose laboratory data equivalent to the patient web API/ELAB;
8. whether a refresh token obtained by the official app works unchanged from a normal VPS.

There is no strong static evidence that normal API calls require Play Integrity or cryptographic Android-device request signing, but runtime verification is still required before relying on the mobile refresh flow in production.

## Security scope

Never publish real access/refresh/ID token values, patient identifiers, cookies, login traces containing credentials, or medical payloads. The useful interoperability information is the protocol shape, endpoint names and sanitized behavior—not captured account secrets.
