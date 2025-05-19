# Examples

This page provides example FHIR Bundles to demonstrate regulatory workflows for two drug applications: a safety update and a shelf life update for the same drug (NDC: 12345-678-90). The examples align with the R2CS streaming architecture, using Apache Kafka to process JSON Bundles in real-time, and include:

- **Transaction Bundles** (`type: transaction`): Original submissions and approvals, containing Task, DocumentReference (inlining ePI, eAF, PQI Bundles in `content.attachment.data`), MedicinalProductDefinition, and Provenance.
- **Document Bundles** (`type: document`): ePI, eAF, FDA Requests for Information (RFIs), and company responses, containing Composition or DocumentReference, Task, and Provenance.
- **Collection Bundles** (`type: collection`): PQI content, inlined in submission Transaction Bundles.

All Bundles are uniquely identified (`Bundle.identifier`), linked to parent applications (`Task.basedOn`), and tracked by NDC codes (`MedicinalProductDefinition`). They are streamed as NDJSON to `r2cs-submissions` or `r2cs-notifications`, with sanitization (schema validation, tag whitelisting, SHA-256 hashes) to ensure security and compliance (e.g., 21 CFR Part 11, EU GMP Annex 11).

## Safety Update Workflow

### Safety Update Submission - Transaction Bundle

