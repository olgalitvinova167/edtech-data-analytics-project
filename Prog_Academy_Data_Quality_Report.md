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

  ---------------------------------------------------------------------------------------
  Dataset                Actual Raw Rows Data Dictionary Rows Expected Grain
  ----------------- -------------------- -------------------- ---------------------------
  Managers                             8                    8 One row per sales manager

  Courses                             17                   17 One row per course

  Leads                           51,779               50,000 One row per CRM lead

  MarketingSpend                  13,968               13,889 Daily × source × campaign ×
                                                              country × device

  Payments                        17,174               17,100 One row per payment
                                                              transaction

  Enrollments                     12,161               12,161 One row per enrollment

  StudentActivity                111,641              112,119 One row per
                                                              student--enrollment--week

  ExchangeRates                    3,644                3,644 Daily × currency

  Cohorts                            242                  242 One row per cohort
  ---------------------------------------------------------------------------------------

The audit uses the actual raw file counts rather than assuming the
overview counts are exact. The differences in `Leads`, `MarketingSpend`,
`Payments` and `StudentActivity` were therefore treated as items to
investigate rather than silently reconciled.

------------------------------------------------------------------------

## 2. Granularity, Keys and Relationship Validation

### 2.1 Dataset Keys

  ---------------------------------------------------------------------------------------------------
  Dataset                 Primary / Natural Key                               Raw-Key Result
  ----------------------- --------------------------------------------------- -----------------------
  Managers                `ManagerID`                                         Unique; no exact
                                                                              duplicates

  Courses                 `CourseID`                                          Unique; no exact
                                                                              duplicates

  Leads                   `LeadID`                                            Not unique in raw data:
                                                                              758 excess duplicate-ID
                                                                              rows across 750
                                                                              duplicate-ID groups

  MarketingSpend          `Date + Source + Campaign + Country + DeviceType`   79 duplicate-grain
                                                                              rows; these are also
                                                                              exact duplicates

  Payments                `PaymentID`                                         74 excess duplicate-ID
                                                                              rows; all are exact
                                                                              duplicate transactions

  Enrollments             `EnrollmentID`                                      Unique; no exact
                                                                              duplicates

  StudentActivity         `ActivityID`; natural grain                         567 duplicate
                          `StudentID + EnrollmentID + ActivityWeek`           `ActivityID` rows and
                                                                              567 duplicate
                                                                              natural-grain rows; all
                                                                              are exact duplicates

  ExchangeRates           `Date + Currency`                                   Unique

  Cohorts                 `CohortID`                                          Unique
  ---------------------------------------------------------------------------------------------------

For `Leads`, 279 of the duplicate rows were exact copies. After
separating exact copies for audit purposes, **479 non-exact repeated
`LeadID` rows** remained and differed only in descriptive/raw fields.

### 2.2 Foreign-Key Relationships

The raw canonical IDs were checked against the reference tables defined
in the Data Dictionary.

  -----------------------------------------------------------------------
  Relationship Group                  Raw Validation
  ----------------------------------- -----------------------------------
  `Managers.ManagerID` → Leads /      0 orphan foreign keys
  Enrollments / Payments              

  `Courses.CourseID` → Leads /        0 orphan foreign keys
  Cohorts / Enrollments / Payments /  
  StudentActivity                     

  `Leads.LeadID` → Enrollments /      0 orphan foreign keys
  Payments                            

  `Cohorts.CohortID` → Enrollments    0 orphan foreign keys

  `Enrollments.EnrollmentID` →        0 orphan foreign keys
  Payments / StudentActivity          

  `StudentID` consistency between     0 enrollment-to-student mismatches
  Enrollments and related Payments /  
  StudentActivity                     
  -----------------------------------------------------------------------

The main relationship problem was therefore **duplicate parent/fact
keys**, not missing canonical foreign keys.

------------------------------------------------------------------------

## 3. Missing-Value Profile

