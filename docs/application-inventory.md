# Bank of Z — Application Inventory

> **Generated from**: Local scanner database (43 files scanned)  
> **Scope**: CICS COBOL programs, IMS COBOL programs, IMS PL/I programs, Batch PL/I program  
> **Architecture**: Transactions route to CICS or IMS based on customer-ID prefix. API layer is z/OS Connect. Frontend is a separate Liberty instance.

---

## Table of Contents

1. [Application Summary](#1-application-summary)
2. [CICS COBOL Programs](#2-cics-cobol-programs)
   - [Screen / Presentation Layer](#21-screen--presentation-layer)
   - [Business Logic / Service Layer](#22-business-logic--service-layer)
   - [Infrastructure / Utility](#23-infrastructure--utility)
   - [Data Initialisation (Batch-like CICS)](#24-data-initialisation-batch-like-cics)
3. [IMS Programs](#3-ims-programs)
   - [IMS Message Processing Programs (MPPs)](#31-ims-message-processing-programs-mpps)
   - [IMS Batch / Load Programs](#32-ims-batch--load-programs)
4. [Batch PL/I Program](#4-batch-pli-program)
5. [Shared Resources Summary](#5-shared-resources-summary)
   - [DB2 Tables](#51-db2-tables)
   - [IMS Databases (DBDs)](#52-ims-databases-dbds)
   - [CICS BMS Maps](#53-cics-bms-maps)
   - [Copybooks](#54-copybooks)
   - [CICS Files (VSAM)](#55-cics-files-vsam)

---

## 1. Application Summary

| Category | Count |
|---|---|
| CICS COBOL programs (source) | 27 |
| IMS COBOL programs | 10 |
| IMS PL/I programs | 1 |
| Batch PL/I programs | 1 |
| **Total programs** | **39** |
| DB2 tables accessed | 5 (ACCOUNT, CUSTOMER, CONTROL, PROCTRAN, STTESTER.CONTROL) |
| IMS databases (DBDs) | 9 |
| BMS maps | 9 |
| Copybooks (CICS) | 40+ |
| CICS transactions | 9 (OMEN, OCAC, OCCA, OCCS, OCRA, ODAC, ODCS, OTFN, OUAC) |
| External libraries called | CEE runtime (CEEDAYS, CEELOCT, CEEGMT, CEEDATM) |

---

## 2. CICS COBOL Programs

### 2.1 Screen / Presentation Layer

These programs implement the pseudo-conversational CICS screen flow. Each uses `EXEC CICS SEND MAP` / `RECEIVE MAP` against a BMS mapset, calls a back-end service program via `EXEC CICS LINK`, and returns control to a CICS transaction code.

---

#### `BNKMENU` — Bank Main Menu
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNKMENU.cbl`](src/base/cics/cobol/BNKMENU.cbl) |
| Type | CICS COBOL |
| Role | Entry-point menu screen; dispatches to all other CICS transactions |
| Paragraphs | 18 |
| CICS transactions | OMEN (own), OCAC, OCCA, OCCS, OCRA, ODAC, ODCS, OTFN, OUAC (targets) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1MAI | SEND/RECEIVE |
| Copybook | BNK1MAI | BMS symbolic map |
| Copybook | ABNDINFO | Abend information structure |
| Copybook | DFHAID | CICS attention identifier keys |
| Called program | ABNDPROC | Error/abend handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1CAC` — Create Account Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1CAC.cbl`](src/base/cics/cobol/BNK1CAC.cbl) |
| Type | CICS COBOL |
| Role | Presentation layer for account creation; collects input then LINKs to CREACC |
| Paragraphs | 18 |
| CICS transactions | OCAC (own), OMEN (return to menu) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1CAM | SEND/RECEIVE/CONTROL |
| Copybook | BNK1CAM | BMS symbolic map |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Called program | CREACC | Create account (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1CCA` — Customer/Account Enquiry Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1CCA.cbl`](src/base/cics/cobol/BNK1CCA.cbl) |
| Type | CICS COBOL |
| Role | Displays accounts for a given customer; LINKs to INQACCCU |
| Paragraphs | 18 |
| CICS transactions | OCCA (own), OMEN (return) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1ACC | SEND/RECEIVE/CONTROL |
| Copybook | BNK1ACC | BMS symbolic map |
| Copybook | INQACCCU | Inquiry interface copybook |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Called program | INQACCCU | Inquire accounts by customer (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1CCS` — Create Customer Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1CCS.cbl`](src/base/cics/cobol/BNK1CCS.cbl) |
| Type | CICS COBOL |
| Role | Presentation layer for new customer registration; LINKs to CRECUST |
| Paragraphs | 24 |
| CICS transactions | OCCS (own), OMEN (return) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1CCM | SEND/RECEIVE |
| Copybook | BNK1CCM | BMS symbolic map |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Called program | CRECUST | Create customer (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1CRA` — Debit/Credit Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1CRA.cbl`](src/base/cics/cobol/BNK1CRA.cbl) |
| Type | CICS COBOL |
| Role | Presentation layer for debit/credit transactions; LINKs to DBCRFUN |
| Paragraphs | 20 |
| CICS transactions | OCRA (own), OMEN (return) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1CDM | SEND/RECEIVE/CONTROL |
| Copybook | BNK1CDM | BMS symbolic map |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Called program | DBCRFUN | Debit/credit function (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1DAC` — Delete Account Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1DAC.cbl`](src/base/cics/cobol/BNK1DAC.cbl) |
| Type | CICS COBOL |
| Role | Presents account for deletion; confirms then LINKs to DELACC |
| Paragraphs | 22 |
| CICS transactions | ODAC (own), OMEN (return) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1DAM | SEND/RECEIVE/CONTROL |
| Copybook | BNK1DAM | BMS symbolic map |
| Copybook | INQACC | Inquiry interface copybook |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Called program | INQACC | Inquire account (via CICS LINK) |
| Called program | DELACC | Delete account (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1DCS` — Delete Customer Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1DCS.cbl`](src/base/cics/cobol/BNK1DCS.cbl) |
| Type | CICS COBOL |
| Role | Displays customer for deletion; orchestrates inquiry, update, and delete sub-calls |
| Paragraphs | 36 |
| CICS transactions | ODCS (own), OMEN (return) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1DCM | SEND/RECEIVE/CONTROL |
| Copybook | BNK1DCM | BMS symbolic map |
| Copybook | INQCUSTZ | Customer inquiry interface |
| Copybook | DELCUS | Delete customer interface |
| Copybook | UPDCUST | Update customer interface |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Copybook | DFHBMSCA | BMS colour/highlight attributes |
| Called program | INQCUST | Inquire customer (via CICS LINK) |
| Called program | DELCUS | Delete customer (via CICS LINK) |
| Called program | UPDCUST | Update customer (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1TFN` — Funds Transfer Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1TFN.cbl`](src/base/cics/cobol/BNK1TFN.cbl) |
| Type | CICS COBOL |
| Role | Presentation layer for inter-account transfer; LINKs to XFRFUN |
| Paragraphs | 20 |
| CICS transactions | OTFN (own), OMEN (return) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1TFM | SEND/RECEIVE/CONTROL |
| Copybook | BNK1TFM | BMS symbolic map |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Called program | XFRFUN | Transfer function (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `BNK1UAC` — Update Account Screen
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BNK1UAC.cbl`](src/base/cics/cobol/BNK1UAC.cbl) |
| Type | CICS COBOL |
| Role | Displays account for update; LINKs to INQACC then UPDACC |
| Paragraphs | 22 |
| CICS transactions | OUAC (own), OMEN (return) |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| BMS Map | BNK1UAM | SEND/RECEIVE/CONTROL |
| Copybook | BNK1UAM | BMS symbolic map |
| Copybook | ABNDINFO | Abend information |
| Copybook | DFHAID | AID keys |
| Called program | INQACC | Inquire account (via CICS LINK) |
| Called program | UPDACC | Update account (via CICS LINK) |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

### 2.2 Business Logic / Service Layer

These programs perform the actual data access (DB2 SQL) and are invoked via `EXEC CICS LINK` from either the screen programs or the z/OS Connect API layer.

---

#### `CREACC` — Create Account
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/CREACC.cbl`](src/base/cics/cobol/CREACC.cbl) |
| Type | CICS COBOL |
| Role | Creates a new bank account; assigns account number from CONTROL table |
| Paragraphs | 24 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | WRITE (INSERT) |
| DB2 Table | CONTROL | READ/WRITE (SELECT + UPDATE) |
| DB2 Table | PROCTRAN | WRITE (INSERT — audit trail) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCOUNT | Account data structure |
| Copybook | ACCTCTRL | Account control structure |
| Copybook | CREACC | Create account interface/commarea |
| Copybook | CUSTOMER | Customer data structure |
| Copybook | INQACCCU | Inquiry-accounts-by-customer interface |
| Copybook | INQCUSTZ | Customer inquiry interface |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | PROCTRAN | Processed-transaction data structure |
| Copybook | SORTCODE | Bank sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | INQACCCU | Validate customer/account existence |
| Called program | INQCUST (alias) | Validate customer record |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `CRECUST` — Create Customer
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/CRECUST.cbl`](src/base/cics/cobol/CRECUST.cbl) |
| Type | CICS COBOL |
| Role | Creates a new customer record; assigns customer number from CONTROL table; uses CEE date services |
| Paragraphs | 25 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | CUSTOMER | WRITE (INSERT) |
| DB2 Table | PROCTRAN | WRITE (INSERT — audit trail) |
| DB2 Table | STTESTER.CONTROL | READ/WRITE (SELECT + UPDATE — customer number sequence) |
| Copybook | CRECUST | Create customer interface/commarea |
| Copybook | CONTDB2 | CONTROL DB2 host variable layout |
| Copybook | CUSTCTRL | Customer control structure |
| Copybook | CUSTDB2 | CUSTOMER DB2 host variable layout |
| Copybook | CUSTOMER | Customer data structure |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | PROCTRAN | Processed-transaction data structure |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | CEEIGZCT | CEE condition token layout |
| Copybook | ABNDINFO | Abend information |
| Called program | CEEDAYS | CEE — convert date to Lilian format |
| Called program | CEELOCT | CEE — get current local time |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `DBCRFUN` — Debit/Credit Function
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/DBCRFUN.cbl`](src/base/cics/cobol/DBCRFUN.cbl) |
| Type | CICS COBOL |
| Role | Applies a debit or credit to an account balance; logs transaction to PROCTRAN |
| Paragraphs | 16 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ/WRITE (SELECT + UPDATE) |
| DB2 Table | PROCTRAN | WRITE (INSERT) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCOUNT | Account data structure |
| Copybook | PAYDBCR | Debit/credit payment interface |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | PROCTRAN | Processed-transaction data structure |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `DELACC` — Delete Account
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/DELACC.cbl`](src/base/cics/cobol/DELACC.cbl) |
| Type | CICS COBOL |
| Role | Deletes an account record from DB2 after validating existence; logs audit entry |
| Paragraphs | 14 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ/WRITE (SELECT + DELETE) |
| DB2 Table | PROCTRAN | WRITE (INSERT) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCOUNT | Account data structure |
| Copybook | ACCTCTRL | Account control structure |
| Copybook | DELACC | Delete account interface/commarea |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | PROCTRAN | Processed-transaction data structure |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `DELCUS` — Delete Customer
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/DELCUS.cbl`](src/base/cics/cobol/DELCUS.cbl) |
| Type | CICS COBOL |
| Role | Deletes a customer and all associated accounts; cascades deletion via DELACC |
| Paragraphs | 16 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | CUSTOMER | READ/WRITE (SELECT + DELETE) |
| DB2 Table | PROCTRAN | WRITE (INSERT) |
| Copybook | ACCOUNT | Account data structure |
| Copybook | CUSTDB2 | CUSTOMER DB2 host variable layout |
| Copybook | CUSTOMER | Customer data structure |
| Copybook | DELCUS | Delete customer interface/commarea |
| Copybook | INQACCCU | Inquiry-accounts-by-customer interface |
| Copybook | INQCUSTZ | Customer inquiry interface |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | PROCTRAN | Processed-transaction data structure |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | INQCUST | Verify customer exists |
| Called program | INQACCCU | Retrieve all accounts for customer |
| Called program | DELACC (alias) | Delete each account |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `INQACC` — Inquire Account
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/INQACC.cbl`](src/base/cics/cobol/INQACC.cbl) |
| Type | CICS COBOL |
| Role | Retrieves a single account record by account number using a DB2 cursor |
| Paragraphs | 18 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ (SELECT via cursor + FETCH) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCOUNT | Account data structure |
| Copybook | INQACC | Inquiry account interface/commarea |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `INQACCCU` — Inquire Accounts by Customer
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/INQACCCU.cbl`](src/base/cics/cobol/INQACCCU.cbl) |
| Type | CICS COBOL |
| Role | Retrieves all accounts belonging to a given customer via a DB2 cursor; calls INQCUST for customer validation |
| Paragraphs | 16 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ (cursor/FETCH) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCOUNT | Account data structure |
| Copybook | CUSTOMER | Customer data structure |
| Copybook | INQACCCU | Interface/commarea |
| Copybook | INQCUSTZ | Customer inquiry interface |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | INQCUST (alias) | Customer validation |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `INQACCS` — Inquire Accounts (Short)
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/INQACCS.cbl`](src/base/cics/cobol/INQACCS.cbl) |
| Type | CICS COBOL |
| Role | Retrieves a condensed account list (account number + balance only) via DB2 cursor |
| Paragraphs | 14 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ (cursor/FETCH) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | INQACCS | Short account inquiry interface |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `INQCUST` — Inquire Customer
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/INQCUST.cbl`](src/base/cics/cobol/INQCUST.cbl) |
| Type | CICS COBOL |
| Role | Retrieves a customer record by customer number via DB2 SELECT |
| Paragraphs | 20 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | CUSTOMER | READ (SELECT ×2) |
| Copybook | CUSTDB2 | CUSTOMER DB2 host variable layout |
| Copybook | CUSTOMER | Customer data structure |
| Copybook | INQCUSTZ | Customer inquiry interface/commarea |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `INQTRAND` — Inquire Transaction Detail
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/INQTRAND.cbl`](src/base/cics/cobol/INQTRAND.cbl) |
| Type | CICS COBOL |
| Role | Retrieves transaction detail records from PROCTRAN via DB2 cursor |
| Paragraphs | 16 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | PROCTRAN | READ (cursor/FETCH) |
| Copybook | INQTRAND | Transaction detail inquiry interface |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `INQTRANL` — Inquire Transaction List
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/INQTRANL.cbl`](src/base/cics/cobol/INQTRANL.cbl) |
| Type | CICS COBOL |
| Role | Retrieves a list of transactions for an account from PROCTRAN using multiple DB2 cursors |
| Paragraphs | 20 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | PROCTRAN | READ (2 cursors/FETCH) |
| Copybook | INQTRANL | Transaction list inquiry interface |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `UPDACC` — Update Account
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/UPDACC.cbl`](src/base/cics/cobol/UPDACC.cbl) |
| Type | CICS COBOL |
| Role | Updates account fields (e.g. overdraft limit, interest rate) via DB2 SELECT then UPDATE |
| Paragraphs | 8 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ/WRITE (SELECT + UPDATE) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCOUNT | Account data structure |
| Copybook | UPDACC | Update account interface/commarea |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `UPDCUST` — Update Customer
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/UPDCUST.cbl`](src/base/cics/cobol/UPDCUST.cbl) |
| Type | CICS COBOL |
| Role | Updates customer demographic fields via DB2 SELECT then UPDATE |
| Paragraphs | 8 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | CUSTOMER | READ/WRITE (SELECT + UPDATE) |
| Copybook | CUSTDB2 | CUSTOMER DB2 host variable layout |
| Copybook | CUSTOMER | Customer data structure |
| Copybook | UPDCUST | Update customer interface/commarea |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `XFRFUN` — Transfer Function
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/XFRFUN.cbl`](src/base/cics/cobol/XFRFUN.cbl) |
| Type | CICS COBOL |
| Role | Transfers funds between two accounts: debit source + credit target + log PROCTRAN ×2 |
| Paragraphs | 20 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ/WRITE (SELECT ×2 + UPDATE ×2) |
| DB2 Table | PROCTRAN | WRITE (INSERT ×2) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCOUNT | Account data structure (×2 included) |
| Copybook | PROCDB2 | PROCTRAN DB2 host variable layout |
| Copybook | PROCTRAN | Processed-transaction data structure |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Copybook | SQLDA | SQL descriptor area |
| Copybook | XFRFUN | Transfer function interface/commarea |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

#### `CRDTAGY1` – `CRDTAGY5` — Credit Rating Agencies 1–5
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/CRDTAGY1.cbl`](src/base/cics/cobol/CRDTAGY1.cbl) … [`CRDTAGY5.cbl`](src/base/cics/cobol/CRDTAGY5.cbl) |
| Type | CICS COBOL |
| Role | Five simulated external credit-rating agency programs; each returns a credit score for a customer. Identical structure — differ only in the score range or delay they simulate |
| Paragraphs | 6 each |

**Dependencies (all five share the same)**

| Resource | Name | Access |
|---|---|---|
| Copybook | CUSTOMER | Customer data structure (input commarea) |
| Copybook | SORTCODE | Sort code constant |
| Copybook | ABNDINFO | Abend information |
| Called program | ABNDPROC | Error handling |
| Called program | ABND-PROGRAM | Abend signaller |

---

### 2.3 Infrastructure / Utility

#### `ABNDPROC` — Abend Processor
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/ABNDPROC.cbl`](src/base/cics/cobol/ABNDPROC.cbl) |
| Type | CICS COBOL |
| Role | Called by all application programs on error; formats abend information and writes to CICS VSAM abend file `ABNDFILE` |
| Paragraphs | 4 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| CICS File | ABNDFILE | WRITE (CICS WRITE) |
| Copybook | ABNDINFO | Abend information structure |

---

#### `GETCOMPY` — Get Company
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/GETCOMPY.cbl`](src/base/cics/cobol/GETCOMPY.cbl) |
| Type | CICS COBOL |
| Role | Returns the bank company name constant |
| Paragraphs | 1 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| Copybook | GETCOMPY | Company name interface |

---

#### `GETSCODE` — Get Sort Code
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/GETSCODE.cbl`](src/base/cics/cobol/GETSCODE.cbl) |
| Type | CICS COBOL |
| Role | Returns the bank sort code constant |
| Paragraphs | 1 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| Copybook | GETSCODE | Sort code interface |
| Copybook | SORTCODE | Sort code constant |

---

### 2.4 Data Initialisation (Batch-like CICS)

#### `BANKDATA` — Bank Data Initialiser
| Attribute | Value |
|---|---|
| Source | [`src/base/cics/cobol/BANKDATA.cbl`](src/base/cics/cobol/BANKDATA.cbl) |
| Type | CICS COBOL |
| Role | Seeds the DB2 tables with initial test data (accounts, customers, CONTROL seed rows); used for environment setup only — not in production flow |
| Paragraphs | 22 |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | WRITE (INSERT + DELETE) |
| DB2 Table | CUSTOMER | WRITE (INSERT + DELETE) |
| DB2 Table | STTESTER.CONTROL | WRITE (INSERT + UPDATE + DELETE — multiple rows) |
| Copybook | ACCDB2 | ACCOUNT DB2 host variable layout |
| Copybook | ACCTCTRL | Account control structure |
| Copybook | CONTDB2 | CONTROL DB2 host variable layout |
| Copybook | CUSTCTRL | Customer control structure |
| Copybook | CUSTDB2 | CUSTOMER DB2 host variable layout |
| Copybook | CUSTOMER | Customer data structure |
| Copybook | SORTCODE | Sort code constant |
| Copybook | SQLCA | SQL communication area |
| Called program | CEEDATM | CEE — convert Lilian date to character |
| Called program | CEEGMT | CEE — get current GMT time |

---

## 3. IMS Programs

The IMS subsystem serves customers whose IDs carry the IMS prefix. Programs use DL/I calls (`CALL 'CBLTDLI'` / `CALL 'PLITDLI'`) against IMS databases defined by DBDs. Each program has a matching PSB.

### 3.1 IMS Message Processing Programs (MPPs)

#### `IBLOGIN1` / `IBLOGIN` — Login
| Attribute | Value |
|---|---|
| Source (COBOL) | [`src/base/ims/cobol/IBLOGIN1.cbl`](src/base/ims/cobol/IBLOGIN1.cbl) |
| Source (PL/I) | [`src/base/ims/pli/IBLOGIN.pli`](src/base/ims/pli/IBLOGIN.pli) |
| Type | IMS MPP (COBOL + PL/I variants) |
| Role | Authenticates a customer; validates password, checks LOGGEDIN flag, updates CUSTOMER segment to mark login |
| PSB | IBLOGIN (PL/I) / not separate PSB for IBLOGIN1 — uses IBLOGIN PSB |
| DL/I calls | GU (IOPCB), GHU (CUSTOMER DBD), REPL, ISRT (IOPCB reply) |

**Dependencies**

| Resource | Name | Access / PROCOPT |
|---|---|---|
| IMS DB | CUSTOMER | READ (GHU — get-hold-unique), REPL (update login flag) |
| IMS PCB (I/O) | IOPCB | GU (get input message), ISRT (send reply) |

---

#### `IBLOGOUT` — Logout
| Attribute | Value |
|---|---|
| Source | [`src/base/ims/cobol/IBLOGOUT.cbl`](src/base/ims/cobol/IBLOGOUT.cbl) |
| Type | IMS MPP (COBOL) |
| PSB | IBLOGOUT |
| Role | Marks a customer as logged out by updating the CUSTOMER segment LOGGEDIN flag |
| DL/I calls | GU (IOPCB), GHU (CUSTOMER), REPL, ISRT (reply) |

**Dependencies**

| Resource | Name | Access / PROCOPT |
|---|---|---|
| IMS DB | CUSTOMER | READ (GHU) + REPL update; PROCOPT=R |
| IMS PCB (I/O) | IOPCB | GU + ISRT |

---

#### `IBGCUDAT` — Get Customer Data
| Attribute | Value |
|---|---|
| Source | [`src/base/ims/cobol/IBGCUDAT.cbl`](src/base/ims/cobol/IBGCUDAT.cbl) |
| Type | IMS MPP (COBOL) |
| PSB | IBGCUDAT |
| Role | Retrieves customer segment data for a given customer ID |
| DL/I calls | GU (IOPCB), GU (CUSTOMER DB), ISRT (reply) |

**Dependencies**

| Resource | Name | Access / PROCOPT |
|---|---|---|
| IMS DB | CUSTOMER | READ (GU); PROCOPT=G |
| IMS PCB (I/O) | IOPCB | GU + ISRT |

---

#### `IBSCUDAT` — Set Customer Data
| Attribute | Value |
|---|---|
| Source | [`src/base/ims/cobol/IBSCUDAT.cbl`](src/base/ims/cobol/IBSCUDAT.cbl) |
| Type | IMS MPP (COBOL) |
| PSB | IBSCUDAT |
| Role | Updates customer segment fields (e.g. address, phone) |
| DL/I calls | GU (IOPCB), GHU (CUSTOMER), REPL, ISRT (reply) |

**Dependencies**

| Resource | Name | Access / PROCOPT |
|---|---|---|
| IMS DB | CUSTOMER | READ+UPDATE (GHU + REPL); PROCOPT=R |
| IMS PCB (I/O) | IOPCB | GU + ISRT |

---

#### `IBACSUM` — Account Summary
| Attribute | Value |
|---|---|
| Source | [`src/base/ims/cobol/IBACSUM.cbl`](src/base/ims/cobol/IBACSUM.cbl) |
| Type | IMS MPP (COBOL) |
| PSB | IBACSUM |
| Role | Returns a summary of all accounts for a customer by traversing CUSTACCS database |
| DL/I calls | GU (IOPCB), GU+GN (ACCOUNT / CUSTACCS), ISRT (reply) |

**Dependencies**

| Resource | Name | Access / PROCOPT |
|---|---|---|
| IMS DB | ACCOUNT | READ (GU+GN); PROCOPT=G |
| IMS DB | CUSTACCS | READ (GN); PROCOPT=G |
| IMS PCB (I/O) | IOPCB | GU + ISRT |

---

#### `IBTRAN` — IMS Transaction Processor
| Attribute | Value |
|---|---|
| Source | [`src/base/ims/cobol/IBTRAN.cbl`](src/base/ims/cobol/IBTRAN.cbl) |
| Type | IMS MPP (COBOL, recursive) |
| PSB | IBTRAN |
| Role | Core IMS transaction processor; handles debit/credit/transfer operations against ACCOUNT database and inserts a HISTORY record; invokes Java via JNI for history persistence |
| DL/I calls | GU (IOPCB), GU+GN (ACCOUNT/CUSTACCS), ISRT (HISTORY), ISRT (reply) |

**Dependencies**

| Resource | Name | Access / PROCOPT |
|---|---|---|
| IMS DB | ACCOUNT | READ (GU+GN); PROCOPT=R |
| IMS DB | CUSTACCS | READ (GN); PROCOPT=G |
| IMS DB | HISTORY | WRITE (ISRT); PROCOPT=I |
| IMS PCB (I/O) | IOPCB | GU + ISRT |
| Copybook | JNI | Java Native Interface binding |

---

### 3.2 IMS Batch / Load Programs

These programs use the IBLOAD PSB (PROCOPT=L — LOAD) to initialise all IMS databases. They are run once for environment setup.

| Program | Source | Role | IMS DBs Loaded |
|---|---|---|---|
| `LOADACCT` | [`src/base/ims/cobol/LOADACCT.cbl`](src/base/ims/cobol/LOADACCT.cbl) | Load ACCOUNT segments | ACCOUNT |
| `LOADCUST` | [`src/base/ims/cobol/LOADCUST.cbl`](src/base/ims/cobol/LOADCUST.cbl) | Load CUSTOMER segments | CUSTOMER |
| `LOADCUSA` | [`src/base/ims/cobol/LOADCUSA.cbl`](src/base/ims/cobol/LOADCUSA.cbl) | Load CUSTACCS (customer-account cross-ref) | CUSTACCS |
| `LOADHIST` | [`src/base/ims/cobol/LOADHIST.cbl`](src/base/ims/cobol/LOADHIST.cbl) | Load HISTORY transaction segments | HISTORY; reads ACCOUNT + CUSTOMER for keys |
| `LOADTSTA` | [`src/base/ims/cobol/LOADTSTA.cbl`](src/base/ims/cobol/LOADTSTA.cbl) | Load TSTAT/TSTATTYP/TTYPE transaction status data | TSTAT, TSTATTYP, TTYPE |

All load programs share PSB **IBLOAD** (PROCOPT=L on all DBDs).

---

## 4. Batch PL/I Program

#### `BNKSTMT` — Bank Monthly Statement Generator
| Attribute | Value |
|---|---|
| Source | [`src/base/batch/pli/BNKSTMT.pli`](src/base/batch/pli/BNKSTMT.pli) |
| JCL | [`src/base/batch/jcl/BNKSTMT.jcl`](src/base/batch/jcl/BNKSTMT.jcl) |
| Type | Batch PL/I |
| Role | Generates monthly account statements for all bank customers; reads customer, account and transaction history from DB2; writes formatted report to SYSPRINT |

**Dependencies**

| Resource | Name | Access |
|---|---|---|
| DB2 Table | ACCOUNT | READ (SELECT) |
| DB2 Table | CUSTOMER | READ (SELECT) |
| DB2 Table | PROCTRAN | READ (cursor/FETCH) |
| DB2 include | SQLCA | SQL communication area |
| Input | JCL parameters | Account number range + statement period |
| Output | SYSPRINT | Formatted monthly statement report |

---

## 5. Shared Resources Summary

### 5.1 DB2 Tables

| Table | Used by Programs | Access Types |
|---|---|---|
| `ACCOUNT` | BANKDATA, CREACC, DBCRFUN, DELACC, DELCUS¹, INQACC, INQACCCU, INQACCS, UPDACC, XFRFUN, BNKSTMT | READ, WRITE (INSERT, UPDATE, DELETE, cursor) |
| `CUSTOMER` | BANKDATA, CRECUST, DELCUS, INQCUST, UPDCUST, BNKSTMT | READ, WRITE (INSERT, UPDATE, DELETE) |
| `CONTROL` | CREACC | READ/WRITE (SELECT + UPDATE — account number sequence) |
| `PROCTRAN` | CREACC, CRECUST, DBCRFUN, DELACC, DELCUS, INQTRAND, INQTRANL, XFRFUN, BNKSTMT | READ, WRITE (INSERT, cursor FETCH) |
| `STTESTER.CONTROL` | BANKDATA, CRECUST | READ/WRITE (customer number sequence) |

> ¹ DELCUS accesses ACCOUNT indirectly through its call to DELACC/INQACCCU.

---

### 5.2 IMS Databases (DBDs)

| DBD | Segment(s) | Used by MPPs | Used by Load |
|---|---|---|---|
| `ACCOUNT` | ACCOUNT | IBACSUM (G), IBTRAN (R) | LOADACCT (L) |
| `ACCTYPE` | ACCTYPE | — | LOADACCT (L) |
| `CUSTACCS` | CUSTACCS | IBACSUM (G), IBTRAN (G) | LOADCUSA (L) |
| `CUSTOMER` | CUSTOMER | IBGCUDAT (G), IBLOGIN (R), IBLOGOUT (R), IBSCUDAT (R) | LOADCUST (L) |
| `CUSTTYPE` | CUSTTYPE | — | LOADACCT (L) |
| `HISTORY` | HISTORY | IBTRAN (I — insert only) | LOADHIST (L) |
| `TSTAT` | TSTAT | — | LOADTSTA (L) |
| `TSTATTYP` | TSTATTYP | — | LOADTSTA (L) |
| `TTYPE` | TTYPE | — | LOADTSTA (L) |

> PROCOPT legend: G=Get, R=Read+update (GHU/REPL), I=Insert, L=Load, A/AP=All

---

### 5.3 CICS BMS Maps

| BMS Mapset file | Symbolic copybook | Used by program | Function |
|---|---|---|---|
| [`BNK1MAI.bms`](src/base/cics/bms/BNK1MAI.bms) | BNK1MAI | BNKMENU | Main menu |
| [`BNK1CAM.bms`](src/base/cics/bms/BNK1CAM.bms) | BNK1CAM | BNK1CAC | Create account |
| [`BNK1ACC.bms`](src/base/cics/bms/BNK1ACC.bms) | BNK1ACC | BNK1CCA | Customer/account enquiry |
| [`BNK1CCM.bms`](src/base/cics/bms/BNK1CCM.bms) | BNK1CCM | BNK1CCS | Create customer |
| [`BNK1CDM.bms`](src/base/cics/bms/BNK1CDM.bms) | BNK1CDM | BNK1CRA | Debit/credit |
| [`BNK1DAM.bms`](src/base/cics/bms/BNK1DAM.bms) | BNK1DAM | BNK1DAC | Delete account |
| [`BNK1DCM.bms`](src/base/cics/bms/BNK1DCM.bms) | BNK1DCM | BNK1DCS | Delete customer |
| [`BNK1TFM.bms`](src/base/cics/bms/BNK1TFM.bms) | BNK1TFM | BNK1TFN | Transfer funds |
| [`BNK1UAM.bms`](src/base/cics/bms/BNK1UAM.bms) | BNK1UAM | BNK1UAC | Update account |
| [`BNK1B2M.bms`](src/base/cics/bms/BNK1B2M.bms) | — | (supplementary) | Auxiliary map (not yet linked to a program in scan) |

---

### 5.4 Copybooks

| Copybook | Purpose | Used by |
|---|---|---|
| `ABNDINFO` | Abend info data structure | All CICS programs |
| `ACCDB2` | ACCOUNT DB2 host variable group | BANKDATA, CREACC, DBCRFUN, DELACC, INQACC, INQACCCU, INQACCS, UPDACC, XFRFUN |
| `ACCOUNT` | Account data structure | CREACC, DBCRFUN, DELACC, DELCUS, INQACC, INQACCCU, UPDACC, XFRFUN |
| `ACCTCTRL` | Account control (sequence) | BANKDATA, CREACC, DELACC |
| `CEEIGZCT` | CEE condition token | CRECUST |
| `CONTDB2` | CONTROL DB2 host variables | BANKDATA, CRECUST |
| `CREACC` | Create account commarea | CREACC |
| `CRECUST` | Create customer commarea | CRECUST |
| `CUSTCTRL` | Customer control (sequence) | BANKDATA, CRECUST |
| `CUSTDB2` | CUSTOMER DB2 host variables | BANKDATA, CRECUST, DELCUS, INQCUST, UPDCUST |
| `CUSTOMER` | Customer data structure | BANKDATA, CRDTAGY1–5, CREACC, CRECUST, DELCUS, INQACCCU, INQCUST, UPDCUST |
| `DELACC` | Delete account commarea | DELACC |
| `DELCUS` | Delete customer commarea | DELCUS, BNK1DCS |
| `DFHAID` | CICS attention-identifier keys | All BNK1xxx screen programs, BNKMENU |
| `DFHBMSCA` | BMS colour/attribute constants | BNK1DCS |
| `INQACC` | Inquire-account commarea | INQACC, BNK1DAC |
| `INQACCCU` | Inquire-accounts-by-customer commarea | INQACCCU, BNK1CCA, CREACC, DELCUS |
| `INQCUSTZ` | Customer inquiry commarea | INQACCCU, INQCUST, CREACC, DELCUS, BNK1DCS |
| `INQTRAND` | Transaction detail inquiry commarea | INQTRAND |
| `INQTRANL` | Transaction list inquiry commarea | INQTRANL |
| `PAYDBCR` | Debit/credit payment commarea | DBCRFUN |
| `PROCDB2` | PROCTRAN DB2 host variables | CREACC, CRECUST, DBCRFUN, DELACC, DELCUS, INQTRAND, INQTRANL, XFRFUN |
| `PROCTRAN` | Processed-transaction data structure | CREACC, CRECUST, DBCRFUN, DELACC, DELCUS, XFRFUN |
| `SORTCODE` | Bank sort code constant | Almost all CICS programs |
| `SQLCA` | SQL communication area | All DB2-accessing CICS programs |
| `SQLDA` | SQL descriptor area | XFRFUN |
| `UPDACC` | Update account commarea | UPDACC, BNK1DCS |
| `UPDCUST` | Update customer commarea | UPDCUST, BNK1DCS |
| `XFRFUN` | Transfer function commarea | XFRFUN |
| `BNK1MAI/CAM/ACC/CCM/CDM/DAM/DCM/TFM/UAM` | BMS-generated symbolic maps | Respective BNK1xxx programs |
| `JNI` | Java Native Interface binding | IBTRAN (IMS) |

---

### 5.5 CICS Files (VSAM)

| Logical Name | Physical Name | Access | Used by |
|---|---|---|---|
| `ABNDFILE` | ABNDFILE | WRITE (CICS WRITE) | ABNDPROC |

---

*End of Inventory*