The pharmaceutical company submits the safety update on May 1, 2025, inlining ePI and eAF Document Bundles.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "safety-submission-2025-05-01"
  },
  "timestamp": "2025-05-01T09:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-safety-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-safety-001",
        "status": "requested",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "submit",
              "display": "Submit"
            }
          ]
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "authoredOn": "2025-05-01T09:00:00Z",
        "requester": {
          "reference": "Organization/pharma-co"
        },
        "owner": {
          "reference": "Organization/fda"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:docref-epi-safety-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "docref-epi-safety-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "34356-7",
              "display": "Product Label"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/json",
              "data": "ewogICJyZXNvdXJjZVR5cGUiOiAiQnVuZGxlIiwKICAidHlwZSI6ICJkb2N1bWVudCIsCiAgImlkZW50aWZpZXIiOiB7CiAgICAic3lzdGVtIjogImh0dHA6Ly9leGFtcGxlLm9yZy9yMmNzL3N1Ym1pc3Npb25zIiwKICAgICJ2YWx1ZSI6ICJzYWZldHktZXBpLTIwMjUtMDUtMDEiCiAgfSwKICAidGltZXN0YW1wIjogIjIwMjUtMDUtMDFUMDk6MDA6MDBaIiwKICAiZW50cnkiOiBbCiAgICB7CiAgICAgICJmdWxsVXJsIjogInVybjp1dWlkOmNvbXAtc2FmZXR5LTAwMSIsCiAgICAgICJyZXNvdXJjZSI6IHsKICAgICAgICAicmVzb3VyY2VUeXBlIjogIkNvbXBvc2l0aW9uIiwKICAgICAgICAiaWQiOiAiY29tcC1zYWZldHktMDAxIiwKICAgICAgICAic3RhdHVzIjogImZpbmFsIiwKICAgICAgICAidHlwZSI6IHsKICAgICAgICAgICJjb2RpbmciOiBbCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAic3lzdGVtIjogImh0dHA6Ly9sb2luYy5vcmciLAogICAgICAgICAgICAgICJjb2RlIjogIjM0MzU2LTciLAogICAgICAgICAgICAgICJkaXNwbGF5IjogIlByb2R1Y3QgTGFiZWwiCiAgICAgICAgICAgIH0KICAgICAgICAgIF0KICAgICAgICB9LAogICAgICAgICJzdWJqZWN0IjogewogICAgICAgICAgInJlZmVyZW5jZSI6ICJNZWRpY2luYWxQcm9kdWN0RGVmaW5pdGlvbi9tcGQtMTIzIgogICAgICAgIH0sCiAgICAgICAgInRpdGxlIjogImVQSSBTYWZldHkgVXBkYXRlIGZvciBbTWVkaWNpbmFsIFByb2R1Y3RdIiwKICAgICAgICAic2VjdGlvbiI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInRpdGxlIjogIldhcm5pbmdzIiwKICAgICAgICAgICAgInRleHQiOiB7CiAgICAgICAgICAgICAgInN0YXR1cyI6ICJnZW5lcmF0ZWQiLAogICAgICAgICAgICAgICJkaXYiOiAiPGRpdiB4bWxucz1cImh0dHA6Ly93d3cudzMub3JnLzE5OTkveGh0bWxcIj48cD5VcGRhdGVkIHdhcm5pbmc6IEluY3JlYXNlZCByaXNrIG9mIGRpemppbmVzcyBpbiBlbGRlcmx5IHBhdGllbnRzLjwvcD48L2Rpdj4iCiAgICAgICAgICAgIH0KICAgICAgICAgIH0KICAgICAgICBdCiAgICAgIH0KICAgIH0sCiAgICB7CiAgICAgICJmdWxsVXJsIjogInVybjp1dWlkOm1wZC0xMjMiLAogICAgICAicmVzb3VyY2UiOiB7CiAgICAgICAgInJlc291cmNlVHlwZSI6ICJNZWRpY2luYWxQcm9kdWN0RGVmaW5pdGlvbiIsCiAgICAgICAgImlkIjogIm1wZC0xMjMiLAogICAgICAgICJpZGVudGlmaWVyIjogWwogICAgICAgICAgewogICAgICAgICAgICAic3lzdGVtIjogImh0dHA6Ly9obDcub3JnL2ZoaXIvc2lkL25kYyIsCiAgICAgICAgICAgICJ2YWx1ZSI6ICIxMjM0NS02NzgtOTAiCiAgICAgICAgICB9CiAgICAgICAgXSwKICAgICAgICAibmFtZSI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInByb2R1Y3ROYW1lIjogIltNZWRpY2luYWwgUHJvZHVjdF0iCiAgICAgICAgICB9CiAgICAgICAgXQogICAgICB9CiAgICB9CiAgXQp9",
              "hash": "qZk+NkcGgWq6PiVxeFDCbJzQ2J0=/nE+3Y2c=",
              "size": 1234,
              "title": "ePI Safety Update Document Bundle"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "DocumentReference"
      }
    },
    {
      "fullUrl": "urn:uuid:docref-eaf-safety-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "docref-eaf-safety-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "51851-4",
              "display": "Administrative Form"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/json",
              "data": "ewogICJyZXNvdXJjZVR5cGUiOiAiQnVuZGxlIiwKICAidHlwZSI6ICJkb2N1bWVudCIsCiAgImlkZW50aWZpZXIiOiB7CiAgICAic3lzdGVtIjogImh0dHA6Ly9leGFtcGxlLm9yZy9yMmNzL3N1Ym1pc3Npb25zIiwKICAgICJ2YWx1ZSI6ICJzYWZldHktZWFmLTIwMjUtMDUtMDEiCiAgfSwKICAidGltZXN0YW1wIjogIjIwMjUtMDUtMDFUMDk6MDA6MDBaIiwKICAiZW50cnkiOiBbCiAgICB7CiAgICAgICJmdWxsVXJsIjogInVybjp1dWlkOmNvbXAtc2FmZXR5LWVhZi0wMDEiLAogICAgICAicmVzb3VyY2UiOiB7CiAgICAgICAgInJlc291cmNlVHlwZSI6ICJDb21wb3NpdGlvbiIsCiAgICAgICAgImlkIjogImNvbXAtc2FmZXR5LWVhZi0wMDEiLAogICAgICAgICJzdGF0dXMiOiAiZmluYWwiLAogICAgICAgICJ0eXBlIjogewogICAgICAgICAgImNvZGluZyI6IFsKICAgICAgICAgICAgewogICAgICAgICAgICAgICJzeXN0ZW0iOiAiaHR0cDovL2xvaW5jLm9yZyIsCiAgICAgICAgICAgICAgImNvZGUiOiAiNTE4NTEtNCIsCiAgICAgICAgICAgICAgImRpc3BsYXkiOiAiQWRtaW5pc3RyYXRpdmUgRm9ybSIKICAgICAgICAgICAgfQogICAgICAgICAgXQogICAgICAgIH0sCiAgICAgICAgInN1YmplY3QiOiB7CiAgICAgICAgICAicmVmZXJlbmNlIjogIk1lZGljaW5hbFByb2R1Y3REZWZpbml0aW9uL21wZC0xMjMiCiAgICAgICAgfSwKICAgICAgICAidGl0bGUiOiAiZUFGPiBmb3IgU2FmZXR5IFVwZGF0ZSBvZiBbTWVkaWNpbmFsIFByb2R1Y3RdIiwKICAgICAgICAic2VjdGlvbiI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInRpdGxlIjogIkFwcGxpY2F0aW9uIERldGFpbHMiLAogICAgICAgICAgICAidGV4dCI6IHsKICAgICAgICAgICAgICAic3RhdHVzIjogImdlbmVyYXRlZCIsCiAgICAgICAgICAgICAgImRpdiI6ICI8ZGl2IHhtbG5zPVwiaHR0cDovL3d3dy53My5vcmcvMTk5OS94aHRtbFwiPjxwPkFwcGxpY2F0aW9uIFR5cGU6IExhYmVsIFNhZmV0eSBVcGRhdGU8L3A+PHA+U3VibWlzc2lvbiBEYXRlOiAyMDI1LTA1LTAxPC9wPjxwPlJlbGF0ZWQgZVBJOiBzYWZldHktZXBpLTIwMjUtMDUtMDE8L3A+PC9kaXY+IgogICAgICAgICAgICB9CiAgICAgICAgICB9CiAgICAgICAgXQogICAgICB9CiAgICB9LAogICAgewogICAgICAiZnVsbFVybCI6ICJ1cm46dXVpZDptcGQtMTIzIiwKICAgICAgInJlc291cmNlIjogewogICAgICAgICJyZXNvdXJjZVR5cGUiOiAiTWVkaWNpbmFsUHJvZHVjdERlZmluaXRpb24iLAogICAgICAgICJpZCI6ICJtcGQtMTIzIiwKICAgICAgICAiaWRlbnRpZmllciI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInN5c3RlbSI6ICJodHRwOi8vaGw3Lm9yZy9maGlyL3NpZC9uZGMiLAogICAgICAgICAgICAidmFsdWUiOiAiMTIzNDUtNjc4LTkwIgogICAgICAgICAgfQogICAgICAgIF0sCiAgICAgICAgIm5hbWUiOiBbCiAgICAgICAgICB7CiAgICAgICAgICAgICJwcm9kdWN0TmFtZSI6ICJbTWVkaWNpbmFsIFByb2R1Y3RdIgogICAgICAgICAgfQogICAgICAgIF0KICAgICAgfQogICAgfQogIF0KfQ==",
              "hash": "rL0Y20zC+Fzt72VPzMsf7oQhAewv3fG4=/pX+3Z2d=",
              "size": 987,
              "title": "eAF Safety Update Document Bundle"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "DocumentReference"
      }
    },
    {
      "fullUrl": "urn:uuid:mpd-123",
      "resource": {
        "resourceType": "MedicinalProductDefinition",
        "id": "mpd-123",
        "identifier": [
          {
            "system": "http://hl7.org/fhir/sid/ndc",
            "value": "12345-678-90"
          }
        ],
        "name": [
          {
            "productName": "[Medicinal Product]"
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "MedicinalProductDefinition"
      }
    },
    {
      "fullUrl": "urn:uuid:prov-safety-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-safety-001",
        "target": [
          {
            "reference": "urn:uuid:task-safety-001"
          },
          {
            "reference": "urn:uuid:docref-epi-safety-001"
          },
          {
            "reference": "urn:uuid:docref-eaf-safety-001"
          }
        ],
        "recorded": "2025-05-01T09:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/pharma-co"
            },
            "onBehalfOf": {
              "reference": "Organization/fda"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "CREATE"
            }
          ]
        }
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    }
  ]
}
```

### FDA Request for Information - Safety Update 
FDA requests additional data on May 3, 2025.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "safety-rfi-2025-05-03"
  },
  "timestamp": "2025-05-03T14:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:comp-safety-rfi-001",
      "resource": {
        "resourceType": "Composition",
        "id": "comp-safety-rfi-001",
        "status": "final",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "74468-0",
              "display": "Question Response"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "title": "RFI for Safety Update of [Medicinal Product]",
        "section": [
          {
            "title": "Request Details",
            "text": {
              "status": "generated",
              "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><p>Please provide clinical data supporting the dizziness warning update.</p></div>"
            }
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:task-safety-rfi-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-safety-rfi-001",
        "status": "requested",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "review",
              "display": "Review"
            }
          ]
        },
        "focus": {
          "reference": "urn:uuid:comp-safety-rfi-001"
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "basedOn": [
          {
            "reference": "Task/task-safety-001"
          }
        ],
        "authoredOn": "2025-05-03T14:00:00Z",
        "requester": {
          "reference": "Organization/fda"
        },
        "owner": {
          "reference": "Organization/pharma-co"
        }
      }
    },
    {
      "fullUrl": "urn:uuid:mpd-123",
      "resource": {
        "resourceType": "MedicinalProductDefinition",
        "id": "mpd-123",
        "identifier": [
          {
            "system": "http://hl7.org/fhir/sid/ndc",
            "value": "12345-678-90"
          }
        ],
        "name": [
          {
            "productName": "[Medicinal Product]"
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:prov-safety-rfi-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-safety-rfi-001",
        "target": [
          {
            "reference": "urn:uuid:comp-safety-rfi-001"
          },
          {
            "reference": "urn:uuid:task-safety-rfi-001"
          }
        ],
        "recorded": "2025-05-03T14:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/fda"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "UPDATE"
            }
          ]
        }
      }
    }
  ]
}
```