Columns not listed below had **0 missing values** under the raw Pandas
import used by the project.

  -------------------------------------------------------------------------
  Dataset                 Columns with Missing      Interpretation
                          Values                    
  ----------------------- ------------------------- -----------------------
  Managers                `TerminationDate` 5       Expected for the 5
                                                    active managers; not
                                                    inherently a quality
                                                    issue

  Courses                 None                      No missing values

  Leads                   `FirstContactAt` 3,186;   Mix of legitimate
                          `ConvertedAt` 38,824;     optional fields and
                          `FullName` 1,613; `Phone` data-quality gaps;
                          1,510; `Region` 385;      field-specific
                          `City` 1,953;             treatment required
                          `PreferredLanguage` 405;  
                          `LeadScore` 1,570;        
                          `Campaign` 11,630;        
                          `Content` 11,405; `Term`  
                          23,912; `ReferrerDomain`  
                          7,491; `LostReason`       
                          21,364                    

  MarketingSpend          `Campaign` 2,938; `Spend` Missing campaign may be
                          106                       legitimate; missing
                                                    spend requires
                                                    validation against
                                                    available base-currency
                                                    evidence

  Payments                `PaymentDate` 75;         Refund fields are
                          `RefundDate` 16,239;      conditional; missing
                          `RefundReason` 16,239;    `PaymentDate` and
                          `InvoiceNumber` 24        invoice values require
                                                    explicit treatment

  Enrollments             `ActualCompletionDate`    Primarily
                          6,520; `CancellationDate` status-dependent
                          11,129;                   fields; not valid to
                          `CancellationReason`      impute universally
                          11,129;                   
                          `CertificateIssuedDate`   
                          9,024                     

  StudentActivity         None                      No cell-level nulls,
                                                    but weekly sequence
                                                    gaps exist

  ExchangeRates           None                      No null cells; coverage
                                                    gaps exist at the
                                                    date-currency level

  Cohorts                 None                      No missing values
  -------------------------------------------------------------------------

------------------------------------------------------------------------

## 4. Detailed Data Quality Findings and Proposed Decisions

### 4.1 Managers, Courses, Cohorts and Exchange Rates

  -----------------------------------------------------------------------------------
  Dataset           Observed Issue /   Verified Evidence and      Proposed Cleaning
                    Finding            Example                    Decision
  ----------------- ------------------ -------------------------- -------------------
  Managers          Nullable           5 missing                  Preserve null
                    termination dates  `TerminationDate` values,  `TerminationDate`
                                       all belonging to active    for active
                                       managers; no duplicate     managers; parse
                                       `ManagerID`, invalid       employment dates
                                       salary/bonus/target        and retain existing
                                       values,                    canonical manager
                                       employment-status/date     records
                                       conflicts or reversed      
                                       employment dates were      
                                       found                      

  Courses           No material        17 unique `CourseID`       Treat `Courses` as
                    source-quality     values; no nulls, exact    the canonical
                    issue identified   duplicates, invalid        course reference;
                                       numeric rules or non-UAH   validate types and
                                       `BaseCurrency` values      retain records

  Cohorts           No material        242 unique `CohortID`      Retain as the
                    key/date/numeric   values; no nulls, exact    canonical cohort
                    issue identified   duplicates, invalid        reference after
                                       `CourseID`, reversed       type validation
                                       start/end dates or invalid 
                                       seat/enrollment values     

  ExchangeRates     Incomplete daily   11 missing date-currency   Preserve the
                    currency coverage  combinations between the   coverage gaps
                                       minimum and maximum rate   rather than invent
                                       dates;                     exchange rates; use
                                       e.g. `2024-01-20 / CZK`,   exact lookup where
                                       `2024-04-11 / EUR`,        available and
                                       `2024-08-18 / USD`,        document any
                                       `2025-08-26 / USD`         justified fallback
                                                                  source
  -----------------------------------------------------------------------------------

### 4.2 Leads

