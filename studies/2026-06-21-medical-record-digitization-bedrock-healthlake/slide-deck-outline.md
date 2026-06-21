# Slide Deck Outline: Medical Record Digitization with Bedrock Data Automation and HealthLake

## Slide 1: Title

- Medical Record Digitization with Amazon Bedrock Data Automation and AWS HealthLake
- Daily GenAI solution architecture study
- Date: 2026-06-21
- Focus: scanned PDFs to FHIR R4 data

**Speaker notes:** Introduce the architecture as a healthcare document AI pattern that turns unstructured records into queryable clinical data.

**Suggested visual:** Title slide with AWS icons for S3, Lambda, Bedrock, and HealthLake.

## Slide 2: Problem Statement

- Historical medical records often exist as scanned PDFs.
- Manual data entry is slow, expensive, and error-prone.
- Clinical teams need searchable patient history.
- Interoperability requires structured standards like FHIR.

**Speaker notes:** Emphasize that scanning alone does not solve the problem; the data must become structured and usable.

**Suggested visual:** Before/after image: paper archive to API-accessible healthcare data.

## Slide 3: Business Use Case

- Digitize legacy patient records during EHR modernization.
- Reduce manual abstraction workload for operations teams.
- Make patient history available to clinical applications.
- Support analytics and care coordination use cases.

**Speaker notes:** Position the solution as an enterprise modernization pattern, not just a document extraction demo.

**Suggested visual:** Personas and outcomes matrix.

## Slide 4: High-Level Architecture

- S3 input bucket receives scanned PDFs.
- Lambda starts Bedrock Data Automation processing.
- S3 output bucket stores extraction JSON.
- Lambda converts JSON to FHIR R4 and NDJSON.
- HealthLake stores and exposes FHIR data.

**Speaker notes:** Walk through the major components from left to right.

**Suggested visual:** Mermaid or AWS architecture diagram.

## Slide 5: End-to-End Data Flow

- Upload PDF to S3.
- Trigger extraction with an event-driven Lambda.
- Generate structured clinical JSON.
- Transform extracted fields into FHIR resources.
- Import and query data in HealthLake.

**Speaker notes:** Explain how event notifications remove manual orchestration and polling.

**Suggested visual:** Numbered sequence diagram.

## Slide 6: AWS Services Used

- Bedrock Data Automation: document intelligence.
- HealthLake: FHIR R4 datastore and API access.
- S3 and Lambda: event-driven pipeline backbone.
- CloudFormation: repeatable deployment.
- CloudWatch, CloudTrail, IAM, and KMS: operations and security.

**Speaker notes:** Describe each service by responsibility, not by marketing category.

**Suggested visual:** Service responsibility table.

## Slide 7: Security and Compliance

- Use synthetic data for demo and portfolio work.
- Apply least-privilege IAM roles and private S3 buckets.
- Encrypt data with KMS where supported.
- Use CloudTrail for audit and CloudWatch for operational logs.
- Real PHI requires additional HIPAA and governance controls.

**Speaker notes:** Be clear that a demo architecture is not automatically production-ready for patient data.

**Suggested visual:** Security control layers around the pipeline.

## Slide 8: Scalability and Cost

- Serverless services scale independently for asynchronous workloads.
- SQS can buffer bursts and control downstream concurrency.
- Step Functions can manage retries and human review paths.
- Costs are driven by page volume, HealthLake usage, Lambda, S3, and KMS.
- Lifecycle policies and budgets help control lab costs.

**Speaker notes:** Discuss both technical scaling and financial scaling.

**Suggested visual:** Cost driver and scaling control table.

## Slide 9: Monitoring and Operations

- Track Lambda invocations, errors, throttles, and duration.
- Monitor BDA job status and extraction failures.
- Observe HealthLake import status and API errors.
- Use alarms for repeated failures and backlog growth.
- Audit access and administrative actions with CloudTrail.

**Speaker notes:** Explain how operators can trace a document from upload to FHIR availability.

**Suggested visual:** Operational dashboard mockup.

## Slide 10: Key Takeaways

- Event-driven architecture is a strong fit for document processing.
- Bedrock Data Automation reduces custom extraction logic.
- FHIR R4 mapping is the key interoperability step.
- HealthLake provides managed validation, indexing, and APIs.
- Healthcare AI needs security, compliance, and governance from day one.

**Speaker notes:** Close with the architecture lessons that transfer to real enterprise projects.

**Suggested visual:** Five takeaway icons.
