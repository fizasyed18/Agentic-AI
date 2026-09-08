# Smart Inventory Reorder Assistant

## Objective 

Build an automated n8n workflow for a retail warehouse that monitors stock levels in real time, escalates urgen inventory shortages to the purchasing manager, and integrates an AI assistant with custom tool capabilities.

## Data Schema

• 1. Inventory Sheet

• 2. Suppliers Sheet

## Core Automation & Live Data Branching

The core workflow performs live monitoring and multi-branch routing:

• **Stage 1:** Live Sheets Trigger: Connect to Google Sheets live instead of static test data; configure a Schedule Trigger to run daily

• **Stage 2:** Status Routing (Switch Node): Route items into 3+ distinct branches based on Stock Status (In Stock, Low Stock, Out of Stock)

• **Stage 3:** Urgency Escalation (IF Node): On the Out of Stock branch, evaluate Units Left against a set threshold to flag critical shortages

• **Stage 4:** Purchasing Alert (Set & Notification): Generate an urgent escalation message addressed to the purchasing manager for critical shortages

• **Stage 5:** Branch Recombination (Merge Node): Recombine all active processing branches back into a single unified data stream

• **Stage 6:** Inventory Summary (Summarize Node): Total and aggregate all inventory items categorized by status

## AI Assistant & Tooling Integration

Built an intelligent conversational assistant to answer team inquiries:

• **Stage 1:** Chat Setup: Connect a Chat Trigger to an AI Agent equipped with a Chat Model and Memory for contextual conversations

• **Stage 2:** Direct Inventory Tool: Create a custom tool allowing the agent to read and query the Inventory Sheet directly

• **Stage 3:** Custom MCP Server & Tool: Build an MCP Server exposing the Suppliers Sheet, connected to n8n via an MCP Client Tool

• **Stage 4:** System Prompt Guardrail: Enforce a strict guardrail: the agent must never email a supplier without showing the draft first

• **Stage 5:** Written MCP Justification: A3-5 sentence written paragraph explaining why Suppliers lookup deserves MCP architecture over a direct tool

• **Multi-Agent Workflow:** - Implement a second AI Agent as a specialized sub-agent tool to draft reorder emails

• **External MCP Connection:** - Connect the same MCP Server to a second client outside n8n with proof of operation

# Documentation & Reflections

**MCP justification**

The Suppliers Sheet holds sensitive, low-volatility contact data — exactly the kind of resource that benefitsfrom being centralized behind a service boundary rather than wired into every workflow that needs it. Buildingit as an MCP Server means the supplier directory has one authoritative implementation, one place to updatecredentials, filtering logic, or rate limits, and one audit trail, instead of duplicating a Google Sheets node (andits OAuth credential) inside every agent that needs supplier data. Because MCP is a standard protocol, thatsame server can be reused by other clients beyond this n8n Agent — a support bot, an internal dashboard, ora second automation — without re-implementing the lookup, which directly enables the stretch goal of anexternal MCP client connection. It also cleanly separates concerns: the Inventory tool changes constantly (unitsleft, statuses) and is cheap to expose directly per-workflow, while the Suppliers tool is closer to sharedinfrastructure that other teams and tools should be able to depend on. Finally, routing supplier lookupsthrough a server makes it far easier to add guardrails later (e.g., redacting emails for unauthorized callers) at asingle choke point, rather than trying to enforce that policy inside every individual tool node.

**Failure Fix**

While wiring the Switch node's three outputs into a single Merge node, the workflow initially stalled afterevery run — the Summarize node showed only the In Stock and Low Stock counts, and Out of Stock itemsnever appeared in the final total. Inspecting each branch individually showed the IF node's true/false outputswere both still connected to Merge input 0 by default, instead of both being routed to the dedicated “Out ofStock” input (index 2) alongside the In Stock (index 0) and Low Stock (index 1) branches — so the critical andnon-critical Out-of-Stock rows were silently overwriting the In Stock rows on the same Merge input instead ofjoining the stream. The fix was to explicitly set numberInputs: 3 on the Merge node and re-point both IFoutputs (critical alert path and standard path) to input index 2, leaving In Stock on index 0 and Low Stock onindex 1. After that change, a test run correctly showed all 12 items recombined and the Summarize nodereported accurate per-status totals.

# Execution Screenshots

**Main Workflow**

<img width="812" height="223" alt="main_workflow" src="https://github.com/user-attachments/assets/7c714d57-5315-4450-bed7-7dd21fef3d7c" />

**AI Agent**

<img width="644" height="282" alt="ai_agent" src="https://github.com/user-attachments/assets/7799b538-2f54-42e2-bb7c-e59d9309db38" />

**MCP Server Workflow**

<img width="548" height="245" alt="mcp_workflow" src="https://github.com/user-attachments/assets/5ea31920-8ad8-463b-9fd4-b296a3d65afd" />