### Company Response to RFI - Safety Update
The company responds with clinical data on May 5, 2025.

```json
{
  "resourceType": "Bundle",
  "type": "document",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "safety-response-2025-05-05"
  },
  "timestamp": "2025-05-05T10:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:docref-safety-response-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "docref-safety-response-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "55107-7",
              "display": "Supporting Documentation"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/pdf",
              "url": "http://example.org/clinical-data.pdf",
              "title": "Clinical Data for Dizziness Warning"
            }
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:task-safety-response-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-safety-response-001",
        "status": "completed",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "fulfill",
              "display": "Fulfill"
            }
          ]
        },
        "focus": {
          "reference": "urn:uuid:docref-safety-response-001"
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "basedOn": [
          {
            "reference": "Task/task-safety-rfi-001"
          }
        ],
        "authoredOn": "2025-05-05T10:00:00Z",
        "requester": {
          "reference": "Organization/pharma-co"
        },
        "owner": {
          "reference": "Organization/fda"
        }
      }
    },
    {
      "fullUrl": "urn:uuid:mpd-123",
      "resource": {
        "resourceType": "MedicinalProductDefinition",
        "id": "mpd-123",
        "identifier": [
          {
            "system": "http://hl7.org/fhir/sid/ndc",
            "value": "12345-678-90"
          }
        ],
        "name": [
          {
            "productName": "[Medicinal Product]"
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:prov-safety-response-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-safety-response-001",
        "target": [
          {
            "reference": "urn:uuid:docref-safety-response-001"
          },
          {
            "reference": "urn:uuid:task-safety-response-001"
          }
        ],
        "recorded": "2025-05-05T10:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/pharma-co"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "UPDATE"
            }
          ]
        }
      }
    }
  ]
}
```

