# Test Cases

This file documents the synthetic test cases used in Demo 1: Pharma / MedTech M365 Controlled Document Intake.

The goal is to prove that the workflow can receive CSV email intake files, save attachments to SharePoint, create tracker records, apply deterministic validation, route files by final status, and send final reply emails.

---

## 1. Test Scope

The test scope covers:

- email intake
- CSV attachment handling
- SharePoint file creation
- SharePoint tracker record creation
- deterministic validation logic
- unsupported email attachment gate
- final status assignment
- review owner assignment
- payment hold flag
- fulfilment hold flag
- file routing
- final reply email

---

## 2. Test Data Location

Synthetic business documents:

```text
/sample-data/txt-business-docs/
```

CSV email templates:

```text
/sample-data/csv-email-templates/
```

Each CSV template contains one synthetic test row.

---

## 3. Test Case Summary

| Case ID | Scenario | Expected Status | Expected Validation Result | Expected Review Owner | Expected Folder |
|---|---|---|---|---|---|
| CASE-01 | Complete CMO split-shipment intake | Complete | Pass | Not required | `/Processed` |
| CASE-02 | Missing EPCIS file status | Needs Review | Warning | QA Serialization | `/NeedsReview` |
| CASE-03 | Split-shipment quantity mismatch | Needs Review | Warning | Supply Chain | `/NeedsReview` |
| CASE-04 | Unsupported file type declared in CSV metadata | Rejected | Fail | Demo Owner | `/Rejected` |
| CASE-05 | API material delay note | Needs Review | Warning | Procurement | `/NeedsReview` |

---

## 4. CASE-01: Complete CMO Split-Shipment Intake

### Scenario

Taiwan CMO sends a complete split-shipment intake case. Required metadata is present, EPCIS readiness is available, aggregation is completed, and the remaining shipment information is known.

### Input

```text
/sample-data/csv-email-templates/case-01-complete-cmo-split-shipment-intake.csv
```

### Expected Result

| Field | Expected Value |
|---|---|
| DemoCaseID | CASE-01 |
| Status | Complete |
| ValidationResult | Pass |
| ReviewRequired | false |
| ReviewOwner | Not required / blank |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | false |
| MissingFields | No exception detected / blank |
| ErrorReason | blank |
| Destination Folder | `/Processed` |

### Evidence

```text
/screenshots/08-final-reply-case01-complete.png
/screenshots/10-folder-routing-processed-needsreview-rejected.png
```

---

## 5. CASE-02: Missing EPCIS File Status

### Scenario

Taiwan CMO sends ASN / invoice-related intake information, but EPCIS file status is not confirmed even though EPCIS is required.

### Input

```text
/sample-data/csv-email-templates/case-02-missing-epcis-file-status.csv
```

### Expected Result

| Field | Expected Value |
|---|---|
| DemoCaseID | CASE-02 |
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewRequired | true |
| ReviewOwner | QA Serialization |
| DSCSAReviewRequired | true |
| PaymentHoldFlag | true |
| FulfilmentHoldFlag | true |
| MissingFields | EPCISFileStatus not confirmed, missing, or unknown while EPCISRequired is true |
| ErrorReason | blank |
| Destination Folder | `/NeedsReview` |

### Business Meaning

QA / serialization review is required before fulfilment and payment readiness can be treated as complete.

---

## 6. CASE-03: Split-Shipment Quantity Mismatch

### Scenario

Taiwan CMO sends split-shipment intake information, but remaining quantity or estimated remaining ship date is missing.

### Input

```text
/sample-data/csv-email-templates/case-03-split-shipment-quantity-mismatch.csv
```

### Expected Result

| Field | Expected Value |
|---|---|
| DemoCaseID | CASE-03 |
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewRequired | true |
| ReviewOwner | Supply Chain |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | true |
| FulfilmentHoldFlag | true |
| MissingFields | Split shipment missing RemainingQuantity or EstimatedRemainingShipDate |
| ErrorReason | blank |
| Destination Folder | `/NeedsReview` |

### Business Meaning

Supply chain review is required because distributor fulfilment readiness cannot be confirmed if the remaining shipment plan is incomplete.

---

## 7. CASE-04: Unsupported File Type

### Scenario

The CSV metadata declares an unsupported referenced business file type.

### Input

```text
/sample-data/csv-email-templates/case-04-unsupported-file-type.csv
```

### Expected Result

| Field | Expected Value |
|---|---|
| DemoCaseID | CASE-04 |
| Status | Rejected |
| ValidationResult | Fail |
| ReviewRequired | true |
| ReviewOwner | Demo Owner |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | false |
| MissingFields | Unsupported file type |
| ErrorReason | Unsupported file type |
| Destination Folder | `/Rejected` |

### Evidence

```text
/screenshots/09-final-reply-case04-rejected.png
/screenshots/10-folder-routing-processed-needsreview-rejected.png
```

### Business Meaning

Unsupported intake should not silently enter a normal operations workflow. It should be rejected and made visible.

---

## 8. CASE-05: API Material Delay Notice

### Scenario

China API supplier sends a material delay notice that may affect Taiwan CMO production and downstream distributor fulfilment.

### Input

```text
/sample-data/csv-email-templates/case-05-api-material-delay-note.csv
```

### Expected Result

| Field | Expected Value |
|---|---|
| DemoCaseID | CASE-05 |
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewRequired | true |
| ReviewOwner | Procurement |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | true |
| MissingFields | API material delay requires procurement review |
| ErrorReason | blank |
| Destination Folder | `/NeedsReview` |

### Business Meaning

Procurement review is required because API material delay may affect CMO production readiness and distributor fulfilment.

---

## 9. Unsupported Actual Email Attachment Gate

### Scenario

Sender attaches a file with an unsupported actual attachment extension, such as `.zip`, `.exe`, or `.dat`.

### Expected Result

| Step | Expected Behavior |
|---|---|
| Attachment format gate | Detects unsupported attachment |
| Reply email | Sender receives supported-format message |
| Flow result | Terminates successfully |
| SharePoint file | No file saved for unsupported attachment |
| SharePoint tracker | No tracker item created for unsupported attachment |

### Supported Actual Attachment Formats

```text
TXT, PDF, DOCX, XLSX, CSV, XML, JSON, EDI, X12, IDOC
```

### Note

The current working intake flow processes CSV metadata templates. Other supported formats are treated as referenced or accepted business document formats for the demo boundary, not parsed content.

---

## 10. Acceptance Criteria

Demo 1 passes when:

| Area | Acceptance Criteria |
|---|---|
| Email intake | Demo email with CSV attachment triggers the flow |
| Attachment storage | CSV attachment is saved to SharePoint `/Incoming` |
| Tracker creation | A SharePoint tracker item is created |
| Validation | Final status is calculated by deterministic rules |
| Review routing | ReviewOwner is assigned based on exception type |
| Hold flags | PaymentHoldFlag and FulfilmentHoldFlag are calculated |
| File movement | CSV is moved to the correct final folder |
| Reply email | Sender receives RequestID, DemoCaseID, final status, validation result, owner, and reason |
| Dashboard | SharePoint views/dashboard reflect final statuses |
| Limitations | README and limitations document state non-production scope clearly |
