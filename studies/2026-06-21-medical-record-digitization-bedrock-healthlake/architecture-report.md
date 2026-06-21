# Architecture Report: Medical Record Digitization with Amazon Bedrock Data Automation and AWS HealthLake

## 1. Executive Summary

This architecture automates the conversion of scanned medical record PDFs into structured FHIR R4 healthcare data. The design uses Amazon S3 as the ingestion and handoff layer, AWS Lambda for orchestration and transformation, Amazon Bedrock Data Automation for clinical field extraction, and AWS HealthLake as the managed FHIR datastore.

The portfolio value of this study is its end-to-end view of a healthcare GenAI workflow: unstructured document ingestion, AI-assisted extraction, standards-based data modeling, secure storage, and API-based access.

> Portfolio note: this study is intended for synthetic data and learning purposes. It should not be used with real Protected Health Information (PHI) unless additional HIPAA, privacy, governance, security, and compliance controls are designed, implemented, validated, and operated.

## 2. Problem Statement

Many healthcare providers still depend on scanned records, faxed documents, and archived PDFs. These files may contain useful patient history, but the information is locked inside unstructured formats. Manual review and data entry are slow, inconsistent, and difficult to scale across large record archives.

The technical challenge is not only scanning documents. The system must extract usable clinical fields, preserve traceability, map the results to healthcare interoperability standards, and make the information queryable by downstream applications.

## 3. Business Use Case

A healthcare organization wants to digitize historical patient records before a migration to a modern clinical data platform. The target users include medical records teams, clinicians, interoperability engineers, and analytics teams.

Business outcomes include:

- Faster access to historical patient context.
- Reduced manual data entry effort.
- Better reuse of legacy records in FHIR-compatible applications.
- A repeatable pattern for processing batches of scanned documents.
- A foundation for analytics, search, and care coordination workflows.

## 4. Architecture Overview

The solution follows an event-driven serverless pattern:

1. A user uploads a scanned medical record PDF to an S3 input bucket.
2. An S3 event triggers a Lambda function.
3. Lambda starts an Amazon Bedrock Data Automation job.
4. Bedrock Data Automation extracts structured clinical data from the document.
5. The extraction result is saved as JSON in an S3 output bucket.
6. A second S3 event triggers another Lambda function.
7. The second Lambda maps the extracted JSON to FHIR R4 resources.
8. The data is exported as JSON or NDJSON.
9. AWS HealthLake imports, validates, indexes, and stores the FHIR resources.
10. Authorized applications query the data through FHIR API endpoints.
11. CloudWatch, CloudTrail, IAM, and KMS support observability, audit, access control, and encryption.

## 5. Services and Technologies Used

| Service / Technology | Architecture Responsibility |
|---|---|
| Amazon Bedrock Data Automation | Extracts structured clinical fields from scanned medical documents. |
| AWS HealthLake | Managed FHIR R4 datastore for validation, indexing, storage, and API access. |
| Amazon S3 | Durable storage for input PDFs, extracted JSON, and FHIR export files. |
| AWS Lambda | Event-driven orchestration and transformation logic. |
| AWS CloudFormation | Repeatable infrastructure deployment. |
| Amazon CloudWatch | Logs, metrics, alarms, dashboards, and operational troubleshooting. |
| AWS CloudTrail | API audit trail for governance and investigation. |
| AWS KMS | Encryption key management for data at rest. |
| IAM roles and policies | Least-privilege authorization between AWS services. |
| FHIR R4 / NDJSON | Healthcare interoperability format and HealthLake import structure. |

## 6. End-to-End Request Flow

| Step | Component | Description |
|---:|---|---|
| 1 | Healthcare staff / user | Uploads a scanned PDF record to the S3 input bucket. |
| 2 | Amazon S3 | Emits an object-created event. |
| 3 | BDA Trigger Lambda | Receives the event, validates file metadata, and starts the Bedrock Data Automation job. |
| 4 | Amazon Bedrock Data Automation | Reads the PDF and extracts structured clinical fields. |
| 5 | S3 output bucket | Stores the extraction output as JSON. |
| 6 | S3 output event | Triggers the FHIR Processor Lambda. |
| 7 | FHIR Processor Lambda | Converts extracted fields into FHIR R4 resources and creates bundle/NDJSON output. |
| 8 | AWS HealthLake | Imports, validates, indexes, and stores the FHIR resources. |
| 9 | FHIR API layer | Enables application queries for patients, conditions, medications, observations, and other resources. |

## 7. Data Flow Explanation

The raw data starts as a scanned PDF. Amazon S3 provides durable object storage and event notifications so the pipeline begins automatically when a file arrives. The first Lambda function acts as a thin orchestration layer rather than a document parser. Its job is to pass the source document location to Amazon Bedrock Data Automation and record processing context.

Bedrock Data Automation performs document understanding and returns structured JSON. That output is treated as an intermediate representation, not the final healthcare record. The second Lambda function maps the extracted fields to FHIR resources such as `Patient`, `Condition`, `MedicationRequest`, `Observation`, and related resource types. The transformed data is written as FHIR JSON or NDJSON for HealthLake import.

