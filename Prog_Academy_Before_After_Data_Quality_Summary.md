# Prog Academy — Before vs After Data Quality Summary

## 1. Purpose

This document summarizes how data quality changed between the nine original raw datasets and the final cleaned datasets produced with Python/Pandas.

It focuses on **Before → Cleaning Action / Decision → After**. Values were corrected or standardized only when the source data supported a clear decision. Where a safe correction was not possible, the original record was kept and the issue was converted to null, preserved, or flagged for review.

## 2. Overall Data Quality Improvement

| Dataset | Raw Rows | Cleaned Rows | Main Initial Issues | Final State |
|---|---:|---:|---|---|
| `Managers` | 8 | 8 | Five null `TerminationDate` values; otherwise structurally clean | All 8 records retained; unique `ManagerID`; legitimate termination-date nulls preserved |
| `Courses` | 17 | 17 | No material data-quality issue identified | All 17 course records retained; unique `CourseID`; dates, numeric rules and currency validated |
| `Leads` | 51,779 | 51,021 | Duplicate `LeadID` records, duplicate-person candidates, inconsistent contact/geography/source fields, mixed dates and invalid expected prices | 51,021 unique `LeadID` values; duplicate IDs resolved; cleaned and validation fields added; unresolved values preserved or flagged |
| `MarketingSpend` | 13,968 | 13,889 | 79 exact duplicates, missing `Spend`, campaign variants, spend/base-currency mismatches and click/impression anomalies | 79 exact duplicates removed; campaign values standardized; 79 missing spend values recovered; remaining exceptions flagged |
| `Payments` | 17,174 | 17,100 | 74 exact duplicates, currency variants, financial arithmetic issues, installment errors, invoice issues and missing dates | 74 exact duplicates removed; currencies, financial calculations and installments validated; remaining invoice/refund exceptions preserved |
| `Enrollments` | 12,161 | 12,161 | Legitimate repeat enrollments, 745 late enrollments, smaller date-order exceptions and status-dependent null dates | No rows removed; unique `EnrollmentID`; late enrollments flagged; other valid or unresolved exceptions preserved |
| `StudentActivity` | 111,641 | 111,074 | 567 exact duplicates, negative activity values, invalid quiz scores, homework/date anomalies and weekly gaps | 567 exact duplicates removed; invalid numeric values converted to null; remaining exceptions flagged |
| `ExchangeRates` | 3,644 | 3,644 | 11 missing date-currency combinations within the observed date range | All rows retained; unique `Date + Currency`; positive rates and UAH rate of 1 validated; coverage gaps preserved |
| `Cohorts` | 242 | 242 | No material key, date or numeric issue identified | All 242 records retained; unique `CohortID`; course, date and enrollment-count checks passed |

## 3. Before vs After by Dataset

### 3.1 Managers

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| `TerminationDate` nulls | 5 null values, all for active managers | Parsed employment dates and kept null termination dates for active managers | 5 null `TerminationDate` values remain as legitimate missing values |
| Structural quality | 8 rows; unique `ManagerID`; no exact duplicates or invalid salary/bonus/target rules | Validated data types, dates and numeric business rules | 8 rows; 0 duplicate `ManagerID`; no material structural issues found |

### 3.2 Courses

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Material data-quality issues | None identified across 17 course records | Parsed `LaunchDate` and validated keys, numeric rules and currency | 17 rows retained; 0 duplicate `CourseID`; 0 missing `LaunchDate`; all `BaseCurrency` values are UAH |

