# Business Requirements Document: Loan Origination System

## Document Information
- **System**: Legacy Loan Origination Platform (LOS)
- **Version**: 4.2.1
- **Last Updated**: 2019-03-15
- **Status**: Production

## 1. System Overview

The Loan Origination System (LOS) is the primary platform for processing consumer and commercial loan applications. The system handles the complete lifecycle from application intake through funding.

### 1.1 Architecture
- **Frontend**: Green-screen terminal interface (3270 emulation)
- **Backend**: COBOL batch programs on IBM z/OS mainframe
- **Database**: IBM DB2 z/OS
- **Middleware**: IBM MQ Series for inter-system messaging
- **Batch Scheduler**: Control-M for nightly batch processing

### 1.2 Integration Points
| System | Protocol | Direction | Description |
|--------|----------|-----------|-------------|
| Core Banking (FIS) | MQ Series | Bidirectional | Account creation, funding |
| Credit Bureaus (Equifax, Experian, TransUnion) | HTTPS/XML | Outbound | Credit pulls |
| Document Management (FileNet) | SOAP/WSDL | Bidirectional | Document storage/retrieval |
| Compliance Engine | Batch file | Outbound | Regulatory reporting |
| General Ledger | Batch file | Outbound | Nightly GL postings |

## 2. Business Rules

### 2.1 Loan Eligibility
- Minimum credit score: 620 (conventional), 580 (FHA)
- Maximum DTI ratio: 43% (conventional), 50% (FHA)
- Employment verification: minimum 2 years continuous
- Income verification: 2 years tax returns + 30 days pay stubs

### 2.2 Pricing Rules
- Base rate from daily rate sheet (pulled from Treasury feed at 6:00 AM ET)
- Risk-based adjustments per credit tier:
  - 760+: -0.25%
  - 720-759: 0.00%
  - 680-719: +0.50%
  - 640-679: +1.00%
  - 620-639: +1.75%
- Lock periods: 30, 45, 60 days
- Lock extension fee: 0.125% per 7 days

### 2.3 Underwriting Decision Matrix
| Loan Type | Auto-Approve Threshold | Manual Review | Auto-Decline |
|-----------|----------------------|---------------|--------------|
| Conventional < $500K | Score >= 740, DTI <= 36% | Score 620-739 or DTI 36-43% | Score < 620 or DTI > 43% |
| Conventional >= $500K | Score >= 760, DTI <= 33% | Score 680-759 or DTI 33-43% | Score < 680 or DTI > 43% |
| FHA | Score >= 680, DTI <= 40% | Score 580-679 or DTI 40-50% | Score < 580 or DTI > 50% |

## 3. Batch Processing Schedule

| Job Name | Schedule | Description | SLA |
|----------|----------|-------------|-----|
| LOS-DAILY-001 | 6:00 AM ET | Rate sheet import | Complete by 7:00 AM |
| LOS-DAILY-002 | 8:00 PM ET | Credit bureau batch pulls | Complete by 11:00 PM |
| LOS-DAILY-003 | 11:30 PM ET | GL posting extract | Complete by 12:30 AM |
| LOS-WEEKLY-001 | Saturday 2:00 AM | Compliance reporting | Complete by 6:00 AM |
| LOS-MONTHLY-001 | 1st of month 1:00 AM | Regulatory filing (HMDA) | Complete by 5:00 AM |

## 4. Data Model (Key Entities)

### LOAN_APPLICATION
- APPLICATION_ID (PK)
- BORROWER_ID (FK)
- LOAN_TYPE_CD
- LOAN_AMOUNT
- PROPERTY_ADDRESS
- APPLICATION_DT
- STATUS_CD
- ASSIGNED_UNDERWRITER_ID
- DECISION_DT
- DECISION_CD

### BORROWER
- BORROWER_ID (PK)
- SSN (encrypted)
- FIRST_NAME, LAST_NAME
- DOB
- EMPLOYMENT_STATUS_CD
- ANNUAL_INCOME
- CREDIT_SCORE
- DTI_RATIO

### LOAN_PRICING
- PRICING_ID (PK)
- APPLICATION_ID (FK)
- BASE_RATE
- RISK_ADJUSTMENT
- FINAL_RATE
- LOCK_DT
- LOCK_EXPIRY_DT
- LOCK_STATUS_CD

## 5. Known Issues / Technical Debt
1. Credit score caching logic uses stale data (30-day cache, should be 14 days per new regulation)
2. DTI calculation does not account for student loan IBR plans (manual override required)
3. Rate lock extension logic has rounding errors above 3 extensions
4. Batch job LOS-DAILY-002 frequently exceeds SLA due to Experian timeout issues
5. No audit trail for manual underwriting overrides (compliance risk)
