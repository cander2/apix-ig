### Introduction

This IG enables the vision of reducing pharmaceutical regulatory transaction and processing timelines from days or hours to subsecond timelines, using digitization, automation, and streaming technologies.

Inspired by the Uppsala Monitoring Centre's (UMC) [IDMP Request and Publish API IG](https://build.fhir.org/ig/Uppsala-Monitoring-Centre/WHO-UMC-IDMP-Service/index.html) and the finance industry's real-time algorithmic systems ([Anderson, Algorri, Abernathy 2023](https://pubmed.ncbi.nlm.nih.gov/37619807/)), R2CS leverages HL7 FHIR and modern API based technologies to deliver automated, interoperable, and scalable data exchange for electronic Product Information (ePI), Pharmaceutical Quality Information (PQI), pharmacovigilance, requests for information, and clinical trials.

R2CS addresses the need for modern, real-time regulatory processes by:
- **Streaming Data**: Using Kafka to stream FHIR Transaction Bundles and Dataset-JSON as NDJSON, achieving subsecond validation and processing.
- **Automation**: Enabling algorithmic workflows for validation, routing, and analytics, akin to trading platforms.
- **Interoperability**: Supporting global standards (HL7 FHIR, CDISC, ICH) for ePI, PQI, and clinical submissions.
- **Scalability**: Handling high-volume submissions (e.g., ~100GB/day compressed JSON) with fault-tolerant architecture.

### Scope

R2CS targets pharmaceutical regulatory affairs, enabling:
- **Electronic Product Information (ePI)**: Drug labeling transactions.
- **Pharmaceutical Quality Information (PQI)**: Pharmaceutical quality transactions transactions.
- **Request for Information (RFI)**: Health authority questions to manufacturers, and manufacturers response back to the health authority.
- **Clinical trial**: Clinical trial transactions.
- **Global Applicability**: Supports regulators, pharma companies, and healthcare institutions with standardized, automated workflows.

This IG provides technical guidance, profiles, and examples for implementing real-time regulatory exchange, with an intitial focus on medicinal products, pharmaceutical quality and drug labeling. Future iterations will expand the scope to include other regulated product types (e.g., medical devices, veterinary, over the counter, and natural health products), and other data domains (e.g., clinical and adverse event reporting).

### Streaming Solution

R2CS’s goal is to enable real-time streaming architecture, which processes FHIR Transaction Bundles with subsecond latency. 

**Figure 1: Anatomy of a Regulatory Application or Transaction on FHIR**
<img src="transactionanatomy.png" alt="Components of the Regulatory Transaction Bundle" style="float: none;" style="width:500px;height:600px;"/>

This IG describes several technical solution options capable of supporting low, medium, or high transaction volumes. However, these are merely suggestions for discussion. Implementers are encouraged to select any FHIR compliant technical solution that suits their unique requirements and means. 

This IG describes various technology options that implementers can use to support low, medium and high transaction volumes:
- **Create Transactions**: Producers (e.g., pharma portals, regulatory gateways) send encoded FHIR Bundles to Consumers (e.g., health authority, CRO, CMO).
- **Processes Data**: Consumers validate against JSON Schema, store in databases (e.g., Elasticsearch, MongoDB), and publish status updates.
- **Enables Querying**: Real-time APIs deliver validated data to stakeholders.
- **Ensures Compliance**: Provenance resources ensure traceability and data integrity.

For detailed implementation, including Kafka setup, code examples, and performance metrics, see the [Solution Options](streaming.html) page.

### Use Case Examples

R2CS supports key regulatory workflows:
1. **ePI Submission Exchanges**: A pharma company streams an ePI Document Bundle to a regulator for real-time validation and regulatory review.
2. **PQI Submission Exchanges**: A pharma company streams quality data to a lab to request a stability study and the lab streams the results back to the company. Or the company streams quality data to a regulator for real-time  validation and regulatory review.
3. **Clinical Data Processing**: Clinical datasets are streamed for real-time processing.
4. **Real-time Analytics**: Regulators monitor submission trends using Kafka Streams or Elasticsearch.

### Resources

- [ePI Implementation Guide](https://build.fhir.org/ig/HL7/emedicinal-product-info/)
- [PQI Implementation Guide](https://build.fhir.org/ig/HL7/uv-dx-pq/)
- [Apache Kafka Documentation](https://kafka.apache.org/)

### Contact

Contact the R2CS project team or contribute via the [R2CS GitHub repository](https://github.com/cander2/recon-ig).

*This IG is under active development. Feedback is welcome to shape its evolution.*
