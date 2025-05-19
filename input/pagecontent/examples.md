### Overview

This section provides example FHIR resources and bundles to illustrate the RECON workflow for submitting, updating, and monitoring regulatory content, such as electronic Product Information (ePI) and Product Quality Information (PQI). These examples demonstrate the use of `Task`, `DocumentReference`, `Provenance`, `OperationOutcome`, and `Subscription` resources, with clear identification of the sender (submitter) and recipient (regulator).

All examples conform to the [R2CS Profiles](https://build.fhir.org/ig/cander2/recon-ig/profiles.html). Downloadable JSON files are available on the [Downloads](https://build.fhir.org/ig/cander2/recon-ig/downloads.html) page.

### Example 1: ePI Submission Bundle

This `Transaction Bundle` submits an ePI document, identifying the sender (`PharmaCorp Inc.`) and recipient (`Regulator`).

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "resource": {
        "resourceType": "Task",
        "id": "task-123",
        "status": "requested",
        "intent": "order",
        "requester": {"reference": "Organization/pharma-co-123", "display": "PharmaCorp Inc."},
        "owner": {"reference": "Organization/regulator-456", "display": "Regulator"},
        "focus": {"reference": "DocumentReference/ePI-123"}
      },
      "request": {"method": "POST", "url": "Task"}
    },
    {
      "resource": {
        "resourceType": "DocumentReference",
        "id": "ePI-123",
        "status": "current",
        "content": [{"attachment": {"contentType": "application/pdf", "url": "http://example.com/ePI.pdf"}}]
      },
      "request": {"method": "POST", "url": "DocumentReference"}
    },
    {
      "resource": {
        "resourceType": "Provenance",
        "target": [
          {"reference": "Task/task-123"},
          {"reference": "DocumentReference/ePI-123"}
        ],
        "recorded": "2025-05-12T10:00:00Z",
        "agent": [
          {
            "type": {
              "coding": [
                {"system": "http://terminology.hl7.org/CodeSystem/provenance-participant-type", "code": "author"}
              ]
            },
            "who": {"reference": "Organization/pharma-co-123", "display": "PharmaCorp Inc."}
          },
          {
            "type": {
              "coding": [
                {"system": "http://terminology.hl7.org/CodeSystem/provenance-participant-type", "code": "performer"}
              ]
            },
            "who": {"reference": "Organization/regulator-456", "display": "Regulator"}
          }
        ]
      },
      "request": {"method": "POST", "url": "Provenance"}
    },
    {
      "resource": {
        "resourceType": "Organization",
        "id": "pharma-co-123",
        "identifier": [
          {"system": "urn:oid:1.3.6.1.4.1.343", "value": "pharma-123"}
        ],
        "name": "PharmaCorp Inc."
      },
      "request": {"method": "POST", "url": "Organization"}
    },
    {
      "resource": {
        "resourceType": "Organization",
        "id": "regulator-456",
        "identifier": [
          {"system": "urn:oid:1.3.6.1.4.1.343", "value": "Regulator-456"}
        ],
        "name": "European Medicines Agency"
      },
      "request": {"method": "POST", "url": "Organization"}
    }
  ]
}
```

### Example 2: Task Update Bundle

This Transaction Bundle updates a Task to completed status with an OperationOutcome output, indicating approval of the ePI.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "resource": {
        "resourceType": "Task",
        "id": "task-123",
        "status": "completed",
        "intent": "order",
        "requester": {"reference": "Organization/pharma-co-123"},
        "owner": {"reference": "Organization/regulator-456"},
        "output": [
          {
            "type": {"coding": [{"system": "http://hl7.org/fhir/task-output-type", "code": "result"}]},
            "valueReference": {"reference": "OperationOutcome/oo-123"}
          }
        ]
      },
      "request": {"method": "PUT", "url": "Task/task-123"}
    },
    {
      "resource": {
        "resourceType": "OperationOutcome",
        "id": "oo-123",
        "issue": [{"severity": "information", "code": "informational", "details": {"text": "ePI approved"}}]
      },
      "request": {"method": "POST", "url": "OperationOutcome"}
    }
  ]
}
```

### Example 3: Subscription for Notifications

This Subscription resource enables real-time notifications for task updates, using a REST-hook channel.

```json
{
  "resourceType": "Subscription",
  "status": "requested",
  "criteria": "Task?status=completed",
  "channel": {
    "type": "rest-hook",
    "endpoint": "https://submitter.example.com/notification",
    "payload": "application/fhir+json"
  }
}
```




















