# Interface Control Document (ICD)-07: Governance services integration

**DATA4CIRC Manufacturing Dataspace (DS4CIRC) Portal <-> Secure Policy Information Point (SPIP) Platform**

---

| Attribute | Value |
|-----------|-------|
| **Version** | 1.0 |
| **Date** | 30 December 2025 |
| **Work Package** | WP3 |
| **Author(s)** | DATA4CIRC WP3 Technical Team (NTT, RWTH) |
| **Provider Owner** | SPIP Platform Service Owner (WP3) |
| **Consumer Owner** | DS4CIRC Portal Service Owner (WP3) |
| **Reviewer** | DATA4CIRC Architecture and Security Review Board |
| **Status** | Approved |

---

## Document Completion Guidelines

This section provides mandatory writing conventions and completion instructions for all Interface Control Documents within the DATA4CIRC project. All contributors shall adhere to these guidelines to ensure consistency, scientific rigour, and compliance with EU Horizon Europe deliverable standards.

### Writing Style Requirements

[WRITING GUIDELINE] MANDATORY: All content shall be written in formal, scientific style conforming to IEEE conventions. The following rules apply throughout the document.

### Abbreviation Rules

[WRITING GUIDELINE] Each abbreviation shall be defined exactly once at first use in the format: Full Term (ABBR). Subsequently, only the abbreviation is used. All abbreviations shall also appear in Section 3 (Abbreviations).

---

## 1. Interface Overview

### 1.1 Purpose

ICD-07 defines the synchronous Hypertext Transfer Protocol (HTTP) Representational State Transfer (REST) application programming interface (API) between the DATA4CIRC Manufacturing Dataspace (DS4CIRC) Portal and the Secure Policy Information Point (SPIP) Platform. The interface enables governance service integration by exposing SPIP Platform capabilities for (i) attribute-based encryption (ABE) policy lifecycle management, (ii) organisational attribute set validation and key lifecycle operations via SPIP Key Vault, and (iii) cryptographic operations (encryption and decryption) executed under validated policies. The interface supports cross-organisational policy creation and key management (SRS-1-1), ABE policy specification for fine-grained access control (SRS-1-2), key generation and distribution (SRS-1-3), and cryptographic services for encryption and decryption (SRS-1-4, SRS-1-5). Software Requirements Specification (SRS) references trace requirements defined in D2.2. Work Package (WP) 3 is responsible for this interface definition.

### 1.2 Communicating Components

| Attribute | Component A | Component B |
|-----------|-------------|-------------|
| **Name** | DS4CIRC Portal | SPIP Platform |
| **Role** | Consumer (client) | Provider (server) |
| **Work Package** | WP3 | WP3 |
| **Responsible Partner** | NTT | NTT |

### 1.3 Architectural Context

The interface is positioned in the governance domain described in D4.1 and is classified as a Governance interface in the ICD Catalogue. The DS4CIRC Portal uses this interface for policy administration and cryptographic governance operations, while the SPIP Platform provides enforcement and key vault capabilities. Upstream dependencies include the Keycloak Identity Provider (IdP) and DS4CIRC Portal authentication flows. Downstream dependencies include SPIP Platform interactions with SPIP Agent (ICD-06) and SPIP Agent interactions with the Dataspace Connector (ICD-05), as well as DS4CIRC governance workflows and policy publication mechanisms.

### 1.4 Interface Dependencies and Lifecycle

| Attribute | Specification |
|-----------|---------------|
| **Prerequisites** | Keycloak Identity Provider (IdP) configured for DS4CIRC Portal client credentials and/or authorisation code flows. <br> Open Authorisation (OAuth) 2.0 / OpenID Connect (OIDC) scopes and roles provisioned for governance operations. <br> SPIP Platform base URL reachable via secured network path; Domain Name System (DNS) and service discovery configured. <br> Transport Layer Security (TLS) trust anchors and certificate chains deployed for server authentication; optional mutual TLS (mTLS) client credentials. <br> SPIP Key Vault initialised; cryptographic algorithms enabled; key wrapping certificates registered per client. <br> JSON Web Token (JWT) claim mappings configured from JWT claims to SPIP attribute namespaces and validated. |
| **Versioning Strategy** | Uniform Resource Identifier (URI)-based versioning (/api/v{major}); Semantic Versioning 2.0.0 applied to API artefacts; backwards compatibility preserved within a major version. |
| **Deprecation Policy** | Deprecation and Sunset headers published for deprecated resources and behaviours; minimum 180-day deprecation window; parallel support for superseding major versions. |
| **Downstream Dependents** | SPIP Platform <-> SPIP Agent (ICD-06); SPIP Agent <-> Dataspace Connector (ICD-05); DS4CIRC governance workflows and policy publication mechanisms. |

---

## 2. Functional Description

### 2.1 Functional Capabilities

| ID | Capability | Description | SRS Reference |
|----|------------|-------------|---------------|
| FC-01 | Policy lifecycle management | Create, retrieve, update, archive, and activate ABE governance policies managed by the SPIP Platform. | SRS-1-1, SRS-1-2 |
| FC-02 | Policy validation and compilation | Validate policy syntax and semantics, including attribute namespace checks and compilation to ABE enforcement artefacts. | SRS-1-2, SRS-1-14 |
| FC-03 | Attribute and claim validation | Validate organisational and subject attributes supplied by DS4CIRC Portal against configured claim mappings and attribute schemas. | SRS-1-14, SRS-1-19 |
| FC-04 | Key lifecycle operations | Issue, rotate, and revoke encryption and decryption key material through SPIP Key Vault, subject to policy constraints. | SRS-1-1, SRS-1-3 |
| FC-05 | Encryption service | Encrypt payloads under validated SPIP policies and return ciphertext and cryptographic metadata. | SRS-1-4, SRS-1-23 |
| FC-06 | Decryption service | Decrypt payloads under validated SPIP policies using associated decryption keys and return plaintext. | SRS-1-5, SRS-1-23 |
| FC-07 | Audit and traceability | Retrieve governance audit events for policy, key, and cryptographic operations for compliance and incident response. | SRS-1-19 |
| FC-08 | Service metadata and health | Expose health, readiness, and version metadata for operational monitoring and orchestration. | SRS-1-24 |

### 2.2 Interaction Patterns

Interaction patterns are defined for governance control-plane and cryptographic service operations. Sequence diagrams are provided in Annex A.

**IP-01 Policy lifecycle management**:
1) DS4CIRC Portal obtains an access token from Keycloak using an authorised OAuth 2.0/OIDC flow.
2) DS4CIRC Portal submits a policy creation or update request to the SPIP Platform (/policies) with a correlation identifier.
3) SPIP Platform validates authorisation scope, validates policy syntax and semantics, persists the policy, and returns the policy identifier and entity tag (ETag).
4) DS4CIRC Portal persists the returned policy identifier and ETag for subsequent conditional updates.

**IP-02 Key issuance**:
1) DS4CIRC Portal submits a key issuance request to the SPIP Platform (/keys/issue) containing policy identifier and subject attributes.
2) SPIP Platform validates authorisation scope and attribute claims, derives key material in SPIP Key Vault, and returns wrapped key material.

**IP-03 Encryption and decryption**:
1) DS4CIRC Portal submits an encryption request (/crypto/encrypt) with policy identifier and base64-encoded plaintext.
2) SPIP Platform encrypts payloads under the selected policy and returns base64-encoded ciphertext.
3) DS4CIRC Portal submits a decryption request (/crypto/decrypt) with ciphertext and key reference; SPIP Platform returns base64-encoded plaintext upon successful policy evaluation.

