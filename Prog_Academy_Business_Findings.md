# Prog Academy — Business Findings

## 1. Purpose

This document summarizes the main actionable findings from the final Power BI analysis. It focuses on the business meaning of the final KPI results, practical next steps, and the main limitations that should be considered before acting on the findings.

### Analytical definitions and data limitations

- **Financial KPI population:** 14,034 eligible payments; test records remain available for audit but are excluded from revenue and refund measures by default.
- **Lead versus person grain:** `Total Leads` counts distinct CRM LeadIDs/lead events, while `Unique Leads` represents contact-resolved people via `PersonKey`; the cleaning notebook reports 43,859 unique people across 51,021 cleaned lead rows. The executive report shows 50,000 non-duplicate analytical leads and 43,859 unique people. These metrics answer different questions.
- **Conversion:** the headline enrollment conversion is Enrollments / analytical Total Leads (12,161 / 50,000 = 24.32%), rather than a count of records with non-null `ConvertedAt`. There are 76 enrolled leads without `ConvertedAt`; this is a source-field inconsistency and must not silently redefine enrollment conversion. The conversion funnel and won-lead metrics use their own documented status populations.
- **Course completion:** the current Power BI Completion Rate KPI is 50.65%, while the Enrollment Status chart shows 46.39% of all enrollments with status `Completed`. These figures have different denominators or calculation contexts and must not be treated as interchangeable. The 50.65% KPI is maturity-adjusted: it divides Completed enrollments with `ExpectedEndDate` on or before 26 February 2026 by all enrollments with `ExpectedEndDate` on or before that date, within the current filter context. The denominator includes any eligible status, including Dropped, Cancelled, or Active; it is not limited to finalized outcomes.
- **Data controls:** cleaned-table keys, relationship consistency, payment/marketing FX reconciliation and export schema are subject to automated validation. The notebook still uses a configured project-root path specific to its Colab environment, so fully environment-independent execution has not been demonstrated.

## 2. Key Business Findings

### Finding 1 — Overall financial performance is strong relative to marketing spend

**Observation:**  
The business generated substantial revenue compared with recorded marketing spend. Financially eligible Gross Revenue reached UAH 201,365,382.59 and Net Revenue reached UAH 163,268,028.59 against UAH 17,716,298.22 of recorded Marketing Spend. Financial KPIs exclude test transactions by default while preserving those rows in the cleaned payments data. This indicates a high observed revenue-to-marketing-spend ratio at portfolio level.

**Supporting metrics:**  
Gross Revenue (financially eligible): UAH 201,365,382.59; Net Revenue (financially eligible): UAH 163,268,028.59; Marketing Spend: UAH 17,716,298.22; Return on Ad Spend: 11.37x; Marketing ROI: 821.57%. The Marketing Analytics dashboard displays Customer Acquisition Cost at approximately UAH 2.77K; this should be read in the model’s current filter context.  
ROAS = Gross Revenue / Marketing Spend. Marketing ROI = (Net Revenue − Marketing Spend) / Marketing Spend. CAC should use the distinct acquired-customer population defined by the active Power BI measure and the same report filter context; it should not be assumed to equal a transaction count.

**Possible explanation:**  
The result may reflect a combination of acquisition performance, course pricing, repeat payment schedules, and revenue generated from leads that are not fully attributable to paid marketing activity.

**Recommended action:**  
Maintain close monitoring of the current revenue-to-spend relationship while evaluating performance at source and campaign level before increasing spend. Budget increases should be tested incrementally rather than based only on the portfolio-level ROAS or ROI.

**Limitation / caveat:**  
ROAS and Marketing ROI compare observed revenue with marketing spend and should not be interpreted as causal marketing impact. Financial KPI eligibility requires a successful payment status (`Completed`, `Refunded`, or `Partially Refunded`) and `IsTestPayment = False`. Of 17,100 cleaned payment records, 68 are test payments (58 of these have successful statuses); none are eligible for financial KPIs, and 14,034 payments are financially eligible. The earlier inclusion of successful test payments would have yielded Gross Revenue of UAH 202,261,004.87 and Net Revenue of UAH 164,017,395.38, compared with the eligible-only values above. Recorded marketing spend also contains 25 missing original Spend values and four flagged UAH Spend/base-currency discrepancies, which limit complete cost attribution. Revenue-to-spend ratios are descriptive, not proof of incremental marketing return.

### Finding 2 — Major marketing source groups show different cost and lead-quality trade-offs

**Observation:**  
Among the major source groups with more than UAH 1 million in recorded marketing spend, Telegram has the lowest Cost per Lead (CPL) and the highest won-lead rate, while Google has the highest CPL. The combined Meta family accounts for the largest marketing spend and has a higher CPL and lower won-lead rate than Telegram. These descriptive results suggest that lead acquisition cost should be evaluated alongside downstream conversion rather than used alone for budget decisions.

