### Introduction
This Implementation Guide (IG) defines how to exchange regulatory content using FHIR R5 compliant APIs, workflows, and infrastructure. This project is part of a program of work related to the following other FHIR projects:
- [Electronic Medicinal Product Information (ePI)](https://build.fhir.org/ig/HL7/emedicinal-product-info/index.html)
- [Pharmaceutical Quality (Industry) IG](https://build.fhir.org/ig/HL7/uv-dx-pq/index.html)
- [UMC IDMP Request and Publish API](https://build.fhir.org/ig/Uppsala-Monitoring-Centre/WHO-UMC-IDMP-Service/index.html)

This IG leverages `Bundle` (type `transaction`) for atomic submissions and `Task` for workflow coordination, suitable for company to company interactions, company to regulator interactions, and regulator to regulator interactions. 

### Goals
- **Efficiency**: Streamline submission processes, reducing manual effort, reducing administrative delays compared to traditional file-based systems.
- **Interoperability**: Ensure content is machine-readable and compatible across systems, supporting global regulatory harmonization.
- **Modernization**: Address the limitations of antiquated file-based systems (e.g., paper/electronic paper, one-way workflows and systems).
- **Innovation**: Create an innovation and AI friendly framework using web standards and a strong focus on being fast and easy to implement.


### Scope and Objective
- **Scope**: This IG's focus is currently limited to regulated medicinal products and regulatory transactions related to ePI (a.k.a drug labeling) and PQI (a.k.a CMC) in FHIR compliant `.xml` or `.json`. Future scope expansions will include other regulated product types (e.g., veterinary, medical device, over the counter, natural health) and other regulatory content domains like adverse events (AEs) or Clinical Trial Applications/ Investigational New Drug Applications. 
- **Objective**: Describe a framework for interoperable exchange of regulatory content, including ePI (as `Bundle` type `document` in `.xml` or `.json`) and PQI data (e.g., quality test results, batch analysis), with application forms and legacy documents as PDFs via `DocumentReference`.
- **Use Case**: A pharmaceutical company submits an ePI update and PQI update (e.g., shelf life extension) to a regulator with a `Task` to track review and approval.
- **Stakeholders**: Regulators, pharmaceutical companies, FHIR server vendors, structured/component authoring vendors.
- **Resources**: `DocumentReference` (for PDFs or referencing `Bundle` type `document`), `Observation` (for PQI data), `Task` (for workflow), `Bundle` (type `transaction` and `document`).
- **Compliance**: Aligns with ISO IDMP, HL7 Vulcan ePI, and HL7 PQI standards.
- **Branding**: Branded as **RECON** to emphasize regulatory content exchange and coordinated workflow system (orchestration) that manages the lifecycle of regulatory submissions using Task for clear, trackable processes.

### Pre-requisites
- **Tools**: Text editor (e.g., VS Code), Git, GitHub for hosting, HAPI FHIR Validator CLI (optional).
- **FHIR Knowledge**: Familiarity with `Bundle` (type `transaction` and `document`), `Task`, `DocumentReference`, and `Observation` in FHIR R5.
- **Infrastructure**: Access to a FHIR R5 server (e.g., HAPI FHIR, Kodjin FHIR Server).
- **Security**: Implement SMART on FHIR and OAuth2 for all API interactions.

### Background

#### Problem Statement
The current landscape of regulatory submissions relies on outdated paradigms that blend paper-based and electronic analogs of paper processes. These approaches are often inconsistent, complex, costly, and labor-intensive, posing challenges for regulators, pharmaceutical companies, and other stakeholders. Key issues include:

- **Fragmented Structures**: Many submission frameworks depend on intricate file-based systems, such as folder hierarchies with PDFs, tables of contents, and hypertext-linked documents. These structures, while functional, demand significant effort to create, validate, and navigate.
- **Diverse Exchange Methods**: There is no unified platform for exchanging regulatory content. Instead, submissions are transmitted through a patchwork of technologies, including portals, AS2, AS3, FTP, secure email, regular email, and even physical media or paper. This lack of standardization complicates workflows and increases the risk of errors.
- **One-Way Workflows**: Most regulatory translation workflows are unidirectional, limiting real-time interaction and feedback. With rare exceptions, such as EMA’s XEVMPD data submissions, there is little support for iterative or dynamic exchanges between submitters and regulators.
- **Unstructured and Legacy Formats**: Regulatory content is often submitted in formats like DOCX, PDF, various XML schemas, CSV, or SAS datasets. These formats, while widely used, lack the structure and interoperability needed for efficient processing, analysis, or reuse in modern systems.

These challenges hinder the ability to achieve streamlined, cost-effective, and globally harmonized regulatory processes, creating a need for a more modern and accessible approach.

#### Solution
The **Regulatory Content Exchange and Orchestration Network (RECON)** IG proposes a forward-looking, web-based standard to transform how regulatory content is exchanged. Built on FHIR R5, RECON leverages interoperable APIs, structured data models, and workflow orchestration to support the seamless exchange of regulatory content, such as ePI and PQI data. Key features include:

- **Structured and Interoperable Content**: RECON uses FHIR resources like `Bundle` (type `document`) for ePI, `Observation` for PQI test results, and `DocumentReference` for PDFs, enabling standardized, machine-readable submissions.
- **Unified Exchange Platform**: By adopting FHIR-compliant APIs, RECON facilitates consistent, secure, and real-time content exchange, reducing reliance on disparate technologies.
- **Dynamic Workflows**: The `Task` resource supports orchestrated workflows, allowing regulators and submitters to track, review, and update submissions interactively.
- **Modern Accessibility**: As a web-based standard, RECON aligns with global initiatives like HL7 Vulcan (ePI), HL7 PQI, and ISO IDMP, ensuring compatibility with emerging regulatory requirements and fostering global harmonization.

RECON offers a modern framework that complements regulator's existing submission processes, enhancing efficiency and interoperability while addressing the complexities of regulatory content exchange.

