### Introduction

This IG enables the vision of reducing pharmaceutical regulatory transaction and processing timelines from days or hours to subsecond timelines, using digitization, automation, and streaming technologies.

Inspired by the Uppsala Monitoring Centre's (UMC) [IDMP Request and Publish API IG](https://build.fhir.org/ig/Uppsala-Monitoring-Centre/WHO-UMC-IDMP-Service/index.html) and the finance industry's real-time algorithmic systems ([Anderson, Algorri, Abernathy 2023](https://pubmed.ncbi.nlm.nih.gov/37619807/)), R2CS leverages HL7 FHIR, Apache Kafka, and APIs to deliver automated, interoperable, and scalable data exchange for electronic Product Information (ePI), Pharmaceutical Quality Information (PQI), pharmacovigilance, and clinical datasets.

R2CS addresses the need for modern, real-time regulatory processes by:
- **Streaming Data**: Using Kafka to stream FHIR Transaction Bundles and Dataset-JSON as NDJSON, achieving subsecond validation and processing.
- **Automation**: Enabling algorithmic workflows for validation, routing, and analytics, akin to trading platforms.
- **Interoperability**: Supporting global standards (HL7 FHIR, CDISC, ICH) for ePI, PQI, and clinical submissions.
- **Scalability**: Handling high-volume submissions (e.g., ~100GB/day compressed JSON) with fault-tolerant architecture.

### Scope

R2CS targets pharmaceutical regulatory affairs, enabling:
- **Electronic Product Information (ePI)**: FHIR-based drug labeling for submission and review.
- **Pharmaceutical Quality Information (PQI)**: Chemistry, Manufacturing, and Controls (CMC) data exchange.
- **Clinical Datasets**: CDISC Dataset-JSON for trial data, streamed in real-time.
- **Global Applicability**: Supports regulators, pharma companies, and healthcare institutions with standardized, automated workflows.

This IG provides technical guidance, profiles, and examples for implementing real-time regulatory exchange, with an intitial focus on medicinal products, pharmaceutical quality and drug labeling. Future iterations will expand the scope to include other regulated product types (e.g., medical devices, veterinary, over the counter, and natural health products), and other data domains (e.g., clinical and adverse event reporting).

### Streaming Solution

R2CS’s core innovation is its real-time streaming architecture, which processes FHIR Transaction Bundles and Dataset-JSON with subsecond latency. Built on Apache Kafka, the solution:
- **Publishes Submissions**: Producers (e.g., pharma portals, regulatory gateways) send NDJSON-encoded FHIR Bundles to Kafka topics (e.g., `r2cs-submissions`).
- **Processes Data**: Parallel consumers validate against JSON Schema, store in databases (e.g., Elasticsearch, MongoDB), and publish status updates.
- **Enables Querying**: Real-time APIs deliver validated data to stakeholders.
- **Ensures Compliance**: Provenance resources and Kafka’s exactly-once semantics ensure traceability and data integrity.

For detailed implementation, including Kafka setup, code examples, and performance metrics, see the [Streaming Solution](streaming.html) page.

### Use Case Examples

R2CS supports key regulatory workflows:
1. **ePI Submission Exchanges**: A pharma company streams an ePI Document Bundle to a regulator for real-time validation and regulatory review.
2. **PQI Submission Exchanges**: A pharma company streams quality data to a lab to request a stability study and the lab streams the results back to the company. or the company streams quality data to a regulator for real-time  alidation and regulatory review.
3. **Clinical Data Processing**: Dataset-JSON clinical datasets are streamed for subsecond analysis and storage.
4. **Real-time Analytics**: Regulators monitor submission trends using Kafka Streams or Elasticsearch.

### Resources

- [ePI Implementation Guide](https://build.fhir.org/ig/HL7/emedicinal-product-info/)
- [PQI Implementation Guide](https://build.fhir.org/ig/HL7/uv-dx-pq/)
- [Apache Kafka Documentation](https://kafka.apache.org/)

### Contact

Contact the R2CS project team or contribute via the [R2CS GitHub repository](https://github.com/cander2/recon-ig).

*This IG is under active development. Feedback is welcome to shape its evolution.*
