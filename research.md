# Hybrid RRF Search Pattern Benefits

## Overview

Our expert skills use hybrid search combining **vector (semantic) search** and **text (keyword) search** using **Reciprocal Rank Fusion (RRF)**. This document explains why this approach is superior to using either search method alone.

## The Problem with Vector Search Alone

| Weakness | Example |
|----------|---------|
| **Misses exact terms** | Searching "HIPAA" might return docs about "healthcare compliance" but miss docs that specifically mention "HIPAA" |
| **Dilutes acronyms** | "MFA" semantically relates to many concepts; exact match is often what you need |
| **Version/number blindness** | "TLS 1.2" vs "TLS 1.3" may appear semantically similar but are critically different |
| **Struggles with proper nouns** | "Terraform" vs "CloudFormation" - semantic similarity doesn't capture the distinction |

## The Problem with Text Search Alone

| Weakness | Example |
|----------|---------|
| **Misses synonyms** | Searching "authentication" won't find docs about "login" or "sign-in" |
| **No conceptual understanding** | "secure password storage" won't match "bcrypt hashing" |
| **Requires exact phrasing** | "container orchestration" won't match "Docker deployment management" |
| **Keyword stuffing sensitivity** | A doc mentioning a term once vs many times affects ranking inappropriately |

## How RRF Fusion Solves This

```sql
FROM rrf(
    vector_search(security_spec, 'authentication requirements for healthcare'),
    text_search(security_spec, 'authentication MFA password HIPAA', content),
    join_key => 'path'
)
```

### RRF Algorithm

- Ranks results from each search independently
- Combines ranks using: `score = Σ 1/(k + rank)` where k is typically 60
- Documents appearing high in **both** searches get boosted
- Documents appearing in **only one** search still appear, but ranked lower

### Fusion Benefits

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Query                                │
│         "cookie consent requirements for GDPR"                   │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│     Vector Search       │     │      Text Search        │
│  (Semantic Meaning)     │     │   (Exact Keywords)      │
├─────────────────────────┤     ├─────────────────────────┤
│ 1. privacy-policy.md    │     │ 1. cookie-usage.md      │
│ 2. data-protection.md   │     │ 2. gdpr-compliance.md   │
│ 3. cookie-usage.md      │     │ 3. privacy-policy.md    │
│ 4. user-rights.md       │     │ 4. consent-forms.md     │
└─────────────────────────┘     └─────────────────────────┘
              │                               │
              └───────────────┬───────────────┘
                              ▼
                    ┌─────────────────┐
                    │   RRF Fusion    │
                    └─────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Fused Results                               │
├─────────────────────────────────────────────────────────────────┤
│ 1. cookie-usage.md       (high in both → top rank)              │
│ 2. privacy-policy.md     (high in both → boosted)               │
│ 3. gdpr-compliance.md    (exact match → preserved)              │
│ 4. data-protection.md    (semantic relevance → included)        │
└─────────────────────────────────────────────────────────────────┘
```

## Real-World Benefits for Expert Specs

| Query Type | Vector Finds | Text Finds | RRF Result |
|------------|--------------|------------|------------|
| "cookie consent requirements" | Privacy docs, GDPR concepts | Exact "cookie" mentions | Both privacy AND cookie-specific docs ranked high |
| "ECS Fargate deployment" | Container docs generally | Exact "ECS" and "Fargate" matches | AWS-specific container docs prioritized |
| "WCAG 2.1 AA compliance" | Accessibility concepts | Exact version "2.1" matches | Correct version docs ranked highest |
| "Snowflake Terraform" | Data warehouse concepts | Exact "Snowflake" matches | Snowflake-specific IaC docs prioritized |

## Why This Matters for Expert Skills

### 1. Compliance Precision
"HIPAA" must match exactly - you can't substitute with "healthcare privacy". The text search ensures exact regulatory terms are found.

### 2. Technical Accuracy
"AES-256-GCM" needs exact matching alongside encryption concepts. Vector search alone might confuse encryption algorithms.

### 3. Completeness
Semantic search finds conceptually related content; text search ensures nothing with key terms is missed.

### 4. Ranking Quality
Documents that are both semantically relevant AND contain key terms rank highest, giving the most relevant results first.

## Configuration

Our `spicepod.yaml` enables both search methods:

```yaml
datasets:
  - name: security_spec
    embeddings:
      - column: content
        use: openai_embeddings    # Enables vector search
    columns:
      - name: content
        full_text_search:
          enabled: true           # Enables text search
```

## Query Pattern

All expert skills use this pattern:

```sql
SELECT path, content, fused_score
FROM rrf(
    vector_search(<dataset>, '<natural language query>'),
    text_search(<dataset>, '<important keywords>', content),
    join_key => 'path'
)
ORDER BY fused_score DESC
LIMIT 5;
```

### Best Practices for Queries

| Component | Guidance |
|-----------|----------|
| **Semantic query** | Use natural language describing what you're looking for |
| **Text keywords** | Include exact terms, acronyms, proper nouns, version numbers |
| **Limit** | Start with 5 results; increase if needed |

### Examples

**Security Query:**
```sql
vector_search(security_spec, 'authentication requirements for healthcare applications')
text_search(security_spec, 'authentication MFA password HIPAA session', content)
```

**Cloud Query:**
```sql
vector_search(cloud_spec, 'container deployment on AWS')
text_search(cloud_spec, 'ECS Fargate Docker container Terraform', content)
```

**UX Query:**
```sql
vector_search(ux_design_spec, 'accessible form design for users with disabilities')
text_search(ux_design_spec, 'accessibility WCAG forms input aria', content)
```

## Comparison Summary

| Approach | Precision | Recall | Best For |
|----------|-----------|--------|----------|
| Vector Only | Medium | High | Conceptual exploration |
| Text Only | High | Low | Exact term lookup |
| **Hybrid RRF** | **High** | **High** | **Production use** |

Hybrid RRF search provides the best of both worlds: the conceptual understanding of semantic search with the precision of keyword matching.
