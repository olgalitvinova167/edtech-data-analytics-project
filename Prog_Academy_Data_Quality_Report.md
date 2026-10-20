# Prog Academy --- Data Quality Report

## 1. Purpose and Audit Scope

This report presents the initial data quality assessment of the nine raw
datasets provided for the Prog Academy EdTech analytics project.

The purpose of the audit is to understand the structure and quality of
the source data before cleaning, transformation, modelling and analysis.

The assessment covers:

-   dataset structure, row counts and expected granularity;
-   primary and natural key uniqueness, relationship integrity and
    foreign-key integrity;
-   missing values, duplicate records, and inconsistent text and
    categorical values;
-   invalid or inconsistent dates, numeric and business-rule violations,
    and cross-field and cross-table consistency;
-   potential outliers and other data-quality exceptions.

The audit is based on:

-   the original `DataDictionary.xlsx`, used to understand field
    definitions, expected dataset grain, key relationships and business
    rules;
-   all nine raw CSV datasets, used to assess the actual condition of
    the source data.

All counts and findings in this report refer to the raw datasets before
any cleaning or transformation.

### 1.1 Actual Raw Row Counts

| Dataset | Actual Raw Rows | Data Dictionary Rows | Expected Grain |
| --- | --- | --- | --- |
| Managers | 8 | 8 | One row per sales manager |
| Courses | 17 | 17 | One row per course |
| Leads | 51,779 | 50,000 | One row per CRM lead |
| MarketingSpend | 13,968 | 13,889 | Daily × source × campaign × country × device |
| Payments | 17,174 | 17,100 | One row per payment transaction |
| Enrollments | 12,161 | 12,161 | One row per enrollment |
| StudentActivity | 111,641 | 112,119 | One row per student--enrollment--week |
| ExchangeRates | 3,644 | 3,644 | Daily × currency |
| Cohorts | 242 | 242 | One row per cohort |

The audit uses the actual raw file counts rather than assuming the
overview counts are exact. The differences in `Leads`, `MarketingSpend`,
`Payments` and `StudentActivity` were therefore treated as items to
investigate rather than silently reconciled.

------------------------------------------------------------------------

## 2. Granularity, Keys and Relationship Validation

### 2.1 Dataset Keys

| Dataset | Primary / Natural Key | Raw-Key Result |
| --- | --- | --- |
| Managers | `ManagerID` | Unique; no exact duplicates |
| Courses | `CourseID` | Unique; no exact duplicates |
| Leads | `LeadID` | Not unique in raw data: 758 excess duplicate-ID rows across 750 duplicate-ID groups |
| MarketingSpend | `Date + Source + Campaign + Country + DeviceType` | 79 duplicate-grain rows; these are also exact duplicates |
| Payments | `PaymentID` | 74 excess duplicate-ID rows; all are exact duplicate transactions |
| Enrollments | `EnrollmentID` | Unique; no exact duplicates |
| StudentActivity | `ActivityID`; natural grain `StudentID + EnrollmentID + ActivityWeek` | 567 duplicate `ActivityID` rows and 567 duplicate natural-grain rows; all are exact duplicates |
| ExchangeRates | `Date + Currency` | Unique |
| Cohorts | `CohortID` | Unique |

For `Leads`, 279 of the duplicate rows were exact copies. After
separating exact copies for audit purposes, **479 non-exact repeated
`LeadID` rows** remained and differed only in descriptive/raw fields.

### 2.2 Foreign-Key Relationships

The raw canonical IDs were checked against the reference tables defined
in the Data Dictionary.

| Relationship Group | Raw Validation |
| --- | --- |
| `Managers.ManagerID` → Leads / Enrollments / Payments | 0 orphan foreign keys |
| `Courses.CourseID` → Leads / Cohorts / Enrollments / Payments / StudentActivity | 0 orphan foreign keys |
| `Leads.LeadID` → Enrollments / Payments | 0 orphan foreign keys |
| `Cohorts.CohortID` → Enrollments | 0 orphan foreign keys |
| `Enrollments.EnrollmentID` → Payments / StudentActivity | 0 orphan foreign keys |
| `StudentID` consistency between Enrollments and related Payments / StudentActivity | 0 enrollment-to-student mismatches |

The main relationship problem was therefore **duplicate parent/fact
keys**, not missing canonical foreign keys.

------------------------------------------------------------------------

## 3. Missing-Value Profile