`Leads.csv` contained the broadest set of data-quality issues and
required both record-level deduplication and field-level
standardisation.

  ------------------------------------------------------------------------------------------
  Observed Issue          Verified Evidence and Example            Proposed Cleaning
                                                                   Decision
  ----------------------- ---------------------------------------- -------------------------
  Exact and partial       51,779 raw rows but 51,021 unique        Remove exact copies;
  duplicate leads         `LeadID` values. There were 758 excess   consolidate repeated
                          duplicate-ID rows across 750 groups,     `LeadID` rows only where
                          including 279 exact duplicate excess     differences are
                          rows. After separating exact copies, 479 deterministic descriptive
                          non-exact repeated-ID rows remained.     variants, using the
                          Example: `LEAD0000064` appears with      canonical ID-based values
                          `CourseNameRaw` values `IT Start (Free)` 
                          / `It Start (Free)`, `Source` `meta` /   
                          `META`, and `City` `Pr@gue` / `Pr@Gue`   

  Duplicate-person        1,033 raw rows were flagged              Do not delete legitimate
  candidates              `IsDuplicateCandidate=True`,             CRM LeadIDs; resolve
                          representing 1,021 unique LeadIDs.       person identity using
                          Identity checks confirmed the 1,021      normalized contact/stable
                          candidates have stable-field             fields and retain a
                          counterparts after LeadID deduplication  separate person-level key
                                                                   for unique-person
                                                                   analysis

  Course-name variants    6,442 raw `CourseNameRaw` values did not Use `CourseID` as the
                          exactly match the canonical `CourseName` authority and map the
                          for the same `CourseID`. Example:        analytical course name to
                          `CRS0005` `Fronted-end - online` vs      `Courses.CourseName`;
                          canonical `Front-End`                    preserve the raw text for
                                                                   auditability

  Manager-name variants   2,740 raw `ManagerNameRaw` values did    Use `ManagerID` as the
                          not exactly match the canonical manager  authority and map to the
                          name for the same `ManagerID`. Example:  canonical manager name;
                          `MGR0002` `Natalia H.` vs canonical      retain the raw field
                          `Natalia H.`                             

  Email formatting and    4,754 raw email strings changed under    Trim and lowercase valid
  invalid values          trim/lowercase normalization. 776 failed addresses; validate
                          the project format checks: 295 without   format; do not invent
                          `@`, 170 with multiple `@`, 145 with     replacements for invalid
                          internal whitespace and 171 with an      addresses; preserve raw
                          invalid domain pattern. Examples:        email and expose validity
                          `jessica.барабаш6844.outlook.com`,       
                          `твердислав.wojciech2211@@icloud.com`,   
                          `meghan.nehring4122 @ icloud.com`        

  Phone formatting and    1,510 blank phones; 1,024                Normalize to E.164 only
  invalid values          invalid/suspicious phones; 6,839         when country/number
                          plausible but non-E.164 values; 42,406   evidence is
                          already E.164-like. Examples: `call me`  deterministic; retain
                          and `380540633447`                       invalid or unresolved raw
                                                                   values rather than
                                                                   guessing

  Country / city / region `Country` contained 120 distinct raw     Normalize country, city
  inconsistency           values although the canonical marketing  and region using
                          country domain contains 7 countries.     deterministic
                          2,490 rows used non-canonical country    synonym/transliteration
                          values such as `UA`, `UKRAINE`, `PL`,    mappings and cross-field
                          `DE`, `POLAND`. `City` had 1,953 missing evidence; convert
                          values plus 355 explicit placeholders;   placeholders to null;
                          `Region` had 385 missing values plus 394 preserve unresolved
                          placeholders. Raw city variants included values rather than
                          `Kiev`, `Odessa`, `Lvov`, `Praha` and    fabricating geography
                          typo/noise forms                         

  Mixed and malformed     Across `CreatedAt`, `UpdatedAt`,         Parse with field-specific
  dates                   `FirstContactAt` and `ConvertedAt`,      supported formats; coerce
                          1,402 nonblank raw date cells were       truly malformed values to
                          malformed, affecting 1,383 raw rows.     null; retain original raw
                          Examples include `2024-13-01`,           timestamps and flag
                          `2024-01-32`, `0000-00-00`, `??`; valid  malformed/date-order
                          dates also appeared in multiple formats  exceptions

  Non-standard lead       544 raw `LeadStatus` rows were outside   Normalize casing and
  statuses and            the Data Dictionary's exact `Won` /      known aliases
  categorical casing      `Lost` / `In Progress` values, including deterministically; do not
                          `WON`, `won`, `lost`, `Open`, `closed`.  force ambiguous business
                          Similar casing variation existed in      statuses into another
                          `LeadTemperature`, `DeviceType` and      status without evidence
                          `PreferredLanguage`                      

  Source aliases and      `Source` had 51 raw spellings,           Apply deterministic
  casing                  collapsing to 18 values after            source alias mapping;
                          trim/lowercase. 589 rows still used      keep `fb-insta` distinct
                          alias forms outside the canonical source where the source data
                          list, including `fb`, `facebook`,        supports separate
                          `facebook_ads`, `www.google.com` and     attribution granularity
                          `www.google.com.ua`                      

  Campaign placeholders   11,630 raw `Campaign` values were        Trim/lowercase for
  and inconsistent text   missing and a further 363 contained      grouping, convert
                          explicit placeholders such as `unknown`, explicit placeholders to
                          `?`, `тест`, `--` or `-`; 163 non-null   null, and map only known
                          raw campaign strings were present        deterministic campaign
                                                                   variants; preserve
                                                                   genuine missing campaign
                                                                   values

  Invalid expected course 247 raw rows had invalid/sentinel        Preserve the raw price,
  prices                  prices: 78 negative values and 169       expose a clean analytical
                          extreme values using `500000`, `9999999` price only when valid,
                          or `12345678`. Example: `LEAD0000330` =  and flag invalid values
                          `-30810 UAH`                             rather than replacing
                                                                   them with an assumed
                                                                   course price
  ------------------------------------------------------------------------------------------

