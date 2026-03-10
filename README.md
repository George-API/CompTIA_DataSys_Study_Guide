# CompTIA DataSys+ DS0-001 Study Guide

![DataSys+ DS0-001 Exam Essentials](DataSys+_DSO-001_Infographic.png)

Exam-focused study guide mapped directly to the official CompTIA DataSys+ objectives and outline.

## Exam at a Glance

| | |
|-|-|
| Exam | DS0-001 |
| Questions | Up to 90 (MCQ + performance-based) |
| Time | 90 minutes |
| Passing | 700 / 900 |
| Experience | 2-3 years DBA |

## Repository Structure

```
├── guides/        One folder per domain guide (guide + systems-thinking pair)
├── 06-Cross-Domain-Review.md
├── 07-Visual-Study-Diagrams.md
├── 08-Terminology-Abbreviations.md
├── 09-Systems-Thinking-Guide.md
├── practice/      Practice questions
└── source/        Official exam objectives and generation prompts
```

## Study Order

### Phase 1 — Domain Guides (`guides/`)

| # | File | Phase | Purpose |
|---|------|-------|---------|
| 1 | `guides/database-fundamentals/guide.md` | DESIGN | Domain 1 - 24% |
| 2 | `guides/database-deployment/guide.md` | BUILD | Domain 2 - 16% |
| 3 | `guides/database-management-maintenance/guide.md` | OPERATE | Domain 3 - 25% |
| 4 | `guides/data-database-security/guide.md` | PROTECT | Domain 4 - 23% |
| 5 | `guides/business-continuity/guide.md` | SURVIVE | Domain 5 - 12% |

### Phase 2 — Reference & Review (root)

| # | File | Purpose |
|---|------|---------|
| 6 | `06-Cross-Domain-Review.md` | Consolidate connections, distinctions, sequences |
| 7 | `07-Visual-Study-Diagrams.md` | Visual reference diagrams for all domains (Mermaid) |
| 8 | `08-Terminology-Abbreviations.md` | Terminology, abbreviations, and quick-reference tables |

### Phase 3 — Systems Thinking (paired in `guides/` + integrated root guide)

| # | File | Phase | Purpose |
|---|------|-------|---------|
| 9 | `09-Systems-Thinking-Guide.md` | | Integrated systems model — causal chains, governing principles, decision engine |
| 10 | `guides/database-fundamentals/systems-thinking.md` | DESIGN | Domain 1 deep dive — design pipeline, abstraction ladder, decision trees |
| 11 | `guides/database-management-maintenance/systems-thinking.md` | OPERATE | Domain 3 deep dive — operations loop, signal system, change management |
| 12 | `guides/data-database-security/systems-thinking.md` | PROTECT | Domain 4 deep dive — defense system, threat map, encryption landscape |
| 13 | `guides/database-deployment/systems-thinking.md` | BUILD | Domain 2 deep dive — assembly line, testing taxonomy, validation gate |
| 14 | `guides/business-continuity/systems-thinking.md` | SURVIVE | Domain 5 deep dive — survival system, DR techniques, backup trust chain |

### Phase 4 — Practice (`practice/`)

| # | File | Purpose |
|---|------|---------|
| 0 | `practice/00-Practice-Questions.md` | Exam-style questions across all domains |

## Content Format

Each domain file follows the objective verb:

- **Compare and contrast** - tables, trade-offs, when to use which
- **Given a scenario** - decision steps, what to check first
- **Explain** - definitions, cause/effect, why it matters
- **Describe** - attack types, procedures, common pitfalls

Exam tips inline where question patterns are predictable.

## Source Material

All in `source/`:

- `Exam-Objectives.md` - official DS0-001 objectives (v5.0)
- `Exam-Outline.md` - high-level domain outline
- `Objectives-Outline-Map.md` - objectives-to-outline cross-reference by domain
- `Study-Guide-Prompt.md` - prompt used to generate the guide
- `Cross-Domain-Question-Bank.md` - question bank design spec