Columns not listed below had **0 missing values** under the raw Pandas
import used by the project.

| Dataset | Columns with Missing Values | Interpretation |
| --- | --- | --- |
| Managers | `TerminationDate` 5 | Expected for the 5 active managers; not inherently a quality issue |
| Courses | None | No missing values |
| Leads | `FirstContactAt` 3,186; `ConvertedAt` 38,824; `FullName` 1,613; `Phone` 1,510; `Region` 385; `City` 1,953; `PreferredLanguage` 405; `LeadScore` 1,570; `Campaign` 11,630; `Content` 11,405; `Term` 23,912; `ReferrerDomain` 7,491; `LostReason` 21,364 | Mix of legitimate optional fields and data-quality gaps; field-specific treatment required |
| MarketingSpend | `Campaign` 2,938; `Spend` 106 | Missing campaign may be legitimate; missing spend requires validation against available base-currency evidence |
| Payments | `PaymentDate` 75; `RefundDate` 16,239; `RefundReason` 16,239; `InvoiceNumber` 24 | Refund fields are conditional; missing `PaymentDate` and invoice values require explicit treatment |
| Enrollments | `ActualCompletionDate` 6,520; `CancellationDate` 11,129; `CancellationReason` 11,129; `CertificateIssuedDate` 9,024 | Primarily status-dependent fields; not valid to impute universally |
| StudentActivity | None | No cell-level nulls, but weekly sequence gaps exist |
| ExchangeRates | None | No null cells; coverage gaps exist at the date-currency level |
| Cohorts | None | No missing values |

------------------------------------------------------------------------

## 4. Detailed Data Quality Findings and Proposed Decisions

### 4.1 Managers, Courses, Cohorts and Exchange Rates

| Dataset | Observed Issue / Finding | Verified Evidence and Example | Proposed Cleaning Decision |
| --- | --- | --- | --- |
| Managers | Nullable termination dates | 5 missing `TerminationDate` values, all belonging to active managers; no duplicate `ManagerID`, invalid salary/bonus/target values, employment-status/date conflicts or reversed employment dates were found | Preserve null `TerminationDate` for active managers; parse employment dates and retain existing canonical manager records |
| Courses | No material source-quality issue identified | 17 unique `CourseID` values; no nulls, exact duplicates, invalid numeric rules or non-UAH `BaseCurrency` values | Treat `Courses` as the canonical course reference; validate types and retain records |
| Cohorts | No material key/date/numeric issue identified | 242 unique `CohortID` values; no nulls, exact duplicates, invalid `CourseID`, reversed start/end dates or invalid seat/enrollment values | Retain as the canonical cohort reference after type validation |
| ExchangeRates | Incomplete daily currency coverage | 11 missing date-currency combinations between the minimum and maximum rate dates; e.g. `2024-01-20 / CZK`, `2024-04-11 / EUR`, `2024-08-18 / USD`, `2025-08-26 / USD` | Preserve the coverage gaps rather than invent exchange rates; use exact lookup where available and document any justified fallback source |

### 4.2 Leads

`Leads.csv` contained the broadest set of data-quality issues and
required both record-level deduplication and field-level
standardisation.

