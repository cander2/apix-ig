This page outlines a scenario demonstrating the use of the **API Exchange IG** and **Structured Regulatory Correspondence IG** for a Type II variation submission to extend the shelf life of a biologic product.

### Scenario Overview

This scenario is for a Type II variation to extend the shelf life of a biologic product from 24 to 36 months. The variation requires updates to labeling (ePI) and pharmaceutical quality (PQI) content. The submission, or `Transaction Bundle`, includes a `Task`, a `DocumentReference` with base64-encoded attachments (application form as a Collection Bundle, ePI Document Bundle, PQI Collection Bundle), and a `Provenance`. The EMA processes the `Transaction Bundle`, sends a `Transaction Bundle` back which includes a LoQ with 10 questions (2 administrative, 3 labeling, 5 CMC) as a `Questionnaire`. PharmaInc responds with a `Transaction Bundle` that includes a `QuestionnaireResponse` (including an SVG image and multi-paragraph text for specific answers). Finally, the EMA issues a Decision Letter `Document Bundle` within a final 'Transaction Bundle'. All exchanges use RESTful FHIR APIs per the API Exchange IG.

**Figure 1: Flow diagram showing the 5 Step exchange between PharmaInc and the EMA, conducted via FHIR Transaction Bundles over RESTful APIs.**

<span style="display: inline-block; vertical-align: middle;">
  <img src="shelflifeextensionflow.svg" alt="5 step process for a shelf life extension" style="width: 800px; height: auto;" />
</span>

### Step 1: PharmaInc Submits Type II Variation

