# IPR advance-registration API notes

Unofficial interoperability notes for the Lithuanian eSveikata IPR advance-registration portal (`ipr.esveikata.lt`). Observed against the production patient frontend in September 2026.

## API origin

The current patient IPR frontend uses:

```text
https://ipr.esveikata.lt/api
```

Public appointment discovery and authenticated patient operations share this API surface.

## Authentication handoff from the patient portal

IPR uses its own short-lived Bearer JWT (`Authorization: Bearer ...`), but a second interactive identity login is not necessary when an authenticated eSveikata patient session already exists.

The observed browserless handoff is:

```text
authenticated pacientas.esveikata.lt session
  -> GET /api/patient/session/token
  -> POST https://ipr.esveikata.lt/api/authentications/jwt
       JSON body: {"token":"<short-lived handoff token>"}
  -> Authorization: Bearer <IPR JWT> response header
```

The short-lived handoff token should be used in memory and never logged or persisted.

The IPR JWT can then be checked/refreshed with:

```text
GET  /api/checks/session
POST /api/authentications/refresh
```

The refresh request uses the current Bearer token and the refreshed JWT is returned in the response `Authorization` header. For a private client, the safer design is to keep this short-lived IPR JWT only in process memory and recreate it from the patient session after restart or expiry.

## Public search endpoints

The following read endpoints were observed to work without an authenticated IPR JWT:

```text
GET /api/searchesNew/professions
GET /api/searchesNew/municipalities
GET /api/searches/appointments/times
GET /api/searches/appointments/times/details
GET /api/appointments/public
```

The search is hierarchical:

1. grouped service/organization availability;
2. practitioner/workplace details;
3. exact appointment slots.

`leftBound`, `rightBound` and appointment dates are epoch milliseconds in the current frontend/API.

A production quirk observed in September 2026: search envelopes can report `meta.totalPages = 0` and still emit a `links.next` value even when the next page is empty. The official frontend does not trust those fields to terminate pagination; it treats a page shorter than the requested page size as the last page. Clients that need completeness should mirror that behavior.

The practitioner-details request (`/searches/appointments/times/details`) should inherit the selected parent group's `healthcareServiceId`, `organizationId`, `fundType.type`, `referralNeed.type`, `appointmentMethodId` and date bounds. Omitting `fundType` currently produces HTTP 400.

Current search enum values observed in the frontend:

```text
paymentType:
  0 = all
  1 = Ligonių kasos / VLK
  2 = patient-paid

referralNeedType:
  0 = all
  1 = referral required
  2 = no referral required
```

## Referrals

Authenticated patient referral lists are loaded through:

```text
GET /api/referrals/active/all?patientId=...
GET /api/referrals/unused/all?patientId=...
```

Referral objects expose data useful for matching a referral to appointment search, including the target specialist profession/qualification code, diagnosis, referral date, notes and used/unused state.

## Appointment registration flow

The current frontend uses a temporary reservation before final registration:

```text
PUT    /api/appointments/reservations/{slotId}
GET    /api/appointments/reservations/{slotId}?serviceId={healthCareServiceId}
POST   /api/appointments/registrations
DELETE /api/appointments/reservations/{slotId}
```

The first request reserves the exact slot. The following GET loads the reservation/service requirements. Final registration is then submitted to `/appointments/registrations`.

The registration payload can include:

- `slotId`;
- `healthCareServiceId`;
- patient/user identifiers resolved from the authenticated session;
- patient contact data; exact phone/email requirements depend on the selected reception/service configuration;
- `referralCompositionId` when a referral is selected;
- appointment method and optional patient comments;
- optional questionnaire/task answers and files for services that require them.

A client should never assume every service has the same required fields. The reservation details should be inspected before final registration.

## Safety for personal tooling

Public search is read-only. Reservation and registration are consequential operations and should be separated from discovery. A local integration should require explicit approval of the exact institution, clinician, date/time and slot before reserving/registering, and it should provide a way to cancel a temporary reservation.

Never publish real JWTs, patient IDs, referral composition IDs, exact user appointment IDs, cookies, or medical payloads.
