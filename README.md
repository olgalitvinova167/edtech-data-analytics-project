# Prog Academy --- EdTech Data Analytics Final Project

## 1. Project Overview

This project is an end-to-end EdTech data analytics solution developed
for Prog Academy. It combines CRM leads, enrollments, payments,
marketing activity, student learning activity, courses, managers,
cohorts, and exchange rates.

The objective was to transform messy operational data into reliable
analytical datasets, build a structured Power BI semantic model, create
business KPIs, and deliver an interactive six-page report that supports
marketing, sales, course, student, financial, and operational analysis.

## 2. Business Questions

The project is designed to answer questions such as:

-   How effectively are leads converted into enrollments?
-   Which marketing sources and campaigns show the strongest balance
    between acquisition cost and downstream lead quality?
-   How much gross and net revenue is generated, and how does it compare
    with recorded marketing spend?
-   How do sales managers differ in conversion, response time, revenue,
    and performance against target?
-   Which courses perform best in enrollment, revenue, completion,
    dropout, engagement, and refunds?
-   What patterns are visible in student completion, dropout,
    engagement, and repeat enrollment?
-   Where are refunds, data-quality exceptions, or other operational
    issues concentrated?

## 3. Data Sources

The project uses nine operational datasets. Canonical IDs such as
`LeadID`, `CourseID`, `ManagerID`, `EnrollmentID`, `StudentID`, and
`CohortID` are used as relationship keys rather than descriptive text
fields.

  -----------------------------------------------------------------------
  Dataset                             Grain / Business Role
  ----------------------------------- -----------------------------------
  `Managers`                          One row per sales manager; manager
                                      profile, employment dates, and
                                      sales targets

  `Courses`                           One row per course; canonical
                                      course information and pricing

  `Leads`                             One row per CRM lead after
                                      cleaning; lead source, campaign,
                                      contact, status, attribution, and
                                      sales information

  `MarketingSpend`                    Daily × source × campaign × country
                                      × device; advertising spend,
                                      impressions, and clicks

  `Payments`                          One row per payment transaction;
                                      revenue, refunds, installments,
                                      payment status, and currency
                                      information

  `Enrollments`                       One row per enrollment; student,
                                      course, cohort, enrollment status,
                                      and completion information

  `StudentActivity`                   One row per
                                      student--enrollment--week; LMS
                                      engagement and learning activity

  `ExchangeRates`                     Daily × currency; FX rates used to
                                      convert financial values to UAH

  `Cohorts`                           One row per cohort; course
                                      schedule, capacity, and
                                      cohort-level information
  -----------------------------------------------------------------------

## 4. Data Cleaning and Preparation

Data preparation was completed in Python using Pandas and is documented
in `EdTech_Data_Cleaning.ipynb`.

The main cleaning steps included:

-   removing exact duplicates and resolving repeated business keys
    conservatively;
-   standardizing course, manager, geography, source, campaign, status,
    and other categorical values;
-   parsing mixed date formats and validating chronology;
-   normalizing and validating email addresses and phone numbers;
-   standardizing currencies and validating payment, refund,
    installment, and FX logic;
-   creating data-quality and analytical eligibility flags for
    unresolved or uncertain records;
-   validating final relationships and exporting the cleaned datasets
    for Power BI.

The raw source files were preserved. Uncertain but potentially
legitimate records were not deleted automatically; they were corrected,
mapped, flagged, converted to null, or preserved according to the
available evidence.

## 5. Final Cleaned Data

The final datasets exported from the Python cleaning workflow contain:

  Dataset                           Final Rows
  ------------------------------- ------------
  `Managers_Cleaned.csv`                     8
  `Courses_Cleaned.csv`                     17
  `Leads_Cleaned.csv`                   51,021
  `MarketingSpend_Cleaned.csv`          13,889
  `Payments_Cleaned.csv`                17,100
  `Enrollments_Cleaned.csv`             12,161
  `StudentActivity_Cleaned.csv`        111,074
  `ExchangeRates_Cleaned.csv`            3,644
  `Cohorts_Cleaned.csv`                    242

The final datasets have unique required primary or natural keys and no
orphan records across the validated canonical relationships.

## 6. Power BI Data Model

The final `EdTech_BI_Project.pbix` uses a star-schema approach with
dimensions surrounding the main analytical fact tables.

**Main dimensions:**

-   `DimDate`
-   `DimCourse`
-   `DimManager`
-   `DimStudent`
-   `DimMarketingSource`
-   `DimCampaign`
-   `DimGeography`
-   `DimCohort`

**Main fact tables:**

-   `FactLeads`
-   `FactPayments`
-   `FactEnrollments`
-   `FactMarketingSpend`
-   `FactStudentActivity`

Canonical IDs are used as relationship keys, and relationships are
primarily configured to filter from dimensions to facts in one
direction. Technical fields are separated from the report-facing
analytical fields where appropriate.

The main source-level marketing analysis uses the normalized source
context through `DimMarketingSource`. First-touch and last-touch source
information is retained separately for attribution comparison. These
attribution views describe how leads are assigned under different
attribution definitions and should not be interpreted as proof that a
marketing source caused a conversion.

## 7. Key Measures

