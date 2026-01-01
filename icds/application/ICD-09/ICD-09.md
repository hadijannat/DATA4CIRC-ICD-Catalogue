# ICD-09: DT/DTh Portal <-> DT/DTh Application

**DT/DTh Portal <-> DT/DTh Application**

---

| Attribute | Value |
|-----------|-------|
| **Version** | 1.0 |
| **Date** | 31 December 2025 |
| **Work Package** | WP5 |
| **Author(s)** | FOS (WP5); IDE (WP5) |
| **Provider Owner** | FOS (DT/DTh Application) |
| **Consumer Owner** | FOS; IDE (DT/DTh Portal) |
| **Reviewer** | RWTH Aachen University (WP4) |
| **Status** | Approved |

---

## 1. Interface Overview

### 1.1 Purpose

This Interface Control Document (ICD) specifies the application programming interface (API) between the Digital Twin/Digital Thread (DT/DTh) Portal and the DT/DTh Application in the DATA4CIRC platform. The interface provides secure, role-controlled access to DT/DTh backend capabilities, comprising telemetry retrieval, simulation scenario management, submission of simulated process data, and collaborative modelling functions such as annotation, stakeholder tagging, and version control. The specification aligns with DATA4CIRC requirements, with traceability to the Software Requirements Specification (SRS) items SRS-11-1 to SRS-11-6 for collaboration and quality and compliance access and SRS-1-19 to SRS-1-24 for security, performance, encryption, and availability.

### 1.2 Communicating Components

| Attribute | Component A | Component B |
|-----------|-------------|-------------|
| **Name** | DT/DTh Portal | DT/DTh Application |
| **Role** | Consumer | Provider |
| **Work Package** | WP5 | WP5 |
| **Responsible Partner** | FOS; IDE | FOS |

### 1.3 Architectural Context

The DT/DTh Portal is deployed in the front-end (application and interaction) layer of the DATA4CIRC reference architecture and provides user-facing dashboards, model editing views, and simulation user interfaces. The DT/DTh Application is deployed in the back-end (core services and tools) layer and exposes a Representational State Transfer (REST) API and a WebSocket channel to the portal. The DT/DTh Application integrates a model repository and simulation services for end-to-end traceability across value-chain processes and mediates access to underlying data sources, including time-series telemetry services (ThingsBoard) and persistence services. Identity and access management is provided by the platform Identity and Access Management (IAM) service (Keycloak). Authentication and authorisation use Open Authorisation 2.0 (OAuth 2.0) and OpenID Connect (OIDC) tokens with role-based access control (RBAC).

### 1.4 Interface Dependencies and Lifecycle

| Attribute | Specification |
|-----------|---------------|
| **Prerequisites** | Keycloak IAM service operational; valid JSON Web Key Set (JWKS) endpoint reachable by the DT/DTh Application; network connectivity between DT/DTh Portal and DT/DTh Application via Hypertext Transfer Protocol Secure (HTTPS) on port 443; time synchronisation via Network Time Protocol (NTP) between components; DT/DTh Application connectivity to ThingsBoard telemetry services and backing time-series databases. |
| **Versioning Strategy** | Uniform Resource Identifier (URI)-based versioning with semantic versioning: /api/v{major}. Backwards-compatible changes shall use minor or patch increments documented in the OpenAPI Specification (OpenAPI) info.version field. Breaking changes shall increment the major version. |
| **Deprecation Policy** | Deprecated operations shall be flagged in OpenAPI and shall emit a deprecation warning header. A minimum overlap period of 6 months shall be maintained between a deprecated operation and the successor operation. |
| **Downstream Dependents** | DT/DTh Portal front-end modules (dashboards, model editor, simulation views) and DATA4CIRC unified portal integrations that embed DT/DTh functionality. |

---

## 2. Functional Description

### 2.1 Functional Capabilities

Each functional capability (FC) is assigned an identifier and traced to SRS requirements.

| ID | Capability | Description | SRS Reference |
|----|------------|-------------|---------------|
| FC-01 | Query Digital Twin Telemetry | Retrieve real and simulated time-series telemetry for Digital Twin entities with support for time-range, key, aggregation, and scenario filtering. | NEED-8-7; D2.2 Table 54/55 |
| FC-02 | Submit Simulated Telemetry | Submit simulated process data points associated with a simulation scenario; persist simulated data with explicit provenance labels. | NEED-8-7; D2.2 Table 52/54/55 |
| FC-03 | Manage Simulation Scenarios | Create, list, and manage simulation scenarios (metadata, parameters, and execution status) for scenario comparison and dashboarding. | NEED-8-7 |
| FC-04 | Access Circular Value Chain Models | Provide access to circular value chain models (CVCM) in the model repository, including listing, retrieval, and controlled update operations for collaborative work. | SRS-11-2 |
| FC-05 | Annotate Model Elements | Create, update, list, and delete annotations attached to specific elements of circular value chain models. | SRS-11-1 |
| FC-06 | Create Collaboration Notes with Stakeholder Tagging | Create and retrieve collaboration notes and support tagging of specific users to direct attention to relevant stakeholders. | SRS-11-2; SRS-11-3 |
| FC-07 | Real-Time Collaboration and Version Control | Distribute model updates and collaboration events via WebSocket and maintain version control with a maximum propagation delay of 5 s in real-time mode. | SRS-11-4 |
| FC-08 | Protect Collaboration Data with Backups | Store collaboration artefacts and associated metadata with daily automatic backups and support restoration verification. | SRS-11-5 |
| FC-09 | Provide Quality Control Data and Compliance Reports | Expose query and retrieval operations for quality control datasets and compliance reports relevant to circular value chain operations. | SRS-11-6 |
| FC-10 | Authenticate Requests via OAuth 2.0/OIDC | Validate OAuth 2.0/OpenID Connect access tokens prior to processing requests, aligned with zero-trust security principles. | SRS-1-19 |
| FC-11 | Enforce Role-Based Access Control | Enforce role-based permissions for all protected operations and tenant-scoped data access. | SRS-1-20 |

### 2.2 Interaction Patterns

Interaction patterns are based on synchronous request-response exchanges over HTTPS for query and create, read, update, delete (CRUD) operations and on a persistent WebSocket channel for real-time collaboration updates. The DT/DTh Portal shall obtain an OAuth 2.0/OIDC access token from Keycloak and shall include the token in the Hypertext Transfer Protocol (HTTP) Authorization header for protected operations. The DT/DTh Application shall validate token signature and claims using JSON Web Token (JWT) verification and shall enforce RBAC prior to invoking downstream services. Telemetry queries are executed by the DT/DTh Application against underlying telemetry services (ThingsBoard) and the time-series store; results are returned as JavaScript Object Notation (JSON) arrays with explicit provenance fields that distinguish measured and simulated data. Simulation submissions are executed as POST operations that persist labelled simulation data and return ingestion status. Collaborative model editing uses optimistic concurrency control with entity tags (ETags) for REST updates and uses JavaScript Object Notation Patch (JSON Patch) operations over WebSocket for real-time update propagation. Sequence diagrams are provided in Annex A.

### 2.3 Error Handling

#### 2.3.1 HTTP/REST Error Handling

For HTTP/REST interfaces, error responses shall conform to Request for Comments (RFC) 9457 (Problem Details for HTTP APIs).

| HTTP Status | Condition | Recovery Action |
|-------------|-----------|-----------------|
| 400 Bad Request | Request payload or query parameters fail schema validation. | Correct the request to satisfy the OpenAPI schema and retry. |
| 401 Unauthorized | Access token missing, expired, or invalid. | Acquire a valid OAuth 2.0/OIDC access token and retry. |
| 403 Forbidden | Authenticated principal lacks required role or tenant scope. | Request appropriate role assignment or tenant access and retry. |
| 404 Not Found | Requested resource identifier does not exist or is not visible under tenant scope. | Verify identifiers and access scope; retry with valid identifiers. |
| 409 Conflict | Version conflict detected (ETag mismatch) during update. | Refresh the resource, resolve conflicts, and retry with updated ETag in If-Match. |
| 413 Payload Too Large | Request entity exceeds maximum payload size. | Reduce payload size or split the submission into multiple requests. |
| 429 Too Many Requests | Rate limit exceeded. | Apply exponential backoff and retry after the server-provided Retry-After interval. |
| 500 Internal Server Error | Unhandled internal processing error. | Retry with idempotency key where applicable; contact service operator if errors persist. |
| 503 Service Unavailable | Upstream dependency unavailable (for example, ThingsBoard) or service in maintenance mode. | Retry according to retry policy; apply circuit breaker to prevent request storms. |

#### 2.3.2 IoT/Async Error Handling

Internet of Things (IoT) and asynchronous interfaces use Dead Letter Queue (DLQ) strategies when applicable. ICD-09 uses HTTP/REST and WebSocket only.