### 2.3 Error Handling

#### 2.3.1 HTTP/REST Error Handling

For HTTP/REST interfaces, error responses shall conform to Request for Comments (RFC) 9457 (Problem Details for HTTP APIs).

| HTTP Status | Condition | Recovery Action |
|-------------|-----------|-----------------|
| 400 Bad Request | Request payload fails JavaScript Object Notation (JSON) Schema validation or contains invalid parameters. | Correct request payload; do not retry without modification. |
| 401 Unauthorized | Access token missing, expired, or fails signature, issuer, or audience validation. | Obtain a new token and retry once. |
| 403 Forbidden | Token lacks required scope or role, or attribute constraints deny access. | Provision correct role or scope, or adjust request to satisfy policy constraints. |
| 404 Not Found | Referenced resource identifier does not exist. | Validate identifier; synchronise local cache; do not retry without modification. |
| 409 Conflict | Resource state conflict (duplicate resource, concurrent update without matching ETag). | Retrieve latest representation and retry with If-Match and updated ETag. |
| 412 Precondition Failed | Conditional request fails (If-Match or If-None-Match). | Retrieve latest representation and retry with correct conditional header. |
| 413 Payload Too Large | Payload exceeds configured maximum. | Reduce payload size or use chunking strategy; retry after modification. |
| 415 Unsupported Media Type | Unsupported Content-Type or Accept header. | Set Content-Type to application/json and Accept to application/json or application/problem+json. |
| 422 Unprocessable Content | Policy compilation or validation fails despite syntactically valid JSON. | Correct policy expression or attribute references; do not retry without modification. |
| 429 Too Many Requests | Rate limit exceeded. | Retry after Retry-After; apply exponential backoff and circuit breaker policy. |
| 500 Internal Server Error | Unhandled server-side error. | Retry with exponential backoff; escalate after repeated failures. |
| 503 Service Unavailable | Service temporarily unavailable or dependency failure. | Retry with exponential backoff; open circuit breaker after threshold breaches. |

#### 2.3.2 IoT/Async Error Handling

For Message Queuing Telemetry Transport (MQTT) and asynchronous interfaces, error handling shall use dedicated error topics and Dead Letter Queue (DLQ) strategies.

| Attribute | Specification |
|-----------|---------------|
| **Error Topic** | Not applicable (synchronous HTTP/REST interface). |
| **DLQ Strategy** | Not applicable (synchronous HTTP/REST interface). |
| **Error Payload Schema** | Not applicable (synchronous HTTP/REST interface). |
| **Retry Policy** | Not applicable (synchronous HTTP/REST interface). |

---

## 3. Abbreviations

| Abbreviation | Definition |
|--------------|------------|
| AAS | Asset Administration Shell |
| ABAC | Attribute-Based Access Control |
| ABE | Attribute-Based Encryption |
| AC | Acceptance Criterion |
| API | Application Programming Interface |
| CA | Certificate Authority |
| CP-ABE | Ciphertext-Policy Attribute-Based Encryption |
| CRUD | Create, Read, Update, Delete |
| DCAT-AP | Data Catalogue Application Profile |
| DLQ | Dead Letter Queue |
| DNS | Domain Name System |
| DS4CIRC | DATA4CIRC Manufacturing Dataspace (Federated Data Space) |
| EDC | Eclipse Dataspace Connector |
| ETag | Entity Tag |
| GDPR | General Data Protection Regulation |
| HTTP | Hypertext Transfer Protocol |
| HTTPS | Hypertext Transfer Protocol Secure |
| ICD | Interface Control Document |
| IdP | Identity Provider |
| IEC CDD | International Electrotechnical Commission Common Data Dictionary |
| IRDI | International Registration Data Identifier |
| JSON | JavaScript Object Notation |
| JSON-LD | JSON for Linking Data |
| JWT | JSON Web Token |
| JWE | JSON Web Encryption |
| KP-ABE | Key-Policy Attribute-Based Encryption |
| LCA | Life Cycle Assessment |
| LEI | Legal Entity Identifier |
| LWT | Last Will and Testament |
| MQTT | Message Queuing Telemetry Transport |
| mTLS | Mutual Transport Layer Security |
| OAuth | Open Authorisation |
| OIDC | OpenID Connect |
| ODRL | Open Digital Rights Language |
| P95 | 95th percentile |
| PII | Personally Identifiable Information |
| RBAC | Role-Based Access Control |
| REST | Representational State Transfer |
| RFC | Request for Comments |
| SLO | Service Level Objective |
| SPIP | Secure Policy Information Point |
| SRS | Software Requirements Specification |
| TCP | Transmission Control Protocol |
| TLS | Transport Layer Security |
| UoM | Unit of Measure |
| URI | Uniform Resource Identifier |
| UUID | Universally Unique Identifier |
| WP | Work Package |

---

## 4. Communication Protocol

### 4.1 Protocol Stack

Transport uses Hypertext Transfer Protocol Secure (HTTPS) over Transmission Control Protocol (TCP) for confidentiality and integrity.

| Layer | Protocol | Specification |
|-------|----------|---------------|
| Application | HTTP/REST | RFC 9110 |
| Security | OAuth 2.0, TLS 1.3, optional mTLS | RFC 6749; RFC 8446 |
| Transport | HTTPS over TCP | RFC 9110 |
| Serialisation | JSON | RFC 8259; JSON Schema 2020-12 |

### 4.2 Connection Parameters

| Parameter | Value |
|-----------|-------|
| **Base URL / Broker** | https://spip.example.internal/api/v1 |
| **Port** | 443/TCP |
| **Network Zone** | Internal service network (segmented); ingress restricted to DS4CIRC Portal execution environment. |
| **Connection Timeout** | 10 seconds |
| **Read Timeout** | 60 seconds |
| **Retry Policy** | Idempotent methods (GET/HEAD) retried up to 3 attempts with exponential backoff (0.5 seconds, 1 second, 2 seconds) and full jitter; POST retried only when Idempotency-Key header is present. |
| **Circuit Breaker** | Open after 5 consecutive transport errors or 5xx responses within 30 seconds; half-open after 60 seconds; close after 3 consecutive successful probes. |
| **Firewall Rules** | Allow inbound TCP 443 from the DS4CIRC Portal service network and deny all other inbound traffic. |

[WRITING GUIDELINE] Circuit breaker patterns prevent cascading failures in distributed systems. The circuit breaker opens after a threshold of consecutive failures, blocking further requests until a reset timeout elapses.

---

## 5. API Specification

### 5.1 Endpoint Definitions

#### 5.1.1 Endpoint catalogue and conventions

| Attribute | Value |
|-----------|-------|
| **Method** | HTTP/REST |
| **Path** | /api/v1 |
| **Purpose** | SPIP governance service root for DS4CIRC Portal integration; resource endpoints specified in Table 5-1 and Annex B. |
| **Authentication** | Bearer Token (OAuth 2.0 access token) |

Resource identifiers use Universally Unique Identifier (UUID) values unless stated otherwise.

Endpoint catalogue:

