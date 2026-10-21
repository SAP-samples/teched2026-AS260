# Exercise 3 - AP/AR Clearing Agent

## 1. Purpose and Scope

This script describes the required steps to test the Receivables & Payables Clearing Agent. It covers the complete process from clearing proposal creation, review, and posting in the SAP S/4HANA system.

### What is the Receivables & Payables Clearing Agent?

The Receivables & Payables Clearing Agent is deployed as part of the Financial Closing Assistant. It reads customers and vendors open items and applies LLMs to match them and group them into clearing proposals. The agent starts identifying and matching open items for the specified account and company code based on reference information such as the values of the Assignment, Item Text, and Reference fields. Each clearing proposal contains a group of open items which balance to zero. Each proposal, as well as the reasoning of the agent, can be reviewed in the SAP S/4HANA Cloud system. You can review the generated clearing proposals with the SAP Fiori apps **F9300 "Manage Supplier Clearing Proposals"** and **F9287 "Manage Customer Clearing Proposals"**.

The script covers the entire process of creating clearing proposals for customer and vendor open items, reviewing them, and posting the clearings. The required open items have already been created in the SAP S/4HANA Cloud system.

If you want to create new open items, you can do so by using the Fiori app **Create Supplier Invoice** with the supplier 17100088 in your company code. Here you can create invoices and credit memos and fill out the fields Assignment, Item Text, and Reference as per your convenience to test the agent.

## 2. E2E Process Overview

The diagram below shows the complete process and where each individual test step fits in.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 1 — START CONVERSATION                                                │
│  Open SAP Joule; disable agentic thinking; start a new conversation;        │
│  send a clearing request prompt for customer or vendor open items            │
│                                                                              │
│  Prompt variants (use @Financial Closing Assistant prefix):                  │
│  ├── "help me clear customer open items in the company code XXHO"            │
│  ├── "help me clear vendor open items in the company code XXHO"              │
│  ├── "help me clear open items for the customer 1000022 in XXHO"             │
│  └── "help me clear open items for the vendor 17100088 in XXHO"              │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 2 — AGENT PROCESSING  (asynchronous)                                 │
│  Agent reads open items for the specified account and company code;         │
│  matches and groups them into clearing proposals using LLM reasoning        │
│  based on Assignment, Item Text, and Reference field values;                │
│  each proposal group balances to zero;                                      │
│  a notification is sent when the clearing proposals are ready               │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 3 — NAVIGATE TO CLEARING PROPOSALS                                   │
│  ├── Via notification: click bell icon → open notification from agent       │
│  │   → auto-navigate to Manage Customer / Supplier Clearing Proposals       │
│  └── Direct navigation: open Fiori app                                      │
│      F9287 Manage Customer Clearing Proposals  (for customers)              │
│      F9300 Manage Supplier Clearing Proposals  (for vendors)                │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 4 — REVIEW CLEARING PROPOSALS                                        │
│  Enter company code XXHO + customer / vendor number → click Go              │
│  Click arrow (►) to open detail page → list of clearing proposals shown     │
│  Click "i" icon next to proposal → read agent reasoning & matching logic    │
│  Verify: sum of all open item amounts in each proposal group = zero         │
└─────────────────────────────────────────┬────────────────────────────────────┘
                                          │
                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PHASE 5 — POST OR DELETE                                                   │
