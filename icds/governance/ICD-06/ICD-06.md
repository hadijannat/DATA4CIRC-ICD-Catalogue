# ICD-06: SPIP Governance Services API

**SPIP Platform <-> SPIP Agent**

---

| Attribute | Value |
|-----------|-------|
| **Version** | 1.0 |
| **Date** | 30 December 2025 |
| **Work Package** | WP3 (Trust and Governance Services) |
| **Author(s)** | DATA4CIRC Task 4.2 (ICD-6 Authoring) |
| **Provider Owner** | Approved |
| **Consumer Owner** | SPIP Agent Owner (Tool Integrators, WP4-WP6) |
| **Reviewer** | DATA4CIRC Technical Board |
| **Status** | Approved |

---

## Document Completion Guidelines

This section provides mandatory writing conventions and completion instructions for all Interface Control Documents within the DATA4CIRC project. All contributors shall adhere to these guidelines to ensure consistency, scientific rigour, and compliance with EU Horizon Europe deliverable standards.

### Writing Style Requirements

- British English and IEEE deliverable style are mandatory.
- Personal pronouns, spatial and temporal references, and subjunctive constructions are excluded.
- Filler words and colloquialisms are excluded.
- Units of measure are specified for all numerical values.

### Abbreviation Rules

Each abbreviation is defined once at first use in the format Full Term (ABBR). Subsequently, only the abbreviation is used. All abbreviations appear in Section 3.

## 1. Interface Overview

### 1.1 Purpose

The Interface Control Document specifies the governance services interface between the Secure Policy Information Point (SPIP) Platform and the SPIP Agent. The SPIP Platform provides server-side encryption, decryption, and policy operations, enabling secure key management and policy management for governed data exchange. The SPIP Agent acts as a client-side component that consumes the SPIP Platform API to perform cryptographic operations on data and to enforce Attribute-Based Access Control (ABAC) and Attribute-Based Encryption (ABE) policies. The interface supports the integration of the SPIP infrastructure stack for policy creation and key management, the implementation of ABE policies, and the generation and distribution of keys through the SPIP Key Vault, aligned with system requirements SRS-1-1 to SRS-1-5.

### 1.2 Communicating Components

| Attribute | Component A | Component B |
|-----------|-------------|-------------|
| **Component Name** | SPIP Platform | SPIP Agent |
| **Role** | Provider | Consumer |
| **Work Package** | WP3 | WP3 |
| **Responsible Partner** | NTT Data | Tool Integrator |

### 1.3 Architectural Context

The SPIP Platform and associated governance services form part of the Central Trust and Governance Plane within the DATA4CIRC reference architecture, together with Identity and Trust Services and Catalogue/Broker functions. ICD-06 is classified as a Governance Interface and specifies communication between the SPIP Platform and the SPIP Agent. The SPIP Agent is deployed within dataspace participant environments and interacts with the SPIP Platform to obtain policies and cryptographic material required for encryption and decryption operations. Downstream integrations include the SPIP Agent <-> Dataspace Connector interface (ICD-05) and tool <-> SPIP Agent interfaces (ICD-14 to ICD-16). Upstream integrations include the DS4CIRC Portal <-> SPIP Platform interface (ICD-07).

### 1.4 Interface Dependencies and Lifecycle

| Attribute | Specification |
|-----------|---------------|
| **Prerequisites** | - SPIP Agent service account registered in the Identity and Trust Service with OAuth 2.0 client credentials and assigned scopes.\n- Mutual Transport Layer Security (mTLS) certificates provisioned for SPIP Agent and SPIP Platform and signed by a trusted Certification Authority (CA).\n- Network connectivity and Domain Name System (DNS) resolution for the SPIP Platform base URL (Transmission Control Protocol (TCP) port 443).\n- SPIP Platform dependencies initialised, including SPIP Key Vault connectivity and master key material provisioning.\n- Policy persistence layer initialised for policy lifecycle operations.\n- Time synchronisation across participating hosts to support token and certificate validation. |
| **Versioning Strategy** | Uniform Resource Identifier (URI) versioning uses a major version prefix in the base path (for example, "/api/v1"). Backward-compatible changes increment minor and patch versions without modifying the base path. Backward-incompatible changes increment the major version and introduce a new base path. |
| **Deprecation Policy** | Deprecation is announced by documentation updates and HTTP response headers. Deprecated versions remain available for 180 d before removal in a subsequent major version. |
| **Downstream Dependents** | ICD-05 (SPIP Agent <-> Dataspace Connector), ICD-07 (DS4CIRC Portal <-> SPIP Platform), and tool integration ICDs ICD-14 to ICD-16 depend on ICD-06 for governance services, cryptographic operations, and policy management. |

## 2. Functional Description

### 2.1 Functional Capabilities

| ID | Capability | Description | SRS Reference |
|----|------------|-------------|---------------|
| FC-01 | Policy lifecycle management (create, update, retrieve, list, revoke) for ABE policies and attribute namespaces. | Policy lifecycle management for ABE policies and attribute namespaces. | SRS-1-2, SRS-1-14 |
| FC-02 | Encryption service that encrypts payloads under an ABE policy and returns ciphertext and an encryption envelope. | Encryption service using ABE policy binding and envelope generation. | SRS-1-4, SRS-1-2 |
| FC-03 | Decryption service that validates authorisation against an ABE policy and returns plaintext for authorised principals. | Decryption with ABE policy validation for authorised principals. | SRS-1-5, SRS-1-2 |
| FC-04 | Attribute key issuance and distribution through the SPIP Key Vault for authorised principals. | Attribute key issuance and distribution through the SPIP Key Vault. | SRS-1-3 |
| FC-05 | Audit event recording for cryptographic operations and policy lifecycle events to enable usage tracking and compliance. | Audit event recording for cryptographic operations and policy lifecycle events. | SRS-1-10 |
| FC-06 | Policy enforcement aligned with data-sharing contracts and usage restrictions by binding decryption operations to policy evaluation. | Policy enforcement aligned with contract and usage restrictions. | SRS-1-11 |
| FC-07 | Endpoint-level authentication and role-based permission enforcement. | Authentication and authorisation enforcement per endpoint. | SRS-1-19, SRS-1-20 |
| FC-08 | Transport security enforcing encryption for data in transit. | Transport security for data in transit. | SRS-1-23 |
| FC-09 | Performance and availability targets for governance services operations. | Performance and availability targets for governance services. | SRS-1-22, SRS-1-24 |

### 2.2 Interaction Patterns

Communication follows a synchronous request-response interaction pattern using HTTPS. Primary interaction flows comprise: (1) policy lifecycle management, including creation, update, retrieval, listing, and revocation of ABE policies; (2) encryption, where the SPIP Agent submits a plaintext payload and policy reference and the SPIP Platform returns ciphertext and an encryption envelope; (3) decryption, where the SPIP Agent submits ciphertext and an encryption envelope and the SPIP Platform evaluates policy conditions against authenticated principal attributes and returns plaintext upon authorisation; (4) key issuance, where the SPIP Agent requests attribute keys and the SPIP Platform issues cryptographic material with a defined validity period; (5) audit logging, where the SPIP Platform records cryptographic operations and policy lifecycle events for auditing and compliance.

### 2.3 Error Handling

#### 2.3.1 HTTP/REST Error Handling

For HTTP/REST interfaces, error responses shall conform to RFC 9457 (Problem Details for HTTP APIs).

| HTTP Status | Condition | Recovery Action |
|------------|-----------|-----------------|
| 400 | urn:data4circ:spip:error:bad-request - Request validation failed (schema violation, malformed base64 encoding, unsupported parameter value). | Correct request payload and resubmit. Retries without correction are not permitted. |
| 401 | urn:data4circ:spip:error:unauthorised - Authentication failure (missing, expired, or invalid access token; missing or invalid mTLS client certificate). | Acquire a valid access token and present a valid client certificate, then resubmit. |
| 403 | urn:data4circ:spip:error:forbidden - Authorisation failure (insufficient scope or role; policy evaluation denied decryption; key issuance not permitted). | Ensure correct scopes and roles and ensure attribute set satisfies policy constraints. |
| 404 | urn:data4circ:spip:error:not-found - Referenced resource not found (policy identifier, key identifier, audit event identifier). | Verify identifier correctness and resource lifecycle state, then resubmit. |
| 409 | urn:data4circ:spip:error:conflict - Conflict detected (concurrent update, policy version mismatch, idempotency key reuse with differing request payload). | Resolve version conflict or generate a new idempotency key, then resubmit. |
| 413 | urn:data4circ:spip:error:payload-too-large - Request payload exceeds configured maximum size. | Reduce payload size or apply payload chunking as specified by Section 4.2. |
| 415 | urn:data4circ:spip:error:unsupported-media-type - Unsupported Content-Type or Accept header. | Use "application/json" for requests and support "application/json" and "application/problem+json" for responses. |
| 422 | urn:data4circ:spip:error:unprocessable-entity - Semantic validation failure (policy expression parsing error, unsupported algorithm selection). | Correct policy expression or select a supported algorithm, then resubmit. |
| 429 | urn:data4circ:spip:error:too-many-requests - Rate limit exceeded. | Apply exponential backoff and resubmit after the server-provided Retry-After interval. |
| 500 | urn:data4circ:spip:error:internal - Unhandled server-side error. | Resubmit using exponential backoff. Escalate through operational support upon repeated occurrences. |
| 503 | urn:data4circ:spip:error:service-unavailable - Service unavailable (dependency failure, maintenance, overload protection). | Resubmit using exponential backoff and honour Retry-After. |

