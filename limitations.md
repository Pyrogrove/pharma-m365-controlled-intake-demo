# Limitations

This repository is a portfolio prototype using synthetic data only.

It demonstrates a Microsoft 365 controlled document-intake workflow pattern for pharma / medtech supply-chain operations. It is not a production compliance system, not a regulated system, and not a validated GxP implementation.

---

## 1. Data Boundary

This demo uses synthetic data only.

Do not submit or store:

- real client data
- real supplier data
- real patient data
- real employee data
- real regulated product data
- real EPCIS data
- real EDI / X12 / IDOC transaction data
- employer-confidential data
- commercially sensitive shipment, invoice, PO, lot, or serialization data

All sample CMO, supplier, PO, ASN, invoice, batch, lot, EPCIS, and shipment references are fictional.

---

## 2. Compliance Boundary

This demo is not:

- GxP validated
- 21 CFR Part 11 compliant
- DSCSA compliant
- an EPCIS validation system
- a production serialization system
- a production quality-management system
- a validated document management system
- a validated electronic record / electronic signature system

A production implementation would require formal validation planning, access control, audit-retention policy, electronic record controls, tenant governance, security review, SOP alignment, and regulated-system assessment.

---

## 3. Functional Boundary

The demo currently processes one-row CSV intake templates sent as email attachments.

It does not perform:

- actual EPCIS XML / JSON parsing
- actual EDI / X12 / IDOC parsing
- ERP integration
- EDI gateway integration
- AP payment approval
- AR invoicing
- purchase order creation
- ASN creation
- shipment confirmation
- 3PL system integration
- supplier portal workflow
- production email authentication design
- production DLP configuration
- production exception escalation

The supported file-format logic only validates declared or attached file extensions for demo intake purposes.

Supported referenced business document formats:

```text
TXT, PDF, DOCX, XLSX, CSV, XML, JSON, EDI, X12, IDOC
```

This does not mean the demo parses those formats.

---

## 4. Power Automate Boundary

The current build uses Microsoft 365 standard connectors.

The current build does not use:

- Graph API
- HTTP premium connector
- Azure Functions
- FastAPI
- n8n
- local LLM
- RAG
- Qdrant
- autonomous agent workflows
- custom authentication
- production deployment pipelines

The flow is intentionally built as a controlled M365 workflow demo rather than a custom application.

---

## 5. AI Boundary

AI is not active in the current implementation.

The current workflow uses deterministic Power Automate logic for validation, status assignment, owner routing, hold flags, file movement, and final reply email.

AI is documented only as a possible future enhancer for:

- extracting metadata from unstructured documents
- classifying document type
- summarizing supplier or CMO notes
- drafting internal follow-up text
- suggesting review owner before human confirmation

AI should not be used for:

- final QA disposition
- DSCSA compliance signoff
- EPCIS validation
- payment approval
- fulfilment release
- external communication without human review
- overriding deterministic validation rules

---

## 6. SharePoint Boundary

SharePoint is used as a lightweight workflow-control layer.

This demo does not claim that SharePoint Lists replace:

- ERP master data
- validated serialization repositories
- quality systems
- warehouse management systems
- transport management systems
- AP / AR systems
- full document management systems

The SharePoint tracker is used to demonstrate status lifecycle, exception routing, and management visibility.

---

## 7. Known Technical Limitations

Current limitations:

- CSV parsing is simple and based on field splitting.
- The demo expects controlled one-row CSV templates.
- Long free-text CSV fields with embedded commas are not fully supported.
- FileLink is treated as non-blocking.
- Final file links may need production hardening if files are moved after initial creation.
- Multi-row CSV processing is not included.
- A dedicated exception log list is not included.
- Human review resolution workflow is not included.
- Dashboard is built using SharePoint-native views, not Power BI.

---

## 8. Production Hardening Required

A production version would require:

- tenant-specific security review
- access control design
- DLP policy review
- audit-retention configuration
- regulated data classification
- validation documentation if used in regulated workflow
- approved AI service configuration if AI is added
- exception logging strategy
- support and monitoring model
- backup and export strategy
- user acceptance testing
- change-control process
- production owner and admin handover

---

## 9. Intended Use

This repo is intended for:

- portfolio demonstration
- recruiter-visible proof of hands-on M365 workflow automation
- SME automation concept demonstration
- solution-consulting storytelling
- controlled workflow design reference

It is not intended for direct production use.
