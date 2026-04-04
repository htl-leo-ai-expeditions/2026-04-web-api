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