PharmaInc submits a Type II variation to extend the shelf life from 24 to 36 months via a Transaction Bundle to the EMA’s FHIR server (`https://ema.example.org/fhir`). The Bundle includes:
- **Task**: Requests the variation, with PharmaInc as requester and EMA as owner.
- **DocumentReference**: References the application form (Collection Bundle), ePI Document Bundle (updated storage conditions), and PQI Collection Bundle (stability data) as base64-encoded attachments.
- **Provenance**: Records the submission’s origin.

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-001",
      "resource": {
        "resourceType": "Task",
        "id": "task-001",
        "status": "requested",
        "intent": "proposal",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/task-code",
              "code": "submit-variation",
              "display": "Submit Type II Variation"
            }
          ]
        },
        "description": "Type II variation for shelf life extension from 24 to 36 months",
        "for": {
          "reference": "MedicinalProductDefinition/mpd-001"
        },
        "authoredOn": "2025-06-29",
        "requester": {
          "reference": "Organization/pharma-inc"
        },
        "owner": {
          "reference": "Organization/ema"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:doc-ref-001",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "doc-ref-001",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "51851-4",
              "display": "Administrative report"
            }
          ]
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/json",
              "data": "base64 data placeholder",
              "hash": "qZk+NkcGgWq6PiVxeFDCbJzQ2J0=/nE+3Y2c=",
              "size": 1234,
              "title": "Type II Variation Application Form Collection Bundle"
            }
          },
          {
            "attachment": {
              "contentType": "application/json",
              "data": "base64 data placeholder",
              "hash": "qZk+NkcGgWq6PiVxeFDCbJzQ2J0=/nE+3Y2c=",
              "size": 1234,
              "title": "ePI Storage Update Document Bundle"
            }
          },
          {
            "attachment": {
              "contentType": "application/json",
              "data": "base64 data placeholder",
              "hash": "qZk+NkcGgWq6PiVxeFDCbJzQ2J0=/nE+3Y2c=",
              "size": 1234,
              "title": "PQI Stability Data Collection Bundle"
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
      "fullUrl": "urn:uuid:provenance-001",
      "resource": {
        "resourceType": "Provenance",
        "id": "provenance-001",
        "target": [
          {
            "reference": "DocumentReference/doc-ref-001"
          }
        ],
        "recorded": "2025-06-29T17:25:00-04:00",
        "agent": [
          {
            "type": {
              "coding": [
                {
                  "system": "http://hl7.org/fhir/provenance-participant-type",
                  "code": "author"
                }
              ]
            },
            "who": {
              "reference": "Organization/pharma-inc"
            }
          }
        ],
        "signature": [
          {
            "type": [
              {
                "system": "urn:iso-astm:E1762-95:2013",
                "code": "1.2.840.10065.1.12.1.1"
              }
            ],
            "when": "2025-06-29T17:25:00-04:00",
            "who": {
              "reference": "Organization/pharma-inc"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    },
    {
      "fullUrl": "urn:uuid:application-form-001",
      "resource": {
        "resourceType": "Bundle",
        "id": "application-form-001",
        "type": "collection",
        "entry": [
          {
            "fullUrl": "urn:uuid:activity-definition-001",
            "resource": {
              "resourceType": "ActivityDefinition",
              "id": "activity-definition-001",
              "status": "draft",
              "description": "Type II variation application for shelf life extension from 24 to 36 months"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Bundle"
      }
    },
    {
      "fullUrl": "urn:uuid:epi-bundle-001",
      "resource": {
        "resourceType": "Bundle",
        "id": "epi-bundle-001",
        "type": "document",
        "entry": [
          {
            "fullUrl": "urn:uuid:epi-composition-001",
            "resource": {
              "resourceType": "Composition",
              "id": "epi-composition-001",
              "status": "preliminary",
              "type": {
                "coding": [
                  {
                    "system": "http://loinc.org",
                    "code": "34391-3",
                    "display": "Product Labeling"
                  }
                ]
              },
              "title": "Updated ePI for Monoclonal Antibody",
              "section": [
                {
                  "title": "Storage",
                  "text": {
                    "status": "generated",
                    "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\">Store at 2-8°C for up to 36 months.</div>"
                  }
                }
              ]
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Bundle"
      }
    },
    {
      "fullUrl": "urn:uuid:pqi-bundle-001",
      "resource": {
        "resourceType": "Bundle",
        "id": "pqi-bundle-001",
        "type": "collection",
        "entry": [
          {
            "fullUrl": "urn:uuid:observation-001",
            "resource": {
              "resourceType": "Observation",
              "id": "observation-001",
              "status": "final",
              "code": {
                "coding": [
                  {
                    "system": "http://loinc.org",
                    "code": "76662-2",
                    "display": "Stability study"
                  }
                ]
              },
              "valueString": "Stability data supports shelf life extension to 36 months"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Bundle"
      }
    }
  ]
}
```

### Step 2: EMA Receives and Processes the Submission

The EMA’s FHIR server receives the Transaction Bundle at `https://ema.example.org/fhir`, validates it per the API Exchange IG, and processes the resources:
- **Task**: Logged as a variation request for `MedicinalProductDefinition/mpd-001`.
- **DocumentReference**: Base64-encoded attachments are decoded to access the application form (Collection Bundle), ePI Document Bundle, and PQI Collection Bundle.
- **Provenance**: Verified for integrity and origin from PharmaInc.

The EMA prepares to send a List of Questions (LoQ). No new FHIR resources are created in this step.

### Step 3: EMA Sends List of Questions

The EMA compiles a LoQ with 10 questions (2 administrative, 3 labeling, 5 CMC) in a `Questionnaire` resource within a Collection Bundle. The Transaction Bundle includes a `Task` (requesting responses), the `Questionnaire`, and a `Provenance`, sent to PharmaInc’s FHIR server (`https://pharma-inc.example.org/fhir`).

**Questions**:
- **Administrative**:
  1. Is the fee for the Type II variation fully paid and documented?
  2. Are all required metadata fields in the application form complete?
- **Labeling**:
  1. **Storage Instructions**: Do the updated storage conditions (2-8°C for 36 months) require additional patient guidance?
  2. **Patient Information**: Is the patient leaflet revised to clarify the extended shelf life?
  3. **Label Clarity**: Is the updated ePI text sufficiently clear for all EU languages?
- **CMC**:
  1. **Stability Data**: Do stability studies cover all required environmental conditions for 36 months?
  2. **Analytical Methods**: Are analytical methods validated for detecting degradation over 36 months?
  3. **Impurities**: Are new impurities identified in extended stability studies?
  4. **Container Closure System**: Is the container closure system validated for 36 months?
  5. **Degradation Products**: Are degradation products fully characterized for 36 months?

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-002",
      "resource": {
        "resourceType": "Task",
        "id": "task-002",
        "status": "requested",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/task-code",
              "code": "respond-to-questions",
              "display": "Respond to List of Questions"
            }
          ]
        },
        "description": "Provide responses to the List of Questions for Type II variation (shelf life extension)",
        "for": {
          "reference": "MedicinalProductDefinition/mpd-001"
        },
        "authoredOn": "2025-06-29T17:56:00-04:00",
        "requester": {
          "reference": "Organization/ema"
        },
        "owner": {
          "reference": "Organization/pharma-inc"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:loq-bundle-001",
      "resource": {
        "resourceType": "Bundle",
        "id": "loq-bundle-001",
        "type": "collection",
        "entry": [
          {
            "fullUrl": "urn:uuid:questionnaire-001",
            "resource": {
              "resourceType": "Questionnaire",
              "id": "questionnaire-001",
              "status": "active",
              "title": "List of Questions for Type II Variation (Shelf Life Extension)",
              "item": [
                {
                  "linkId": "admin-1",
                  "text": "Is the fee for the Type II variation fully paid and documented?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "administrative"
                      }
                    }
                  ]
                },
                {
                  "linkId": "admin-2",
                  "text": "Are all required metadata fields in the application form complete?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "administrative"
                      }
                    }
                  ]
                },
                {
                  "linkId": "labeling-1",
                  "text": "Do the updated storage conditions (2-8°C for 36 months) require additional patient guidance?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "labeling-storage"
                      }
                    }
                  ]
                },
                {
                  "linkId": "labeling-2",
                  "text": "Is the patient leaflet revised to clarify the extended shelf life?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "labeling-patient-info"
                      }
                    }
                  ]
                },
                {
                  "linkId": "labeling-3",
                  "text": "Is the updated ePI text sufficiently clear for all EU languages?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "labeling-clarity"
                      }
                    }
                  ]
                },
                {
                  "linkId": "cmc-1",
                  "text": "Do stability studies cover all required environmental conditions for 36 months?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "cmc-stability"
                      }
                    }
                  ]
                },
                {
                  "linkId": "cmc-2",
                  "text": "Are analytical methods validated for detecting degradation over 36 months?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "cmc-analytical"
                      }
                    }
                  ]
                },
                {
                  "linkId": "cmc-3",
                  "text": "Are new impurities identified in extended stability studies?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "cmc-impurities"
                      }
                    }
                  ]
                },
                {
                  "linkId": "cmc-4",
                  "text": "Is the container closure system validated for 36 months?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "cmc-container"
                      }
                    }
                  ]
                },
                {
                  "linkId": "cmc-5",
                  "text": "Are degradation products fully characterized for 36 months?",
                  "type": "string",
                  "extension": [
                    {
                      "url": "http://example.org/fhir/StructureDefinition/question-category",
                      "valueCoding": {
                        "system": "http://example.org/codes",
                        "code": "cmc-degradation"
                      }
                    }
                  ]
                }
              ]
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Bundle"
      }
    },
    {
      "fullUrl": "urn:uuid:provenance-002",
      "resource": {
        "resourceType": "Provenance",
        "id": "provenance-002",
        "target": [
          {
            "reference": "Bundle/loq-bundle-001"
          }
        ],
        "recorded": "2025-06-29T17:56:00-04:00",
        "agent": [
          {
            "type": {
              "coding": [
                {
                  "system": "http://hl7.org/fhir/provenance-participant-type",
                  "code": "author"
                }
              ]
            },
            "who": {
              "reference": "Organization/ema"
            }
          }
        ],
        "signature": [
          {
            "type": [
              {
                "system": "urn:iso-astm:E1762-95:2013",
                "code": "1.2.840.10065.1.12.1.1"
              }
            ],
            "when": "2025-06-29T17:56:00-04:00",
            "who": {
              "reference": "Organization/ema"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    }
  ]
}
```

### Step 4: PharmaInc Responds to List of Questions

PharmaInc’s FHIR server processes the LoQ, prepares a `QuestionnaireResponse` with answers to all 10 questions, and sends a Transaction Bundle to the EMA’s FHIR server (`https://ema.example.org/fhir`). For `cmc-4` (container closure system), the response includes a base64-encoded SVG image. For `cmc-5` (degradation products), the response includes two paragraphs. The Bundle includes a `Task` (indicating completion), the `QuestionnaireResponse`, and a `Provenance`.

**Responses**:
- **Administrative**:
  1. Fee paid on June 25, 2025, with documentation.
  2. All metadata fields are complete per EMA guidelines.
- **Labeling**:
  1. No additional patient guidance needed for storage conditions.
  2. Patient leaflet updated for 36-month shelf life.
  3. ePI text translated and clear in all EU languages.
- **CMC**:
  1. Stability studies cover all environmental conditions per ICH guidelines.
  2. Analytical methods validated, with results in PQI.
  3. No new impurities identified.
  4. Container closure system validated, with SVG diagram.
  5. Degradation products characterized, with no significant changes (two paragraphs).

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-003",
      "resource": {
        "resourceType": "Task",
        "id": "task-003",
        "status": "completed",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/task-code",
              "code": "submit-response",
              "display": "Submit Response to List of Questions"
            }
          ]
        },
        "description": "Submission of responses to the List of Questions for Type II variation (shelf life extension)",
        "for": {
          "reference": "MedicinalProductDefinition/mpd-001"
        },
        "authoredOn": "2025-06-29T18:15:00-04:00",
        "requester": {
          "reference": "Organization/pharma-inc"
        },
        "owner": {
          "reference": "Organization/ema"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:response-bundle-001",
      "resource": {
        "resourceType": "Bundle",
        "id": "response-bundle-001",
        "type": "collection",
        "entry": [
          {
            "fullUrl": "urn:uuid:questionnaire-response-001",
            "resource": {
              "resourceType": "QuestionnaireResponse",
              "id": "questionnaire-response-001",
              "questionnaire": "Questionnaire/questionnaire-001",
              "status": "completed",
              "authored": "2025-06-29T18:15:00-04:00",
              "item": [
                {
                  "linkId": "admin-1",
                  "text": "Is the fee for the Type II variation fully paid and documented?",
                  "answer": [
                    {
                      "valueString": "Yes, the fee was paid on June 25, 2025, with documentation included in the application form."
                    }
                  ]
                },
                {
                  "linkId": "admin-2",
                  "text": "Are all required metadata fields in the application form complete?",
                  "answer": [
                    {
                      "valueString": "All metadata fields are complete, as verified against EMA guidelines."
                    }
                  ]
                },
                {
                  "linkId": "labeling-1",
                  "text": "Do the updated storage conditions (2-8°C for 36 months) require additional patient guidance?",
                  "answer": [
                    {
                      "valueString": "No additional guidance is needed; the ePI clearly states storage at 2-8°C for 36 months."
                    }
                  ]
                },
                {
                  "linkId": "labeling-2",
                  "text": "Is the patient leaflet revised to clarify the extended shelf life?",
                  "answer": [
                    {
                      "valueString": "Yes, the patient leaflet has been updated to reflect the 36-month shelf life."
                    }
                  ]
                },
                {
                  "linkId": "labeling-3",
                  "text": "Is the updated ePI text sufficiently clear for all EU languages?",
                  "answer": [
                    {
                      "valueString": "The ePI text has been translated and reviewed for clarity in all EU languages."
                    }
                  ]
                },
                {
                  "linkId": "cmc-1",
                  "text": "Do stability studies cover all required environmental conditions for 36 months?",
                  "answer": [
                    {
                      "valueString": "Yes, stability studies include temperature, humidity, and light conditions for 36 months, per ICH guidelines."
                    }
                  ]
                },
                {
                  "linkId": "cmc-2",
                  "text": "Are analytical methods validated for detecting degradation over 36 months?",
                  "answer": [
                    {
                      "valueString": "Analytical methods are fully validated, with results included in the PQI submission."
                    }
                  ]
                },
                {
                  "linkId": "cmc-3",
                  "text": "Are new impurities identified in extended stability studies?",
                  "answer": [
                    {
                      "valueString": "No new impurities were identified; all impurities are within acceptable limits."
                    }
                  ]
                },
                {
                  "linkId": "cmc-4",
                  "text": "Is the container closure system validated for 36 months?",
                  "answer": [
                    {
                      "valueString": "The container closure system is validated for 36 months, with supporting data in the PQI Bundle."
                    },
                    {
                      "valueAttachment": {
                        "contentType": "image/svg+xml",
                        "data": "PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIj48cmVjdCB3aWR0aD0iMTAwIiBoZWlnaHQ9IjEwMCIgZmlsbD0iI2ZmZiIvPjx0ZXh0IHg9IjUwIiB5PSI1MCIgZm9udC1zaXplPSIxNSIgZmlsbD0iIzAwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSI+U1ZHIFBsYWNlaG9sZGVyPC90ZXh0Pjwvc3ZnPg==",
                        "title": "Container Closure System Diagram"
                      }
                    }
                  ]
                },
                {
                  "linkId": "cmc-5",
                  "text": "Are degradation products fully characterized for 36 months?",
                  "answer": [
                    {
                      "valueString": "Degradation products are fully characterized. No significant changes were observed in the 36-month stability studies.\nAll identified degradation products remain within acceptable limits, as detailed in the PQI submission."
                    }
                  ]
                }
              ]
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Bundle"
      }
    },
    {
      "fullUrl": "urn:uuid:provenance-003",
      "resource": {
        "resourceType": "Provenance",
        "id": "provenance-003",
        "target": [
          {
            "reference": "Bundle/response-bundle-001"
          }
        ],
        "recorded": "2025-06-29T18:15:00-04:00",
        "agent": [
          {
            "type": {
              "coding": [
                {
                  "system": "http://hl7.org/fhir/provenance-participant-type",
                  "code": "author"
                }
              ]
            },
            "who": {
              "reference": "Organization/pharma-inc"
            }
          }
        ],
        "signature": [
          {
            "type": [
              {
                "system": "urn:iso-astm:E1762-95:2013",
                "code": "1.2.840.10065.1.12.1.1"
              }
            ],
            "when": "2025-06-29T18:15:00-04:00",
            "who": {
              "reference": "Organization/pharma-inc"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    }
  ]
}
```

### Step 5: EMA Issues Decision Letter

The EMA reviews PharmaInc’s LoQ responses and approves the variation. The Decision Letter is a Document Bundle with a `Composition`, encoded as base64 data in a `DocumentReference`. The Transaction Bundle includes a `Task`, the `DocumentReference`, and a `Provenance`, sent to PharmaInc’s FHIR server (`https://pharma-inc.example.org/fhir`).

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "fullUrl": "urn:uuid:task-004",
      "resource": {
        "resourceType": "Task",
        "id": "task-004",
        "status": "completed",
        "intent": "order",
        "code": {
          "coding": [
            {
              "system": "http://hl7.org/fhir/task-code",
              "code": "issue-decision",
              "display": "Issue Decision Letter"
            }
          ]
        },
        "description": "Issuance of Decision Letter approving Type II variation for shelf life extension",
        "for": {
          "reference": "MedicinalProductDefinition/mpd-001"
        },
        "authoredOn": "2025-06-29T18:30:00-04:00",
        "requester": {
          "reference": "Organization/ema"
        },
        "owner": {
          "reference": "Organization/pharma-inc"
        }
      },
      "request": {
        "method": "POST",
        "url": "Task"
      }
    },
    {
      "fullUrl": "urn:uuid:doc-ref-002",
      "resource": {
        "resourceType": "DocumentReference",
        "id": "doc-ref-002",
        "status": "current",
        "type": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "51851-4",
              "display": "Administrative report"
            }
          ]
        },
        "content": [
          {
            "attachment": {
              "contentType": "application/fhir+json",
              "data": "eyJyZXNvdXJjZVR5cGUiOiJCdW5kbGUiLCJpZCI6ImRlY2lzaW9uLWxldHRlci1kb2MtMDAxIiwidHlwZSI6ImRvY3VtZW50IiwiZW50cnkiOlt7ImZ1bGxVcmwiOiJ1cm46dXVpZDpkZWNpc2lvbi1sZXR0ZXItY29tcC0wMDEiLCJyZXNvdXJjZSI6eyJyZXNvdXJjZVR5cGUiOiJDb21wb3NpdGlvbiIsImlkIjoiZGVjaXNpb24tbGV0dGVyLWNvbXAtMDAxIiwic3RhdHVzIjoiZmluYWwiLCJ0eXBlIjp7ImNvZGluZyI6W3sic3lzdGVtIjoiaHR0cDovL2xvaW5jLm9yZyIsImNvZGUiOiI1MTg1MS00IiwiZGlzcGxheSI6IkFkbWluaXN0cmF0aXZlIHJlcG9ydCJ9XX0sInRpdGxlIjoiVHlwZSBJSSBWYXJpYXRpb24gQXBwcm92YWwgRGVjaXNpb24gTGV0dGVyIiwiZGF0ZSI6IjIwMjUtMDYtMjlUMTg6MzA6MDAtMDQ6MDAiLCJhdXRob3IiOlt7InJlZmVyZW5jZSI6Ik9yZ2FuaXphdGlvbi9lbWEifV0sInNlY3Rpb24iOlt7InRpdGxlIjoiQXBwcm92YWwiLCJ0ZXh0Ijp7InN0YXR1cyI6ImdlbmVyYXRlZCIsImRpdiI6IjxkaXYgeG1sbnM9XCJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hodG1sXCI+VGhlIEV1cm9wZWFuIE1lZGljaW5lcyBBZ2VuY3kgKEVNQSkgYXBwcm92ZXMgdGhlIFR5cGUgSUkgdmFyaWF0aW9uIGZvciB0aGUgbW9ub2Nsb25hbCBhbnRpYm9keSwgZXh0ZW5kaW5nIHRoZSBzaGVsZiBsaWZlIGZyb20gMjQgdG8gMzYgbW9udGhzLCBlZmZlY3RpdmUgSnVuZSAyOSwgMjAyNS4gVGhlIHVwZGF0ZWQgZVBJIHN0b3JhZ2UgY29uZGl0aW9ucyAoMi04wrBDIGZvciAzNiBtb250aHMpIGFuZCBQUUkgc3RhYmlsaXR5IGRhdGEgYXJlIGFjY2VwdGVkIGJhc2VkIG9uIHRoZSBwcm92aWRlZCBkb2N1bWVudGF0aW9uIGFuZCByZXNwb25zZXMgdG8gdGhlIExpc3Qgb2YgUXVlc3Rpb25zLjwvZGl2PiJ9fV19fX1dfQ==",
              "hash": "qZk+NkcGgWq6PiVxeFDCbJzQ2J0=/nE+3Y2c=",
              "size": 1234,
              "title": "Type II Variation Approval Decision Letter Document Bundle"
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
      "fullUrl": "urn:uuid:provenance-004",
      "resource": {
        "resourceType": "Provenance",
        "id": "provenance-004",
        "target": [
          {
            "reference": "DocumentReference/doc-ref-002"
          }
        ],
        "recorded": "2025-06-29T18:30:00-04:00",
        "agent": [
          {
            "type": {
              "coding": [
                {
                  "system": "http://hl7.org/fhir/provenance-participant-type",
                  "code": "author"
                }
              ]
            },
            "who": {
              "reference": "Organization/ema"
            }
          }
        ],
        "signature": [
          {
            "type": [
              {
                "system": "urn:iso-astm:E1762-95:2013",
                "code": "1.2.840.10065.1.12.1.1"
              }
            ],
            "when": "2025-06-29T18:30:00-04:00",
            "who": {
              "reference": "Organization/ema"
            }
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Provenance"
      }
    }
  ]
}
```
