# Exception Rules

This file documents the exception taxonomy and routing logic used in Demo 1.

---

## 1. Exception Design Principle

The workflow should not treat every intake as complete.

When required metadata is missing, risky, unsupported, or operationally unclear, the item should be routed to a human owner with a clear reason.

The exception pattern is:

```text
Exception detected
-> Status assigned
-> ValidationResult assigned
-> ReviewOwner assigned
-> hold flags assigned
-> reason captured
-> file routed
-> final reply email sent
```

---

## 2. Exception Taxonomy

| Exception Type | Example | Owner | Status |
|---|---|---|---|
| Missing EPCIS readiness | EPCISRequired is true but EPCISFileStatus is not confirmed | QA Serialization | Needs Review |
| Split-shipment incomplete | Remaining quantity or estimated remaining ship date is missing | Supply Chain | Needs Review |
| API material delay | China API supplier reports delay affecting CMO production | Procurement | Needs Review |
| Unsupported file type | Referenced file extension is unsupported | Demo Owner | Rejected |
| Unsupported actual attachment | Email attachment extension is unsupported | Demo Owner / sender reply | Flow terminates before tracker creation |
| System failure | File or tracker creation fails | Demo Owner | Failed, future enhancement |

---

## 3. Missing EPCIS Readiness

### Trigger

```text
EPCISRequired = Yes
EPCISFileStatus = blank / Not Confirmed / Missing / Unknown
```

### Routing

| Field | Value |
|---|---|
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewOwner | QA Serialization |
| DSCSAReviewRequired | true |
| PaymentHoldFlag | true |
| FulfilmentHoldFlag | true |
| Reason | EPCISFileStatus not confirmed, missing, or unknown while EPCISRequired is true |

### Rationale

Serialized product movement may require EPCIS readiness review before fulfilment or payment readiness is treated as complete.

---

## 4. Split-Shipment Incomplete

### Trigger

```text
ShipmentType = Split Shipment
RemainingQuantity is blank
or
EstimatedRemainingShipDate is blank
```

### Routing

| Field | Value |
|---|---|
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewOwner | Supply Chain |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | true |
| FulfilmentHoldFlag | true |
| Reason | Split shipment missing RemainingQuantity or EstimatedRemainingShipDate |

### Rationale

Split shipment without complete remaining-quantity or remaining-date information creates fulfilment and planning uncertainty.

---

## 5. API Material Delay

### Trigger

```text
PartnerRole = API Supplier
BusinessDocumentType = API Delay Notice
```

or synthetic test case:

```text
DemoCaseID = CASE-05
```

### Routing

| Field | Value |
|---|---|
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewOwner | Procurement |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | true |
| Reason | API material delay requires procurement review |

### Rationale

API material delays may affect CMO production continuity and downstream distributor fulfilment.

---

## 6. Unsupported Referenced File Type

### Trigger

Referenced business file extension is not supported.

Supported list:

```text
TXT, PDF, DOCX, XLSX, CSV, XML, JSON, EDI, X12, IDOC
```

### Routing

| Field | Value |
|---|---|
| Status | Rejected |
| ValidationResult | Fail |
| ReviewOwner | Demo Owner |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | false |
| Reason | Unsupported file type |
| ErrorReason | Unsupported file type |

### Rationale

Unsupported business-document formats should not proceed as normal intake.

---

## 7. Unsupported Actual Email Attachment

### Trigger

The sender attaches an actual email attachment with unsupported extension.

Examples:

```text
.exe
.zip
.dat
```

### Routing

| Step | Result |
|---|---|
| Unsupported attachment detected | true |
| Reply to sender | sent |
| Flow termination | succeeded |
| SharePoint file creation | skipped |
| Tracker item creation | skipped |

### Rationale

The email-level attachment gate prevents unsupported files from entering SharePoint storage or tracker creation.

---

## 8. Review Owner Rules

| Trigger | ReviewOwner |
|---|---|
| Missing EPCIS status | QA Serialization |
| Split shipment incomplete | Supply Chain |
| API material delay | Procurement |
| Unsupported referenced file type | Demo Owner |
| Complete case | blank / Not required |

---

## 9. Hold Flag Rules

| Trigger | PaymentHoldFlag | FulfilmentHoldFlag |
|---|---:|---:|
| Missing EPCIS status | true | true |
| Split shipment incomplete | true | true |
| API material delay | false | true |
| Unsupported referenced file type | false | false |
| Complete case | false | false |

---

## 10. Exception Priority

If multiple rules could apply, the intended priority is:

```text
1. Unsupported file type
2. Missing EPCIS status
3. Split shipment incomplete
4. API material delay
5. Complete
```

This ensures rejected input does not continue as a normal review item.

---

## 11. Current Boundary

The demo does not include a closed-loop review-resolution process.

For example, the current demo does not automatically move a `Needs Review` case to `Complete` after a reviewer resolves it.

That is a future enhancement and would require:

- reviewer action capture
- approval or resolution fields
- updated status transition rules
- audit trail
- notification logic
- possible dashboard ageing