### FDA Approval - Safety Update
FDA approves the safety update on May 7, 2025.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "safety-approval-2025-05-07"
  },
  "timestamp": "2025-05-07T16:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-safety-approval-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-safety-approval-001",
        "status": "completed",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "approve",
              "display": "Approve"
            }
          ]
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "basedOn": [
          {
            "reference": "Task/task-safety-001"
          }
        ],
        "authoredOn": "2025-05-07T16:00:00Z",
        "requester": {
          "reference": "Organization/fda"
        },
        "owner": {
          "reference": "Organization/pharma-co"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:prov-safety-approval-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-safety-approval-001",
        "target": [
          {
            "reference": "urn:uuid:task-safety-approval-001"
          }
        ],
        "recorded": "2025-05-07T16:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/fda"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "UPDATE"
            }
          ]
        }
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    }
  ]
}
```

## Shelf Life Update Workflow
### Shelf Life Update Submission - Transaction Bundle
The company submits the shelf life update on May 8, 2025, inlining ePI, eAF, and PQI Bundles.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "shelf-life-submission-2025-05-08"
  },
  "timestamp": "2025-05-08T09:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-shelf-life-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-shelf-life-001",
        "status": "requested",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "submit",
              "display": "Submit"
            }
          ]
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "authoredOn": "2025-05-08T09:00:00Z",
        "requester": {
          "reference": "Organization/pharma-co"
        },
        "owner": {
          "reference": "Organization/fda"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:docref-epi-shelf-life-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "docref-epi-shelf-life-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "34356-7",
              "display": "Product Label"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/json",
              "data": "ewogICJyZXNvdXJjZVR5cGUiOiAiQnVuZGxlIiwKICAidHlwZSI6ICJkb2N1bWVudCIsCiAgImlkZW50aWZpZXIiOiB7CiAgICAic3lzdGVtIjogImh0dHA6Ly9leGFtcGxlLm9yZy9yMmNzL3N1Ym1pc3Npb25zIiwKICAgICJ2YWx1ZSI6ICJzZWxmLWxpZmUtZXBpLTIwMjUtMDUtMDgiCiAgfSwKICAidGltZXN0YW1wIjogIjIwMjUtMDUtMDhUMDk6MDA6MDBaIiwKICAiZW50cnkiOiBbCiAgICB7CiAgICAgICJmdWxsVXJsIjogInVybjp1dWlkOmNvbXAtc2hlbGYtbGlmZS0wMDEiLAogICAgICAicmVzb3VyY2UiOiB7CiAgICAgICAgInJlc291cmNlVHlwZSI6ICJDb21wb3NpdGlvbiIsCiAgICAgICAgImlkIjogImNvbXAtc2hlbGYtbGlmZS0wMDEiLAogICAgICAgICJzdGF0dXMiOiAiZmluYWwiLAogICAgICAgICJ0eXBlIjogewogICAgICAgICAgImNvZGluZyI6IFsKICAgICAgICAgICAgewogICAgICAgICAgICAgICJzeXN0ZW0iOiAiaHR0cDovL2xvaW5jLm9yZyIsCiAgICAgICAgICAgICAgImNvZGUiOiAiMzQzNTYtNyIsCiAgICAgICAgICAgICAgImRpc3BsYXkiOiAiUHJvZHVjdCBMYWJlbCIKICAgICAgICAgICAgfQogICAgICAgICAgXQogICAgICAgIH0sCiAgICAgICAgInN1YmplY3QiOiB7CiAgICAgICAgICAicmVmZXJlbmNlIjogIk1lZGljaW5hbFByb2R1Y3REZWZpbml0aW9uL21wZC0xMjMiCiAgICAgICAgfSwKICAgICAgICAidGl0bGUiOiAiZVBPIFNoZWxmIExpZmUgVXBkYXRlIGZvciBbTWVkaWNpbmFsIFByb2R1Y3RdIiwKICAgICAgICAic2VjdGlvbiI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInRpdGxlIjogIlN0b3JhZ2UiLAogICAgICAgICAgICAidGV4dCI6IHsKICAgICAgICAgICAgICAic3RhdHVzIjogImdlbmVyYXRlZCIsCiAgICAgICAgICAgICAgImRpdiI6ICI8ZGl2IHhtbG5zPVwiaHR0cDovL3d3dy53My5vcmcvMTk5OS94aHRtbFwiPjxwPlVwZGF0ZWQgc2hlbGYgbGlmZTogMzYgbW9udGhzIGF0IDI1wrBDLjwvcD48L2Rpdj4iCiAgICAgICAgICAgIH0KICAgICAgICAgIH0KICAgICAgICBdCiAgICAgIH0KICAgIH0sCiAgICB7CiAgICAgICJmdWxsVXJsIjogInVybjp1dWlkOm1wZC0xMjMiLAogICAgICAicmVzb3VyY2UiOiB7CiAgICAgICAgInJlc291cmNlVHlwZSI6ICJNZWRpY2luYWxQcm9kdWN0RGVmaW5pdGlvbiIsCiAgICAgICAgImlkIjogIm1wZC0xMjMiLAogICAgICAgICJpZGVudGlmaWVyIjogWwogICAgICAgICAgewogICAgICAgICAgICAic3lzdGVtIjogImh0dHA6Ly9obDcub3JnL2ZoaXIvc2lkL25kYyIsCiAgICAgICAgICAgICJ2YWx1ZSI6ICIxMjM0NS02NzgtOTAiCiAgICAgICAgICB9CiAgICAgICAgXSwKICAgICAgICAibmFtZSI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInByb2R1Y3ROYW1lIjogIltNZWRpY2luYWwgUHJvZHVjdF0iCiAgICAgICAgICB9CiAgICAgICAgXQogICAgICB9CiAgICB9CiAgXQp9",
              "hash": "aB9+MjdFhWq7OiWxeGDCcKzR3J1=/mF+4Z3e=",
              "size": 1456,
              "title": "ePI Shelf Life Update Document Bundle"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "DocumentReference"
      }
    },
    {
      "fullUrl": "urn:uuid:docref-eaf-shelf-life-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "docref-eaf-shelf-life-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "51851-4",
              "display": "Administrative Form"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/json",
              "data": "ewogICJyZXNvdXJjZVR5cGUiOiAiQnVuZGxlIiwKICAidHlwZSI6ICJkb2N1bWVudCIsCiAgImlkZW50aWZpZXIiOiB7CiAgICAic3lzdGVtIjogImh0dHA6Ly9leGFtcGxlLm9yZy9yMmNzL3N1Ym1pc3Npb25zIiwKICAgICJ2YWx1ZSI6ICJzZWxmLWxpZmUtZWFmLTIwMjUtMDUtMDgiCiAgfSwKICAidGltZXN0YW1wIjogIjIwMjUtMDUtMDhUMDk6MDA6MDBaIiwKICAiZW50cnkiOiBbCiAgICB7CiAgICAgICJmdWxsVXJsIjogInVybjp1dWlkOmNvbXAtc2hlbGYtbGlmZS1lYWYtMDAxIiwKICAgICAgInJlc291cmNlIjogewogICAgICAgICJyZXNvdXJjZVR5cGUiOiAiQ29tcG9zaXRpb24iLAogICAgICAgICJpZCI6ICJjb21wLXNoZWxmLWxpZmUtZWFmLTAwMSIsCiAgICAgICAgInN0YXR1cyI6ICJmaW5hbCIsCiAgICAgICAgInR5cGUiOiB7CiAgICAgICAgICAiY29kaW5nIjogWwogICAgICAgICAgICB7CiAgICAgICAgICAgICAgInN5c3RlbSI6ICJodHRwOi8vbG9pbmMub3JnIiwKICAgICAgICAgICAgICAiY29kZSI6ICI1MTg1MS00IiwKICAgICAgICAgICAgICAiZGlzcGxheSI6ICJBZG1pbmlzdHJhdGl2ZSBGb3JtIgogICAgICAgICAgICB9CiAgICAgICAgICBdCiAgICAgICAgfSwKICAgICAgICAgInN1YmplY3QiOiB7CiAgICAgICAgICAicmVmZXJlbmNlIjogIk1lZGljaW5hbFByb2R1Y3REZWZpbml0aW9uL21wZC0xMjMiCiAgICAgICAgfSwKICAgICAgICAgInRpdGxlIjogImVBRiBmb3IgU2hlbGYgTGlmZSBVcGRhdGUgb2YgW01lZGljaW5hbCBQcm9kdWN0XSIsCiAgICAgICAgInNlY3Rpb24iOiBbCiAgICAgICAgICB7CiAgICAgICAgICAgICJ0aXRsZSI6ICJBcHBsaWNhdGlvbiBEZXRhaWxzIiwKICAgICAgICAgICAgInRleHQiOiB7CiAgICAgICAgICAgICAgInN0YXR1cyI6ICJnZW5lcmF0ZWQiLAogICAgICAgICAgICAgICJkaXYiOiAiPGRpdiB4bWxucz1cImh0dHA6Ly93d3cudzMub3JnLzE5OTkveGh0bWxcIj48cD5BcHBsaWNhdGlvbiBUeXBlOiBTaGVsZiBMaWZlIFVwZGF0ZTwvcD48cD5TdWJtaXNzaW9uIERhdGU6IDIwMjUtMDUtMDg8L3A+PHA+UmVsYXRlZCBlUEk6IHNoZWxmLWxpZmUtZXBpLTIwMjUtMDUtMDg8L3A+PC9kaXY+IgogICAgICAgICAgICB9CiAgICAgICAgICB9CiAgICAgICAgXQogICAgICB9CiAgICB9LAogICAgewogICAgICAiZnVsbFVybCI6ICJ1cm46dXVpZDptcGQtMTIzIiwKICAgICAgInJlc291cmNlIjogewogICAgICAgICJyZXNvdXJjZVR5cGUiOiAiTWVkaWNpbmFsUHJvZHVjdERlZmluaXRpb24iLAogICAgICAgICJpZCI6ICJtcGQtMTIzIiwKICAgICAgICAiaWRlbnRpZmllciI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInN5c3RlbSI6ICJodHRwOi8vaGw3Lm9yZy9maGlyL3NpZC9uZGMiLAogICAgICAgICAgICAidmFsdWUiOiAiMTIzNDUtNjc4LTkwIgogICAgICAgICAgfQogICAgICAgIF0sCiAgICAgICAgIm5hbWUiOiBbCiAgICAgICAgICB7CiAgICAgICAgICAgICJwcm9kdWN0TmFtZSI6ICJbTWVkaWNpbmFsIFByb2R1Y3RdIgogICAgICAgICAgfQogICAgICAgIF0KICAgICAgfQogICAgfQogIF0KfQ==",
              "hash": "sK8+MjdFiWq8PjWyeHEDdLzS4K2=/nG+5Z4f=",
              "size": 1098,
              "title": "eAF Shelf Life Update Document Bundle"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "DocumentReference"
      }
    },
    {
      "fullUrl": "urn:uuid:docref-pqi-shelf-life-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "docref-pqi-shelf-life-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://example.org/r2cs",
              "code": "stability",
              "display": "Stability Update"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/json",
              "data": "ewogICJyZXNvdXJjZVR5cGUiOiAiQnVuZGxlIiwKICAidHlwZSI6ICJjb2xsZWN0aW9uIiwKICAiaWRlbnRpZmllciI6IHsKICAgICJzeXN0ZW0iOiAiaHR0cDovL2V4YW1wbGUub3JnL3IyY3Mvc3VibWlzc2lvbnMiLAogICAgInZhbHVlIjogInNoZWxmLWxpZmUtcHFpLTIwMjUtMDUtMDgiCiAgfSwKICAidGltZXN0YW1wIjogIjIwMjUtMDUtMDhUMDk6MDA6MDBaIiwKICAiZW50cnkiOiBbCiAgICB7CiAgICAgICJmdWxsVXJsIjogInVybjp1dWlkOnJlZy1hdXRoLXNoZWxmLWxpZmUtMDAxIiwKICAgICAgInJlc291cmNlIjogewogICAgICAgICJyZXNvdXJjZVR5cGUiOiAiUmVndWxhdGVkQXV0aG9yaXphdGlvbiIsCiAgICAgICAgImlkIjogInJlZy1hdXRoLXNoZWxmLWxpZmUtMDAxIiwKICAgICAgICAic3RhdHVzIjogewogICAgICAgICAgImNvZGluZyI6IFsKICAgICAgICAgICAgewogICAgICAgICAgICAgICJzeXN0ZW0iOiAiaHR0cDovL2hsNy5vcmcvZmhpci9wdWJsaWNhdGlvbi1zdGF0dXMiLAogICAgICAgICAgICAgICJjb2RlIjogImFjdGl2ZSIKICAgICAgICAgICAgfQogICAgICAgICAgXQogICAgICAgIH0sCiAgICAgICAgInN1YmplY3QiOiB7CiAgICAgICAgICAicmVmZXJlbmNlIjogIk1lZGljaW5hbFByb2R1Y3REZWZpbml0aW9uL21wZC0xMjMiCiAgICAgICAgfSwKICAgICAgICAidHlwZSI6IHsKICAgICAgICAgICJjb2RpbmciOiBbCiAgICAgICAgICAgIHsKICAgICAgICAgICAgICAic3lzdGVtIjogImh0dHA6Ly9leGFtcGxlLm9yZy9yMmNzIiwKICAgICAgICAgICAgICAiY29kZSI6ICJzdGFiaWxpdHkiLAogICAgICAgICAgICAgICJkaXNwbGF5IjogIlN0YWJpbGl0eSBVcGRhdGUiCiAgICAgICAgICAgIH0KICAgICAgICAgIF0KICAgICAgICB9LAogICAgICAgICJkZXNjcmlwdGlvbiI6ICJTdGFiaWxpdHkgZGF0YSBzdXBwb3J0aW5nIDM2LW1vbnRoIHNoZWxmIGxpZmUgYXQgMjXCsEMuIgogICAgICB9CiAgICB9LAogICAgewogICAgICAiZnVsbFVybCI6ICJ1cm46dXVpZDptcGQtMTIzIiwKICAgICAgInJlc291cmNlIjogewogICAgICAgICJyZXNvdXJjZVR5cGUiOiAiTWVkaWNpbmFsUHJvZHVjdERlZmluaXRpb24iLAogICAgICAgICJpZCI6ICJtcGQtMTIzIiwKICAgICAgICAiaWRlbnRpZmllciI6IFsKICAgICAgICAgIHsKICAgICAgICAgICAgInN5c3RlbSI6ICJodHRwOi8vaGw3Lm9yZy9maGlyL3NpZC9uZGMiLAogICAgICAgICAgICAidmFsdWUiOiAiMTIzNDUtNjc4LTkwIgogICAgICAgICAgfQogICAgICAgIF0sCiAgICAgICAgIm5hbWUiOiBbCiAgICAgICAgICB7CiAgICAgICAgICAgICJwcm9kdWN0TmFtZSI6ICJbTWVkaWNpbmFsIFByb2R1Y3RdIgogICAgICAgICAgfQogICAgICAgIF0KICAgICAgfQogICAgfQogIF0KfQ==",
              "hash": "tM0Y30zD+Fzt83WPzNsG7oRiBexw4J3=/oI+6Z5g=",
              "size": 876,
              "title": "PQI Shelf Life Update Collection Bundle"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "DocumentReference"
      }
    },
    {
      "fullUrl": "urn:uuid:mpd-123",
      "resource": {
        "resourceType": "MedicinalProductDefinition",
        "id": "mpd-123",
        "identifier": [
          {
            "system": "http://hl7.org/fhir/sid/ndc",
            "value": "12345-678-90"
          }
        ],
        "name": [
          {
            "productName": "[Medicinal Product]"
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "MedicinalProductDefinition"
      }
    },
    {
      "fullUrl": "urn:uuid:prov-shelf-life-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-shelf-life-001",
        "target": [
          {
            "reference": "urn:uuid:task-shelf-life-001"
          },
          {
            "reference": "urn:uuid:docref-epi-shelf-life-001"
          },
          {
            "reference": "urn:uuid:docref-eaf-shelf-life-001"
          },
          {
            "reference": "urn:uuid:docref-pqi-shelf-life-001"
          }
        ],
        "recorded": "2025-05-08T09:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/pharma-co"
            },
            "onBehalfOf": {
              "reference": "Organization/fda"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "CREATE"
            }
          ]
        }
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    }
  ]
}
```