| Observed Issue | Verified Evidence and Example | Proposed Cleaning Decision |
| --- | --- | --- |
| Exact and partial duplicate leads | 51,779 raw rows but 51,021 unique `LeadID` values. There were 758 excess duplicate-ID rows across 750 groups, including 279 exact duplicate excess rows. After separating exact copies, 479 non-exact repeated-ID rows remained. Example: `LEAD0000064` appears with `CourseNameRaw` values `IT Start (Free)` / `It Start (Free)`, `Source` `meta` / `META`, and `City` `Pr@gue` / `Pr@Gue` | Remove exact copies; consolidate repeated `LeadID` rows only where differences are deterministic descriptive variants, using the canonical ID-based values |
| Duplicate-person candidates | 1,033 raw rows were flagged `IsDuplicateCandidate=True`, representing 1,021 unique LeadIDs. Identity checks confirmed the 1,021 candidates have stable-field counterparts after LeadID deduplication | Do not delete legitimate CRM LeadIDs; resolve person identity using normalized contact/stable fields and retain a separate person-level key for unique-person analysis |
| Course-name variants | 6,442 raw `CourseNameRaw` values did not exactly match the canonical `CourseName` for the same `CourseID`. Example: `CRS0005` `Fronted-end - online` vs canonical `Front-End` | Use `CourseID` as the authority and map the analytical course name to `Courses.CourseName`; preserve the raw text for auditability |
| Manager-name variants | 2,740 raw `ManagerNameRaw` values did not exactly match the canonical manager name for the same `ManagerID`. Example: `MGR0002`<br>Raw:       ' Natalia H. ' (leading/trailing spaces)<br>Canonical: 'Natalia H.' | Use `ManagerID` as the authority and map to the canonical manager name; retain the raw field |
| Email formatting and invalid values | 4,754 raw email strings changed under trim/lowercase normalization. 776 failed the project format checks: 295 without `@`, 170 with multiple `@`, 145 with internal whitespace and 171 with an invalid domain pattern. Examples: `jessica.барабаш6844.outlook.com`, `твердислав.wojciech2211@@icloud.com`, `meghan.nehring4122 @ icloud.com` | Trim and lowercase valid addresses; validate format; do not invent replacements for invalid addresses; preserve raw email and expose validity |
| Phone formatting and invalid values | 1,510 blank phones; 1,024 invalid/suspicious phones; 6,839 plausible but non-E.164 values; 42,406 already E.164-like. Examples: `call me` and `380540633447` | Normalize to E.164 only when country/number evidence is deterministic; retain invalid or unresolved raw values rather than guessing |
| Country / city / region inconsistency | `Country` contained 120 distinct raw values although the canonical marketing country domain contains 7 countries. 2,490 rows used non-canonical country values such as `UA`, `UKRAINE`, `PL`, `DE`, `POLAND`. `City` had 1,953 missing values plus 355 explicit placeholders; `Region` had 385 missing values plus 394 placeholders. Raw city variants included `Kiev`, `Odessa`, `Lvov`, `Praha` and typo/noise forms | Normalize country, city and region using deterministic synonym/transliteration mappings and cross-field evidence; convert placeholders to null; preserve unresolved values rather than fabricating geography |
| Mixed and malformed dates | Across `CreatedAt`, `UpdatedAt`, `FirstContactAt` and `ConvertedAt`, 1,402 nonblank raw date cells were malformed, affecting 1,383 raw rows. Examples include `2024-13-01`, `2024-01-32`, `0000-00-00`, `??`; valid dates also appeared in multiple formats | Parse with field-specific supported formats; coerce truly malformed values to null; retain original raw timestamps and flag malformed/date-order exceptions |
| Non-standard lead statuses and categorical casing | 544 raw `LeadStatus` rows were outside the Data Dictionary's exact `Won` / `Lost` / `In Progress` values, including `WON`, `won`, `lost`, `Open`, `closed`. Similar casing variation existed in `LeadTemperature`, `DeviceType` and `PreferredLanguage` | Normalize casing and known aliases deterministically; do not force ambiguous business statuses into another status without evidence |
| Source aliases and casing | `Source` had 51 raw spellings, collapsing to 18 values after trim/lowercase. 589 rows still used alias forms outside the canonical source list, including `fb`, `facebook`, `facebook_ads`, `www.google.com` and `www.google.com.ua` | Apply deterministic source alias mapping; keep `fb-insta` distinct where the source data supports separate attribution granularity |
| Campaign placeholders and inconsistent text | 11,630 raw `Campaign` values were missing and a further 363 contained explicit placeholders such as `unknown`, `?`, `тест`, `--` or `-`; 163 non-null raw campaign strings were present | Trim/lowercase for grouping, convert explicit placeholders to null, and map only known deterministic campaign variants; preserve genuine missing campaign values |
| Invalid expected course prices | 247 raw rows had invalid/sentinel prices: 78 negative values and 169 extreme values using `500000`, `9999999` or `12345678`. Example: `LEAD0000330` = `-30810 UAH` | Preserve the raw price, expose a clean analytical price only when valid, and flag invalid values rather than replacing them with an assumed course price |

### 4.3 Marketing Spend

