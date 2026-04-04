# AI Expedition Logbook

## Title
Iteratively Developing a REST API Design Exercise with AI

## Participants
- Rainer Stropek (Teacher)
- Claude, Anthropic's AI assistant (multiple sessions)

## Date
2026-04-04

---

## Goal
Can a teacher and an AI collaboratively develop a complete, high-quality student exercise — including the handout, didactical concept, worked examples, student simulations, and a feedback report prompt — through a series of short, focused prompts? The exercise should teach 4th-year vocational CS students to design a RESTful web API by writing a Low-Level Design (LLD) document precise enough for an LLM to implement.

---

## How We Worked Together

The entire exercise was built through **12 iterative prompts**, each given to a fresh AI session. The AI had no memory between sessions — it relied on the files in the repository (especially `progress.md`) to understand what had been done before. Each session read all existing files, made one focused improvement, and committed the result.

This approach mirrors how a teacher might refine a worksheet over multiple revision passes, but compressed into a single working session. The teacher steered *what* to focus on; the AI did the heavy lifting of reading, writing, and maintaining consistency across documents.

### The Iteration Cycle

Every AI session followed the same workflow (defined in `CLAUDE.md`):

1. Read all existing files to understand the current state
2. Decide on one focused improvement
3. Make the change (updating `exercise.md`, `didactical-concept.md`, and/or `goal.md` as needed)
4. Document what was done and why in `progress.md`

This "one focus per run" constraint was key — it prevented the AI from trying to do everything at once and kept each change reviewable.

---

## The Prompts — Step by Step

Below are the exact prompts given to the AI, in order, along with what each one produced.

### Prompt 1 — Bootstrap
> Do one iteration. Once done, commit with a meaningful commit message.

**What happened:** The AI chose the fitness studio domain ("FitBook"), wrote the complete first draft of `exercise.md` with 12 business rules, a 4-phase top-down workflow, and core/advanced requirement checklists. It also created `didactical-concept.md` explaining the pedagogical reasoning. This single prompt produced the entire skeleton of the exercise.

### Prompt 2 — Worked Example
> Do one iteration. Once done, commit with a meaningful commit message.

**What happened:** The AI added a fully specified worked example (`POST /api/v1/members`) showing the exact level of detail students should aim for — field-level validation, multiple error cases with JSON bodies, status codes, and business rule references.

### Prompt 3 — Design Decisions
> Do one iteration. Once done, commit with a meaningful commit message.

**What happened:** The AI added a "Design Decisions to Think About" section with seven open-ended questions (booking resource structure, qualification modeling, cancellation semantics, etc.) that scaffold the *thinking process* without giving answers.

### Prompt 4 — Polish & Deduplicate
> Do one iteration, but do not add things this time. Look at the current status with a fresh eye and improve, restructure, rephrase, reformat, remove duplication. Once done, commit with a meaningful commit message.

**What happened:** After three additive iterations, the exercise had accumulated redundancy. The AI removed a redundant "What You Will Practice" subsection, inlined the estimated effort, tightened verbose prose, and cleaned up `goal.md` by replacing tentative domain candidates with the definitive choice. No new content — purely structural improvement.

### Prompt 5 — Student-Friendly Tone
> Do one iteration. This time, focus on making the language of the exercise more appealing to students. Once done, commit with a meaningful commit message.

**What happened:** The AI rewrote framing text and transitions to use a direct, conversational tone. "Here's the twist: you won't write a single line of code." "Picture a small fitness studio around the corner." "15 well-designed endpoints beat 30 messy ones. Every time." The didactical concept got a new section explaining why conversational tone is a deliberate pedagogical choice.

### Prompt 6 — Length & Level Review
> Do one iteration. Critically reflect if the exercise has the correct length and level. If necessary adjust, extend, or remove aspect. Once done, commit with a meaningful commit message.

**What happened:** The AI identified that Phase 4 would take 5+ hours if students fully specified every endpoint — with most of that time spent on mechanical repetition (GET /members/{id} is structurally identical to GET /instructors/{id}). It added a time budget table and guidance allowing students to abbreviate structurally similar endpoints while still requiring full detail for endpoints with unique business logic.

### Prompt 7 — Clear Deliverables
> Do one iteration. This time, focus on providing clear instructions about deliverables to students (e.g. empty tables, checklists, etc.). Once done, commit with a meaningful commit message.

**What happened:** The AI added copy-ready templates (empty tables, skeleton structures) to each phase and a business rule coverage checklist in the submission section. This addresses blank-page paralysis — students who know *what* to write but stall on *how to format it*.

### Prompt 8 — Final Student/Teacher Review
> Give it a final polish. Put yourself in the shoes of a student and read the exercise. Does it make sense? Is it clear what to do? Does it have the right level? Put yourself in the shoes of a teacher — is the didactical concept clear and complete? If not, adjust. Once done, commit with a meaningful commit message.

**What happened:** The AI found and fixed a contradiction (worked example note said "aim for this detail on every endpoint" but Phase 4 said abbreviation is fine), rephrased an ambiguous intro in the Design Decisions section, and added a missing "Self-Check and Tips" section to the didactical concept.

### Prompt 9 — Student Simulation Testing
> Sometimes, one must do an exercise to really understand whether the instructions are clear. So, start three subagents. Derive three different student personas, one student who struggles with keeping up, one average student, and one student who is very good at the subject. In the subagents' prompts, describe the persona and give the subagent the exercise. Tell the subagents that they are NOT allowed to read any other files. The agents must generate a solution in a subfolder `solution-<basic|average|good>`. They must also give you a report about how they experienced the exercise, what they found difficult, what they found easy, and what they would change, etc. Read the reports carefully and improve the documents based on the feedback. Once done, commit with a meaningful commit message.

