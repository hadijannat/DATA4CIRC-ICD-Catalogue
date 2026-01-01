# ICD Template Completion Instructions

This document provides step-by-step guidance for completing the Universal ICD Template (`ICD_TEMPLATE.md`).

## Before Starting

### Prerequisites

1. Register the interface in `catalogue/ICD_CATALOGUE.md`
2. Create the ICD directory structure under `icds/{category}/ICD-XX/`
3. Copy the template to the new directory

### Reference Documents

- D2.2 DATA4CIRC Requirements and Specifications (SRS references)
- D4.1 Platform Architecture and Open-Source Protocols (architectural context)
- IDTA AAS Metamodel Specification (semantic identifiers)

## Section-by-Section Guide

### Header Section

Complete the document metadata table:

| Field | Guidance |
|-------|----------|
| Version | Start with 0.1 for initial draft |
| Date | Use DD Month YYYY format (e.g., 15 January 2026) |
| Work Package | WP# responsible for the interface |
| Author(s) | Name, Organisation format |
| Provider Owner | Partner responsible for the provider component |
| Consumer Owner | Partner responsible for the consumer component |
| Status | Draft / Under Review / Approved |

### Section 1: Interface Overview

**1.1 Purpose**: State the primary function of the interface. Reference WP objectives and D2.2 requirements.

**1.2 Communicating Components**: Identify both components with their roles (Provider/Consumer) and responsible partners.

**1.3 Architectural Context**: Reference D4.1 and specify the interface category (Portal-Level, Governance, Application, Dataspace, Data Source).

**1.4 Dependencies**: List prerequisite interfaces (e.g., ICD-1 for SSO) and downstream dependents.

### Section 2: Functional Description

**2.1 Capabilities**: Create a table with capability IDs (FC-01, FC-02, etc.) and trace each to SRS requirements.

**2.2 Interaction Patterns**: Describe message flows. Reference sequence diagrams in Annex A.

**2.3 Error Handling**: 
- For HTTP/REST interfaces: Use RFC 9457 format
- For MQTT/IoT interfaces: Define error topics and DLQ strategies

### Section 3: Abbreviations

List all abbreviations used in the document alphabetically. Each abbreviation shall be defined at first use in the body text.

### Section 4: Communication Protocol

**4.1 Protocol Stack**: Specify protocols at each layer (Application, Security, Transport, Serialisation).

**4.2 Connection Parameters**: Include:
- Base URL or broker address
- Ports and network zones
- Timeout values (specific numbers, not "fast")
- Circuit breaker thresholds

### Section 5: API Specification

**For HTTP/REST interfaces (Sections 5.1, 5.2)**:
- Document each endpoint with method, path, and parameters
- Provide request/response examples with realistic data

**For MQTT/IoT interfaces (Section 5.3)**:
- Define topics, QoS levels, and trigger conditions
- Specify payload formats

### Section 6: Data Structures and Semantics

**6.1 Data Model**: For each field, specify:
- Type (String, Number, Boolean, Object, Array)
- Unit of measure (mandatory for numerical fields)
- Semantic ID (IRDI from ECLASS or IEC CDD for AAS fields)

**6.3 Data Governance**: Flag PII fields and define retention policies per GDPR.

### Section 7: Security Requirements

**7.1-7.3**: Document authentication, authorisation, and transport security.

**7.4 ODRL Policies**: Required for dataspace interfaces (ICD-14 to ICD-17). Define permissions, constraints, duties, and prohibitions.

### Section 8: Performance Requirements

Use specific numerical values:

| Correct | Incorrect |
|---------|-----------|
| < 2 seconds | fast |
| 100 requests/second | high throughput |
| 99.5% availability | highly available |

### Section 9: Implementation Guidelines

**9.1-9.2**: Provide working code examples in Python and Java.

**9.3**: Include Docker Compose or Kubernetes deployment snippets.

**9.4**: Define health check mechanisms:
- HTTP: GET /health, GET /ready endpoints
- MQTT: Last Will and Testament (LWT) configuration

**9.5**: List all environment variables required for deployment.

### Section 10: Requirements Traceability

Map each interface feature to D2.2 SRS requirements. Common SRS references:
- SRS-1-19: Authentication
- SRS-1-20: Role-based access
- SRS-1-22: Response time
- SRS-1-23: Transport encryption
- SRS-1-24: Availability

### Section 11: Acceptance Criteria

Write testable statements:

| Correct | Incorrect |
|---------|-----------|
| The interface returns HTTP 200 within 2 seconds for 95% of requests | The interface should respond quickly |
| Authentication fails with HTTP 401 for invalid tokens | Authentication works properly |

### Annexes

**Annex A**: Include PlantUML sequence diagrams.

**Annex B**: Include complete OpenAPI 3.1.0 specification.

**Annex C**: Include detailed test cases with preconditions and expected results.

**Annex D**: Complete all checklist items before submission.

## Writing Style Checklist

Before submission, verify:

- [ ] British English spelling (serialisation, synchronise)
- [ ] No personal pronouns (the API is implemented, not "we implement")
- [ ] No spatial references (In Section 3, not "above")
- [ ] No temporal references (In M18, not "currently")
- [ ] No subjunctive mood (shall, is, not "could", "might")
- [ ] No filler words (remove "greatly", "heavily", "very")
- [ ] No colloquialisms (retrieve, store, not "get", "put")
- [ ] No em dashes (use commas or parentheses)
- [ ] Units for all numerical values
- [ ] Lowercase unless proper noun

## Submission

1. Complete Annex D Quality Checklist
2. Commit changes with message: `[ICD-XX] feat: Initial specification`
3. Push to feature branch
4. Create pull request for peer review
5. Assign reviewer from different work package