| Observed Issue | Verified Evidence and Example | Proposed Cleaning Decision |
| --- | --- | --- |
| Duplicate daily-grain records | 79 excess rows were exact duplicates and also duplicated the defined `Date + Source + Campaign + Country + DeviceType` grain. Example duplicate: `2024-04-28 / google / Search - QA - Ukraine / Ukraine / Tablet` | Remove exact duplicate rows before aggregation |
| Missing spend | 106 raw rows had missing `Spend`; 81 of them had a positive `SpendBaseCurrency` value. Example: `2024-09-06 / google / Search - QA - Ukraine` had missing `Spend` and `SpendBaseCurrency = 6153.22` | Recover `Spend` only when supported deterministically by available base-currency information; otherwise preserve the missing value |
| Spend/base-currency inconsistency | 13 UAH rows had `Spend != SpendBaseCurrency`: 9 had a non-zero base amount and 4 had `SpendBaseCurrency = 0`. Example: `2025-04-16` had `Spend = 72048` and `SpendBaseCurrency = 2881.92` | Correct only where the source relationship is deterministic; retain unresolved rows with an explicit exception flag |
| Campaign naming inconsistency | 162 distinct non-null raw campaign strings were present; 14 case-insensitive campaign families had multiple spellings, with additional typo/copy variants such as `AI_COURSE_V1`, `Search - Brand - Ukraine_copy` and `Seach - Brand - Ukraine` | Standardize known campaign variants to canonical campaign names before grouping; preserve the raw campaign value |
| Clicks greater than impressions | 16 raw rows violated `Clicks <= Impressions`. Example: `2024-07-12 / Search - QA - Ukraine` had 2,405 impressions and 9,100 clicks | Treat as an explicit source-data anomaly; do not invent corrected click or impression values without evidence |
| Statistical outlier candidates | Raw IQR profiling identified 590 `Spend` and 1,360 `Impressions` outlier candidates | Review as possible anomalies but do not automatically remove or cap them because an outlier is not necessarily invalid |

### 4.4 Payments

| Observed Issue | Verified Evidence and Example | Proposed Cleaning Decision |
| --- | --- | --- |
| Duplicate transactions | 74 excess `PaymentID` rows; all were exact duplicates. Example: `PAY0000174` appeared twice with the same enrollment, date, amount and status | Remove exact duplicate transaction copies |
| Missing payment dates | 75 unique payment records had missing `PaymentDate` | Preserve when no deterministic transaction date is available; prevent unsupported date-based assumptions and document any FX fallback |
| Non-canonical currency values | 72 raw rows, representing 71 unique PaymentIDs, used currencies outside the ExchangeRates domain. Examples: `грн`, `UAH`, `usd`, `Dollar`, `EURO`, `XxX` | Normalize only where exchange-rate or transaction evidence identifies the intended currency; avoid blind text replacement for ambiguous codes |
| Negative gross amounts | 16 unique payments had `GrossAmount < 0`. Example: `PAY0000518` had `GrossAmount = -12533.33` | Correct sign only when the other financial fields provide consistent deterministic evidence |
| Refund / gross inconsistency | 74 raw rows initially satisfied `RefundAmount > GrossAmount`; this total included records affected by negative gross signs. After isolating those sign anomalies, 58 refund-specific conflicts remained | Reconcile refund values against gross, discount, processing fee, net amount and refund metadata; correct deterministic arithmetic cases and flag unresolved status inconsistencies |
| Net amount arithmetic inconsistency | 80 deduplicated raw payment records did not satisfy `NetAmount = GrossAmount - DiscountAmount - ProcessingFee - RefundAmount` before correction analysis | Recalculate or correct only where the component fields demonstrate the intended value; otherwise preserve and flag |
| Installment rule violations | 112 unique payments had `InstallmentNumber > InstallmentCount`. Example: `PAY0000466` had installment 8 of 6 | Correct installment sequencing only when payment type, enrollment history and transaction order provide deterministic evidence |
| Invoice quality | 24 invoices were missing and 131 non-missing values failed the raw `INV-YYYY-######` pattern, for 155 invoice-quality exceptions in total. Examples: `INV 2024 000509`, `INV-INV-000621` | Normalize trim/case/spacing where mechanically recoverable; retain missing or genuinely malformed invoice values as explicit exceptions |
| Test payments | 68 deduplicated payment records were marked `IsTestPayment=True` | Preserve test records for audit traceability; handle analytical eligibility separately from physical data cleaning |

### 4.5 Enrollments