| Attribute | Specification |
|-----------|---------------|
| **Error Topic** | Not applicable. ICD-09 uses HTTP/REST and WebSocket. Error handling uses RFC 9457 problem details and WebSocket close codes for connection-level errors. |
| **DLQ Strategy** | Not applicable. ICD-09 uses HTTP/REST and WebSocket. Error handling uses RFC 9457 problem details and WebSocket close codes for connection-level errors. |
| **Error Payload Schema** | Not applicable. ICD-09 uses HTTP/REST and WebSocket. Error handling uses RFC 9457 problem details and WebSocket close codes for connection-level errors. |
| **Retry Policy** | Not applicable. ICD-09 uses HTTP/REST and WebSocket. Error handling uses RFC 9457 problem details and WebSocket close codes for connection-level errors. |

---

## 3. Abbreviations

| Abbreviation | Definition |
|--------------|------------|
| AC | Acceptance Criterion |
| API | Application Programming Interface |
| CORS | Cross-Origin Resource Sharing |
| CRUD | Create, Read, Update, Delete |
| CVCM | Circular Value Chain Model |
| DLQ | Dead Letter Queue |
| DMZ | Demilitarised Zone |
| DT/DTh | Digital Twin/Digital Thread |
| ETag | Entity Tag |
| FC | Functional Capability |
| GDPR | General Data Protection Regulation |
| HTTP | Hypertext Transfer Protocol |
| HTTPS | Hypertext Transfer Protocol Secure |
| IAM | Identity and Access Management |
| ICD | Interface Control Document |
| IoT | Internet of Things |
| IRDI | International Registration Data Identifier |
| JSON | JavaScript Object Notation |
| JSON Patch | JavaScript Object Notation Patch |
| JWT | JSON Web Token |
| JWKS | JSON Web Key Set |
| MQTT | Message Queuing Telemetry Transport |
| NTP | Network Time Protocol |
| OAuth 2.0 | Open Authorisation 2.0 |
| ODRL | Open Digital Rights Language |
| OIDC | OpenID Connect |
| OpenAPI | OpenAPI Specification |
| OTel | OpenTelemetry |
| OTLP | OpenTelemetry Protocol |
| P95 | 95th percentile |
| PII | Personally Identifiable Information |
| PKCE | Proof Key for Code Exchange |
| RBAC | Role-Based Access Control |
| RFC | Request for Comments |
| REST | Representational State Transfer |
| SRS | Software Requirements Specification |
| TLS | Transport Layer Security |
| TTL | Time-to-live |
| UCUM | Unified Code for Units of Measure |
| URI | Uniform Resource Identifier |
| URL | Uniform Resource Locator |
| URN | Uniform Resource Name |
| UTC | Coordinated Universal Time |
| UUID | Universally Unique Identifier |
| W3C | World Wide Web Consortium |

---

## 4. Communication Protocol

### 4.1 Protocol Stack

| Layer | Protocol | Specification |
|-------|----------|---------------|
| Application | HTTP/REST; WebSocket (real-time collaboration channel) | OpenAPI Specification 3.1.0; RFC 6455 |
| Security | OAuth 2.0 / OpenID Connect (JWT bearer access tokens); Transport Layer Security (TLS) 1.3 | RFC 6749; OpenID Connect Core 1.0; RFC 8446 |
| Transport | HTTPS (HTTP/1.1 or HTTP/2); WebSocket over TLS | RFC 9110/9112; RFC 6455 |
| Serialisation | JSON; JSON Patch; Problem Details | RFC 8259; RFC 6902; RFC 9457 |

### 4.2 Connection Parameters

Connection parameters use the base Uniform Resource Locator (URL) for the DT/DTh Application and assume deployment behind a demilitarised zone (DMZ) ingress controller.

| Parameter | Value |
|-----------|-------|
| **Base URL / Broker** | https://&lt;dt-dth-api-hostname&gt;/api/v1 |
| **Port** | 443 |
| **Network Zone** | DMZ / ingress controller (HTTPS) |
| **Connection Timeout** | 10 s |
| **Read Timeout** | 60 s |
| **Retry Policy** | 3 retries with exponential backoff (1 s, 2 s, 4 s); idempotency keys for POST ingestion |
| **Circuit Breaker** | Open after 5 consecutive upstream failures; half-open after 30 s; close after 3 consecutive successes |
| **Firewall Rules** | Allow inbound port 443 from DT/DTh Portal network segment to DT/DTh Application ingress; deny all other inbound traffic |

---

## 5. API Specification

### 5.1 Endpoint Definitions

Identifiers use the Universally Unique Identifier (UUID) format unless stated otherwise. Timestamps are expressed as Unix epoch milliseconds in Coordinated Universal Time (UTC).

#### 5.1.1 Query Digital Twin Telemetry

| Attribute | Value |
|-----------|-------|
| **Method** | GET |
| **Path** | /api/v1/dt/entities/{entityId}/telemetry |
| **Purpose** | Retrieve time-series telemetry for a Digital Twin entity. The operation supports measured and simulated data retrieval with optional aggregation and scenario filtering. |
| **Authentication** | Bearer access token (OAuth 2.0/OIDC JWT) |

**Path Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| entityId | string (UUID) | Yes | Digital Twin entity identifier (device or asset identifier). |

**Query Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| keys | string | Yes | Comma-separated list of telemetry keys to retrieve. |
| startTs | integer (int64) | Yes | Start timestamp (Unix epoch milliseconds, UTC). |
| endTs | integer (int64) | Yes | End timestamp (Unix epoch milliseconds, UTC). |
| aggregation | string | No | Aggregation mode: NONE, AVG, MIN, MAX, SUM. |
| intervalMs | integer (int64) | No | Aggregation interval in milliseconds. Required when aggregation is not NONE. |
| source | string | No | Data source filter: REAL, SIMULATED, BOTH. Default: BOTH. |
| scenarioId | string (UUID) | No | Simulation scenario identifier. Required when source is SIMULATED. |
| limit | integer (int32) | No | Maximum number of data points per key. Default: 10000. |

#### 5.1.2 Submit Simulated Telemetry

| Attribute | Value |
|-----------|-------|
| **Method** | POST |
| **Path** | /api/v1/dt/entities/{entityId}/telemetry/simulated |
| **Purpose** | Ingest simulated telemetry data points for a Digital Twin entity under a simulation scenario. The DT/DTh Application shall persist the submitted data with explicit provenance labels and scenario binding. |
| **Authentication** | Bearer access token (OAuth 2.0/OIDC JWT) |

**Path Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| entityId | string (UUID) | Yes | Digital Twin entity identifier. |

**Query Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| scenarioId | string (UUID) | Yes | Simulation scenario identifier used to label and group simulated data. |

**Header Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| Idempotency-Key | string | No | Idempotency key for safe retries of ingestion requests. |

**Request Body:** Array of Telemetry Data Point objects (Section 6.1.1).

#### 5.1.3 Create Simulation Scenario

| Attribute | Value |
|-----------|-------|
| **Method** | POST |
| **Path** | /api/v1/simulations |
| **Purpose** | Create a simulation scenario and allocate a scenario identifier. Scenario metadata defines the target entity, time horizon, and parameterisation for scenario comparison. |
| **Authentication** | Bearer access token (OAuth 2.0/OIDC JWT) |

**Header Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| Idempotency-Key | string | No | Idempotency key for safe retries of scenario creation. |

**Request Body:** CreateScenarioRequest object (Annex B).

#### 5.1.4 List Circular Value Chain Models

| Attribute | Value |
|-----------|-------|
| **Method** | GET |
| **Path** | /api/v1/models |
| **Purpose** | List circular value chain models visible under tenant scope. The operation supports pagination and filtering by model type. |
| **Authentication** | Bearer access token (OAuth 2.0/OIDC JWT) |

**Query Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| page | integer (int32) | No | Page index (0-based). Default: 0. |
| size | integer (int32) | No | Page size. Default: 50. Maximum: 500. |
| modelType | string | No | Model type filter (for example, CVCM). |
| sort | string | No | Sort expression (for example, updatedAt,desc). |

#### 5.1.5 Create Model Element Annotation

| Attribute | Value |
|-----------|-------|
| **Method** | POST |
| **Path** | /api/v1/models/{modelId}/elements/{elementId}/annotations |
| **Purpose** | Create an annotation attached to a specific model element. Annotation content supports structured types and optional stakeholder mentions. |
| **Authentication** | Bearer access token (OAuth 2.0/OIDC JWT) |

**Path Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| modelId | string (UUID) | Yes | Model identifier. |
| elementId | string | Yes | Model element identifier (stable identifier within the model). |

**Request Body:** CreateAnnotationRequest object (Annex B).

#### 5.1.6 Create Collaboration Note with Stakeholder Tagging

