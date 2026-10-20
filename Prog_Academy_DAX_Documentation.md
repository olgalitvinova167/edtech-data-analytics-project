# Prog Academy — DAX Documentation

## 1\. Purpose

This document provides concise documentation of the DAX measures in the final `EdTech_BI_Project.pbix`. The measure overview lists **62 DAX measures**, including the newly added `Average Response Time Coverage %`, exceeding the assignment requirement of at least 20\. Section 2 documents these measures; Section 3 provides the DAX definitions for 26 main business and KPI measures used across the report.&nbsp;

## 2\. Measure Overview

| Measure | Category | Short description |
| :---- | :---- | :---- |
| `Total Leads` | Leads & Conversion | Analytical lead count excluding resolved duplicate-person candidate rows. |
| `Unique Leads` | Leads & Conversion | Distinct people based on the contact-resolved PersonKey (43,859 in the unfiltered updated report). |
| `Conversion Rate` | Leads & Conversion | Enrollments divided by Total Leads. |
| `First Touch Leads` | Leads & Conversion | Analytical leads attributed by first-touch source. |
| `Last Touch Leads` | Leads & Conversion | Analytical leads attributed by last-touch source. |
| `Won Leads` | Leads & Conversion | Analytical leads with normalized status Won. |
| `Sales Funnel Value` | Leads & Conversion | Stage selector for the strict sales funnel. |
| `Enrolled Won Leads` | Leads & Conversion | Won leads that eventually have a matching enrollment. |
| `Gross Revenue` | Revenue & Payments | Gross UAH revenue from financially eligible payments. |
| `Net Revenue` | Revenue & Payments | Net UAH revenue from financially eligible payments. |
| `Refund Amount` | Revenue & Payments | Refund value in UAH using the applied FX rate. |
| `Refund Rate` | Revenue & Payments | Refund Amount divided by Gross Revenue. |
| `Average Order Value` | Revenue & Payments | Gross Revenue per distinct eligible paying enrollment. |
| `Marketing Spend` | Marketing | Total base-currency marketing spend. |
| `Cost per Lead` | Marketing | Marketing Spend per analytical lead. |
| `Customer Acquisition Cost` | Marketing | Marketing Spend per distinct eligible paying student. |
| `Return on Ad Spend` | Marketing | Gross Revenue divided by Marketing Spend. |
| `Marketing ROI` | Marketing | Net Revenue less Marketing Spend, divided by Marketing Spend. |
| `Allocated Marketing Spend` | Marketing | Lead-proportional allocation of spend into course context. |
| `Allocated Return on Ad Spend` | Marketing | Gross Revenue divided by Allocated Marketing Spend. |
| `Impressions` | Marketing | Total ad impressions. |
| `Clicks` | Marketing | Total ad clicks. |
| `CTR` | Marketing | Clicks divided by Impressions. |
| `Cost per Click` | Marketing | Marketing Spend divided by Clicks. |
| `Average Campaign Spend` | Marketing | Average spend across selected paid campaigns. |
| `Average Campaign Revenue` | Marketing | Average gross revenue across selected paid campaigns. |
| `Marketing Observation Count` | Marketing | Count of marketing-spend fact rows, returning zero for blank. |
| `Marketing Spend Display` | Marketing | Zero-safe display version of Marketing Spend. |
| `Impressions Display` | Marketing | Zero-safe display version of Impressions. |
| `Clicks Display` | Marketing | Zero-safe display version of Clicks. |
| `CTR Display` | Marketing | Zero-safe display version of CTR. |
| `Cost per Click Display` | Marketing | Zero-safe display version of Cost per Click. |
| `Cost per Lead Display` | Marketing | Zero-safe display version of Cost per Lead. |
| `Enrollments` | Students & Enrollments | Count of enrollment records. |
| `Average Days to Enrollment` | Students & Enrollments | Average days from lead creation to enrollment. |
| `Active Students` | Students & Enrollments | Distinct students with Active enrollment status. |
| `Completion Rate` | Students & Enrollments | Completed enrollments among mature enrollments with ExpectedEndDate on or before 26 February 2026\. |
| `Returning Student Rate` | Students & Enrollments | Students with more than one enrollment divided by all students. |
| `Average Weekly Platform Hours` | Students & Enrollments | Average valid PlatformHours across observed activity rows. |
| `Dropout Rate` | Students & Enrollments | Dropped enrollments divided by all enrollments. |
| `Students by Engagement Segment` | Students & Enrollments | Student count within selected engagement-hours segment. |
| `Platform Users` | Students & Enrollments | Distinct students with observed LMS activity. |
| `Students Enrolled` | Students & Enrollments | Distinct students represented in enrollments. |
| `Lead Conversion vs Previous Period` | Time Intelligence | Percentage-point change in Conversion Rate versus previous month. |
| `Revenue MoM %` | Time Intelligence | Net Revenue change versus previous month. |
| `Revenue YoY %` | Time Intelligence | Net Revenue change versus same period last year. |
| `Revenue MoM Comparison Label` | Time Intelligence | Dynamic previous-month comparison label. |
| `Revenue YoY Comparison Label` | Time Intelligence | Dynamic previous-year comparison label. |
| `Lead Conversion Comparison Label` | Time Intelligence | Dynamic previous-month conversion label. |
| `Average Response Time` | Manager & Course | Average hours from lead creation to first contact for non-duplicate leads with valid timestamp-precision rows. |
| `Average Response Time Coverage %` | Manager & Course | Share of non-duplicate leads meeting the timestamp and time-precision requirements of Average Response Time. |
| `Revenue per Manager` | Manager & Course | Gross Revenue in manager filter context. |
| `Revenue per Course` | Manager & Course | Gross Revenue in course filter context. |
| `Manager Conversion Rate` | Manager & Course | Won Leads divided by Total Leads in current context. |
| `Sales Target` | Manager & Course | Employment-aware monthly manager target total. |
| `Target Attainment %` | Manager & Course | Gross Revenue divided by Sales Target. |
| `Course & Cohort Enrollments` | Manager & Course | Course/cohort matrix enrollment helper. |
| `Course & Cohort Completion Rate` | Manager & Course | Course/cohort matrix completion helper. |
| `Course & Cohort Conversion Rate` | Manager & Course | Course/cohort matrix conversion display helper. |
| `Course & Cohort Gross Revenue` | Manager & Course | Course/cohort matrix revenue display helper. |
| `Course & Cohort Refund Rate` | Manager & Course | Course/cohort matrix refund display helper. |
| `Course & Cohort Avg Weekly Platform Hours` | Manager & Course | Course/cohort matrix engagement display helper. |