### 3.3 Leads

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Duplicate `LeadID` records | 51,779 rows and 51,021 unique `LeadID` values; 758 excess duplicate-ID rows, including 279 exact duplicates and 479 non-exact repeated-ID rows | Removed exact duplicates and consolidated repeated `LeadID` records where differences were limited to fields that could be resolved from the available data | 51,021 rows; 51,021 unique `LeadID`; 0 duplicate `LeadID`; 0 exact duplicates |
| Duplicate-person candidates | 1,033 raw candidate rows representing 1,021 unique candidate `LeadID` values | Kept legitimate CRM lead records and created `PersonKey` for unique-person analysis | 1,021 candidate leads retained; 50,000 unique `PersonKey` values |
| Course and manager name variants | 6,442 `CourseNameRaw` and 2,740 `ManagerNameRaw` values differed from the ID-based reference names | Mapped names using `CourseID` and `ManagerID`; kept the original raw text | 0 course-name mapping mismatches and 0 manager-name mapping mismatches |
| Email quality | 4,754 raw emails required trim/lowercase normalization; 776 failed format checks | Created `EmailNormalized` and `EmailIsValid`; kept the original `Email`; invalid addresses were not replaced | 766 invalid emails remain after lead deduplication and can be identified with `EmailIsValid` |
| Phone quality | 1,510 blank phones, 1,024 invalid/suspicious values and 6,839 plausible but non-E.164 values | Normalized supported phone numbers to E.164; kept invalid or unresolved values; applied 73 targeted corrections supported by matching person data | 48,156 Valid, 1,489 Missing, 1,007 Invalid and 369 Unresolved; all Valid normalized phones pass the E.164 format check |
| Geography | 120 raw country values; 2,490 rows used non-standard country values; city and region fields also contained missing values, placeholders and variants | Standardized country, city and region values using approved mappings; placeholders were converted to null | Geography was reduced to standardized values; 29 `CityNormalized` and 29 `RegionNormalized` values remain missing |
| Mixed or malformed dates | 1,402 malformed nonblank date values affecting 1,383 raw rows | Parsed supported date formats, kept the raw timestamp fields, converted malformed values to null and added date-quality flags | 1,362 cleaned leads have `HasMalformedDate=True`; 2 retain a date-order issue; 6 are flagged as created before course launch |
| Source, campaign and other categorical variants | `Source` had 51 raw spellings; 589 rows used known source aliases; campaign also contained missing values and placeholders | Standardized source aliases, status, temperature, device, language and campaign values; kept `fb-insta` as a separate source; campaign placeholders were converted to null | Normalized source fields use a consistent set of values; `CampaignNormalized` contains 0 placeholder values |
| Invalid expected course price | 247 raw rows contained negative or clear sentinel/extreme price values | Kept the original `ExpectedCoursePrice`; created `ExpectedCoursePriceClean` and `ExpectedCoursePriceIsValid`; invalid prices were converted to null in the clean field | 244 invalid price rows remain after deduplication; all have null `ExpectedCoursePriceClean` |

### 3.4 Marketing Spend

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Duplicate daily-grain rows | 79 exact duplicate rows at `Date + Source + Campaign + Country + DeviceType` grain | Removed exact duplicates | 13,889 rows; 0 exact duplicates; 0 duplicate logical-grain rows |
| Campaign naming | 162 distinct non-null raw campaign strings with multiple case, typo and copy variants | Standardized known campaign variants while keeping the original `Campaign` value | 15 standardized non-null `CampaignNormalized` values; no remaining case-insensitive duplicate campaign families |
| Missing `Spend` | 106 raw missing values; 81 had positive `SpendBaseCurrency` evidence | Filled `Spend` only when the available UAH base-currency value supported the correction | 79 values were filled after duplicate removal; 25 `Spend` values remain missing, and none has a positive `SpendBaseCurrency` value available for recovery |
| `Spend` / `SpendBaseCurrency` mismatch | 13 UAH mismatches: 9 with a positive base value and 4 with `SpendBaseCurrency = 0` | Corrected the 9 supported cases; kept and flagged the 4 unresolved cases | 4 mismatches remain; all 4 have `SpendBaseCurrencyUnresolvedFlag=True` |
| Click/impression anomalies and outliers | 16 rows had `Clicks > Impressions`; statistical outliers were also identified during profiling | Kept the source values where no safe correction was available; click/impression anomalies were flagged; statistical outliers were not automatically removed | 16 rows remain flagged with `ClicksImpressionsAnomalyFlag=True`; outlier values remain available for analysis |

### 3.5 Payments

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Duplicate transactions | 74 excess `PaymentID` rows; all were exact duplicates | Removed exact duplicate transaction rows | 17,100 rows; 17,100 unique `PaymentID`; 0 exact duplicates |
| Currency and FX | 72 raw rows used non-standard currency values; some payments could not use an exact date-currency lookup | Standardized supported currency values, used `ExchangeRates` where available and used the stored payment rate where the exact lookup was unavailable | 0 unresolved currencies; 0 missing or non-positive `FXRateApplied`; 0 gross/net base-currency reconciliation mismatches |
| Financial amount and refund arithmetic | 16 payments had negative `GrossAmount`; 80 deduplicated records failed the net-amount formula; refund/gross conflicts were also present | Corrected amount/sign and refund values where the payment fields supported a clear correction; kept unresolved refund-status meaning as a flag | 0 negative `GrossAmount`; 0 `RefundAmount > GrossAmount`; 0 net-amount arithmetic mismatches; 50 `RefundStatusIsInconsistent=True` records remain flagged |
| Installment numbering | 112 payments had `InstallmentNumber > InstallmentCount` | Corrected installment numbers using payment type and transaction history | 0 remaining installment-number violations |
| Invoice quality | 24 invoices were missing and 131 non-missing raw values failed the required pattern | Standardized recoverable formatting; kept genuinely missing or malformed values | 24 invoices remain missing; 68 remain malformed; 0 duplicate valid normalized invoice numbers |
| Missing `PaymentDate` | 75 deduplicated payments had no payment date | Kept the records because no reliable replacement date was available; stored payment FX data was used where required | 75 `PaymentDate` values remain missing; all records still have a valid `FXRateApplied` |
| Test payments and KPI eligibility | 68 deduplicated payments were marked `IsTestPayment=True` | Kept test payments for auditability. `IsFinancialKPIEligible` is based on successful payment status (`Completed`, `Refunded`, `Partially Refunded`), not on the test-payment flag | 68 test payments retained; 14,092 payments are KPI-eligible; 3,008 unsuccessful payments are excluded from financial KPIs; all 58 successful test payments are included in KPI eligibility |

