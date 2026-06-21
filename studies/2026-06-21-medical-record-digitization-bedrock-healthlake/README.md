# Medical Record Digitization with Amazon Bedrock Data Automation and AWS HealthLake

**Date:** 2026-06-21  
**Source:** [Automate medical record digitization with Amazon Bedrock Data Automation and AWS HealthLake](https://aws.amazon.com/blogs/architecture/automate-medical-record-digitization-with-amazon-bedrock-data-automation-and-aws-healthlake/)

## Short Summary

This study explains an event-driven AWS architecture that converts scanned PDF medical records into structured healthcare data. Amazon Bedrock Data Automation extracts clinical fields from documents, AWS Lambda transforms the extraction output into FHIR R4 resources, and AWS HealthLake validates, indexes, and stores the data for standards-based FHIR API access.

## Business Problem

Healthcare organizations often keep historical patient information in scanned documents or paper archives. These records are difficult to search, slow to review, and expensive to manually enter into clinical systems. The result is incomplete patient context, administrative overhead, and delayed downstream analytics.

## Architecture Objective

Design a repeatable pipeline that accepts scanned medical records, extracts clinically relevant fields, converts the data into interoperable FHIR R4 format, and stores it in AWS HealthLake for secure querying by authorized applications.

## Key AWS Services

| Service | Role in the Architecture |
|---|---|
| Amazon Bedrock Data Automation | Extracts structured information from scanned medical documents. |
| AWS HealthLake | Stores, validates, indexes, and exposes FHIR R4 healthcare data. |
| Amazon S3 | Hosts input PDFs and generated extraction/FHIR output files. |
| AWS Lambda | Orchestrates document extraction and FHIR transformation steps. |
| AWS CloudFormation | Deploys infrastructure as code. |
| Amazon CloudWatch | Captures logs, metrics, and operational alarms. |
| AWS CloudTrail | Records API activity for audit and traceability. |
| AWS KMS | Provides encryption key management. |
| IAM roles and policies | Enforce least-privilege service access. |

## Files in This Folder

| File | Purpose |
|---|---|
| [architecture-report.md](architecture-report.md) | Full architecture report with business, technical, security, and operating considerations. |
| [architecture-diagram.mmd](architecture-diagram.mmd) | Mermaid architecture diagram. |
| [slide-deck-outline.md](slide-deck-outline.md) | 10-slide presentation outline. |
| [case-study.md](case-study.md) | Practical healthcare enterprise scenario. |
| [implementation-notes.md](implementation-notes.md) | Technical implementation guidance and production hardening checklist. |
| [references.md](references.md) | Primary reference and related AWS services. |

## Key Learning Outcomes

- Understand how S3 event notifications can drive a serverless document-processing pipeline.
- Explain the role of Bedrock Data Automation in extracting clinical fields from unstructured PDFs.
- Map extracted healthcare data into FHIR R4 resources and NDJSON import files.
- Position AWS HealthLake as a managed FHIR datastore and query layer.
- Identify security, compliance, monitoring, and production-readiness requirements for healthcare workloads.
