# DATA4CIRC Interface Control Document Catalogue

[![ICD Validation](https://img.shields.io/badge/ICD%20Validation-Passing-green)](https://github.com/hadijannat/DATA4CIRC-ICD-Catalogue/actions)
[![Documentation](https://img.shields.io/badge/Documentation-Available-blue)](./catalogue/ICD_CATALOGUE.md)
[![License](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey)](./LICENSE)

This repository contains the Interface Control Document (ICD) specifications for the DATA4CIRC federated dataspace platform. The ICD Catalogue defines interfaces between subsystems across all work packages within the EU Horizon Europe DATA4CIRC project.

## Repository Structure

| Directory | Description |
|-----------|-------------|
| `catalogue/` | Master ICD register and identifier authority |
| `templates/` | Universal ICD template with completion instructions |
| `schemas/` | JSON Schema and OpenAPI template definitions |
| `icds/` | Completed ICD specifications by category |

### Repository Layout

```
DATA4CIRC-ICD-Catalogue/
├── README.md                          # Repository documentation
├── CONTRIBUTING.md                    # Contribution guidelines
├── LICENSE                            # CC BY-SA 4.0 License
├── .github/
│   └── workflows/
│       ├── validate-icd.yml           # ICD validation workflow
│       └── generate-catalogue.yml     # Catalogue generation workflow
├── catalogue/
│   └── ICD_CATALOGUE.md               # Master interface register
├── templates/
│   ├── ICD_TEMPLATE.md                # Universal ICD template
│   └── TEMPLATE_INSTRUCTIONS.md       # Template completion guide
├── schemas/
│   ├── icd-schema.json                # JSON Schema for ICD validation
│   └── openapi-template.yaml          # OpenAPI template baseline
└── icds/
    ├── portal-level/                  # ICD-1 to ICD-4
    ├── governance/                    # ICD-5 to ICD-7
    ├── application/                   # ICD-8 to ICD-13
    ├── dataspace/                     # ICD-14 to ICD-17
    └── data-source/                   # ICD-18.1, ICD-18.2
```

## Interface Categories

| Category | ICD Range | Description |
|----------|-----------|-------------|
| Portal-Level | ICD-1 to ICD-4 | SSO authentication, deep linking, role propagation |
| Governance | ICD-5 to ICD-7 | SPIP infrastructure, policy enforcement |
| Application | ICD-8 to ICD-13 | Portal-application and inter-application communication |
| Dataspace | ICD-14 to ICD-17 | EDC connector-to-connector federation |
| Data Source | ICD-18.x | ERP, PLM, MES, IoT integration |

## Creating a New ICD

1. Register the interface in `catalogue/ICD_CATALOGUE.md`
2. Copy `templates/ICD_TEMPLATE.md` to the appropriate category folder
3. Follow the instructions in `templates/TEMPLATE_INSTRUCTIONS.md`
4. Submit a pull request for peer review

### Command Line Quick Start

```bash
# Clone the repository
git clone https://github.com/hadijannat/DATA4CIRC-ICD-Catalogue.git
cd DATA4CIRC-ICD-Catalogue

# Create feature branch
git checkout -b icd/ICD-05/spip-agent-connector

# Create ICD directory structure
mkdir -p icds/governance/ICD-05/diagrams
cp templates/ICD_TEMPLATE.md icds/governance/ICD-05/ICD-05.md
touch icds/governance/ICD-05/openapi.yaml

# Edit the ICD document
# [Complete all sections following template instructions]

# Commit and push
git add .
git commit -m "[ICD-05] feat: Initial SPIP Agent to Dataspace Connector interface specification"
git push origin icd/ICD-05/spip-agent-connector

# Create pull request for peer review
```

## Writing Style Requirements

All content within this repository shall conform to the following mandatory writing rules:

| Rule | Guidance |
|------|----------|
| British English | Use British spelling conventions (serialisation, synchronise) |
| No personal pronouns | Avoid first-person references (the API is implemented, not "we implement") |
| No spatial references | Reference specific sections instead of spatial terms (In Section 3, not "above") |
| No temporal references | Use milestone references instead of relative time (In M18, not "currently") |
| No subjunctive mood | Use definitive language (shall, is, provides, not "could" or "might") |
| No filler words | Remove qualifiers or use precise terms (remove "greatly", "heavily", "very") |
| No colloquialisms | Use formal terminology (retrieve, store, not "get", "put") |
| No em dashes | Use commas or parentheses instead |
| No ambiguous quantifiers | Use specific values (< 200 ms, 10 MB maximum) |
| Lowercase unless proper noun | Use lower case for common nouns (digital product passport tool) |
| Units mandatory | All numerical values shall include units of measure |

## Normative References

The following specifications define normative requirements for ICD documents:

- RFC 9457: Problem Details for HTTP APIs
- OpenAPI Specification v3.1.0
- TLS 1.3 (RFC 8446)
- OAuth 2.0 (RFC 6749) and OpenID Connect Core 1.0
- IDTA AAS Metamodel Specification (IDTA-01001)
- W3C ODRL Information Model 2.2
- OASIS MQTT Version 5.0

## Project References

| Document | Description |
|----------|-------------|
| D2.2 | DATA4CIRC Requirements and Specifications |
| D4.1 | Platform Architecture and Open-Source Protocols |

## Validation Workflows

This repository includes automated GitHub Actions workflows that validate ICD submissions:

- **Markdown Linting**: Validates Markdown structure and formatting
- **OpenAPI Validation**: Validates OpenAPI specification files against the OpenAPI v3.1.0 standard
- **Link Checking**: Verifies internal cross-references and external URLs

## Contributing

Contributions are managed through pull requests. Each submission shall include:

- Updated catalogue entry in `catalogue/ICD_CATALOGUE.md`
- Completed ICD document with all mandatory sections
- OpenAPI specification (for HTTP/REST interfaces)
- Completed Quality Checklist (Annex D)
- At least one reviewer from a different work package

Refer to [CONTRIBUTING.md](./CONTRIBUTING.md) for detailed contribution guidelines.

## License

This work is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License.

## Acknowledgements

This project receives funding from the European Union's Horizon Europe research and innovation programme under Grant Agreement No. [Grant Number].

---

**Repository Maintainer**: WP4 Interface Design and Development Team

**Contact**: [Project Contact Information]
