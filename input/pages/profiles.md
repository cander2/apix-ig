This section defines the FHIR profiles for the APIX Implementation Guide (IG), which supports real-time, API-based exchange of biopharmaceutical information for regulatory submissions, extending the ePI and PQI projects. These profiles constrain FHIR R5 resources to ensure standardized data for regulatory workflows, such as variation submissions.

## Overview

The profiles support transaction-based submissions of regulatory data, including documents, tasks, and provenance records.

## Profile: ApixTransactionBundleProfile

- **Base Resource**: Bundle
- **Description**: Represents a transaction bundle for regulatory submissions (e.g., variation applications) via API.
- **Constraints**:
  - `type`: Mandatory, fixed to `transaction`.
  - `entry`: Must include at least one `DocumentReference`, `Task`, or `Provenance`.
  - `entry.request.method`: Mandatory, restricted to `POST` or `PUT`.
  - `identifier`: Mandatory, must include a regulatory submission identifier.
- **Extensions**:
  - `SubmissionType`: Type of submission (e.g., variation, initial).
- **Example**: [ApixTransactionBundleExample](examples/ExampleApixTransactionBundle.json)
- **StructureDefinition**: [ApixTransactionBundleProfile](StructureDefinition-apix-transaction-bundle-profile.json)

## Profile: ApixTaskProfile

- **Base Resource**: Task
- **Description**: Manages regulatory submission workflows, such as variation requests.
- **Constraints**:
  - `status`: Mandatory, restricted to `requested`, `in-progress`, `completed`, or `cancelled`.
  - `intent`: Mandatory, fixed to `order`.
  - `focus`: Mandatory, must reference a `DocumentReference`.
  - `code`: Mandatory, bound to `ApixTaskCodeValueSet`.
- **Extensions**:
  - `RegulatoryPriority`: Priority of the submission (e.g., urgent, standard).
- **Example**: [ApixTaskExample](examples/ExampleApixTask.json)
- **StructureDefinition**: [ApixTaskProfile](StructureDefinition-apix-task-profile.json)

## Profile: ApixDocumentReferenceProfile

- **Base Resource**: DocumentReference
- **Description**: Represents regulatory documents (e.g., variation reports, SmPC) in API submissions.
- **Constraints**:
  - `type`: Mandatory, bound to LOINC or `ApixDocumentTypeValueSet`.
  - `status`: Mandatory, restricted to `current` or `superseded`.
  - `content.attachment.contentType`: Mandatory, restricted to `application/pdf` or `application/json`.
  - `content.attachment.data`: Mandatory, contains base64-encoded document content.
  - `content.attachment.url`: Not allowed.
  - `content.attachment.size`: Optional, indicates size of the decoded data.
  - `content.attachment.hash`: Optional, provides a hash of the data.
  - `content.attachment.title`: Mandatory, document title.
- **Extensions**: None
- **Example**: [ApixDocumentReferenceExample](examples/ExampleApixDocumentReference.json)
- **StructureDefinition**: [ApixDocumentReferenceProfile](StructureDefinition-apix-document-reference-profile.json)

## Profile: ApixProvenanceProfile

- **Base Resource**: Provenance
- **Description**: Tracks the origin and history of regulatory submission data.
- **Constraints**:
  - `target`: Mandatory, must reference a `DocumentReference` or `Task`.
  - `recorded`: Mandatory, captures submission timestamp.
  - `agent`: Mandatory, at least one agent for the submitting entity.
  - `activity`: Mandatory, bound to `ApixProvenanceActivityValueSet`.
- **Extensions**:
  - `RegulatoryContext`: Context of the submission (e.g., EMA, FDA).
- **Example**: [ApixProvenanceExample](examples/ExampleApixProvenance.json)
- **StructureDefinition**: [ApixProvenanceProfile](StructureDefinition-apix-provenance-profile.json)
