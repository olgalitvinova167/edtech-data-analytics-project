# Prog Academy — Business Findings

## 1. Purpose

This document summarizes the main actionable findings from the final Power BI analysis. It focuses on the business meaning of the final KPI results, practical next steps, and the main limitations that should be considered before acting on the findings.

## 2. Key Business Findings

### Finding 1 — Overall financial performance is strong relative to marketing spend

**Observation:**  
The business generated substantial revenue compared with recorded marketing spend. Gross Revenue reached UAH 202,261,004.87 and Net Revenue reached UAH 164,017,395.38 against UAH 17,716,298.22 of Marketing Spend. This indicates a high observed revenue-to-marketing-spend ratio at portfolio level.

**Supporting metrics:**  
Gross Revenue: UAH 202,261,004.87; Net Revenue: UAH 164,017,395.38; Marketing Spend: UAH 17,716,298.22; Return on Ad Spend: 11.42x; Marketing ROI: 825.80%; Customer Acquisition Cost: UAH 2,758.26.  
ROAS = Gross Revenue / Marketing Spend. Marketing ROI = (Net Revenue − Marketing Spend) / Marketing Spend. CAC = Marketing Spend / distinct students represented in financially eligible payments.

**Possible explanation:**  
The result may reflect a combination of acquisition performance, course pricing, repeat payment schedules, and revenue generated from leads that are not fully attributable to paid marketing activity.

**Recommended action:**  
Maintain close monitoring of the current revenue-to-spend relationship while evaluating performance at source and campaign level before increasing spend. Budget increases should be tested incrementally rather than based only on the portfolio-level ROAS or ROI.

**Limitation / caveat:**  
ROAS and Marketing ROI compare observed revenue with marketing spend and should not be interpreted as causal marketing impact. The final financial KPI logic includes 58 successful test payments because they meet the approved payment-status eligibility rule. A sensitivity check excluding all `IsTestPayment = True` records reduced Gross Revenue to UAH 201,365,382.59, Net Revenue to UAH 163,268,028.59, ROAS to 11.37x and Marketing ROI to 821.57%; all changes were below 0.6%, so the financial conclusions did not materially change.

### Finding 2 — Major marketing sources show different cost and lead-quality trade-offs

**Observation:**  
Among the major sources with more than UAH 1 million of recorded spend and substantial lead volume, Instagram had the lowest Cost per Lead, while Telegram had the strongest won-lead rate. Instagram's won-lead rate was lower than Telegram's and Google's, while Google had the highest spend and the highest Cost per Lead of these four sources. This suggests that CPL alone should not determine budget allocation.

**Supporting metrics:**  
Source-level metrics below use the report's primary normalized source context (`SourceNormalized` via `DimMarketingSource`), rather than the explicit first-touch or last-touch measures. Google: UAH 6,476,230.34 spend, UAH 644.85 CPL, 24.78% won-lead rate. Meta: UAH 5,487,665.94 spend, UAH 462.94 CPL, 21.14% won-lead rate. Instagram: UAH 1,485,107.94 spend, UAH 302.53 CPL, 19.98% won-lead rate. Telegram: UAH 1,363,048.03 spend, UAH 399.84 CPL, 25.37% won-lead rate.

**Possible explanation:**  
Different sources may attract audiences with different levels of purchase intent, while campaign mix, geography, creative, and targeting can also affect both lead cost and sales qualification.

**Recommended action:**  
Prioritize controlled testing of Telegram and Instagram rather than immediately reallocating budget, and evaluate the results using paid-student CAC, revenue, and refund behaviour. Continue assessing CPL together with downstream lead quality rather than optimizing only for lead volume.

**Limitation / caveat:**  
The source comparison is not causal attribution. First-touch and last-touch are analyzed separately in the final report, and attribution choice can materially change channel counts: Meta has 11,859 first-touch leads versus 10,945 last-touch leads, while Perplexity has 763 first-touch versus 1,200 last-touch leads. The marketing data also retains 25 missing `Spend` values and 4 unresolved spend/base-currency mismatches.

### Finding 3 — Shorter lead response coincides with stronger manager conversion in selected portfolios

**Observation:**  
The final Sales Performance analysis shows a descriptive pattern in which shorter response times coincide with stronger manager conversion for the manager portfolios highlighted below. Vasyl F. handled a high lead volume with a 3.01-hour average response time and a 27.76% manager conversion rate, while two slower-response portfolios were close to 14 hours and had conversion rates below 21%.

**Supporting metrics:**  
Vasyl F.: 11,004 leads, 3.01-hour Average Response Time, 27.76% Manager Conversion Rate, UAH 50,971,903.46 Gross Revenue. John З.: 8,858 leads, 13.96 hours, 18.88%, UAH 27,503,117.44. Иларион Д.: 7,032 leads, 14.06 hours, 20.76%, UAH 23,798,625.93.

