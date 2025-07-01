### Overview

The API exchange of medicinal product information (APIX) enables seamless exchange of regulatory submissions between stakeholders, including regulatory authorities (e.g., global health agencies) and pharmaceutical companies. To manage the high volume of submission data—such as electronic Product Information (ePI), Pharmaceutical Quality Information (PQI), and clinical datasets—APIX employs a streaming solution using Apache Kafka and JSON streaming. This page outlines the streaming architecture, its integration with FHIR Transaction Bundles, and implementation guidance for processing large-scale regulatory data in FHIR compliant JSON format.

This solution supports the high-throughput needs of regulatory workflows, handling thousands of daily submissions (e.g., ~100GB compressed JSON) and enabling real-time validation, storage, and querying. It aligns with global trends toward modern data formats and streaming architectures in regulatory exchange.

### Streaming Architecture

The APIX streaming solution leverages Apache Kafka, a distributed streaming platform, to process FHIR Transaction Bundles and Dataset-JSON payloads in real-time. Kafka’s high-throughput, fault-tolerant design ensures scalability and reliability for diverse regulatory and pharma workflows.

#### Key Components

- **Kafka Topics**: Logical channels for data streams (e.g., `APIX-submissions` for FHIR Bundles, `APIX-notifications` for status updates).
- **Producers**: Systems (e.g., submission portals, regulatory gateways) that publish JSON-encoded FHIR Bundles or Dataset-JSON to topics.
- **Consumers**: Services that subscribe to topics for validation, storage, or analytics (e.g., JSON Schema validation, database indexing).
- **Brokers**: Kafka servers in a cluster (3-5 nodes recommended) that store and replicate data.
- **Schema Registry**: Enforces JSON Schema for FHIR Bundles and Dataset-JSON, ensuring compliance with regulatory standards.

#### Workflow

1. **Submission Ingestion**: A producer (e.g., pharma submission system, regulatory gateway) converts submissions (e.g., ePI as FHIR Bundle, clinical data as Dataset-JSON) to JSON and publishes to the `APIX-submissions` topic.
2. **Streaming Processing**:
   - Consumer 1 validates JSON against schemas (e.g., APIX Bundle profile, CDISC Dataset-JSON).
   - Consumer 2 stores valid data in a database (e.g., MongoDB, Elasticsearch).
   - Consumer 3 publishes status updates to `APIX-notifications`.
3. **Real-Time Querying**: APIs (e.g., Elasticsearch-based) serve validated data to stakeholders.
4. **Monitoring**: Prometheus tracks throughput and latency; logs capture JSON payloads for debugging.

#### FHIR Integration

- **Transaction Bundles**: APIX uses Bundle (type: document) in JSON format to encapsulate ePI, PQI, or clinical data. These are streamed as NDJSON (Newline-Delimited JSON) for Kafka compatibility.
- **Task Resource**: Captures submission metadata (e.g., sender, recipient) and links to validation outcomes, included in the Bundle.
- **Provenance Resource**: Tracks data origin and processing history, ensuring traceability for regulatory compliance.

### Technical Implementation

#### Kafka Setup

- **Cluster**: Deploy a 3-5 node Kafka cluster (e.g., Confluent Cloud, AWS MSK, or on-premises) for fault tolerance.
- **Topics**:
  - `APIX-submissions`: FHIR Bundles and Dataset-JSON.
  - `APIX-validation-errors`: Failed validations.
  - `APIX-notifications`: Submission status.
- **Retention**: Configure retention (e.g., 7-30 days) based on regulatory auditing requirements.
- **Schema Registry**: Use Confluent Schema Registry with JSON Schema for APIX profiles and Dataset-JSON.

#### Streaming JSON

- **Format**: Use NDJSON to stream FHIR Bundles and Dataset-JSON (e.g., one Bundle or dataset per line).
- **Libraries**:
  - Python: `ijson` for parsing large JSON datasets.
  - Node.js: `JSONStream` for incremental processing.
- ** Example (Python Producer)**:
  ```python
  from kafka import KafkaProducer
  import json

  producer = KafkaProducer(
      bootstrap_servers=['kafka:9092'],
      value_serializer=lambda v: json.dumps(v).encode('utf-8')
  )

  # FHIR Bundle (simplified ePI)
  bundle = {
      "resourceType": "Bundle",
      "type": "document",
      "id": "ePI-123",
      "entry": [
          {
              "resource": {
                  "resourceType": "Composition",
                  "status": "final",
                  "title": "ePI for [Medicinal Product]"
              }
          }
      ]
  }
  producer.send('APIX-submissions', bundle)
  producer.flush()
```
- **Example (Python Consumer)**:
  ```python
from kafka import KafkaConsumer
from jsonschema import validate
import json

# APIX Bundle schema (simplified)
schema = {
    "type": "object",
    "properties": {
        "resourceType": {"const": "Bundle"},
        "type": {"const": "document"}
    },
    "required": ["resourceType", "type"]
}

consumer = KafkaConsumer(
    'APIX-submissions',
    bootstrap_servers=['kafka:9092'],
    value_deserializer=lambda v: json.loads(v.decode('utf-8'))
)

for message in consumer:
    try:
        validate(instance=message.value, schema=schema)
        print(f"Valid Bundle: {message.value['id']}")
        # Store in database
    except Exception as e:
        print(f"Validation error: {e}")
        producer.send('APIX-validation-errors', {"error": str(e), "bundle": message.value})
```

