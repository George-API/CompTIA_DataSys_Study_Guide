# Prompt: Generate CompTIA DataSys+ DS0-001 Study Guide

**Required inputs (in context or attached):** `Objectives-Outline-Map.md`, `Exam-Objectives.md`, and `Exam-Outline.md`. The map is the primary driver for scope and structure; use the objectives and outline only as specified by the map.

---

## How to use the map

**For each domain (1.0–5.0):**

1. **Start from that domain’s section in Objectives-Outline-Map.md.** Do not pull objectives or outline bullets from another domain.
2. **Build the section from the map’s table:** For each row under “Exam-Objectives.md”:
   - **Objective** = which sub-objective (e.g., 1.1, 3.3) you’re covering.
   - **Verb / task** = how to structure the content (see “Content by verb” below).
   - **Key topics to cover** = checklist; every item here must appear in the study guide. Use Exam-Objectives.md for exact wording and sub-bullets only for these topics.
3. **Anchor with the map’s Exam-Outline bullets.** Ensure each bullet listed under “Exam-Outline.md” for that domain is clearly reflected (as a heading, summary, or subsection). Nothing from the outline may be dropped or moved to another domain.
4. **Set depth by the domain’s exam weight** (shown in the map heading, e.g. “- 24%”). Higher weight → more detail, examples, and “how to apply”; lower weight → concise, no filler.

**Referential integrity:** If it’s not in the map for that domain, don’t put it in that section. The map defines the only objectives and outline points that belong in each file.

---

## Task

Generate an optimized, exam-focused study guide for CompTIA DataSys+ DS0-001: **one Markdown file per domain** (5 files), using Objectives-Outline-Map.md as the single source of truth for what goes where.

**Output files:**

| File | Domain (from map) |
|------|-------------------|
| `01-Database-Fundamentals.md` | 1.0 Database Fundamentals - 24% |
| `02-Database-Deployment.md` | 2.0 Database Deployment - 16% |
| `03-Database-Management-Maintenance.md` | 3.0 Database Management and Maintenance - 25% |
| `04-Data-Database-Security.md` | 4.0 Data and Database Security - 23% |
| `05-Business-Continuity.md` | 5.0 Business Continuity - 12% |

---

## Content by verb (from map “Verb / task” column)

Use the map’s **Verb / task** for each objective row to choose structure:

| Verb / task | Study-guide structure |
|-------------|------------------------|
| **Compare and contrast** | Side-by-side comparison (table or bullets); when to use which; trade-offs. |
| **Given a scenario** | Decision steps; what to check first; recommended actions; “Given X, you should…” |
| **Explain** | Clear definition; cause/effect; “why it matters” for the exam. |
| **Describe** | Procedures or concepts to know; common pitfalls or exam traps. |

If the map says “Explain” for an objective, don’t write a scenario-style section for it; if it says “Given a scenario,” include concrete steps and choices.

---

## Content rules

1. **Scope = map only.** Cover every objective row and every outline bullet for that domain in the map. Include all tools, types, and examples mentioned in the map’s “Key topics” (and in Exam-Objectives for those topics). Do not add unrelated topics or mix domains.
2. **Exam-ready.** Optimize for core understanding (correct mental models), learning (concept → key points → example), and reference (scannable headings, bullets, tables, **bold** terms and acronyms).
3. **Exam-style hooks.** Where the map’s key topics imply scenario or “choose the best” questions (e.g., ORM impact, backup strategies, authentication policies), add a short **Exam tip** or **What they might ask** so the guide doubles as a question-type refresher.

**Tone:** Concise, authoritative, practical. Reader has ~2–3 years DBA or data systems experience and needs to fill gaps and lock in details to pass.

---

*Place all five MD files in the same folder as Objectives-Outline-Map.md, Exam-Objectives.md, and Exam-Outline.md.*
