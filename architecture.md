# Expert Organization Architecture

## System Architecture Diagram

```mermaid
flowchart TB
    subgraph Stakeholders["Stakeholder Teams"]
        SEC[Security Team]
        CLOUD[Cloud Platform Team]
        UX[UX Design Team]
        QA[Quality Team]
    end

    subgraph GitHub["GitHub Repositories"]
        SECSPEC[(SecuritySpec)]
        CLOUDSPEC[(CloudSpec)]
        UXSPEC[(UXDesignSpec)]
        QASPEC[(QualitySpec)]
    end

    subgraph SpiceAI["SpiceAI Runtime"]
        INGEST[GitHub Ingestion]
        EMBED[OpenAI Embeddings]
        DS1[(security_spec)]
        DS2[(cloud_spec)]
        DS3[(ux_design_spec)]
        DS4[(quality_spec)]
        VECTOR[Vector Search]
        TEXT[Text Search]
        RRF[RRF Fusion]
    end

    subgraph Skills["AI Skills Layer"]
        SKILL1[security-expert]
        SKILL2[cloud-expert]
        SKILL3[ux-design-expert]
        SKILL4[quality-expert]
    end

    subgraph AIAssistants["AI Coding Assistants"]
        CLAUDE[Claude Code]
        COPILOT[GitHub Copilot]
    end

    DEV[Developers]

    SEC --> SECSPEC
    CLOUD --> CLOUDSPEC
    UX --> UXSPEC
    QA --> QASPEC

    SECSPEC --> INGEST
    CLOUDSPEC --> INGEST
    UXSPEC --> INGEST
    QASPEC --> INGEST

    INGEST --> EMBED

    EMBED --> DS1
    EMBED --> DS2
    EMBED --> DS3
    EMBED --> DS4

    DS1 --> VECTOR
    DS2 --> VECTOR
    DS3 --> VECTOR
    DS4 --> VECTOR
    DS1 --> TEXT
    DS2 --> TEXT
    DS3 --> TEXT
    DS4 --> TEXT

    VECTOR --> RRF
    TEXT --> RRF

    SKILL1 -.-> RRF
    SKILL2 -.-> RRF
    SKILL3 -.-> RRF
    SKILL4 -.-> RRF

    CLAUDE --> SKILL1
    CLAUDE --> SKILL2
    CLAUDE --> SKILL3
    CLAUDE --> SKILL4
    COPILOT --> SKILL1
    COPILOT --> SKILL2
    COPILOT --> SKILL3
    COPILOT --> SKILL4

    DEV --> CLAUDE
    DEV --> COPILOT
```

## Data Flow Sequence

```mermaid
sequenceDiagram
    participant ST as Stakeholder Team
    participant GH as GitHub
    participant SM as SpiceManager
    participant OAI as OpenAI API
    participant AI as Claude or Copilot
    participant DEV as Developer

    ST->>GH: Push markdown updates
    GH->>GH: Store in main branch
    SM->>GH: Pull md files via GitHub API
    GH-->>SM: Return file content
    SM->>OAI: Generate embeddings
    OAI-->>SM: Return vectors
    SM->>SM: Index for text search
    SM->>SM: Store in accelerated dataset
    DEV->>AI: Ask compliance question
    AI->>AI: Select appropriate skill
    AI->>SM: Execute hybrid RRF query
    SM->>SM: Vector search semantic
    SM->>SM: Text search keyword
    SM->>SM: RRF fusion scoring
    SM-->>AI: Return ranked results
    AI->>AI: Synthesize guidance
    AI-->>DEV: Provide answer with citations
```

## Hybrid RRF Query Flow

```mermaid
flowchart LR
    Q1[Natural Language Query]
    Q2[Keyword Query]
    VS[vector_search]
    TS[text_search]
    RRF[RRF Fusion]
    RESULTS[Ranked Results]

    Q1 --> VS
    Q2 --> TS
    VS --> RRF
    TS --> RRF
    RRF --> RESULTS
```

## Compliance Matrix

```mermaid
flowchart TB
    subgraph Frameworks["Compliance Frameworks"]
        HIPAA[HIPAA]
        HITRUST[HITRUST]
        SOC2[SOC2]
    end

    subgraph Apps["Application Types"]
        A1[Customer-facing with PHI]
        A2[Customer-facing no PHI]
        A3[Internal with PHI]
        A4[Internal no PHI]
    end

    A1 -->|Required| HIPAA
    A1 -->|Required| HITRUST
    A1 -->|Required| SOC2

    A2 -->|Required| HITRUST
    A2 -->|Required| SOC2

    A3 -->|Required| HIPAA
    A3 -->|Recommended| HITRUST

    A4 -->|Optional| HITRUST
```

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Source Control** | GitHub | Spec repositories, version control |
| **Runtime** | SpiceAI | Data federation, search, acceleration |
| **Embeddings** | OpenAI text-embedding-3-small | Vector generation for semantic search |
| **Search** | Hybrid RRF | Combined vector + keyword search |
| **AI Interface** | Claude Code, GitHub Copilot | Developer interaction layer |
| **Skills** | Markdown files | Skill definitions with query templates |

## How It Works

1. **Stakeholder teams** maintain compliance specs as markdown in GitHub repos
2. **SpiceManager** pulls markdown files and generates OpenAI embeddings
3. **Datasets** store both vector embeddings and text indices
4. **Hybrid RRF search** combines semantic and keyword search for best results
5. **AI Skills** define query templates for each domain expert
6. **Developers** invoke skills via Claude Code or GitHub Copilot
7. **AI assistants** return guidance with citations from the spec documents