**Supporting metrics:**  
Source-group metrics use `SourceGroup` in `DimMarketingSource` to align lead and marketing-spend granularity. Meta (combining `meta`, `facebook`, `instagram`, and `fb-insta`): UAH 9,833,895.51 spend, UAH 583.61 CPL, 20.79% won-lead rate. Google: UAH 6,476,230.34 spend, UAH 644.85 CPL, 24.78% won-lead rate. Telegram: UAH 1,363,048.03 spend, UAH 399.84 CPL, 25.37% won-lead rate. Meta has 16,850 analytical leads in the grouped calculation. The report's first-touch and last-touch visual presents grouped source families for comparison; these attribution measures represent a different analytical perspective from the primary source-group cost comparison.

**Possible explanation:**  
Source groups may reach audiences with different purchase intent, while differences in campaign mix, targeting, geography, and creative may also contribute to observed CPL and won-lead rates. The data does not isolate the causal impact of any one factor.

**Recommended action:**  
Prioritize controlled evaluation of Telegram's apparent cost-and-conversion advantage, while maintaining comparative tests across Meta and Google. Assess incremental results using paid-student CAC, attributed revenue, refund behaviour, and lead quality before reallocating budgets. Do not recommend shifting budget specifically to Instagram based on its former standalone CPL, because Meta-related spend and CRM lead sources were recorded at inconsistent levels of detail.

**Limitation / caveat:**  
Meta, Facebook, Instagram, and FB/Insta were separate source labels in the original data, but the spend and CRM lead classifications were not consistently granular; combining them prevents misleading standalone CPL comparisons. Source-group results are descriptive, not causal attribution, and first-touch and last-touch counts may differ. Recorded zero spend must be distinguished from missing spend information: Direct, Organic, and Referral show zero recorded base-currency spend in the current source comparison, whereas other lead sources have no corresponding spend records; neither condition establishes that acquisition had no economic cost. In addition, one Direct record dated 2025-08-05 has `Spend = 659,508 UAH` but `SpendBaseCurrency = 0 UAH`; the unresolved inconsistency was flagged rather than silently corrected, so Direct should not be described as conclusively cost-free. The cleaned marketing data also retains 25 missing `Spend` values and 4 unresolved spend/base-currency mismatches.

### Finding 3 — Shorter lead response coincides with stronger manager conversion in selected portfolios

**Observation:**  
The final Sales Performance analysis shows a descriptive pattern in which shorter response times coincide with stronger manager conversion for the manager portfolios highlighted below. Vasyl F. handled a high lead volume with a 3.01-hour average response time and a 27.76% manager conversion rate, while two slower-response portfolios were close to 14 hours and had conversion rates below 21%.

**Supporting metrics:**  
Vasyl F.: 11,004 leads, 3.01-hour Average Response Time, 27.76% Manager Conversion Rate, approximately UAH 50.88M Gross Revenue in the updated dashboard. John З.: 8,858 leads, 13.96 hours, 18.88%, approximately UAH 27.23M Gross Revenue. Иларион Д.: 7,032 leads, 14.06 hours, 20.76%, approximately UAH 23.77M Gross Revenue. The figures shown in the manager chart are rounded.

**Possible explanation:**  
Faster contact may help reach leads while interest is still high, but manager experience, lead mix, course mix, workload, source quality, geography, and manager tenure may also explain part of the difference.

**Recommended action:**  
Review lead-assignment workload and introduce or review a practical first-contact response target. Track response time together with conversion by manager over time to test whether the descriptive pattern remains consistent.

**Limitation / caveat:**  
This is an observational comparison, not evidence that response time alone caused the conversion difference. Average Response Time uses only non-duplicate leads with valid creation and first-contact timestamps containing time components. The KPI has 81.53% coverage of non-duplicate leads in the overall report context; the remaining 18.47% are excluded from the response-time calculation and may differ systematically. Coverage should be viewed alongside the response-time analysis. Manager portfolios also differ in lead volume and potentially in course, source, geography, workload, and tenure mix.

### Finding 4 — Free introductory courses generate high enrollment conversion but also high dropout

**Observation:**  
The two free introductory courses generated high enrollment volumes and lead-to-enrollment conversion rates in the course analysis, but more than 41% of their enrollments dropped. Because these courses generate no direct Gross Revenue, their business value depends on engagement and progression into later paid learning rather than direct course revenue.

**Supporting metrics:**  
Data Analytics (Free Intro): 2,183 enrollments, 43.51% Conversion Rate, 50.02% Completion Rate, 41.73% Dropout Rate, UAH 0 Gross Revenue. IT Start (Free): 2,079 enrollments, 42.55% Conversion Rate, 48.87% Completion Rate, 42.76% Dropout Rate, UAH 0 Gross Revenue.

**Possible explanation:**  
The free price point lowers the barrier to enrollment, which can increase sign-ups but may also attract learners with lower commitment or less certainty about continuing.

**Recommended action:**  
Strengthen onboarding and early engagement for free-course students, and track how many learners later enroll in paid courses. Use progression to paid study, not free-course enrollment volume alone, as the main measure of funnel value.