#### Performance

- **Throughput**: A 5-node Kafka cluster processes ~1GB/s, handling 100GB/day (typical compressed JSON volume for large regulatory workflows) in ~100 seconds.
- **Latency**: Sub-second validation and storage with parallel consumers.
- **Scalability**: Add brokers or partitions to manage peak submission periods (e.g., annual renewals).

#### Dataset-JSON Support

- Clinical datasets use CDISC Dataset-JSON, streamed as NDJSON for compatibility with clinical trial submissions.
- Validation leverages CDISC JSON Schema, integrated with the Schema Registry.
- Example: Stream SDTM dataset rows for real-time validation or analysis.

#### Security and Compliance

- **Encryption**: Use HTTPS for API endpoints and TLS for Kafka communication to protect sensitive data.
- **Validation**: Enforce JSON Schema to comply with international standards (e.g., HL7 FHIR, CDISC, ICH).
- **Access Control**: Implement OAuth2 or mutual TLS for producer/consumer authentication.
- **Auditability**: Log all events with Provenance resources; retain logs to meet regulatory audit requirements (e.g., 21 CFR Part 11, EU GMP Annex 11).
- **Data Integrity**: Use Kafka’s exactly-once semantics to ensure no loss or duplication of submissions.

#### Deployment Considerations

- **Cloud**: Deploy on managed services (e.g., AWS MSK, Confluent Cloud, Azure Event Hubs) or on-premises for flexibility.
- **Databases**: Use Elasticsearch for real-time querying or MongoDB for JSON storage.
- **Monitoring**: Prometheus for metrics (e.g., submission rate, error rate); ELK stack or similar for logging JSON payloads.
- **Legacy Integration**: Support legacy formats (e.g., SAS XPT) by converting to Dataset-JSON using open-source tools from CDISC initiatives.

#### Example Use Case

**Scenario**: A pharmaceutical company submits an ePI Bundle and a Dataset-JSON clinical dataset to a regulatory authority via APIX.
1.  **Submission**: The company’s portal publishes both to APIX-submissions as NDJSON.
2.  **Validation**: A consumer validates against APIX Bundle profile and CDISC schema.
3.  **Storage**: Valid data is indexed in Elasticsearch for querying.
4.  **Notification**: Submission status (accepted or rejected) is published to APIX-notifications.
5.  **Querying**: Regulators or sponsors query Elasticsearch for the ePI or dataset.
  **Sample NDJSON:**
    ```ndjson
  {"resourceType":"Bundle","type":"document","id":"ePI-123","entry":[{"resource":{"resourceType":"Composition","title":"ePI for [Medicinal Product]"}}]}
  {"datasetId":"TRIAL-456","type":"SDTM","data":{"rows":[...]}}
```

#### Implementation Guidance

1.  **Start Small**: Deploy a single-node Kafka cluster for testing with sample FHIR Bundles and Dataset-JSON.
2.  **Schema Development**: Define JSON Schema for APIX Bundle profiles and Dataset-JSON, hosted in the Schema Registry.
3.  **Consumer Logic**: Implement parallel consumers for validation, storage, and notifications, tailored to regulatory requirements.
4.  **Testing**: Use the APIX GitHub repository ([insert link]) to share test Bundles, schemas, and Kafka configurations.
5.  **Scale Up**: Transition to a multi-node cluster for production, integrating with submission portals and regulatory systems.
6.  **Interoperability**: Ensure compatibility with global standards (e.g., FHIR R4/R5, CDISC ODM) and regional requirements.

#### Resources

- **Kafka**: [Apache Kafka Documentation](https://kafka.apache.org/)
- **Schema Registry**: [Confluent Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)
- **FHIR**: [HL7 FHIR Bundle](http://hl7.org/fhir/bundle.html)
- **Tools**: [ijson (Python)](https://pypi.org/project/ijson/), [JSONStream (Node.js)](https://www.npmjs.com/package/JSONStream), [kcat (CLI)](https://github.com/edenhill/kcat)

#### Future Work

- Integrate with GraphQL APIs for advanced querying of submission data.
- Support WebSockets for real-time status updates to stakeholders.
- Expand to additional workflows (e.g., adverse event reporting, marketing authorization applications), as outlined in APIX’s roadmap.