### FDA Request for Information - Shelf Life Update
FDA requests additional stability data on May 10, 2025.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "shelf-life-rfi-2025-05-10"
  },
  "timestamp": "2025-05-10T14:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:comp-shelf-life-rfi-001",
      "resource": {
        "resourceType": "Composition",
        "id": "comp-shelf-life-rfi-001",
        "status": "final",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "74468-0",
              "display": "Question Response"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "title": "RFI for Shelf Life Update of [Medicinal Product]",
        "section": [
          {
            "title": "Request Details",
            "text": {
              "status": "generated",
              "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><p>Please provide additional stability test results for 36-month shelf life.</p></div>"
            }
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:task-shelf-life-rfi-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-shelf-life-rfi-001",
        "status": "requested",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "review",
              "display": "Review"
            }
          ]
        },
        "focus": {
          "reference": "urn:uuid:comp-shelf-life-rfi-001"
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "basedOn": [
          {
            "reference": "Task/task-shelf-life-001"
          }
        ],
        "authoredOn": "2025-05-10T14:00:00Z",
        "requester": {
          "reference": "Organization/fda"
        },
        "owner": {
          "reference": "Organization/pharma-co"
        }
      }
    },
    {
      "fullUrl": "urn:uuid:mpd-123",
      "resource": {
        "resourceType": "MedicinalProductDefinition",
        "id": "mpd-123",
        "identifier": [
          {
            "system": "http://hl7.org/fhir/sid/ndc",
            "value": "12345-678-90"
          }
        ],
        "name": [
          {
            "productName": "[Medicinal Product]"
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:prov-shelf-life-rfi-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-shelf-life-rfi-001",
        "target": [
          {
            "reference": "urn:uuid:comp-shelf-life-rfi-001"
          },
          {
            "reference": "urn:uuid:task-shelf-life-rfi-001"
          }
        ],
        "recorded": "2025-05-10T14:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/fda"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "UPDATE"
            }
          ]
        }
      }
    }
  ]
}
```

### Company Response to RFI - Shelf Life Update
The company responds with stability data on May 12, 2025.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "shelf-life-response-2025-05-12"
  },
  "timestamp": "2025-05-12T10:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:docref-shelf-life-response-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "docref-shelf-life-response-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "55107-7",
              "display": "Supporting Documentation"
            }
          ]
        },
        "subject": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/pdf",
              "url": "http://example.org/stability-data.pdf",
              "title": "Additional Stability Test Results"
            }
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:task-shelf-life-response-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-shelf-life-response-001",
        "status": "completed",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "fulfill",
              "display": "Fulfill"
            }
          ]
        },
        "focus": {
          "reference": "urn:uuid:docref-shelf-life-response-001"
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "basedOn": [
          {
            "reference": "Task/task-shelf-life-rfi-001"
          }
        ],
        "authoredOn": "2025-05-12T10:00:00Z",
        "requester": {
          "reference": "Organization/pharma-co"
        },
        "owner": {
          "reference": "Organization/fda"
        }
      }
    },
    {
      "fullUrl": "urn:uuid:mpd-123",
      "resource": {
        "resourceType": "MedicinalProductDefinition",
        "id": "mpd-123",
        "identifier": [
          {
            "system": "http://hl7.org/fhir/sid/ndc",
            "value": "12345-678-90"
          }
        ],
        "name": [
          {
            "productName": "[Medicinal Product]"
          }
        ]
      }
    },
    {
      "fullUrl": "urn:uuid:prov-shelf-life-response-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-shelf-life-response-001",
        "target": [
          {
            "reference": "urn:uuid:docref-shelf-life-response-001"
          },
          {
            "reference": "urn:uuid:task-shelf-life-response-001"
          }
        ],
        "recorded": "2025-05-12T10:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/pharma-co"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "UPDATE"
            }
          ]
        }
      }
    }
  ]
}
```

