# Travel Planning Assistant

## Objective

Design and develop a practical AI agent that solves a real-world problem using AI APIs, Retrieval-Augmented Generation (RAG), LangChain, external tools, and a user interface.

## 1. Project Overview

- **The problem being solved:**
  
  Planning a trip means juggling scattered information — weather forecasts, currency conversions, budgeting, general destination research, and personal documents like itineraries or visa paperwork. This project pulls all of that into one place: you just ask, and the assistant figures out what it needs to check.

- **Target users:**
  
  Individual travelers planning a trip, who want quick answers about a destination without manually checking five different websites or apps to get there.

- **Expected input and output:**

  - **Input:** 
  
    You type a normal question — something like "What's the weather in Germany this week, and how much is $300 in EUR?" — and optionally hand it a travel document (an itinerary, a guide, a visa PDF). 

  - **Output:** 
    
    What comes back is one plain-English answer that pulls together whatever information was actually needed to answer it, and you can also see exactly which tools it used and what each one returned.

- **Why an AI agent is suitable for the use case:**
  
  The questions travelers ask are open-ended and often require combining multiple data sources (e.g. "should I pack a coat and can I afford this hotel in my currency?"). A single API or script can't handle that variety — An agent works better here because it can actually read the question, decide on its own which tool (or tools) apply, run them in whatever order makes sense, and stitch the results into one coherent answer.

## 2. Architecture

```
User (Gradio UI)
       │
       ▼
 run_agent(message, history)
       │
       ├── get_weather              -> External API 1: Open-Meteo API 
       ├── convert_currency         -> External API 2: Exchange Rate API 
       ├── calculate_trip_budget    -> Python Function (Custom Tool)
       ├── web_search               -> Additional Tool :DuckDuckGo 
       └── retrieve_docs            -> RAG pipeline over Chroma vector store
                                          │
                                          ├── Document loading (PyPDFLoader)
                                          ├── Text splitting (RecursiveCharacterTextSplitter)
                                          ├── Embedding generation (sentence-transformers(all-MiniLM-L6-v2))
                                          ├── Vector storage (Chroma, persisted to ./chroma_db)
                                          └── Similarity-based retrieval
```

The `ai_agent` recieves user's message, decides which tool(s) to call, executes them writes the final answer.

## 3. Tools and APIs Used

| Tool | Type | Source | Authentication required |
|---|---|---|---|
| `get_weather` | External API | Open-Meteo | free, no key |
| `convert_currency` | External API | Exchange Rate | free, no key |
| `calculate_trip_budget` | Custom Python Function | Local arithmetic | N/A |
| `web_search` | Additional Tool | DuckDuckGo | free, no key |
| `retrieve_docs` | RAG Tool | Chroma vector store + Sentence Transformer | free | 

## 4. Setup Instructions

1. Get a free Groq API key at https://console.groq.com/keys
2. Set it as an environment variable named assg_key: assg_key=your_groq_api_key_here
3. You can put this in a .env file in the same folder as the notebook.
4. Run the notebook's cells in order, top to bottom.
5. The last cell launches the Gradio app with share=True — this also prints a temporary public URL in addition to the local one, useful for testing from a phone or sharing with someone else, but it expires after a few hours and anyone with the link can use it while it's live.
6. Open URL in a browser and start chatting.

## 5. Example queries

- "What's the weather in Paris?"
- "How much is 150 USD in AUD?"
- "We are 2 people spending 5 days in Paris. Flight cost is $50/person, Hotel is $45/night, food is $25/day and activities are $30/day - What's my extimated budget?"
- "What are the visa requirements for a Pakistani citizen visiting Paris? (It uses web search)"
- "According to the itinerary I uploaded, what day do we arrive in Paris? (It uses RAG by uploading a document)"
- "Should I pack warm clothes for Rome next week, and how much is 200 USD in EUR? (It combines weather and currency tool)

## 6. Limitations

- Web search uses DuckDuckGo's free service, which can occasionally slow down or give thinner results than a paid search API would.
- The weather lookup can sometimes match the wrong city if two places share the same name in different countries — typing "City, Country" (like "Paris, France") avoids this.
- The RAG index is stored locally in `./chroma_db` and is not multi-user-isolated; in a shared deployment, uploaded documents from different users would need separate collections.
- The budget calculator uses simple flat daily averages and one flat "Misc expense" percentage. It doesn't adjust for things like a destination being more expensive in peak season, and one-time costs like flights aren't factored in unless you enter them separately.
- Tool selection quality depends on the underlying LLM. For questions with a lot going on at once, it sometimes misses part of the request — rephrasing or breaking the question into smaller parts usually fixes this.