**Possible explanation:**  
Faster contact may help reach leads while interest is still high, but manager experience, lead mix, course mix, workload, source quality, geography, and manager tenure may also explain part of the difference.

**Recommended action:**  
Review lead-assignment workload and introduce or review a practical first-contact response target. Track response time together with conversion by manager over time to test whether the descriptive pattern remains consistent.

**Limitation / caveat:**  
This is an observational comparison, not evidence that response time alone caused the conversion difference. Manager portfolios differ in lead volume and may also differ in course, source, geography, workload, and tenure mix.

### Finding 4 — Free introductory courses generate high enrollment conversion but also high dropout

**Observation:**  
The two free introductory courses generated the highest enrollment volumes and had the highest lead-to-enrollment conversion rates in the course analysis, but more than 41% of their enrollments dropped. Because these courses generate no direct Gross Revenue, their business value depends on engagement and progression into later paid learning rather than direct course revenue.

**Supporting metrics:**  
Data Analytics (Free Intro): 2,183 enrollments, 43.51% Conversion Rate, 50.02% Completion Rate, 41.73% Dropout Rate, UAH 0 Gross Revenue. IT Start (Free): 2,079 enrollments, 42.55% Conversion Rate, 48.87% Completion Rate, 42.76% Dropout Rate, UAH 0 Gross Revenue.

**Possible explanation:**  
The free price point lowers the barrier to enrollment, which can increase sign-ups but may also attract learners with lower commitment or less certainty about continuing.

**Recommended action:**  
Strengthen onboarding and early engagement for free-course students, and track how many learners later enroll in paid courses. Use progression to paid study, not free-course enrollment volume alone, as the main measure of funnel value.

**Limitation / caveat:**  
Free and paid course conversion rates are not directly comparable because the enrollment decision has a different financial barrier. The current finding does not prove that completing a free course leads to a later paid enrollment.

### Finding 5 — Student retention and repeat enrollment remain important opportunities

**Observation:**  
Less than half of enrollments are completed, while almost one-third are dropped. Repeat enrollment is also limited: only 474 students have more than one enrollment in the dataset. These results identify learner retention and post-course re-engagement as potential opportunities to increase customer lifetime value.

**Supporting metrics:**  
Completion Rate: 46.39%; Dropout Rate: 31.11%; Active Students: 1,692; Returning Student Rate: 4.06%; returning students: 474 of 11,676 distinct enrolled students.

**Possible explanation:**  
Dropout may reflect course fit, learner expectations, workload, engagement, or external circumstances. The low returning rate may also reflect a limited follow-up journey after completion or the available observation window.

**Recommended action:**  
Create targeted retention follow-ups for at-risk students and a separate post-completion journey for graduates who may be suitable for another course. Track dropout and repeat enrollment by course and cohort to identify where intervention has the greatest potential value.

**Limitation / caveat:**  
Returning Student Rate measures students with more than one enrollment within the available project data; it is not a time-adjusted cohort retention metric. Active enrollments have also not yet reached a final completion or dropout outcome.

### Finding 6 — Refund exposure is concentrated more heavily in some courses

**Observation:**  
The overall monetary Refund Rate is 7.02%, but several paid courses are above that level. QA Engineer has the highest observed course refund rate among the main paid courses reviewed, while QA Automation and Front-End are also above the overall rate. This suggests that course-level refund monitoring can identify areas where the customer experience or expectation setting should be reviewed.

**Supporting metrics:**  
Overall Refund Amount: UAH 14,202,401.33; overall Refund Rate: 7.02%. QA Engineer Refund Rate: 9.06%; QA Automation: 8.10%; Front-End: 8.04%.

**Possible explanation:**  
Higher refund levels may be related to course expectations, content fit, cohort experience, payment plans, or other factors, but the current analysis does not identify a single confirmed cause and does not show that higher refunds are evidence of poor course quality.

**Recommended action:**  
Review refund reasons, payment plans, and cohort patterns for the higher-refund courses before changing pricing or marketing. Compare refund patterns with completion and engagement results to identify recurring issues that can be addressed operationally.

**Limitation / caveat:**  
Refund Rate is a monetary measure calculated as Refund Amount divided by Gross Revenue, not the percentage of payment transactions refunded. The cleaned payments data also retains 50 refund-status inconsistencies as explicit quality flags, so those records should be considered during detailed refund investigation.

## 3. Overall Summary

The final analysis shows a high observed revenue-to-marketing-spend ratio, while the detailed results identify several areas for further testing and investigation. Marketing sources differ in cost and lead quality, attribution changes how channel contribution is viewed, shorter response times coincide with stronger manager conversion in selected portfolios, free courses need stronger progression and retention tracking, repeat enrollment remains low, and refund exposure is higher in selected courses. The recommended next step is to preserve careful measurement while using controlled testing, monitoring, and targeted follow-up across acquisition, sales, course delivery, student retention, and refunds.