### FDA Approval - Shelf Life Update (Transaction Bundle)
FDA approves the shelf life update on May 14, 2025.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "identifier": {
    "system": "http://example.org/r2cs/submissions",
    "value": "shelf-life-approval-2025-05-14"
  },
  "timestamp": "2025-05-14T16:00:00Z",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-shelf-life-approval-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-shelf-life-approval-001",
        "status": "completed",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/CodeSystem/task-code",
              "code": "approve",
              "display": "Approve"
            }
          ]
        },
        "for": {
          "reference": "MedicinalProductDefinition/mpd-123"
        },
        "basedOn": [
          {
            "reference": "Task/task-shelf-life-001"
          }
        ],
        "authoredOn": "2025-05-14T16:00:00Z",
        "requester": {
          "reference": "Organization/fda"
        },
        "owner": {
          "reference": "Organization/pharma-co"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:prov-shelf-life-approval-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "prov-shelf-life-approval-001",
        "target": [
          {
            "reference": "urn:uuid:task-shelf-life-approval-001"
          }
        ],
        "recorded": "2025-05-14T16:00:00Z",
        "agent": [
          {
            "who": {
              "reference": "Organization/fda"
            }
          }
        ],
        "activity": {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v3-DataOperation",
              "code": "UPDATE"
            }
          ]
        }
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    }
  ]
}
```