&nbsp;

## 3\. Main DAX Measure Definitions

The formulas below document the measures confirmed in the current project discussion. In particular, the updated maturity-based Completion Rate and the newly added response-time coverage measure supersede earlier definitions. Other listed formulas are retained from the supplied documentation unless otherwise noted.

### Total Leads

**DAX**

```DAX
CALCULATE(
    COUNTROWS(FactLeads),
    FactLeads[IsDuplicateCandidate] = FALSE()
)
```

&nbsp;

### Unique Leads

**DAX**

```DAX
DISTINCTCOUNT(FactLeads[PersonKey])
```

&nbsp;

### Conversion Rate

**DAX**

```DAX
VAR LeadCount =
    [Total Leads]
VAR EnrollmentCount =
    COALESCE([Enrollments], 0)

RETURN
    DIVIDE(
        EnrollmentCount,
        LeadCount
    )
```

&nbsp;

### Won Leads

**DAX**

```DAX
CALCULATE(
    [Total Leads],
    FactLeads[LeadStatusNormalized] = "Won"
)
```

&nbsp;

### Gross Revenue

**DAX**

```DAX
CALCULATE(
    SUM(FactPayments[GrossAmountBase]),
    FactPayments[IsFinancialKPIEligible] = TRUE()
)
```

&nbsp;

### Net Revenue

**DAX**

