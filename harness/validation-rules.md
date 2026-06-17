# Validation Rules

This file documents the deterministic validation rules used in Demo 1.

The workflow does not trust CSV-provided final status, review owner, validation result, or hold flags. These final outcome fields are calculated inside Power Automate.

---

## 1. Validation Principle

The validation layer follows this pattern:

```text
CSV metadata
-> normalize key fields
-> evaluate deterministic rules
-> calculate final outcome
-> write final outcome to SharePoint tracker
-> route CSV file by final status
-> send final reply email
```

---

## 2. Calculated Fields

Power Automate calculates these final fields:

| Field | Purpose |
|---|---|
| Status | Final workflow state |
| ValidationResult | Pass, Warning, or Fail |
| ReviewOwner | Human owner for review |
| ReviewRequired | Whether review is required |
| DSCSAReviewRequired | Whether QA / serialization review is required |
| PaymentHoldFlag | Whether payment should be held |
| FulfilmentHoldFlag | Whether fulfilment should be held |
| MissingFields | Short validation reason |
| ErrorReason | Error or rejection reason |
| LastUpdatedDateTime | Runtime update timestamp |
| RecordCount | Set to 1 for reporting |

---

## 3. Supported Referenced File Formats

Supported referenced business document formats:

```text
TXT, PDF, DOCX, XLSX, CSV, XML, JSON, EDI, X12, IDOC
```

Excluded from supported list:

```text
DAT
EXE
ZIP
```

Reason for excluding `.dat`:

- `.dat` is too generic
- it can represent many unrelated export types
- real use would require source validation and stricter partner rules

---

## 4. Rule VR-001: Unsupported File Type

### Condition

The referenced or declared business file extension is not in the supported extension list.

### Outcome

| Field | Value |
|---|---|
| Status | Rejected |
| ValidationResult | Fail |
| ReviewOwner | Demo Owner |
| ReviewRequired | true |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | false |
| MissingFields | Unsupported file type |
| ErrorReason | Unsupported file type |

### Example Case

```text
CASE-04
```

---

## 5. Rule VR-002: Missing EPCIS Status

### Condition

`EPCISRequired = Yes`

and

`EPCISFileStatus` is one of:

```text
blank
Not Confirmed
Missing
Unknown
```

### Outcome

| Field | Value |
|---|---|
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewOwner | QA Serialization |
| ReviewRequired | true |
| DSCSAReviewRequired | true |
| PaymentHoldFlag | true |
| FulfilmentHoldFlag | true |
| MissingFields | EPCISFileStatus not confirmed, missing, or unknown while EPCISRequired is true |
| ErrorReason | blank |

### Example Case

```text
CASE-02
```

---

## 6. Rule VR-003: Split Shipment Incomplete

### Condition

`ShipmentType = Split Shipment`

and either:

```text
RemainingQuantity is blank
EstimatedRemainingShipDate is blank
```

### Outcome

| Field | Value |
|---|---|
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewOwner | Supply Chain |
| ReviewRequired | true |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | true |
| FulfilmentHoldFlag | true |
| MissingFields | Split shipment missing RemainingQuantity or EstimatedRemainingShipDate |
| ErrorReason | blank |

### Example Case

```text
CASE-03
```

---

## 7. Rule VR-004: API Material Delay Notice

### Condition

Either:

```text
DemoCaseID = CASE-05
```

or:

```text
PartnerRole = API Supplier
BusinessDocumentType = API Delay Notice
```

### Outcome

| Field | Value |
|---|---|
| Status | Needs Review |
| ValidationResult | Warning |
| ReviewOwner | Procurement |
| ReviewRequired | true |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | true |
| MissingFields | API material delay requires procurement review |
| ErrorReason | blank |

### Example Case

```text
CASE-05
```

---

## 8. Rule VR-005: Complete Case

### Condition

No rejection or review-triggering rule is true.

### Outcome

| Field | Value |
|---|---|
| Status | Complete |
| ValidationResult | Pass |
| ReviewOwner | blank / Not required |
| ReviewRequired | false |
| DSCSAReviewRequired | false |
| PaymentHoldFlag | false |
| FulfilmentHoldFlag | false |
| MissingFields | blank / No exception detected |
| ErrorReason | blank |

### Example Case

```text
CASE-01
```

---

## 9. Rule Priority

Rule priority:

```text
1. Unsupported file type
2. Missing EPCIS status
3. Split shipment incomplete
4. API material delay
5. Complete
```

Unsupported file type has highest priority because unsupported intake should not continue as a normal operational case.

---

## 10. Reason Text

The final reason text is used in:

- MissingFields
- final reply email
- tracker screenshot evidence

Reason mapping:

| Trigger | Reason |
|---|---|
| Unsupported file type | Unsupported file type |
| Missing EPCIS status | EPCISFileStatus not confirmed, missing, or unknown while EPCISRequired is true |
| Split shipment incomplete | Split shipment missing RemainingQuantity or EstimatedRemainingShipDate |
| API material delay | API material delay requires procurement review |
| Complete | No exception detected |

---

## 11. Boolean Mapping

SharePoint Yes/No fields must receive true/false values, not text values such as `Yes` or `No`.

Calculated boolean fields include:

- ReviewRequired
- DSCSAReviewRequired
- PaymentHoldFlag
- FulfilmentHoldFlag

---

## 12. Date Mapping

SharePoint DateTime fields use:

```text
utcNow()
```

Reason:

- Power Automate stores DateTime in UTC
- SharePoint displays DateTime based on site/user regional settings
- converting to Singapore time before writing to SharePoint can create display confusion

Filename timestamps may use Singapore time because filenames are plain text and SharePoint does not reinterpret them.
