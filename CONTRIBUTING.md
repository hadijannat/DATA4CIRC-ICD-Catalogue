# Contributing to DATA4CIRC ICD Catalogue

This document establishes the procedures for submitting Interface Control Documents (ICDs) and modifications to the DATA4CIRC ICD Catalogue repository.

## Contribution Workflow

The contribution workflow follows a branch-based development model:

1. Create a feature branch from the main branch
2. Register the interface in the ICD Catalogue
3. Create the ICD document from the template
4. Complete all mandatory sections
5. Execute the quality checklist (Annex D)
6. Submit a pull request for peer review
7. Address review comments
8. Merge upon approval

## Branch Naming Convention

Feature branches shall follow the naming convention:

```
icd/{ICD-number}/{brief-description}
```

**Examples:**
- `icd/ICD-05/spip-agent-connector`
- `icd/ICD-08/portal-lca-tool`
- `icd/ICD-14/edc-catalog-exchange`

## Creating a New ICD

### Step 1: Create Feature Branch

```bash
git checkout main
git pull origin main
git checkout -b icd/ICD-XX/brief-description
```

### Step 2: Register the Interface

Add a new entry to `catalogue/ICD_CATALOGUE.md` with the following information:

| Field | Description |
|-------|-------------|
| ICD-# | Assigned interface identifier |
| Interface Description | Component A ↔ Component B |
| Interface Details | Technology qualifiers (SSO, REST, MQTT, EDC) |
| WP# | Responsible work package |
| Status | Initial status: Draft |

### Step 3: Create ICD Directory

```bash
mkdir -p icds/{category}/ICD-XX/diagrams
cp templates/ICD_TEMPLATE.md icds/{category}/ICD-XX/ICD-XX.md
touch icds/{category}/ICD-XX/openapi.yaml
```

Where `{category}` is one of:
- `portal-level` (ICD-01 to ICD-04)
- `governance` (ICD-05 to ICD-07)
- `application` (ICD-08 to ICD-13)
- `dataspace` (ICD-14 to ICD-17)
- `data-source` (ICD-18-1, ICD-18-2)

### Step 4: Complete the ICD Document

Follow the embedded instructions in the template to complete all 13 mandatory sections and 4 annexes. Consult `templates/TEMPLATE_INSTRUCTIONS.md` for detailed guidance.

### Step 5: Execute Quality Checklist

Complete all items in Annex D of the ICD template before submission.

## Pull Request Requirements

Each pull request shall include:

| Requirement | Description |
|-------------|-------------|
| Updated catalogue entry | Entry in `catalogue/ICD_CATALOGUE.md` |
| Completed ICD document | All mandatory sections populated |
| OpenAPI specification | For HTTP/REST interfaces |
| Completed quality checklist | Annex D with all items checked |
| Reviewer assignment | At least one reviewer from a different work package |

### Pull Request Title Format

```
[ICD-XX] <type>: <description>
```

Where `<type>` is one of:
- `feat`: New capability or endpoint
- `fix`: Correction to specification
- `docs`: Documentation improvement
- `refactor`: Restructuring without functional change
- `breaking`: Breaking change to interface contract

**Examples:**
- `[ICD-05] feat: Initial SPIP Agent to Dataspace Connector specification`
- `[ICD-08] fix: Correct authentication flow sequence diagram`
- `[ICD-14] breaking: Update EDC catalog exchange protocol to DSP 2024-1`

## Commit Message Format

Commit messages shall follow the format:

```
[ICD-XX] <type>: <description>

<optional body>

<optional footer>
```

## Review Process

### Reviewer Responsibilities

Reviewers shall verify:

1. All mandatory sections are completed
2. Writing style requirements are followed (British English, no subjunctive mood, no personal pronouns)
3. Technical accuracy of interface specifications
4. Consistency with referenced D2.2 requirements
5. OpenAPI specification validity (for REST interfaces)
6. Sequence diagrams accurately reflect interaction patterns

### Review Timeline

| Phase | Duration |
|-------|----------|
| Initial review | 5 working days |
| Author revision | 3 working days |
| Final approval | 2 working days |

## Quality Assurance Checklist

The following criteria shall be verified prior to pull request submission:

| Check | Criterion |
|-------|-----------|
| ☐ | Units of measure specified for all numerical fields |
| ☐ | Semantic IDs (IRDIs) provided for AAS-compliant fields |
| ☐ | Environment variables listed for DevOps deployment |
| ☐ | Circuit breaker thresholds defined for resilience |
| ☐ | PII fields flagged and retention policies defined |
| ☐ | ODRL policies defined for dataspace interfaces (ICD-14 to ICD-17) |
| ☐ | MQTT topics, QoS, and LWT defined for IoT interfaces |
| ☐ | Error handling appropriate for protocol (RFC 9457 or DLQ) |
| ☐ | Health check mechanism defined (HTTP endpoint or MQTT LWT) |
| ☐ | Interface dependencies and versioning documented |
| ☐ | British English and IEEE style followed throughout |
| ☐ | No subjunctive mood, personal pronouns, or filler words |
| ☐ | Abbreviations defined once and listed in Section 3 |
| ☐ | Performance targets use specific numerical values |

## Version Control and Change Management

### Semantic Versioning

ICD documents shall follow semantic versioning (MAJOR.MINOR):

| Version Type | Description |
|--------------|-------------|
| MAJOR | Breaking changes to the interface contract |
| MINOR | Backwards-compatible additions or clarifications |

### Version History Updates

All interface changes shall be captured in Section 13 (Version History) of the ICD. Backwards compatibility impact shall be stated explicitly within the version history notes.

## Contact

For questions regarding contribution procedures, contact the WP4 Interface Design and Development Team.