| Attribute | Value |
|-----------|-------|
| **Method** | POST |
| **Path** | /api/v1/models/{modelId}/collaboration/notes |
| **Purpose** | Create a collaboration note linked to a model and distribute the note to subscribed participants. Mentions shall be represented as explicit user identifiers. |
| **Authentication** | Bearer access token (OAuth 2.0/OIDC JWT) |

**Path Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| modelId | string (UUID) | Yes | Model identifier. |

**Request Body:** CreateNoteRequest object (Annex B).

#### 5.1.7 Request WebSocket Ticket

| Attribute | Value |
|-----------|-------|
| **Method** | POST |
| **Path** | /api/v1/models/{modelId}/realtime/tickets |
| **Purpose** | Issue a short-lived WebSocket ticket for browser-based clients. The ticket binds a principal to a model-specific real-time session and limits token exposure in WebSocket URLs. |
| **Authentication** | Bearer access token (OAuth 2.0/OIDC JWT) |

**Path Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| modelId | string (UUID) | Yes | Model identifier. |

#### 5.1.8 Real-Time Collaboration WebSocket Channel

| Attribute | Value |
|-----------|-------|
| **Method** | GET (WebSocket Upgrade) |
| **Path** | /api/v1/models/{modelId}/realtime?ticket={ticket} |
| **Purpose** | Establish a WebSocket connection for real-time distribution of model updates and collaboration events. Update propagation delay shall not exceed 5 s. |
| **Authentication** | WebSocket ticket (query parameter) obtained from Section 5.1.7 |

**Path Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| modelId | string (UUID) | Yes | Model identifier. |

**Query Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| ticket | string | Yes | Short-lived WebSocket ticket issued by Section 5.1.7. |

### 5.2 Request and Response Examples

**Request Example**

```
POST /api/v1/dt/entities/2f9c2a4d-7c14-4c0e-9f3d-4b1b3a2c9f1a/telemetry/simulated?scenarioId=8b2a4b51-2d3b-4c8a-9f3a-5b4a1f2e9c7d HTTP/1.1
Host: <dt-dth-api-hostname>
Authorization: Bearer <JWT>
Content-Type: application/json
Accept: application/json
Idempotency-Key: 3a0b8e2a-7e8d-4d5f-9a5f-0f2d0d7a4a90
X-Request-ID: 6f1c9d2e-1b4a-4c3d-9e2f-3b1a0c9d8e7f

[
  {
    "timestamp": 1735603200000,
    "key": "energy_consumption",
    "value": 12.4,
    "unit": "kW.h",
    "source": "SIMULATED",
    "scenarioId": "8b2a4b51-2d3b-4c8a-9f3a-5b4a1f2e9c7d"
  },
  {
    "timestamp": 1735603260000,
    "key": "energy_consumption",
    "value": 12.1,
    "unit": "kW.h",
    "source": "SIMULATED",
    "scenarioId": "8b2a4b51-2d3b-4c8a-9f3a-5b4a1f2e9c7d"
  }
]
```

**Response Example (202 Accepted)**

```
HTTP/1.1 202 Accepted
Content-Type: application/json
X-Request-ID: 6f1c9d2e-1b4a-4c3d-9e2f-3b1a0c9d8e7f

{
  "scenarioId": "8b2a4b51-2d3b-4c8a-9f3a-5b4a1f2e9c7d",
  "entityId": "2f9c2a4d-7c14-4c0e-9f3d-4b1b3a2c9f1a",
  "acceptedPoints": 2,
  "rejectedPoints": 0,
  "ingestionStatus": "QUEUED",
  "timestamp": 1735603300123
}
```

### 5.3 Event and Message Specifications (Asynchronous/MQTT)

Message Queuing Telemetry Transport (MQTT) messaging is not used for ICD-09.

#### 5.3.1 Not applicable

| Attribute | Specification |
|-----------|---------------|
| **Topic/Channel** | Not applicable. ICD-09 uses HTTPS/REST and WebSocket rather than MQTT. |
| **Direction** | Not applicable. ICD-09 uses HTTPS/REST and WebSocket rather than MQTT. |
| **Quality of Service Level** | Not applicable. ICD-09 uses HTTPS/REST and WebSocket rather than MQTT. |
| **Trigger Condition** | Not applicable. ICD-09 uses HTTPS/REST and WebSocket rather than MQTT. |
| **Payload Format** | Not applicable. ICD-09 uses HTTPS/REST and WebSocket rather than MQTT. |
| **Retention** | Not applicable. ICD-09 uses HTTPS/REST and WebSocket rather than MQTT. |

---

## 6. Data Structures

### 6.1 Data Model

Timestamps are expressed as Unix epoch milliseconds in UTC. Units for numeric telemetry values use the Unified Code for Units of Measure (UCUM). Semantic identifiers are listed as International Registration Data Identifiers (IRDIs) within the DATA4CIRC Uniform Resource Name (URN) namespace.

#### 6.1.1 Telemetry Data Point

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|--------------------|-----|-------------|
| timestamp | integer (int64) | Unix epoch milliseconds (UTC) | urn:data4circ:dt-dth:telemetry:timestamp | Yes | Sampling timestamp for the telemetry value. |
| key | string | N/A | urn:data4circ:dt-dth:telemetry:key | Yes | Telemetry key name as stored in the telemetry service. |
| value | number or string or boolean or object | N/A | urn:data4circ:dt-dth:telemetry:value | Yes | Telemetry value. Numeric values shall be accompanied by a unit. |
| unit | string | UCUM | urn:data4circ:dt-dth:telemetry:unit | Conditional | Unit of measure for numeric values in UCUM representation. |
| source | string | Enum: REAL, SIMULATED | urn:data4circ:dt-dth:telemetry:source | Yes | Provenance indicator distinguishing measured and simulated values. |
| scenarioId | string (UUID) | N/A | urn:data4circ:dt-dth:simulation:scenarioId | Conditional | Scenario identifier for simulated values. Mandatory when source is SIMULATED. |
| quality | string | Enum: RAW, VALIDATED, SIMULATED | urn:data4circ:dt-dth:telemetry:quality | No | Quality indicator assigned by the ingestion pipeline. |
| attributes | object | JSON object | urn:data4circ:dt-dth:telemetry:attributes | No | Additional metadata such as sensor identifier, ingestion batch identifier, or processing flags. |

#### 6.1.2 Simulation Scenario

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|--------------------|-----|-------------|
| scenarioId | string (UUID) | N/A | urn:data4circ:dt-dth:simulation:scenarioId | Yes | Unique identifier of the simulation scenario. |
| name | string | N/A | urn:data4circ:dt-dth:simulation:name | Yes | Human-readable scenario name. |
| description | string | N/A | urn:data4circ:dt-dth:simulation:description | No | Scenario description. |
| entityId | string (UUID) | N/A | urn:data4circ:dt-dth:dt:entityId | Yes | Target Digital Twin entity identifier for the scenario. |
| createdBy | string | N/A | urn:data4circ:dt-dth:iam:principal | Yes | Identifier of the creating principal (subject claim). |
| createdAt | integer (int64) | Unix epoch milliseconds (UTC) | urn:data4circ:dt-dth:time:createdAt | Yes | Creation timestamp. |
| status | string | Enum: CREATED, QUEUED, RUNNING, COMPLETED, FAILED | urn:data4circ:dt-dth:simulation:status | Yes | Execution status of the scenario. |
| parameters | object | JSON object | urn:data4circ:dt-dth:simulation:parameters | No | Scenario parameterisation (key-value structure). |
| timeRange | object | startTs and endTs in epoch ms | urn:data4circ:dt-dth:time:range | No | Scenario time horizon. |

#### 6.1.3 Circular Value Chain Model Metadata

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|--------------------|-----|-------------|
| modelId | string (UUID) | N/A | urn:data4circ:dt-dth:model:modelId | Yes | Unique identifier of the circular value chain model. |
| name | string | N/A | urn:data4circ:dt-dth:model:name | Yes | Model name. |
| description | string | N/A | urn:data4circ:dt-dth:model:description | No | Model description. |
| modelType | string | Enum: CVCM | urn:data4circ:dt-dth:model:type | Yes | Model type classification. |
| format | string | Enum: CVCM_JSON | urn:data4circ:dt-dth:model:format | Yes | Serialisation format identifier. |
| version | integer (int32) | N/A | urn:data4circ:dt-dth:model:version | Yes | Monotonic version number. |
| etag | string | HTTP ETag | urn:data4circ:dt-dth:http:etag | Yes | Entity tag representing the current version for optimistic concurrency control. |
| updatedAt | integer (int64) | Unix epoch milliseconds (UTC) | urn:data4circ:dt-dth:time:updatedAt | Yes | Last update timestamp. |
| updatedBy | string | N/A | urn:data4circ:dt-dth:iam:principal | Yes | Identifier of the updating principal (subject claim). |

