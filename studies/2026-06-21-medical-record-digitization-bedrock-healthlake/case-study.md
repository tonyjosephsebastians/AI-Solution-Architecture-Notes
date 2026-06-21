# Case Study: Digitizing Historical Medical Records for a Regional Healthcare Network

## Business Context

A regional healthcare network is consolidating several clinics onto a common interoperability platform. Each clinic has years of scanned records stored as PDFs. These records contain diagnoses, medications, lab results, allergies, visit notes, and demographic information, but they are difficult to search or reuse in digital workflows.

The organization wants to automate the first pass of record digitization while preserving controls for validation, audit, and future compliance review.

## User Personas

| Persona | Needs |
|---|---|
| Medical records specialist | Upload batches of scanned records and track processing status. |
| Clinician | Access relevant historical patient information quickly. |
| Interoperability engineer | Convert extracted data into FHIR R4 resources. |
| Security officer | Ensure least-privilege access, auditability, and encryption. |
| Data analyst | Query standardized healthcare data for operational insights. |

## Current Manual Workflow

1. Staff scan or receive PDF medical records.
2. A records specialist opens each file and manually reviews content.
3. Key fields are copied into spreadsheets or clinical systems.
4. Another reviewer validates the entered data.
5. Engineers or analysts later transform subsets of the data for reporting.
6. Historical context remains incomplete because the process is slow and expensive.

## Proposed Automated Workflow

1. Staff upload scanned PDFs to an S3 input bucket.
2. S3 triggers a Lambda function automatically.
3. Lambda starts an Amazon Bedrock Data Automation extraction job.
4. Extracted clinical JSON is written to an S3 output bucket.
5. A second Lambda converts the JSON to FHIR R4 resources and NDJSON.
6. AWS HealthLake imports and validates the FHIR data.
7. Clinical and analytics applications query the data through FHIR API endpoints.
8. Low-confidence records can be routed to human review in a future enhancement.

## Architecture Components

| Component | Purpose |
|---|---|
| S3 input bucket | Stores uploaded scanned PDFs. |
| BDA Trigger Lambda | Starts the Bedrock Data Automation extraction job. |
| Amazon Bedrock Data Automation | Extracts structured clinical fields from medical documents. |
| S3 output bucket | Stores extraction JSON and transformed output. |
| FHIR Processor Lambda | Maps extracted JSON to FHIR R4 resources. |
| AWS HealthLake | Validates, indexes, stores, and exposes FHIR data. |
| CloudWatch | Provides logs, metrics, and alarms. |
| CloudTrail | Captures API activity for audit. |
| IAM and KMS | Provide access control and encryption support. |

## Benefits

- Reduces repetitive manual data entry.
- Makes legacy information easier to query and integrate.
- Uses FHIR R4 to align with healthcare interoperability standards.
- Separates extraction logic from FHIR transformation logic.
- Provides an event-driven foundation that can scale with document volume.
- Creates a clearer audit trail for processing and access patterns.

## Limitations

- OCR and extraction quality can vary with scan quality and document layout.
- FHIR mapping requires domain-specific validation and testing.
- Automated extraction should not replace clinical review for high-risk decisions.
- The demo pattern should use synthetic data only.
- Real PHI requires additional compliance, governance, privacy, and security controls.

## Future Improvements

- Add Amazon SQS for buffering and concurrency control.
- Use AWS Step Functions for job tracking, retries, and human review branches.
- Add confidence-score thresholds and reviewer queues.
- Store provenance metadata linking FHIR resources to source documents.
- Integrate with enterprise identity and role-based access control.
- Add dashboards for processing volume, error rate, and HealthLake import status.
