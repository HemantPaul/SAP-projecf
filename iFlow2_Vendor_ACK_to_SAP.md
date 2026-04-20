# iFlow 2: Vendor_ACK_to_SAP

**Project:** SAP Integration Suite – P2P Integration Blueprint  
**Student:** HEMANT PAUL | Roll: 2305784 | SAP Integration Developer

---

## Overview

| Field | Details |
|-------|---------|
| **iFlow Name** | Vendor_ACK_to_SAP |
| **Direction** | Vendor Portal → SAP CPI → SAP ECC (MM) |
| **Trigger** | Timer-based polling — every 5 minutes |
| **Protocol (Sender)** | HTTP Adapter – REST GET (poll Vendor Portal) |
| **Protocol (Receiver)** | RFC Adapter (CPI → SAP ECC MM) |
| **Message Format In** | JSON (Vendor Portal ACK response) |
| **Message Format Out** | RFC call to update PO confirmation status |

---

## iFlow Steps (Process)

```
[Timer Start Event]
     │  Polls every 5 minutes
     ▼
[HTTP Sender Channel]
     │  REST GET → https://vendor-portal.techmart.in/api/po-acknowledgements?status=PENDING
     ▼
[JSON to XML Converter]
     │  Convert JSON response → XML for CPI processing
     ▼
[Splitter]
     │  Split if multiple ACKs returned in one poll
     ▼
[Content Modifier]
     │  Extract: PO Number, ACK Status, Vendor Comments
     ▼
[Idempotency Check]
     │  Check CPI Data Store: Has this ACK been processed?
     │  YES → Skip | NO → Continue
     ▼
[RFC Receiver Channel]
     │  Call BAPI_PO_CHANGE to update PO confirmation in SAP MM
     │  Fields updated: Confirmation Status, Vendor Delivery Date
     ▼
[Data Store Write]
     │  Mark ACK as processed (store PO Number + timestamp)
     ▼
[End]
```

---

## Adapter Configuration

### Sender – HTTP Adapter (Poll Vendor Portal)
```
Adapter Type    : HTTP
Method          : GET
URL             : https://vendor-portal.techmart.in/api/po-acknowledgements
Query Params    : status=PENDING&buyerCode=TM01
Authentication  : OAuth 2.0 Client Credentials
Poll Interval   : 5 minutes (Timer Event)
```

### Receiver – RFC Adapter (CPI → SAP ECC)
```
Adapter Type    : RFC
BAPI            : BAPI_PO_CHANGE
SAP System      : TM_ECC_PROD
Client          : 100
Connection Pool : 5
Authentication  : Basic Auth (SAP technical user)
```

---

## Error Handling

| Error | Handling |
|-------|---------|
| No ACKs returned (empty list) | Log "No pending ACKs" — normal, next poll in 5 min |
| RFC BAPI error | Alert raised, ACK held in retry queue |
| Vendor Portal unreachable | Skip this poll cycle, alert if 3 consecutive failures |
| Duplicate ACK | Idempotency check catches it, silently discarded |

---

## Business Impact
- Before: Vendor ACKs were received by email, manually checked by purchase team
- After: SAP MM automatically shows "Vendor Confirmed" status within 5 minutes of vendor clicking ACK