### 4.3 Marketing Spend

  ------------------------------------------------------------------------------------------------------------------
  Observed Issue          Verified Evidence and Example                                      Proposed Cleaning
                                                                                             Decision
  ----------------------- ------------------------------------------------------------------ -----------------------
  Duplicate daily-grain   79 excess rows were exact duplicates and also duplicated the       Remove exact duplicate
  records                 defined `Date + Source + Campaign + Country + DeviceType` grain.   rows before aggregation
                          Example duplicate:                                                 
                          `2024-04-28 / google / Search - QA - Ukraine / Ukraine / Tablet`   

  Missing spend           106 raw rows had missing `Spend`; 81 of them had a positive        Recover `Spend` only
                          `SpendBaseCurrency` value. Example:                                when supported
                          `2024-09-06 / google / Search - QA - Ukraine` had missing `Spend`  deterministically by
                          and `SpendBaseCurrency = 6153.22`                                  available base-currency
                                                                                             information; otherwise
                                                                                             preserve the missing
                                                                                             value

  Spend/base-currency     13 UAH rows had `Spend != SpendBaseCurrency`: 9 had a non-zero     Correct only where the
  inconsistency           base amount and 4 had `SpendBaseCurrency = 0`. Example:            source relationship is
                          `2025-04-16` had `Spend = 72048` and `SpendBaseCurrency = 2881.92` deterministic; retain
                                                                                             unresolved rows with an
                                                                                             explicit exception flag

  Campaign naming         162 distinct non-null raw campaign strings were present; 14        Standardize known
  inconsistency           case-insensitive campaign families had multiple spellings, with    campaign variants to
                          additional typo/copy variants such as `AI_COURSE_V1`,              canonical campaign
                          `Search - Brand - Ukraine_copy` and `Seach - Brand - Ukraine`      names before grouping;
                                                                                             preserve the raw
                                                                                             campaign value

  Clicks greater than     16 raw rows violated `Clicks <= Impressions`. Example:             Treat as an explicit
  impressions             `2024-07-12 / Search - QA - Ukraine` had 2,405 impressions and     source-data anomaly; do
                          9,100 clicks                                                       not invent corrected
                                                                                             click or impression
                                                                                             values without evidence

  Statistical outlier     Raw IQR profiling identified 590 `Spend` and 1,360 `Impressions`   Review as possible
  candidates              outlier candidates                                                 anomalies but do not
                                                                                             automatically remove or
                                                                                             cap them because an
                                                                                             outlier is not
                                                                                             necessarily invalid
  ------------------------------------------------------------------------------------------------------------------

