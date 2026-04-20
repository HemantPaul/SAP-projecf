# SAP Integration Suite - P2P Integration Project


Student Name   : HEMANT PAUL
Roll Number    : 2305784
Program        : SAP INTEGRATION DEVELOPER
College        : KIIT, Bhubaneswar
Submission Date: April 21, 2026


---


WHAT IS THIS PROJECT ABOUT?

In this project I tried to solve a common problem that happens in many companies.
When a company wants to buy something, they create a Purchase Order in SAP and
then someone manually emails it to the vendor. The vendor sends back an invoice
by email or WhatsApp. Then someone in finance manually types that invoice into SAP.

This whole process takes 5 to 7 days and many mistakes happen because of manual work.

So I designed a solution using SAP Integration Suite where all of this happens
automatically. SAP sends the Purchase Order to the vendor by itself. The vendor's
invoice comes into SAP by itself. No one has to do it manually.

I used a made-up company called TechMart Pvt. Ltd. to show how this would work
in a real company.


---


HOW THE SYSTEM WORKS

I designed 4 integration flows called iFlows in SAP Integration Suite.
Each iFlow does one job in the process.

iFlow 1 - PO Send to Vendor
When a Purchase Order is created in SAP MM, this iFlow automatically sends it
to the Vendor Portal as a JSON message. No manual email needed.

iFlow 2 - Vendor ACK to SAP
This iFlow checks the Vendor Portal every 5 minutes. When the vendor confirms
the order, it automatically updates the PO status in SAP MM.

iFlow 3 - Invoice Receive
When the vendor submits an invoice on the portal, this iFlow picks it up and
posts it directly into SAP FI. SAP then does a 3-way match automatically
to check if PO, Goods Receipt and Invoice all match.

iFlow 4 - Payment Confirm
After SAP processes the payment, this iFlow sends a payment confirmation
back to the vendor portal automatically.


---


THE FULL PROCESS STEP BY STEP

Step 1 - Someone in the company raises a Purchase Requisition in SAP MM
Step 2 - The PR becomes a Purchase Order and iFlow 1 sends it to the vendor
Step 3 - Vendor confirms the order and iFlow 2 brings that confirmation into SAP
Step 4 - Goods arrive and the warehouse team posts a Goods Receipt in SAP
Step 5 - Vendor submits invoice on portal and iFlow 3 posts it into SAP FI
Step 6 - SAP automatically checks if PO, GR and Invoice all match (3-way match)
Step 7 - Payment is done and iFlow 4 sends confirmation to vendor


---


FILES IN THIS REPOSITORY

HEMANT_PAUL_FINAL_SUBMIT.pdf
  This is my main project report. It contains everything about the project.

architecture/architecture_diagram.svg
  This is a diagram I made showing how SAP ECC, SAP Integration Suite
  and the Vendor Portal are all connected to each other.

iflow-designs/iFlow1_PO_Send_to_Vendor.md
  Detailed design of iFlow 1 showing each step, adapter settings and error handling.

iflow-designs/iFlow2_Vendor_ACK_to_SAP.md
  Detailed design of iFlow 2 showing how vendor confirmation polling works.

iflow-designs/iFlow3_Invoice_Receive.md
  Detailed design of iFlow 3 showing how vendor invoice gets posted in SAP FI.

iflow-designs/iFlow4_Payment_Confirm.md
  Detailed design of iFlow 4 showing how payment confirmation is sent to vendor.

message-samples/sample_PO_IDOC_ORDERS05.xml
  This is an example of the SAP IDOC message that iFlow 1 receives as input.
  SAP sends Purchase Orders in this XML format.

message-samples/sample_PO_Vendor_JSON.json
  This is an example of the JSON message that iFlow 1 sends to the Vendor Portal
  after converting from IDOC format.

message-samples/sample_Invoice_JSON.json
  This is an example of the invoice JSON that the vendor submits to the portal.
  iFlow 3 picks this up and posts it to SAP FI.

mapping/message_mapping_rules.md
  This file shows exactly which field in the SAP IDOC maps to which field
  in the JSON. For example BELNR in IDOC becomes poNumber in JSON.

testing/test_cases.md
  I wrote 25 test cases to check if all 4 iFlows work correctly including
  normal cases and error cases like duplicate invoices and portal downtime.


---


TECHNOLOGIES I USED

SAP BTP               - This is SAP's cloud platform where everything runs
SAP Integration Suite - This is the main tool I used to build the iFlows
SAP ECC 6.0           - The backend SAP system with MM and FI modules
Vendor Portal         - An external system that communicates using REST API and JSON
REST API              - The way SAP Integration Suite talks to the Vendor Portal
RFC and BAPI          - The way SAP Integration Suite talks to SAP ECC internally
IDOC ORDERS05         - The message format SAP uses to send Purchase Orders
JSON                  - The message format the Vendor Portal understands
OAuth 2.0             - Used to securely connect to the Vendor Portal
mTLS                  - Used to securely connect to SAP ECC
PGP Encryption        - Used to protect financial data in transit


---


SECURITY

I thought about security while designing this project. Here is what I included:

The connection to the Vendor Portal is secured using OAuth 2.0.
This means only our SAP system can connect to the vendor portal.

The connection to SAP ECC uses mTLS which means both sides check
each other's identity before sharing any data.

All financial messages are encrypted using PGP so even if someone
intercepts the message they cannot read it.

No passwords or API keys are written inside the iFlow code.
They are all stored safely in SAP CPI Secure Parameter Store.

Every message that passes through SAP Integration Suite is logged
in SAP CPI Message Monitor and kept for 90 days for audit purposes.


---


FICTITIOUS COMPANY DETAILS

Company Name     : TechMart Pvt. Ltd.
Industry         : Retail and Light Manufacturing
SAP Client       : 100
Company Code     : TM01
Plant            : TM_PLANT_01 located in Jamshedpur
Purchase Org     : TM_PORG_01
Vendor Name      : TechSupplies India Pvt. Ltd.
Vendor ID        : V1001


---


WHAT I WOULD ADD IN THE FUTURE

1. Connect SAP Ariba
   Right now the Purchase Order is created manually inside SAP MM.
   If we connect SAP Ariba, even the procurement planning part becomes automatic.

2. Use SAP Event Mesh instead of polling
   Currently iFlow 2 checks the vendor portal every 5 minutes.
   With SAP Event Mesh the vendor confirmation would arrive instantly
   the moment the vendor clicks confirm.

3. Add AI for reading invoices
   SAP has a service called Document Information Extraction or DOX.
   It can automatically read invoice PDFs using AI and extract the data.
   This would be helpful for vendors who send invoices as PDF attachments.

4. Build a vendor self-service portal
   Vendors currently have to call the company to check payment status.
   If we expose a simple API using SAP API Management, vendors can check
   their PO and payment status on their own anytime.

5. Add an analytics dashboard
   A dashboard in SAP Analytics Cloud showing how many POs are pending,
   how long invoices are taking to get approved, and payment aging reports
   would help the management team make better decisions.


---


IMPORTANT NOTE

All company names, vendor names, amounts and data used in this project
are completely made up. Nothing here represents any real company.
This project was made only for my college capstone submission as part of
the SAP Integration Developer course at KIIT University, Bhubaneswar.