#### 6.1.4 Model Annotation

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|--------------------|-----|-------------|
| annotationId | string (UUID) | N/A | urn:data4circ:dt-dth:collab:annotationId | Yes | Unique identifier of the annotation. |
| modelId | string (UUID) | N/A | urn:data4circ:dt-dth:model:modelId | Yes | Model identifier. |
| elementId | string | N/A | urn:data4circ:dt-dth:model:elementId | Yes | Identifier of the model element to which the annotation is attached. |
| authorId | string | N/A | urn:data4circ:dt-dth:iam:principal | Yes | Identifier of the authoring principal. |
| createdAt | integer (int64) | Unix epoch milliseconds (UTC) | urn:data4circ:dt-dth:time:createdAt | Yes | Creation timestamp. |
| type | string | Enum: COMMENT, ISSUE, DECISION, RISK | urn:data4circ:dt-dth:collab:type | Yes | Annotation classification. |
| text | string | N/A | urn:data4circ:dt-dth:collab:text | Yes | Annotation text. |
| tags | array[string] | N/A | urn:data4circ:dt-dth:collab:tags | No | Tag set used for filtering. |
| mentionUserIds | array[string (UUID)] | N/A | urn:data4circ:dt-dth:collab:mentions | No | Mentioned user identifiers. |
| resolved | boolean | N/A | urn:data4circ:dt-dth:collab:resolved | No | Resolution flag. |

#### 6.1.5 Collaboration Note

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|--------------------|-----|-------------|
| noteId | string (UUID) | N/A | urn:data4circ:dt-dth:collab:noteId | Yes | Unique identifier of the collaboration note. |
| modelId | string (UUID) | N/A | urn:data4circ:dt-dth:model:modelId | Yes | Model identifier. |
| authorId | string | N/A | urn:data4circ:dt-dth:iam:principal | Yes | Identifier of the authoring principal. |
| createdAt | integer (int64) | Unix epoch milliseconds (UTC) | urn:data4circ:dt-dth:time:createdAt | Yes | Creation timestamp. |
| text | string | N/A | urn:data4circ:dt-dth:collab:text | Yes | Collaboration note text. |
| mentionUserIds | array[string (UUID)] | N/A | urn:data4circ:dt-dth:collab:mentions | No | Mentioned user identifiers. |
| relatedElementIds | array[string] | N/A | urn:data4circ:dt-dth:model:elementId | No | Referenced model element identifiers. |
| version | integer (int32) | N/A | urn:data4circ:dt-dth:collab:version | Yes | Version number for note updates. |

#### 6.1.6 Problem Details

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|--------------------|-----|-------------|
| type | string | URI | urn:data4circ:dt-dth:http:error:type | Yes | Problem type URI. |
| title | string | N/A | urn:data4circ:dt-dth:http:error:title | Yes | Short, human-readable summary. |
| status | integer (int32) | HTTP status code | urn:data4circ:dt-dth:http:error:status | Yes | HTTP status code. |
| detail | string | N/A | urn:data4circ:dt-dth:http:error:detail | No | Human-readable explanation specific to the occurrence. |
| instance | string | URI | urn:data4circ:dt-dth:http:error:instance | No | URI reference identifying the specific occurrence. |
| errorCode | string | N/A | urn:data4circ:dt-dth:http:error:code | No | Stable, service-defined error code. |
| traceId | string | World Wide Web Consortium (W3C) Trace Context traceparent | urn:data4circ:dt-dth:obs:traceId | No | Trace correlation identifier. |

### 6.2 Semantic Mappings

Semantic interoperability for ICD-09 is ensured through explicit typing, unit normalisation, and stable identifier schemes. Time instants are represented as Unix epoch milliseconds (UTC) to align with telemetry services. Units for numeric telemetry values are represented in UCUM. Identifiers are represented as UUIDs for tenant-scoped resources (models, scenarios, notes) and as stable string identifiers for model elements. Semantic identifiers in data structure tables use DATA4CIRC URN namespaces to enable consistent cross-interface mapping.

### 6.3 Data Governance and Compliance

Data governance aligns with General Data Protection Regulation (GDPR) obligations and marks fields that contain personally identifiable information (PII).

| Data Entity | PII (Y/N) | Classification | Retention Period |
|-------------|-----------|----------------|------------------|
| Telemetry time-series (measured) | N | Confidential | Configurable; default 24 months |
| Telemetry time-series (simulated) | N | Confidential | Configurable; default 24 months |
| Simulation scenario metadata | N | Internal | Configurable; default 24 months |
| Circular value chain model artefacts | N | Confidential | Configurable; default 36 months |
| Annotations and collaboration notes (text, tags) | Y (potential) | Confidential | Configurable; default 36 months |
| Mention and author identifiers (subject identifiers) | Y (pseudonymous) | Confidential | Configurable; default 36 months |
| Audit logs (access and change events) | Y (pseudonymous) | Confidential | Configurable; default 12 months |
| Backup archives | N | Confidential | Configurable; default 30 days |

---

## 7. Security Requirements

### 7.1 Authentication

The WebSocket ticket time-to-live (TTL) is configurable.

| Mechanism | OAuth 2.0 / OpenID Connect (Authorisation Code + Proof Key for Code Exchange (PKCE)); JWT bearer access tokens |
|-----------|-------------------------------------------------------------------------------------------------------------|
| Identity Provider | Keycloak (DATA4CIRC IAM) |
| Token Type | JWT (RS256 signed access token); JWKS-based verification |
| Token Lifetime | Access token: 3600 s (configurable); WebSocket ticket: 30 s (configurable) |

### 7.2 Authorisation

| Operation | Required Role | SRS Reference |
|-----------|---------------|---------------|
| GET /dt/entities/* | dt_viewer | SRS-1-20 |
| GET /dt/entities/{entityId}/telemetry | dt_viewer | SRS-1-20 |
| POST /dt/entities/{entityId}/telemetry/simulated | dt_simulator | SRS-1-20; NEED-8-7 |
| POST /simulations | dt_simulator | SRS-1-20; NEED-8-7 |
| GET /models | dth_viewer | SRS-1-20 |
| POST /models/{modelId}/elements/*/annotations | dth_editor | SRS-1-20; SRS-11-1 |
| POST /models/{modelId}/collaboration/notes | dth_editor | SRS-1-20; SRS-11-2; SRS-11-3 |
| WebSocket /models/{modelId}/realtime | dth_editor | SRS-1-20; SRS-11-4 |
| GET /compliance/reports* | compliance_reader | SRS-1-20; SRS-11-6 |
| GET /admin/backups/status | admin | SRS-1-20; SRS-11-5 |

### 7.3 Transport Security

| Attribute | Specification |
|-----------|---------------|
| **TLS Version** | TLS 1.3 |
| **Certificate Validation** | X.509 server certificate validation shall be enforced, including hostname verification and certificate-chain validation anchored in a trusted certificate authority. |
| **Cipher Suites** | TLS 1.3 cipher suites shall be enabled (TLS_AES_128_GCM_SHA256 and TLS_AES_256_GCM_SHA384). Legacy and NULL cipher suites shall be disabled. |

### 7.4 Usage Control (ODRL Policies)

Open Digital Rights Language (ODRL) policies are not applicable. ICD-09 specifies intra-platform access between the DT/DTh Portal and DT/DTh Application. Data usage control and contract-based policy enforcement are implemented via DATA4CIRC dataspace interfaces and are outside the scope of ICD-09.

| Policy Element | Specification |
|----------------|---------------|
| Permission | Not applicable. ICD-09 specifies intra-platform access between the DT/DTh Portal and DT/DTh Application. Data usage control and contract-based policy enforcement are implemented via DATA4CIRC dataspace interfaces and are outside the scope of ICD-09. |
| Constraint | Not applicable. ICD-09 specifies intra-platform access between the DT/DTh Portal and DT/DTh Application. Data usage control and contract-based policy enforcement are implemented via DATA4CIRC dataspace interfaces and are outside the scope of ICD-09. |
| Duty | Not applicable. ICD-09 specifies intra-platform access between the DT/DTh Portal and DT/DTh Application. Data usage control and contract-based policy enforcement are implemented via DATA4CIRC dataspace interfaces and are outside the scope of ICD-09. |
| Prohibition | Not applicable. ICD-09 specifies intra-platform access between the DT/DTh Portal and DT/DTh Application. Data usage control and contract-based policy enforcement are implemented via DATA4CIRC dataspace interfaces and are outside the scope of ICD-09. |

---

## 8. Performance Requirements

Performance targets use the 95th percentile (P95) for response time measurements.

