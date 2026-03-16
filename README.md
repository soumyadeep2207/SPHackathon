# Deal Support Hackathon — Sales Deal Support Agent

A Microsoft Copilot Studio agent built for the **SPHackathon 2026** that helps sales and delivery teams quickly access policies, draft deal documents, find approvers, and submit deals for approval — all powered by SharePoint as the primary data source.

## Overview

**Solution:** `DealSupportHackathon` (v1.0.0.1, managed)
**Agent name:** Sales Deal Support Agent (`cr1d7_salesDealSupportAgent`)
**Platform:** Microsoft Copilot Studio + Power Automate
**Data source:** SharePoint site `https://v7m6l.sharepoint.com/sites/SPHackathon2026`

The agent uses AI intent classification to route each user message to the right topic, and leverages SharePoint knowledge search, Adaptive Cards, and Power Automate flows to fulfil requests.

---

## Features

### 1. Policy Finder
Ask the agent about company policies across four categories:

| Category | Description |
|---|---|
| Discount / Pricing Approval | Maximum allowed discounts, approval thresholds, exception handling |
| Deal Governance / Exceptions | Deal review requirements, approval hierarchy, escalation rules |
| Security / Compliance | Data residency, PII handling, SOC2, security exception approvals |
| Delivery / Risk | Fixed bid delivery rules, risk review, project escalation procedures |

The agent searches the **Policy** SharePoint library and filters results by matching the `category` column to the user's selection, ensuring answers are always from the correct policy document.

### 2. Deal Document / SOW Generation
The agent helps users draft Statements of Work using Word templates stored in SharePoint. It collects key deal details (client name, deal value, terms) and populates the template via a **Power Automate** flow (`PopulateWordTemplate`), which uses the Word Online and SharePoint connectors.

### 3. Find Deal Approvers
Users can enter their deal size (CAD) and discount percentage into an Adaptive Card form. The agent calls a Power Automate flow (`FetchApprovers`) which looks up the appropriate approver(s) from SharePoint and returns their names in a formatted card.

### 4. Submit Deal for Approval
Users can submit a deal for approval directly from the chat. The agent collects:
- Customer name
- Deal size (CAD)
- Discount percentage

It then calls a Power Automate flow (`SubmitDealforApproval`) that creates an approval record in SharePoint and returns a link to the created item along with the name(s) of the approver(s) the request was sent to.

### 5. Intent Routing (AI Builder)
Every conversation first passes through the `FindPolicyCopy` topic, which uses an **AI Builder** model to classify the user's intent into one of:

| Intent | Routes to |
|---|---|
| `POLICY_QUERY` | Policy Finder topic |
| `APPROVAL_QUERY` | Fetch Approver Details topic |
| `SUBMIT_DEAL_FOR_APPROVAL` | Submit Deal for Approval topic |
| `DOCUMENT_GENERATION_REQUEST` | SOW / Document Generation topic |
| _(none matched)_ | Agent Identification topic |

---

## Solution Components

| Component | Type | Description |
|---|---|---|
| `cr1d7_salesDealSupportAgent` | Copilot Studio Bot | Main agent |
| `PopulateWordTemplate` | Power Automate Flow | Generates SOW from Word template |
| `SubmitDealforApproval` | Power Automate Flow | Creates SharePoint approval record |
| `FetchApprovers` | Power Automate Flow | Retrieves approver list by deal size/discount |
| AI Builder Model | `msdyn_aimodel` | Classifies user intent |

### Bot Topics

| Topic | Purpose |
|---|---|
| `ConversationStart` | Welcome message with Adaptive Card |
| 
| `FindPolicy` | SharePoint knowledge search for policy documents |
| `FetchApproverDetails` | Collects deal info, calls flow, displays approvers |
| `SubmitDealforApproval` | Collects deal info, submits approval via flow |
| `Escalate` | Routes unresolved questions to an escalation channel |
| `AgentIdentification` | Fallback when no intent is matched |
| `Greeting`, `Goodbye`, `ThankYou` | Conversational courtesy topics |
| `Signin`, `EndofConversation`, `ResetConversation`, `StartOver` | Session management |

---


---

## Architecture

```
User Message
     │
     ▼
FindPolicyCopy Topic
     │
     ├─ AI Builder Intent Classification
     │
     ├── POLICY_QUERY ──────────────► FindPolicy Topic
     │                                  └─ SharePoint Knowledge Search (Policy library)
     │
     ├── APPROVAL_QUERY ────────────► FetchApproverDetails Topic
     │                                  └─ Power Automate: FetchApprovers flow
     │
     ├── SUBMIT_DEAL_FOR_APPROVAL ──► SubmitDealforApproval Topic
     │                                  └─ Power Automate: SubmitDealforApproval flow
     │
     ├── DOCUMENT_GENERATION_REQUEST► CreateSOW Topic
     │                                  └─ Power Automate: PopulateWordTemplate flow
     │
     └── (no match) ────────────────► AgentIdentification Topic
```

---

## Deployment

This solution is distributed as a **managed** Power Platform solution package (`DealSupportHackathon_1_0_0_1_managed.zip`).

### Prerequisites
- Microsoft Power Platform environment with Copilot Studio enabled
- SharePoint site at `https://v7m6l.sharepoint.com/sites/SPHackathon2026` with:
  - **Policy** document library (with a `category` column)
  - **SOW Templates** document library
  - SharePoint list for approver lookup
- Power Automate connection references:
  - `shared_wordonlinebusiness` — Word Online (Business)
  - `shared_sharepointonline` — SharePoint Online
  - `shared_commondataserviceforapps` — Dataverse

### Import Steps
1. Go to **Power Platform Admin Center** → your target environment.
2. Navigate to **Solutions** → **Import solution**.
3. Upload `DealSupportHackathon_1_0_0_1_managed.zip`.
4. Map the connection references to existing connections (or create new ones).
5. Complete the import and publish the solution.
6. Open **Copilot Studio** and verify the **Sales Deal Support Agent** is available.
7. Test the agent using the built-in test chat pane.

---

## Usage Examples

| User says | Agent action |
|---|---|
| "What's the discount policy for large deals?" | Asks policy category → searches SharePoint Policy library → returns policy summary |
| "Who needs to approve a 10% discount on a $500,000 deal?" | Shows deal info form → calls FetchApprovers flow → displays approver card |
| "I need to submit my deal for approval" | Shows deal submission form → calls SubmitDealforApproval flow → returns approval link |
| "Generate an SOW for my deal" | Collects deal details → calls PopulateWordTemplate flow → returns document |
