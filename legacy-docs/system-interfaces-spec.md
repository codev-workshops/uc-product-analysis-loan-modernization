# System Interface Specification: Loan Origination System

## 1. Credit Bureau Interface

### 1.1 Request Format (Outbound)
```xml
<CreditRequest>
  <Header>
    <RequestorID>LOS-PROD-001</RequestorID>
    <Timestamp>2024-01-15T10:30:00Z</Timestamp>
    <Bureau>EQUIFAX|EXPERIAN|TRANSUNION</Bureau>
  </Header>
  <Borrower>
    <SSN>encrypted_value</SSN>
    <FirstName>JOHN</FirstName>
    <LastName>DOE</LastName>
    <DOB>1985-03-22</DOB>
    <Address>
      <Street>123 MAIN ST</Street>
      <City>ANYTOWN</City>
      <State>CA</State>
      <Zip>90210</Zip>
    </Address>
  </Borrower>
  <ProductType>MORTGAGE</ProductType>
</CreditRequest>
```

### 1.2 Response Format (Inbound)
```xml
<CreditResponse>
  <Header>
    <ResponseID>EQX-2024-001234</ResponseID>
    <Timestamp>2024-01-15T10:30:05Z</Timestamp>
    <Status>SUCCESS|ERROR|TIMEOUT</Status>
  </Header>
  <CreditReport>
    <Score>742</Score>
    <ScoreModel>FICO8</ScoreModel>
    <Tradelines count="12">
      <Tradeline>
        <Creditor>CHASE VISA</Creditor>
        <Balance>4500.00</Balance>
        <Limit>15000.00</Limit>
        <MonthlyPayment>135.00</MonthlyPayment>
        <Status>CURRENT</Status>
      </Tradeline>
      <!-- additional tradelines -->
    </Tradelines>
    <PublicRecords count="0"/>
    <Inquiries count="2"/>
  </CreditReport>
</CreditResponse>
```

## 2. Core Banking Interface (FIS)

### 2.1 MQ Message Format
- Queue Manager: QM_LOS_PROD
- Request Queue: LOS.TO.FIS.REQUEST
- Response Queue: FIS.TO.LOS.RESPONSE
- Message Format: Fixed-width EBCDIC (COBOL copybook layout)

### 2.2 Copybook Layout (Account Creation)
```
01 FIS-ACCT-CREATE-REQ.
   05 MSG-TYPE          PIC X(4)   VALUE 'ACCT'.
   05 LOAN-NUMBER       PIC 9(12).
   05 BORROWER-SSN      PIC 9(9).
   05 LOAN-TYPE         PIC X(3).
   05 LOAN-AMOUNT       PIC 9(9)V99.
   05 INTEREST-RATE     PIC 9(2)V9999.
   05 TERM-MONTHS       PIC 9(3).
   05 FIRST-PMT-DATE    PIC 9(8).
   05 FUNDING-DATE      PIC 9(8).
   05 FILLER            PIC X(50).
```

## 3. Document Management Interface (FileNet)

### 3.1 SOAP Endpoint
- WSDL: `https://filenet.internal.bank.com/wsi/FNCEWS40MTOM/`
- Operations: CreateDocument, RetrieveDocument, SearchDocuments
- Authentication: WS-Security UsernameToken

### 3.2 Document Types
| Doc Type Code | Description | Required For |
|--------------|-------------|--------------|
| APP-001 | Loan Application (1003) | All loans |
| INC-001 | Pay Stubs | All loans |
| INC-002 | W-2 Forms | All loans |
| INC-003 | Tax Returns | All loans |
| PPT-001 | Property Appraisal | Purchase/Refi |
| TTL-001 | Title Report | All loans |
| INS-001 | Homeowners Insurance | All loans |

## 4. Batch File Interfaces

### 4.1 GL Posting File
- Format: Fixed-width ASCII
- Delivery: SFTP to `gl-feed.internal.bank.com:/incoming/los/`
- Naming: `LOS_GL_YYYYMMDD.dat`

```
Record Layout:
Pos 1-8:    Posting Date (YYYYMMDD)
Pos 9-20:   GL Account Number
Pos 21-22:  Debit/Credit Indicator (DR/CR)
Pos 23-37:  Amount (13.2, zero-filled)
Pos 38-50:  Loan Number
Pos 51-80:  Description
Pos 81-85:  Cost Center
```

### 4.2 Compliance Extract
- Format: Pipe-delimited CSV
- Delivery: SFTP to `compliance.internal.bank.com:/incoming/los/`
- Naming: `LOS_HMDA_YYYYMM.csv`
- Fields: Application ID | Action Taken | Action Date | Loan Type | Loan Purpose | Loan Amount | Property Type | Property State | Applicant Race | Applicant Sex | Applicant Income | Rate Spread
