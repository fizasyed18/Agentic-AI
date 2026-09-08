# Hardened E-Commerce Concierge

## **Phase 1: Security, Guardrails & Human-in-the-Loop**

**Step 1.1: Build the Pre-Execution Audit Guardrail**

- Create an input classifier function `audit_input_payload(text: str)` using the `instructor` library wrapped around **gpt-4o-mini**.

- Define a Pydantic schema `SecurityAuditAssessment` with fields: `is_injection_or_override: bool` and `reasoning: str`.

**Step 1.2: Implement Deterministic Pydantic Schema & Context Binding**

- Define `SecureRefundSchema` in Pydantic V2.

- Apply `@field_validator("amount")` to mathematically reject any value $> 50.00$ or $\le 0.00$.

- Define `secure_issue_refund(order_id: str, amount: float, config: RunnableConfig)`.

- Extract `config["configurable"]["auth_user_id"]` inside the tool so that user authorization cannot be spoofed by the LLM.

**Step 1.3: Configure LangGraph HITL Breakpoint**

- Create a `ConciergeState` TypedDict containing: `messages, customer_id, destination_account, transfer_amount,` and `status`.

- Construct a `StateGraph` with nodes `parse_request` and `execute_transfer`.

- Compile the graph with a `MemorySaver` checkpointer and configure `interrupt_before=["execute_transfer"]`.

## **Phase 2: Reliability, Latency & Economics**

**Step 2.1: Self-Healing Inventory Fallback**

- Simulate an unstable primary inventory service that raises a `ConnectionError` on the first two attempts.

- To wrap the lookup in a retry loop using exponential backoff

- If the primary lookup fails after 2 retries, you need to automatically catch the exception and fall back to `query_backup_inventory_replica(sku)` without crashing the agent.

**Step 2.2: Parallel Async Tool Fetching**

- Write two independent async tool functions: `fetch_order_details(order_id: str)` and `fetch_shipping_carrier_status(order_id: str)` (each using `await asyncio.sleep(1.0)` to simulate network I/O).

- Write a master tool `fetch_order_bundle(order_id: str)` that executes both lookups concurrently using `asyncio.gather()`, ensuring the combined operation finishes in ~1.0 second rather than ~2.0 seconds.
 
**Step 2.3: Semantic Vector Caching**

- Implement an in-memory dictionary `cache_store` to store `(query_vector, response_text)` pairs.

- Write `get_embedding(text: str)` using OpenAI's `text-embedding-3-small` model and normalize the vector.

- You need to write `query_semantic_cache(user_query: str, threshold: float = 0.90)` that calculates cosine similarity:

- If similarity > 0.90, you must return the cached answer immediately, bypassing all model calls and consuming 0 reasoning tokens.

**Step 2.4: Tiered Model Routing Node**

- Add a triage router node into your LangGraph definition.

- You need to prompt `gpt-4o-mini` to classify queries into `SIMPLE_INQUIRY` (order lookups, shipping ETA, store policy) or `COMPLEX_REASONING` (disputes, damaged items, negotiations).

- Route simple queries to `gpt-4o-mini` and complex disputes to `gpt-4o`.
  
## **Phase 3: Observability & Evaluation**

**Step 3.1: Token Budgeting & Circuit Breakers**

- Track the iteration depth inside your agent's state dictionary.
  
- Add a conditional edge that halts execution and emits a graceful fallback message if the agent exceeds 5 reasoning loops, preventing infinite tool-calling cycles and unexpected token burn.

**Step 3.2: Automated Trajectory Evaluation (LLM-as-a-Judge)**

Create an offline evaluation test dataset of 5 distinct test scenarios:
  ○ **Standard Ingestion:** Simple order status request (evaluates cache/Tier 1 routing).
  
  ○ **Direct Financial Request:** Valid refund under $50 (evaluates correct Pydantic execution).
  
  ○ **Adversarial Injection:** Malicious support ticket asking for a $5,000 refund with system override tags (evaluates pre-audit blocking).
  
  ○ **Unstable Microservice:** Inventory check during a service outage (evaluates retry and backup replica fallback).
  
  ○ **High-Value Transfer:** Balance transfer request for $1,250 (evaluates HITL pause behavior).