```DAX
CALCULATE(
    SUM(FactPayments[NetAmountBase]),
    FactPayments[IsFinancialKPIEligible] = TRUE()
)
```

&nbsp;

### Refund Amount

**DAX**

```DAX
CALCULATE(
    SUMX(
        FactPayments,
        FactPayments[RefundAmount] * FactPayments[FXRateApplied]
    ),
    FactPayments[IsFinancialKPIEligible] = TRUE()
)
```

&nbsp;

### Refund Rate

**DAX**

```DAX
DIVIDE(
    [Refund Amount],
    [Gross Revenue]
)
```

&nbsp;

### Average Order Value

**DAX**

```DAX
DIVIDE(
    [Gross Revenue],
    CALCULATE(
        DISTINCTCOUNT(FactPayments[EnrollmentID]),
        FactPayments[IsFinancialKPIEligible] = TRUE()
    )
)
```

&nbsp;

### Marketing Spend

**DAX**

```DAX
SUM(FactMarketingSpend[SpendBaseCurrency])
```

&nbsp;

### Cost per Lead

**DAX**

```DAX
DIVIDE(
    [Marketing Spend],
    [Total Leads]
)
```

&nbsp;

### Customer Acquisition Cost

**DAX**

```DAX
DIVIDE(
    [Marketing Spend],
    CALCULATE(
        DISTINCTCOUNT(FactPayments[StudentID]),
        FactPayments[IsFinancialKPIEligible] = TRUE()
    )
)
```

&nbsp;

### Return on Ad Spend

**DAX**

```DAX
DIVIDE(
    [Gross Revenue],
    [Marketing Spend]
)
```

&nbsp;

### Marketing ROI

**DAX**

```DAX
DIVIDE(
    [Net Revenue] - [Marketing Spend],
    [Marketing Spend]
)
```

&nbsp;

### Enrollments

**DAX**

```DAX
COUNTROWS(FactEnrollments)
```

&nbsp;

### Average Days to Enrollment

**DAX**

```DAX
AVERAGEX(
    FactEnrollments,
    VAR LeadCreatedAt =
        LOOKUPVALUE(
            FactLeads[CreatedAt],
            FactLeads[LeadID],
            FactEnrollments[LeadID]
        )
    RETURN
        IF(
            NOT ISBLANK(LeadCreatedAt),
            FactEnrollments[EnrollmentDate] - LeadCreatedAt
        )
)
```

&nbsp;

### Active Students

**DAX**

```DAX
CALCULATE(
    DISTINCTCOUNT(FactEnrollments[StudentID]),
    FactEnrollments[EnrollmentStatus] = "Active"
)
```

&nbsp;

### Completion Rate

Measures the proportion of completed enrollments among cohorts that have reached their expected end date by 26 February 2026\. It is not the share of `Completed` status among all enrollments.

**DAX**

```DAX
VAR CutoffDate = DATE(2026, 2, 26)
VAR CompletedMature =
    CALCULATE(
        [Enrollments],
        FactEnrollments[EnrollmentStatus] = "Completed",
        FactEnrollments[ExpectedEndDate] <= CutoffDate
    )
VAR TotalMature =
    CALCULATE(
        [Enrollments],
        FactEnrollments[ExpectedEndDate] <= CutoffDate
    )
RETURN
    DIVIDE(CompletedMature, TotalMature)
```

&nbsp;

**Unfiltered report value:** 50.65%. The separate all-enrollment `Completed` status share is 46.39%.

### Returning Student Rate

**DAX**

```DAX
VAR StudentEnrollmentCounts =
    ADDCOLUMNS(
        VALUES(FactEnrollments[StudentID]),
        "@EnrollmentCount",
            CALCULATE(
                COUNTROWS(FactEnrollments)
            )
    )
VAR ReturningStudents =
    COUNTROWS(
        FILTER(
            StudentEnrollmentCounts,
            [@EnrollmentCount] > 1
        )
    )
VAR TotalStudents =
    DISTINCTCOUNT(FactEnrollments[StudentID])
RETURN
    IF(
        TotalStudents = 0,
        BLANK(),
        DIVIDE(
            COALESCE(ReturningStudents, 0),
            TotalStudents
        )
    )
```

