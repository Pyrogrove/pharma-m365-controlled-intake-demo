# Status Lifecycle

This file documents the status lifecycle used in Demo 1.

---

## 1. Status Values

| Status | Meaning |
|---|---|
| New | Intake item exists but has not been processed |
| Processing | Flow or reviewer is processing the item |
| Complete | Required validation passed and no review-blocking issue remains |
| Needs Review | Missing, ambiguous, or risky information requires human review |
| Rejected | Input is unsupported or out of scope |
| Failed | System or workflow failure occurred |

---

## 2. Current Runtime Statuses

The current v3/v4/v5 runtime flow primarily generates:

```text
Complete
Needs Review
Rejected
```

`New`, `Processing`, and `Failed` are retained as lifecycle concepts for future enhancement and documentation completeness.

---

## 3. Happy Path

```text
Email received
-> CSV attachment accepted
-> CSV saved to /Incoming
-> tracker item created
-> deterministic validation passes
-> Status = Complete
-> file moved to /Processed
-> final reply email sent
```

Example:

```text
CASE-01
```

---

## 4. Review Path

```text
Email received
-> CSV attachment accepted
-> CSV saved to /Incoming
-> tracker item created
-> deterministic validation detects review issue
-> Status = Needs Review
-> ReviewOwner assigned
-> hold flags calculated
-> file moved to /NeedsReview
-> final reply email sent
```

Examples:

```text
CASE-02
CASE-03
CASE-05
```

---

## 5. Rejection Path

```text
Email received
-> CSV attachment accepted
-> CSV saved to /Incoming
-> tracker item created
-> deterministic validation detects unsupported referenced file type
-> Status = Rejected
-> ReviewOwner = Demo Owner
-> ValidationResult = Fail
-> file moved to /Rejected
-> final reply email sent
```

Example:

```text
CASE-04
```

---

## 6. Unsupported Actual Attachment Path

```text
Email received
-> attachment format gate checks actual email attachment extension
-> unsupported actual attachment detected
-> reply email sent to sender
-> flow terminates successfully
-> no SharePoint file created
-> no tracker item created
```

This path prevents unsupported actual attachments from entering the controlled CSV intake workflow.

---

## 7. File Routing by Status

| Final Status | Destination Folder |
|---|---|
| Complete | `/Processed` |
| Needs Review | `/NeedsReview` |
| Rejected | `/Rejected` |

Default branch is intentionally empty.

---

## 8. ValidationResult Relationship

| Status | ValidationResult | Meaning |
|---|---|---|
| Complete | Pass | No exception detected |
| Needs Review | Warning | Business review required |
| Rejected | Fail | Unsupported or rejected input |
| Failed | Fail | System or flow error |

---

## 9. ReviewOwner Relationship

| Status | ReviewOwner |
|---|---|
| Complete | blank / Not required |
| Needs Review due to EPCIS issue | QA Serialization |
| Needs Review due to split shipment issue | Supply Chain |
| Needs Review due to API material delay | Procurement |
| Rejected | Demo Owner |

---

## 10. Hold Flag Relationship

| Scenario | PaymentHoldFlag | FulfilmentHoldFlag |
|---|---:|---:|
| Complete case | false | false |
| Missing EPCIS status | true | true |
| Split shipment incomplete | true | true |
| API material delay | false | true |
| Unsupported file type | false | false |

---

## 11. Human Review Boundary

`Needs Review` does not mean the workflow has failed.

It means the system has correctly detected that human review is required before operational decisions should proceed.

Human review is required for:

- QA / serialization readiness
- split-shipment quantity or schedule ambiguity
- API material delay impact
- unsupported or rejected intake follow-up

---

## 12. Future Lifecycle Enhancements

Possible future enhancements:

- add a dedicated `Processing` update before validation
- add a dedicated exception log list
- add human approval workflow
- add reviewer completion status
- add closed-loop resolution from `Needs Review` to `Complete`
- add SLA ageing and escalation
- add Power BI status ageing dashboard
