# Didactical Concept: FitBook API Design Exercise

## Why This Domain

The fitness studio domain was chosen for several reasons:

- **Familiar context.** Most students have visited a gym or studio — they intuitively understand members, classes, and bookings without needing domain expertise.
- **Right-sized complexity.** Three core entities (members, instructors, classes) plus a booking relationship create enough design surface for a thorough exercise without overwhelming students.
- **Natural business rules.** Capacity limits, scheduling conflicts, qualification checks, and temporal constraints (no past bookings) arise organically from the domain. Students don't need to be told these are important — they feel obvious once stated.
- **Rich edge cases.** The domain naturally raises questions about concurrency (two people booking the last spot), idempotency (double-click on "Book"), and data integrity (deleting an instructor with future classes) — which serve the advanced tier.

## What Each Major Element Teaches

### Data Model (Phase 1)

| Element | Teaching Purpose |
|---------|-----------------|
| Three entities + join relationship | Students practice identifying resources and deciding what becomes its own entity vs. an attribute |
| Unique email constraints | Introduces the concept of natural keys and uniqueness validation |
| Instructor qualifications (list attribute) | Forces a decision: model as a separate entity, a JSON array, or a comma-separated string? Exposes data modeling trade-offs |
| Class category field | Makes Rule 4 (qualification matching) concrete. The explicit `category` field eliminates ambiguity about how to match a class to an instructor's qualifications — the match is category-to-qualification, not title-to-qualification. This was added after student testing revealed that all skill levels struggled when matching was left to the class title. |
| Membership status (active/inactive) | Simple state that drives authorization-like logic without requiring an auth system |

### Conventions (Phase 2)

This phase exists to teach students that API-wide consistency matters more than any single endpoint's design. Common mistakes:

- Skipping this step and ending up with inconsistent naming (e.g. `/api/members` and `/api/v1/Classes`)
- Defining error formats differently per endpoint
- Not thinking about date/time format until deep in endpoint design

**Teacher note:** If students struggle here, point them to a real-world API's documentation (e.g. Stripe) and ask them to extract the conventions.

### Endpoint Overview (Phase 3)

The table-first approach forces students to think about their API's surface area before getting lost in details. This is where students learn:

- CRUD mapping to HTTP methods
- URI design (nesting vs. flat, e.g. `/classes/{id}/bookings` vs. `/bookings?classId={id}`)
- That not every entity needs every CRUD operation

### Endpoint Details (Phase 4)

This is where the bulk of the work happens. Key learning moments:

- **Status code selection**: Students must choose between 400, 404, 409, 422, etc. — and justify their choice.
- **Validation rules in writing**: Forcing students to write out "email must be a valid email format" makes them realize how much implicit knowledge they usually leave unspecified.
- **Error scenarios**: Students typically forget the unhappy path. The business rules force them to think about what can go wrong.
- **JSON examples**: Writing concrete examples often reveals inconsistencies in the abstract specification.

### Design Decisions Section (between Business Rules and Task)

This section poses open-ended design questions that students will encounter during their work. It serves several pedagogical purposes:

- **Scaffolds without spoiling.** Students who freeze at "now design the API" get concrete angles to think about, but no answers are given — they must reason through trade-offs themselves.
- **Teaches that design involves choices.** Many students assume there is one correct REST design. The questions make it explicit that reasonable people disagree and that the key skill is *deciding and documenting*, not guessing the "right" answer.
- **Surfaces non-obvious complexity.** Questions like "what happens when capacity is reduced below current bookings?" force students to discover edge cases they would otherwise miss until implementation. This is exactly the kind of thinking an LLD should capture.
- **Models professional practice.** In real API design, teams discuss these trade-offs before building. The section mirrors a design review or RFC process.

**Teacher note:** If students treat these questions as a checklist to answer in a separate section, redirect them. The questions should inform decisions visible *throughout* the LLD (in conventions, endpoint specs, and data model), not be answered in isolation. A student who picked "bookings nested under classes" should show this consistently in their URI design, endpoint table, and examples.

### Worked Example (between Phase 4 and Requirements)

The worked example (`POST /api/v1/members`) serves as an **anchor point** — it answers the question "How detailed should I be?" more effectively than any instruction can. Pedagogical rationale:

- **Reduces ambiguity.** Students see the exact format and depth expected: field-level validation, multiple error cases with JSON bodies, explicit status codes, and a business rule reference.
- **Models good practice.** The example demonstrates server-generated fields (`id`, `createdAt`), default values (`membershipStatus`), and structured error responses with a `details` array — patterns students should adopt.
- **Prevents over-/under-specification.** Without an example, some students write one-liners ("POST creates a member") while others write pages of prose. The example calibrates expectations.
- **Deliberately uses `POST /members`** (the simplest write endpoint) so that it is easy to understand but still shows validation, uniqueness conflicts (409), and server-generated fields. It does not spoil the more interesting design decisions (booking logic, scheduling conflicts).

**Teacher note:** If students' work looks significantly less detailed than the worked example, point them back to it. If a student copies the example's conventions but applies them consistently to all endpoints, that is a sign of good work — they understood the template.

### Second Worked Example (Booking Creation)

A second worked example (`POST /api/v1/classes/{classId}/bookings`) was added after student testing revealed that all three skill levels wanted to see how a multi-rule endpoint should be documented. The member creation example is useful but relatively simple (one business rule). The booking endpoint enforces four rules simultaneously (active membership, no duplicates, capacity, future-only), which is representative of the complexity students will face in their own specs.

The second example also demonstrates:
- How one endpoint maps to multiple error responses, each tied to a specific rule
- The use of `403 Forbidden` for authorization-like checks (inactive member) vs. `409 Conflict` for state conflicts (capacity, duplicates)
- Nested resource URIs in practice (`/classes/{classId}/bookings`)

**Teacher note:** If a student's booking spec is significantly thinner than this example, it's a signal they're not thinking about all the error cases. Ask: "What happens if the member is inactive? If the class is full? If they already booked?"

## Pacing Guidance and the Repetition Problem

The exercise includes a phase-by-phase time budget and explicit guidance on handling repetitive endpoints. This addresses a real risk: without it, conscientious students spend 8+ hours writing nearly identical specs for structurally similar endpoints (e.g. every GET-by-id, every DELETE), learning nothing new after the third one. Meanwhile, the endpoints with genuinely interesting business logic (bookings, class creation with qualification/overlap checks) get rushed at the end.

The guidance tells students to:
- **Fully detail endpoints with unique business logic** (the ones that teach the most)
- **Abbreviate structurally similar endpoints** by referencing an established pattern, while still listing fields, status codes, and endpoint-specific validation
- **Never skip an endpoint entirely** — every endpoint from Phase 3 must appear in Phase 4

This preserves the exercise's completeness requirement (an LLM still needs to know what every endpoint does) while redirecting student effort toward the design decisions that actually matter.

**Teacher note:** Watch for two failure modes. (1) Students who abbreviate *everything* and produce a superficial spec — redirect them to the worked example's detail level for any endpoint that enforces a business rule. (2) Students who ignore the guidance and write 20 pages of copy-pasted boilerplate — tell them this is a sign they should extract shared patterns into their conventions section.

## Deliverable Templates and the Business Rule Checklist

Each phase now includes copy-ready templates (empty tables, skeleton structures) that students can paste into their LLD and fill in. A business rule coverage checklist is added to the submission section. This addresses several problems:

- **Reduces blank-page paralysis.** Students who know *what* to write but not *how to format it* often stall. A pre-structured table answers "what does this section look like?" before students have to think about content.
- **Sets expectations concretely.** Templates make the expected granularity visible — e.g. the entity table has columns for Type, Required, Unique, and Notes, signaling that students need to think about all of these per attribute.
- **Enforces completeness.** The business rule checklist asks students to map each rule to the endpoint(s) that enforce it. This makes gaps immediately visible and teaches traceability — a core practice in professional specification work.
- **Supports the "LLD as prompt" framing.** Templates produce structured, machine-readable output. An LLM can work with a filled-in table far more reliably than with free-form prose.

**Teacher note:** Some students will treat the templates as rigid constraints. Clarify that the templates are a starting point — students may add columns, split tables, or restructure as needed. What matters is that the information is present, not that it matches the template exactly. The abbreviated endpoint template is particularly important: it legitimizes brevity for repetitive endpoints while still requiring students to state what differs.

## Where Students Commonly Struggle