**Limitation / caveat:**  
Free and paid course conversion rates are not directly comparable because the enrollment decision has a different financial barrier. Course-level Completion Rate values use the same maturity-adjusted logic as the headline KPI: Completed enrollments divided by all enrollments with `ExpectedEndDate` on or before 26 February 2026, under the applicable course and report filters. They do not represent outcomes restricted to finalized enrollment statuses. Separately, the Enrollment Status chart counts 46.39% of all enrollments as Completed, whereas the maturity-adjusted headline KPI is 50.65%; these have different denominators and should not be presented as equivalent. Free-versus-paid differences should not be interpreted as causal effects of course price. The current finding does not prove that completing a free course leads to a later paid enrollment.

### Finding 5 — Student retention and repeat enrollment remain important opportunities

**Observation:**  
At the current observation point, 46.39% of enrollments have Completed status and 31.11% have Dropped status. The maturity-adjusted Completion Rate KPI is 50.65% among enrollments with `ExpectedEndDate` on or before 26 February 2026. Repeat enrollment is also limited: only 474 students have more than one enrollment in the dataset. These results identify learner retention and post-course re-engagement as potential opportunities to increase customer lifetime value.

**Supporting metrics:**  
Maturity-adjusted Completion Rate KPI (expected end date ≤ 26 February 2026): 50.65%; share of enrollments with Completed status: 46.39%; Dropout Rate: 31.11%; Current Active Students: 1,692; Returning Student Rate: 4.06%; returning students: 474 of 11,676 distinct enrolled students. The Enrollment Status chart shows Active 14.02% and Cancelled 8.49%.

**Possible explanation:**  
Dropout may reflect course fit, learner expectations, workload, engagement, or external circumstances. The low returning rate may also reflect a limited follow-up journey after completion or the available observation window.

**Recommended action:**  
Create targeted retention follow-ups for at-risk students and a separate post-completion journey for graduates who may be suitable for another course. Track dropout and repeat enrollment by course and cohort to identify where intervention has the greatest potential value.

**Limitation / caveat:**  
Returning Student Rate measures students with more than one enrollment within the available project data; it is not a time-adjusted cohort retention metric. The 46.39% figure is the share of all enrollments currently labelled Completed. The 50.65% KPI instead divides Completed enrollments with `ExpectedEndDate` on or before 26 February 2026 by all enrollments meeting that date cutoff, in the current filter context. This reduces bias from not-yet-due courses, but it still includes any Active or Cancelled enrollment meeting the cutoff in its denominator and is not a finalized-outcomes-only rate. The fixed cutoff date should be reviewed when extending the reporting period.

### Finding 6 — Refund exposure is concentrated more heavily in some courses

**Observation:**  
The updated dashboard displays an overall monetary Refund Rate of 7.04%, but several paid courses are above that level. QA Engineer has the highest observed course refund rate among the main paid courses reviewed, while QA Automation and Front-End are also above the overall rate. This suggests that course-level refund monitoring can identify areas where the customer experience or expectation setting should be reviewed.

**Supporting metrics:**  
Overall monetary Refund Rate: 7.04% in the updated Power BI model. QA Engineer: 9.10%; QA Automation: 8.08%; Front-End: 8.01%. These are current course-level Power BI values after exclusion of test payments from financial KPI eligibility. The earlier raw-inclusive refund total of UAH 14,202,401.33 is not used as the current financially eligible refund total.

**Possible explanation:**  
Higher refund levels may be related to course expectations, content fit, cohort experience, payment plans, or other factors, but the current analysis does not identify a single confirmed cause and does not show that higher refunds are evidence of poor course quality.

**Recommended action:**  
Review refund reasons, payment plans, and cohort patterns for the higher-refund courses before changing pricing or marketing. Compare refund patterns with completion and engagement results to identify recurring issues that can be addressed operationally.

**Limitation / caveat:**  
Refund Rate is a monetary measure calculated as financially eligible Refund Amount divided by financially eligible Gross Revenue, not the percentage of payment transactions refunded. The cleaned payments data also retains 50 refund-status inconsistencies as explicit quality flags, so those records should be considered during detailed refund investigation.

## 3. Overall Summary

The final analysis uses financially eligible payments for revenue and refund KPIs, contact-resolved PersonKey for unique-person analysis, grouped marketing-source families for comparable acquisition analysis, and separate reporting of response-time coverage and course-completion definitions. It shows a high observed revenue-to-marketing-spend ratio while identifying several areas for further testing and investigation. Marketing sources differ in cost and lead quality, attribution changes how channel contribution is viewed, shorter response times coincide with stronger manager conversion in selected portfolios, free courses need stronger progression and retention tracking, repeat enrollment remains low, and refund exposure is higher in selected courses. The recommended next step is to preserve careful measurement while using controlled testing, monitoring, and targeted follow-up across acquisition, sales, course delivery, student retention, and refunds.
