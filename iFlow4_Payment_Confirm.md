# iFlow 4: Payment_Confirm

**Project:** SAP Integration Suite – P2P Integration Blueprint  
**Student:** HEMANT PAUL | Roll: 2305784 | SAP Integration Developer

---

## Overview

| Field | Details |
|-------|---------|
| **iFlow Name** | Payment_Confirm |
| **Direction** | SAP ECC (FI) → SAP CPI → Vendor Portal |
| **Trigger** | Payment clearing event in SAP FI (F-53 / F110 transaction) |
| **Protocol (Sender)** | IDOC Adapter (SAP FI sends REMADV IDOC on payment) |
| **Protocol (Receiver)** | HTTP Adapter – REST PATCH (CPI → Vendor Portal) |
| **Message Format In** | IDOC REMADV (Remittance Advice) |
| **Message Format Out** | JSON PATCH request to Vendor Portal |

---

## iFlow Steps (Process)

```
[SAP FI — Payment Run F110]
     │  Payment posted, REMADV IDOC triggered
     ▼
[IDOC Sender Channel]
     │  IDOC REMADV received by CPI
     ▼
[Content Modifier]
     │  Extract: Invoice Number, PO Number, Amount Paid,
     │           Payment Date, Bank Reference, Vendor ID
     ▼
[Message Mapping]
     │  IDOC REMADV → JSON payment confirmation payload
     ▼
[Content Modifier]
     │  Set HTTP Method: PATCH
     │  Set URL: /api/invoices/{invoiceNumber}/status
     │  Set Authorization: Bearer Token
     ▼
[HTTP Receiver Channel]
     │  REST PATCH → Vendor Portal
     │  Body: { "status": "PAYMENT_CLEARED", "paymentDate": "...", "amount": ... }
     ▼
[Response Handler]
     │  HTTP 200 → Log success in Message Monitor
     │  HTTP 4xx/5xx → Retry 3× → Alert if still failing
     ▼
[End]
```

---

## Adapter Configuration

### Sender – IDOC Adapter (SAP FI → CPI)
```
Adapter Type    : IDOC
IDOC Type       : REMADV
Message Type    : REMADV
Direction       : Outbound from SAP FI
Triggered by    : F110 (Automatic Payment Program) or F-53 (Manual Payment)
```

### Receiver – HTTP Adapter (CPI → Vendor Portal)
```
Adapter Type    : HTTP
Method          : PATCH
URL             : https://vendor-portal.techmart.in/api/invoices/{invoiceNumber}/status
Authentication  : OAuth 2.0 Client Credentials
Content-Type    : application/json
Retry           : 3× with 30s backoff
```

---

## Sample Output JSON (Payment Confirmation)

```json
{
  "invoiceNumber": "INV-V1001-2026-0042",
  "poNumber": "4500001234",
  "vendorId": "V1001",
  "status": "PAYMENT_CLEARED",
  "paymentDate": "2026-05-28",
  "amountPaid": 541030.00,
  "currency": "INR",
  "bankReference": "NEFT-HDFC-20260528-001",
  "clearedBy": "TM01-FI",
  "timestamp": "2026-05-28T14:30:00Z"
}
```

---

## Business Impact
- Vendor immediately knows payment has been made — no need to call TechMart finance
- Reduces vendor payment enquiry calls by ~90%
- Full audit trail: payment reference stored in both SAP FI and Vendor Portal