| Observed Issue / Finding | Verified Evidence and Example | Proposed Cleaning Decision |
| --- | --- | --- |
| Repeated students are legitimate, not duplicate enrollments | `EnrollmentID` is fully unique. 474 students had more than one enrollment, representing 485 additional enrollments beyond one per student; maximum was 3 enrollments | Preserve repeated student enrollments because the dataset grain is one row per enrollment |
| Late enrollment | 745 enrollments had `EnrollmentDate > CourseStartDate`. Example: `ENR010913` enrolled `2025-11-13` for a course starting `2025-11-08` | Preserve as a business exception and expose a `LateEnrollment` flag rather than deleting the record |
| Stronger chronology exceptions | 35 enrollments occurred after `ExpectedEndDate`; 5 had `ActualCompletionDate < EnrollmentDate`. Example: `ENR011920` enrolled `2026-01-01` although expected end was `2025-12-30` | Preserve and flag chronology exceptions; do not invent corrected dates without supporting source evidence |
| Conditional missing dates | Completion, cancellation and certificate dates have high null counts but are status-dependent | Validate against `EnrollmentStatus` and related fields; do not impute dates simply because they are null |

No duplicate `EnrollmentID`, orphan foreign keys, invalid agreed
prices/discount percentages, reversed course start/end dates, or basic
cohort-key inconsistencies were identified in the raw enrollment
structure.

### 4.6 Student Activity

| Observed Issue | Verified Evidence and Example | Proposed Cleaning Decision |
| --- | --- | --- |
| Duplicate activity rows | 567 exact duplicate excess rows; the same 567 rows duplicated both `ActivityID` and the natural `StudentID + EnrollmentID + ActivityWeek` grain | Remove exact duplicate copies before activity analysis |
| Negative activity metrics | 464 raw rows contained at least one negative value in `Logins`, `LessonsViewed`, `VideoMinutesWatched` or `PlatformHours`. Example: `ACT00000146` had `VideoMinutesWatched = -259` | Treat invalid negative metrics as missing/invalid rather than legitimate activity; preserve the record |
| Lessons completed greater than lessons viewed | 126 raw rows violated `LessonsCompleted <= LessonsViewed`, often because `LessonsViewed` itself was negative | Resolve the invalid source metric first and prevent impossible lesson completion relationships |
| Invalid quiz percentages | 631 raw rows had `AverageQuizScore > 100`; no scores below 0 were found. Example: `ACT00000147` had `137.3` | Mark invalid quiz scores as missing rather than cap them to 100, because the intended score cannot be inferred |
| Homework inconsistency | 645 raw rows had `HomeworkAccepted > HomeworkSubmitted`. Example: `ACT00000014` had 1 submitted and 4 accepted | Preserve the source values and expose an exception flag unless a deterministic correction exists |
| `LastActivityAt` outside the activity period | 362 rows had `LastActivityAt >= ActivityWeek + 7 days`. Example: `ACT00000107` had `ActivityWeek = 2024-05-15` and `LastActivityAt = 2026-12-14 11:44:13` | Preserve the timestamp and flag the anomaly rather than invent a replacement date |
| `ActivityWeek` does not consistently represent Monday week-start | 97,275 raw rows had an `ActivityWeek` weekday other than Monday, despite the Data Dictionary description "Week start (Mon)" | Do not shift dates without evidence. Preserve the supplied weekly anchor and use interval-based validation; document the semantic discrepancy |
| Missing weekly observations | After removing exact duplicate copies solely to avoid distorting sequence analysis, 778 enrollments had 817 internal jumps longer than 7 days, representing 827 directly detectable missing weekly periods | Do not fabricate zero-activity rows; retain observed records and flag/count internal gaps |

------------------------------------------------------------------------

## 5. Audit Conclusion

The raw datasets were **relationship-complete at the canonical ID
level**, but they were not analysis-ready.

The main quality risks were concentrated in:

-   duplicate and duplicate-person handling in `Leads`;
-   inconsistent lead contact, date, geography, course/manager text,
    source and campaign values;
-   duplicate and financially inconsistent `Payments`;
-   duplicate and incomplete `MarketingSpend` records;
-   enrollment chronology exceptions;
-   invalid activity metrics, weekly-period inconsistencies and missing
    activity periods;
-   incomplete date-currency coverage in `ExchangeRates`.

The reference datasets `Managers`, `Courses` and `Cohorts` were
structurally strong and suitable to serve as canonical lookup sources.

The proposed decisions follow a cautious data-treatment principle:
**correct or map only when deterministic evidence exists; remove exact
duplicates; preserve legitimate missing values; flag unresolved
anomalies; and retain raw values where auditability is important.**