| Metric | Target | SRS Reference |
|--------|--------|---------------|
| API response time (P95) - GET telemetry | <= 3 s | SRS-1-22 |
| API response time (P95) - CRUD operations | <= 3 s | SRS-1-22 |
| Real-time update propagation delay | <= 5 s | SRS-11-4 |
| Service availability (monthly) | >= 99.5% | SRS-1-24 |
| Transport encryption | TLS 1.3 | SRS-1-23 |

---

## 9. Implementation Guidelines

### 9.1 Client Implementation Example

**TypeScript (DT/DTh Portal) Example**

```ts
// DT/DTh Portal client-side example (TypeScript)

type TelemetryValue = number | string | boolean | Record<string, unknown>;

interface TelemetryPoint {
  timestamp: number; // epoch ms (UTC)
  key: string;
  value: TelemetryValue;
  unit?: string;     // UCUM
  source: "REAL" | "SIMULATED";
  scenarioId?: string;
}

interface TelemetryQueryResult {
  entityId: string;
  keys: string[];
  points: TelemetryPoint[];
}

export async function queryTelemetry(
  apiBaseUrl: string,
  token: string,
  entityId: string,
  keys: string[],
  startTs: number,
  endTs: number
): Promise<TelemetryQueryResult> {
  const url = new URL(`${apiBaseUrl}/dt/entities/${entityId}/telemetry`);
  url.searchParams.set("keys", keys.join(","));
  url.searchParams.set("startTs", `${startTs}`);
  url.searchParams.set("endTs", `${endTs}`);
  url.searchParams.set("source", "BOTH");

  const response = await fetch(url.toString(), {
    method: "GET",
    headers: {
      "Authorization": `Bearer ${token}`,
      "Accept": "application/json",
      "X-Request-ID": crypto.randomUUID()
    }
  });

  if (!response.ok) {
    // RFC 9457 Problem Details payload expected
    throw await response.json();
  }
  return await response.json();
}

export async function openRealtimeChannel(
  apiBaseUrl: string,
  token: string,
  modelId: string
): Promise<WebSocket> {
  const ticketResponse = await fetch(`${apiBaseUrl}/models/${modelId}/realtime/tickets`, {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${token}`,
      "Accept": "application/json",
      "X-Request-ID": crypto.randomUUID()
    }
  });

  if (!ticketResponse.ok) {
    throw await ticketResponse.json();
  }

  const { ticket } = await ticketResponse.json();
  const wsBaseUrl = apiBaseUrl.replace(/^https:/, "wss:");
  const wsUrl = `${wsBaseUrl}/models/${modelId}/realtime?ticket=${encodeURIComponent(ticket)}`;

  const ws = new WebSocket(wsUrl, "dt-dth.v1");
  return ws;
}
```

### 9.2 Server Implementation Example

**Python (DT/DTh Application - FastAPI) Example**

```python
# DT/DTh Application server-side example (Python, FastAPI)

from __future__ import annotations

import time
from typing import Any, Optional, Union
from uuid import UUID, uuid4

from fastapi import Depends, FastAPI, Header, Query, WebSocket, WebSocketDisconnect
from pydantic import BaseModel, Field

app = FastAPI(title="DT/DTh Application API", version="1.0.0")

class TelemetryPoint(BaseModel):
    timestamp: int = Field(..., description="Unix epoch milliseconds (UTC)")
    key: str
    value: Union[float, int, str, bool, dict[str, Any]]
    unit: Optional[str] = Field(None, description="UCUM unit for numeric values")
    source: str = Field(..., pattern="^(REAL|SIMULATED)$")
    scenarioId: Optional[UUID] = None

class TicketResponse(BaseModel):
    ticket: str
    expiresAt: int

def validate_access_token() -> dict[str, Any]:
    # JWT validation via JWKS (Keycloak) and RBAC enforcement shall be implemented in this dependency.
    return {"sub": "principal-id", "roles": ["dth_editor"]}

@app.get("/api/v1/dt/entities/{entityId}/telemetry")
async def get_telemetry(
    entityId: UUID,
    keys: str = Query(..., description="Comma-separated telemetry keys"),
    startTs: int = Query(..., ge=0),
    endTs: int = Query(..., ge=0),
    source: str = Query("BOTH", pattern="^(REAL|SIMULATED|BOTH)$"),
    aggregation: str = Query("NONE", pattern="^(NONE|AVG|MIN|MAX|SUM)$"),
    intervalMs: Optional[int] = Query(None, ge=1),
    scenarioId: Optional[UUID] = Query(None),
    principal: dict[str, Any] = Depends(validate_access_token),
):
    # Telemetry retrieval from ThingsBoard and time-series store shall be implemented.
    return {"entityId": str(entityId), "keys": keys.split(","), "points": []}

@app.post("/api/v1/models/{modelId}/realtime/tickets", response_model=TicketResponse)
async def issue_ws_ticket(
    modelId: UUID,
    principal: dict[str, Any] = Depends(validate_access_token),
    x_request_id: Optional[str] = Header(None, alias="X-Request-ID"),
):
    # Ticket storage shall use a server-side cache (for example, Redis) with a TTL.
    ticket = uuid4().hex
    expires_at = int(time.time() * 1000) + 30_000
    return TicketResponse(ticket=ticket, expiresAt=expires_at)

@app.websocket("/api/v1/models/{modelId}/realtime")
async def realtime_ws(modelId: UUID, websocket: WebSocket, ticket: str):
    # Ticket verification and subscription authorisation shall be implemented.
    await websocket.accept(subprotocol="dt-dth.v1")
    try:
        while True:
            _msg = await websocket.receive_text()
            # Message handling and JSON Patch distribution shall be implemented.
    except WebSocketDisconnect:
        return
```

**Java (DT/DTh Application - Spring Boot) Example**

```java
// DT/DTh Application server-side example (Java, Spring Boot)

package eu.data4circ.dtdth.api;

import java.util.List;
import java.util.Map;
import java.util.UUID;

import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1")
public class TelemetryController {

    @GetMapping("/dt/entities/{entityId}/telemetry")
    @PreAuthorize("hasAuthority('dt_viewer')")
    public ResponseEntity<Map<String, Object>> getTelemetry(
            @PathVariable UUID entityId,
            @RequestParam String keys,
            @RequestParam long startTs,
            @RequestParam long endTs,
            @RequestParam(defaultValue = "BOTH") String source,
            @RequestParam(required = false) UUID scenarioId) {

        // Telemetry retrieval from ThingsBoard and time-series store shall be implemented.
        return ResponseEntity.ok(Map.of(
                "entityId", entityId.toString(),
                "keys", List.of(keys.split(",")),
                "points", List.of()
        ));
    }

    @PostMapping("/dt/entities/{entityId}/telemetry/simulated")
    @PreAuthorize("hasAuthority('dt_simulator')")
    public ResponseEntity<Map<String, Object>> submitSimulatedTelemetry(
            @PathVariable UUID entityId,
            @RequestParam UUID scenarioId,
            @RequestHeader(value = "Idempotency-Key", required = false) String idempotencyKey,
            @RequestBody List<Map<String, Object>> points) {

        // Schema validation and ingestion to time-series store shall be implemented.
        return ResponseEntity.accepted().body(Map.of(
                "scenarioId", scenarioId.toString(),
                "entityId", entityId.toString(),
                "acceptedPoints", points.size(),
                "rejectedPoints", 0,
                "ingestionStatus", "STORED"
        ));
    }
}
```

### 9.3 Deployment Configuration

The DT/DTh Application shall be deployed behind an ingress controller with TLS termination. Configuration shall be provided through environment variables and Kubernetes secrets. The DT/DTh Portal shall be configured with the API base URL and Keycloak realm metadata.

```yaml
# Helm values.yaml snippet (DT/DTh Application)

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: <dt-dth-api-hostname>
      paths:
        - path: /api/v1
          pathType: Prefix
  tls:
    - secretName: dt-dth-tls
      hosts:
        - <dt-dth-api-hostname>

env:
  KEYCLOAK_URL: "https://<keycloak-hostname>"
  KEYCLOAK_REALM: "data4circ"
  KEYCLOAK_AUDIENCE: "dt-dth-api"
  JWKS_URL: "https://<keycloak-hostname>/realms/data4circ/protocol/openid-connect/certs"
  THINGSBOARD_BASE_URL: "https://<thingsboard-hostname>"
  THINGSBOARD_SERVICE_TOKEN: "<k8s-secret>"
  CORS_ALLOWED_ORIGINS: "https://<dt-dth-portal-hostname>,https://<data4circ-portal-hostname>"
  API_RATE_LIMIT_RPM: "1000"
  LOG_LEVEL: "INFO"
  OTEL_EXPORTER_OTLP_ENDPOINT: "http://otel-collector:4317"
