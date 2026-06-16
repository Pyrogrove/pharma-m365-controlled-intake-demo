# Pharma M365 Controlled Intake Demo

Work-in-progress portfolio demo showing a controlled document intake workflow for a pharma supply-chain scenario using Microsoft 365.

## Scenario

This demo simulates a US pharmaceutical brand owner coordinating with:

- a Taiwan contract manufacturer (CMO)
- a China API / raw material supplier
- a US 3PL warehouse
- a US distributor

The operational pain is not just receiving documents. The real pain is that CMO split shipments, batch/lot references, ASN updates, invoice timing, 3PL receipt status, and DSCSA/EPCIS readiness are interrelated. If key documents or metadata are missing, the brand owner cannot confidently release downstream fulfilment, approve CMO payment, or maintain audit-ready traceability.

## Demo Objective

The demo uses Microsoft 365 to show a controlled intake pattern:

```text
Incoming CMO / supplier document
-> SharePoint document library
-> SharePoint intake tracker
-> metadata validation
-> exception status
-> human review queue
-> management visibility