AWS HealthLake becomes the system of query for this digitized subset. It validates FHIR R4 structure, indexes resources, and exposes standards-based APIs that downstream applications can call.

## 8. Security Considerations

Security should be designed around healthcare data sensitivity, even when using synthetic data for a demo.

Key controls:

- Use synthetic data for portfolio demonstrations.
- Store S3 objects in private buckets with public access blocked.
- Encrypt S3 buckets, Lambda environment variables, and HealthLake data with AWS KMS where supported.
- Use IAM roles with least-privilege permissions for Lambda, Bedrock Data Automation, S3, and HealthLake interactions.
- Enable CloudTrail for management and data event visibility where appropriate.
- Avoid logging full document content, extracted PHI, or FHIR payloads in CloudWatch.
- Use separate AWS accounts or environments for development, testing, and production.
- For real PHI, complete HIPAA, BAA, privacy, governance, threat modeling, retention, incident response, and access-review work before deployment.

## 9. Scalability Considerations

The architecture scales well for asynchronous document workflows because S3, Lambda, and managed AWS services absorb much of the operational load. For larger volumes, the design should add backpressure and orchestration controls.

Recommended scaling enhancements:

- Add Amazon SQS between S3 events and Lambda to buffer burst uploads.
- Use AWS Step Functions for retries, exception handling, manual review routing, and job status tracking.
- Track Bedrock Data Automation service quotas and concurrency limits.
- Use idempotency keys to prevent duplicate HealthLake imports.
- Partition S3 prefixes by date, tenant, facility, or document type.
- Separate low-confidence extraction workflows from fully automated imports.

## 10. Cost Considerations

Primary cost drivers are document-processing volume, page count, HealthLake storage/query usage, Lambda duration, S3 storage, KMS requests, and any production networking controls such as VPC endpoints.

Cost controls:

- Use lifecycle policies on input, intermediate, and output buckets.
- Delete demo stacks when not in use.
- Track processing cost per document and per page.
- Monitor Lambda duration and memory settings.
- Set AWS Budgets alerts for lab environments.
- Avoid repeated imports of the same FHIR resources.

## 11. Monitoring and Observability

CloudWatch and CloudTrail should make the pipeline explainable from upload to query.

Operational signals:

- Count of PDFs uploaded to the input bucket.
- BDA Trigger Lambda invocation, error, throttle, and duration metrics.
- Bedrock Data Automation job status and failures.
- Count of extraction JSON files written to the output bucket.
- FHIR Processor Lambda conversion errors and validation failures.
- HealthLake import job status.
- CloudTrail API events for data access and administrative changes.
- Alarms for repeated failures, DLQ growth, and unusual access patterns.

## 12. Risks and Trade-offs

| Risk / Trade-off | Explanation | Mitigation |
|---|---|---|
| Extraction quality | Scanned images may be noisy, incomplete, or inconsistent. | Use confidence thresholds, synthetic test sets, and human review queues. |
| FHIR mapping complexity | Clinical data does not always map cleanly from raw extraction to FHIR resources. | Maintain mapping rules, validation tests, and clinical review. |
| Compliance scope | Healthcare data introduces strict security and privacy obligations. | Use synthetic data for demos and complete compliance controls for PHI. |
| Asynchronous latency | The workflow is not designed for instant synchronous responses. | Communicate job status and use event-driven notifications. |
| Duplicate imports | Reprocessing the same document can create duplicate resources. | Add idempotency, deduplication, and provenance tracking. |

## 13. Production Readiness Notes

Before using this pattern for real patient data, add:

- Confirmed AWS Business Associate Addendum coverage where required.
- HIPAA-aligned security architecture and documented shared responsibility review.
- Private networking and VPC endpoint strategy.
- Encryption policy, key rotation, and access controls.
- PHI-safe logging and redaction controls.
- Formal data retention and deletion policies.
- Clinical validation of extraction and FHIR mapping accuracy.
- Human-in-the-loop review for low-confidence or high-risk fields.
- Disaster recovery and backup procedures.
- CI/CD pipeline with security scanning and infrastructure policy checks.

## 14. LinkedIn Post Version

Today’s GenAI solution architecture study: automating medical record digitization with Amazon Bedrock Data Automation and AWS HealthLake.

The pattern is simple but powerful:

- Upload scanned PDFs to Amazon S3.
- Trigger Lambda from S3 events.
- Use Bedrock Data Automation to extract clinical fields.
- Convert extracted JSON into FHIR R4 resources.
- Import NDJSON into AWS HealthLake.
- Query healthcare data through FHIR APIs.

The biggest takeaway: successful healthcare AI architecture is not just about extraction. It is about trustworthy data flow, standards-based interoperability, security, auditability, and operational controls.

For portfolio/demo work, synthetic data is essential. Real PHI requires additional HIPAA, privacy, governance, and production security controls.

## 15. GitHub README Version

This study documents an AWS event-driven architecture for transforming scanned medical record PDFs into FHIR R4-compliant healthcare data. It uses Amazon Bedrock Data Automation for document extraction, AWS Lambda for orchestration and FHIR conversion, Amazon S3 for input/output storage, and AWS HealthLake as the managed FHIR datastore. Supporting services include CloudFormation, CloudWatch, CloudTrail, IAM, and KMS.