| Method | Path | Operation | Authorisation (scope/role) |
|--------|------|-----------|----------------------------|
| GET | /health | Liveness probe | public (no token) or token optional |
| GET | /ready | Readiness probe | public (no token) or token optional |
| GET | /version | Service and API version metadata | spip.read |
| GET | /policies | List policies (filtered by owner, status) | spip.policy.read |
| POST | /policies | Create policy | spip.policy.write |
| GET | /policies/{policyId} | Retrieve policy | spip.policy.read |
| PUT | /policies/{policyId} | Replace policy (conditional via If-Match) | spip.policy.write |
| DELETE | /policies/{policyId} | Archive policy (soft delete) | spip.policy.write |
| POST | /policies/{policyId}/validate | Validate and compile policy | spip.policy.write |
| POST | /keys/issue | Issue encryption/decryption key material (wrapped) | spip.key.issue |
| POST | /crypto/encrypt | Encrypt payload under policy | spip.crypto.encrypt |
| POST | /crypto/decrypt | Decrypt payload under policy | spip.crypto.decrypt |
| GET | /audit/events | Query audit events | spip.audit.read |

### 5.2 Request and Response Examples

**Request Example:**

Example A: Create policy

```
POST /api/v1/policies
Content-Type: application/json
Accept: application/json
Authorization: Bearer <JWT>
X-Correlation-ID: 2f8a8a6e-8df9-4a34-9b2c-0f552fe0e0b1
Idempotency-Key: 8b176c1c-1b88-4d02-8b89-79f5b2c2c8c9

{
  "name": "dpp-material-disclosure-v1",
  "description": "ABE policy for controlled disclosure of material composition data.",
  "policyType": "CP_ABE",
  "abePolicy": {
    "scheme": "CP-ABE",
    "expression": "(org.role:manufacturer AND org.id:LEI:529900T8BM49AURSDO55) OR (org.role:auditor AND purpose:lca)"
  },
  "ownerOrgId": "LEI:529900T8BM49AURSDO55",
  "tags": ["dpp","materials","governance"]
}
```

Example B: Encrypt payload

```
POST /api/v1/crypto/encrypt
Content-Type: application/json
Accept: application/json
Authorization: Bearer <JWT>
X-Correlation-ID: 9c173c1b-7f06-4f33-a2c0-2e61d2e9e84f

{
  "policyId": "0d8b7b2d-2b2b-4b3e-9c46-2f6fd1c9a0c7",
  "plaintextContentType": "application/json",
  "plaintextB64": "eyJtYXRlcmlhbCI6ICJTdGVlbCIsICJ3ZWlnaHRfa2ciOiAxLjI1fQ=="
}
```

**Response Example (200 OK):**

Example A: Create policy response

```
HTTP/1.1 201 Created
Content-Type: application/json
ETag: "W/\"3\""
Location: /api/v1/policies/0d8b7b2d-2b2b-4b3e-9c46-2f6fd1c9a0c7
X-Correlation-ID: 2f8a8a6e-8df9-4a34-9b2c-0f552fe0e0b1

{
  "policyId": "0d8b7b2d-2b2b-4b3e-9c46-2f6fd1c9a0c7",
  "name": "dpp-material-disclosure-v1",
  "version": 1,
  "status": "DRAFT",
  "integrityHash": "sha256:7d5f0b45b0d0f76b3b52e8d9f4a1f2e0c8b0c0a1c7a2c0c6c9a3e5f7b1a2d3e4",
  "createdAt": "2025-12-30T10:15:30Z",
  "updatedAt": "2025-12-30T10:15:30Z"
}
```

Example B: Encrypt payload response

```
HTTP/1.1 200 OK
Content-Type: application/json
X-Correlation-ID: 9c173c1b-7f06-4f33-a2c0-2e61d2e9e84f

{
  "policyId": "0d8b7b2d-2b2b-4b3e-9c46-2f6fd1c9a0c7",
  "ciphertextContentType": "application/octet-stream",
  "ciphertextB64": "AAECAwQFBgcICQoLDA0ODw==",
  "algorithm": "CP-ABE+AES-256-GCM",
  "integrity": {
    "alg": "SHA-256",
    "value": "sha256:5f70bf18a08660b4a9b6d95ab3e1d6b42e..."
  }
}
```

### 5.3 Event and Message Specifications (Asynchronous/MQTT)

#### 5.3.1 Not applicable

| Attribute | Specification |
|-----------|---------------|
| **Topic/Channel** | Not applicable (no MQTT messaging within ICD-07). |
| **Direction** | Not applicable (no MQTT messaging within ICD-07). |
| **QoS Level** | Not applicable (no MQTT messaging within ICD-07). |
| **Trigger Condition** | Not applicable (no MQTT messaging within ICD-07). |
| **Payload Format** | Not applicable (no MQTT messaging within ICD-07). |
| **Retention** | Not applicable (no MQTT messaging within ICD-07). |

---

## 6. Data Structures

### 6.1 Data Model

**6.1.1 SPIPPolicy**

Policy expressions follow Attribute-Based Access Control (ABAC) and ABE expression grammar. Policy types include Ciphertext-Policy Attribute-Based Encryption (CP-ABE) and Key-Policy Attribute-Based Encryption (KP-ABE); enumeration values CP_ABE and KP_ABE correspond to these modes. Owner organisation identifiers use Legal Entity Identifier (LEI) codes or URIs. Open Digital Rights Language (ODRL) policies use JSON for Linking Data (JSON-LD) serialisation.

| Field | Type | Unit/Format | Semantic ID (IRDI) | Req | Description |
|-------|------|-------------|--------------------|-----|-------------|
| policyId | String | UUID | urn:data4circ:spip:policy:id | Y | Unique policy identifier. |
| name | String | UTF-8 | urn:data4circ:spip:policy:name | Y | Stable policy name within owner scope. |
| description | String | UTF-8 | urn:data4circ:spip:policy:description | N | Human-readable policy description. |
| policyType | String | CP_ABE or KP_ABE | urn:data4circ:spip:policy:type | Y | Policy type and enforcement mode. |
| abePolicy.scheme | String | CP-ABE | urn:data4circ:spip:abe:scheme | Y | ABE scheme identifier. |
| abePolicy.expression | String | ABAC/ABE expression grammar | urn:data4circ:spip:abe:expression | Y | Policy expression evaluated against subject and environment attributes. |
| odrlPolicy | Object | JSON-LD | urn:data4circ:spip:odrl:policy | N | Optional usage control policy expressed using ODRL 2.2. |
| ownerOrgId | String | LEI or URI | urn:data4circ:spip:org:id | Y | Owner organisation identifier. |
| status | String | DRAFT, ACTIVE, or ARCHIVED | urn:data4circ:spip:policy:status | Y | Lifecycle state. |
| version | Integer | >= 1 | urn:data4circ:spip:policy:version | Y | Monotonic policy version. |
| integrityHash | String | sha256:<hex> | urn:data4circ:spip:policy:integrity | Y | Integrity hash of canonicalised policy representation. |
| createdAt | String | ISO 8601 date-time | urn:data4circ:spip:timestamp:created | Y | Creation timestamp. |
| updatedAt | String | ISO 8601 date-time | urn:data4circ:spip:timestamp:updated | Y | Update timestamp. |

