# Recurring Month-End Expense-Report Corrections

![Power BI Dashboard](Recurring%20month-end%20corrections.png)

## Overview

This project investigates recurring manual corrections to an operating expense report in a fictional SME distribution business.

Using synthetic data covering **April–September 2026**, the case study moves from identifying repeated adjustments to examining the records and working practices behind them. Power BI brings together source expenses, initial report contributions, correction records and supporting process evidence.

The workflow is:

Synthetic Excel dataset → Power Query preparation → Power BI model and DAX measures → frequency and impact analysis → supporting evidence → proposed operational improvements

The project demonstrates reporting investigation, reconciliation, data modelling and practical recommendations for finance and operational teams.

**All records and process observations are synthetic. This is an illustrative portfolio case study, not a client engagement or evidence of realised savings.**

---

## The Problem

A monthly correction can become so familiar that nobody asks why it remains necessary.

The initial report contains **£3,168,001.15** in expenses. Manual corrections increase that figure by a net **£41,280**, giving a revised total of **£3,209,281.15**. The net change is approximately **1.3%** of the initial total.

That percentage alone does not explain the work involved. Additions and reductions offset, while reallocating an expense between departments has no net effect on company-level expenses.

The investigation asks:

* Which adjustments recur each month?
* How much preparation time is recorded?
* Which source records explain the corrections?
* Does the remedy belong in the report, coding instructions or close workflow?
* Which recurring adjustments serve a legitimate purpose?

> Repetition is a reason to investigate. The explanation determines the appropriate action.

---

## The Approach

### Dataset and scope

| Item | Scope |
|---|---|
| Reporting period | April–September 2026 |
| Evidence snapshot | 12 October 2026 |
| Departments | Warehouse, Transport, Sales support, Administration |
| Source expense lines | 1,800 |
| Initial report contribution rows | 1,812 |
| Approved adjustments | 55 |
| Correction impact lines | 61 |
| Currency | GBP, excluding VAT |

This is a **service-month operating expense report**. Its manual corrections are report-only adjustments, not a general-ledger journal population or a complete profit and loss account.

### Stage 1: Frequency, financial effect and preparation time

The first stage compares initial and revised expenses by department, adjustment counts by month, recorded reasons and active preparation time.

It identifies nine adjustments recurring in each month and one additional export-value correction in July. Five recorded reasons appear in all six months.

### Stage 2: Explaining selected corrections

The second stage follows the recurring patterns into supporting evidence:

* invoice receipt, approval and posting dates compared with extraction;
* source-line counts compared with report contribution counts;
* report account inclusion rules;
* coding templates compared with approved department allocations;
* and the documented basis for utility estimates.

Selected **April and August** examples illustrate the timing and duplication checks. Their totals are a subset of the six-month population.

---

## The Solution

### Data structure

The workbook is named `StraightLine_MonthEnd_Corrections.xlsx`.

| Table | Grain and purpose |
|---|---|
| `Periods` | One row per service month, including extract timestamp |
| `SourceLines` | One row per invoice line or monthly utility estimate |
| `ReportRows` | One row per initial report contribution, including zero-value excluded rows |
| `Adjustments` | One row per approved adjustment, with recorded reason and preparation minutes |
| `CorrectionLines` | One row per signed departmental cost impact |
| `Departments` | One row per department |
| `Accounts` | One row per expense account |
| `ProcessEvidence` | One row per supporting synthetic evidence record |

`ReadMe` provides import instructions and starter measures. `Dictionary` describes fields. `AuthorKey` contains investigation benchmarks and should be treated separately from reader-facing analysis.

An adjustment can have more than one impact line. Department reallocations have two opposing lines, explaining why there are 61 correction lines but only 55 adjustments.

### Power BI relationships

Use one-to-many relationships with single-direction filtering from the one side:

| One side | Many side | Key |
|---|---|---|
| `Periods` | `SourceLines` | `PeriodID` |
| `SourceLines` | `ReportRows` | `SourceLineID` |
| `SourceLines` | `Adjustments` | `SourceLineID` |
| `Adjustments` | `CorrectionLines` | `AdjustmentID` |
| `Departments` | `ReportRows` | `DepartmentID` |
| `Departments` | `CorrectionLines` | `DepartmentID` |
| `Accounts` | `SourceLines` | `AccountCode` |
| `ProcessEvidence` | `Adjustments` | `EvidenceID` |