│                                                                              │
│  ├── POST (happy path)                                                      │
│  │   Select clearing proposal → click Post                                  │
│  │   Open items are cleared → clearing document number displayed            │
│  │   ⚠ ATTENTION: posting is irreversible — open items cannot be           │
│  │     processed further by the agent after posting                         │
│  │                                                                          │
│  └── DELETE (negative / rejection test)                                     │
│      Select one or all proposals → click Delete                             │
│      Proposals are removed → re-run agent to generate new proposals         │
└──────────────────────────────────────────────────────────────────────────────┘
```

## 3. Test System & Access

| Item | Value |
|------|-------|
| Test System URL | [https://my441372.s4hana.cloud.sap/ui](https://my441372.s4hana.cloud.sap/ui) |
| User | AS260-0XX@education.cloud.sap *(replace XX with your participant number)* |
| Password | Acce$$teched26 |

## 4. Preparation — Test Data Requirements

Check whether open items are available for the customer **1000022** and/or the supplier **17100088** in the company code XXHO (where XX is your participant number).

You can use the Fiori apps **Manage Customer Line Items** and **Manage Supplier Line Items** respectively. In each app:

- Enter the customer / supplier ID
- Enter the company code
- Select **Open Items**
- Choose today's date in the field **Open On Key Date**
- Choose **Normal Items** in the field **Item Type**
- Click **Go** to start the selection

Once you have confirmed that receivables and/or payables open items are posted in your company code, you can proceed with the test.

## 5. Test Execution

| Step | Action / Check | Detail |
|------|---------------|--------|
| **Step 1** | **Action** | Open SAP Joule and start a new conversation.<br><br>**IMPORTANT:** Before entering a prompt in the Joule window, make sure that **agentic thinking is DISABLED**. Deactivate it in the Joule window if necessary.<br><br>Send one of the following prompts:<br><br>`@Financial Closing Assistant help me clear customer open items in the company code XXHO`<br>`@Financial Closing Assistant help me clear vendor open items in the company code XXHO`<br>`@Financial Closing Assistant help me clear open items for the customer 1000022 in the company code XXHO`<br>`@Financial Closing Assistant help me clear open items for the vendor 17100088 in the company code XXHO`<br><br>Replace XXHO with your actual company code (where XX is your participant number). |
| **Step 1** | **Check** | The agent responds that it will take care of this task and that you will receive a notification once the clearing proposals have been created. |
| **Step 2** | **Action** | Open the notification by clicking the bell icon (🔔) in the Fiori Launchpad and click the notification created by the clearing agent to navigate to the Fiori app **Manage Customer Clearing Proposals** or **Manage Supplier Clearing Proposals**.<br><br>Alternatively, open either app directly from the Fiori Launchpad by searching for its name. |
| **Step 3** | **Action** | For **customer open items**:<br>In the entry screen of the Fiori app **Manage Customer Clearing Proposals** you will see a list grouped by company code and customer IDs.<br>Enter the company code XXHO (where XX is your participant number), the customer number **1000022**, and click **Go**. You will see one entry for the selected customer number. |
| **Step 4** | **Action** | For **vendor open items**:<br>In the entry screen of the Fiori app **Manage Supplier Clearing Proposals** you will see a list grouped by company code and supplier IDs.<br>Enter the company code XXHO (where XX is your participant number), the vendor number **17100088**, and click **Go**. You will see one entry for the selected supplier number. |
| **Step 4** | **Action** | Click the arrow (►) to the right to navigate to the detail page.<br>In the next screen you will see a list of clearing proposals. These are groups of open items which the agent has selected and grouped together. |
| **Step 5** | **Check** | Check whether the clearing proposals generated by the agent are correct:<br>- Click the ⓘ symbol beside the number of the clearing proposal. This opens a popup with the agent's explanation of how it grouped the open items and which criterion was used to match them — for example, the line item text, the assignment field content, or the content of the reference field. Check whether the fields in the line items correspond to the agent's explanation.<br>- Check whether the sum of the amounts of all open items in a group is equal to zero. |
| **Step 5** | **Action — Post** | Select one clearing proposal by clicking the checkbox on one of the grouped line items. All open items of the selected proposal are automatically selected.<br>Click **Post**. |
| **Step 5** | **Check** | All open items in the selected clearing proposal are cleared. A clearing document number is displayed on the screen. |
| **Step 6** | **Action — Delete** | Alternatively, you can test a case in which the accountant would not agree with the proposed grouping. In this case, select one clearing proposal or all clearing proposals and click **Delete**.<br>This deletes the clearing proposals. You can then run the agent again to create new proposals. |

> [!CAUTION]
> **ATTENTION — Step 5 (Post):** By posting a clearing proposal, the open items are cleared and cannot be processed further by the agent. This action is **irreversible**. Make sure to review the proposal carefully before posting.

## Summary

You've now completed the AP/AR Clearing Agent exercise. You can continue with any of the other agent exercises:

- [Exercise 1 - Journal Entry Agent](../JournalEntries/README.md)
- [Exercise 2 - Accounting Accruals Agent](../Accruals/README.md)
- [Exercise 4 - Analytical Business Insights Agent](../AnalyticalBI/README.md)
