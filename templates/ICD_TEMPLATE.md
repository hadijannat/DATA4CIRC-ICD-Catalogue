# ICD-[XX]: [Interface Name]

**[Component A] ↔ [Component B]**

---

| Attribute | Value |
|-----------|-------|
| **Version** | [X.Y] |
| **Date** | [DD Month YYYY] |
| **Work Package** | [WP#] |
| **Author(s)** | [Name, Organisation] |
| **Provider Owner** | [Name, Organisation] |
| **Consumer Owner** | [Name, Organisation] |
| **Reviewer** | [Name, Organisation] |
| **Status** | [Draft / Under Review / Approved] |

---

## Document Completion Guidelines

> **MANDATORY**: All content shall be written in formal, scientific style conforming to IEEE conventions. The following rules apply throughout the document.

| Rule | Incorrect Example | Correct Example |
|------|-------------------|-----------------|
| British English | serialization, synchronize | serialisation, synchronise |
| No personal pronouns | We implement the API... | The API is implemented... |
| No spatial references | here, there, above, below | In Section 3, In Table 2 |
| No temporal references | now, currently, later, early | At the time of publication, In M18 |
| No subjunctive mood | could, would, might, maybe | shall, is, provides, implements |
| No filler words | greatly, heavily, very, easily | [Remove or use precise terms] |
| No colloquialisms | get, put, thing, stuff | retrieve, store, component, data |
| No em dashes | The system—which is fast— | The system, which is fast, |
| No ambiguous quantifiers | fast, minimal, high-performance | < 200 ms, 10 MB maximum |
| Lowercase unless proper noun | Digital Product Passport Tool | digital product passport tool |
| Units mandatory | weight: 5.2 | weight: 5.2 kg |

> **ABBREVIATION RULE**: Each abbreviation shall be defined exactly once at first use in the format: Full Term (ABBR). Subsequently, only the abbreviation is used. All abbreviations shall also appear in Section 3 (Abbreviations).

---

## 1. Interface Overview

### 1.1 Purpose

> **INSTRUCTION**: Describe the primary function of the interface. State what data or control flows are exchanged and why this interface is necessary for platform operation. Reference the relevant Work Package objectives.

[Describe the interface purpose. State the communication objectives, the type of data exchanged, and the operational necessity. Reference WP objectives and D2.2 requirements as appropriate.]

### 1.2 Communicating Components

> **INSTRUCTION**: Identify the two components or systems that communicate through this interface. Specify the component names, their architectural roles, and the responsible Work Package.

| Attribute | Component A | Component B |
|-----------|-------------|-------------|
| **Name** | [Component name] | [Component name] |
| **Role** | [e.g., Consumer, Provider] | [e.g., Consumer, Provider] |
| **Work Package** | [WP#] | [WP#] |
| **Responsible Partner** | [Partner acronym] | [Partner acronym] |

### 1.3 Architectural Context

> **INSTRUCTION**: Position the interface within the DATA4CIRC platform architecture. Reference the architectural domain (Portal-Level, Governance, Application, Dataspace, or Data Source) and explain the interface position in the overall data flow.

[Describe the architectural placement. Reference D4.1 architectural documentation. Identify upstream and downstream interfaces. Specify the interface category from the ICD Catalogue.]

### 1.4 Interface Dependencies and Lifecycle

> **INSTRUCTION**: Identify upstream interfaces required for this interface to function. Define the versioning strategy and deprecation policy. This section ensures implementers understand interface prerequisites and evolution management.

| Attribute | Specification |
|-----------|---------------|
| **Prerequisites** | [e.g., ICD-1 (Active), DNS Resolution, Keycloak availability] |
| **Versioning Strategy** | [e.g., Semantic Versioning in URI: /api/v2 or Header-based: X-API-Version] |
| **Deprecation Policy** | [e.g., 6-month sunset period with deprecation header warnings] |
| **Downstream Dependents** | [e.g., ICD-8, ICD-12 depend on this interface] |

---

## 2. Functional Description

### 2.1 Functional Capabilities

> **INSTRUCTION**: List and describe each function the interface provides. Use a structured table format. Each capability shall trace to requirements in D2.2.

| ID | Capability | Description | SRS Reference |
|----|------------|-------------|---------------|
| FC-01 | [Capability name] | [Brief description] | [SRS-X-Y] |
| FC-02 | [Capability name] | [Brief description] | [SRS-X-Y] |

### 2.2 Interaction Patterns

> **INSTRUCTION**: Describe the sequence of interactions between components. Include initiating conditions, message exchange sequences, and terminating conditions. Reference sequence diagrams in Annex A where appropriate.

[Describe interaction patterns. Include request-response flows, asynchronous messaging patterns, and event-driven behaviours. Reference sequence diagrams.]

### 2.3 Error Handling

> **INSTRUCTION**: Specify error conditions and recovery procedures. Select the appropriate error handling mechanism based on the interface protocol type. HTTP/REST interfaces shall conform to RFC 9457. IoT/Async interfaces shall define error topics and Dead Letter Queue (DLQ) strategies.

#### 2.3.1 HTTP/REST Error Handling

For HTTP/REST interfaces, error responses shall conform to RFC 9457 (Problem Details for HTTP APIs).

| HTTP Status | Condition | Recovery Action |
|-------------|-----------|-----------------|
| [4xx/5xx] | [Error condition description] | [Recovery procedure] |

#### 2.3.2 IoT/Async Error Handling

For MQTT and asynchronous interfaces, error handling shall use dedicated error topics and Dead Letter Queue (DLQ) strategies.

| Attribute | Specification |
|-----------|---------------|
| **Error Topic** | [e.g., data4circ/{component}/errors] |
| **DLQ Strategy** | [e.g., Route to DLQ after 3 processing failures] |
| **Error Payload Schema** | [e.g., {error_code, message, timestamp, original_payload}] |
| **Retry Policy** | [e.g., Exponential backoff: 1s, 2s, 4s, then DLQ] |

---

## 3. Abbreviations

> **INSTRUCTION**: List all abbreviations used in this document in alphabetical order. Include only abbreviations that appear in the document text. Each abbreviation shall be defined at first use in the body text.

| Abbreviation | Definition |
|--------------|------------|
| AAS | Asset Administration Shell |
| API | Application Programming Interface |
| DLQ | Dead Letter Queue |
| IRDI | International Registration Data Identifier |
| LWT | Last Will and Testament (MQTT) |
| [ABBR] | [Full definition] |

---

## 4. Communication Protocol

### 4.1 Protocol Stack

> **INSTRUCTION**: Specify the communication protocols, transport mechanisms, and connection parameters. Reference applicable standards and specifications. For IoT interfaces (ICD-18.x), specify MQTT v5, OPC UA, or database connector protocols.

| Layer | Protocol | Specification |
|-------|----------|---------------|
| Application | [e.g., HTTP/REST, DSP, MQTT v5] | [Standard reference] |
| Security | [e.g., OAuth 2.0, TLS 1.3, mTLS] | [RFC reference] |
| Transport | [e.g., HTTPS, TCP, WebSocket] | [Standard reference] |
| Serialisation | [e.g., JSON, JSON-LD, Protobuf] | [RFC/Standard] |

### 4.2 Connection Parameters

> **INSTRUCTION**: Specify connection endpoints, ports, timeout values, retry policies, and resilience patterns. Include network zone classification, circuit breaker thresholds, and firewall requirements.

| Parameter | Value |
|-----------|-------|
| **Base URL / Broker** | [https://api.example.com/v1 or mqtt://broker.example.com] |
| **Port** | [443 / 8883] |
| **Network Zone** | [e.g., Public Internet / DMZ / Manufacturing VLAN] |
| **Connection Timeout** | [30 seconds] |
| **Read Timeout** | [60 seconds] |
| **Retry Policy** | [3 retries with exponential backoff: 1s, 2s, 4s] |
| **Circuit Breaker** | [e.g., Open after 5 consecutive failures; Half-open after 30s; Close on success] |
| **Firewall Rules** | [e.g., Allow inbound TCP 8883 from 10.0.0.0/8] |

> **NOTE**: Circuit breaker patterns prevent cascading failures in distributed systems. The circuit breaker opens after a threshold of consecutive failures, blocking further requests until a reset timeout elapses.

---

## 5. API Specification

### 5.1 Endpoint Definitions

> **INSTRUCTION**: Document each API endpoint with method, path, purpose, and parameters. Group related endpoints logically.

#### 5.1.1 [Endpoint Name]

| Attribute | Value |
|-----------|-------|
| **Method** | [GET / POST / PUT / DELETE] |
| **Path** | [/api/v1/resource/{id}] |
| **Purpose** | [Brief description of endpoint purpose] |
| **Authentication** | [Bearer Token / API Key / None] |

**Path Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| [param] | [string] | [Yes/No] | [Description] |

### 5.2 Request and Response Examples

> **INSTRUCTION**: Provide concrete examples of request and response payloads. Use realistic data that demonstrates typical usage scenarios.

**Request Example:**

```json
{
  "field": "value"
}
```

**Response Example (200 OK):**

```json
{
  "field": "value"
}
```

### 5.3 Event and Message Specifications (Asynchronous/MQTT)

> **INSTRUCTION**: Complete this section for event-driven and IoT interfaces (ICD-18.x). Define MQTT topics, message directions, QoS levels, and trigger conditions instead of HTTP methods and paths.

#### 5.3.1 [Message/Event Name]

| Attribute | Specification |
|-----------|---------------|
| **Topic/Channel** | [e.g., factory/line1/telemetry/+] |
| **Direction** | [Publish / Subscribe] |
| **QoS Level** | [0 / 1 / 2] |
| **Trigger Condition** | [e.g., On value change > 5%, periodic 60s interval] |
| **Payload Format** | [JSON / Protobuf / SparkplugB] |
| **Retention** | [Retained / Non-retained] |

---

## 6. Data Structures and Semantics

### 6.1 Data Model

> **INSTRUCTION**: Document each data structure with its fields, types, units of measure, and semantic identifiers. Use JSON Schema or similar notation for formal specification. All numerical fields shall specify the unit of measure. Semantic IDs (IRDIs) from ECLASS or IEC CDD shall be provided for AAS-compliant fields.

> **CRITICAL**: All numerical fields shall specify the Unit of Measure (UoM). Unitless numbers are grounds for rejection in AAS/LCA contexts. Semantic IDs from ECLASS (0173-1#...) or IEC CDD ensure interoperability across the DATA4CIRC ecosystem.

#### 6.1.1 [Data Structure Name]

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| weight | Number | kg | 0173-1#02-BAA123#001 | Y | Net weight of product |
| timestamp | String | ISO 8601 | N/A | Y | Measurement timestamp |
| temperature | Number | °C | 0173-1#02-AAB999#002 | N | Operating temperature |
| [field] | [type] | [unit] | [IRDI or N/A] | [Y/N] | [Description] |

### 6.2 Semantic Mappings

> **INSTRUCTION**: Document mappings to standard vocabularies such as ECLASS, IEC CDD, or AAS submodel templates. Specify semantic identifiers (IRDIs) where applicable.

[Document semantic mappings to ECLASS, IEC CDD, DCAT-AP, or other standard vocabularies. Include IRDI references.]

### 6.3 Data Governance and Compliance

> **INSTRUCTION**: Flag data fields for GDPR compliance (PII identification) and define retention rules. This section ensures SPIP (Security Privacy Integrity Protection) compliance and supports data protection impact assessments.

| Data Entity | PII (Y/N) | Classification | Retention Period |
|-------------|-----------|----------------|------------------|
| [UserEmail] | Y | Confidential | [Delete on account closure] |
| [MachineID] | N | Internal | [Permanent] |
| [DataEntity] | [Y/N] | [Classification] | [Retention rule] |

> **NOTE**: Classification levels: Public (no restrictions), Internal (organisation access), Confidential (role-based access), Restricted (specific authorisation required). Retention periods shall comply with GDPR Article 5(1)(e) storage limitation principle.

---

## 7. Security Requirements

### 7.1 Authentication

> **INSTRUCTION**: Specify the authentication mechanism, identity provider, token type, and token lifetime.

| Attribute | Value |
|-----------|-------|
| **Mechanism** | [OAuth 2.0 / OpenID Connect / mTLS / API Key] |
| **Identity Provider** | [Keycloak / External IdP] |
| **Token Type** | [JWT / Opaque Token] |
| **Token Lifetime** | [e.g., 3600 seconds] |

### 7.2 Authorisation

> **INSTRUCTION**: Specify the authorisation model (RBAC, ABAC) and required roles/permissions for each operation.

| Operation | Required Role | SRS Reference |
|-----------|---------------|---------------|
| [Operation] | [Role] | [SRS-1-19] |

### 7.3 Transport Security

| Attribute | Value |
|-----------|-------|
| **TLS Version** | [TLS 1.3 minimum] |
| **Certificate Validation** | [X.509 / CA-signed / Self-signed for dev] |
| **Cipher Suites** | [TLS_AES_256_GCM_SHA384, ...] |

### 7.4 Usage Control (ODRL Policies)

> **INSTRUCTION**: For dataspace interfaces (ICD-14 to ICD-17), define the ODRL (Open Digital Rights Language) policies enforced by the Eclipse Dataspace Connector. Distinguish between access control (visibility) and usage control (processing rights).

> **NOTE**: Access control determines visibility (can the consumer see the data?). Usage control determines processing rights (can the consumer store, modify, or redistribute the data?). ODRL policies are enforced by the EDC Policy Engine.

| Policy Element | Specification |
|----------------|---------------|
| **Permission** | [e.g., USE, DISPLAY, DISTRIBUTE, MODIFY] |
| **Constraint** | [e.g., spatial equals "EU", temporal before "2026-12-31"] |
| **Duty** | [e.g., LOG usage, DELETE after processing, NOTIFY provider] |
| **Prohibition** | [e.g., TRANSFER to third parties, STORE beyond session] |

---

## 8. Performance Requirements

> **INSTRUCTION**: Specify quantitative performance targets derived from D2.2 requirements. Include response time, throughput, and availability targets. Avoid ambiguous quantifiers; use specific numerical values.

| Metric | Target | SRS Reference |
|--------|--------|---------------|
| Response Time (P95) | [< 2 seconds] | [SRS-1-22] |
| Throughput | [100 requests/second] | [SRS-X-Y] |
| Availability | [99.5%] | [SRS-1-24] |
| Max Payload Size | [10 MB] | [SRS-X-Y] |

---

## 9. Implementation Guidelines

### 9.1 Client Implementation Example

> **INSTRUCTION**: Provide a working code example demonstrating client-side interface implementation. Include error handling and authentication.

**Python (FastAPI) Example:**

```python
# [Insert Python implementation code with comments explaining key integration points]
```

### 9.2 Server Implementation Example

**Java (Spring Boot) Example:**

```java
// [Insert Java implementation code with annotations and comments]
```

### 9.3 Deployment Configuration

> **INSTRUCTION**: Provide Docker Compose or Kubernetes configuration examples for deployment.

```yaml
# [Insert docker-compose.yml or Kubernetes manifest snippet]
```

### 9.4 Observability and Tracing

> **INSTRUCTION**: Define observability requirements for distributed tracing and health monitoring. Select the appropriate mechanism based on the interface protocol type. HTTP interfaces use standard headers and endpoints. MQTT interfaces use User Properties and Last Will and Testament (LWT) messages.

| Attribute | Specification |
|-----------|---------------|
| **Trace ID Source** | [HTTP: X-Request-ID header OR MQTT: trace_id User Property] |
| **Health Check** | [HTTP: GET /health returns 200 OR MQTT: Last Will and Testament (LWT)] |
| **Readiness** | [HTTP: GET /ready OR MQTT: Birth message on connect] |
| **Metrics Endpoint** | [e.g., GET /metrics returns Prometheus-format metrics] |
| **Log Format** | [e.g., JSON structured logging with trace_id field] |

> **NOTE**: For MQTT interfaces, the Last Will and Testament (LWT) message is published by the broker when a client disconnects unexpectedly. The LWT topic and payload shall be defined to enable downstream systems to detect service unavailability.

### 9.5 Configuration and Environment Variables

> **INSTRUCTION**: List all configuration keys or environment variables required to operate this interface. This section enables DevOps teams to deploy and configure the interface correctly.

| Env Variable / Key | Default | Required | Description |
|--------------------|---------|----------|-------------|
| BROKER_HOST | localhost | Yes | MQTT broker address |
| BROKER_PORT | 8883 | No | MQTT broker port (TLS) |
| KEYCLOAK_URL | [None] | Yes | Authentication server URL |
| LOG_LEVEL | INFO | No | Logging verbosity (DEBUG, INFO, WARN, ERROR) |
| [ENV_VAR] | [default] | [Yes/No] | [Description] |

> **NOTE**: Environment variables enable containerised deployment without code modification. Sensitive values (credentials, API keys) shall be injected via secrets management rather than environment variables in production environments.

---

## 10. Requirements Traceability Matrix

> **INSTRUCTION**: Map interface features to D2.2 Software Requirements Specifications. This matrix demonstrates compliance with project requirements and supports EU grant reporting.

| SRS ID | Requirement | Interface Feature | Verification Method |
|--------|-------------|-------------------|---------------------|
| [SRS-X-Y] | [Requirement text] | [Feature/section ref] | [Test/Analysis] |
| [SRS-X-Y] | [Requirement text] | [Feature/section ref] | [Test/Analysis] |

> **REFERENCE**: D2.2 for complete SRS specifications. Common SRS references include: SRS-1-19 (Authentication), SRS-1-20 (Role-based access), SRS-1-22 (Response time), SRS-1-23 (Transport encryption), SRS-1-24 (Availability).

---

## 11. Acceptance Criteria

> **INSTRUCTION**: Define testable acceptance criteria for interface validation. Each criterion shall be measurable and traceable to requirements.

> **NOTE**: Acceptance criteria shall use clear, measurable language. Avoid subjunctive constructions. Correct: "The interface returns HTTP 200 within 2 seconds for 95% of requests." Incorrect: "The interface should respond quickly."

| AC ID | Criterion | Test Method | SRS Ref |
|-------|-----------|-------------|---------|
| AC-01 | [Testable criterion statement] | [Integration/Unit] | [SRS-X-Y] |
| AC-02 | [Testable criterion statement] | [Integration/Unit] | [SRS-X-Y] |

---

## 12. References

> **INSTRUCTION**: List all referenced documents, standards, and specifications. Use consistent citation format.

[1] D2.2 DATA4CIRC Requirements and Specifications

[2] D4.1 Platform Architecture and Open-Source Protocols

[3] IDTA AAS Metamodel Specification (IDTA-01001)

[4] Eclipse Dataspace Connector Documentation

[5] OAuth 2.0 (RFC 6749) and OpenID Connect Core 1.0 (Keycloak implementation)

[6] RFC 9457 Problem Details for HTTP APIs (obsoletes RFC 7807)

[7] W3C ODRL Information Model 2.2

[8] [Add additional references as needed]

---

## 13. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | [Date] | [Author] | Initial draft |
| [X.Y] | [Date] | [Author] | [Description of changes] |

---

## Annex A: Sequence Diagrams

> **INSTRUCTION**: Include sequence diagrams illustrating the interaction patterns described in Section 2. Use PlantUML or similar notation.

[Insert sequence diagram(s) showing message flows between components]

---

## Annex B: Complete API Schema

> **INSTRUCTION**: Include the complete OpenAPI/Swagger specification or JSON Schema definitions.

[Insert OpenAPI Specification (e.g., v3.1.0) or JSON Schema definitions]

---

## Annex C: Test Cases

> **INSTRUCTION**: Include detailed test cases for acceptance criteria validation.

[Insert test case specifications with preconditions, steps, and expected results]

---

## Annex D: Quality Checklist

> **INSTRUCTION**: Complete this checklist before submission to ensure compliance with DATA4CIRC documentation standards.

| Check | Criterion | Section |
|-------|-----------|---------|
| ☐ | Units of measure specified for all numerical fields | Section 6.1 |
| ☐ | Semantic IDs (IRDIs) provided for AAS-compliant fields | Section 6.1 |
| ☐ | Environment variables listed for DevOps deployment | Section 9.5 |
| ☐ | Circuit breaker thresholds defined for resilience | Section 4.2 |
| ☐ | PII fields flagged and retention policies defined | Section 6.3 |
| ☐ | ODRL policies defined for dataspace interfaces | Section 7.4 |
| ☐ | MQTT topics, QoS, and LWT defined for IoT interfaces | Section 5.3, 9.4 |
| ☐ | Error handling appropriate for protocol (RFC 9457 or DLQ) | Section 2.3 |
| ☐ | Health check mechanism defined (HTTP endpoint or MQTT LWT) | Section 9.4 |
| ☐ | Interface dependencies and versioning documented | Section 1.4 |
| ☐ | British English and IEEE style followed throughout | All sections |
| ☐ | No subjunctive mood, personal pronouns, or filler words | All sections |
| ☐ | Abbreviations defined once and listed in Section 3 | Section 3 |
| ☐ | Performance targets use specific numerical values | Section 8 |