&nbsp;

### Average Weekly Platform Hours

**DAX**

```DAX
AVERAGE(
    FactStudentActivity[PlatformHours]
)
```

&nbsp;

### Dropout Rate

**DAX**

```DAX
VAR DroppedEnrollments =
    CALCULATE(
        [Enrollments],
        FactEnrollments[EnrollmentStatus] = "Dropped"
    )
RETURN
    DIVIDE(
        DroppedEnrollments,
        [Enrollments]
    )
```

&nbsp;

### Average Response Time

**DAX**

```DAX
AVERAGEX(
    FILTER(
        FactLeads,
        FactLeads[IsDuplicateCandidate] = FALSE()
            && NOT ISBLANK(FactLeads[CreatedAt])
            && NOT ISBLANK(FactLeads[FirstContactAt])
            && CONTAINSSTRING(FactLeads[CreatedAtRaw], ":")
            && CONTAINSSTRING(FactLeads[FirstContactAtRaw], ":")
    ),
    (FactLeads[FirstContactAt] - FactLeads[CreatedAt]) * 24
)
```

&nbsp;

### Average Response Time Coverage %

Measures the share of non-duplicate leads with both valid parsed timestamps and explicit time components. Uses the same timestamp eligibility criteria as `Average Response Time`, and respects the current report filter context.

**DAX**

```DAX
VAR EligibleLeads =
    CALCULATE(
        COUNTROWS(FactLeads),
        FactLeads[IsDuplicateCandidate] = FALSE(),
        NOT ISBLANK(FactLeads[CreatedAt]),
        NOT ISBLANK(FactLeads[FirstContactAt]),
        CONTAINSSTRING(FactLeads[CreatedAtRaw], ":"),
        CONTAINSSTRING(FactLeads[FirstContactAtRaw], ":")
    )
VAR TotalEligibleLeads =
    CALCULATE(
        COUNTROWS(FactLeads),
        FactLeads[IsDuplicateCandidate] = FALSE()
    )
RETURN
    DIVIDE(EligibleLeads, TotalEligibleLeads)
```

&nbsp;

**Format:** Percentage. **Unfiltered report value:** 81.53%.

### Manager Conversion Rate

**DAX**

```DAX
DIVIDE(
    [Won Leads],
    [Total Leads]
)
```

&nbsp;

### Sales Target

**DAX**

```DAX
VAR CourseIsFiltered =
    ISFILTERED(DimCourse[CourseName])

VAR SelectedMonths =
    SUMMARIZE(
        ALLSELECTED(DimDate),
        DimDate[Year],
        DimDate[MonthNumber]
    )

RETURN
    IF(
        CourseIsFiltered,
        BLANK(),
        SUMX(
            VALUES(DimManager[ManagerID]),
            
            VAR MonthlyTarget =
                CALCULATE(
                    MAX(DimManager[MonthlySalesTarget])
                )
                
            VAR HireDate =
                CALCULATE(
                    MIN(DimManager[HireDate])
                )
                
            VAR TerminationDate =
                CALCULATE(
                    MAX(DimManager[TerminationDate])
                )
                
            VAR ActiveMonthCount =
                COUNTROWS(
                    FILTER(
                        SelectedMonths,
                        
                        VAR MonthStart =
                            DATE(
                                DimDate[Year],
                                DimDate[MonthNumber],
                                1
                            )
                            
                        VAR MonthEnd =
                            EOMONTH(MonthStart, 0)
                            
                        RETURN
                            MonthEnd >= HireDate
                                &&
                            (
                                ISBLANK(TerminationDate)
                                    || MonthStart <= TerminationDate
                            )
                    )
                )
                
            RETURN
                MonthlyTarget * ActiveMonthCount
        )
    )
```

&nbsp;

### Target Attainment %

**DAX**

```DAX
DIVIDE(
    [Gross Revenue],
    [Sales Target]
)
```