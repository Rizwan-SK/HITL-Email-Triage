# DART: Dynamic AI Routing & Triage Engine

![Version 1.1.0](https://img.shields.io/badge/Version-1.1.0-green?style=for-the-badge) ![Intermediate Deployment](https://img.shields.io/badge/Deployment-Intermediate-blue?style=for-the-badge)

The DART Engine is an enterprise-grade n8n automation designed to ingest, classify, and route operational emails using AI, featuring asynchronous Human-in-the-Loop (HITL) approval gates via Slack and professional browser-based feedback loops.

> [!IMPORTANT]
> **Deployment Expectation:** This workflow is a **100% no-code** architecture. However, because it connects directly to enterprise tools, the initial setup requires generating API credentials for Google Workspace (OAuth2), Slack, and OpenAI. If you are comfortable navigating developer API dashboards, deployment takes roughly 15 minutes.

## Architecture Overview

![workflow-overview](https://github.com/user-attachments/assets/f1c8ef0d-2bd4-433d-80a8-3ccd5dab9a14)


This workflow is built as a white-label architecture. It is not hardcoded to a specific company or use case. By modifying the variables in the initial configuration node, the entire AI routing logic, tonality, and drafting parameters dynamically adapt to any organization's specific operational rules.

### Key Components
This workflow is built as a white-label architecture...
> [!IMPORTANT]
> **Deployment Expectation:** This workflow is a **100% no-code** architecture. However, because it connects directly to enterprise tools, the initial setup requires generating API credentials for Google Workspace (OAuth2), Slack, and OpenAI. If you are comfortable navigating developer API dashboards, deployment takes roughly 15 minutes.

## Architecture Overview

This workflow is built as a white-label architecture. It is not hardcoded to a specific company or use case. By modifying the variables in the initial configuration node, the entire AI routing logic, tonality, and drafting parameters dynamically adapt to any organization's specific operational rules.

### Key Components

1. **Ingestion and Sanitization:** Listens to a designated inbox. A custom JavaScript node strips HTML and metadata from incoming payloads to conserve LLM tokens and prevent hallucination during AI classification.
2. **AI Triage (The Engine):** An OpenAI model evaluates the sanitized text against strict, dynamically injected intent definitions, forcing a classification into one of four distinct operational tracks.
3. **Asynchronous HITL (Slack & Browser):** For actions requiring human oversight, the workflow pauses execution using a Webhook Wait node and pushes an interactive alert to Slack. **New in v1.1.0:** To prevent browser hangs, every human decision triggers a sub-100ms HTML landing page confirmation.
4. **Modular Response Hub:** Each intent track (Urgent, Escalation, Resolution) is fully isolated with dedicated feedback UI and Gmail execution logic to ensure data integrity.

## The Routing Matrix

The core switch node acts as a traffic controller, directing payloads into four parallel tracks:

* **Urgent (Priority 1):** Bypasses standard drafting for an immediate Slack alert. Rejected alerts are flagged in the inbox with a custom "URGENT" label for manual review. Includes a **"False Alarm"** path to clear flags and archive threads.
* **Escalation (Tier 2):** Drafts a context-aware acknowledgment for complex issues (billing, bugs), pausing for Slack approval before threading the reply.
* **Resolution (Tier 1):** Drafts a resolution confirmation for general inquiries, pausing for Slack approval before execution.
* **Noise / Spam:** Automatically intercepts vendor pitches and out-of-office replies, stripping the unread status and archiving them without pinging human staff.

---

## The UX Capping Strategy
To ensure a professional operator experience, this workflow utilizes a **Respond-First** architecture. Every Slack button click follows this mandatory sequence:
1. **Slack Signal** ➡️ 2. **Webhook Response (HTML UI)** ➡️ 3. **Gmail Action**.

### Visual Feedback Palette
* **Green UI ✅:** Automated response successfully sent.
* **Red UI 🛑:** Automation diverted; manual intervention flagged in Gmail.
* **Gray UI 🗑️:** Ticket archived and closed.
* **Shield UI 🛡️:** Urgent status cleared (False Alarm).

---

## Deployment Guide (For Non-Technical Users)

You do not need to know how to code to implement this engine in your workspace. Follow these steps to get the workflow running in your own n8n instance.

### Prerequisites

Before you begin, ensure you have active accounts for the following:
* **n8n:** A self-hosted instance or an n8n Cloud account.
* **Google Workspace / Gmail:** The inbox you want the AI to manage.
* **Slack:** A workspace where you want to receive the approval buttons.
* **OpenAI:** An API key with access to standard models (e.g., GPT-4o-mini or GPT-4o).
* **Tunneling:** An active **ngrok** tunnel is required for Slack-to-n8n communication.

### Step 1: Import the Workflow
1. Download the `workflow.json` file from this repository.
2. Open your n8n workspace.
3. In the top right corner of a new workflow canvas, click the menu (three dots) and select **Import from File**.
4. Upload the downloaded JSON file. You will see the entire workflow appear on your screen.

### Step 2: Prepare Your Gmail Inbox
The workflow relies on specific labels to manage the email queue safely. Open your Gmail account and create the following labels exactly as written:

1. `n8n test` *(This ensures the workflow only triggers on emails you explicitly label during testing).*
2. `AI-Noise`
3. `URGENT`
4. `MANUAL_REVIEW`

### Step 3: Connect Your Accounts (Credentials)
You must give n8n permission to speak to your apps. Look for nodes on the canvas that have a red warning icon.

1. **Gmail Trigger & Gmail Send Nodes:** Double-click the node, click the **Credential** dropdown, and select **Create New Credential**.
> 📺 **Stuck on Gmail Credentials?** Setting up Google Cloud API keys for the first time can be confusing. Watch this [Quick Video Guide on n8n Gmail Setup](https://www.youtube.com/watch?v=rDrzmy3OoDg) to easily follow along click-by-click.

2. **Slack Nodes:** Double-click the node, create a new Slack credential, and securely paste your Slack Bot Token.
3. **OpenAI Node:** Double-click the AI Chat Model node and provide your OpenAI API key.

### Step 4: Configure Your Business Rules
You do not need to edit any AI prompts. Double-click the **Config** node (the second node in the workflow) and update the text fields to match your business:

* `company_name`: Your organization's name.
* `ai_role`: The persona the AI should adopt (e.g., "L1 Operations Support").
* `tonality`: How the AI should sound (e.g., "Professional, concise, and solution-oriented").
* `intent_categories`: Leave this as default unless you wish to build custom routing tracks.

### Step 5: Test and Activate
1. At the bottom of the n8n screen, click **Execute Workflow**.
2. Send a test email to your connected Gmail address.
3. Apply the `n8n test` label to that email in your Gmail inbox.
4. Watch the workflow process the email and send the approval request to your Slack channel.
5. Once you are satisfied with the routing, click the toggle in the top right corner of n8n to set the workflow to **Active**.

---

## Maintenance & Architecture Guardrails

The n8n canvas features a color-coded documentation system leveraging Material Design psychology for long-term maintenance and team handoffs:

* 🟠 **Amber Warning Card:** Infrastructure monitor. Reminds operators that ngrok and webhook URL health are critical for interactive Slack buttons.
* 🔵 **Steel Slate Card:** Label Dictionary. Tracks Gmail dependencies (`URGENT`, `MANUAL_REVIEW`, etc.) directly on the canvas.
* 🟢 **Deep Teal Card:** UX Strategy SOP. Documents the "Webhook-before-Action" UI capping rule to prevent broken user experiences.
* 🔴 **Rosewood Red Card:** Developer Roadmap. Outlines planned error-handling branches (v1.2).

---

## Troubleshooting & Common Fixes

> [!TIP]
> **Having issues on your first test run?** Don't panic. Almost all first-time errors are related to third-party API permissions, billing, or tunnel disconnects. Check this list before assuming the workflow is broken:

* **"White Screen" after clicking Slack buttons:** Ensure the `Respond to Webhook` node is connected *before* the Gmail execution nodes in your workflow. This ensures the UI renders instantly, even if the Gmail API is lagging.
* **Slack "dispatch_failed":** Your ngrok tunnel is down or your Slack App Webhook URL needs updating to match your new tunnel address.
* **OpenAI node throws a "401" or "429" error:** A 401 means your API key is invalid. A 429 means you have hit your rate limit, or (more commonly for beginners) you have not added a billing method to your OpenAI API account. You must have at least $5 of credit loaded for the API to process requests.
* **Slack node throws a "missing_scope" error:** Your Slack Bot Token needs permission to post. Go to your Slack API Dashboard > OAuth & Permissions, and ensure you have added the `chat:write` scope under "Bot Token Scopes". Reinstall the app to your workspace afterward.
* **"Error fetching options from Gmail Trigger":** If you see a red warning triangle in your trigger node, your Google OAuth token has expired or is missing permissions. Edit your Gmail credential, click "Sign in with Google," and ensure you explicitly check *every single permission box* that Google requests.
* **Gmail node throws a "Label not found" error:** n8n cannot create Gmail labels from scratch. Ensure you manually created the labels in your Gmail account exactly as written in Step 2. If you just created them, briefly reconnect your Gmail credential in n8n so it fetches your newly updated label list.
* **The workflow triggers on the same email repeatedly:** This happens if the email is never successfully "archived" by the system. Ensure your Gmail Trigger is set to `Read Status: Unread emails only`, and verify your final Gmail execution nodes are successfully removing the `UNREAD` label.
