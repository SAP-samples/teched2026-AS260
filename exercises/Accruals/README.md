# Exercise 2 - Accounting Accruals Agent

## 1. Purpose and Scope

This script describes the required steps to test the Accounting Accrual Agent. It covers the complete process from uploading the accruals policy handbook, through accrual proposal creation and review, to the final posting in SAP S/4HANA.

### What is the Accounting Accrual Agent?

The Accounting Accrual Agent is deployed as part of the Financial Closing Assistant. It reads an uploaded accruals policy handbook — a document that defines the rules, GL accounts, cost centers, and calculation methods for each accrual category — and applies large language model (LLM) reasoning to analyze historical GL account postings. Based on this analysis, the agent calculates the accrual amounts for each defined category and creates accrual proposals in SAP S/4HANA. Each proposal contains the full accounting data (debit and credit GL accounts, amount, cost center, document type, item description) and the agent's reasoning for the proposed amount. You can review the generated accrual proposals with the SAP Fiori app **Manage Accrual Proposals**.

The script covers the entire process from uploading the accruals policy handbook through accrual proposal creation, review, and posting. The required historical posting data is already available in the SAP S/4HANA Cloud system.

## 2. E2E Process Overview

The diagram below shows the complete process and where each individual test step fits in.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE 1 — POLICY HANDBOOK UPLOAD                                           │
│  Upload or replace the accruals policy handbook for a company code          │
│  The handbook defines GL accounts, cost centers, document types,            │
│  calculation methods, and accrual categories for each accrual type          │
│  Sub-cases: new upload | replace existing | display / download              │
└────────────────────────────────┬────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE 2 — REQUEST ACCRUAL PROPOSALS                                        │
│  Open a new Joule conversation; send a prompt to create accruals:           │
│  "@Financial Closing Assistant Create accrual proposals for                 │
│   company code XXHO for period YYYY/MM"                                     │
│  The agent acknowledges and begins processing                               │
└────────────────────────────────┬────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE 3 — AGENT PROCESSING                                                 │
│  Agent reads historical GL account postings and applies the calculation     │
│  method defined in the policy handbook for each accrual type:               │
│  ├── Electricity: average of current + 3 prior months' electricity expenses │
│  ├── Water:       average of current + 3 prior months' water expenses       │
│  └── Audit:       year-to-date cumulative based on estimated annual fee     │
│  Agent returns accrual proposals in Joule and confirms creation             │
└────────────────────────────────┬────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE 4 — REVIEW PROPOSALS IN FIORI APP                                    │
│  Open Manage Accrual Proposals app                                          │
│  For each proposal: verify GL accounts, amounts, cost centers, doc type     │
│  against the rules in the policy handbook                                   │
│  Review agent reasoning: how was the amount derived from historical data?   │
└────────────────────────────────┬────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  PHASE 5 — POST OR REJECT PROPOSALS                                         │
│  ├── POST (happy path)                                                      │
│  │   Select proposal → click Post (upper-right corner)                      │
│  │   Accrual FI document created in SAP S/4HANA                             │
│  │   Auto-reversal scheduled for first day of next period (reason code 05)  │
│  │   Exception: December audit accrual is NOT reversed                      │
│  └── REJECT (negative test)                                                 │
│      Delete proposals → re-run agent for a different period or scenario     │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 3. Test System & Access