### 3.6 Enrollments

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Repeated students | 474 students had more than one enrollment, representing 485 additional enrollments; `EnrollmentID` was unique | Kept repeated enrollments because the dataset grain is one row per enrollment | All 12,161 rows retained; 0 duplicate `EnrollmentID` |
| Late and unusual date sequences | 745 late enrollments; 35 enrollments after `ExpectedEndDate`; 5 records had `ActualCompletionDate < EnrollmentDate` | Added `LateEnrollment`; kept the stronger date exceptions because there was not enough evidence to replace the source dates | 745 `LateEnrollment=True`; the 35 and 5 stronger date exceptions remain unchanged |
| Status-dependent missing dates | Completion, cancellation and certificate dates contained many nulls, but these fields depend on enrollment status | Checked the dates against `EnrollmentStatus`, cancellation fields and certificate rules; did not fill legitimate nulls | 0 status/date inconsistencies, 0 cancellation-reason inconsistencies and 0 certificate inconsistencies |
| Relationship integrity | Foreign keys were valid in the raw data | Rechecked lead, course, manager and cohort relationships | 0 orphan foreign keys; 0 enrollment-to-cohort inconsistencies; 0 cohort enrollment-count mismatches |

### 3.7 Student Activity

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Duplicate activity rows | 567 exact duplicate rows duplicated both `ActivityID` and the `StudentID + EnrollmentID + ActivityWeek` grain | Removed exact duplicates | 111,074 rows; 0 duplicate `ActivityID`; 0 duplicate natural-grain rows |
| Negative activity metrics and lesson consistency | 464 raw rows contained at least one negative activity value; 126 raw rows had `LessonsCompleted > LessonsViewed` | Converted invalid negative metrics to null instead of zero; validated the lesson relationship after cleaning | 460 affected rows remained after deduplication and were cleaned; 0 negative activity values remain; 0 `LessonsCompleted > LessonsViewed` cases remain |
| Invalid quiz scores | 631 raw rows had `AverageQuizScore > 100` | Converted invalid quiz scores to null rather than capping them at 100 | 627 invalid scores remained after deduplication and were converted to null; 0 quiz scores remain outside 0–100 |
| Homework inconsistency | 645 raw rows had `HomeworkAccepted > HomeworkSubmitted` | Kept the original values because no safe correction was available; added `HomeworkConsistencyFlag` | 642 cases remain after deduplication; all 642 are flagged |
| `LastActivityAt` anomaly | 362 rows had `LastActivityAt` outside the expected seven-day activity period | Kept the timestamp and added `LastActivityAtAnomalyFlag` | 362 cases remain and all are flagged |
| Weekly activity semantics and gaps | 97,275 raw `ActivityWeek` values were not Monday; sequence checks also found missing weekly periods | Kept the supplied weekly dates rather than shifting them; did not create artificial zero-activity rows; added gap fields | 817 rows have `ActivityGapBeforeFlag=True`, representing 827 detected missing weekly periods |

### 3.8 Exchange Rates

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Daily-currency coverage | 11 missing `Date + Currency` combinations within the observed date range | Kept the coverage gaps instead of inventing exchange rates | 3,644 rows retained; 0 duplicate `Date + Currency` keys; 0 non-positive rates; all UAH rates equal 1; 11 coverage gaps remain |

### 3.9 Cohorts

| Issue | Before | Cleaning Action / Decision | After |
|---|---|---|---|
| Material data-quality issues | None identified across 242 cohort records | Parsed dates and validated `CourseID`, date order, seat/enrollment rules and cohort totals | 242 rows retained; 0 duplicate `CohortID`; 0 invalid `CourseID`; no date or numeric-rule issues; 0 cohort enrollment-count mismatches |

## 4. Key Quality Improvements

