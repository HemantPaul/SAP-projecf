# iFlow 1: PO_Send_to_Vendor

**Project:** SAP Integration Suite – P2P Integration Blueprint  
**Student:** HEMANT PAUL | Roll: 2305784 | SAP Integration Developer

---

## Overview

| Field | Details |
|-------|---------|
| **iFlow Name** | PO_Send_to_Vendor |
| **Direction** | SAP ECC (MM) → SAP CPI → Vendor Portal |
| **Trigger** | IDOC ORDERS05 generated when PO is created in SAP MM |
| **Protocol (Sender)** | IDOC Adapter (SAP ECC → CPI) |
| **Protocol (Receiver)** | HTTP Adapter – REST POST (CPI → Vendor Portal) |
| **Message Format In** | XML (SAP IDOC ORDERS05) |
| **Message Format Out** | JSON (Vendor Portal REST API) |

---

## iFlow Steps (Process)

```
[SAP ECC MM]
     │
     │  IDOC ORDERS05 (XML)
     ▼
[IDOC Sender Channel]
     │
     ▼
[Content Modifier]
     │  Extract: PO Number, Vendor ID, Line Items
     ▼
[Validator]
     │  Schema validation on IDOC structure
     ▼
[Message Mapping]
     │  IDOC XML → Vendor JSON
     │  (See: mapping/message_mapping_rules.md)
     ▼
[Content Modifier]
     │  Set HTTP headers: Content-Type: application/json
     │  Set Authorization: Bearer {OAuth Token}
     ▼
[HTTP Receiver Channel]
     │  REST POST → https://vendor-portal.techmart.in/api/purchase-orders
     ▼
[Exception Subprocess]
     │  On HTTP 5xx: Retry 3× with 30s exponential backoff
     │  On HTTP 4xx: Dead Letter Queue + Alert email
     ▼
[End]
```

---

## Adapter Configuration

### Sender – IDOC Adapter (SAP ECC → CPI)
```
Adapter Type    : IDOC
Message Type    : ORDERS05
Partner Profile : TM_PORG_01
Port            : SAPCPI_OUT
Direction       : Outbound
```

### Receiver – HTTP Adapter (CPI → Vendor Portal)
```
Adapter Type    : HTTP
Method          : POST
URL             : https://vendor-portal.techmart.in/api/purchase-orders
Authentication  : OAuth 2.0 Client Credentials
Content-Type    : application/json
Timeout         : 30 seconds
Retry           : 3 attempts, exponential backoff (30s, 60s, 120s)
```

---

## Error Handling

| Error | Handling |
|-------|---------|
| HTTP 200 OK | Message completed, logged in Monitor |
| HTTP 4xx Client Error | Alert raised, message moved to Dead Letter Queue |
| HTTP 5xx Server Error | Retry 3× → if still failing, queue for 1-hour retry |
| IDOC Schema Invalid | Reject at validator, send error alert to admin |
| Timeout | Treated as 5xx, retry logic applies |

---

## Security
- OAuth 2.0 token fetched from Vendor Portal token endpoint before each call
- Token stored in SAP CPI Secure Parameter Store (never hardcoded)
- TLS 1.2+ enforced on all HTTP connections

---

## Idempotency
- Before processing, iFlow checks PO Number against a stored list in CPI Data Store
- If PO Number already processed → discard duplicate, log to audit trail
- Prevents duplicate POs being sent to vendor during system retries
