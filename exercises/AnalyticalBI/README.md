# Exercise 4 - Analytical Business Insights (ABI) Agent

## 1. Purpose and Scope

This test script validates the end-to-end functionality of the Analytical Business Insights (ABI) Agent as embedded in the SAP Fiori application **Financial Statements Review Booklet**.

The ABI Agent is a conversational AI capability surfaced via SAP Joule that allows financial controllers and GL accountants to calculate KPIs, drill into variances, and perform contribution analysis — directly within the context of live financial statement data — without leaving the report.

The script covers the complete interaction sequence from report setup through KPI calculation, individual KPI analysis, component-level drill-down, smart drill-down, variance summarization, and navigation back to the KPI list.

## 2. E2E Process Overview

The diagram below shows the complete process.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 1 — REPORT SETUP                                                     │
│  Open Financial Statements Review Booklet; set filter parameters;           │
│  execute report; verify data loaded under "All Accounts" tab                │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 2 — LAUNCH ABI AGENT                                                 │
│  Open Joule from top-right corner; start a new conversation                 │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 3 — KPI CALCULATION                                                  │
│  Enter prompt "Calculate all KPIs";                                         │
│  verify KPI list displayed, sorted by variance %                            │
│  Click "View More" to see full list                                         │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 4 — KPI ANALYSIS (DRILL INTO SINGLE KPI)                             │
│  Select a KPI (e.g. Quick Ratio); click "Analyze";                          │
│  verify formula components filtered in grid, KPI details in Joule           │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 5 — COMPONENT-LEVEL DRILL-DOWN                                       │
│  Select a formula component (e.g. Cash & Cash Equivalents);                 │
│  verify report filtered by component; comparison chart and                  │
│  summary displayed in Joule; Smart Drill Down & Summarize Variance shown    │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 6 — SMART DRILL DOWN                                                 │
│  Click "Smart Drill Down"; verify additional dimensions added to report     │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 7 — SUMMARIZE VARIANCE                                               │
│  Click "Summarize Variance"; verify contribution analysis displayed         │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 8 — RETURN TO KPI LIST                                               │
│  Click "KPI List"; verify navigation back to full sorted KPI list           │
└──────────────────────────────────────────────────────────────────────────────┘
```

## 3. Test System & Access

| Item | Value |
|------|-------|
| Test System | [https://my441372.s4hana.cloud.sap/ui](https://my441372.s4hana.cloud.sap/ui) |
| User | AS260-0XX *(replace XX with your participant number)* |
| Password | Acce$$teched26 |

## 4. Preparation — Test Data Requirements

In this test case, you will work with the following data.

| Data Element | Value / Requirement |
|-------------|---------------------|
| Company Code | 1710 (US Company Code) |
| Ledger | 0L (Leading Ledger) |
| Financial Statement Version | 1799 — Financial Statement Version US |
| Fiscal Year (Current) | 2026 — Financial data must be posted for FY 2026 in company code 1710 |
| Fiscal Year (Comparison) | 2025 — Financial data must be available for FY 2025 for year-on-year comparison |
| Fiscal Periods | Choose periods 1 to 9 for both the current year 2026 and the comparison year 2025 |
| Currency Type | 10 (Company Code Currency = USD) |
| Language | English — The ABI Agent AI features do not operate on non-English content |

### 4.1 Data Readiness Check

Before running the test, open the **Financial Statements Review Booklet** and execute the report with the parameters listed above. If no data is displayed, stop and verify that financial data exists for FY 2026 and FY 2025 in company code 1710.

To do this, login into the SAP S/4HANA system and type in *"Financial statement review booklet"* in the Apps search field. Once the icon is shown, click it to open the app. Enter the selection parameters as indicated above and click **Go** to execute the report.

### 4.2 Glossary

| Term | Definition |
|------|-----------|
| **ABI Agent** | Analytical Business Insights Agent — a conversational AI capability embedded in SAP Fiori finance applications via SAP Joule. It automatically calculates financial KPIs, performs variance analysis, and supports drill-down exploration based on the data currently visible in the report. |
| **Financial Statements Review Booklet** | An SAP Fiori application for reviewing financial statements. It displays a structured view of the balance sheet and income statement, with comparison between two fiscal years, for a given company code, ledger, and financial statement version. |
| **KPI** | A calculated financial metric derived from the report data, such as Quick Ratio, Current Ratio, or EBITDA Margin. The ABI Agent calculates KPIs automatically based on the current report context and ranks them by variance percentage. |
| **Variance** | The difference — expressed as an absolute value and as a percentage — between the current fiscal year figure and the comparison fiscal year figure for a given account or KPI. |
| **Analyze** | An action button next to each KPI in the Joule panel. Clicking it filters the report grid to show only the GL accounts that form the formula components of that KPI. |
| **Formula Component** | One of the individual GL account groups or balance sheet line items that make up a KPI formula (e.g., Quick Ratio uses Cash & Cash Equivalents, Short-term Investments, Accounts Receivable, and Current Liabilities). |
| **Smart Drill Down** | An AI-driven analysis step that automatically adds the most relevant analytical dimensions (e.g., Profit Center, Cost Center, Segment) to the report to deepen the variance analysis. |
| **Summarize Variance** | An AI-driven contribution analysis that identifies which specific member or combination of members contributes most to the overall variance for the selected component or KPI. |
| **KPI List** | The initial view shown by the ABI Agent after "Calculate all KPIs" is executed. Displays all calculated KPIs sorted by variance percentage. |
| **View More** | A button shown in the Joule panel after KPI calculation that expands the KPI list from a summary view to the complete list of all calculated KPIs. |

## 5. Test Execution

### 5.1 Report Setup

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **1** | From the Fiori Launchpad, search for and open the application **Financial Statements Review Booklet**. | The application opens, and the filter bar is displayed. No data is loaded yet. |
| **2** | In the filter bar, fill in all mandatory fields:<br>**Ledger:** 0L · **Company Code:** 1710 · **Financial Statement Version:** 1799<br>**Fiscal Year:** 2026 · **From/To Fiscal Period:** 1 / 9<br>**Comparison From/To Period:** 1 / 9 · **Comparison Fiscal Year:** 2025 · **Currency Type:** 10 | All filter fields are filled without validation errors. |
| **3** | Click **Go** (or press Enter) to execute the report. | The report executes. The **All Accounts** page is active by default, showing balance sheet and income statement line items for FY 2026 with comparison figures for FY 2025. |

### 5.2 Launch the ABI via Joule

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **4** | Click the diamond icon (◇) in the top-right area of the application to open the SAP Joule side panel. | The Joule side panel opens on the right side of the screen. The report remains visible on the left. |
| **5** | In the Joule panel, click the menu button to start a new conversation (or click **New Conversation** if a button is shown). | A fresh conversation context is established in Joule. Any previous conversation history is cleared. |

### 5.3 KPI Calculation

> [!WARNING]
> **IMPORTANT:** Before entering a prompt, make sure that **"agentic thinking" is DISABLED**. Also, do **NOT** enter "@financial closing assistant" — start typing the prompt directly.

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **6** | In the Joule input field, type and send:<br>`Calculate all KPIs`<br><br>*Alternative prompts: "View KPI values", "Show all KPIs", "Calculate Financial KPIs".* | The ABI Agent returns a KPI summary list sorted in descending order by absolute variance percentage. Each entry shows KPI name, current-year value, comparison-year value, and variance %. A **View More** button and an **Analyze** button are visible next to each entry. |
| **7** | Click the **View More** button in the Joule panel. | The complete list of all calculated KPIs is expanded, sorted by variance percentage descending. An **Analyze** button is shown next to each KPI. |

### 5.4 KPI Analysis (Drill down into a single KPI)

*Example KPI: Quick Ratio. Any KPI from the list may be used; the behavior is identical for all KPIs.*

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **8** | From the KPI list, choose a KPI — for example, **Quick Ratio** — and click the **Analyze** button next to it. | **(a) Report grid:** Filtered to show only the GL accounts forming the KPI formula components. Unrelated accounts are hidden.<br>**(b) Joule panel:** Agent displays KPI formula, current-year and comparison-year values per component, overall KPI value and variance. Each formula component appears as a clickable **Analyze [Component Name]** button. |

### 5.5 Component-level drill down

*Example component: Cash & Cash Equivalents.*

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **9** | Click the **Analyze [Component Name]** button for one component — for example, **Analyze Cash & Cash Equivalents**. | **(a) Report grid:** Further filtered to show only the accounts belonging to the selected component.<br>**(b) Joule panel:** A comparison chart (bar or column) showing current-year vs. comparison-year amounts; a written variance summary (absolute change, % change, brief narrative); action buttons **Smart Drill Down** and **Summarize Variance** are shown. |

### 5.6 Smart drill down

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **10** | In the Joule panel, click the **Smart Drill Down** button. | The agent automatically adds one or more analytical dimensions (e.g., Profit Center, Cost Center, Segment) to the report grid. The grid updates to display data broken down by the added dimension(s). The Joule panel explains which dimensions were added and why. |

### 5.7 Summarize Variance

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **11** | In the Joule panel, click the **Summarize Variance** button. | The agent performs a contribution analysis and displays: the specific member or combination of members contributing most to the overall variance; the contribution amount (absolute and %); a ranked list of top contributing members; a brief narrative summary. |

### 5.8 Return to KPI List

| Step | Description | Expected Result / Check |
|------|-------------|------------------------|
| **12** | In the Joule panel, click the **KPI List** button. | The Joule panel navigates back to the full sorted KPI list (same view as Step 7). The report grid resets to show all accounts. The tester can select a different KPI and continue the analysis cycle. |

## 9. Further Notes and Hints

### 9.1 General Checks

- Verify that the Joule panel and the report grid stay in sync throughout the test — each agent action should visibly update the report grid as described in the expected results.
- Verify that the KPI list is consistently sorted by variance percentage (highest first) each time it is displayed.
- Verify that the revenue comparison chart correctly reflects the current-year vs. comparison-year data visible in the grid at the time.
- Verify that the contribution analysis (Summarize Variance) correctly identifies the dimension member with the highest contribution — cross-check the top entry against the dimension-level detail visible in the grid after Smart Drill Down.

### 9.2 Language Requirements

The ABI Agent AI features operate exclusively on **English-language content**. If the Financial Statement Version contains account group descriptions in a language other than English, the KPI calculation and analysis steps will fail or produce incomplete results.

### 9.3 Hints

- **KPI selection:** The Quick Ratio is used as the example throughout this script. Any KPI from the calculated list can be substituted — the interaction pattern (Analyze → component selection → Smart Drill Down → Summarize Variance → KPI List) is identical for all KPIs.
- **Additional analysis cycles:** After Step 12, click Analyze on a different KPI to run another cycle. This is recommended to verify consistency across different KPI types (liquidity, profitability, leverage, etc.).
- **Joule conversation context:** The ABI Agent maintains context within a single Joule conversation. Returning to the KPI List does not lose the conversation context. Starting a new conversation resets the agent's context and requires re-entering the "Calculate all KPIs" prompt.
- **Smart Drill Down dimensions:** The specific dimensions added depend on the underlying data structure. Accept any meaningful business dimension (Profit Center, Cost Center, Segment, etc.) as a valid result.

## Summary

You've now completed the Analytical Business Insights Agent exercise. You can continue with any of the other agent exercises:

- [Exercise 1 - Journal Entry Agent](../JournalEntries/README.md)
- [Exercise 2 - Accounting Accruals Agent](../Accruals/README.md)
- [Exercise 3 - AP/AR Clearing Agent](../APAR-Clearing/README.md)