- **Key uniqueness:** all final primary or natural keys required by the project are unique. Duplicate grains were removed from `Leads`, `MarketingSpend`, `Payments` and `StudentActivity`.
- **Duplicate removal:** 999 exact duplicate rows were removed across `Leads` (279), `MarketingSpend` (79), `Payments` (74) and `StudentActivity` (567). A further 479 non-exact repeated `LeadID` rows were consolidated, giving a total row reduction of 1,478.
- **Reference-data mapping:** `CourseID` and `ManagerID` were used to map consistent course and manager names while keeping the original raw text.
- **Text and category standardization:** lead geography, source aliases, statuses, device/language values and campaign fields were standardized for consistent reporting.
- **Contact-data validation:** email and phone fields were normalized and validated. Invalid or unresolved values were kept with clear validation fields instead of being guessed.
- **Date validation:** mixed date formats were parsed, malformed dates were converted to null, raw lead timestamps were retained, and remaining date-order exceptions were flagged.
- **Financial-data validation:** payment currencies, FX rates, amounts, refunds, installments and invoices were checked and corrected where supported by the data. Financial KPI eligibility is stored separately in `IsFinancialKPIEligible`.
- **Marketing-spend validation:** exact duplicates and supported spend/campaign issues were corrected, while unresolved spend/base and click/impression anomalies remain flagged.
- **Student-activity validation:** invalid negative values and quiz scores were converted to null; homework, timestamp and weekly-gap issues remain visible through data-quality flags.
- **Foreign-key integrity:** the cleaned datasets have 0 orphan foreign keys across the validated manager, course, lead, cohort and enrollment relationships.

## 5. Issues Intentionally Preserved or Flagged

| Dataset | Preserved / Flagged Issue | Final Treatment and Rationale |
|---|---|---|
| `Managers` | 5 null `TerminationDate` values | Kept because they belong to active managers and are legitimate nulls |
| `Leads` | 766 invalid emails; 1,007 invalid and 369 unresolved phones | Original values were kept and validation fields were used so these records can be filtered or reviewed without inventing contact details |
| `Leads` | 1,362 leads with malformed dates, 2 date-order issues and 6 pre-launch leads | Malformed parsed values were converted to null while raw timestamps were kept; remaining date exceptions are identifiable through flags |
| `Leads` | First-touch and last-touch attribution differences | Both attribution fields were kept because they represent different valid marketing views rather than a data error |
| `MarketingSpend` | 25 missing `Spend`, 4 spend/base mismatches and 16 `Clicks > Impressions` cases | Values without enough evidence for correction were kept missing or unchanged and the relevant anomalies were flagged |
| `Payments` | 75 missing `PaymentDate`, 50 refund-status inconsistencies, 24 missing invoices and 68 malformed invoices | Records were kept because there was not enough evidence to safely replace these values; the relevant exceptions remain visible |
| `Payments` | 68 test payments | All test records were kept. KPI eligibility is based on successful payment status, so the 58 successful test payments are included in `IsFinancialKPIEligible` |
| `Enrollments` | 745 late enrollments, 35 enrollments after expected end and 5 completion-before-enrollment cases | Late enrollments were flagged; stronger date exceptions were kept unchanged because a correct replacement date could not be inferred |
| `Enrollments` | Status-dependent missing completion, cancellation and certificate dates | Kept as legitimate nulls where the business status does not require a date |
| `StudentActivity` | 642 homework inconsistencies and 362 `LastActivityAt` anomalies | Original values were kept and flagged because no safe correction was available |
| `StudentActivity` | Non-Monday `ActivityWeek` values and 827 detected missing weekly periods | Supplied weekly dates were kept and missing weeks were not filled with invented zero-activity rows; gaps remain identifiable |
| `ExchangeRates` | 11 missing date-currency combinations | Kept as source coverage gaps rather than filled with invented FX rates |

## 6. Final Data Quality Status

The nine cleaned datasets are suitable for Power BI modelling and KPI calculation.

All required final primary or natural keys are unique, and the validated foreign-key relationships have **0 orphan records**. Exact duplicates were removed where appropriate, while legitimate repeated business records, such as multiple enrollments for the same student, were kept.

Invalid values were corrected only when the source data supported a clear decision. Otherwise, values were converted to null, kept in the original field, or marked with a data-quality flag. Raw values were retained where they are useful for audit and validation, including original lead timestamps and descriptive/contact fields.

Cleaning was also kept separate from KPI filtering. In `Payments`, all records remain available in the cleaned dataset, while `IsFinancialKPIEligible` identifies payments included in financial KPIs based on successful payment status. This includes 58 successful test payments, matching the final notebook logic.
