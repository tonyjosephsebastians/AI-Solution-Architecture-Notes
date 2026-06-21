# Implementation Notes

## Prerequisites

- AWS account with permissions for S3, Lambda, IAM, CloudFormation, CloudWatch, CloudTrail, KMS, Amazon Bedrock Data Automation, and AWS HealthLake.
- AWS CLI configured for the target account and Region.
- Basic understanding of FHIR R4 resources and NDJSON import patterns.
- Synthetic scanned medical record PDFs for testing.
- Clear naming convention for buckets, Lambda functions, IAM roles, and CloudWatch log groups.

> Security note: use synthetic data for demo and portfolio purposes. Do not claim or treat this as production-ready for real PHI unless additional HIPAA, privacy, governance, security, and compliance controls are implemented and validated.

## Deployment Approach

A practical deployment can be organized into three layers:

1. **Storage and security foundation:** S3 buckets, KMS keys, IAM roles, and logging.
2. **Processing layer:** Lambda functions, S3 event notifications, Bedrock Data Automation configuration, and CloudWatch log groups.
3. **Healthcare data layer:** AWS HealthLake datastore, import permissions, and FHIR API access patterns.

Deploy lower-level dependencies first, then attach event notifications after the Lambda functions and permissions exist.

## Infrastructure as Code Approach

Use AWS CloudFormation to define:

- S3 input and output buckets.
- Bucket encryption and public access block settings.
- Lambda functions and execution roles.
- IAM policies for S3, Bedrock Data Automation, HealthLake, CloudWatch Logs, and KMS.
- CloudWatch log groups and alarms.
- KMS keys and aliases.
- HealthLake FHIR R4 datastore.
- S3 event notifications for input and output buckets.

Keep environment-specific values such as prefixes, retention periods, and bucket names parameterized.

## Lambda Function Responsibilities

### BDA Trigger Lambda

- Receive S3 `ObjectCreated` events from the input bucket.
- Validate that the uploaded object is an expected PDF file.
- Build the Bedrock Data Automation request using the S3 object URI.
- Start the extraction job.
- Write correlation IDs and job metadata to logs or a tracking store.
- Avoid logging sensitive document content.

### FHIR Processor Lambda

- Receive S3 `ObjectCreated` events from the output bucket.
- Load the extracted JSON output.
- Validate required fields and confidence scores.
- Map extracted fields to FHIR R4 resources.
- Generate a FHIR Bundle and NDJSON files.
- Start or prepare the AWS HealthLake import process.
- Record conversion errors without exposing PHI in logs.

## S3 Event-Driven Processing

S3 event notifications connect the pipeline stages without polling:

- Input bucket event: PDF upload triggers BDA Trigger Lambda.
- Output bucket event: extraction JSON arrival triggers FHIR Processor Lambda.

For higher-volume production workloads, consider adding SQS between S3 and Lambda. SQS can absorb bursts, support retries, and reduce the chance of overwhelming downstream service limits.

## Bedrock Data Automation Role

Amazon Bedrock Data Automation acts as the document intelligence layer. It reads scanned medical records from S3 and returns structured extraction output. In this architecture, BDA should be scoped to the minimum input and output locations required for the workflow.

Implementation considerations:

- Define the clinical fields that must be extracted.
- Preserve confidence scores for downstream validation.
- Version extraction configuration when field definitions change.
- Track job IDs for troubleshooting and replay.

## FHIR Conversion Logic

The FHIR Processor Lambda should treat BDA JSON as an intermediate format. Common mappings may include:

| Extracted Data | FHIR R4 Resource |
|---|---|
| Patient demographics | `Patient` |
| Diagnoses and ICD-10 codes | `Condition` |
| Medications | `MedicationRequest` or `MedicationStatement` |
| Lab results | `Observation` |
| Vital signs | `Observation` |
| Source document metadata | `DocumentReference` or `Provenance` |

Conversion logic should include:

- Required-field checks.
- Data type normalization.
- Code system mapping.
- Confidence threshold handling.
- Resource relationship linking.
- Idempotency and duplicate detection.

## HealthLake Ingestion

AWS HealthLake imports FHIR R4 resources, validates them, indexes them, and makes them available through FHIR APIs. For batch imports, NDJSON is a common exchange format.

Recommended ingestion practices:

- Store NDJSON files in a controlled S3 prefix.
- Use clear import job naming and correlation metadata.
- Monitor import job status.
- Capture validation errors for remediation.
- Query sample resources after import to verify availability.

## Testing Approach Using Synthetic Data

Use synthetic PDFs and synthetic patient profiles only. A safe testing plan includes:

- Unit tests for field mapping functions.
- Contract tests for expected BDA JSON structure.
- FHIR validation tests for generated resources.
- S3 event simulation tests for Lambda handlers.
- End-to-end tests with synthetic PDF uploads.
- Negative tests for unsupported file types, missing fields, and low-confidence extraction.

## Error Handling

- Reject unsupported file extensions early.
- Add retries for transient service errors.
- Use dead-letter queues or failure destinations for Lambda errors.
- Capture BDA job failures with correlation IDs.
- Separate validation failures from infrastructure failures.
- Route low-confidence extractions to review instead of automatically importing them.
- Make processing idempotent so replaying an event does not create duplicate data.

## Logging and Monitoring

Use CloudWatch for operational visibility and CloudTrail for audit visibility.

Recommended logs and metrics:

- PDF upload count.
- Lambda invocation count, duration, errors, and throttles.
- BDA job started, completed, and failed counts.
- Extraction output files created.
- FHIR conversion success and failure counts.
- HealthLake import job status.
- API access patterns from CloudTrail.

Avoid logging full documents, full extracted medical content, or complete FHIR payloads in shared logs.

## Production Hardening Checklist

- [ ] Confirm HIPAA and AWS Business Associate Addendum requirements.
- [ ] Use private networking and VPC endpoints where appropriate.
- [ ] Enforce least-privilege IAM roles and regular access reviews.
- [ ] Encrypt data at rest and in transit.
- [ ] Add SQS buffering and DLQs.
- [ ] Add Step Functions for retries, status tracking, and human review.
- [ ] Implement PHI-safe logging, redaction, and retention policies.
- [ ] Add FHIR validation and clinical review workflows.
- [ ] Add provenance links between FHIR resources and source documents.
- [ ] Implement idempotency and duplicate detection.
- [ ] Add backup, recovery, and incident response procedures.
- [ ] Run security, privacy, compliance, and threat-model reviews before real PHI use.
