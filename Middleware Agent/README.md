# ** Building a Support Desk across Three Architectures

## ** Stage 1 -- LangChain (create_agent)
Build a single agent with:
• At least 2 tools relevant to domain
• At least 1 built-in middleware (PII redaction, summarization, or a tool-call limit)
• Structured output for the final response, via a Pydantic schema

## ** Stage 2 -- LangGraph (multi-agent)
Rebuild Stage 1 as a supervised multi-agent graph with:
• 2-3 specialist nodes behind a supervisor
• A genuine cycle -- the supervisor routes back to itself at least once before finishing
• A checkpointer (thread-based memory)
• One raw interrupt() gating a sensitive action, resolved via Command(resume=...)

## **Stage 3 -- Deep Agents
Rebuild Stage 2's coordinator as a deep agent with:
• Subagents (deepagents' subagents=) instead of hand-rolled specialist nodes
• One memory file (AGENTS.md) with at least one house rule
• Structured output via response_format