```

### 9.4 Observability and Tracing

OpenTelemetry (OTel) traces and metrics are exported through the OpenTelemetry Protocol (OTLP) endpoint.

| Attribute | Specification |
|-----------|---------------|
| Trace ID Source | W3C Trace Context traceparent request header; generated when absent. |
| Health Check | HTTP: GET /api/v1/health returns 200 when the service process is alive. |
| Readiness | HTTP: GET /api/v1/ready returns 200 when upstream dependencies are reachable. |
| Metrics Endpoint | GET /api/v1/metrics (restricted to admin or service accounts). |
| Log Format | Structured JSON logging with fields: timestamp, level, message, traceId, requestId (X-Request-ID), principal (sub), tenantId, and operation. |

### 9.5 Configuration and Environment Variables

Cross-Origin Resource Sharing (CORS) origins are configured through environment variables.

| Env Variable / Key | Default | Required | Description |
|--------------------|---------|----------|-------------|
| VITE_DT_DTH_API_BASE_URL | - | Yes | DT/DTh Application base URL used by the DT/DTh Portal. |
| VITE_KEYCLOAK_URL | - | Yes | Keycloak base URL used by the DT/DTh Portal OIDC client. |
| VITE_KEYCLOAK_REALM | data4circ | Yes | Keycloak realm name. |
| VITE_KEYCLOAK_CLIENT_ID | dt-dth-portal | Yes | Keycloak client identifier for the portal. |
| KEYCLOAK_URL | - | Yes | Keycloak base URL used by the DT/DTh Application for OIDC metadata and JWKS retrieval. |
| KEYCLOAK_REALM | data4circ | Yes | Keycloak realm name. |
| KEYCLOAK_AUDIENCE | dt-dth-api | Yes | Expected audience claim for access tokens. |
| JWKS_URL | - | Yes | JWKS URL for JWT signature verification. |
| THINGSBOARD_BASE_URL | - | Yes | ThingsBoard base URL used for telemetry queries and ingestion. |
| THINGSBOARD_SERVICE_TOKEN | - | Yes | Service token or API key for ThingsBoard integration. |
| DATABASE_URL | - | Yes | Connection string for persistence of models, scenarios, and collaboration artefacts. |
| REDIS_URL | - | No | Cache endpoint for WebSocket tickets and rate-limiting state. |
| CORS_ALLOWED_ORIGINS | - | Yes | Comma-separated list of allowed origins for the portal and unified portal. |
| API_RATE_LIMIT_RPM | 1000 | No | Per-principal rate limit in requests per minute. |
| MAX_PAYLOAD_MB | 25 | No | Maximum request payload size in megabytes. |
| WEBSOCKET_TICKET_TTL_MS | 30000 | No | WebSocket ticket time-to-live in milliseconds. |
| LOG_LEVEL | INFO | No | Logging verbosity level. |
| OTEL_EXPORTER_OTLP_ENDPOINT | - | No | OpenTelemetry OTLP collector endpoint for traces and metrics export. |

---

## 10. Requirements Traceability Matrix

| SRS ID | Requirement | Interface Feature | Verification Method |
|--------|-------------|-------------------|---------------------|
| SRS-1-19 | Identity and access management shall be applied to platform interfaces (zero-trust security). | OAuth 2.0/OIDC JWT validation for all protected REST and WebSocket operations. | Security test (token validation, replay protection); integration test. |
| SRS-1-20 | Role-based access control shall be enforced for platform operations. | RBAC enforcement per endpoint (dt_viewer, dt_simulator, dth_editor, compliance_reader, admin). | Integration test with role matrix; negative authorisation tests. |
| SRS-1-21 | Input and output data shall be validated against defined schemas. | OpenAPI 3.1 schemas; server-side validation; RFC 9457 error responses for invalid requests. | Unit tests for schema validation; contract tests. |
| SRS-1-22 | Response time shall not exceed 3 seconds. | Performance targets for telemetry query and CRUD endpoints; caching and pagination support. | Load and performance test; P95 measurement. |
| SRS-1-23 | Data transmission shall be encrypted. | TLS 1.3 enforced for HTTPS and WebSocket over TLS. | Configuration verification; TLS scan. |
| SRS-1-24 | System availability shall reach at least 99.5%. | Health and readiness endpoints; observability instrumentation. | Operational monitoring; SLA reporting. |
| SRS-11-1 | Model annotation capability shall be provided. | POST/GET annotations for model elements. | Functional test (CRUD) and regression test. |
| SRS-11-2 | Multi-user collaboration for model editing shall be supported. | Collaboration note endpoints; model list and retrieval operations; tenant scoping. | Functional test with multiple principals; concurrency test. |
| SRS-11-3 | Stakeholder tagging within collaboration tools shall be supported. | Mentions represented via mentionUserIds; notification events on WebSocket channel. | Functional test (mention creation and retrieval). |
| SRS-11-4 | Real-time updates and version control shall be provided with propagation delay <= 5 s. | WebSocket channel with JSON Patch and version metadata; ETag support for REST updates. | Integration test with dual clients; latency measurement. |
| SRS-11-5 | Daily automatic backups shall be provided for collaboration artefacts. | Backup scheduling and backup status endpoint; backup restore verification procedure. | Operational test; audit of backup logs. |
| SRS-11-6 | Quality control data and compliance reports shall be available. | Compliance report query and retrieval endpoints with RBAC protection. | Functional test; access control test. |
| NEED-8-7 | Simulation dashboard shall support measured and simulated process data visualisation. | Telemetry query endpoint with source filter; simulated telemetry ingestion; simulation scenario management. | Functional test (scenario creation, simulated data ingestion, dashboard retrieval). |

---

## 11. Acceptance Criteria

Each Acceptance Criterion (AC) defines a testable requirement for ICD-09.

| AC ID | Criterion | Test Method | SRS Ref |
|-------|-----------|-------------|---------|
| AC-01 | Protected endpoints reject missing or invalid tokens with HTTP 401 and RFC 9457 payload. | Integration test | SRS-1-19 |
| AC-02 | Protected endpoints reject insufficient roles with HTTP 403 and RFC 9457 payload. | Integration test (role matrix) | SRS-1-20 |
| AC-03 | GET telemetry returns correct key series for specified time range and completes within P95 <= 3 s under nominal load. | Performance test | SRS-1-22 |
| AC-04 | Real-time collaboration updates propagate to subscribed clients with end-to-end delay <= 5 s. | Integration test (dual client) | SRS-11-4 |
| AC-05 | Model element annotation creation and retrieval operate under RBAC and persist the author identifier. | Functional test | SRS-11-1 |
| AC-06 | Collaboration note creation with mentionUserIds persists mentions and emits a corresponding event on the WebSocket channel. | Functional test | SRS-11-2; SRS-11-3 |
| AC-07 | Collaboration artefacts are included in daily backups and backup status endpoint reports the last successful backup timestamp. | Operational test | SRS-11-5 |
| AC-08 | Compliance report retrieval returns report metadata and artefact reference under compliance_reader role. | Functional test | SRS-11-6 |
| AC-09 | All interface traffic uses TLS 1.3 with validated server certificates and approved cipher suites. | Security configuration audit | SRS-1-23 |
| AC-10 | Monthly service availability is >= 99.5% based on health monitoring. | Operational monitoring | SRS-1-24 |
| AC-11 | Simulated telemetry submitted for a scenario is retrievable via GET telemetry with source=SIMULATED and scenarioId. | Functional test | NEED-8-7 |

---

## 12. References

[1] DATA4CIRC Deliverable D2.2: Requirements and Specifications, RWTH Aachen University.

[2] DATA4CIRC Deliverable D4.1: Platform Architecture and Open-Source Protocols.

[3] OpenAPI Initiative, OpenAPI Specification v3.1.0.

[4] IETF, "The OAuth 2.0 Authorization Framework," RFC 6749.

[5] OpenID Foundation, "OpenID Connect Core 1.0," Nov. 2014.

[6] IETF, "Problem Details for HTTP APIs," RFC 9457.

[7] IETF, "The WebSocket Protocol," RFC 6455.

[8] IETF, "The Transport Layer Security (TLS) Protocol Version 1.3," RFC 8446.

[9] IETF, "The JavaScript Object Notation (JSON) Data Interchange Format," RFC 8259.

[10] IETF, "JavaScript Object Notation (JSON) Patch," RFC 6902.

[11] Regenstrief Institute, "Unified Code for Units of Measure (UCUM)."

[12] ThingsBoard Documentation: REST API and Telemetry Data Model.

---

## 13. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 15 December 2025 | FOS; IDE | Initial draft aligned to template; interface scope definition. |
| 0.9 | 24 December 2025 | FOS; IDE | Completed endpoint catalogue, data structures, security, and RTM. |
| 1.0 | 31 December 2025 | FOS; IDE | Final review updates; consistency checks; approval baseline. |

---

## Annex A: Sequence Diagrams

Annex A provides normative sequence diagrams in PlantUML notation for the principal interaction patterns between the DT/DTh Portal, the DT/DTh Application API, and downstream services.

```plantuml
@startuml
title ICD-09 Sequence: Telemetry Query (GET /dt/entities/{entityId}/telemetry)

