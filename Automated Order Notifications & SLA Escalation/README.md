# n8n Order Status & Delayed-Shipment Escalation Notifier

## Objective

Build an automated n8n workflow for GreenCart that routes customer orders by status, sends automated status update emails, and automatically escalates shipments delayed beyond the 2-day SLA boundary.

## Pipeline Stages

The workflow must consist of 7 core stages:

• **Stage 1:** Trigger: Manual Trigger / Webhook (accepting order_id, customer_name, email, order_status, days_in_status)

• **Stage 2:** Data Cleanup: Set node maps and trims incoming fields into a consistent object

• **Stage 3:** Status Routing: Switch node routes by order_status into Processing, Shipped, or Delayed

• **Stage 4:** Email Content Builder: Set nodes build subject & body text for each branch

• **Stage 5:** Delay Escalation Check: IF node on Delayed branch tests whether days_in_status > 2

• **Stage 6:** Escalation Email Builder: Set node creates internal warehouse alert when IF condition is True

• **Stage 7:** Send Email: Gmail nodes dispatch customer updates & warehouse escalation emails

• **Stage 8:** Google Sheets Logging: Append row per order: timestamp, order_id, status, days, escalated

## Deliverables

**1. Documentation**

• Problem statement & weekly manual effort savings

• Architecture breakdown (Trigger → Set → Switch → IF → Gmail)

• Why separate Set nodes per branch

• Escalation boundary & edge-case handling

**2. Required Screenshots**

• 1. Full Workflow Canvas (complete pipeline)

• 2. Switch Node configuration (all 3 conditions)

• 3. IF Node Escalation rule (days_in_status > 2)

• 4. Execution Log showing green status output

**3. Live Test Results**

Execute and verify 4 distinct test scenarios:

• **Test 1:** Processing: Set node output & Processing email sent

• **Test 2:** Shipped: Set node output & Shipping confirmation sent

• **Test 3:** Delayed (days_in_status = 4): IF True, customer Delay email + Warehouse alert sent

• **Test 4:** Delayed (days_in_status = 2): IF False, customer Delay email sent, NO warehouse alert

**4. n8n workflow**
