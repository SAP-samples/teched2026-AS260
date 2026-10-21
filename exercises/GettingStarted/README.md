# Getting Started

In this session, you are going to test the Financial Closing Assistant and the SAP AI agents which are part of it.

These are:

- Journal Entry Agent
- Accounting Accruals Agent
- Accounts Payable and Accounts Receivable Clearing Agent
- Analytical Business Insights Agent

You will be working in an SAP S/4HANA Cloud system and in Joule or Joule Work. You can start by looking at the Financial Closing Assistant in Joule Work and then dive into one of the agents by following the respective exercise.

## System Access

| Item | Value |
|------|-------|
| Test System URL | [https://my441372.s4hana.cloud.sap/ui](https://my441372.s4hana.cloud.sap/ui) |
| User | AS260-0XX@education.cloud.sap *(replace XX with your participant number)* |
| Password | Acce$$teched26 |

## First Steps

1. Login into the SAP S/4HANA system with the user and password indicated above.
2. Open Joule: you might want to stay in the Joule window or open Joule Work for more convenience.
3. Start a new conversation.
4. In the new conversation, please deactivate the **"agentic thinking"**.
5. Then start typing `@Financial Closing Assistant`.
6. Ask for example: *"I would like to close the month of October 2026 in the company code XXHO (where XX is your participant number), can you help me with that?"*

The assistant shall propose a series of actions which can be executed with the help of AI agents. These are, for example, clearing of customer and vendor open items, uploading journal entries, preparing accruals or analyzing financial statements KPIs.

You can also start directly by asking *"I would like to post accruals for the month of October 2026 in the company code XXHO"* (where XX is your participant number) or *"please, help me clear open items in the company code XXHO"* (where XX is your participant number). This way, you will directly ask Joule to trigger the accrual agent or the clearing agent respectively.

Depending on your interest, please proceed with one of the exercises on the individual agents:

- [Exercise 1 - Journal Entry Agent](../JournalEntries/)
- [Exercise 2 - Accounting Accruals Agent](../Accruals/)
- [Exercise 3 - AP/AR Clearing Agent](../APAR-Clearing/)
- [Exercise 4 - Analytical Business Insights Agent](../AnalyticalBI/)

It is not necessary to execute the exercises in a given sequence — you can select the one you are most interested in and start testing.