participant "DT/DTh Portal" as Portal
participant "DT/DTh Application API" as API
participant "ThingsBoard" as TB

Portal -> API: GET /api/v1/dt/entities/{entityId}/telemetry
Authorization: Bearer <JWT>
keys,startTs,endTs,source
activate API
API -> API: Validate JWT (JWKS)
Enforce RBAC (dt_viewer)
API -> TB: Query telemetry time-series
(entityId, keys, startTs, endTs)
activate TB
TB --> API: Telemetry result set
deactivate TB
API --> Portal: 200 OK
TelemetryQueryResult (JSON)
deactivate API
@enduml
```

```plantuml
@startuml
title ICD-09 Sequence: Simulated Telemetry Ingestion (POST /dt/entities/{entityId}/telemetry/simulated)

participant "DT/DTh Portal" as Portal
participant "DT/DTh Application API" as API
participant "ThingsBoard / Time-series Store" as TS

Portal -> API: POST /api/v1/dt/entities/{entityId}/telemetry/simulated?scenarioId={scenarioId}
Authorization: Bearer <JWT>
Body: [TelemetryPoint...]
activate API
API -> API: Validate JWT (JWKS)
Enforce RBAC (dt_simulator)
Validate schema
API -> TS: Persist points with provenance=SIMULATED
and scenario binding
activate TS
TS --> API: Ingestion acknowledgement
deactivate TS
API --> Portal: 202 Accepted
IngestionResult (JSON)
deactivate API
@enduml
```

```plantuml
@startuml
title ICD-09 Sequence: Real-Time Collaboration (WebSocket)

participant "DT/DTh Portal A" as A
participant "DT/DTh Portal B" as B
participant "DT/DTh Application API" as API

A -> API: POST /api/v1/models/{modelId}/realtime/tickets
Authorization: Bearer <JWT>
API --> A: 200 OK {ticket, expiresAt}

A -> API: WebSocket /api/v1/models/{modelId}/realtime?ticket={ticket}
activate API
API -> API: Validate ticket
Bind session to principal and modelId
API --> A: WebSocket accepted

B -> API: POST /api/v1/models/{modelId}/realtime/tickets
Authorization: Bearer <JWT>
API --> B: 200 OK {ticket, expiresAt}
B -> API: WebSocket /api/v1/models/{modelId}/realtime?ticket={ticket}
API --> B: WebSocket accepted

A -> API: model.patch {baseVersion, patch}
API -> API: Validate baseVersion
Apply patch
Persist model version
API --> A: ack {newVersion}
API --> B: model.patch {newVersion, patch}

deactivate API
@enduml
```

## Annex B: Complete API Schema

Annex B provides an OpenAPI v3.1.0 definition for the REST endpoints in ICD-09. The WebSocket channel is documented as an extension (x-websocket) because OpenAPI does not standardise bidirectional messaging contracts.

```yaml
openapi: 3.1.0
info:
  title: DT/DTh Application API (ICD-09)
  version: "1.0.0"
  description: >
    REST API contract between the DT/DTh Portal and the DT/DTh Application.
servers:
  - url: https://<dt-dth-api-hostname>/api/v1
security:
  - bearerAuth: []
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    oidc:
      type: openIdConnect
      openIdConnectUrl: https://<keycloak-hostname>/realms/<realm>/.well-known/openid-configuration
  schemas:
    ProblemDetails:
      type: object
      required: [type, title, status]
      properties:
        type: { type: string, format: uri }
        title: { type: string }
        status: { type: integer }
        detail: { type: string }
        instance: { type: string, format: uri }
        errorCode: { type: string }
        traceId: { type: string }
    TelemetryPoint:
      type: object
      required: [timestamp, key, value, source]
      properties:
        timestamp: { type: integer, format: int64, description: "Unix epoch milliseconds (UTC)" }
        key: { type: string }
        value:
          oneOf:
            - { type: number }
            - { type: integer }
            - { type: string }
            - { type: boolean }
            - { type: object }
        unit: { type: string, description: "UCUM unit for numeric values" }
        source: { type: string, enum: [REAL, SIMULATED] }
        scenarioId: { type: string, format: uuid }
        quality: { type: string, enum: [RAW, VALIDATED, SIMULATED] }
        attributes: { type: object }
    TelemetryQueryResult:
      type: object
      required: [entityId, keys, points]
      properties:
        entityId: { type: string, format: uuid }
        keys:
          type: array
          items: { type: string }
        points:
          type: array
          items: { $ref: "#/components/schemas/TelemetryPoint" }
    IngestionResult:
      type: object
      required: [scenarioId, entityId, acceptedPoints, rejectedPoints, ingestionStatus, timestamp]
      properties:
        scenarioId: { type: string, format: uuid }
        entityId: { type: string, format: uuid }
        acceptedPoints: { type: integer }
        rejectedPoints: { type: integer }
        ingestionStatus: { type: string, enum: [QUEUED, STORED, FAILED] }
        timestamp: { type: integer, format: int64 }
    CreateScenarioRequest:
      type: object
      required: [name, entityId]
      properties:
        name: { type: string }
        description: { type: string }
        entityId: { type: string, format: uuid }
        parameters: { type: object }
        timeRange:
          type: object
          properties:
            startTs: { type: integer, format: int64 }
            endTs: { type: integer, format: int64 }
    SimulationScenario:
      type: object
      required: [scenarioId, name, entityId, createdBy, createdAt, status]
      properties:
        scenarioId: { type: string, format: uuid }
        name: { type: string }
        description: { type: string }
        entityId: { type: string, format: uuid }
        createdBy: { type: string }
        createdAt: { type: integer, format: int64 }
        status: { type: string, enum: [CREATED, QUEUED, RUNNING, COMPLETED, FAILED] }
        parameters: { type: object }
        timeRange: { type: object }
    ModelMetadata:
      type: object
      required: [modelId, name, modelType, format, version, etag, updatedAt, updatedBy]
      properties:
        modelId: { type: string, format: uuid }
        name: { type: string }
        description: { type: string }
        modelType: { type: string, enum: [CVCM] }
        format: { type: string, enum: [CVCM_JSON] }
        version: { type: integer }
        etag: { type: string }
        updatedAt: { type: integer, format: int64 }
        updatedBy: { type: string }
    CreateAnnotationRequest:
      type: object
      required: [type, text]
      properties:
        type: { type: string, enum: [COMMENT, ISSUE, DECISION, RISK] }
        text: { type: string }
        tags:
          type: array
          items: { type: string }
        mentionUserIds:
          type: array
          items: { type: string, format: uuid }
    Annotation:
      allOf:
        - $ref: "#/components/schemas/CreateAnnotationRequest"
        - type: object
          required: [annotationId, modelId, elementId, authorId, createdAt]
          properties:
            annotationId: { type: string, format: uuid }
            modelId: { type: string, format: uuid }
            elementId: { type: string }
            authorId: { type: string }
            createdAt: { type: integer, format: int64 }
            resolved: { type: boolean }
    CreateNoteRequest:
      type: object
      required: [text]
      properties:
        text: { type: string }
        mentionUserIds:
          type: array
          items: { type: string, format: uuid }
        relatedElementIds:
          type: array
          items: { type: string }
    CollaborationNote:
      allOf:
        - $ref: "#/components/schemas/CreateNoteRequest"
        - type: object
          required: [noteId, modelId, authorId, createdAt, version]
          properties:
            noteId: { type: string, format: uuid }
            modelId: { type: string, format: uuid }
            authorId: { type: string }
            createdAt: { type: integer, format: int64 }
            version: { type: integer }
    WsTicketResponse:
      type: object
      required: [ticket, expiresAt]
      properties:
        ticket: { type: string }
        expiresAt: { type: integer, format: int64 }