#### 2.3.2 IoT/Async Error Handling

For MQTT and asynchronous interfaces, error handling shall use dedicated error topics and Dead Letter Queue (DLQ) strategies.

| Attribute | Specification |
|-----------|---------------|
| **Error Topic** | Not applicable (synchronous HTTPS interface). |
| **DLQ Strategy** | Not applicable. |
| **Error Payload Schema** | RFC 9457 application/problem+json (Section 2.3). |
| **Retry Policy** | Client retry and backoff policy defined in Section 4.2. |

## 3. Abbreviations

| Abbreviation | Definition |
|--------------|------------|
| ABAC | Attribute-Based Access Control |
| ABE | Attribute-Based Encryption |
| API | Application Programming Interface |
| CA | Certification Authority |
| DNS | Domain Name System |
| DS4CIRC | Manufacturing Dataspace for Circular Economy |
| HTTP | Hypertext Transfer Protocol |
| HTTPS | Hypertext Transfer Protocol Secure |
| ICD | Interface Control Document |
| JSON | JavaScript Object Notation |
| JWT | JSON Web Token |
| mTLS | Mutual Transport Layer Security |
| OIDC | OpenID Connect |
| OAuth 2.0 | OAuth 2.0 Authorization Framework |
| PII | Personally Identifiable Information |
| REST | Representational State Transfer |
| RFC | Request for Comments |
| SPIP | Secure Policy Information Point |
| SRS | System Requirement Specification |
| TCP | Transmission Control Protocol |
| TLS | Transport Layer Security |
| URI | Uniform Resource Identifier |
| UUID | Universally Unique Identifier |

## 4. Communication Protocol

### 4.1 Protocol Stack

| Layer | Protocol | Specification |
|-------|----------|---------------|
| Application | HTTP/REST (JSON payloads); OpenAPI 3.1.0 | Synchronous request-response API. Content uses application/json with explicit base64 encoding for binary payloads. |
| Security | OAuth 2.0 (client credentials) with OpenID Connect JWT access tokens; role-based access control; mutual TLS | Access tokens issued by the Identity and Trust Service. Endpoint authorisation uses scopes and roles. Mutual TLS provides service identity. |
| Transport | HTTPS (HTTP/1.1 and HTTP/2) over TLS 1.3 | Transport security enforces encryption for data in transit and supports mutual authentication via X.509 certificates. |
| Serialisation | JSON; JSON-LD; Base64 | RFC 8259; JSON-LD 1.1; RFC 4648 |

### 4.2 Connection Parameters

| Parameter | Value |
|-----------|-------|
| Base URL / Broker | https://<spip-platform-fqdn>/api/v1 |
| Port | 443 |
| Network Zone | Central Trust and Governance Plane |
| Connection Timeout | 3 s |
| Read Timeout | 10 s (encrypt/decrypt); 5 s (policy operations) |
| Retry Policy | Maximum 5 retries with exponential backoff (0.5 s, 1 s, 2 s, 4 s, 8 s) on HTTP 429, 503, and 504. Retries apply to GET operations and to POST operations that include an Idempotency-Key. |
| Circuit Breaker | Open after 10 failures within 30 s; transition to half-open after 60 s; close after 5 consecutive successes. |
| Firewall Rules | Allow outbound TCP 443 from SPIP Agent network segment to SPIP Platform endpoint. Restrict ingress to SPIP Platform to authenticated mTLS clients. |
| Maximum Payload Size | 10 MiB per request payload (base64-encoded content). |
| TLS Client Authentication | mTLS mandatory. Client certificate subject alternative name (SAN) identifies SPIP Agent service instance. |
| OAuth 2.0 Token Endpoint | https://<idp-fqdn>/realms/<realm>/protocol/openid-connect/token |
| Rate Limit | 100 requests/s per client identity; server returns HTTP 429 with Retry-After. |

## 5. API Specification

### 5.1 Endpoint Definitions

#### 5.1.1 SPIP Governance Services API

