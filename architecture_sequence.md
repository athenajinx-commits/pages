# Idea Bank – Architecture Diagrams

This document describes the end-to-end flow of the Idea Bank platform using a sequence diagram and a process flowchart.

---

## 1. Sequence diagram

## High-level flow

```mermaid
sequenceDiagram
    autonumber
    participant Producer as Mock Producer
    participant Kafka as Confluent Kafka
    participant Bridge as Flask Bridge API
    participant GCSArchiver as GCS Archiver
    participant GCS as Google Cloud Storage
    participant Frontend as Next.js Frontend
    participant User as User
    participant VertexAI as Vertex AI (Gemini)

    Note over Producer,Kafka: Event streaming
    Producer->>Kafka: Produce event (ideas-stream, every 30s)
    Kafka-->>Bridge: Consumer thread polls and updates STATE
    Kafka-->>GCSArchiver: Optional consumer archives to GCS

    Note over Frontend,Bridge: Dashboard (polling)
    loop Every 3 seconds
        Frontend->>Bridge: GET /api/ideas
        Bridge-->>Frontend: JSON (ideas, events_count, last_event)
    end
    Frontend->>User: Render Kanban and metrics

    Note over User,VertexAI: AI Chat
    User->>Frontend: Send question
    Frontend->>Bridge: POST /api/chat with question
    Bridge->>GCS: Fetch event context (events/ prefix)
    GCS-->>Bridge: JSON event data
    Bridge->>VertexAI: generate_content (context + question)
    VertexAI-->>Bridge: Text response
    Bridge-->>Frontend: JSON with response
    Frontend-->>User: Show AI reply
```

## Diagram legend

| Participant | Description |
|------------|-------------|
| **Mock Producer** | Python script (`producer.py`) that generates Epics/Features/Issues events and publishes to Kafka. |
| **Confluent Kafka** | Topic `ideas-stream`; events are consumed by the Bridge and optionally by the GCS Archiver. |
| **Flask Bridge API** | `app.py`: runs a Kafka consumer in a background thread (updates in-memory STATE) and serves REST endpoints. |
| **GCS Archiver** | Optional process `consumer_storage.py` that consumes from Kafka and uploads each event to GCS. |
| **Google Cloud Storage** | Stores archived events under `events/`; the chat engine reads this as context for Gemini. |
| **Next.js Frontend** | Polls `/api/ideas` for the dashboard; sends questions to `/api/chat` for the AI assistant. |
| **Vertex AI (Gemini)** | RAG-style chat: context from GCS + user question → Gemini → plain-text answer. |

## 2. Process flow (flowchart)

The flowchart below shows the main process flows: event streaming, dashboard updates, and AI chat.

```mermaid
flowchart TB
    subgraph producerFlow [Producer process]
        P_Start([Producer starts])
        P_Gen[Generate mock event]
        P_Send[Publish to ideas-stream]
        P_Wait[Wait 30 seconds]
        P_Start --> P_Gen --> P_Send --> P_Wait --> P_Gen
    end

    subgraph kafkaLayer [Kafka layer]
        Kafka[(ideas-stream topic)]
    end

    subgraph bridgeFlow [Bridge API process]
        B_Poll[Consumer thread polls Kafka]
        B_Update[Update or append idea in STATE]
        B_Serve[Serve GET /api/ideas and POST /api/chat]
        B_Poll --> B_Update --> B_Poll
        B_Serve --> B_Poll
    end

    subgraph archiverFlow [GCS Archiver process]
        A_Poll[Consumer polls Kafka]
        A_Upload[Upload event JSON to GCS]
        A_Poll --> A_Upload --> A_Poll
    end

    subgraph dashboardFlow [Dashboard flow]
        D_Open([User opens dashboard])
        D_Poll[Frontend polls GET /api/ideas every 3s]
        D_Render[Render Kanban and metrics]
        D_Open --> D_Poll --> D_Render --> D_Poll
    end

    subgraph chatFlow [AI Chat flow]
        C_Ask([User sends question])
        C_Post[Frontend POST /api/chat]
        C_Load[Load context from GCS]
        C_Gemini[Call Vertex AI Gemini]
        C_Return[Return reply to user]
        C_Ask --> C_Post --> C_Load --> C_Gemini --> C_Return
    end

    P_Send --> Kafka
    Kafka --> B_Poll
    Kafka --> A_Poll
    D_Poll --> B_Serve
    C_Post --> B_Serve
    C_Load --> GCS[(GCS bucket)]
    GCS --> C_Load
```

### Flowchart summary

| Flow | Description |
|------|-------------|
| **Producer** | Loops: generate event → publish to Kafka → wait 30s. |
| **Bridge** | Background consumer updates in-memory STATE from Kafka; HTTP server serves dashboard and chat. |
| **GCS Archiver** | Optional process that consumes from Kafka and uploads each event to GCS. |
| **Dashboard** | Frontend repeatedly polls `/api/ideas` and re-renders the board and metrics. |
| **AI Chat** | User question → Bridge loads context from GCS → Gemini generates answer → Bridge returns text to frontend. |

---

## Diagram legend (sequence diagram)

| Participant | Description |
|------------|-------------|
| **Mock Producer** | Python script (`producer.py`) that generates Epics/Features/Issues events and publishes to Kafka. |
| **Confluent Kafka** | Topic `ideas-stream`; events are consumed by the Bridge and optionally by the GCS Archiver. |
| **Flask Bridge API** | `app.py`: runs a Kafka consumer in a background thread (updates in-memory STATE) and serves REST endpoints. |
| **GCS Archiver** | Optional process `consumer_storage.py` that consumes from Kafka and uploads each event to GCS. |
| **Google Cloud Storage** | Stores archived events under `events/`; the chat engine reads this as context for Gemini. |
| **Next.js Frontend** | Polls `/api/ideas` for the dashboard; sends questions to `/api/chat` for the AI assistant. |
| **Vertex AI (Gemini)** | RAG-style chat: context from GCS + user question → Gemini → plain-text answer. |

## Data flow summary

1. **Streaming**: Producer → Kafka → Bridge (STATE) and optionally → GCS Archiver → GCS.
2. **Dashboard**: Frontend polls Bridge; Bridge returns current STATE (no direct Kafka from browser).
3. **Chat**: Frontend → Bridge → chat engine loads context from GCS and calls Vertex AI; Bridge returns the model reply to the frontend.
