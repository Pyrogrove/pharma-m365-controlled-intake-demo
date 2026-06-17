# Architecture

This document explains the architecture for Demo 1: Pharma / MedTech M365 Controlled Document Intake.

The architecture follows a deterministic workflow spine first. AI is not active in the current implementation and is documented only as a possible future enhancer.

---

## 1. Architecture Objective

The objective is to demonstrate a controlled Microsoft 365 workflow pattern for supplier / CMO intake.

The pattern:

```text
Email intake
-> attachment format gate
-> SharePoint storage
-> CSV metadata parse
-> SharePoint tracker item
-> deterministic validation
-> status / owner / hold flags
-> file routing
-> final reply email
-> dashboard visibility
```

---

## 2. Business Scenario

A US pharmaceutical brand owner receives synthetic intake documents from:

- Taiwan CMO
- China API supplier

The intake relates to:

- PO / ASN / invoice references
- batch / lot tracking
- serialized finished goods
- DSCSA / EPCIS readiness metadata
- split shipment handling
- US 3PL readiness
- payment hold
- fulfilment hold
- procurement / QA serialization / supply chain review

---

## 3. Current Technical Architecture

```text
External sender
    |
    v
Office 365 Outlook
"When a new email arrives (V3)"
    |
    v
Subject filter for Demo 1 email
    |
    v
Attachment check
    |
    v
Scope: v3.2 Email Attachment Format Gate
    |
    +--> Unsupported attachment found
    |       -> Reply to sender
    |       -> Terminate
    |
    +--> No unsupported attachment found
            |
            v
        Filter CSV attachments
            |
            v
        Has CSV attachment?
            |
            +--> No
            |       -> Reply: no CSV attachment found
            |
            +--> Yes
                    |
                    v
                Apply to each CSV attachment
                    |
                    v
                Get attachment content
                    |
                    v
                Create file in SharePoint /Incoming
                    |
                    v
                CSV parse
                    |
                    v
                Scope: CSV Parse and v3 Deterministic Validation Logic
                    |
                    v
                Create item in SharePoint Tracker
                    |
                    v
                Scope: v4 Route CSV File by Final Status
                    |
                    +--> Complete      -> /Processed
                    +--> Needs Review  -> /NeedsReview
                    +--> Rejected      -> /Rejected
                    |
                    v
                v5 final reply email
```

---

## 4. Microsoft 365 Components

| Component | Role |
|---|---|
| Office 365 Outlook | Receives demo intake emails |
| Power Automate cloud | Runs intake, validation, routing, and reply workflow |
| SharePoint Document Library | Stores CSV attachments and routed files |
| SharePoint List | Tracks workflow state and business metadata |
| SharePoint Views / Dashboard | Shows operational visibility by status, owner, EPCIS readiness, and hold flags |
| GitHub | Stores sample data, screenshots, documentation, and portfolio explanation |

---

## 5. SharePoint Document Library

Library:

```text
Pharma Intake Documents
```

Folders:

```text
/Incoming
/Processed
/NeedsReview
/Rejected
/Archive
/csv template
```

Folder purpose:

| Folder | Purpose |
|---|---|
| `/Incoming` | Initial landing folder for saved CSV intake attachments |
| `/Processed` | Final location for completed intake records |
| `/NeedsReview` | Final location for records requiring human review |
| `/Rejected` | Final location for unsupported or rejected records |
| `/Archive` | Reserved for future archive handling |
| `/csv template` | Stores reusable CSV intake templates in SharePoint |

---

## 6. SharePoint Tracker

List:

```text
Pharma Intake Tracker
```

The tracker stores:

- runtime RequestID
- DemoCaseID
- sender metadata
- partner role
- business document type
- PO / ASN / invoice references
- batch / lot references
- shipment and quantity data
- EPCIS readiness fields
- validation result
- review owner
- review required flag
- payment hold flag
- fulfilment hold flag
- status
- missing fields / reason
- timestamps
- RecordCount for reporting

---

## 7. Deterministic Workflow Spine

The deterministic workflow spine is the core of this demo.

It performs:

```text
Accepted input check
-> CSV attachment save
-> metadata parsing
-> rule-based validation
-> status assignment
-> owner assignment
-> hold flag assignment
-> file routing
-> reply email
```

The flow does not rely on AI for final decisions.

---

## 8. Validation Architecture

Validation is split into a staged logic pattern:

```text
Raw CSV fields
-> normalized helper values
-> rule flags
-> final outcome fields
-> SharePoint tracker mapping
```

Examples of helper and rule fields:

| Logic Type | Example |
|---|---|
| Raw field | DemoCaseID, PartnerRole, BusinessDocumentType |
| Normalized field | FileExtension |
| Rule flag | IsUnsupportedFileType |
| Rule flag | IsMissingEPCISStatus |
| Rule flag | IsSplitShipmentIncomplete |
| Rule flag | IsAPIDelayNotice |
| Final output | FinalStatus |
| Final output | FinalValidationResult |
| Final output | FinalReviewOwner |
| Final output | FinalReason |

This makes the flow easier to troubleshoot in run history.

---

## 9. File Routing Architecture

After the tracker item is created, the file is moved according to final status.

| Final Status | Destination |
|---|---|
| Complete | `/Processed` |
| Needs Review | `/NeedsReview` |
| Rejected | `/Rejected` |

Default switch branch is intentionally left empty.

---

## 10. Reply Email Architecture

The final reply email includes:

- RequestID
- DemoCaseID
- Final Status
- Validation Result
- Review Owner
- Reason
- synthetic demo disclaimer

The email confirms that the file has been saved, logged, and routed based on deterministic validation rules.

---

## 11. Dashboard Architecture

The dashboard is SharePoint-native.

It uses SharePoint list views instead of Power BI for the current build.

Example views:

- All Items
- Open Items
- Needs Review
- DSCSA Review
- Payment Hold
- Fulfilment Hold
- Status Summary
- Review Owner Summary
- EPCIS Summary
- Payment Hold Summary
- Fulfilment Hold Summary

Power BI is a possible future enhancement, but not required for this demo.

---

## 12. AI Boundary

AI is not active in the current build.

Potential future AI role:

```text
Unstructured document
-> AI-assisted extraction / classification / summary
-> deterministic validation rules
-> human review
-> controlled workflow action
```

AI should remain an enhancer, not the decision authority.

---

## 13. Excluded Architecture

The current architecture excludes:

- RAG
- n8n
- local LLM
- Qdrant
- FastAPI
- Graph API
- custom authentication
- production deployment pipeline
- ERP integration
- EDI integration
- AP automation
- AR automation
- real EPCIS parsing
- real regulated data

---

## 14. Design Principle

The design principle is:

```text
Deterministic workflow spine first.
AI only as a controlled enhancer.
Human review for risky decisions.
Auditability and exception handling built into the workflow.
```