| Attribute | Specification |
|-----------|---------------|
| **Method** | GET, POST, PUT, DELETE |
| **Path** | /api/v1/* |
| **Purpose** | Synchronous HTTPS API exposing governance services between the SPIP Agent and the SPIP Platform. Operations include encryption, decryption, policy lifecycle management, key issuance, capability discovery, and health endpoints. The normative path-level specification is provided in Annex B (OpenAPI 3.1). |
| **Authentication** | OAuth 2.0 bearer access token (JWT) issued by the Identity and Trust Service and mTLS. |

**Headers and Parameters**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| Authorization | string | Yes | Bearer access token (JWT) issued by the Identity and Trust Service. Format: "Bearer <token>". |
| traceparent | string | No | W3C Trace Context header propagated end-to-end for distributed tracing. |
| X-Request-ID | string | No | Client-generated correlation identifier (UUID) used for log correlation and troubleshooting. |
| Idempotency-Key | string | Conditional | Idempotency key required for POST /crypto/encrypt and POST /keys/issue. Format: UUID. Enables safe retries. |
| Content-Type | string | Conditional | Mandatory for requests with a body. Value: "application/json; charset=utf-8". |
| Accept | string | No | Accepted response types. Values: "application/json" and "application/problem+json". |
| policyId | string (UUID) | Conditional | Path parameter for policy operations on /policies/{policyId}. |
| page | integer | No | Query parameter for pagination in GET /policies. Default: 0. Unit: count. |
| pageSize | integer | No | Query parameter for pagination in GET /policies. Default: 50. Unit: count. |
| includeRevoked | boolean | No | Query parameter in GET /policies to include revoked policies. Default: false. |

### 5.2 Request and Response Examples

Endpoint summary (normative details in Annex B):

- POST /crypto/encrypt - Encrypt payload under an ABE policy. Required scope: spip.encrypt.
- POST /crypto/decrypt - Decrypt payload subject to ABE policy evaluation. Required scope: spip.decrypt.
- POST /policies - Create policy. Required scope: spip.policies:write.
- GET /policies - List policies. Required scope: spip.policies:read.
- GET /policies/{policyId} - Retrieve policy. Required scope: spip.policies:read.
- PUT /policies/{policyId} - Update policy. Required scope: spip.policies:write.
- DELETE /policies/{policyId} - Revoke policy. Required scope: spip.policies:write.
- POST /keys/issue - Issue attribute key material. Required scope: spip.keys:issue.
- GET /capabilities - Retrieve supported algorithms and policy language capabilities. Required scope: spip.read.
- GET /health/live - Liveness probe. Authentication optional (deployment-specific).
- GET /health/ready - Readiness probe. Authentication optional (deployment-specific).
- GET /metrics - Metrics endpoint (Prometheus format). Required scope: spip.monitor.

Request Example:

```json
{
  "requestId": "b3c8bb5a-6b6f-4d68-91c8-3f5c6a0f0a4d",
  "payload": "eyJtZXNzYWdlIjoiSGVsbG8sIFNQSVAhIn0=",
  "payloadContentType": "application/json",
  "payloadEncoding": "base64",
  "policyRef": {
    "policyId": "2f4f1bb7-2b5d-4a1f-9a24-8df4f9c3f8b1",
    "policyVersion": 3
  },
  "encryptionContext": {
    "assetId": "urn:uuid:9b7d0a2e-9b5a-4a4e-8c72-8e5e9a0c9c1f",
    "purpose": "data-exchange"
  }
}
```

Response Example (200 OK):

```json
{
  "requestId": "b3c8bb5a-6b6f-4d68-91c8-3f5c6a0f0a4d",
  "ciphertext": "Q0lQSEVSVEVYVF9FWEFNUExFX0JBU0U2NA==",
  "ciphertextEncoding": "base64",
  "envelope": {
    "envelopeVersion": 1,
    "contentAlgorithm": "AES-256-GCM",
    "keyAlgorithm": "CP-ABE",
    "policyId": "2f4f1bb7-2b5d-4a1f-9a24-8df4f9c3f8b1",
    "policyVersion": 3,
    "iv": "aXYtYmFzZTY0LWV4YW1wbGU=",
    "tag": "dGFnLWJhc2U2NC1leGFtcGxl",
    "abeCiphertext": "YWJlLWJhc2U2NC1leGFtcGxl",
    "issuedAt": "2025-12-30T00:00:00Z"
  }
}
```

### 5.3 Event and Message Specifications (Asynchronous/MQTT)

Not applicable. ICD-06 specifies a synchronous HTTP/REST interface.

#### 5.3.1 Not applicable

| Attribute | Specification |
|-----------|---------------|
| Topic/Channel | Not applicable. |
| Direction | Not applicable. |
| QoS Level | Not applicable. |
| Trigger Condition | Not applicable. |
| Payload Format | Not applicable. |
| Retention | Not applicable. |

## 6. Data Structures

### 6.1 Data Model

#### 6.1.1 EncryptRequest

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| requestId | string | UUID | N/A | Yes | Client-generated request identifier. Used for correlation and idempotency. |
| payload | string | base64 | N/A | Yes | Plaintext payload encoded as base64. |
| payloadContentType | string | MIME type | N/A | Yes | Media type of the plaintext payload (for example, application/json). |
| payloadEncoding | string | enum: base64 | N/A | Yes | Payload encoding identifier. Value: base64. |
| policyRef.policyId | string | UUID | N/A | Conditional | Policy identifier. Mandatory when policyRef is present. |
| policyRef.policyVersion | integer | count | N/A | Conditional | Policy version used for optimistic concurrency control. |
| policyInline | object | PolicyDefinition | N/A | Conditional | Inline policy definition. Mutually exclusive with policyRef. |
| encryptionContext | object | JSON object | N/A | No | Non-sensitive metadata bound to the encryption envelope for audit correlation. |
| aad | string | base64 | N/A | No | Additional authenticated data for authenticated encryption with associated data (AEAD). |

#### 6.1.2 EncryptResponse

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| requestId | string | UUID | N/A | Yes | Echo of EncryptRequest.requestId. |
| ciphertext | string | base64 | N/A | Yes | Ciphertext payload encoded as base64. |
| ciphertextEncoding | string | enum: base64 | N/A | Yes | Ciphertext encoding identifier. Value: base64. |
| envelope.envelopeVersion | integer | count | N/A | Yes | Encryption envelope schema version. |
| envelope.contentAlgorithm | string | enum | N/A | Yes | Content encryption algorithm identifier (for example, AES-256-GCM). |
| envelope.keyAlgorithm | string | enum | N/A | Yes | Key encapsulation algorithm identifier (for example, CP-ABE). |
| envelope.policyId | string | UUID | N/A | Yes | Policy identifier bound to the ciphertext. |
| envelope.policyVersion | integer | count | N/A | Yes | Policy version bound to the ciphertext. |
| envelope.iv | string | base64 | N/A | Yes | Initialisation vector for AEAD. |
| envelope.tag | string | base64 | N/A | Yes | Authentication tag for AEAD. |
| envelope.abeCiphertext | string | base64 | N/A | Yes | Attribute-based encryption ciphertext containing the wrapped data encryption key. |
| envelope.issuedAt | string | date-time (RFC 3339) | N/A | Yes | Timestamp assigned by SPIP Platform (UTC). |

#### 6.1.3 DecryptRequest

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| requestId | string | UUID | N/A | Yes | Client-generated request identifier. |
| ciphertext | string | base64 | N/A | Yes | Ciphertext payload encoded as base64. |
| ciphertextEncoding | string | enum: base64 | N/A | Yes | Ciphertext encoding identifier. Value: base64. |
| envelope | object | EncryptionEnvelope | N/A | Yes | Encryption envelope returned by POST /crypto/encrypt. |
| decryptionContext | object | JSON object | N/A | No | Non-sensitive metadata used for audit correlation. |
| aad | string | base64 | N/A | No | Additional authenticated data for AEAD. Shall match EncryptRequest.aad when provided. |

#### 6.1.4 DecryptResponse

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| requestId | string | UUID | N/A | Yes | Echo of DecryptRequest.requestId. |
| payload | string | base64 | N/A | Yes | Plaintext payload encoded as base64. |
| payloadContentType | string | MIME type | N/A | Yes | Media type of the plaintext payload. |
| payloadEncoding | string | enum: base64 | N/A | Yes | Plaintext encoding identifier. Value: base64. |
| policyId | string | UUID | N/A | Yes | Policy identifier evaluated for decryption. |
| policyVersion | integer | count | N/A | Yes | Policy version evaluated for decryption. |
| auditEventId | string | UUID | N/A | No | Audit event identifier assigned by SPIP Platform for usage tracking. |

#### 6.1.5 PolicyDefinition

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| policyId | string | UUID | N/A | Conditional | Policy identifier. Omitted for create requests; present for responses and update requests. |
| name | string | UTF-8 | N/A | Yes | Human-readable policy name. |
| description | string | UTF-8 | N/A | No | Human-readable policy description. |
| expression | string | policy language | N/A | Yes | ABE policy expression in SPIP policy language (Section 6.2). |
| attributeNamespace | string | URI | N/A | No | Namespace identifier for attribute keys referenced in expression. |
| version | integer | count | N/A | Yes | Monotonic policy version. |
| status | string | enum: ACTIVE, REVOKED | N/A | Yes | Policy lifecycle state. |
| validFrom | string | date-time (RFC 3339) | N/A | No | Policy validity start timestamp (UTC). |
| validUntil | string | date-time (RFC 3339) | N/A | No | Policy validity end timestamp (UTC). |
| createdAt | string | date-time (RFC 3339) | N/A | Conditional | Creation timestamp assigned by SPIP Platform. |
| updatedAt | string | date-time (RFC 3339) | N/A | Conditional | Last update timestamp assigned by SPIP Platform. |

#### 6.1.6 KeyIssueRequest

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| requestId | string | UUID | N/A | Yes | Client-generated request identifier. |
| keyPurpose | string | enum | N/A | Yes | Key purpose. Values: DECRYPT and ENCRYPT. |
| ttlSeconds | integer | s | N/A | No | Requested key validity duration in seconds. Server may enforce an upper bound. |
| policyId | string | UUID | N/A | No | Optional policy identifier used to scope key issuance. |
| audience | string | URI | N/A | No | Optional audience identifier for issued key material. |

#### 6.1.7 KeyIssueResponse

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| requestId | string | UUID | N/A | Yes | Echo of KeyIssueRequest.requestId. |
| keyId | string | UUID | N/A | Yes | Issued key identifier. |
| keyMaterial | string | base64 | N/A | Yes | Issued key material encrypted for the requesting principal. |
| keyMaterialEncoding | string | enum: base64 | N/A | Yes | Key material encoding identifier. Value: base64. |
| issuedAt | string | date-time (RFC 3339) | N/A | Yes | Key issuance timestamp (UTC). |
| expiresAt | string | date-time (RFC 3339) | N/A | Yes | Key expiration timestamp (UTC). |

#### 6.1.8 ProblemDetails

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|-------------------|-----|-------------|
| type | string | URI | N/A | Yes | Problem type identifier (URN or URL). |
| title | string | UTF-8 | N/A | Yes | Short, human-readable summary of the problem type. |
| status | integer | HTTP status code | N/A | Yes | HTTP status code generated by origin server. |
| detail | string | UTF-8 | N/A | No | Human-readable explanation specific to the occurrence. |
| instance | string | URI | N/A | No | URI reference identifying the specific occurrence. |
| errorCode | string | UTF-8 | N/A | No | DATA4CIRC extension: stable error code for programmatic handling. |
| correlationId | string | UUID | N/A | No | DATA4CIRC extension: correlation identifier for troubleshooting. |
| retryable | boolean | true or false | N/A | No | DATA4CIRC extension: retry guidance for transient failures. |
| validationErrors | array | JSON array | N/A | No | DATA4CIRC extension: list of field-level validation errors for HTTP 400 and 422. |

### 6.2 Semantic Mappings

Semantic mappings for identity claims and access control attributes are specified in Table 6-9.

| Claim / Attribute | Source | Standard / Profile | Meaning |
|-------------------|--------|--------------------|---------|
| sub | OIDC access token (JWT) | OpenID Connect Core 1.0 | Subject identifier for the authenticated principal. |
| iss | OIDC access token (JWT) | OpenID Connect Core 1.0 | Issuer identifier for token validation. |
| aud | OIDC access token (JWT) | OpenID Connect Core 1.0 | Audience restriction for token validation. |
| exp | OIDC access token (JWT) | OpenID Connect Core 1.0 | Token expiration timestamp (UTC). |
| realm_access.roles | Keycloak JWT claim | Keycloak JWT profile | Role set used for role-based authorisation decisions. |
| scope | OAuth 2.0 access token | IETF RFC 6749 | Space-delimited scope list used for endpoint-level authorisation. |
| org_id | Custom JWT claim | DATA4CIRC profile | Organisation identifier for ABAC and ABE policy evaluation. |
| spip_attributes | Custom JWT claim | DATA4CIRC profile | Attribute set used for ABAC and ABE policy evaluation. Encoding: JSON object with string values. |

Semantic alignment for claims and metadata follows standard vocabularies where available. Authentication and authorisation contexts reuse OpenID Connect standard claims (for example, sub, iss, aud, exp) and JSON Web Token conventions. Access control attributes used for ABAC and ABE policy evaluation use a controlled attribute namespace defined by the SPIP Platform configuration. ABE policy expressions use the SPIP policy language defined in Section 6.2, which supports Boolean combinations of attribute predicates using AND, OR, and parentheses.

### 6.3 Data Governance and Compliance

| Data Entity | PII (Y/N) | Classification | Retention Period |
|-------------|-----------|----------------|------------------|
| Plaintext payload - plaintext data provided for encryption or returned by decryption. Persistence is prohibited. | Y | Restricted | Transient (in-memory only) |
| Ciphertext payload - encrypted data returned by SPIP Platform. Persistence is controlled by the calling component. | N | Confidential | Not stored by SPIP Platform |
| Encryption envelope - cryptographic metadata required for decryption (policy identifier, wrapped data key, IV, tag). | N | Confidential | Not stored by SPIP Platform |
| Policy definition - ABE policy metadata and expressions used for encryption and decryption authorisation. | Y/N | Internal | Persisted until revoked; retention configurable (default 365 days) |
| Principal identity and attributes - identity claims and ABAC attributes used for authentication and authorisation decisions. | Y | Confidential | Transient; cached validation metadata configurable (default 15 min) |
| Issued attribute key material - key material issued for ABE operations. Key material remains encrypted for transport. | N | Restricted | Valid until expiry; persistence prohibited |
| Audit records - usage tracking records for cryptographic operations and policy lifecycle events. | Y | Confidential | Retention configurable (default 90 days) |

## 7. Security Requirements

### 7.1 Authentication

| Attribute | Value |
|-----------|-------|
| Mechanism | OAuth 2.0 client credentials with OpenID Connect JWT access tokens; mutual TLS (mTLS) |
| Identity Provider | DATA4CIRC Identity and Trust Service (Keycloak) |
| Token Type | JSON Web Token (JWT), signed (JWS) |
| Token Lifetime | 3600 s (configurable; maximum 86400 s) |

### 7.2 Authorisation

| Operation | Required Role | SRS Reference |
|-----------|---------------|---------------|
| POST /crypto/encrypt | Role: SPIP_AGENT; Scope: spip.encrypt | SRS-1-20, SRS-1-2 |
| POST /crypto/decrypt | Role: SPIP_AGENT; Scope: spip.decrypt; Policy evaluation: ABE/ABAC attributes | SRS-1-20, SRS-1-2 |
| POST /policies | Role: SPIP_POLICY_ADMIN; Scope: spip.policies:write | SRS-1-20, SRS-1-14 |
| GET /policies | Role: SPIP_POLICY_READER; Scope: spip.policies:read | SRS-1-20 |
| GET /policies/{policyId} | Role: SPIP_POLICY_READER; Scope: spip.policies:read | SRS-1-20 |
| PUT /policies/{policyId} | Role: SPIP_POLICY_ADMIN; Scope: spip.policies:write | SRS-1-20, SRS-1-14 |
| DELETE /policies/{policyId} | Role: SPIP_POLICY_ADMIN; Scope: spip.policies:write | SRS-1-20 |
| POST /keys/issue | Role: SPIP_AGENT; Scope: spip.keys:issue | SRS-1-20, SRS-1-3 |
| GET /capabilities | Role: SPIP_AGENT; Scope: spip.read | SRS-1-20 |
| GET /metrics | Role: SPIP_MONITOR; Scope: spip.monitor | SRS-1-20 |

### 7.3 Transport Security

| Attribute | Value |
|-----------|-------|
| TLS Version | TLS 1.3 minimum |
| Certificate Validation | X.509 certificates validated against a trusted Certification Authority. Server certificate subject alternative name (SAN) shall match the SPIP Platform DNS name. Client certificate validation is mandatory for mTLS. |
| Cipher Suites | TLS_AES_256_GCM_SHA384; TLS_AES_128_GCM_SHA256; TLS_CHACHA20_POLY1305_SHA256 |

### 7.4 Usage Control (ODRL Policies)

| Policy Element | Specification |
|----------------|---------------|
| Permission | SPIP operation permission (ENCRYPT, DECRYPT, POLICY_READ, POLICY_WRITE, KEY_ISSUE). |
| Constraint | ABE/ABAC constraint expressed as a Boolean policy over attributes (for example, org_role == "remanufacturer" AND region == "EU"). |
| Duty | Duty: AUDIT_LOG (audit event generation is mandatory for cryptographic operations); RETENTION_LIMIT (policy metadata retention enforced by platform configuration). |
| Prohibition | Prohibition expressed as policy evaluation failure resulting in DENY (HTTP 403) or key issuance refusal. |

## 8. Performance Requirements

| Metric | Target | SRS Reference |
|--------|--------|---------------|
| Response Time (P95) | <= 3 s | SRS-1-22 |
| Throughput | >= 100 requests/s (sustained) | SRS-1-22 |
| Availability | >= 99.5% (monthly) | SRS-1-24 |
| Max Payload Size | 10 MiB | SRS-1-22 |

## 9. Implementation Guidelines

### 9.1 Client Implementation Example

Python (FastAPI) Example:

```python
import os
import uuid
import base64
import requests

SPIP_BASE_URL = os.environ["SPIP_PLATFORM_BASE_URL"]       # https://<spip-platform-fqdn>/api/v1
TOKEN_URL = os.environ["SPIP_OIDC_TOKEN_URL"]              # https://<idp-fqdn>/realms/<realm>/protocol/openid-connect/token
CLIENT_ID = os.environ["SPIP_OIDC_CLIENT_ID"]
CLIENT_SECRET = os.environ["SPIP_OIDC_CLIENT_SECRET"]

MTLS_CERT = os.environ["SPIP_MTLS_CERT_PATH"]
MTLS_KEY = os.environ["SPIP_MTLS_KEY_PATH"]
CA_BUNDLE = os.environ.get("SPIP_TLS_CA_BUNDLE_PATH")      # path to CA bundle (PEM)

CONNECT_TIMEOUT_SECONDS = 3
READ_TIMEOUT_SECONDS = 10

def get_access_token() -> str:
    payload = {
        "grant_type": "client_credentials",
        "client_id": CLIENT_ID,
        "client_secret": CLIENT_SECRET,
    }
    resp = requests.post(
        TOKEN_URL,
        data=payload,
        timeout=(CONNECT_TIMEOUT_SECONDS, READ_TIMEOUT_SECONDS),
        verify=CA_BUNDLE,
    )
    resp.raise_for_status()
    return resp.json()["access_token"]

def encrypt_payload(payload_bytes: bytes, policy_id: str, policy_version: int) -> dict:
    token = get_access_token()
    request_id = str(uuid.uuid4())

    body = {
        "requestId": request_id,
        "payload": base64.b64encode(payload_bytes).decode("ascii"),
        "payloadContentType": "application/json",
        "payloadEncoding": "base64",
        "policyRef": {"policyId": policy_id, "policyVersion": policy_version},
        "encryptionContext": {"purpose": "data-exchange"},
    }

    headers = {
        "Authorization": f"Bearer {token}",
        "Idempotency-Key": request_id,
        "Content-Type": "application/json; charset=utf-8",
        "Accept": "application/json",
    }

    resp = requests.post(
        f"{SPIP_BASE_URL}/crypto/encrypt",
        json=body,
        headers=headers,
        cert=(MTLS_CERT, MTLS_KEY),
        verify=CA_BUNDLE,
        timeout=(CONNECT_TIMEOUT_SECONDS, READ_TIMEOUT_SECONDS),
    )
    resp.raise_for_status()
    return resp.json()
```

### 9.2 Server Implementation Example

Java (Spring Boot) Example:

```java
package org.data4circ.spip.api;

import jakarta.validation.Valid;
import jakarta.validation.constraints.NotNull;
import org.springframework.http.MediaType;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

import java.util.UUID;

@RestController
@RequestMapping(path = "/api/v1")
public final class CryptoController {

    private final CryptoService cryptoService;

    public CryptoController(CryptoService cryptoService) {
        this.cryptoService = cryptoService;
    }

    @PostMapping(
            path = "/crypto/encrypt",
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    @PreAuthorize("hasAuthority('SCOPE_spip.encrypt') and hasRole('SPIP_AGENT')")
    public EncryptResponse encrypt(
            @RequestHeader(name = "Idempotency-Key") @NotNull UUID idempotencyKey,
            @RequestHeader(name = "traceparent", required = false) String traceparent,
            @Valid @RequestBody EncryptRequest request
    ) {
        return cryptoService.encrypt(request, idempotencyKey, traceparent);
    }

    @PostMapping(
            path = "/crypto/decrypt",
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    @PreAuthorize("hasAuthority('SCOPE_spip.decrypt') and hasRole('SPIP_AGENT')")
    public DecryptResponse decrypt(
            @RequestHeader(name = "traceparent", required = false) String traceparent,
            @Valid @RequestBody DecryptRequest request
    ) {
        return cryptoService.decrypt(request, traceparent);
    }
}
```

### 9.3 Deployment Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spip-platform
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spip-platform
  template:
    metadata:
      labels:
        app: spip-platform
    spec:
      containers:
        - name: spip-platform
          image: data4circ/spip-platform:1.0.0
          ports:
            - name: https
              containerPort: 8443
          env:
            - name: SPIP_HTTP_PORT
              value: "8443"
            - name: SPIP_BASE_PATH
              value: "/api/v1"
            - name: SPIP_OIDC_ISSUER
              value: "https://<idp-fqdn>/realms/<realm>"
            - name: SPIP_OIDC_JWKS_URI
              value: "https://<idp-fqdn>/realms/<realm>/protocol/openid-connect/certs"
            - name: SPIP_KEY_VAULT_URL
              value: "https://<spip-key-vault-fqdn>"
          volumeMounts:
            - name: tls
              mountPath: /etc/spip/tls
              readOnly: true
      volumes:
        - name: tls
          secret:
            secretName: spip-platform-tls

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spip-agent
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spip-agent
  template:
    metadata:
      labels:
        app: spip-agent
    spec:
      containers:
        - name: spip-agent
          image: data4circ/spip-agent:1.0.0
          env:
            - name: SPIP_PLATFORM_BASE_URL
              value: "https://<spip-platform-fqdn>/api/v1"
            - name: SPIP_OIDC_TOKEN_URL
              value: "https://<idp-fqdn>/realms/<realm>/protocol/openid-connect/token"
            - name: SPIP_OIDC_CLIENT_ID
              value: "<client-id>"
            - name: SPIP_OIDC_CLIENT_SECRET
              valueFrom:
                secretKeyRef:
                  name: spip-agent-oidc
                  key: client-secret
            - name: SPIP_MTLS_CERT_PATH
              value: "/etc/spip/tls/tls.crt"
            - name: SPIP_MTLS_KEY_PATH
              value: "/etc/spip/tls/tls.key"
          volumeMounts:
            - name: tls
              mountPath: /etc/spip/tls
              readOnly: true
      volumes:
        - name: tls
          secret:
            secretName: spip-agent-tls
```

### 9.4 Observability and Tracing

| Attribute | Specification |
|-----------|---------------|
| Trace ID Source | W3C Trace Context traceparent header; fallback X-Request-ID. |
| Health Check | GET /health/live returns HTTP 200 for liveness. Response body: {"status":"UP"}. |
| Readiness | GET /health/ready returns HTTP 200 when mandatory dependencies are reachable (OIDC JWKS, policy store, SPIP Key Vault). |
| Metrics Endpoint | GET /metrics returns Prometheus-format metrics (selected examples: spip_crypto_encrypt_seconds, spip_crypto_decrypt_seconds, spip_auth_denied_total). |
| Log Format | JSON structured logging including timestamp, severity, service name, requestId, correlationId, trace identifiers, policyId, and outcome (success or errorCode). |

### 9.5 Configuration and Environment Variables

| Env Variable / Key | Default | Required | Description |
|--------------------|---------|----------|-------------|
| SPIP_PLATFORM_BASE_URL | Base URL of SPIP Platform API (including /api/v1). | N/A | Yes (SPIP Agent) |
| SPIP_OIDC_TOKEN_URL | OAuth 2.0 token endpoint for client credentials flow. | N/A | Yes (SPIP Agent) |
| SPIP_OIDC_CLIENT_ID | OAuth 2.0 client identifier for SPIP Agent service account. | N/A | Yes (SPIP Agent) |
| SPIP_OIDC_CLIENT_SECRET | OAuth 2.0 client secret for SPIP Agent service account. | N/A | Yes (SPIP Agent) |
| SPIP_MTLS_CERT_PATH | Filesystem path to SPIP Agent client certificate (PEM). | N/A | Yes (SPIP Agent) |
| SPIP_MTLS_KEY_PATH | Filesystem path to SPIP Agent private key (PEM). | N/A | Yes (SPIP Agent) |
| SPIP_TLS_CA_BUNDLE_PATH | Filesystem path to CA bundle used to validate SPIP Platform certificate. | System trust store | No |
| SPIP_CONNECT_TIMEOUT_SECONDS | HTTPS connection timeout. | 3 | No |
| SPIP_READ_TIMEOUT_SECONDS | HTTPS read timeout for encrypt/decrypt operations. | 10 | No |
| SPIP_RETRY_MAX_ATTEMPTS | Maximum number of retries for retryable requests. | 5 | No |
| SPIP_CB_FAILURE_THRESHOLD | Circuit breaker failure threshold (count of failures). | 10 | No |
| SPIP_CB_OPEN_INTERVAL_SECONDS | Circuit breaker open interval before half-open transition. | 60 | No |
| SPIP_LOG_LEVEL | Application log level. | INFO | No |
| SPIP_OIDC_ISSUER | OIDC issuer identifier used by SPIP Platform for token validation. | N/A | Yes (SPIP Platform) |
| SPIP_OIDC_JWKS_URI | OIDC JSON Web Key Set (JWKS) URI used by SPIP Platform. | N/A | Yes (SPIP Platform) |
| SPIP_KEY_VAULT_URL | Endpoint URL of SPIP Key Vault. | N/A | Yes (SPIP Platform) |
| SPIP_POLICY_DB_URL | Policy persistence connection string. | N/A | Yes (SPIP Platform) |

## 10. Requirements Traceability Matrix

| SRS ID | Requirement | Interface Feature | Verification Method |
|--------|-------------|------------------|---------------------|
| SRS-1-1 | System integration with SPIP infrastructure stack for policy creation and key management. | Section 5.1 (policy lifecycle endpoints); Section 5.1 (key issuance endpoint) | Test (TC-01) |
| SRS-1-2 | Implementation of attribute-based encryption policies. | Section 6.2 (policy language); POST /crypto/encrypt; POST /crypto/decrypt | Analysis + Test (TC-02) |
| SRS-1-3 | Key generation and distribution through SPIP Key Vault. | POST /keys/issue; Section 6.1.6-6.1.7 | Test (TC-03) |
| SRS-1-4 | Encryption of document content before storage or distribution. | POST /crypto/encrypt; Section 6.1.1-6.1.2 | Test (TC-04) |
| SRS-1-5 | Decryption of document content for authorised data consumers. | POST /crypto/decrypt; Section 6.1.3-6.1.4 | Test (TC-05) |
| SRS-1-10 | Usage tracking and audit logging. | Section 2.2 (audit logging); Section 9.4 (logging and metrics) | Test (TC-06) |
| SRS-1-11 | Enforcement of data-sharing contracts and usage restrictions. | Policy evaluation on decryption; Section 7.2; Section 6.2 | Test (TC-07) |
| SRS-1-14 | Definition of granular access policies. | PolicyDefinition schema; policy lifecycle endpoints | Test (TC-08) |
| SRS-1-19 | User authentication mechanism. | Section 7.1 (OAuth 2.0/OIDC); Section 4.1 | Test (TC-09) |
| SRS-1-20 | Role-based access control. | Section 7.2 (roles and scopes) | Test (TC-10) |
| SRS-1-22 | Response time <= 3 s under normal load. | Section 8 (performance requirements); Section 4.2 (timeouts) | Test (TC-11) |
| SRS-1-23 | Encryption for data in transit. | Section 7.3 (TLS 1.3 and mTLS) | Analysis (TC-12) |
| SRS-1-24 | Availability >= 99.5%. | Section 8 (availability); Section 9.4 (health endpoints) | Analysis (TC-13) |

## 11. Acceptance Criteria

| AC ID | Criterion | Test Method | SRS Ref |
|-------|----------|-------------|---------|
| AC-01 | Authenticated endpoints reject requests with missing or invalid access token or missing client certificate and return HTTP 401. | Integration test | SRS-1-19, SRS-1-23 |
| AC-02 | Decryption requests that do not satisfy ABE policy constraints return HTTP 403 with an RFC 9457 problem response. | Negative functional test | SRS-1-2, SRS-1-20 |
| AC-03 | Encryption returns ciphertext and an encryption envelope for a valid policy. Decryption returns original plaintext for an authorised principal. | Functional test | SRS-1-4, SRS-1-5 |
| AC-04 | Key issuance returns key material with explicit validity period for authorised principals and rejects unauthorised principals. | Integration test | SRS-1-3, SRS-1-20 |
| AC-05 | Encrypt and decrypt operations satisfy a 95th percentile response time of 3 s or less for a 1 MiB payload under normal load. | Performance test | SRS-1-22 |
| AC-06 | SPIP Platform availability is at least 99.5% measured on a monthly basis. | Operational measurement | SRS-1-24 |
| AC-07 | Audit records include requestId, policyId, principal identifier, outcome, and timestamp for each cryptographic operation. | Log inspection | SRS-1-10 |
| AC-08 | All traffic between SPIP Agent and SPIP Platform uses TLS 1.3 with mTLS. | Security configuration review | SRS-1-23 |

## 12. References

[1] D2.2 DATA4CIRC Requirements and Specifications (RWTH_R_WP2_2.2_V1.0) and Appendix (RWTH_R_WP2_appendix)
[2] D4.1 DATA4CIRC Platform Architecture and Open-Source Protocols (RWTH_R_WP4_4.1_V1.0)
[3] DATA4CIRC ICD Catalogue T4.2 (ICD_Catalogue_T4.2_aligned_v2)
[4] DATA4CIRC ICD Authoring Manual v2.1 (DATA4CIRC_ICD_Authoring_Manual_aligned_v2.1)
[5] OpenAPI Specification 3.1.0 (OpenAPI Initiative)
[6] OAuth 2.0 Authorization Framework (IETF RFC 6749) and OpenID Connect Core 1.0 (OpenID Foundation)
[7] Problem Details for HTTP APIs (IETF RFC 9457)
[8] The Transport Layer Security (TLS) Protocol Version 1.3 (IETF RFC 8446); W3C Trace Context; JSON Schema 2020-12

## 13. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.9 | 15 December 2025 | DATA4CIRC Task 4.2 | Internal review draft. |
| 1.0 | 30 December 2025 | DATA4CIRC Task 4.2 | Initial release of ICD-06 defining SPIP Governance Services API between SPIP Platform and SPIP Agent, including OpenAPI specification, data structures, security profile, RTM, and test cases. |

---

## Annex A: Sequence Diagrams

Sequence diagram A-1: Policy lifecycle management (create)

```
SPIP Agent            SPIP Platform               Policy Store
    |  POST /policies (PolicyDefinition)                |
    |-------------------------------------------------->|
    |                      persist policy                |
    |                                  ---------------->|
    |                                  <----------------|
    |                      201 Created (PolicyDefinition)|
    |<--------------------------------------------------|
```

Sequence diagram A-2: Encryption

```
SPIP Agent            SPIP Platform               Policy Store           SPIP Key Vault          Audit Log
    |  POST /crypto/encrypt (EncryptRequest)           |                     |                     |
    |------------------------------------------------->|                     |                     |
    |                    resolve policyRef              |                     |                     |
    |                                  ---------------->|                     |                     |
    |                                  <----------------|                     |                     |
    |                    obtain crypto material                               |                     |
    |--------------------------------------------------------------->|        |                     |
    |<---------------------------------------------------------------|        |                     |
    |                    generate DEK and encrypt payload                                   |
    |                    wrap DEK under ABE policy                                         |
    |                    record audit event                                                   |
    |--------------------------------------------------------------------------------------->|
    |<---------------------------------------------------------------------------------------|
    |  200 OK (EncryptResponse: ciphertext + envelope)                                       |
    |<-------------------------------------------------|
```

Sequence diagram A-3: Decryption

```
SPIP Agent            SPIP Platform               Policy Store           SPIP Key Vault          Audit Log
    |  POST /crypto/decrypt (DecryptRequest)           |                     |                     |
    |------------------------------------------------->|                     |                     |
    |                 validate token and roles          |                     |                     |
    |                 evaluate ABE policy against attributes               |                     |
    |                 obtain attribute key material                           |                     |
    |--------------------------------------------------------------->|        |                     |
    |<---------------------------------------------------------------|        |                     |
    |                 unwrap DEK and decrypt payload                                     |
    |                 record audit event                                                   |
    |--------------------------------------------------------------------------------------->|
    |<---------------------------------------------------------------------------------------|
    |  200 OK (DecryptResponse: plaintext)                                                 |
    |<-------------------------------------------------|
```

Sequence diagram A-4: Attribute key issuance

```
SPIP Agent            SPIP Platform               SPIP Key Vault          Audit Log
    |  POST /keys/issue (KeyIssueRequest)              |                     |
    |------------------------------------------------->|                     |
    |                 validate token and roles          |                     |
    |                 derive / issue attribute key material               |
    |--------------------------------------------------------------->|        |
    |<---------------------------------------------------------------|        |
    |                 record audit event                               |
    |--------------------------------------------------------------->|
    |<---------------------------------------------------------------|
    |  200 OK (KeyIssueResponse: keyMaterial + expiry)                |
    |<-------------------------------------------------|
```

Annex A provides sequence diagrams for policy management, encryption, decryption, and key issuance flows.

---

## Annex B: Complete API Schema

```yaml
openapi: 3.1.0
info:
  title: DATA4CIRC SPIP Governance Services API
  version: 1.0.0
  description: >
    Governance services API between the SPIP Agent (client) and the SPIP Platform (server).
    Transport security uses HTTPS with TLS 1.3 and mutual TLS. Authentication and authorisation use
    OAuth 2.0 client credentials with OpenID Connect JWT access tokens.
servers:
  - url: https://{spipHost}/api/v1
    variables:
      spipHost:
        default: spip.example.com

security:
  - oauth2:
      - spip.read

paths:
  /capabilities:
    get:
      summary: Retrieve supported algorithms and policy language capabilities
      operationId: getCapabilities
      security:
        - oauth2:
            - spip.read
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
      responses:
        '200':
          description: Capabilities response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CapabilitiesResponse'
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '500':
          $ref: '#/components/responses/InternalError'

  /crypto/encrypt:
    post:
      summary: Encrypt payload under an ABE policy
      operationId: encrypt
      security:
        - oauth2:
            - spip.encrypt
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
        - $ref: '#/components/parameters/IdempotencyKey'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/EncryptRequest'
      responses:
        '200':
          description: Encryption successful
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/EncryptResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '409':
          $ref: '#/components/responses/Conflict'
        '413':
          $ref: '#/components/responses/PayloadTooLarge'
        '422':
          $ref: '#/components/responses/UnprocessableEntity'
        '429':
          $ref: '#/components/responses/TooManyRequests'
        '500':
          $ref: '#/components/responses/InternalError'
        '503':
          $ref: '#/components/responses/ServiceUnavailable'

  /crypto/decrypt:
    post:
      summary: Decrypt payload subject to ABE policy evaluation
      operationId: decrypt
      security:
        - oauth2:
            - spip.decrypt
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/DecryptRequest'
      responses:
        '200':
          description: Decryption successful
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DecryptResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
        '422':
          $ref: '#/components/responses/UnprocessableEntity'
        '429':
          $ref: '#/components/responses/TooManyRequests'
        '500':
          $ref: '#/components/responses/InternalError'
        '503':
          $ref: '#/components/responses/ServiceUnavailable'

  /policies:
    get:
      summary: List policies
      operationId: listPolicies
      security:
        - oauth2:
            - spip.policies:read
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
        - name: page
          in: query
          schema:
            type: integer
            minimum: 0
          required: false
        - name: pageSize
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 200
          required: false
        - name: includeRevoked
          in: query
          schema:
            type: boolean
          required: false
      responses:
        '200':
          description: Policy list
          content:
            application/json:
              schema:
                type: object
                required: [items, page, pageSize, total]
                properties:
                  items:
                    type: array
                    items:
                      $ref: '#/components/schemas/PolicyDefinition'
                  page:
                    type: integer
                  pageSize:
                    type: integer
                  total:
                    type: integer
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '500':
          $ref: '#/components/responses/InternalError'
    post:
      summary: Create policy
      operationId: createPolicy
      security:
        - oauth2:
            - spip.policies:write
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
        - $ref: '#/components/parameters/IdempotencyKey'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PolicyDefinition'
      responses:
        '201':
          description: Policy created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyDefinition'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '409':
          $ref: '#/components/responses/Conflict'
        '422':
          $ref: '#/components/responses/UnprocessableEntity'
        '500':
          $ref: '#/components/responses/InternalError'

  /policies/{policyId}:
    parameters:
      - name: policyId
        in: path
        required: true
        schema:
          type: string
          format: uuid
    get:
      summary: Retrieve policy
      operationId: getPolicy
      security:
        - oauth2:
            - spip.policies:read
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
      responses:
        '200':
          description: Policy
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyDefinition'
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
        '500':
          $ref: '#/components/responses/InternalError'
    put:
      summary: Update policy
      operationId: updatePolicy
      security:
        - oauth2:
            - spip.policies:write
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
        - $ref: '#/components/parameters/IdempotencyKey'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PolicyDefinition'
      responses:
        '200':
          description: Updated policy
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyDefinition'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
        '409':
          $ref: '#/components/responses/Conflict'
        '422':
          $ref: '#/components/responses/UnprocessableEntity'
        '500':
          $ref: '#/components/responses/InternalError'
    delete:
      summary: Revoke policy
      operationId: revokePolicy
      security:
        - oauth2:
            - spip.policies:write
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
        - $ref: '#/components/parameters/IdempotencyKey'
      responses:
        '204':
          description: Policy revoked
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
        '500':
          $ref: '#/components/responses/InternalError'

  /keys/issue:
    post:
      summary: Issue attribute key material
      operationId: issueKey
      security:
        - oauth2:
            - spip.keys:issue
      parameters:
        - $ref: '#/components/parameters/TraceParent'
        - $ref: '#/components/parameters/XRequestId'
        - $ref: '#/components/parameters/IdempotencyKey'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/KeyIssueRequest'
      responses:
        '200':
          description: Key issued
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/KeyIssueResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorised'
        '403':
          $ref: '#/components/responses/Forbidden'
        '429':
          $ref: '#/components/responses/TooManyRequests'
        '500':
          $ref: '#/components/responses/InternalError'
        '503':
          $ref: '#/components/responses/ServiceUnavailable'

  /health/live:
    get:
      summary: Liveness probe
      operationId: liveness
      responses:
        '200':
          description: Service is live
          content:
            application/json:
              schema:
                type: object
                required: [status]
                properties:
                  status:
                    type: string
                    enum: [UP]

  /health/ready:
    get:
      summary: Readiness probe
      operationId: readiness
      responses:
        '200':
          description: Service is ready
          content:
            application/json:
              schema:
                type: object
                required: [status]
                properties:
                  status:
                    type: string
                    enum: [UP]
        '503':
          $ref: '#/components/responses/ServiceUnavailable'

  /metrics:
    get:
      summary: Metrics endpoint (Prometheus format)
      operationId: metrics
      security:
        - oauth2:
            - spip.monitor
      responses:
        '200':
          description: Metrics payload
          content:
            text/plain:
              schema:
                type: string

components:
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://<idp-fqdn>/realms/<realm>/protocol/openid-connect/token
          scopes:
            spip.read: Read capabilities
            spip.encrypt: Encrypt payloads
            spip.decrypt: Decrypt payloads
            'spip.policies:read': Read policies
            'spip.policies:write': Create and update policies
            'spip.keys:issue': Issue attribute keys
            spip.monitor: Access metrics

  parameters:
    TraceParent:
      name: traceparent
      in: header
      required: false
      schema:
        type: string
      description: W3C Trace Context header.
    XRequestId:
      name: X-Request-ID
      in: header
      required: false
      schema:
        type: string
        format: uuid
      description: Client-generated correlation identifier.
    IdempotencyKey:
      name: Idempotency-Key
      in: header
      required: true
      schema:
        type: string
        format: uuid
      description: Idempotency key used for safe retries.

  responses:
    BadRequest:
      description: Bad request
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    Unauthorised:
      description: Unauthorised
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    Forbidden:
      description: Forbidden
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    NotFound:
      description: Not found
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    Conflict:
      description: Conflict
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    PayloadTooLarge:
      description: Payload too large
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    UnprocessableEntity:
      description: Unprocessable entity
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    TooManyRequests:
      description: Too many requests
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    InternalError:
      description: Internal server error
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    ServiceUnavailable:
      description: Service unavailable
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'

  schemas:
    EncryptRequest:
      type: object
      additionalProperties: false
      required:
        - requestId
        - payload
        - payloadContentType
        - payloadEncoding
      properties:
        requestId:
          type: string
          format: uuid
        payload:
          type: string
          contentEncoding: base64
        payloadContentType:
          type: string
        payloadEncoding:
          type: string
          enum: [base64]
        policyRef:
          $ref: '#/components/schemas/PolicyRef'
        policyInline:
          $ref: '#/components/schemas/PolicyDefinition'
        encryptionContext:
          type: object
          additionalProperties:
            type: string
        aad:
          type: string
          contentEncoding: base64
      oneOf:
        - required: [policyRef]
        - required: [policyInline]

    EncryptResponse:
      type: object
      additionalProperties: false
      required:
        - requestId
        - ciphertext
        - ciphertextEncoding
        - envelope
      properties:
        requestId:
          type: string
          format: uuid
        ciphertext:
          type: string
          contentEncoding: base64
        ciphertextEncoding:
          type: string
          enum: [base64]
        envelope:
          $ref: '#/components/schemas/EncryptionEnvelope'

    DecryptRequest:
      type: object
      additionalProperties: false
      required:
        - requestId
        - ciphertext
        - ciphertextEncoding
        - envelope
      properties:
        requestId:
          type: string
          format: uuid
        ciphertext:
          type: string
          contentEncoding: base64
        ciphertextEncoding:
          type: string
          enum: [base64]
        envelope:
          $ref: '#/components/schemas/EncryptionEnvelope'
        decryptionContext:
          type: object
          additionalProperties:
            type: string
        aad:
          type: string
          contentEncoding: base64

    DecryptResponse:
      type: object
      additionalProperties: false
      required:
        - requestId
        - payload
        - payloadContentType
        - payloadEncoding
        - policyId
        - policyVersion
      properties:
        requestId:
          type: string
          format: uuid
        payload:
          type: string
          contentEncoding: base64
        payloadContentType:
          type: string
        payloadEncoding:
          type: string
          enum: [base64]
        policyId:
          type: string
          format: uuid
        policyVersion:
          type: integer
        auditEventId:
          type: string
          format: uuid

    EncryptionEnvelope:
      type: object
      additionalProperties: false
      required:
        - envelopeVersion
        - contentAlgorithm
        - keyAlgorithm
        - policyId
        - policyVersion
        - iv
        - tag
        - abeCiphertext
        - issuedAt
      properties:
        envelopeVersion:
          type: integer
          minimum: 1
        contentAlgorithm:
          type: string
          enum: [AES-256-GCM]
        keyAlgorithm:
          type: string
          enum: [CP-ABE]
        policyId:
          type: string
          format: uuid
        policyVersion:
          type: integer
        iv:
          type: string
          contentEncoding: base64
        tag:
          type: string
          contentEncoding: base64
        abeCiphertext:
          type: string
          contentEncoding: base64
        issuedAt:
          type: string
          format: date-time

    PolicyRef:
      type: object
      additionalProperties: false
      required: [policyId, policyVersion]
      properties:
        policyId:
          type: string
          format: uuid
        policyVersion:
          type: integer
          minimum: 1

    PolicyDefinition:
      type: object
      additionalProperties: false
      required:
        - name
        - expression
        - version
        - status
      properties:
        policyId:
          type: string
          format: uuid
        name:
          type: string
          minLength: 1
        description:
          type: string
        expression:
          type: string
          minLength: 1
        attributeNamespace:
          type: string
          format: uri
        version:
          type: integer
          minimum: 1
        status:
          type: string
          enum: [ACTIVE, REVOKED]
        validFrom:
          type: string
          format: date-time
        validUntil:
          type: string
          format: date-time
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time

    KeyIssueRequest:
      type: object
      additionalProperties: false
      required:
        - requestId
        - keyPurpose
      properties:
        requestId:
          type: string
          format: uuid
        keyPurpose:
          type: string
          enum: [DECRYPT, ENCRYPT]
        ttlSeconds:
          type: integer
          minimum: 60
        policyId:
          type: string
          format: uuid
        audience:
          type: string
          format: uri

    KeyIssueResponse:
      type: object
      additionalProperties: false
      required:
        - requestId
        - keyId
        - keyMaterial
        - keyMaterialEncoding
        - issuedAt
        - expiresAt
      properties:
        requestId:
          type: string
          format: uuid
        keyId:
          type: string
          format: uuid
        keyMaterial:
          type: string
          contentEncoding: base64
        keyMaterialEncoding:
          type: string
          enum: [base64]
        issuedAt:
          type: string
          format: date-time
        expiresAt:
          type: string
          format: date-time

    CapabilitiesResponse:
      type: object
      additionalProperties: false
      required: [serviceName, apiVersion, supportedContentAlgorithms, supportedKeyAlgorithms, policyLanguageVersion, maxPayloadSizeBytes]
      properties:
        serviceName:
          type: string
          const: SPIP Platform
        apiVersion:
          type: string
          pattern: '^[0-9]+\.[0-9]+\.[0-9]+$'
        supportedContentAlgorithms:
          type: array
          items:
            type: string
        supportedKeyAlgorithms:
          type: array
          items:
            type: string
        policyLanguageVersion:
          type: string
        maxPayloadSizeBytes:
          type: integer
          minimum: 1

    ProblemDetails:
      type: object
      additionalProperties: true
      required: [type, title, status]
      properties:
        type:
          type: string
          format: uri
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
        instance:
          type: string
          format: uri
        errorCode:
          type: string
        correlationId:
          type: string
          format: uuid
        retryable:
          type: boolean
        validationErrors:
          type: array
          items:
            type: object
            additionalProperties: false
            required: [field, message]
            properties:
              field:
                type: string
              message:
                type: string
```

Annex B contains the normative OpenAPI 3.1 specification for the SPIP Governance Services API.

---

## Annex C: Test Cases

TC-01: Policy lifecycle management
Related requirements: SRS-1-1, SRS-1-14
Preconditions:
  - Valid OAuth 2.0 access token with scope spip.policies:write and role SPIP_POLICY_ADMIN.
  - mTLS client certificate trusted by SPIP Platform.
Test steps:
  1) Submit POST /policies with a PolicyDefinition containing name, expression, version=1, status=ACTIVE.
  2) Verify HTTP 201 response and the presence of policyId, createdAt, updatedAt.
  3) Submit GET /policies/{policyId} and verify HTTP 200 and payload equality for immutable fields.
  4) Submit PUT /policies/{policyId} with version increment and modified expression.
  5) Verify HTTP 200 response and updatedAt change.
  6) Submit DELETE /policies/{policyId} and verify HTTP 204 response.
  7) Submit GET /policies/{policyId} and verify status=REVOKED or HTTP 404 (deployment profile).
Expected results:
  - Policy objects persist across requests and reflect lifecycle transitions.
  - Optimistic concurrency control rejects stale versions with HTTP 409.

TC-02: Policy expression validation
Related requirements: SRS-1-2
Preconditions:
  - Valid OAuth 2.0 access token with scope spip.policies:write.
Test steps:
  1) Submit POST /policies with an invalid expression (syntax error).
Expected results:
  - HTTP 422 response with application/problem+json.
  - ProblemDetails.errorCode indicates policy_parse_error and includes validationErrors.

TC-03: Attribute key issuance
Related requirements: SRS-1-3
Preconditions:
  - Valid OAuth 2.0 access token with scope spip.keys:issue and role SPIP_AGENT.
Test steps:
  1) Submit POST /keys/issue with KeyIssueRequest containing requestId, keyPurpose=DECRYPT, ttlSeconds.
Expected results:
  - HTTP 200 response with keyId, keyMaterial (base64), issuedAt, expiresAt.
  - expiresAt - issuedAt equals ttlSeconds within server tolerance.

TC-04: Encryption operation
Related requirements: SRS-1-4
Preconditions:
  - Valid OAuth 2.0 access token with scope spip.encrypt and role SPIP_AGENT.
  - Existing ACTIVE policyId and policyVersion.
Test steps:
  1) Submit POST /crypto/encrypt with EncryptRequest including payload and policyRef.
Expected results:
  - HTTP 200 response with ciphertext and encryption envelope.
  - Envelope includes contentAlgorithm=AES-256-GCM and keyAlgorithm=CP-ABE.

TC-05: Decryption operation (authorised)
Related requirements: SRS-1-5
Preconditions:
  - Valid OAuth 2.0 access token with scope spip.decrypt and role SPIP_AGENT.
  - Access token contains attributes satisfying policy expression.
  - Ciphertext and envelope produced by TC-04.
Test steps:
  1) Submit POST /crypto/decrypt with DecryptRequest containing ciphertext and envelope.
Expected results:
  - HTTP 200 response with payload.
  - Decoded payload equals original plaintext from TC-04.

TC-06: Audit logging for cryptographic operations
Related requirements: SRS-1-10
Preconditions:
  - Audit logging enabled in SPIP Platform deployment.
Test steps:
  1) Execute TC-04 and TC-05.
  2) Retrieve platform logs or audit store entries for requestId values.
Expected results:
  - Audit records exist for encrypt and decrypt operations.
  - Audit record fields include requestId, policyId, principal identifier, outcome, and timestamp.

TC-07: Decryption operation (unauthorised)
Related requirements: SRS-1-2, SRS-1-11
Preconditions:
  - Valid OAuth 2.0 access token with scope spip.decrypt and role SPIP_AGENT.
  - Access token attributes do not satisfy policy expression.
  - Ciphertext and envelope produced by TC-04.
Test steps:
  1) Submit POST /crypto/decrypt with DecryptRequest.
Expected results:
  - HTTP 403 response with application/problem+json.
  - ProblemDetails.type equals urn:data4circ:spip:error:forbidden.
  - No plaintext is returned.

TC-08: Granular policy constraints
Related requirements: SRS-1-14
Preconditions:
  - PolicyDefinition expression combines multiple attributes (AND/OR with parentheses).
Test steps:
  1) Create policy with expression requiring at least two attributes.
  2) Verify decryption succeeds for principals satisfying constraints and fails otherwise.
Expected results:
  - Policy evaluation enforces the stated Boolean constraints.

TC-09: Authentication enforcement
Related requirements: SRS-1-19
Preconditions:
  - None.
Test steps:
  1) Submit POST /crypto/encrypt without Authorization header.
  2) Submit POST /crypto/encrypt with expired token.
  3) Submit POST /crypto/encrypt without client certificate.
Expected results:
  - HTTP 401 response for token failures.
  - TLS handshake failure or HTTP 401 for missing client certificate (deployment profile).
  - No cryptographic operation is executed.

TC-10: Role-based access control enforcement
Related requirements: SRS-1-20
Preconditions:
  - Valid OAuth 2.0 token lacking required scopes or roles.
Test steps:
  1) Submit POST /policies with a token missing spip.policies:write.
Expected results:
  - HTTP 403 response with application/problem+json.

TC-11: Performance test
Related requirements: SRS-1-22
Preconditions:
  - Representative deployment environment and monitoring instrumentation.
Test steps:
  1) Execute 50 requests/s for POST /crypto/encrypt and POST /crypto/decrypt with 1 MiB payload.
  2) Measure 95th percentile response time.
Expected results:
  - 95th percentile response time <= 3 s.

TC-12: Transport security verification
Related requirements: SRS-1-23
Preconditions:
  - Access to network capture or TLS configuration audit tools.
Test steps:
  1) Establish connection from SPIP Agent to SPIP Platform.
  2) Inspect negotiated TLS version and cipher suite.
Expected results:
  - TLS version equals 1.3.
  - Cipher suite belongs to the configured allow-list.
  - Client certificate authentication is enforced.

TC-13: Availability measurement
Related requirements: SRS-1-24
Preconditions:
  - Monitoring system configured for /health/ready and /health/live endpoints.
Test steps:
  1) Collect service uptime measurements over a monthly reporting period.
Expected results:
  - Availability >= 99.5%.

Annex C provides test case specifications aligned with acceptance criteria and requirements traceability.

---

## Annex D: Quality Checklist

| Check | Criterion | Section |
|-------|----------|---------|
| Yes | Units of measure specified for all numerical fields | Section 6.1 |
| N/A | Semantic IDs (IRDIs) provided for AAS-compliant fields | Section 6.1 |
| Yes | Environment variables listed for DevOps deployment | Section 9.5 |
| Yes | Circuit breaker thresholds defined for resilience | Section 4.2 |
| Yes | PII fields flagged and retention policies defined | Section 6.3 |
| N/A | ODRL policies defined for dataspace interfaces | Section 7.4 |
| N/A | MQTT topics, QoS, and LWT defined for IoT interfaces | Section 5.3, 9.4 |
| Yes | Error handling appropriate for protocol (RFC 9457 or DLQ) | Section 2.3 |
| Yes | Health check mechanism defined (HTTP endpoint or MQTT LWT) | Section 9.4 |
| Yes | Interface dependencies and versioning documented | Section 1.4 |
| Yes | British English and IEEE style followed throughout | All sections |
| Yes | No subjunctive mood, personal pronouns, or filler words | All sections |
| Yes | Abbreviations defined once and listed in Section 3 | Section 3 |
| Yes | Performance targets use specific numerical values | Section 8 |
