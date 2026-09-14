# Patient portal API

Observed base URL:

```text
https://pacientas.esveikata.lt/api/patient
```

All examples below assume an authenticated patient portal browser session. The list is incomplete.

## Session

```text
GET /session/current
GET /session/patient
GET /session/representative
GET /session/token
```

`/session/token` is used by the patient portal to bootstrap the separate ELAB application. Treat its value as a secret.

## Patient data

```text
GET /patients/current/withRelatedPersons
GET /patients/current/diagnosisForList
GET /patients/current/allergiesForList
GET /patients/current/vaccinations
GET /patients/current/activeAllergies
GET /patients/current/lastDiagnosis
```

## Encounters

Observed list pattern:

```text
GET /encounters?patient=current&page=<n>&count=<n>&searchDocsCount=true
```

Responses are structured JSON and include encounter metadata rather than rendered HTML.

## Laboratory / research results

Dashboard-style research result list:

```text
GET /patients/researchResultsForList?count=<n>
```

Observed response shape includes a `documents` array. Frontend code reads fields such as:

```text
id
date
additionalData.laboratoryResearch.name
additionalData.specimen.name
```

General document listing used by the laboratory-results page:

```text
GET /documents2/forList?appendTotals=true&page=1&count=10&patient=current&docType=e200&docType=e200a&docType=e014&docType=e014a
```

A concrete structured document can be fetched with:

```text
GET /documents2/{documentId}
```

The observed document JSON can contain order data, laboratory research metadata, specimen/storage information, encounter metadata, author/custodian/organization information and document status.

PDF-related frontend helpers construct:

```text
GET /documents2/{documentId}/pdf
GET /documents2/{documentId}/pdfSealed
```

## Patient summary

```text
GET /patient-summary/current/has-summary
GET /patient-summary/current/summary/history/last
```

## ePrescription

Observed patient-side list pattern:

```text
GET /erx/patient/MedicationPrescription/documents?page=1&count=10&sort=DATE
```

## Notes

- Most patient portal reads are cookie-authenticated JSON requests.
- The frontend frequently uses the literal patient selector `current` instead of exposing a patient identifier in the URL.
- Do not assume a route is stable merely because it exists in a frontend bundle.
- Use the least-privileged read-only path needed for interoperability.
