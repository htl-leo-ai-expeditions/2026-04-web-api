# Progress Log

## 2026-04-04 — Initial exercise draft: domain choice and full handout structure

**What:** Chose the fitness studio domain ("FitBook") and wrote the complete first draft of `exercise.md`. Also created the initial `didactical-concept.md`.

**Why:** The exercise was empty — the most impactful first step was selecting a domain and establishing the full structure. The fitness studio domain was chosen because it is familiar to students, has the right complexity (3 core entities + a many-to-many booking relationship), produces natural business rules (capacity, scheduling conflicts, qualifications), and offers rich edge cases for advanced students.

**Details:**
- `exercise.md`: Full handout with scenario, 12 business rules, a 4-phase top-down workflow (big picture → conventions → endpoint overview → endpoint details), core/advanced requirement checklists, self-check questions, and tips.
- `didactical-concept.md`: Teacher guide explaining domain choice, what each phase/element teaches, common student struggles with remediation suggestions, the core/advanced split rationale, and the LLD-as-prompt framing.
- `goal.md`: Unchanged (already solid).

## 2026-04-04 — Add worked example endpoint to exercise

**What:** Added a fully specified worked example (`POST /api/v1/members`) to `exercise.md`, placed between the Phase 4 description and the Requirements checklist. Also added a corresponding section to `didactical-concept.md` explaining its pedagogical purpose.

**Why:** Students consistently struggle with calibrating the level of detail expected in specification work. Instructions like "be specific" and "include examples" are too abstract. A concrete, fully worked-out endpoint — with request/response schemas, multiple error cases with JSON bodies, status codes, and business rule references — answers the question "how detailed should I be?" far more effectively. The example was deliberately chosen as `POST /members` because it is simple enough to understand quickly but still demonstrates validation, conflict handling (409), server-generated fields, and error response structure. It does not spoil the more interesting design decisions (booking logic, scheduling conflicts) that students should work through themselves.

**Files changed:**
- `exercise.md`: New "Worked Example" section with complete `POST /api/v1/members` specification.
- `didactical-concept.md`: New subsection explaining the worked example's role and how teachers can use it.

## 2026-04-04 — Add "Design Decisions to Think About" section

**What:** Added a new section to `exercise.md` between the business rules and the task description. It poses six open-ended design questions that students will face (booking resource structure, qualification modeling, cancellation semantics, class update edge cases, time zones, server-generated fields). Also added a corresponding entry in `didactical-concept.md`.

**Why:** The exercise already tells students *what* to design (business rules) and *how detailed* to be (worked example), but it doesn't scaffold the *thinking process*. Students who understand REST mechanics can still freeze when facing ambiguous design decisions with no single correct answer. The new section surfaces these decisions explicitly, helping students recognize that API design involves deliberate trade-offs — and that documenting those trade-offs is part of a good LLD. The questions are carefully chosen to: (1) not have obvious answers, (2) affect multiple parts of the LLD, and (3) connect back to specific business rules. No answers are given — students must reason and decide for themselves.

**Files changed:**
- `exercise.md`: New "Design Decisions to Think About" section with 6 guided questions and a note about documenting choices.
- `didactical-concept.md`: New subsection explaining the section's pedagogical purpose and a teacher note about how students should integrate (not isolate) their answers.

## 2026-04-04 — Polish pass: reduce redundancy and tighten prose

**What:** Cleanup and tightening across `exercise.md` and `goal.md` — no new content added.

**Why:** After three additive iterations, the exercise had accumulated some redundancy and verbosity. A fresh read revealed: the "What You Will Practice" subsection restated what the intro already covered; the "Estimated Effort" didn't need its own heading; the Design Decisions intro was wordy; the worked example closing note was verbose; the "Be consistent" tip duplicated the self-check consistency question; and `goal.md` still listed domain candidates despite the domain being chosen in iteration 1.

**Changes:**
- `exercise.md`: Removed the "What You Will Practice" subsection (redundant with intro). Inlined estimated effort as a bold line. Tightened the Design Decisions intro paragraph. Shortened the worked example closing note. Removed the "Be consistent" tip (covered by self-check item 4). Sharpened self-check items 3 and 4 for clarity.
- `goal.md`: Replaced the "Domain Choice (to be decided)" section with a definitive one-paragraph summary of the chosen domain and why.

## 2026-04-04 — Tone pass: make exercise language more appealing to students

**What:** Rewrote framing text, transitions, and instructions throughout `exercise.md` to use a more direct, conversational tone. Added a "Tone and Language" section to `didactical-concept.md`.

