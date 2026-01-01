# DATA4CIRC Interface Control Document Catalogue

## 1. Introduction

The Interface Control Document (ICD) Catalogue defines the interfaces between subsystems of the DATA4CIRC platform. Each ICD specifies the communication protocols, data formats, and interaction patterns required for inter-component integration.

This catalogue serves as the interface register and identifier authority for the DATA4CIRC federated dataspace platform.

## 2. Abbreviations

| Abbreviation | Definition |
|--------------|------------|
| AAS | Asset Administration Shell |
| API | Application Programming Interface |
| DPP | Digital Product Passport |
| DT/DTh | Digital Twin/Digital Thread |
| EDC | Eclipse Dataspace Connector |
| ERP | Enterprise Resource Planning |
| ICD | Interface Control Document |
| IdP | Identity Provider |
| IoT | Internet of Things |
| LCA | Life Cycle Assessment |
| MES | Manufacturing Execution System |
| MQTT | Message Queuing Telemetry Transport |
| ODRL | Open Digital Rights Language |
| PLM | Product Lifecycle Management |
| REST | Representational State Transfer |
| SPIP | Secure Policy Information Point |
| SSO | Single Sign-On |
| TLS | Transport Layer Security |

## 3. ICD Register

### 3.1 Portal-Level Interfaces

| ICD-# | Interface Description | Interface Details | WP# | Status |
|-------|----------------------|-------------------|-----|--------|
| [ICD-01](../icds/portal-level/ICD-01/ICD-01.md) | DATA4CIRC Portal ↔ DS4CIRC Portal | SSO, deep link, role propagation | WP3 | Approved |
| [ICD-02](../icds/portal-level/ICD-02/ICD-02.md) | DATA4CIRC Portal ↔ DPP/AAS Portal | SSO, deep link, role propagation | WP4 | Approved |
| [ICD-03](../icds/portal-level/ICD-03/ICD-03.md) | DATA4CIRC Portal ↔ DT/DTh Portal | SSO, deep link, role propagation | WP5 | Approved |
| [ICD-04](../icds/portal-level/ICD-04/ICD-04.md) | DATA4CIRC Portal ↔ DT/DTh Tool Portal | SSO, deep link, role propagation | WP4 | Draft |

### 3.2 Governance Interfaces

| ICD-# | Interface Description | Interface Details | WP# | Status |
|-------|----------------------|-------------------|-----|--------|
| [ICD-05](../icds/governance/ICD-05/ICD-05.md) | SPIP Agent ↔ Dataspace Connector | Policy enforcement, access control | WP4 | Draft |
| [ICD-06](../icds/governance/ICD-06/ICD-06.md) | SPIP Agent ↔ Identity Provider | Authentication, token exchange | WP4 | Draft |
| [ICD-07](../icds/governance/ICD-07/ICD-07.md) | SPIP Agent ↔ Policy Repository | Policy retrieval, caching | WP4 | Draft |

### 3.3 Application Interfaces

| ICD-# | Interface Description | Interface Details | WP# | Status |
|-------|----------------------|-------------------|-----|--------|
| [ICD-08](../icds/application/ICD-08/ICD-08.md) | DATA4CIRC Portal ↔ LCA Tool | REST API, data exchange | WP4 | Draft |
| [ICD-09](../icds/application/ICD-09/ICD-09.md) | DATA4CIRC Portal ↔ DPP/AAS Tool | REST API, AAS operations | WP4 | Draft |
| [ICD-10](../icds/application/ICD-10/ICD-10.md) | DATA4CIRC Portal ↔ DT/DTh Tool | REST API, simulation data | WP4 | Draft |
| [ICD-11](../icds/application/ICD-11/ICD-11.md) | LCA Tool ↔ DPP/AAS Tool | Data synchronisation | WP4 | Draft |
| [ICD-12](../icds/application/ICD-12/ICD-12.md) | LCA Tool ↔ DT/DTh Tool | Simulation parameters | WP4 | Draft |
| [ICD-13](../icds/application/ICD-13/ICD-13.md) | DPP/AAS Tool ↔ DT/DTh Tool | Asset state synchronisation | WP4 | Draft |

### 3.4 Dataspace Interfaces

| ICD-# | Interface Description | Interface Details | WP# | Status |
|-------|----------------------|-------------------|-----|--------|
| [ICD-14](../icds/dataspace/ICD-14/ICD-14.md) | EDC ↔ EDC Catalog Exchange | DSP protocol, catalog queries | WP4 | Draft |
| [ICD-15](../icds/dataspace/ICD-15/ICD-15.md) | EDC ↔ EDC Contract Negotiation | DSP protocol, contract offers | WP4 | Draft |
| [ICD-16](../icds/dataspace/ICD-16/ICD-16.md) | EDC ↔ EDC Data Transfer | DSP protocol, data plane | WP4 | Draft |
| [ICD-17](../icds/dataspace/ICD-17/ICD-17.md) | EDC ↔ SPIP Policy Evaluation | ODRL policies, access decisions | WP4 | Draft |

### 3.5 Data Source Interfaces

| ICD-# | Interface Description | Interface Details | WP# | Status |
|-------|----------------------|-------------------|-----|--------|
| [ICD-18-1](../icds/data-source/ICD-18-1/ICD-18-1.md) | Data Source ↔ EDC (ERP/PLM/MES) | Database connectors, batch sync | WP5 | Draft |
| [ICD-18-2](../icds/data-source/ICD-18-2/ICD-18-2.md) | Data Source ↔ EDC (IoT/Sensors) | MQTT, real-time telemetry | WP5 | Draft |

## 4. Interface Category Definitions

| Category | Description | ICD Range |
|----------|-------------|-----------|
| Portal-Level | User-facing portal integration with SSO and deep linking | ICD-01 to ICD-04 |
| Governance | Security policy infrastructure and identity management | ICD-05 to ICD-07 |
| Application | Inter-application communication within the platform | ICD-08 to ICD-13 |
| Dataspace | Eclipse Dataspace Connector federation protocols | ICD-14 to ICD-17 |
| Data Source | External system and IoT device integration | ICD-18.x |

## 5. Status Definitions

| Status | Definition |
|--------|------------|
| Draft | Initial specification under development |
| Under Review | Submitted for peer review |
| Approved | Approved for implementation |
| Implemented | Interface implemented and tested |
| Deprecated | Interface scheduled for removal |

## 6. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | January 2026 | WP4 Team | Initial catalogue release |