| Struggle | How to Help |
|----------|-------------|
| Confusion about overlap boundary cases (Rules 5/6) | The exercise now clarifies that touching boundaries are allowed (class ending at 11:00, another starting at 11:00 = no overlap). If students still struggle, ask them to draw a timeline with two example classes and check whether any minute belongs to both. |
| Not knowing whether to use PUT or PATCH | The design decisions section now explicitly raises this question. Neither is "wrong" — what matters is a documented, consistent choice. If a student uses PUT everywhere, that's fine for this exercise. If they use PATCH, they should specify which fields are patchable. |
| Jumping straight to endpoint details without data modeling | Remind them the phases exist for a reason. Ask: "What are your entities?" before "What are your endpoints?" |
| Inconsistent conventions across endpoints | Have them write the conventions section first and review it before moving on |
| Missing error cases | Ask: "What happens if the member doesn't exist? If the class is full? If the email is already taken?" |
| Vague specifications ("returns an error") | Ask: "Which error? What status code? What does the body look like?" |
| Over-engineering (adding auth, caching, deployment) | Redirect: the exercise is about API *design*, not infrastructure. Keep them focused on the LLD |
| Not knowing where to start | Point them to the Phase 1 deliverable. "Just list the entities and their attributes first." |

## Core vs. Advanced Split

The split serves two purposes:

1. **Protects weaker students** from being overwhelmed. The core requirements are achievable for everyone who has understood the lecture material. The checklist format makes progress visible.
2. **Challenges stronger students** without making the exercise longer. Advanced topics (pagination, concurrency, idempotency) add *depth* to the same endpoints, not *breadth*. A strong student's LLD for the same 15 endpoints will simply be more thorough.

The checklist format in the exercise makes it clear which items are required and which are stretch goals. Teachers should emphasize that a complete, clean core submission scores better than a half-finished one that attempts all advanced topics.

The advanced topics are now grouped by estimated effort (quick wins, medium, deep dives). This was added after student testing showed that stronger students wanted to attempt advanced topics but couldn't judge which ones were realistic in their remaining time. Without effort guidance, students either avoided all advanced topics or sank hours into a deep dive like OpenAPI while skipping easier wins like pagination.

## Tone and Language

The exercise uses a direct, conversational tone rather than formal academic language. This is deliberate:

- **Reduces distance.** Students aged 18-20 engage more readily with text that talks *to* them rather than *at* them. Phrases like "Here's the twist" or "This is the main event" create momentum.
- **Models professional communication.** Real-world technical documents (design docs, RFCs, API guides) increasingly use clear, informal language. Formal stiffness is not a sign of quality.
- **Lowers perceived difficulty.** The exercise content is genuinely challenging. Approachable language prevents students from feeling intimidated before they even start reading the business rules.
- **Maintains precision where it counts.** The conversational tone applies to framing, transitions, and instructions — not to the business rules, the worked example, or the requirements checklist. Those stay precise and unambiguous.

**Teacher note:** If students perceive the tone as "too casual," the content will correct that impression quickly once they hit the design decisions and business rules.

## Self-Check and Tips

The exercise closes with two lightweight tools: a five-question self-check and a short tips section.

- **Self-check questions** mirror the exercise's core quality criteria (implementability, business rule coverage, error handling, consistency, concrete examples). They serve as a final gate before submission — students who can honestly answer "yes" to all five have likely produced a solid LLD. The questions are deliberately framed as yes/no so students can't hide behind vague self-assessment.
- **Tips** reinforce the most common failure modes in a punchy, memorable format. "15 well-designed endpoints beat 30 messy ones" sticks better than a paragraph about scope management. The tip to study real-world APIs (Stripe, GitHub, Spotify) gives struggling students a concrete next step.

**Teacher note:** If a student's submission fails the self-check on multiple questions, use those questions as structured feedback — they point directly at what needs improvement.

## Connection to Broader Learning Objective

The exercise frames the LLD as a "prompt for an AI." This serves multiple purposes:

- **Motivates precision.** Students understand *why* they need to be specific — vague specs produce vague implementations, whether the implementer is human or AI.
- **Makes the skill transferable.** Writing clear specifications is useful whether students become developers, architects, product managers, or AI-assisted engineers.
- **Testable outcome.** A teacher (or student) can literally paste the LLD into an LLM and see if the generated code matches the spec. This makes quality tangible and self-assessable.