paths:
  /dt/entities/{entityId}/telemetry:
    get:
      summary: Query Digital Twin telemetry
      parameters:
        - name: entityId
          in: path
          required: true
          schema: { type: string, format: uuid }
        - name: keys
          in: query
          required: true
          schema: { type: string }
        - name: startTs
          in: query
          required: true
          schema: { type: integer, format: int64 }
        - name: endTs
          in: query
          required: true
          schema: { type: integer, format: int64 }
        - name: source
          in: query
          required: false
          schema: { type: string, enum: [REAL, SIMULATED, BOTH], default: BOTH }
        - name: aggregation
          in: query
          required: false
          schema: { type: string, enum: [NONE, AVG, MIN, MAX, SUM], default: NONE }
        - name: intervalMs
          in: query
          required: false
          schema: { type: integer, format: int64 }
        - name: scenarioId
          in: query
          required: false
          schema: { type: string, format: uuid }
        - name: limit
          in: query
          required: false
          schema: { type: integer, format: int32, default: 10000 }
      responses:
        "200":
          description: Telemetry query result
          content:
            application/json:
              schema: { $ref: "#/components/schemas/TelemetryQueryResult" }
        "400": { description: Bad Request, content: { application/problem+json: { schema: { $ref: "#/components/schemas/ProblemDetails" } } } }
        "401": { description: Unauthorised, content: { application/problem+json: { schema: { $ref: "#/components/schemas/ProblemDetails" } } } }
        "403": { description: Forbidden, content: { application/problem+json: { schema: { $ref: "#/components/schemas/ProblemDetails" } } } }
  /dt/entities/{entityId}/telemetry/simulated:
    post:
      summary: Submit simulated telemetry
      parameters:
        - name: entityId
          in: path
          required: true
          schema: { type: string, format: uuid }
        - name: scenarioId
          in: query
          required: true
          schema: { type: string, format: uuid }
        - name: Idempotency-Key
          in: header
          required: false
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: array
              items: { $ref: "#/components/schemas/TelemetryPoint" }
      responses:
        "202":
          description: Accepted for ingestion
          content:
            application/json:
              schema: { $ref: "#/components/schemas/IngestionResult" }
        "400": { description: Bad Request, content: { application/problem+json: { schema: { $ref: "#/components/schemas/ProblemDetails" } } } }
        "401": { description: Unauthorised, content: { application/problem+json: { schema: { $ref: "#/components/schemas/ProblemDetails" } } } }
        "403": { description: Forbidden, content: { application/problem+json: { schema: { $ref: "#/components/schemas/ProblemDetails" } } } }
  /simulations:
    post:
      summary: Create simulation scenario
      parameters:
        - name: Idempotency-Key
          in: header
          required: false
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/CreateScenarioRequest" }
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema: { $ref: "#/components/schemas/SimulationScenario" }
  /models:
    get:
      summary: List models
      parameters:
        - name: page
          in: query
          required: false
          schema: { type: integer, default: 0 }
        - name: size
          in: query
          required: false
          schema: { type: integer, default: 50, maximum: 500 }
        - name: modelType
          in: query
          required: false
          schema: { type: string, enum: [CVCM] }
        - name: sort
          in: query
          required: false
          schema: { type: string }
      responses:
        "200":
          description: Model page
          content:
            application/json:
              schema:
                type: object
                properties:
                  items:
                    type: array
                    items: { $ref: "#/components/schemas/ModelMetadata" }
                  page: { type: integer }
                  size: { type: integer }
                  total: { type: integer }
  /models/{modelId}/elements/{elementId}/annotations:
    post:
      summary: Create annotation
      parameters:
        - { name: modelId, in: path, required: true, schema: { type: string, format: uuid } }
        - { name: elementId, in: path, required: true, schema: { type: string } }
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/CreateAnnotationRequest" }
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Annotation" }
  /models/{modelId}/collaboration/notes:
    post:
      summary: Create collaboration note
      parameters:
        - { name: modelId, in: path, required: true, schema: { type: string, format: uuid } }
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/CreateNoteRequest" }
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema: { $ref: "#/components/schemas/CollaborationNote" }
  /models/{modelId}/realtime/tickets:
    post:
      summary: Request WebSocket ticket
      parameters:
        - { name: modelId, in: path, required: true, schema: { type: string, format: uuid } }
      responses:
        "200":
          description: Ticket issued
          content:
            application/json:
              schema: { $ref: "#/components/schemas/WsTicketResponse" }
x-websocket:
  /models/{modelId}/realtime:
    protocol: "wss"
    subprotocol: "dt-dth.v1"
    queryParameters:
      ticket: "WebSocket ticket from /models/{modelId}/realtime/tickets"
    messages:
      - name: model.patch
        payload:
          type: object
          properties:
            baseVersion: { type: integer }
            patch: { type: array }
      - name: ack
        payload:
          type: object
          properties:
            newVersion: { type: integer }
```

## Annex C: Test Cases

Annex C provides integration-level test case specifications for ICD-09. Test cases are defined with preconditions, steps, and expected results.

**TC-ICD9-01 Authentication Enforcement**

Preconditions:
- DT/DTh Application reachable via HTTPS.

Steps:
1) Invoke GET /api/v1/dt/entities/{entityId}/telemetry without Authorization header.

Expected Results:
- HTTP 401.
- Content-Type: application/problem+json.
- ProblemDetails.status = 401.

**TC-ICD9-02 RBAC Enforcement**

Preconditions:
- Access token with role dt_viewer.

Steps:
1) Invoke POST /api/v1/dt/entities/{entityId}/telemetry/simulated?scenarioId={scenarioId}.

Expected Results:
- HTTP 403.
- ProblemDetails.status = 403.

**TC-ICD9-03 Telemetry Query Correctness**

Preconditions:
- Access token with role dt_viewer.
- Telemetry data exists for entityId and keys within [startTs, endTs].

Steps:
1) Invoke GET /api/v1/dt/entities/{entityId}/telemetry with keys, startTs, endTs, source=BOTH.

Expected Results:
- HTTP 200.
- Returned points match the requested time window.
- Each point includes source and timestamp.

**TC-ICD9-04 Simulated Telemetry Idempotency**

Preconditions:
- Access token with role dt_simulator.
- Existing scenarioId.

Steps:
1) Invoke POST /telemetry/simulated with Idempotency-Key=K and payload P.
2) Repeat request with identical Idempotency-Key=K and identical payload P.

Expected Results:
- Second request returns HTTP 202 (or 200) without duplicating stored points.

**TC-ICD9-05 Annotation CRUD**

Preconditions:
- Access token with role dth_editor.
- Existing modelId and elementId.

Steps:
1) POST annotation with type=COMMENT and text field.
2) GET annotations for modelId and elementId.

Expected Results:
- HTTP 201 for POST and HTTP 200 for GET.
- Annotation returned contains authorId and createdAt fields.

**TC-ICD9-06 Stakeholder Mention Propagation**

Preconditions:
- Two principals subscribed to modelId real-time channel.

Steps:
1) Create collaboration note with mentionUserIds including Principal B.

Expected Results:
- Collaboration note stored with mentionUserIds.
- WebSocket message emitted to subscribed clients including Principal B.

**TC-ICD9-07 WebSocket Ticket TTL**

Preconditions:
- Access token with role dth_editor.

Steps:
1) Request WebSocket ticket.
2) Establish WebSocket channel within TTL.
3) Establish WebSocket channel after TTL expiry.

Expected Results:
- Connection within TTL succeeds.
- Connection after TTL fails with WebSocket close code indicating authentication failure.

**TC-ICD9-08 Real-Time Update Latency**

Preconditions:
- Two WebSocket clients subscribed to modelId.

Steps:
1) Client A sends model.patch message.
2) Measure time until Client B receives corresponding patch.

Expected Results:
- End-to-end propagation delay <= 5 s.

**TC-ICD9-09 Backup Status Reporting**

Preconditions:
- Access token with role admin.

Steps:
1) Invoke GET /api/v1/admin/backups/status.

Expected Results:
- HTTP 200.
- Response includes lastSuccessfulBackup timestamp and backup policy metadata.

**TC-ICD9-10 Compliance Report Retrieval**

Preconditions:
- Access token with role compliance_reader.

Steps:
1) Invoke GET /api/v1/compliance/reports?modelId={modelId}.

Expected Results:
- HTTP 200.
- Response includes report metadata and artefact reference.

## Annex D: Quality Checklist

| Check | Criterion | Section |
|-------|----------|---------|
| Yes | Units of measure specified for all numerical fields | Section 6.1 |
| N/A | Semantic IDs (IRDIs) provided for Asset Administration Shell compliant fields | Section 6.1 |
| Yes | Environment variables listed for DevOps deployment | Section 9.5 |
| Yes | Circuit breaker thresholds defined for resilience | Section 4.2 |
| Yes | PII fields flagged and retention policies defined | Section 6.3 |
| N/A | ODRL policies defined for dataspace interfaces | Section 7.4 |
| N/A | MQTT topics, quality of service levels, and last will and testament defined for IoT interfaces | Section 5.3, 9.4 |
| Yes | Error handling appropriate for protocol (RFC 9457 or DLQ) | Section 2.3 |
| Yes | Health check mechanism defined (HTTP endpoint or MQTT last will and testament) | Section 9.4 |
| Yes | Interface dependencies and versioning documented | Section 1.4 |
| Yes | British English and IEEE style followed throughout | All sections |
| Yes | No subjunctive mood, personal pronouns, or filler words | All sections |
| Yes | Abbreviations defined once and listed in Section 3 | Section 3 |
| Yes | Performance targets use specific numerical values | Section 8 |
