# SharePoint Schema

This document describes the SharePoint list and library structure used in Demo 1.

---

## 1. SharePoint Site

Site:

```text
AI Workflow Demo Portfolio
```

---

## 2. Document Library

Library:

```text
Pharma Intake Documents
```

Folders:

| Folder | Purpose |
|---|---|
| `/Incoming` | Initial landing folder for CSV email attachments |
| `/Processed` | Final folder for completed records |
| `/NeedsReview` | Final folder for records requiring human review |
| `/Rejected` | Final folder for rejected records |
| `/Archive` | Reserved for future archive handling |
| `/csv template` | Stores reusable CSV email templates |

---

## 3. SharePoint List

List:

```text
Pharma Intake Tracker
```

The list acts as the workflow control table.

---

## 4. Core Tracker Columns

| Column Name | Type | Purpose |
|---|---|---|
| RequestID | Single line of text | Runtime-generated unique request ID |
| DemoCaseID | Single line of text | Synthetic case identifier, such as CASE-01 |
| ReceivedDateTime | Date and time | Email intake timestamp |
| SourceChannel | Choice / text | Runtime source channel, usually Email |
| SenderName | Single line of text | Actual email sender display name |
| SenderEmail | Single line of text | Actual email sender address |
| SourcePartner | Single line of text | Synthetic CMO / supplier name |
| PartnerRole | Choice / text | CMO, API Supplier, 3PL, Distributor, Other |
| OriginalFileName | Single line of text | Original or referenced business file name |
| FileLink | Hyperlink / text | Optional SharePoint file reference |
| BusinessDocumentType | Choice / text | Type of business document represented by intake |

---

## 5. Business Reference Columns

| Column Name | Type | Purpose |
|---|---|---|
| DistributorPO | Single line of text | Downstream distributor PO reference |
| BrandOwnerCMOPO | Single line of text | Brand owner PO to CMO |
| BrandOwnerAPIPO | Single line of text | Brand owner PO to API supplier |
| CMOASN | Single line of text | CMO ASN reference |
| CMOInvoice | Single line of text | CMO invoice reference |
| APIASN | Single line of text | API supplier ASN reference |
| ProductCode | Single line of text | Finished product code |
| MaterialCode | Single line of text | API / raw material code |
| BatchOrLotNumber | Single line of text | Finished goods or API lot reference |

---

## 6. Shipment and Quantity Columns

| Column Name | Type | Purpose |
|---|---|---|
| ShipmentType | Choice / text | Standard Shipment, Split Shipment, Not Applicable, Unknown |
| SplitShipmentSequence | Single line of text | Example: 1 of 2 |
| PlannedQuantity | Number | Planned order quantity |
| ShippedQuantity | Number | Shipped quantity |
| RemainingQuantity | Number | Remaining quantity for split shipment |
| QuantityUOM | Choice / text | Units, kg, Cartons, Pallets, Unknown |
| ShipToLocation | Single line of text | Synthetic destination |
| EstimatedRemainingShipDate | Date | Estimated remaining shipment date |

---

## 7. Serialization / EPCIS Columns

| Column Name | Type | Purpose |
|---|---|---|
| SerializedProduct | Yes/No | Indicates whether serialization is relevant |
| EPCISRequired | Yes/No | Indicates whether EPCIS readiness is expected |
| EPCISFileStatus | Choice / text | Available, Not Confirmed, Not Required, Missing, Unknown |
| EPCISFileName | Single line of text | Referenced EPCIS file name |
| AggregationStatus | Choice / text | Completed, Not Confirmed, Not Required, Unknown |
| DSCSAReviewRequired | Yes/No | Calculated review flag for serialization / EPCIS issues |

---

## 8. Control and Review Columns

| Column Name | Type | Purpose |
|---|---|---|
| PaymentHoldFlag | Yes/No | Calculated payment hold indicator |
| FulfilmentHoldFlag | Yes/No | Calculated fulfilment hold indicator |
| ReviewOwner | Choice / text | Procurement, Supply Chain, QA Serialization, Finance AP, Demo Owner |
| ReviewRequired | Yes/No | Calculated human review flag |
| ValidationResult | Choice / text | Pass, Warning, Fail, Not Checked |
| MissingFields | Multiple lines of text | Validation reason or missing-field summary |
| Status | Single line of text | Final status: Complete, Needs Review, Rejected, etc. |
| LastUpdatedDateTime | Date and time | Last update timestamp |
| ErrorReason | Multiple lines of text | Error or rejection reason |
| Notes | Multiple lines of text | Additional synthetic notes |
| RecordCount | Number | Always set to 1 for reporting/counting |

---

## 9. Current Status Values

The current demo uses these status values:

| Status | Meaning |
|---|---|
| New | Intake created but not processed |
| Processing | Flow or reviewer is processing the item |
| Complete | Required checks passed |
| Needs Review | Human review is required |
| Rejected | Input is unsupported or out of scope |
| Failed | System or workflow failure |

The runtime v3/v4/v5 flow primarily produces:

```text
Complete
Needs Review
Rejected
```

---

## 10. Current ValidationResult Values

| ValidationResult | Meaning |
|---|---|
| Pass | No exception detected |
| Warning | Human review required |
| Fail | Rejected or failed validation |
| Not Checked | Initial / seed state before runtime validation |

---

## 11. Current ReviewOwner Values

| ReviewOwner | Typical Trigger |
|---|---|
| QA Serialization | EPCIS readiness issue |
| Supply Chain | Split-shipment quantity / remaining shipment issue |
| Procurement | API material delay |
| Demo Owner | Unsupported or rejected demo intake |
| Not required / blank | Complete case |

---

## 12. RecordCount Logic

`RecordCount` is set to `1` for each tracker row.

Purpose:

- supports grouped SharePoint views
- supports future Power BI aggregation
- allows simple count measures
- avoids relying only on text-label counts

Example future measure:

```text
Total Intake Records = SUM(RecordCount)
```

---

## 13. Dashboard Views

Current views include:

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

These views support management visibility without requiring Power BI for the current demo.
