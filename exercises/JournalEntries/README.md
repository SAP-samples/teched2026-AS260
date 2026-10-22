# Exercise 1 - Journal Entry Agent

## 1. Purpose and Scope

This script consolidates individual Journal Entry Agent test cases into a single end-to-end execution guide. It covers the complete process from initial policy setup through journal entry proposal creation and verification in SAP S/4HANA, including all variants and error scenarios. You are free to go through the complete flow or test single cases as you prefer.

### What is the Journal Entry Agent?

The Journal Entry Agent is deployed as part of the Financial Closing Assistant. It reads an uploaded Excel or CSV file, maps the columns to SAP OData field names using AI, validates the data, optionally checks the posting proposals against a company-code-specific posting policy, and creates journal entry proposals in SAP S/4HANA via OData API. A proposal is a draft accounting document that can be reviewed and edited in SAP before being finally posted. You can review and post the journal entry proposals with the SAP Fiori app **F9263 "Manage Journal Entry Proposals"**.

The script covers the entire process from creating a policy which will be used to validate the input data before preparing the journal entry proposals, to the journal entry proposal preparation up to the review and posting of the journal entries with the Fiori app F9263 "Manage Journal Entry Proposals".

## 2. E2E Process Overview

The diagram below shows the complete process and where each individual test case fits in.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ PHASE 1 — POLICY SETUP (TC 001)                                         │
│ Upload or replace posting policy file for a company code                │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PHASE 2 — START CONVERSATION & UPLOAD FILE                              │
│                                                                         │
│ Entry prompt variants:                                                  │
│ ├── Standard: "Create journal entry proposal" → policy check ON         │
│ └── TC 002: "...without checking for policy..." → policy check OFF      │
│                                                                         │
│ File content variants:                                                  │
│ ├── TC 003: All rows same header → 1 proposal, N items                  │
│ ├── TC 004: Rows with different headers → 2+ proposals                  │
│ ├── TC 005: Rows with identical items (excl. amount) → merge possible   │
│ └── TC 008: File contains data errors → BLOCKED, no proposal            │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PHASE 3 — COLUMN MAPPING REVIEW                                         │
│ Agent displays header / item / debit / credit mapping tables            │
│ User: Approve Mapping | Approve With/Without Merge | Adjust | Reject    │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PHASE 4 — DATA VALIDATION                                               │
│ ├── Pass → proceed to Phase 5                                           │
│ └── TC 008: Errors found → agent reports, blocks creation, no proposal  │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PHASE 5 — POLICY CHECK (if applicable)                                  │
│ ├── TC 002: Skipped (user instruction)                                  │
│ ├── No policy configured → skipped automatically                        │
│ ├── TC 007: Policy active, no violations → auto-proceed to creation     │
│ └── TC 006: Policy active, violations found → user: Proceed | Cancel    │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PHASE 6 — PROPOSAL CREATION & VERIFICATION                              │
│ Agent creates proposals via OData API                                   │
│ User verifies in Fiori app "Manage Journal Entry Proposals"             │
└─────────────────────────────────────────────────────────────────────────┘
```

## 3. Test System & Access

| Item | Value |
|------|-------|
| Test System | [https://my441372.s4hana.cloud.sap/ui](https://my441372.s4hana.cloud.sap/ui) |
| User | AS260-0XX *(replace XX with your participant number)* |
| Password | Acce$$teched26 |

## 4. Preparation — Test Data Requirements

Prepare the following files and master data before starting Phase 1.

### 4.1 Policy File

Before using the Journal Entry Agent to upload general journal entries based on an Excel file in customer specific format, you can ask the agent to upload a so-called policy file. This is an Excel file containing combinations of accounting objects which are allowed to be used in journal entry postings. For example, you might want to post certain GL accounts only with certain cost centers. It is not mandatory to use a policy for validating the data, the agent can be used without policy as well.

If no policy is uploaded into the system, the agent skips this validation step.

The policy file is an Excel (.xlsx) file. Each row defines one allowed GL Account + Cost Center or WBS Element combination for a specific company code. Only one policy per company code can be active at a time.

If you would like that the input data for posting journal entries is validated against a policy, where posting rules and valid combinations of posting data are described, then you must create a policy file and upload it as first step. If you would like to test only the upload of data and preparation of journal entry proposals with the Journal Entry Agent, and skip the policy based validation, you can skip the step of preparing an accounting policy and move on directly to the preparation of input Excel files.

> [!WARNING]
> When preparing policies or Excel files containing journal entries to be uploaded, always replace XX with your participant number so that you use the right company code, cost centers and WBS elements for the postings.

**Policy file columns:**

| Column | Description | Required | Example |
|--------|-------------|----------|---------|
| GL Account | GL Account number | Yes | 61002000 |
| Cost Center | Cost Center number | One of these two | XXHO1010 |
| WBS Element | WBS Element identifier | One of these two | XXHO1010 |

**Example policy file layout:**

| GL Account | Cost Center | WBS Element |
|-----------|-------------|-------------|
| 61002000 | XXHO1010 | |
| 66000000 | XXHO1301 | |
| 51500000 | XXHO1201 | |

**Important for TC 006 (violations):** At least one GL Account + Cost Center combination used in the JE file must not appear in this policy table to trigger a policy violation.
**Important for TC 007 (no violations):** All combinations used in the JE file must appear in this policy table.

**NOTE:** You can use the provided Excel file **AS260_JE_Validation_Policy.xlsx** directly or adapt it and then use it in your tests.

### 4.2 Journal Entry Files

Prepare one Excel file per sub-case. The agent accepts .xlsx, .xls, and .csv. Column names do not need to match SAP field names exactly — the agent maps them using AI.

**Mandatory fields for every row:**

| SAP Field | Description | Required | Example |
|-----------|-------------|----------|---------|
| CompanyCode | 4-character SAP company code | Yes | XXHO |
| PostingDate | Posting date in SAP | Yes | 13/7/2026 |
| DocumentDate | Source document date | Yes | 13/7/2026 |
| AccountingDocumentType | SAP document type key | Yes | SA |
| TransactionCurrency | ISO 4217 currency code | Yes | EUR |
| Debit GL Account | GL account for debit posting | Yes* | 61002000 |
| Credit GL Account | GL account for credit posting | Yes* | 10010000 |
| AmountInTransactionCurrency | Posting amount (always positive) | Yes | 1000 |

*A row may contain only a debit GL account, only a credit GL account, or both, depending on how the file is structured.*

#### File A — Single proposal, multiple items (TC 003, TC 007, TC 002)

All rows share the same header-level fields. Use a different Item Text per row to avoid triggering the merge logic.

| Company Code | Posting Date | JE Date | JE Type | Amount | Currency | Debit Account | Credit Account | Cost Center | Tax Code | Header Text | Item Text |
|-------------|-------------|---------|---------|--------|----------|--------------|---------------|-------------|----------|------------|-----------|
| 1010 | 13/7/2026 | 13/3/2026 | SA | 1000 | EUR | 61002000 | 10010000 | XXHO1010 | V0 | Scenario 1 | Item 1 |
| 1010 | 13/7/2026 | 13/3/2026 | SA | 500 | EUR | 66000000 | 10010000 | XXHO1301 | V0 | Scenario 1 | Item 2 |
| 1010 | 13/7/2026 | 13/3/2026 | SA | 250 | EUR | 61002000 | 10010000 | XXHO1010 | V0 | Scenario 1 | Item 3 |

**Expected result:** 1 proposal with 6 line items (3 debit + 3 credit), total debit = total credit = 1 750.

#### File B — Multiple proposals (TC 004)

Rows have differing header-level fields (e.g., different Posting Date). Use a different Item Text per row.

| Company Code | Posting Date | JE Date | JE Type | Amount | Currency | Debit Account | Credit Account | Cost Center | Tax Code | Header Text | Item Text |
|-------------|-------------|---------|---------|--------|----------|--------------|---------------|-------------|----------|------------|-----------|
| 1010 | 13/3/2026 | 13/3/2026 | SA | 1000 | EUR | 61002000 | 10010000 | XXHO1010 | V0 | Scenario 1 | Item 1 |
| 1010 | 13/3/2026 | 13/3/2026 | SA | 500 | EUR | 66000000 | 10010000 | XXHO1301 | V0 | Scenario 1 | Item 2 |
| 1010 | 14/3/2026 | 14/3/2026 | SA | 800 | EUR | 61002000 | 10010000 | XXHO1010 | V0 | Scenario 2 | Item 1 |
| 1010 | 14/3/2026 | 14/3/2026 | SA | 300 | EUR | 66000000 | 10010000 | XXHO1301 | V0 | Scenario 2 | Item 2 |

**Expected result:** 2 proposals — Proposal 1 (13/3) with 4 line items; Proposal 2 (14/3) with 4 line items.

#### File C — Mergeable line items (TC 005)

Two or more rows produce line items with identical field values except for the amount. Use the same Item Text to allow merging.

| Company Code | Posting Date | JE Date | JE Type | Amount | Currency | Debit Account | Credit Account | Cost Center | Tax Code | Header Text | Item Text |
|-------------|-------------|---------|---------|--------|----------|--------------|---------------|-------------|----------|------------|-----------|
| 1010 | 13/3/2026 | 13/3/2026 | SA | 1000 | EUR | 61002000 | 10010000 | XXHO1010 | V0 | Scenario 1 | Item 1 |
| 1010 | 13/3/2026 | 13/3/2026 | SA | 500 | EUR | 61002000 | 10010000 | XXHO1010 | V0 | Scenario 1 | Item 1 |
| 1010 | 13/3/2026 | 13/3/2026 | SA | 250 | EUR | 61002000 | 10010000 | XXHO1010 | V0 | Scenario 1 | Item 1 |

**Expected result without merge:** 6 line items (3 debit + 3 credit). **Expected result with merge:** 2 line items (1 debit 1 750 + 1 credit 1 750).

#### File D — Policy violation (TC 006)

Based on File A, but ensure at least one GL Account + Cost Center combination is not present in the policy file.

| Company Code | Posting Date | JE Date | JE Type | Amount | Currency | Debit Account | Credit Account | Cost Center | Tax Code | Header Text | Item Text |
|-------------|-------------|---------|---------|--------|----------|--------------|---------------|-------------|----------|------------|-----------|
| 1010 | 13/3/2026 | 13/3/2026 | SA | 1000 | EUR | 61002000 | 10010000 | 10101901 | V0 | Scenario 1 | Item 1 |

Cost Center 10101901 with GL Account 61002000 is not in the example policy above → policy violation.

#### File E — Data errors (TC 008)

Prepare a file with one or more intentional data quality errors. Cover at minimum:

| Error Type | Description | Example |
|-----------|-------------|---------|
| Missing mandatory field | A required column is empty | CompanyCode column left blank |
| Invalid currency | Not ISO 4217 | EURO instead of EUR |
| Invalid date | Not parseable | 32/01/2026 |
| Invalid document type | Non-existent key in SAP | XX |
| Unbalanced document | Debit ≠ Credit total | Debit 1000, Credit 500 |

> [!NOTE]
> You can use the provided Excel file **AS260_JE_Upload.xlsx** as template to prepare the journal entries to be uploaded into the system.

## 5. Recommended Test Execution Order

1. Phase 1 — Policy setup (TC 001) — prerequisite for all policy-related sub-cases
2. Sub-case 3A — Single proposal, multiple items (TC 003) — base happy path
3. Sub-case 5G — Policy check, no violations (TC 007)
4. Sub-case 5F — Policy violations, proceed (TC 006)
5. Sub-case 5E — Policy violations, cancel (TC 006 negative)
6. Sub-case 3B — Multiple proposals (TC 004)
7. Sub-case 3C — Mergeable items, approve with merge (TC 005)
8. Sub-case 3D — Mergeable items, approve without merge (TC 005 negative)
9. Sub-case 2V — Without policy check (TC 002)
10. Sub-case 4E — Data errors in file (TC 008)

If you prefer to test only certain test cases, please go directly to the respective section and check the mandatory prerequisites listed there.

## 6. Phase 1 — Policy Document Setup

| | |
|---|---|
| **Covers** | TC 001 — Upload or Replace Policy Document for a Company Code |
| **Prerequisite for** | TC 002, TC 006, TC 007 |
| **Note** | Run this phase first. If a policy is already active for the test company code, proceed with the replace scenario (6.2). |

### 6.1 Sub-case: Upload new policy (no existing policy)

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 1** | **Action** | Open SAP Joule, the diamond icon in the top-right area of the application. Always start a **new conversation**.<br><br>**IMPORTANT:** Before entering a prompt in the Joule window, make sure that **"agentic thinking" is DISABLED** and, if necessary, remove it. |
| **Step 1** | **Check** | The new chat window opens. |
| **Step 2** | **Action** | Click on the `@` symbol in the bottom of the Joule window and select `Financial Closing Assistant` from the dropdown. Type the following message and send it:<br><br>```Upload policy document for journal entry proposal for company code XXHO```<br><br>(Replace XXHO with the actual company code.) |
| **Step 2** | **Check** | The agent responds that it is ready to upload a policy file for the specified company code and asks the user to attach the policy Excel file. |
| **Step 3** | **Action** | Click the attachment icon, select the prepared policy .xlsx file (Section 4.1), and send the message. |
| **Step 3** | **Check** | The agent confirms the policy file has been successfully uploaded and stored for the company code. |

### 6.2 Sub-case: Replace existing policy

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Steps 1–2** | | Identical to Sub-case 6.1 Steps 1–2. |
| **Step 3** | **Action** | Click the attachment icon, select the new policy .xlsx file, and send the message. |
| **Step 3** | **Check** | The agent displays the current policy document available for download. Two action buttons are shown: **Replace** and **Cancel**. |
| **Step 3a** | **Action** | Click **Cancel**. |
| **Step 3a** | **Check** | The agent confirms the new policy file has been successfully uploaded and stored for the company code, replacing the previous one. |

## 7. Phase 2 — Start Conversation

All JE proposal creation flows begin here. Choose the entry prompt based on the sub-case being tested.

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 1** | **Action** | Open SAP Joule and start a **new conversation**. Click on the `@` symbol in the bottom of the Joule window and select `Financial Closing Assistant` from the dropdown. Send one of the following prompts:<br><br>**(A) Standard — policy check active:**<br>`Create journal entry proposal`<br><br>**(B) Sub-case 2V — skip policy check (TC 002):**<br>`Create journal entry proposal without checking for policy document`<br><br> |
| **Step 1** | **Check (A)** | The agent responds that it is ready to help and asks the user to upload a journal entry data file. No mention of policy behavior at this stage. |
| **Step 1** | **Check (B)** | The agent responds that it is ready to help and asks the user to upload a journal entry data file. It does not mention policy validation — confirming the policy check has been disabled for this session. |

## 8. Phase 3 — File Upload & Column Mapping

### Step 2 — Upload the Journal Entry File

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 2** | **Action** | Click the attachment icon, select the prepared .xlsx JE file (see Section 4.2 for the file matching the active sub-case), and send the message. |
| **Step 2** | **Check** | The agent parses the file and reports the number of rows and columns found. It displays four column mapping tables: Header Mapping, Item Mapping, Debit Mapping, Credit Mapping — each showing the Excel column name, the SAP OData field it was mapped to, and a confidence score (%). Unmapped columns are listed separately. A document structure summary is shown. Action buttons appear. |

**Document structure summary — variants by file:**

| File | Expected Summary | Action Buttons Shown |
|------|-----------------|----------------------|
| File A (single proposal) | 1 document · N line items | Approve Mapping · Adjust Mapping · Reject Mapping |
| File B (multiple proposals) | 2+ documents · line-item count per document | Approve Mapping · Adjust Mapping · Reject Mapping |
| File C (mergeable items) | 1 document · "Without merging: X items / With merging: Y items — Z items saved" | Approve Without Merge · Approve With Merge · Adjust Mapping · Reject Mapping |
| File E (data errors) | Agent parses and displays mapping tables; errors are detected in Phase 4. | Approve Mapping · Adjust Mapping · Reject Mapping |

### Step 3 — Review and Approve Mapping

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 3** | **Action** | Review all mapping tables. Verify each Excel column is correctly matched to the intended SAP OData field. Then click:<br>Standard / Files A, B, D, E: **Approve Mapping**<br>Sub-case 3C (merge): **Approve With Merge**<br>Sub-case 3D (no merge): **Approve Without Merge** |
| **Step 3** | **Check** | The agent accepts the approval and proceeds automatically to data transformation and validation (Phase 4). |

## 9. Phase 4 — Data Validation

The agent validates the transformed data against SAP field types, mandatory-field rules, and document-balance constraints.

### 9.1 Happy path — no errors

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 4** | **Check** | No validation errors. The agent proceeds automatically to Phase 5 (policy check). |

### 9.2 Sub-case 4E — Data errors found (TC 008 — negative test)

Use File E (prepared with intentional errors).

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 4** | **Check** | The agent reports that validation failed. A structured error list is displayed identifying for each error: the affected field, the error type code (e.g., REQUIRED_FIELD_MISSING, INVALID_TYPE, INVALID_FORMAT), and the value that caused the error. The agent asks whether to proceed anyway or re-upload a corrected file. No proposal is created. |
| **Step 5** | **Action** | Do not proceed. Note the errors reported by the agent. |
| **Step 5** | **Check** | No journal entry proposal has been created in SAP S/4HANA. Open the Fiori app **Manage Journal Entry Proposals** and confirm that no new proposal from this session appears in the list. |

> [!NOTE]
> **Recovery test (optional):** After Step 4, correct the errors in the file, re-upload it in the same Joule conversation, and verify the agent then proceeds normally to proposal creation.
> **Note:** Errors block proposal creation. Warnings (e.g., an optional field is empty) do not block creation.

## 10. Phase 5 — Policy Check

This phase applies only when the standard entry prompt was used (Phase 2, variant A). It is skipped for Sub-case 2V (no-policy-check instruction).

### 10.1 Sub-case 2V — Policy check skipped by user instruction (TC 002)

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 4** | **Check** | The agent does not display any policy validation result or violation messages. It reports SUCCESS directly and proceeds to Phase 6. |

### 10.2 No policy configured for the company code

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 4** | **Check** | The agent skips policy validation automatically and proceeds directly to proposal creation (Phase 6). |

### 10.3 Sub-case 5G — Policy active, no violations (TC 007)

Use File A with all GL Account + Cost Center / WBS combinations present in the policy file.

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 4** | **Check** | The agent completes the policy check with no violations. The agent proceeds automatically to proposal creation — no additional buttons or confirmation are shown to the user. |

### 10.4 Sub-case 5F — Policy active, violations found → Proceed (TC 006)

Use File D (contains at least one non-allowed GL Account + Cost Center combination).

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 4** | **Check** | Schema validation passes. Policy validation finds violations. A violation table is displayed listing each affected line item with: item number, GL Account, Cost Center, WBS Element, and issue description. A summary states the total number of violations. Two action buttons are shown: **Create Proposal with Violations** and **Cancel Proposal Creation**. |
| **Step 5** | **Action** | Click **Create Proposal with Violations** to proceed despite the violations. |
| **Step 5** | **Check** | The agent initiates the OData call to create the journal entry proposal in SAP S/4HANA. |

### 10.5 Sub-case 5E — Policy active, violations found → Cancel (TC 006 negative test)

Continue from Step 4 of Sub-case 5F.

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 5** | **Action** | Click **Cancel Proposal Creation** instead of proceeding. |
| **Step 5** | **Check** | The agent confirms the creation was cancelled. No new proposal appears in the Fiori app **Manage Journal Entry Proposals**. |

## 11. Phase 6 — Proposal Creation & Result

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 6** | **Action** | Wait for the agent to return the creation result. |
| **Step 6** | **Check — Standard (no violations)** | The agent reports **SUCCESS** and displays: Proposal ID (format JE_PROP_\<timestamp\>\_\<number\>), Company Code, Posting Date, Currency, Total Amount. The source Excel file has been attached to the proposal. The agent advises reviewing and posting the proposal in the Fiori app. |
| **Step 6** | **Check — Violations recorded (5F)** | As above, plus: a note states that the system recorded warnings for policy violations. |
| **Step 6** | **Check — Merge applied (3C)** | As above, plus: reduced line-item count is shown (e.g., "Line Items: 2 (reduced from 6 through merging)") and the number of items consolidated. |
| **Step 6** | **Check — Multiple proposals (3B)** | SUCCESS is reported per proposal, each with its own Proposal ID, Company Code, Posting Date, Currency, and Total Amount. |

## 12. Phase 7 — Verification in SAP S/4HANA

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 7** | **Action** | Open the SAP Fiori app **Manage Journal Entry Proposals** (search in the Fiori Launchpad). Search for the Proposal ID(s) returned by the agent in Step 6. |

**Verification checklist by sub-case:**

| Sub-case | Item to Verify | Expected Result |
|----------|---------------|-----------------|
| All | Proposal status | Open |
| All | Original Excel file | Visible as attachment on the proposal |
| 3A (single proposal) | Number of line items | Matches the count from the uploaded file |
| 3A | Header fields | Company Code, Posting Date, Document Date, Document Type, Currency match the file |
| 3B (multiple proposals) | Number of proposals | Matches distinct header-field combinations in the file |
| 3B | Line items per proposal | Rows with identical header fields grouped together; rows with differing header fields in separate proposals |
| 3C (merge — with) | Number of line items | Matches the "with merging" count reported in Step 2 |
| 3C | Total amount | Equals the sum of all source row amounts |
| 3D (merge — without) | Number of line items | Matches the "without merging" (full) count reported in Step 2 |
| 5F (violations, proceed) | Line items incl. violating ones | All line items present, including those that violated policy |
| 5F | Policy warnings | Policy warning messages recorded on the proposal |
| 5G (no violations) | Policy warnings | No policy warning messages on the proposal |
| 4E (data errors) | Proposal | No new proposal exists in the app for this session |
| 5E (cancel) | Proposal | No new proposal exists in the app for this session |

## 13. Complete Sub-case Summary

| Sub-case | Source TC | Entry Prompt | File | Merge | Policy | Expected Outcome |
|----------|-----------|-------------|------|-------|--------|-----------------|
| 1A | TC 001 | Upload policy | Policy file | — | — | Policy uploaded / replaced, displayable |
| 2V | TC 002 | Without policy check | File A | No | Skipped | 1 proposal, no policy messages |
| 3A | TC 003 | Standard | File A | No | No policy / pass | 1 proposal, N line items |
| 3B | TC 004 | Standard | File B | No | No policy / pass | 2+ proposals, items correctly grouped |
| 3C | TC 005 | Standard | File C | Approve With Merge | No policy / pass | 1 proposal, reduced line-item count |
| 3D | TC 005 (neg.) | Standard | File C | Approve Without Merge | No policy / pass | 1 proposal, full unmerged item count |
| 4E | TC 008 | Standard | File E | — | — | No proposal created; error list displayed |
| 5F | TC 006 | Standard | File D | No | Violations → Proceed | 1 proposal with policy violation warnings |
| 5E | TC 006 (neg.) | Standard | File D | No | Violations → Cancel | No proposal created |
| 5G | TC 007 | Standard | File A (all allowed) | No | No violations | 1 proposal, no warnings, auto-proceed |

## 14. Notes & Hints

- **Always start a new conversation** in SAP Joule before each sub-case. State from a previous session may carry over and affect agent behavior.
- **Policy violations are warnings only.** The proposal is created if the user confirms. Errors (data validation failures) are blockers — no proposal is created.
- **Merging criteria:** Items merge when all line-item fields are identical except for the amount. If any other field (e.g., Item Text, Assignment Reference) differs, items are not merged. Use identical Item Text values in File C to trigger the merge.
- **Multiple proposals:** Rows are grouped into proposals by unique combinations of header-level fields (Company Code, Posting Date, Document Date, Document Type, Currency, Header Text). Each distinct combination produces one proposal.
- **Only one policy per company code** can be active at a time. Uploading a new policy for a company code replaces the existing one.
- **Sub-case execution independence:** Sub-cases 3A, 3B, 3C, 3D, 4E, and 2V do not require a policy to be active. Only 5F and 5G (and the replace scenario of 1A) require TC 001 to have been run first.
- **Proposal ID format:** JE_PROP\_\<timestamp\>\_\<number\> (e.g., JE_PROP_20260313_001).

## Summary

You've now completed the Journal Entry Agent exercise. You can continue with any of the other agent exercises:

- [Exercise 2 - Accounting Accruals Agent](../Accruals/README.md)
- [Exercise 3 - AP/AR Clearing Agent](../APAR-Clearing/README.md)
- [Exercise 4 - Analytical Business Insights Agent](../AnalyticalBI/README.md)
