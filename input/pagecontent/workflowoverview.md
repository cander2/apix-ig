# Workflow Overview - Regulatory Content Exchange and Orchestration Network (RECON) v0.1.0

**Regulatory Content Exchange and Orchestration Network (RECON)**  
**Version**: 0.1.0 - ci-build  
**Publisher**: Gravitate Health Project  
**Status**: This guide is not an authorized publication; it is a continuous build for version 0.1.0, based on the content available at [https://github.com/cander2/recon-ig/](https://github.com/cander2/recon-ig/). See the [Directory of published versions](https://build.fhir.org/ig/cander2/recon-ig/history.html) for more details.

## Table of Contents
- [Overview](#overview)
- [Workflow Steps](#workflow-steps)
- [Identifying Sender and Recipient](#identifying-sender-and-recipient)
- [Security and Privacy](#security-and-privacy)
- [Error Handling](#error-handling)
- [Conformance and Extensibility](#conformance-and-extensibility)
- [Diagram](#diagram)
- [Examples](#examples)
- [Next Steps for Implementers](#next-steps-for-implementers)
- [Feedback](#feedback)

## Overview

The RECON workflow facilitates the exchange and orchestration of regulatory content, such as electronic Product Information (ePI) and Product Quality Information (PQI), between submitters (e.g., pharmaceutical companies) and regulators. The process leverages FHIR resources, primarily `Task`, `DocumentReference`, `Observation`, and `Provenance`, to manage submissions, processing, updates, and notifications. This guide outlines the workflow, sender/recipient identification, security considerations, and implementation details.

## Workflow Steps

1. **Submission**  
   - **Action**: The submitter creates a FHIR `Transaction Bundle` containing:  
     - A `Task` resource with `status` set to `requested`, specifying the regulatory action (e.g., submission of ePI or PQI).  
     - A `DocumentReference` resource (for ePI or PDF content) or an `Observation` resource (for PQI data).  
     - A `Provenance` resource to document the submission’s origin and sender.  
     - Optional: Additional resources like `Composition` for structured ePI or `Organization` for sender/recipient details.  
   - **Endpoint**: The bundle is submitted via `POST` to `[base]`.  
   - **Validation**: The server validates the bundle for FHIR conformance and RECON-specific constraints (e.g., required profiles).  
   - **Response**: The server returns a `Bundle` with the created resources’ IDs and a `201 Created` status or an `OperationOutcome` for errors.

2. **Processing**  
   - **Action**: The regulator retrieves the submitted `Task` and associated resources using a search query:  
     ```http
     GET [base]/Task?status=requested&_include=Task:focus
     ```
   - **Details**:  
     - The `_include=Task:focus` parameter retrieves the referenced `DocumentReference` or `Observation`.  
     - The regulator reviews the content (e.g., ePI PDF or PQI data) and performs necessary actions (e.g., validation, approval, or rejection).  
   - **Access Control**: Only the regulator specified in `Task.owner` can access the task, enforced via OAuth 2.0 or SMART-on-FHIR.

3. **Updates**  
   - **Action**: The regulator updates the `Task` resource to reflect the processing outcome (e.g., `accepted`, `rejected`, or `completed`) and adds outputs to `Task.output`.  
   - **Method**: A `Transaction Bundle` is submitted via `POST` to `[base]` with:  
     - An updated `Task` resource.  
     - Optional: Additional resources like `OperationOutcome` for comments or `DocumentReference` for revised ePI.  
   - **Validation**: The server ensures the `Task` update adheres to RECON profiles and state transitions.

4. **Notification**  
   - **Action**: The submitter is notified of `Task` updates through:  
     - **Polling**: Periodic `GET [base]/Task?status=completed&_id=task-123` requests.  
     - **Subscription**: A FHIR `Subscription` resource for real-time notifications (if supported by the server).  
   - **Details**:  
     - The submitter retrieves the updated `Task` and its `output` (e.g., `OperationOutcome` or revised `DocumentReference`).  
     - Subscriptions may use REST-hook, WebSocket, or email channels, as defined in the server’s `CapabilityStatement`.

## Identifying Sender and Recipient

To ensure proper routing and auditability, RECON transaction bundles must clearly identify the sender (submitter) and recipient (regulator) of the submission. This is achieved through FHIR resources and security mechanisms:

- **Sender (Submitter)**:
  - **Task.requester**: References the `Organization` or `Practitioner` submitting the request (e.g., a pharmaceutical company).  
  - **Provenance**: A `Provenance` resource documents the submission’s origin, with `Provenance.agent.who` (type `author`) linking to the sender.  
  - **Security**: The sender is authenticated via OAuth 2.0 or SMART-on-FHIR, with the access token’s claims (e.g., `client_id`) mapping to the `Task.requester`.  

- **Recipient (Regulator)**:
  - **Task.owner**: References the `Organization` or `PractitionerRole` responsible for processing the task (e.g., a regulatory authority).  
  - **Task.restriction.recipient** (optional): Specifies authorized recipients for the task.  
  - **Provenance**: Optionally includes an `agent` with type `performer` for the recipient.  
  - **Server Routing**: The FHIR server routes the submission to the regulator based on `Task.owner` or server-specific logic.

- **Best Practices**:
  - Include a `Provenance` resource in the `Transaction Bundle` to document both sender and recipient for auditability.  
  - Use globally unique identifiers (e.g., regulatory IDs, UUIDs) in `Organization.identifier` to avoid ambiguity.  
  - Ensure the FHIR server enforces access controls to restrict task retrieval to the intended recipient.

- **Security Considerations**:
  - All submissions must use HTTPS and OAuth 2.0 or SMART-on-FHIR authentication.  
  - The server validates that the authenticated sender matches the `Task.requester` and that only the `Task.owner` can process the task.

For detailed resource constraints, see the [RECON Task Profile](https://build.fhir.org/ig/cander2/recon-ig/StructureDefinition-recon-task.html), [RECON Provenance Profile](https://build.fhir.org/ig/cander2/recon-ig/StructureDefinition-recon-provenance.html), and [RECON Organization Profile](https://build.fhir.org/ig/cander2/recon-ig/StructureDefinition-recon-organization.html).

## Security and Privacy

- All interactions must use HTTPS.  
- Authentication and authorization follow SMART-on-FHIR or OAuth 2.0 standards.  
- Sensitive data in `DocumentReference` or `Observation` resources should be encrypted or access-controlled.  
- Servers must log sender and recipient details (via `Provenance` or audit events) for regulatory compliance.

## Error Handling

- Servers must return `OperationOutcome` resources for validation errors, unauthorized access, or processing failures.  
- Common errors include:  
  - Invalid FHIR resources or missing required fields (`400 Bad Request`).  
  - Mismatch between authenticated sender and `Task.requester` (`403 Forbidden`).  
  - Inaccessible tasks or resources (`404 Not Found` or `403 Forbidden`).  

## Conformance and Extensibility

- Implementers must adhere to RECON-specific FHIR profiles (see [StructureDefinition](https://build.fhir.org/ig/cander2/recon-ig/profiles.html)).  
- Servers must publish a `CapabilityStatement` detailing supported operations, search parameters, and subscription channels.  
- The workflow supports additional resources (e.g., `Composition` for structured ePI) and custom `Task.output` types.  
- Future versions may include support for `MessageHeader` for complex message exchanges.

## Diagram

The following UML sequence diagram illustrates the RECON workflow:

```mermaid
sequenceDiagram
    participant S as Submitter
    participant R as Regulator
    participant F as FHIR Server

    S->>F: POST Transaction Bundle (Task, DocumentReference/Observation, Provenance)
    F-->>S: 201 Created or OperationOutcome
    R->>F: GET Task?status=requested&_include=Task:focus
    F-->>R: Task + Referenced Resources
    R->>F: POST Transaction Bundle (Updated Task, Task.output)
    F-->>R: 200 OK or OperationOutcome
    S->>F: GET Task?status=completed&_id=task-123 (Polling)
    alt Subscription Enabled
        F-->>S: Notify (Task Update via Subscription)
    end
    F-->>S: Updated Task + Output