### 4.4 Payments

  ---------------------------------------------------------------------------------------------------------------------------
  Observed Issue          Verified Evidence and Example                                               Proposed Cleaning
                                                                                                      Decision
  ----------------------- --------------------------------------------------------------------------- -----------------------
  Duplicate transactions  74 excess `PaymentID` rows; all were exact duplicates. Example:             Remove exact duplicate
                          `PAY0000174` appeared twice with the same enrollment, date, amount and      transaction copies
                          status                                                                      

  Missing payment dates   75 unique payment records had missing `PaymentDate`                         Preserve when no
                                                                                                      deterministic
                                                                                                      transaction date is
                                                                                                      available; prevent
                                                                                                      unsupported date-based
                                                                                                      assumptions and
                                                                                                      document any FX
                                                                                                      fallback

  Non-canonical currency  72 raw rows, representing 71 unique PaymentIDs, used currencies outside the Normalize only where
  values                  ExchangeRates domain. Examples: `грн`, `UAH`, `usd`, `Dollar`, `EURO`,      exchange-rate or
                          `XxX`                                                                       transaction evidence
                                                                                                      identifies the intended
                                                                                                      currency; avoid blind
                                                                                                      text replacement for
                                                                                                      ambiguous codes

  Negative gross amounts  16 unique payments had `GrossAmount < 0`. Example: `PAY0000518` had         Correct sign only when
                          `GrossAmount = -12533.33`                                                   the other financial
                                                                                                      fields provide
                                                                                                      consistent
                                                                                                      deterministic evidence

  Refund / gross          74 raw rows initially satisfied `RefundAmount > GrossAmount`; this total    Reconcile refund values
  inconsistency           included records affected by negative gross signs. After isolating those    against gross,
                          sign anomalies, 58 refund-specific conflicts remained                       discount, processing
                                                                                                      fee, net amount and
                                                                                                      refund metadata;
                                                                                                      correct deterministic
                                                                                                      arithmetic cases and
                                                                                                      flag unresolved status
                                                                                                      inconsistencies

  Net amount arithmetic   80 deduplicated raw payment records did not satisfy                         Recalculate or correct
  inconsistency           `NetAmount = GrossAmount - DiscountAmount - ProcessingFee - RefundAmount`   only where the
                          before correction analysis                                                  component fields
                                                                                                      demonstrate the
                                                                                                      intended value;
                                                                                                      otherwise preserve and
                                                                                                      flag

  Installment rule        112 unique payments had `InstallmentNumber > InstallmentCount`. Example:    Correct installment
  violations              `PAY0000466` had installment 8 of 6                                         sequencing only when
                                                                                                      payment type,
                                                                                                      enrollment history and
                                                                                                      transaction order
                                                                                                      provide deterministic
                                                                                                      evidence

  Invoice quality         24 invoices were missing and 131 non-missing values failed the raw          Normalize
                          `INV-YYYY-######` pattern, for 155 invoice-quality exceptions in total.     trim/case/spacing where
                          Examples: `INV 2024 000509`, `INV-INV-000621`                               mechanically
                                                                                                      recoverable; retain
                                                                                                      missing or genuinely
                                                                                                      malformed invoice
                                                                                                      values as explicit
                                                                                                      exceptions

  Test payments           68 deduplicated payment records were marked `IsTestPayment=True`            Preserve test records
                                                                                                      for audit traceability;
                                                                                                      handle analytical
                                                                                                      eligibility separately
                                                                                                      from physical data
                                                                                                      cleaning
  ---------------------------------------------------------------------------------------------------------------------------