Do not also connect Departments to SourceLines, or ProcessEvidence to SourceLines. Those references can remain available as fields without additional active relationships.

Keep the fact tables separate. Expanding report contributions and correction lines into a single table can multiply amounts and preparation minutes.

### Core measures

These examples assume query names match the sheet names, without the Excel table-name suffix `Table`.

```DAX
Initial Cost GBP =
SUM('ReportRows'[AmountGBP])
```

```DAX
Correction Impact GBP =
SUM('CorrectionLines'[AmountGBP])
```

```DAX
Revised Cost GBP =
[Initial Cost GBP] + [Correction Impact GBP]
```

```DAX
Correction Impact % =
DIVIDE([Correction Impact GBP], [Initial Cost GBP])
```

```DAX
Adjustment Count =
DISTINCTCOUNT('Adjustments'[AdjustmentID])
```

```DAX
Correction Hours =
DIVIDE(SUM('Adjustments'[PreparationMinutes]), 60)
```

```DAX
Included Report Row Count =
CALCULATE(
    COUNTROWS('ReportRows'),
    'ReportRows'[IncludedInExtract] = TRUE()
)
```

Positive correction impacts increase reported expense. Negative impacts reduce it. Preserve the sign in both calculations and display formats.

Department filters affect cost measures through ReportRows and CorrectionLines. They do **not** filter the adjustment register in this model. Analyse adjustment counts and preparation time by period or recorded reason rather than presenting them as departmental effort.

---

## Dashboard Highlights

### Initial overview

The overview combines departmental cost reconciliation, monthly adjustment counts, preparation time and recorded reasons.

| Recorded reason | Adjustment count | Net correction impact |
|---|---:|---:|
| Late invoice included | 18 | £33,585 |
| Duplicate contribution removed | 12 | −£14,130 |
| Expense added to report | 12 | £7,650 |
| Department allocation amended | 6 | £0 |
| Utility estimate added | 6 | £13,725 |
| Export value corrected | 1 | £450 |
| **Total** | **55** | **£41,280** |

Five months show nine adjustments and **2.85 hours** of preparation each. July shows ten adjustments and **3.35 hours**, bringing the six-month total to **17.6 hours**.

### Supporting evidence and selected examples

The detailed pages show evidence records by department, invoice-processing dates, source-to-report contribution counts and expected utility estimates.

For the selected April and August duplication examples, four source lines total **£4,660**, but contribute **£9,320** to the initial report. Corrections of **−£4,660** restore the source amounts.

The department comparison shows the financial effect while the event dates and row counts explain selected mechanisms.

---

## Key Findings

### 1. Invoice processing continues after extraction

The recurring invoices were received before the extract but approved and posted afterwards. This supports reviewing approval routing, capacity and extraction timing. It does not establish that invoices arrived late or that an individual caused the delay.

### 2. Duplicate contributions originate in the report join

The supporting evidence identifies a join from supplier to contact records. Two active contacts produce two report contributions for a single source expense line.

The relevant remedy is to correct the join and verify contribution counts. Duplicate invoice-entry warnings would not address this mechanism.

### 3. An account is omitted from the report configuration

Account A699 is absent from the inclusion list, requiring **£7,650** of additions across six months. The same report configuration remains in use. The evidence does not establish that a fix was already created or approved.

### 4. A coding template repeatedly assigns the wrong department

An active template allocates a recurring expense to Administration, while the approved invoice allocation specifies Warehouse. Corrections transfer **£8,100** across the six months. The company-level net effect is zero, although both departmental results change.

### 5. Utility estimates are expected adjustments

The **£13,725** of utility estimates are authorised report-only accruals awaiting invoices. Subsequent settlement and reversal are not included. Estimate accuracy cannot be assessed without later invoices and supporting consumption or rate information.

### 6. Not every correction recurs

The **£450** export-value correction appears once, in July. This dataset provides no evidence of repetition for that incident.

---

## Operational Recommendations

These are proposals arising from the fictional evidence. No changes or savings have been implemented or verified.

### 1. Coordinate the month-end close