The final Power BI semantic model contains **62 DAX measures**,
including core business KPIs and supporting/helper measures.

The main measure groups are:

-   **Lead & Conversion** --- leads, unique leads, converted leads,
    enrollments, conversion rates, funnel metrics, and attribution
    measures;
-   **Revenue & Payments** --- Gross Revenue, Net Revenue, Refund
    Amount, Refund Rate, and Average Order Value;
-   **Marketing** --- Marketing Spend, Cost per Lead, Customer
    Acquisition Cost, ROAS, Marketing ROI, CTR, CPC, and allocated
    marketing measures;
-   **Students & Enrollments** --- Active Students, Completion Rate,
    Dropout Rate, Returning Student Rate, and engagement measures;
-   **Time Intelligence** --- revenue MoM/YoY and lead-conversion
    comparison measures;
-   **Manager & Course Performance** --- response time, manager
    conversion, revenue by manager/course, Sales Target, and Target
    Attainment.

Detailed definitions and DAX formulas are documented in
`Prog_Academy_DAX_Documentation.md`.

## 8. Power BI Report Pages

The final Power BI report contains six pages:

1.  **Executive Summary** --- provides a high-level view of financial
    performance, marketing efficiency, leads, enrollments, conversion,
    students, and major trends.
2.  **Marketing Analytics** --- analyzes spend, campaigns, sources,
    impressions, clicks, acquisition efficiency, attribution, and
    marketing performance.
3.  **Sales Performance** --- compares managers using lead volume,
    conversion, response time, revenue, AOV, funnel performance, and
    sales targets.
4.  **Course Analytics** --- analyzes course enrollment, conversion,
    revenue, refunds, completion, engagement, and cohort performance.
5.  **Student Analytics** --- focuses on active students, completion,
    dropout, returning students, engagement, and LMS activity patterns.
6.  **Operational Details** --- supports detailed record-level
    investigation, filtering, exception review, and drill-through across
    operational data.

## 9. Main Business Findings

The final analysis highlights several business themes:

-   The portfolio shows a high observed revenue-to-marketing-spend
    ratio, while ROAS and ROI remain observational measures rather than
    proof of marketing causality.
-   Major marketing sources show different trade-offs between Cost per
    Lead and downstream lead quality, supporting controlled testing
    rather than budget decisions based on CPL alone.
-   In selected manager portfolios, shorter average response times
    coincide with stronger conversion, providing a useful pattern for
    further operational monitoring.
-   Free introductory courses have the highest lead-to-enrollment
    conversion rates in the course analysis but also high dropout and no
    direct Gross Revenue, making progression into paid learning an
    important measure of their value.
-   Completion, dropout, and repeat-enrollment results identify student
    retention and post-course re-engagement as areas for further
    analysis and testing.
-   Refund exposure is higher in selected paid courses, supporting
    deeper investigation of refund reasons, cohorts, payment plans,
    completion, and engagement before making pricing or marketing
    changes.

The complete findings, supporting metrics, recommendations, and
limitations are documented in `Prog_Academy_Business_Findings.md`.

## 10. Key Assumptions and Limitations

-   Marketing attribution is observational, not causal; normalized
    source, first-touch, and last-touch views represent different
    analytical perspectives.
-   Unresolved data-quality exceptions are retained or flagged where a
    safe correction is not supported by the source data.
-   `Allocated Marketing Spend` is an analytical allocation used for
    course-level analysis and is not directly observed course-level
    advertising spend.
-   `Returning Student Rate` measures repeat enrollment within the
    available observation window and is not a time-adjusted
    cohort-retention measure.
-   The final model treats `MonthlySalesTarget` as a gross-revenue
    target. Financial KPI eligibility follows the implemented
    successful-payment-status rule, including 58 successful test
    payments; excluding test payments did not materially change the main
    financial conclusions.

## 11. Submission Files

### Core project files

-   `EdTech_BI_Project.pbix`
-   `EdTech_Data_Cleaning.ipynb`
-   `Managers_Cleaned.csv`
-   `Courses_Cleaned.csv`
-   `Leads_Cleaned.csv`
-   `MarketingSpend_Cleaned.csv`
-   `Payments_Cleaned.csv`
-   `Enrollments_Cleaned.csv`
-   `StudentActivity_Cleaned.csv`
-   `ExchangeRates_Cleaned.csv`
-   `Cohorts_Cleaned.csv`

### Documentation

-   `README.md`
-   `Prog_Academy_Data_Quality_Report.md`
-   `Prog_Academy_Before_After_Data_Quality_Summary.md`
-   `Prog_Academy_DAX_Documentation.md`
-   `Prog_Academy_Business_Findings.md`

## 12. How to Review the Project

1.  Open `EdTech_Data_Cleaning.ipynb` to review the Python/Pandas
    cleaning workflow and validation steps.
2.  Review the nine cleaned CSV files that provide the final analytical
    data used by Power BI.
3.  Open `EdTech_BI_Project.pbix` in Power BI Desktop.
4.  Explore the six report pages using the available slicers, filters,
    and navigation.
5.  Use **Operational Details** for record-level investigation and
    drill-through.
6.  Review the supporting Markdown documents for detailed data-quality
    evidence, DAX definitions, cleaning outcomes, and business findings.