### 4.5 Enrollments

  ------------------------------------------------------------------------------------------
  Observed Issue /        Verified Evidence and Example              Proposed Cleaning
  Finding                                                            Decision
  ----------------------- ------------------------------------------ -----------------------
  Repeated students are   `EnrollmentID` is fully unique. 474        Preserve repeated
  legitimate, not         students had more than one enrollment,     student enrollments
  duplicate enrollments   representing 485 additional enrollments    because the dataset
                          beyond one per student; maximum was 3      grain is one row per
                          enrollments                                enrollment

  Late enrollment         745 enrollments had                        Preserve as a business
                          `EnrollmentDate > CourseStartDate`.        exception and expose a
                          Example: `ENR010913` enrolled `2025-11-13` `LateEnrollment` flag
                          for a course starting `2025-11-08`         rather than deleting
                                                                     the record

  Stronger chronology     35 enrollments occurred after              Preserve and flag
  exceptions              `ExpectedEndDate`; 5 had                   chronology exceptions;
                          `ActualCompletionDate < EnrollmentDate`.   do not invent corrected
                          Example: `ENR011920` enrolled `2026-01-01` dates without
                          although expected end was `2025-12-30`     supporting source
                                                                     evidence

  Conditional missing     Completion, cancellation and certificate   Validate against
  dates                   dates have high null counts but are        `EnrollmentStatus` and
                          status-dependent                           related fields; do not
                                                                     impute dates simply
                                                                     because they are null
  ------------------------------------------------------------------------------------------

No duplicate `EnrollmentID`, orphan foreign keys, invalid agreed
prices/discount percentages, reversed course start/end dates, or basic
cohort-key inconsistencies were identified in the raw enrollment
structure.

### 4.6 Student Activity

  --------------------------------------------------------------------------------------------
  Observed Issue          Verified Evidence and Example                Proposed Cleaning
                                                                       Decision
  ----------------------- -------------------------------------------- -----------------------
  Duplicate activity rows 567 exact duplicate excess rows; the same    Remove exact duplicate
                          567 rows duplicated both `ActivityID` and    copies before activity
                          the natural                                  analysis
                          `StudentID + EnrollmentID + ActivityWeek`    
                          grain                                        

  Negative activity       464 raw rows contained at least one negative Treat invalid negative
  metrics                 value in `Logins`, `LessonsViewed`,          metrics as
                          `VideoMinutesWatched` or `PlatformHours`.    missing/invalid rather
                          Example: `ACT00000146` had                   than legitimate
                          `VideoMinutesWatched = -259`                 activity; preserve the
                                                                       record

  Lessons completed       126 raw rows violated                        Resolve the invalid
  greater than lessons    `LessonsCompleted <= LessonsViewed`, often   source metric first and
  viewed                  because `LessonsViewed` itself was negative  prevent impossible
                                                                       lesson completion
                                                                       relationships

  Invalid quiz            631 raw rows had `AverageQuizScore > 100`;   Mark invalid quiz
  percentages             no scores below 0 were found. Example:       scores as missing
                          `ACT00000147` had `137.3`                    rather than cap them to
                                                                       100, because the
                                                                       intended score cannot
                                                                       be inferred

  Homework inconsistency  645 raw rows had                             Preserve the source
                          `HomeworkAccepted > HomeworkSubmitted`.      values and expose an
                          Example: `ACT00000014` had 1 submitted and 4 exception flag unless a
                          accepted                                     deterministic
                                                                       correction exists

  `LastActivityAt`        362 rows had                                 Preserve the timestamp
  outside the activity    `LastActivityAt >= ActivityWeek + 7 days`.   and flag the anomaly
  period                  Example: `ACT00000107` had                   rather than invent a
                          `ActivityWeek = 2024-05-15` and              replacement date
                          `LastActivityAt = 2026-12-14 11:44:13`       

  `ActivityWeek` does not 97,275 raw rows had an `ActivityWeek`        Do not shift dates
  consistently represent  weekday other than Monday, despite the Data  without evidence.
  Monday week-start       Dictionary description "Week start (Mon)"    Preserve the supplied
                                                                       weekly anchor and use
                                                                       interval-based
                                                                       validation; document
                                                                       the semantic
                                                                       discrepancy

  Missing weekly          After removing exact duplicate copies solely Do not fabricate
  observations            to avoid distorting sequence analysis, 778   zero-activity rows;
                          enrollments had 817 internal jumps longer    retain observed records
                          than 7 days, representing 827 directly       and flag/count internal
                          detectable missing weekly periods            gaps
  --------------------------------------------------------------------------------------------

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