[WRITING GUIDELINE] CRITICAL: All numerical fields shall specify the Unit of Measure (UoM). Unitless numbers are grounds for rejection in Asset Administration Shell (AAS) and Life Cycle Assessment (LCA) contexts. Semantic IDs from ECLASS (0173-1#...) or the International Electrotechnical Commission Common Data Dictionary (IEC CDD) ensure interoperability across the DATA4CIRC ecosystem. International Registration Data Identifier (IRDI) references shall be maintained for AAS compliance.

### 6.2 Semantic Mappings

Document semantic mappings to ECLASS, IEC CDD, Data Catalogue Application Profile (DCAT-AP), or other standard vocabularies. Include IRDI references.

### 6.3 Data Governance and Compliance

Key material is returned as JSON Web Encryption (JWE) objects.

| Data Entity | Personally Identifiable Information (PII) (Y/N) | Classification | Retention Period |
|------------|-----------------------------------------------|----------------|------------------|
| subjectId | Y | Confidential | 365 days (audit log retention) |
| ownerOrgId | N | Internal | 365 days (policy metadata retention) |
| policyId | N | Internal | 365 days (policy metadata retention) |
| accessToken (JWT) | Y | Restricted | 0 days (not persisted); in-memory only |
| keyMaterialJwe (JWE) | Y | Restricted | 0 days (not persisted); transient response content |
| ciphertextB64 | N | Internal | Client-controlled; SPIP Platform does not persist ciphertext by default |
| auditEvent | Y | Confidential | 365 days (audit log retention) |

[WRITING GUIDELINE] Classification levels: Public (no restrictions), Internal (organisation access), Confidential (role-based access), Restricted (specific authorisation required). Retention periods shall comply with General Data Protection Regulation (GDPR) Article 5(1)(e) storage limitation principle.

---

## 7. Security Requirements

### 7.1 Authentication

| Attribute | Specification |
|-----------|---------------|
| Mechanism | OAuth 2.0 / OIDC; optional mTLS for service-to-service hardening. |
| Identity Provider | Keycloak (project IdP baseline) or federated external IdP via Keycloak brokering. |
| Token Type | JWT access token (RS256/ES256) with scope and attribute claims. |
| Token Lifetime | 3600 seconds (access token); refresh token usage prohibited for client-credentials flow. |

### 7.2 Authorisation

Role-Based Access Control (RBAC) is enforced through OAuth 2.0 scopes and endpoint policies.

| Operation | Required Role | SRS Reference |
|-----------|---------------|---------------|
| GET /policies, GET /policies/{policyId} | spip.policy.read | SRS-1-20, SRS-1-14 |
| POST /policies, PUT /policies/{policyId}, DELETE /policies/{policyId} | spip.policy.write | SRS-1-20, SRS-1-1 |
| POST /policies/{policyId}/validate | spip.policy.write | SRS-1-2 |
| POST /keys/issue | spip.key.issue | SRS-1-3 |
| POST /crypto/encrypt | spip.crypto.encrypt | SRS-1-4 |
| POST /crypto/decrypt | spip.crypto.decrypt | SRS-1-5 |
| GET /audit/events | spip.audit.read | SRS-1-19 |
| GET /version | spip.read | SRS-1-20 |

### 7.3 Transport Security

| Attribute | Specification |
|-----------|---------------|
| TLS Version | TLS 1.3 minimum (TLS 1.2 disabled in production profiles). |
| Certificate Validation | X.509 certificate chain validation against trusted Certificate Authority (CA); hostname verification mandatory; certificate pinning optional for DS4CIRC Portal. |
| Cipher Suites | TLS_AES_256_GCM_SHA384; TLS_CHACHA20_POLY1305_SHA256; TLS_AES_128_GCM_SHA256. |

### 7.4 Usage Control (ODRL Policies)

| Policy Element | Specification |
|--------------|--------------|
| Permission | USE; DISPLAY; DERIVE; DISTRIBUTE (subject to policy); ENCRYPT; DECRYPT. |
| Constraint | purpose equals <purpose-id>; recipientOrgId in <allow-list>; processingLocation equals <region-id>; validityInterval within <contractual-window>. |
| Duty | LOG access events; NOTIFY provider on contract breach; DELETE derived artefacts upon obligation trigger. |
| Prohibition | TRANSFER to unregistered participants; STORE beyond contractual retention; REIDENTIFY pseudonymous subjects. |

[WRITING GUIDELINE] Access control determines visibility (can the consumer see the data?). Usage control determines processing rights (can the consumer store, modify, or redistribute the data?). ODRL policies are enforced by the Eclipse Dataspace Connector (EDC) Policy Engine.

---

## 8. Performance Requirements

Performance targets are expressed at the 95th percentile (P95). Create, Read, Update, Delete (CRUD) operations are included in governance performance measurements.

| Metric | Target | SRS Reference |
|--------|--------|---------------|
| Response Time (P95) | < 3 seconds under normal load for governance operations (policy CRUD, key issuance, audit query). | SRS-1-22 |
| Encryption/Decryption Latency (P95) | < 5 seconds for payloads up to 10 MB (base64-encoded). | SRS-1-22 |
| Throughput | >= 100 requests/second sustained for read operations (GET /policies, GET /audit/events) with 10 concurrent clients. | SRS-1-24 |
| Availability | >= 99.5% availability for governance service access during operational hours. | SRS-1-24 |
| Max Payload Size | 10 MB plaintext payload (pre-base64) per /crypto/encrypt and /crypto/decrypt request. | SRS-1-22 |

---

## 9. Implementation Guidelines

### 9.1 Client Implementation Example

Python (FastAPI) Example:

```python
# Python 3.11 client example (FastAPI service-to-service integration)
import os
import uuid
import httpx

SPIP_BASE_URL = os.environ["SPIP_BASE_URL"].rstrip("/")
OIDC_TOKEN_URL = os.environ["OIDC_TOKEN_URL"]
OIDC_CLIENT_ID = os.environ["OIDC_CLIENT_ID"]
OIDC_CLIENT_SECRET = os.environ["OIDC_CLIENT_SECRET"]

def obtain_access_token() -> str:
    payload = {
        "grant_type": "client_credentials",
        "client_id": OIDC_CLIENT_ID,
        "client_secret": OIDC_CLIENT_SECRET,
    }
    r = httpx.post(OIDC_TOKEN_URL, data=payload, timeout=10.0)
    r.raise_for_status()
    return r.json()["access_token"]

def create_policy(policy: dict) -> dict:
    token = obtain_access_token()
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
        "Accept": "application/json",
        "X-Correlation-ID": str(uuid.uuid4()),
        "Idempotency-Key": str(uuid.uuid4()),
    }
    r = httpx.post(f"{SPIP_BASE_URL}/api/v1/policies", json=policy, headers=headers, timeout=30.0)

    content_type = r.headers.get("Content-Type", "")
    if content_type.startswith("application/problem+json"):
        raise RuntimeError(r.json())

    r.raise_for_status()
    return r.json()
```

### 9.2 Server Implementation Example

Java (Spring Boot) Example:

```java
// Java 21 / Spring Boot 3.x server-side skeleton (SPIP Platform endpoint shape)
@RestController
@RequestMapping(path = "/api/v1/policies", produces = MediaType.APPLICATION_JSON_VALUE)
public class PolicyController {

  @GetMapping
  @PreAuthorize("hasAuthority('SCOPE_spip.policy.read')")
  public ResponseEntity<PolicyListResponse> listPolicies(
      @RequestParam Optional<String> ownerOrgId,
      @RequestParam Optional<String> status,
      @RequestParam(defaultValue = "50") int limit,
      @RequestParam(defaultValue = "0") int offset) {
    return ResponseEntity.ok(new PolicyListResponse(/* ... */));
  }

  @PostMapping(consumes = MediaType.APPLICATION_JSON_VALUE)
  @PreAuthorize("hasAuthority('SCOPE_spip.policy.write')")
  public ResponseEntity<PolicyResponse> createPolicy(
      @RequestHeader(value = "Idempotency-Key", required = false) String idempotencyKey,
      @RequestHeader(value = "X-Correlation-ID", required = false) String correlationId,
      @Valid @RequestBody PolicyCreateRequest request) {
    PolicyResponse response = /* ... */;
    return ResponseEntity.created(URI.create("/api/v1/policies/" + response.policyId()))
        .eTag(response.etag())
        .body(response);
  }
}
```

### 9.3 Deployment Configuration

```yaml
# Kubernetes ingress and service-level configuration (excerpt)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: spip-platform-ingress
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  tls:
  - hosts:
    - spip.example.internal
    secretName: spip-tls
  rules:
  - host: spip.example.internal
    http:
      paths:
      - path: /api/v1
        pathType: Prefix
        backend:
          service:
            name: spip-platform
            port:
              number: 443
```

### 9.4 Observability and Tracing

| Attribute | Specification |
|-----------|---------------|
| Trace ID Source | HTTP X-Correlation-ID header propagated end-to-end; OpenTelemetry traceparent header supported for distributed tracing. |
| Health Check | HTTP GET /api/v1/health returns 200 with status payload. |
| Readiness | HTTP GET /api/v1/ready returns 200 when dependencies (Key Vault, persistence) are available. |
| Metrics Endpoint | HTTP GET /api/v1/metrics returns Prometheus-format metrics (text/plain; version=0.0.4). |
| Log Format | Structured JSON logs containing correlationId, subjectId (pseudonymous), policyId, operation, outcome, and latencyMs. |

[WRITING GUIDELINE] For MQTT interfaces, the Last Will and Testament (LWT) message is published by the broker when a client disconnects unexpectedly. The LWT topic and payload shall be defined to enable downstream systems to detect service unavailability.

### 9.5 Configuration and Environment Variables

| Env Variable / Key | Default | Required | Description |
|--------------------|---------|----------|-------------|
| SPIP_BASE_URL | https://spip.example.internal | Yes | Base URL of SPIP Platform ingress. |
| OIDC_TOKEN_URL | https://keycloak.example.internal/realms/data4circ/protocol/openid-connect/token | Yes | OAuth 2.0 token endpoint (client credentials). |
| OIDC_CLIENT_ID | [None] | Yes | OAuth 2.0 client identifier assigned to DS4CIRC Portal. |
| OIDC_CLIENT_SECRET | [None] | Yes | OAuth 2.0 client secret (inject via secrets management). |
| OIDC_AUDIENCE | spip-platform | No | Expected audience claim for SPIP Platform tokens. |
| TLS_CA_BUNDLE_PATH | /etc/ssl/certs/ca-bundle.crt | No | CA bundle path for server certificate validation. |
| HTTP_CONNECT_TIMEOUT_SECONDS | 10 | No | HTTP connect timeout for SPIP Platform requests. |
| HTTP_READ_TIMEOUT_SECONDS | 60 | No | HTTP read timeout for SPIP Platform requests. |
| HTTP_RETRY_MAX_ATTEMPTS | 3 | No | Maximum retry attempts for eligible requests. |
| LOG_LEVEL | INFO | No | Logging verbosity (DEBUG, INFO, WARN, ERROR). |
| ENABLE_MTLS | false | No | Enable mTLS for service-to-service calls. |
| MTLS_CLIENT_CERT_PATH | [None] | No | Client certificate path for mutual TLS. |
| MTLS_CLIENT_KEY_PATH | [None] | No | Client private key path for mutual TLS. |

[WRITING GUIDELINE] Environment variables enable containerised deployment without code modification. Sensitive values (credentials, API keys) shall be injected via secrets management rather than environment variables in production environments.

---

## 10. Requirements Traceability Matrix

| SRS ID | Requirement | Interface Feature | Verification Method |
|--------|-------------|-------------------|---------------------|
| SRS-1-1 | The system shall integrate with the SPIP infrastructure stack for supporting policy creation and key management across organisational boundaries. | Section 5.1; /policies; /keys/issue; Annex B OpenAPI | Integration test; API conformance against OpenAPI |
| SRS-1-2 | The system shall implement ABE policies for enabling fine-grained access control to the data shared with other organisations. | Section 6.1 SPIPPolicy.abePolicy; /policies/{policyId}/validate | Unit test for policy compiler; negative validation tests |
| SRS-1-3 | The system shall generate and distribute user decryption and encryption keys based on validated organisational attribute sets through SPIP Key Vault. | Section 5.1; /keys/issue; Section 7.2 authorisation | Integration test with Key Vault; security test for key wrapping |
| SRS-1-4 | The system shall enable data providers to encrypt content using SPIP policies before storing the content in local data repositories or before distributing the content to other organisations. | Section 5.1; /crypto/encrypt | Integration test with reference plaintext/ciphertext vectors |
| SRS-1-5 | The system shall enable data consumers to decrypt content using SPIP policies. | Section 5.1; /crypto/decrypt | Integration test with reference ciphertext/plaintext vectors |
| SRS-1-14 | The application shall provide granular access policies based on ABAC models, by enabling operations based on a specific set of attributes. | Section 6.2 attribute namespace; Section 7.2 scope and claim enforcement | Security test of attribute constraints; negative authorisation tests |
| SRS-1-19 | The application shall authenticate users based on Zero-Trust Security principles before granting access to Federated Data Spaces. | Section 7.1 OAuth 2.0/OIDC; token validation and claim checks | Security test: invalid or expired token scenarios; penetration test |
| SRS-1-20 | The application shall enforce role-based permissions to control access to specific data within Federated Data Spaces. | Section 7.2 roles and scopes; endpoint protection in Annex B | Security test: role matrix; least-privilege verification |
| SRS-1-22 | The application shall ensure response times for Federated Data Space access do not exceed 3 seconds under normal load conditions. | Section 8 performance targets; Section 4.2 timeout and circuit breaker | Performance test (P95 latency) |
| SRS-1-23 | The application shall encrypt all data transmissions to and from Federated Data Spaces using a strong industry-standard encryption algorithm. | Section 4 protocol stack (TLS 1.3); Section 7.3 transport security | Security test: TLS configuration scanning |
| SRS-1-24 | The application shall maintain at least 99.5% availability for Federated Data Space access services during operational hours. | Section 8 availability; Section 9.4 health and readiness probes | Operational monitoring; chaos testing; Service Level Objective (SLO) verification |

---

## 11. Acceptance Criteria

Each Acceptance Criterion (AC) shall use clear, measurable language.

| AC ID | Criterion | Test Method | SRS Ref |
|-------|-----------|-------------|---------|
| AC-01 | POST /api/v1/policies returns HTTP 201 and a valid policyId for syntactically and semantically valid policy definitions. | Integration | SRS-1-1, SRS-1-2 |
| AC-02 | POST /api/v1/policies/{policyId}/validate returns HTTP 200 with valid=true for valid ABE expressions and returns HTTP 422 with problem details for invalid expressions. | Integration | SRS-1-2 |
| AC-03 | POST /api/v1/keys/issue returns HTTP 200 and keyMaterialJwe for authorised requests and returns HTTP 403 for insufficient scope or attribute constraints. | Integration/Security | SRS-1-3, SRS-1-20 |
| AC-04 | POST /api/v1/crypto/encrypt returns ciphertext that decrypts to the original plaintext when processed by POST /api/v1/crypto/decrypt under the same policy context. | Integration | SRS-1-4, SRS-1-5 |
| AC-05 | All protected endpoints reject missing or invalid tokens with HTTP 401 and application/problem+json response bodies. | Security | SRS-1-19 |
| AC-06 | P95 latency for policy CRUD operations does not exceed 3 seconds under normal load conditions. | Performance | SRS-1-22 |
| AC-07 | TLS 1.3 is enforced for all HTTPS endpoints and weak ciphers are disabled. | Security | SRS-1-23 |
| AC-08 | Health endpoint returns HTTP 200 when SPIP Platform dependencies are available and readiness endpoint returns non-200 when critical dependencies are unavailable. | Integration/Operational | SRS-1-24 |

[WRITING GUIDELINE] Acceptance criteria shall use clear, measurable language. Avoid subjunctive constructions. Correct: 'The interface returns HTTP 200 within 2 seconds for 95% of requests.' Incorrect: 'The interface should respond quickly.'

---

## 12. References

[1] D2.2 DATA4CIRC Requirements and Specifications
[2] D4.1 Platform Architecture and Open-Source Protocols
[3] IDTA AAS Metamodel Specification (IDTA-01001)
[4] Eclipse Dataspace Connector Documentation
[5] OAuth 2.0 (RFC 6749) and OpenID Connect Core 1.0 (Keycloak implementation)
[6] RFC 9457 Problem Details for HTTP APIs (obsoletes RFC 7807)
[7] W3C ODRL Information Model 2.2
[8] OpenAPI Specification 3.1.0
[9] RFC 8446 The Transport Layer Security (TLS) Protocol Version 1.3
[10] RFC 7519 JSON Web Token (JWT)
[11] RFC 8259 The JavaScript Object Notation (JSON) Data Interchange Format
[12] JSON Schema 2020-12
[13] NIST SP 800-207 Zero Trust Architecture

---

## 13. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 30 December 2025 | DATA4CIRC WP3 Technical Team (NTT, RWTH) | Approved baseline for ICD-07; OpenAPI v3.1.0 annex included; test cases and requirements traceability matrix completed. |

---

## Annex A: Sequence Diagrams

PlantUML sources for interaction patterns.

### A.1 Policy lifecycle management

```
@startuml
actor "Portal Operator" as Operator
participant "DS4CIRC Portal" as Portal
participant "Keycloak IdP" as IdP
participant "SPIP Platform" as SPIP
database "SPIP Key Vault" as KV

Operator -> Portal: Submit policy definition
Portal -> IdP: Token request (OAuth 2.0/OIDC)
IdP --> Portal: Access token (JWT)
Portal -> SPIP: POST /api/v1/policies
note right: X-Correlation-ID, Idempotency-Key
SPIP -> SPIP: Authorisation + policy validation
SPIP -> KV: Persist policy context
KV --> SPIP: Context identifier
SPIP --> Portal: 201 Created (policyId, ETag)
@enduml
```

### A.2 Key issuance

```
@startuml
participant "DS4CIRC Portal" as Portal
participant "SPIP Platform" as SPIP
database "SPIP Key Vault" as KV

Portal -> SPIP: POST /api/v1/keys/issue
note right: subjectId + attributes + policyId
SPIP -> SPIP: Validate scope + attribute constraints
SPIP -> KV: Derive key material
KV --> SPIP: Key material
SPIP --> Portal: 200 OK (keyMaterialJwe, keyId)
@enduml
```

### A.3 Encryption and decryption

```
@startuml
participant "DS4CIRC Portal" as Portal
participant "SPIP Platform" as SPIP
database "SPIP Key Vault" as KV

Portal -> SPIP: POST /api/v1/crypto/encrypt
SPIP -> KV: Resolve encryption context
KV --> SPIP: Key reference
SPIP --> Portal: 200 OK (ciphertextB64)

Portal -> SPIP: POST /api/v1/crypto/decrypt
SPIP -> KV: Resolve decryption context
KV --> SPIP: Key reference
SPIP --> Portal: 200 OK (plaintextB64)
@enduml
```

---

## Annex B: Complete API Schema

OpenAPI Specification v3.1.0 (machine-readable artefact: ICD-07_SPIP_Governance_API_openapi_v1.0.yaml).

```yaml
openapi: 3.1.0
jsonSchemaDialect: https://json-schema.org/draft/2020-12/schema
info:
  title: DATA4CIRC SPIP Governance API
  version: 1.0.0
  description: OpenAPI Specification for ICD-07 (DS4CIRC Portal <-> SPIP Platform).
servers:
- url: https://{spip_host}/api/v1
  variables:
    spip_host:
      default: spip.example.internal
security:
- oauth2:
  - spip.read
paths:
  /health:
    get:
      summary: Liveness probe
      operationId: getHealth
      responses:
        '200':
          description: Service is alive
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HealthStatus'
  /ready:
    get:
      summary: Readiness probe
      operationId: getReadiness
      responses:
        '200':
          description: Service is ready
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HealthStatus'
        '503':
          $ref: '#/components/responses/ProblemDetails'
  /version:
    get:
      summary: Service and API version metadata
      operationId: getVersion
      security:
      - oauth2:
        - spip.read
      responses:
        '200':
          description: Version
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/VersionInfo'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
  /policies:
    get:
      summary: List policies
      operationId: listPolicies
      security:
      - oauth2:
        - spip.policy.read
      parameters:
      - name: ownerOrgId
        in: query
        schema:
          type: string
        required: false
        description: Filter by owner organisation identifier.
      - name: status
        in: query
        schema:
          type: string
          enum:
          - DRAFT
          - ACTIVE
          - ARCHIVED
        required: false
        description: Filter by status.
      - name: limit
        in: query
        schema:
          type: integer
          minimum: 1
          maximum: 200
          default: 50
        required: false
      - name: offset
        in: query
        schema:
          type: integer
          minimum: 0
          default: 0
        required: false
      responses:
        '200':
          description: Policy list
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyListResponse'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
    post:
      summary: Create policy
      operationId: createPolicy
      security:
      - oauth2:
        - spip.policy.write
      parameters:
      - name: Idempotency-Key
        in: header
        schema:
          type: string
          format: uuid
        required: false
      - name: X-Correlation-ID
        in: header
        schema:
          type: string
          format: uuid
        required: false
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PolicyCreateRequest'
      responses:
        '201':
          description: Created
          headers:
            ETag:
              schema:
                type: string
            Location:
              schema:
                type: string
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyResponse'
        '400':
          $ref: '#/components/responses/ProblemDetails'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '409':
          $ref: '#/components/responses/ProblemDetails'
        '422':
          $ref: '#/components/responses/ProblemDetails'
  /policies/{policyId}:
    get:
      summary: Retrieve policy
      operationId: getPolicy
      security:
      - oauth2:
        - spip.policy.read
      parameters:
      - $ref: '#/components/parameters/policyId'
      responses:
        '200':
          description: Policy
          headers:
            ETag:
              schema:
                type: string
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyResponse'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '404':
          $ref: '#/components/responses/ProblemDetails'
    put:
      summary: Replace policy
      operationId: replacePolicy
      security:
      - oauth2:
        - spip.policy.write
      parameters:
      - $ref: '#/components/parameters/policyId'
      - name: If-Match
        in: header
        schema:
          type: string
        required: true
        description: ETag value for optimistic concurrency control.
      - name: X-Correlation-ID
        in: header
        schema:
          type: string
          format: uuid
        required: false
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PolicyUpdateRequest'
      responses:
        '200':
          description: Updated
          headers:
            ETag:
              schema:
                type: string
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyResponse'
        '400':
          $ref: '#/components/responses/ProblemDetails'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '404':
          $ref: '#/components/responses/ProblemDetails'
        '409':
          $ref: '#/components/responses/ProblemDetails'
        '412':
          $ref: '#/components/responses/ProblemDetails'
        '422':
          $ref: '#/components/responses/ProblemDetails'
    delete:
      summary: Archive policy
      operationId: archivePolicy
      security:
      - oauth2:
        - spip.policy.write
      parameters:
      - $ref: '#/components/parameters/policyId'
      responses:
        '204':
          description: Archived
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '404':
          $ref: '#/components/responses/ProblemDetails'
  /policies/{policyId}/validate:
    post:
      summary: Validate and compile policy
      operationId: validatePolicy
      security:
      - oauth2:
        - spip.policy.write
      parameters:
      - $ref: '#/components/parameters/policyId'
      requestBody:
        required: false
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PolicyValidationRequest'
      responses:
        '200':
          description: Validation result
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PolicyValidationResponse'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '404':
          $ref: '#/components/responses/ProblemDetails'
        '422':
          $ref: '#/components/responses/ProblemDetails'
  /keys/issue:
    post:
      summary: Issue encryption/decryption key material (wrapped)
      operationId: issueKey
      security:
      - oauth2:
        - spip.key.issue
      parameters:
      - name: X-Correlation-ID
        in: header
        schema:
          type: string
          format: uuid
        required: false
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/KeyIssueRequest'
      responses:
        '200':
          description: Issued
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/KeyIssueResponse'
        '400':
          $ref: '#/components/responses/ProblemDetails'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '404':
          $ref: '#/components/responses/ProblemDetails'
        '422':
          $ref: '#/components/responses/ProblemDetails'
  /crypto/encrypt:
    post:
      summary: Encrypt payload under policy
      operationId: encrypt
      security:
      - oauth2:
        - spip.crypto.encrypt
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/EncryptRequest'
      responses:
        '200':
          description: Ciphertext
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/EncryptResponse'
        '400':
          $ref: '#/components/responses/ProblemDetails'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '404':
          $ref: '#/components/responses/ProblemDetails'
        '413':
          $ref: '#/components/responses/ProblemDetails'
        '422':
          $ref: '#/components/responses/ProblemDetails'
        '429':
          $ref: '#/components/responses/ProblemDetails'
  /crypto/decrypt:
    post:
      summary: Decrypt payload under policy
      operationId: decrypt
      security:
      - oauth2:
        - spip.crypto.decrypt
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/DecryptRequest'
      responses:
        '200':
          description: Plaintext
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DecryptResponse'
        '400':
          $ref: '#/components/responses/ProblemDetails'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
        '404':
          $ref: '#/components/responses/ProblemDetails'
        '413':
          $ref: '#/components/responses/ProblemDetails'
        '422':
          $ref: '#/components/responses/ProblemDetails'
        '429':
          $ref: '#/components/responses/ProblemDetails'
  /audit/events:
    get:
      summary: Query audit events
      operationId: getAuditEvents
      security:
      - oauth2:
        - spip.audit.read
      parameters:
      - name: resourceId
        in: query
        schema:
          type: string
        required: false
      - name: eventType
        in: query
        schema:
          type: string
        required: false
      - name: limit
        in: query
        schema:
          type: integer
          minimum: 1
          maximum: 200
          default: 100
        required: false
      responses:
        '200':
          description: Audit events
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AuditEventListResponse'
        '401':
          $ref: '#/components/responses/ProblemDetails'
        '403':
          $ref: '#/components/responses/ProblemDetails'
components:
  parameters:
    policyId:
      name: policyId
      in: path
      required: true
      schema:
        type: string
        format: uuid
      description: Policy identifier (UUID).
  responses:
    ProblemDetails:
      description: RFC 9457 Problem Details response
      content:
        application/problem+json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://{idp_host}/realms/{realm}/protocol/openid-connect/token
          scopes:
            spip.read: Read access to service metadata.
            spip.policy.read: Read policies.
            spip.policy.write: Create/update/archive policies.
            spip.key.issue: Issue key material.
            spip.crypto.encrypt: Encrypt payloads.
            spip.crypto.decrypt: Decrypt payloads.
            spip.audit.read: Read audit events.
  schemas:
    HealthStatus:
      type: object
      properties:
        status:
          type: string
          enum:
          - UP
          - DOWN
        components:
          type: object
          additionalProperties:
            type: string
      required:
      - status
    VersionInfo:
      type: object
      properties:
        service:
          type: string
        version:
          type: string
        buildHash:
          type: string
        openapiVersion:
          type: string
      required:
      - service
      - version
    PolicyCreateRequest:
      type: object
      properties:
        name:
          type: string
          minLength: 3
          maxLength: 128
        description:
          type: string
          maxLength: 2048
        policyType:
          type: string
          enum:
          - CP_ABE
          - KP_ABE
        abePolicy:
          $ref: '#/components/schemas/AbePolicy'
        odrlPolicy:
          type: object
          description: Optional ODRL policy (JSON-LD).
          additionalProperties: true
        ownerOrgId:
          type: string
        tags:
          type: array
          items:
            type: string
          maxItems: 32
      required:
      - name
      - policyType
      - abePolicy
      - ownerOrgId
    PolicyUpdateRequest:
      allOf:
      - $ref: '#/components/schemas/PolicyCreateRequest'
      - type: object
        properties:
          status:
            type: string
            enum:
            - DRAFT
            - ACTIVE
            - ARCHIVED
    AbePolicy:
      type: object
      properties:
        scheme:
          type: string
          enum:
          - CP-ABE
        expression:
          type: string
          minLength: 1
          maxLength: 4096
      required:
      - scheme
      - expression
    PolicyResponse:
      type: object
      properties:
        policyId:
          type: string
          format: uuid
        name:
          type: string
        description:
          type: string
        policyType:
          type: string
        abePolicy:
          $ref: '#/components/schemas/AbePolicy'
        odrlPolicy:
          type: object
          additionalProperties: true
        ownerOrgId:
          type: string
        status:
          type: string
          enum:
          - DRAFT
          - ACTIVE
          - ARCHIVED
        version:
          type: integer
          minimum: 1
        integrityHash:
          type: string
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time
        etag:
          type: string
      required:
      - policyId
      - name
      - policyType
      - abePolicy
      - ownerOrgId
      - status
      - version
      - integrityHash
      - createdAt
      - updatedAt
    PolicyListResponse:
      type: object
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/PolicyResponse'
        limit:
          type: integer
        offset:
          type: integer
        total:
          type: integer
      required:
      - items
      - limit
      - offset
    PolicyValidationRequest:
      type: object
      properties:
        strict:
          type: boolean
          default: true
        sampleAttributes:
          type: object
          additionalProperties:
            type: string
    PolicyValidationResponse:
      type: object
      properties:
        valid:
          type: boolean
        errors:
          type: array
          items:
            type: string
      required:
      - valid
    KeyIssueRequest:
      type: object
      properties:
        policyId:
          type: string
          format: uuid
        subjectId:
          type: string
        subjectAttributes:
          type: object
          additionalProperties:
            type: string
        keyType:
          type: string
          enum:
          - ENCRYPT
          - DECRYPT
        clientKeyId:
          type: string
          description: Identifier of client public key used for key wrapping.
      required:
      - policyId
      - subjectId
      - subjectAttributes
      - keyType
    KeyIssueResponse:
      type: object
      properties:
        keyId:
          type: string
          format: uuid
        policyId:
          type: string
          format: uuid
        subjectId:
          type: string
        keyType:
          type: string
          enum:
          - ENCRYPT
          - DECRYPT
        algorithm:
          type: string
        keyMaterialJwe:
          type: string
          description: Wrapped key material as a JWE compact serialisation.
        validFrom:
          type: string
          format: date-time
        validTo:
          type: string
          format: date-time
      required:
      - keyId
      - policyId
      - subjectId
      - keyType
      - algorithm
      - keyMaterialJwe
    EncryptRequest:
      type: object
      properties:
        policyId:
          type: string
          format: uuid
        plaintextB64:
          type: string
          format: byte
        plaintextContentType:
          type: string
        aadB64:
          type: string
          format: byte
      required:
      - policyId
      - plaintextB64
      - plaintextContentType
    EncryptResponse:
      type: object
      properties:
        policyId:
          type: string
          format: uuid
        ciphertextB64:
          type: string
          format: byte
        ciphertextContentType:
          type: string
        algorithm:
          type: string
        integrity:
          type: object
          properties:
            alg:
              type: string
            value:
              type: string
          required:
          - alg
          - value
      required:
      - policyId
      - ciphertextB64
      - ciphertextContentType
      - algorithm
    DecryptRequest:
      type: object
      properties:
        policyId:
          type: string
          format: uuid
        ciphertextB64:
          type: string
          format: byte
        ciphertextContentType:
          type: string
      required:
      - policyId
      - ciphertextB64
      - ciphertextContentType
    DecryptResponse:
      type: object
      properties:
        policyId:
          type: string
          format: uuid
        plaintextB64:
          type: string
          format: byte
        plaintextContentType:
          type: string
      required:
      - policyId
      - plaintextB64
      - plaintextContentType
    AuditEvent:
      type: object
      properties:
        eventId:
          type: string
          format: uuid
        timestamp:
          type: string
          format: date-time
        actorSubjectId:
          type: string
        actorOrgId:
          type: string
        eventType:
          type: string
        resourceType:
          type: string
        resourceId:
          type: string
        outcome:
          type: string
          enum:
          - SUCCESS
          - DENY
          - ERROR
        correlationId:
          type: string
          format: uuid
        details:
          type: object
          additionalProperties: true
      required:
      - eventId
      - timestamp
      - eventType
      - resourceType
      - resourceId
      - outcome
    AuditEventListResponse:
      type: object
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/AuditEvent'
        limit:
          type: integer
      required:
      - items
    ProblemDetails:
      type: object
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
        retryable:
          type: boolean
        remediation:
          type: string
        invalidParams:
          type: array
          items:
            type: object
            properties:
              name:
                type: string
              reason:
                type: string
            required:
            - name
            - reason
      required:
      - type
      - title
      - status
```

---

## Annex C: Test Cases

Test cases validating the acceptance criteria are specified in Table C-1.

| TC ID | Preconditions | Steps | Expected Result | Mapped AC |
|------|---------------|-------|-----------------|-----------|
| [x] | Valid OAuth 2.0 token with spip.policy.write scope. | 1) Submit POST /policies with valid JSON schema. 2) Inspect response. | HTTP 201; response contains policyId (UUID); ETag header present. | AC-01 |
| [x] | Valid token with spip.policy.write scope; existing policyId. | 1) Submit POST /policies/{policyId}/validate with valid abePolicy.expression. 2) Inspect response. | HTTP 200; body contains valid=true and empty error list. | AC-02 |
| [x] | Valid token with spip.policy.write scope; existing policyId. | 1) Submit POST /policies/{policyId}/validate with invalid abePolicy.expression. 2) Inspect response. | HTTP 422; Content-Type application/problem+json; type and errorCode populated. | AC-02 |
| [x] | Valid token with spip.key.issue scope; policy status ACTIVE; subject attributes satisfy policy. | 1) Submit POST /keys/issue. 2) Inspect response. | HTTP 200; keyMaterialJwe present; keyId present. | AC-03 |
| [x] | Valid token without spip.key.issue scope. | 1) Submit POST /keys/issue. 2) Inspect response. | HTTP 403; problem details; remediation indicates missing scope. | AC-03 |
| [x] | Valid token with spip.crypto.encrypt and spip.crypto.decrypt scopes; policyId valid. | 1) Submit POST /crypto/encrypt with plaintext. 2) Submit POST /crypto/decrypt with returned ciphertext. 3) Compare plaintext. | Decrypted plaintext equals original plaintext; integrity metadata present. | AC-04 |
| [x] | No token. | 1) Submit GET /policies. 2) Inspect response. | HTTP 401; application/problem+json. | AC-05 |
| [x] | Performance test harness; 10 concurrent clients; normal load profile. | 1) Execute 10,000 operations (policy list and create) over 10 minutes. 2) Measure latency. | P95 latency < 3 seconds for measured operations. | AC-06 |
| [x] | TLS scanner configured against SPIP ingress endpoint. | 1) Execute TLS configuration scan. 2) Inspect supported protocol versions and cipher suites. | TLS 1.3 enabled; TLS 1.2 disabled; only approved cipher suites enabled. | AC-07 |
| [x] | SPIP Platform dependency injection for Key Vault failure simulation. | 1) Induce Key Vault unavailability. 2) Call GET /ready. 3) Restore Key Vault. 4) Call GET /ready. | Readiness returns non-200 during dependency failure and returns 200 after recovery. | AC-08 |

