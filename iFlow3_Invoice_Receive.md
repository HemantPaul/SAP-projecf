# iFlow 3: Invoice_Receive

**Project:** SAP Integration Suite – P2P Integration Blueprint  
**Student:** HEMANT PAUL | Roll: 2305784 | SAP Integration Developer

---

## Overview

| Field | Details |
|-------|---------|
| **iFlow Name** | Invoice_Receive |
| **Direction** | Vendor Portal → SAP CPI → SAP ECC (FI) |
| **Trigger** | Vendor submits invoice via REST POST to CPI endpoint |
| **Protocol (Sender)** | HTTPS Adapter (Vendor Portal → CPI) |
| **Protocol (Receiver)** | RFC Adapter (CPI → SAP ECC FI via BAPI) |
| **Message Format In** | JSON (Vendor Invoice) |
| **Message Format Out** | BAPI_ACC_DOCUMENT_POST call (XML/RFC) |

---

## iFlow Steps (Process)

```
[HTTPS Sender Channel]
     │  Vendor Portal POSTs invoice JSON to CPI endpoint
     │  Authentication: OAuth 2.0 Bearer Token verified by CPI
     ▼
[JSON Validator]
     │  Validate required fields: invoiceNumber, poReference, items[], totalAmount
     │  If invalid → HTTP 400 response + alert
     ▼
[Idempotency Check]
     │  Check: Has invoice number already been processed?
     │  YES → Return HTTP 200 (duplicate acknowledged, not reprocessed)
     │  NO  → Continue
     ▼
[Content Modifier]
     │  Extract: PO Number, Invoice Number, Line Items, Tax, Total
     ▼
[PO Validation]
     │  Cross-check PO Number exists in SAP MM (RFC lookup)
     │  If PO not found → Dead Letter Queue + alert
     ▼
[Message Mapping]
     │  JSON Invoice → BAPI_ACC_DOCUMENT_POST XML structure
     │  Map: vendor, amount, tax, GL account, cost center
     ▼
[RFC Receiver Channel]
     │  Post FI document via BAPI_ACC_DOCUMENT_POST
     │  SAP FI: Creates Invoice Receipt (IR) document
     │  Triggers automatic Three-Way Match: PO + GR + IR
     ▼
[Three-Way Match Result]
     │  MATCH OK  → Invoice approved, payment scheduled
     │  MISMATCH  → Invoice parked for manual review, alert to Finance
     ▼
[Data Store Write]
     │  Mark invoice as processed
     ▼
[HTTP Response to Vendor]
     │  HTTP 200: {"status": "RECEIVED", "fiDocument": "5100001234"}
     ▼
[End]
```

---

## Adapter Configuration

### Sender – HTTPS Adapter (Vendor → CPI)
```
Adapter Type    : HTTPS
Method          : POST
Endpoint URL    : https://cpi-tenant.hana.ondemand.com/http/invoice-receive
Authentication  : OAuth 2.0 (CPI verifies vendor token)
Content-Type    : application/json
Max Message Size: 10 MB
```

### Receiver – RFC Adapter (CPI → SAP FI)
```
Adapter Type    : RFC
BAPI            : BAPI_ACC_DOCUMENT_POST
SAP System      : TM_ECC_PROD
Client          : 100
Company Code    : TM01
Document Type   : RE (Invoice Receipt)
```

---

## Three-Way Match Logic (SAP FI Automatic)

| Check | Field Compared | Tolerance |
|-------|---------------|-----------|
| Price Check | Invoice price vs PO price | ±2% allowed |
| Quantity Check | Invoice qty vs GR qty | Exact match required |
| Vendor Check | Invoice vendor vs PO vendor | Exact match required |

- **All checks pass** → FI document posted, payment scheduled per payment terms
- **Any check fails** → Invoice parked (status: BLOCKED), Finance team notified

---

## Error Handling

| Error | Handling |
|-------|---------|
| Invalid JSON | HTTP 400 returned to vendor, alert to admin |
| PO not found | Dead Letter Queue, alert to Purchase team |
| BAPI posting error | SAP error message returned, invoice held |
| Three-way mismatch | Invoice parked in SAP, Finance team alert |
| Duplicate invoice | Idempotency check — HTTP 200, not reprocessed |
