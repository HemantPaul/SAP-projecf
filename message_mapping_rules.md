# Message Mapping Rules — IDOC ↔ JSON

**Project:** SAP Integration Suite – P2P Integration Blueprint  
**Student:** HEMANT PAUL | Roll: 2305784 | SAP Integration Developer

---

## Overview

SAP ECC communicates using **IDOC (Intermediate Documents)** in XML format.
The Vendor Portal communicates using **JSON** via REST APIs.
SAP Integration Suite (CPI) performs the transformation in both directions.

---

## Mapping 1: ORDERS05 IDOC → Vendor PO JSON (iFlow 1)

### Header Level Mapping

| Source Field (IDOC) | Source Segment | Target Field (JSON) | Transformation |
|---------------------|---------------|---------------------|---------------|
| BELNR | E1EDK01 | purchaseOrder.poNumber | Direct copy |
| BSART | E1EDK01 | purchaseOrder.poType | Map: NB→STANDARD, ZNB→CUSTOM |
| CURCY | E1EDK01 | purchaseOrder.currency | Direct copy |
| ZTERM | E1EDK01 | purchaseOrder.paymentTerms | Map: NT30→NET30, NT60→NET60 |
| PARTN (LF) | E1EDKA1 | purchaseOrder.vendorId | Direct copy |
| NAMEF (LF) | E1EDKA1 | purchaseOrder.vendorName | Direct copy |
| DATUM (012) | E1EDK03 | purchaseOrder.orderDate | Format: YYYYMMDD → YYYY-MM-DD |
| DATUM (002) | E1EDK03 | purchaseOrder.requestedDeliveryDate | Format: YYYYMMDD → YYYY-MM-DD |
| STRAS (WE) | E1EDKA1 | deliveryAddress.street | Direct copy |
| ORT01 (WE) | E1EDKA1 | deliveryAddress.city | Direct copy |
| PSTLZ (WE) | E1EDKA1 | deliveryAddress.pincode | Direct copy |

### Line Item Level Mapping

| Source Field (IDOC) | Source Segment | Target Field (JSON) | Transformation |
|---------------------|---------------|---------------------|---------------|
| POSEX | E1EDP01 | items[n].lineItem | Integer conversion |
| MATNR | E1EDP01 | items[n].materialCode | Remove leading zeros |
| KTEXT | E1EDP19 | items[n].description | Direct copy |
| MENGE | E1EDP01 | items[n].quantity | Decimal → Integer |
| MENEE | E1EDP01 | items[n].unit | Direct copy |
| NETPR | E1EDP01 | items[n].unitPrice | Keep 2 decimal places |
| NETWR | E1EDP01 | items[n].totalValue | Direct copy |
| MWSBT | E1EDP01 | items[n].taxAmount | Direct copy |

### Calculated Fields

| Target Field | Formula | Notes |
|-------------|---------|-------|
| items[n].totalValue | MENGE × NETPR | Custom function in mapping |
| items[n].taxRate | (MWSBT / NETWR) × 100 | Calculate GST % |
| totals.netAmount | SUM of all items[n].totalValue | Aggregate function |
| totals.totalTax | SUM of all items[n].taxAmount | Aggregate function |
| totals.grossAmount | netAmount + totalTax | Custom function |
| orderDate | getCurrentDate() | CPI standard function |

---

## Mapping 2: Vendor Invoice JSON → BAPI_ACC_DOCUMENT_POST (iFlow 3)

| Source Field (JSON) | Target Field (BAPI) | Transformation |
|--------------------|---------------------|---------------|
| vendorInvoice.invoiceNumber | DOCUMENTHEADER.REF_DOC_NO | Direct copy |
| vendorInvoice.invoiceDate | DOCUMENTHEADER.DOC_DATE | Format: YYYY-MM-DD → YYYYMMDD |
| vendorInvoice.currency | DOCUMENTHEADER.CURR | Direct copy |
| vendorInvoice.vendorId | ACCOUNTPAYABLE.VENDOR_NO | Direct copy |
| vendorInvoice.poReference | ACCOUNTPAYABLE.PMNT_REF | Direct copy |
| items[n].netAmount | CURRENCYAMOUNT.AMT_DOCCUR | Per line item |
| items[n].totalTax | TAX.TAX_AMT | GST amount |
| totals.grossAmount | DOCUMENTHEADER.COMP_CODE | Company code TM01 |

### Fixed Values (Hardcoded in iFlow)
| BAPI Field | Value | Reason |
|-----------|-------|--------|
| DOCUMENTHEADER.COMP_CODE | TM01 | TechMart company code |
| DOCUMENTHEADER.DOC_TYPE | RE | Invoice Receipt document type |
| DOCUMENTHEADER.PSTNG_DATE | Today's date | Posting date = processing date |
| ACCOUNTPAYABLE.PMNT_METH | B | Bank transfer payment method |

---

## Mapping 3: REMADV IDOC → Payment Confirmation JSON (iFlow 4)

| Source Field (IDOC) | Source Segment | Target Field (JSON) | Transformation |
|---------------------|---------------|---------------------|---------------|
| DOCNUM | EDI_DC40 | integrationRef | Direct copy |
| REFNR | E1EDKAI | invoiceNumber | Direct copy |
| BELNR | E1EDKAI | poNumber | Direct copy |
| BETRW | E1EDKAI | amountPaid | Decimal, 2 places |
| WAERS | E1EDKAI | currency | Direct copy |
| ZFBDT | E1EDKAI | paymentDate | Format YYYYMMDD → YYYY-MM-DD |

### Fixed Values
| JSON Field | Value |
|-----------|-------|
| status | "PAYMENT_CLEARED" |
| clearedBy | "TM01-FI" |
| timestamp | getCurrentDateTime() ISO 8601 |

---

## Data Type Reference

| SAP IDOC Type | JSON Type | Conversion Note |
|--------------|-----------|----------------|
| CHAR | string | Direct |
| NUMC | integer | Remove leading zeros |
| DEC | number | Keep precision |
| DATS (YYYYMMDD) | string (YYYY-MM-DD) | Date format conversion |
| TIMS (HHMMSS) | string (HH:MM:SS) | Time format conversion |
| CURR | number | 2 decimal places |