**What happened:** Three AI agents independently worked through the exercise as simulated students (struggling, average, strong). All three independently flagged the same pain points:
- Qualification matching (Rule 4) was the #1 issue — the class had no explicit `category` field
- All three wanted a second worked example for a complex endpoint
- PUT vs PATCH was never mentioned
- Overlap boundary cases were confusing

The AI then added a `category` field, a second worked example (`POST /classes/{classId}/bookings`), PUT/PATCH as a design decision, overlap boundary clarification, and effort estimates for advanced topics. The three sample solutions are preserved in `solution-basic/`, `solution-average/`, and `solution-good/`.

### Prompt 10 — Feedback Report Prompt
> Write a `report-prompt.md` file that contains a prompt that teachers can later use to generate feedback reports for students. The agent that will generate the report will receive all documents including that report-prompt.md file. The goal of the `report-prompt.md` file is that multiple agents generating feedback reports for different students come up with similar reports. The tonality, the structure, the level of detail, etc. should be similar across different agents. Once done, commit with a meaningful commit message.

**What happened:** The AI created a structured feedback report prompt with an exact 8-section template (completeness, business rule coverage, design quality, specification precision, strengths, areas for improvement, advanced topics, summary), tone guidelines, and word count bounds to ensure consistency across different AI reviewers.

### Prompt 11 — No AI Grading
> I made a mistake by not telling you that there MUST NOT be AI-based grading. The feedback reports should be qualitative and descriptive, not quantitative. Grades and/or points are later given by human teachers. Please update report-prompt.md accordingly. Once done, AMEND the previous commit with a meaningful commit message.

**What happened:** The AI updated `report-prompt.md` to explicitly prohibit grades, scores, points, or any quantitative assessment. The report is purely descriptive — human teachers retain full grading authority. This was an amendment to the previous commit, correcting an oversight in the teacher's original prompt.

### Prompt 12 — This Logbook
> Complete the AI Expedition Logbook (readme.md). Describe how we worked together to develop and refine the exercise step-by-step. Include the prompts so that other teachers can see how we did it.

---

## Results & Insights

### What Worked

- **Iterative prompting with one focus per run** kept each change small, reviewable, and high-quality. The AI never tried to boil the ocean.
- **`progress.md` as shared memory** across sessions worked remarkably well. Each new AI session could reconstruct the full history of decisions without any context from prior conversations.
- **Simulating students with different skill levels** (Prompt 9) was the single most valuable iteration. It surfaced real ambiguities that eight rounds of author-perspective editing had missed.
- **Steering without micromanaging.** Most prompts were just "do one iteration" — the AI chose *what* to improve based on its assessment of the current state. When the teacher wanted a specific focus (tone, length, deliverables), a one-sentence hint was enough.
- **The correction loop works.** Prompt 11 shows that mistakes are easily fixed — the teacher caught a missing constraint (no AI grading) and the AI amended it immediately.

### What Didn't Work

- **The AI couldn't catch all ambiguities from an author's perspective alone.** It took simulated student testing (Prompt 9) to find the qualification-matching gap, even though the AI had written and reviewed the rules eight times.
- **Additive iterations accumulate redundancy.** Without the explicit "clean up, don't add" prompt (Prompt 4), the exercise would have grown bloated. Teachers using this approach should schedule polish passes.

### Key Takeaways

- **Short, focused prompts beat long, detailed ones.** "Do one iteration" with a clear workflow definition (`CLAUDE.md`) produced better results than a detailed specification of what to change would have.
- **AI is excellent at maintaining consistency across documents.** When the exercise changed, the didactical concept and progress log were updated in the same commit — something a human author might forget.
- **Testing with simulated personas is a powerful technique.** It's like user testing for educational materials, and AI can do it in minutes.
- **The human steers, the AI builds.** The teacher's role was choosing *direction* (what to focus on next), *constraints* (no AI grading), and *quality gates* (simulate students, do a polish pass). The AI handled reading, writing, cross-referencing, and maintaining consistency.
- **Every prompt should have a clear intent.** The most productive prompts were the ones with a specific angle: "focus on tone," "focus on deliverables," "simulate students." Generic "improve it" prompts work too, but targeted ones produce more valuable iterations.

---

## Artifacts

### Files Produced

| File | Purpose |
|------|---------|
| `exercise.md` | The student handout — the primary deliverable |
| `didactical-concept.md` | Teacher guide explaining the pedagogical reasoning |
| `goal.md` | Learning objectives and scope boundaries |
| `progress.md` | Append-only iteration log (AI's shared memory) |
| `report-prompt.md` | Prompt for AI-generated qualitative feedback reports |
| `CLAUDE.md` | Agent instructions defining the workflow |
| `solution-basic/` | Sample LLD from simulated struggling student |
| `solution-average/` | Sample LLD from simulated average student |
| `solution-good/` | Sample LLD from simulated strong student |

### Tools Used
- **Claude** (Anthropic) — AI assistant, multiple sessions via Claude Code CLI
- **Git** — version control, one commit per iteration

---

## Next Steps

- Use the exercise in a real classroom and collect student feedback
- Run `report-prompt.md` on actual student submissions to test feedback consistency
- Consider having students paste their LLD into an LLM to see if it generates working code — closing the "LLD as prompt" loop
- Adapt the approach (iterative AI prompting with `CLAUDE.md` workflow) to develop exercises for other topics