Agree a working-day timetable for reviewing pending invoices, completing approvals and postings, resolving exceptions, closing the period and producing the final extract.

Test the timetable against actual processing dates and capacity. Posting restrictions or automated closure should support an agreed process with authorised exception handling, rather than simply blocking unresolved work.

### 2. Correct reporting and coding configurations

Review the duplicate-producing join, include the omitted account and amend the department coding template. Assign responsibility for testing, approval and implementation.

Verify source-line counts, amounts and department allocations in the next reporting cycle. Explain the revised coding and checking instructions to the staff using them.

### 3. Retain necessary estimates and monitor results

Continue authorised utility estimates and compare them with subsequent invoices when available. Investigate persistent differences using consumption, rates and the estimate basis.

Monitor recurring correction counts and preparation time after any implemented changes. This would establish whether repeated work falls while reporting remains complete and appropriately controlled.

---

## Data Limitations

* All source records, amounts, timings, roles and evidence notes are synthetic.
* The dataset is an operating expense-reporting exercise, not a complete accounting system or financial audit.
* Preparation minutes exclude investigation, waiting, meetings and wider checking activity. They are not a measured saving opportunity.
* Net correction impact is not total correction activity, financial loss or a forecast saving.
* ReportRows intentionally contains duplicated contributions and zero-value excluded rows. Do not remove them before investigating the initial report.
* Service month and posting date are distinct. Use Periods for the reporting-month analysis.
* Utility estimates have no subsequent invoice settlement or reversal records in this scope.
* ProcessEvidence supports selected illustrative explanations. It does not establish staff intent or the prevalence of these issues in real businesses.
* No post-change outcome data is supplied.

---

## Demo

Suggested walkthrough order:

1. Read the scope, dictionary and table definitions.
2. Review monthly counts and preparation time.
3. Compare net financial impact with the work involved.
4. Examine recurring reasons and their supporting evidence.
5. Follow the selected April and August invoice dates and duplicated contributions.
6. Separate necessary estimates from potentially avoidable rework.
7. Review the proposed actions and how their effects would be verified.

The accompanying PDF is named `Recurring month-end corrections.pdf`. The dataset is named `StraightLine_MonthEnd_Corrections.xlsx`. Use the workbook for reproduction and the PDF for the narrative walkthrough.

---

## How to Run

### Requirements

* Power BI Desktop
* The supplied Excel workbook

No SQL database is required for this version.

### Steps

1. Download the workbook from the published GitHub or Kaggle project files.
2. In Power BI Desktop, select **Get data → Excel workbook** and choose the named data tables.
3. Rename imported queries without the `Table` suffix if using the DAX above.
4. In Power Query, confirm amounts are decimal numbers, preparation minutes are whole numbers and IncludedInExtract is Boolean. Retain ExtractTimestamp as date/time.
5. Create the single-direction relationships shown above and remove conflicting automatically detected relationships.
6. Add the core measures.
7. Build a department table with initial cost, correction impact, revised cost and correction percentage.
8. Build period and recorded-reason views for adjustment count and preparation hours.
9. Add an evidence table and a selected-source detail view. Use SourceLines[SourceLineID] to group the duplication investigation.
10. Reconcile the six-month totals to **55 adjustments**, **17.6 hours** and **£41,280 net correction impact**. Revised costs should total **£3,209,281.15**.

The same Markdown can support the GitHub README and Kaggle dataset description. The case-study PDF documents the findings; it is not a substitute for the underlying workbook when reproducing calculations.

---

## Suggested Business Use

This case study supports conversations with finance managers, report owners and operational teams about recurring reporting work.

It demonstrates how to trace a familiar correction to its supporting records, identify where a proposed change belongs, and distinguish necessary close activity from avoidable repetition.

The practical starting point is one question:

> Which correction does your team make every month, and has anyone traced why it keeps returning?

---

## Tools Used

* Synthetic Excel data
* Power BI Desktop
* Power Query
* DAX
* PDF case study export

Prepared by **StraightLine Data & Training**.

---

## Project Status

Completed as a two-stage illustrative portfolio case study. Recommendations remain proposed and untested.

Potential future work includes adding subsequent utility invoices, documenting implemented changes, comparing later reporting cycles and measuring whether recurring adjustments and preparation time decrease.