---

## Annex D: Quality Checklist

| Check | Criterion | Section |
|-------|----------|---------|
| [ ] | Units of measure specified for all numerical fields | Section 6.1 |
| [ ] | Semantic IDs (IRDIs) provided for AAS-compliant fields | Section 6.1 |
| [ ] | Environment variables listed for DevOps deployment | Section 9.5 |
| [ ] | Circuit breaker thresholds defined for resilience | Section 4.2 |
| [ ] | PII fields flagged and retention policies defined | Section 6.3 |
| [ ] | ODRL policies defined for dataspace interfaces | Section 7.4 |
| [ ] | MQTT topics, QoS, and LWT defined for IoT interfaces | Section 5.3, Section 9.4 |
| [ ] | Error handling appropriate for protocol (RFC 9457 or DLQ) | Section 2.3 |
| [ ] | Health check mechanism defined (HTTP endpoint or MQTT LWT) | Section 9.4 |
| [ ] | Interface dependencies and versioning documented | Section 1.4 |
| [ ] | British English and IEEE style followed throughout | All sections |
| [ ] | No subjunctive mood, personal pronouns, or filler words | All sections |
| [ ] | Abbreviations defined once and listed in Section 3 | Section 3 |
| [ ] | Performance targets use specific numerical values | Section 8 |