| Item | Value |
|------|-------|
| Test System URL | [https://my441372.s4hana.cloud.sap/ui](https://my441372.s4hana.cloud.sap/ui) |
| User | AS260-0XX *(replace XX with your participant number)* |
| Password | Acce$$teched26 |

## 4. Preparation — Policy Handbook

### 4.1 What is the Accruals Policy Handbook?

The accruals policy handbook is a document (PDF) that defines the accrual posting rules for a company code. It specifies, for each accrual category, the GL accounts, cost centers, calculation method, document type, item description, and reversal behavior. When uploaded via Joule, the agent reads and stores this handbook and uses it as the basis for all subsequent accrual proposal generation.

The policy handbook for this test is the file **Accruals_Policy_All_Template_V2.pdf**, which is provided with the session materials. It defines four accrual types for company code XXHO.

Before running the test, you need to copy the handbook and adapt it to your company code and cost centers.

### 4.2 Policy Handbook Summary

The table below summarizes the four accrual types defined in the policy handbook. Use it to verify that the proposals generated by the agent are correct.

| Accrual Type | Expense Category | Debit GL Account | Credit GL Account | Cost Center | Tax Code | Doc Type | Item Description must include | Calculation Method | Reversal |
|-------------|-----------------|-----------------|------------------|-------------|----------|----------|------------------------------|--------------------|---------|
| Electricity | XXHO_Elctrcty | 63001000 (electricity expense) | 21725000 (accrued liabilities) | XXHO1301 | V0 | SA | "XXHO electricity accruals" | Average of electricity expenses posted in the current month and the past 3 months (GL account 63001000, doc type KR, description includes "electricity fee") | 1st day of next period, reason code 05 |
| Water | XXHO_Wtr | 63002000 (water expense) | 21725000 (accrued liabilities) | XXHO1201 | V0 | SA | "XXHO water accruals" | Average of water expenses posted in the current month and the past 3 months (GL account 63002000, doc type KR, description includes "water fee") | 1st day of next period, reason code 05 |
| Audit | XXHO_AUDIT | 65300000 (audit expense) | 24090000 (other provisions) | XXHO1301 | V0 | SA | "XXHO audit" | Year-to-date cumulative: estimate annual fee = last year's audit invoice × 1.02 (2% inflation). Allocate quarterly (Q1: 12%, Q2: 24%, Q3: 24%, Q4: 40%), divide each quarter evenly by 3 months. Post cumulative YTD amount each month. | 1st day of next period, reason code 05 — except December (December accrual is not reversed) |

> [!NOTE]
> **Note on company code:** The policy handbook uses XXHO as the example company code and XXHO1201 / XXHO1301 as cost centers. In the test prompts, replace XXHO with your participant company code. The agent will apply the policy rules to the company code you specify in the prompt and derive amounts from its historical postings.

> [!WARNING]
> **Data readiness check:** Before running the test, verify that historical GL account postings exist in your company code for the relevant accounts (63001000 for electricity, 63002000 for water, 65300000 for audit). Without historical data, the agent cannot calculate meaningful accrual amounts.

## 5. Recommended Test Execution Order

1. Phase 1 — Policy handbook upload — prerequisite for all sub-cases
2. Phase 2 — Create accrual proposals
3. Phase 3 — Review and verify accrual amounts against policy rules
4. Phase 4 — Post or delete accrual proposals

If you prefer to test only certain steps, go directly to the respective phase and check the mandatory prerequisites listed in that section.

## 6. Phase 1 — Policy Handbook Upload

| | |
|---|---|
| **Covers** | Upload or replace the accruals policy handbook for a company code |
| **Prerequisite for** | All subsequent phases — Phase 1 must be completed before requesting accrual proposals |
| **Policy file** | Accruals_Policy_All_Template_V2.pdf — provided with session materials |

### 6.1 Sub-case 1A: Upload new policy handbook (no existing handbook)

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 1** | **Action** | Open SAP Joule (click the diamond icon ◇ in the top-right area of the Fiori Launchpad). Always start a new conversation.<br><br>**IMPORTANT:** Before entering a prompt, make sure that **agentic thinking is DISABLED** in the Joule window. Remove it if necessary. |
| **Step 1** | **Check** | The new chat window opens. |
| **Step 2** | **Action** | Type the following message and send it:<br><br>`@Financial Closing Assistant Upload accruals policy handbook for company code XXHO`<br><br>(Replace XXHO with your actual company code.) |
| **Step 2** | **Check** | The agent responds that it is ready to upload an accruals policy handbook for the specified company code and asks the user to attach the policy document. |
| **Step 3** | **Action** | Click the attachment icon, select the policy handbook file (your accounting policy in PDF format created on the basis of Accruals_Policy_All_Template_V2.pdf), and send the message. |
| **Step 3** | **Check** | The agent confirms the policy handbook has been successfully uploaded and stored for the company code. |

### 6.2 Sub-case 1B: Replace existing policy handbook

There is always only one policy per company code which is valid in the system. So, if a new policy document is uploaded into the system, this will automatically replace the previous one.

> [!WARNING]
> **IMPORTANT:** Please adapt the policy document to your company code and always perform all steps in the test script for your company code only, so that other tests are not impacted.

## 7. Phase 2 — Request Accrual Proposals

Start a new Joule conversation and request the agent to create accrual proposals for your company code and the current fiscal period.

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 1** | **Action** | Open SAP Joule and start a new conversation. Make sure **agentic thinking is DISABLED**. Send the following prompt:<br><br>`@Financial Closing Assistant Create accrual proposals for company code XXHO for period YYYY/MM`<br><br>Replace XXHO with your company code and YYYY/MM with the target fiscal year and period (e.g., 2026/10 for October 2026). |
| **Step 2** | **Action** | Wait for the agent to complete the analysis and return the results. |
| **Step 2** | **Check** | The agent returns a summary of the accrual proposals created, listing for each accrual type: the accrual category name, the proposed accrual amount, and the GL accounts to be used. |

## 8. Phase 3 — Review Accrual Proposals in Fiori App

Navigate to the **Manage Accrual Proposals** app and verify that the proposals generated by the agent are correct.

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 3** | **Action** | Click the bell symbol to open the notification sent to you and from there navigate directly to the accrual proposals.<br><br>Alternatively, open the SAP Fiori app **Manage Accrual Proposals** (search in the Fiori Launchpad). Filter by your company code XXHO and the target period, then click **Go**. |
| **Step 3** | **Check** | The app displays a list of accrual proposals with status **Open**. One proposal is shown for each accrual category active in the policy handbook (Electricity, Water, Audit — depending on whether historical data is available for all three). |
| **Step 4** | **Action** | Open each proposal by clicking on it and review the accounting details. Cross-check each field against the policy handbook summary in Section 4.2 of this exercise. |
| **Step 4** | **Check** | For each proposal, verify the following fields against the policy handbook: GL accounts, cost center, document type, item description, and reversal settings. |
| **Step 5** | **Action** | Review the agent reasoning shown for each proposal. This explains how the proposed amount was derived from historical GL account postings. |
| **Step 5** | **Check** | The agent reasoning is consistent with the calculation method defined in the policy handbook. |
| **Step 6** | **Action** | Verify the accrual amounts by manually re-calculating them. Use the historical GL account data visible in the Fiori app or in the agent reasoning panel as input values. |
| **Step 6** | **Check** | The amounts proposed by the agent match (within rounding) the amounts you calculate manually using the policy rules. |

## 9. Phase 4 — Post or Reject Accrual Proposals

### 9.1 Sub-case 2C — Post accrual proposals (happy path)

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 7** | **Action** | In the **Manage Accrual Proposals** app, select one or more accrual proposals you have reviewed and verified. Click the **Post** button in the upper-right corner of the app. |
| **Step 7** | **Check** | The accrual FI document is created in SAP S/4HANA. The system displays a posting confirmation with the FI document number. The proposal status changes from **Open** to **Posted**. The posting is an accounting document with document type SA, posting date on the last day of the period, and an automatic reversal posting scheduled for the first day of the next period with reason code 05. |

### 9.2 Sub-case 2D — Reject / delete proposals (negative test)

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 7** | **Action** | Instead of posting, select one or more proposals and click **Delete**. This simulates a scenario where the accountant disagrees with the proposed amounts or grouping. |
| **Step 7** | **Check** | The selected proposals are deleted. No FI document is created. The list in the **Manage Accrual Proposals** app no longer shows the deleted proposals. |
| **Step 8** | **Action** | *(Optional)* Return to Joule, start a new conversation, and re-submit the same prompt to create accrual proposals for the same company code and period. |
| **Step 8** | **Check** | The agent creates new proposals. The **Manage Accrual Proposals** app shows fresh proposals in **Open** status for the same period, ready for review and posting. |

## 10. Notes & Hints

- **Always start a new conversation** in SAP Joule before each phase.
- **Agentic thinking must be disabled.** In each new conversation, before entering any prompt, verify that the "agentic thinking" toggle in the Joule panel is switched off. If it is active, disable it before proceeding.
- **Company code in the prompt vs. policy handbook:** The policy handbook (Accruals_Policy_All_Template_V2.pdf) uses XXHO as the reference company code. When running the test, use your own company code in the policy and in the prompt. The agent applies the policy rules to the company code you specify and reads historical postings from that company code.
- **Historical data prerequisite:** The agent calculates accrual amounts from historical GL postings. If an accrual type has no historical postings (e.g., no electricity invoices posted in the past 4 months), the agent may skip that accrual type or propose a zero amount. This is expected behavior and not an error.
- **Posting is irreversible (manual reversal):** Once a proposal is posted, the FI document is created in SAP. The automatic reversal is scheduled by the system for the first day of the next period.
- **Only one policy handbook per company code** can be active at a time. Uploading a new handbook for the same company code triggers the Replace / Cancel workflow.

## Summary

You've now completed the Accounting Accruals Agent exercise. You can continue with any of the other agent exercises:

- [Exercise 1 - Journal Entry Agent](../JournalEntries/README.md)
- [Exercise 3 - AP/AR Clearing Agent](../APAR-Clearing/README.md)
- [Exercise 4 - Analytical Business Insights Agent](../AnalyticalBI/README.md)
