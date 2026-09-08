# n8n Order Status & Delayed-Shipment Escalation Notifier

## Objective

Build an automated n8n workflow for GreenCart that routes customer orders by status, sends automated status update emails, and automatically escalates shipments delayed beyond the 2-day SLA boundary.

## Pipeline Stages

The workflow consists of 8 core stages:

• **Stage 1:** Trigger: Manual Trigger / Webhook (accepting order_id, customer_name, email, order_status, days_in_status)

• **Stage 2:** Data Cleanup: Set node maps and trims incoming fields into a consistent object

• **Stage 3:** Status Routing: Switch node routes by order_status into Processing, Shipped, or Delayed

• **Stage 4:** Email Content Builder: Set nodes build subject & body text for each branch

• **Stage 5:** Delay Escalation Check: IF node on Delayed branch tests whether days_in_status > 2

• **Stage 6:** Escalation Email Builder: Set node creates internal warehouse alert when IF condition is True

• **Stage 7:** Send Email: Gmail nodes dispatch customer updates & warehouse escalation emails

• **Stage 8:** Google Sheets Logging: Append row per order: timestamp, order_id, status, days, escalated

## Deliverables

### **1. Documentation**

<img width="560" height="160" alt="Architecture-Breakdown" src="https://github.com/user-attachments/assets/fc3c9842-42c4-432e-8fb3-cb028c79308c" />

• **Problem statement & weekly manual effort savings**

GreenCart's support team checks each order's status individually and writes a separate email for every single one that takes about 2–3 minutes per order. With 100+ orders a day, that's ~29 hours/week just typing status emails. On top of that, nobody notices when an order gets stuck until a customer complains about it. 

This workflow fixes both problems: it writes and sends the right email automatically based on the order's status, and it catches delayed orders on its own the moment they've been stuck for more than 2 days, no more waiting for a complaint.

• **Architecture breakdown**

• **Why separate Set nodes per branch**

Each order status needs a completely different tone — Processing should sound reassuring ("we're on it"), Shipped should sound like good news, and Delayed needs to sound apologetic. Trying to cram all three into one node with a bunch of if/else logic inside a single expression would be a mess to read and a nightmare to fix later. Giving each branch its own Set node means you can open just the one you care about, see exactly what it says, and test it on its own without touching the others.

• **Escalation boundary & edge-case handling**

The rule is simple: if an order has been "Delayed" for more than 2 days, it escalates — meaning the warehouse team gets an internal alert email in addition to the customer's apology email. If it's been delayed for exactly 2 days or less, only the customer gets an email — no warehouse alert. 

As for messy data — if days_in_status comes in blank or isn't actually a number, the workflow doesn't crash or throw an error. It just treats that value as 0 so the order still moves through the system safely instead of getting stuck.

**2. Required Screenshots**

• 1. Full Workflow Canvas (complete pipeline)

<img width="648" height="241" alt="Full-Workflow-Canvas" src="https://github.com/user-attachments/assets/bd6fb04a-37ac-4967-a544-bdfe1ff5b545" />

• 2. Switch Node configuration (all 3 conditions)

<img width="948" height="363" alt="Switch-Node-Configuration" src="https://github.com/user-attachments/assets/b3e3ffe8-6719-4430-ac3c-eb88703cc83a" />

• 3. IF Node Escalation rule (days_in_status > 2)

<img width="944" height="346" alt="IF-Node-Escalation-Rule" src="https://github.com/user-attachments/assets/885afd8b-0c36-4957-be7a-5271a9c19389" />

• 4. Execution Log showing green status output

<img width="786" height="364" alt="Execution-Log" src="https://github.com/user-attachments/assets/fa6be86e-f363-4883-a849-275ebfb6f4c8" />

**3. Live Test Results**

Execute and verify 4 distinct test scenarios:

• **Test 1: Processing**

{ "order_id": "GC-1002", "customer_name": "Sarah Chen", "email": "sarah.chen@example.com", "order_status": "Processing", "days_in_status": 2 }

Set node output ("Processing Email"): subject and body confirming the order is being processed. Email sent: "Send Processing Email" fires to sarah.chen@example.com. No warehouse alert.
  
• **Test 2: Shipped** 

{ "order_id": "GC-1003", "customer_name": "Marcus Lee", "email": "marcus.lee@example.com", "order_status": "Shipped", "days_in_status": 1 }

Set node output ("Shipped Email"): shipping confirmation subject/body. Email sent: "Send Shipped Email" fires to marcus.lee@example.com. No warehouse alert.
  
• **Test 3: Delayed (days_in_status = 4): IF True, customer Delay email + Warehouse alert sent**  

{ "order_id": "GC-1001", "customer_name": "Taylor Chen", "email": "taylor.chen@example.com", "order_status": "Delayed", "days_in_status": 4 }

IF result: 4 > 2 → True. Emails sent: "Send Delayed Email" (apology to customer) and "Send Warehouse Email" (SLA breach alert to warehouse) both fire. Sheet row: logged via "Delayed-Escalated Log" — status= Delayed, days= 4, escalated= true (after applying the fix from earlier).
  
• **Test 4: Delayed (days_in_status = 2): IF False, customer Delay email sent, NO warehouse alert** 

{ "order_id": "GC-1005", "customer_name": "Daniel Khan", "email": "daniel.khan@example.com", "order_status": "Delayed", "days_in_status": 2 }

<img width="560" height="160" alt="Architecture-Breakdown" src="https://github.com/user-attachments/assets/439ef799-2f19-4517-831a-1737b28f12c1" />
IF result: 2 > 2 is false → False. Email sent: "Send Delayed Email" only (apology to customer). No warehouse alert sent. Sheet row: logged via "Delayed-Not Escalated Log" — status= Delayed, days= 2, escalated= false.
  
**4. n8n workflow**