**Why:** The exercise content is solid but the language read like a formal specification — fine for a teacher guide, less engaging for 18-20 year olds encountering API design for the first time. Students are more likely to read carefully and stay motivated when the text talks *to* them rather than *at* them. The changes make the exercise feel approachable without sacrificing any precision in the business rules, worked example, or requirements.

**Changes:**
- `exercise.md`: Rewrote the introduction to be more engaging ("Here's the twist: you won't write a single line of code"). Made the scenario more vivid ("Picture a small fitness studio around the corner"). Added momentum to phase descriptions ("This is the main event"). Made tips punchier ("15 well-designed endpoints beat 30 messy ones. Every time."). Softened imperative language throughout while keeping instructions clear.
- `didactical-concept.md`: New "Tone and Language" section explaining why conversational tone is a deliberate pedagogical choice, not casualness.

## 2026-04-04 — Add pacing guidance and reduce repetitive busywork

**What:** Added a phase-by-phase time budget to the introduction and practical guidance in Phase 4 about handling repetitive endpoints. Updated the core requirements checklist to match. Added a new "Pacing Guidance and the Repetition Problem" section to `didactical-concept.md`.

**Why:** Critical review of the exercise's length revealed a problem: Phase 4 asks students to "fully specify every endpoint" at the worked example's detail level. For ~18 endpoints across 3 entities + bookings, that's 5+ hours of Phase 4 work alone — and much of it is mechanical repetition (GET /members/{id} is structurally identical to GET /instructors/{id}). This repetition doesn't teach anything new and risks students either burning out on boilerplate or rushing the genuinely interesting endpoints (bookings, class creation with overlap/qualification logic). The fix: students must still cover every endpoint, but they can abbreviate structurally similar ones by referencing an established pattern, reserving full detail for endpoints with unique business logic. The time budget helps students pace themselves and reinforces that Phases 1-3 deserve real investment.

**Files changed:**
- `exercise.md`: Added time budget table after "Estimated effort." Added "practical note about repetition" in Phase 4. Updated core requirements checklist wording.
- `didactical-concept.md`: New section explaining the pacing guidance rationale and two failure modes teachers should watch for.

## 2026-04-04 — Add deliverable templates and business rule coverage checklist

**What:** Added copy-ready templates (empty tables, skeleton structures) to each phase in `exercise.md`, plus a business rule coverage checklist in the submission section. Added a corresponding section to `didactical-concept.md`.

**Why:** The exercise told students *what* to deliver but not *what it should look like*. Students who understand the content requirements can still stall on formatting: "Do I write prose or a table? What columns should the table have? How do I structure an endpoint spec?" The templates answer these questions concretely. Specifically:
- Phase 1 gets an entity attribute table template (with Type, Required, Unique, Notes columns) and a relationship table.
- Phase 2 gets a conventions table, an error response JSON skeleton, and a status code table.
- Phase 4 gets both a full endpoint spec template and an abbreviated template for repetitive endpoints.
- The submission section gets a business rule coverage checklist where students map each of the 12 rules to the endpoint(s) that enforce it, making gaps immediately visible.

The templates also reinforce the "LLD as prompt" framing — structured tables are more machine-readable than free-form prose.

**Files changed:**
- `exercise.md`: Added templates to Phases 1, 2, and 4. Added business rule coverage checklist to submission section.
- `didactical-concept.md`: New section explaining the pedagogical purpose of templates and the coverage checklist.

## 2026-04-04 — Final polish: fix contradictions, fill didactical gaps

**What:** Read the full exercise from a student's perspective and the didactical concept from a teacher's perspective. Fixed issues found and filled gaps.

**Why:** After eight additive iterations, a fresh end-to-end read revealed a few issues:

1. **Worked example closing note contradicted Phase 4 guidance.** The note said "This is the level of detail you should aim for on every endpoint" — but Phase 4 explicitly tells students they can abbreviate repetitive endpoints. A student following both instructions would be confused. Fixed to say "on endpoints with unique business logic" and reference the Phase 4 abbreviation guidance.
2. **Design Decisions intro was slightly misleading.** "Don't try to answer these upfront" was ambiguous since students read the section before starting work. Rephrased to "You don't need to settle all of these before you start — you'll encounter them naturally as you work through the phases."
3. **Didactical concept had no coverage of the self-check and tips sections.** These are pedagogical tools a teacher should understand. Added a new section explaining the purpose of both and how to use the self-check as structured feedback.

**Files changed:**
- `exercise.md`: Fixed worked example closing note; rephrased Design Decisions intro.
- `didactical-concept.md`: Added "Self-Check and Tips" section.
- `progress.md`: This entry.
