This page defines the FHIR profiles for the APIX Implementation Guide (IG), which supports real-time, API-based exchange of biopharmaceutical information for regulatory submissions, extending the ePI and PQI projects. These profiles constrain FHIR R5 resources to ensure standardized data for regulatory workflows, such as variation submissions.

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
  - `content.attachment.contentType`: Mandatory, restricted to `application/pdf` or `text/html`.
  - `content.attachment.url`: Mandatory.
- **Extensions**:
  - `SubmissionDate`: Date of document submission.
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

## Value Sets
- **ApixSubmissionTypeValueSet**: Codes for submission types (e.g., variation, initial, renewal).
- **ApixTaskCodeValueSet**: Codes for task types (e.g., submit-variation, review).
- **ApixDocumentTypeValueSet**: Codes for document types (e.g., SmPC, variation-report, PIL).
- **ApixProvenanceActivityValueSet**: Codes for provenance activities (e.g., submission, update).
- **ApixRegulatoryPriority**: Codes for submission priorities (e.g., urgent, standard).
- **ApixRegulatoryContext**: Codes for regulatory contexts (e.g., EMA, FDA).

## Conformance
Systems implementing the APIX IG MUST support these profiles and validate instances against their constraints. Profiles are designed for RESTful API interactions, supporting transaction-based submissions via POST or PUT operations.