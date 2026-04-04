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

## Where Students Commonly Struggle

| Struggle | How to Help |
|----------|-------------|
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

## Tone and Language

The exercise uses a direct, conversational tone rather than formal academic language. This is deliberate:

- **Reduces distance.** Students aged 18-20 engage more readily with text that talks *to* them rather than *at* them. Phrases like "Here's the twist" or "This is the main event" create momentum.
- **Models professional communication.** Real-world technical documents (design docs, RFCs, API guides) increasingly use clear, informal language. Formal stiffness is not a sign of quality.
- **Lowers perceived difficulty.** The exercise content is genuinely challenging. Approachable language prevents students from feeling intimidated before they even start reading the business rules.
- **Maintains precision where it counts.** The conversational tone applies to framing, transitions, and instructions — not to the business rules, the worked example, or the requirements checklist. Those stay precise and unambiguous.

**Teacher note:** If students perceive the tone as "too casual," the content will correct that impression quickly once they hit the design decisions and business rules.

## Connection to Broader Learning Objective

The exercise frames the LLD as a "prompt for an AI." This serves multiple purposes:

- **Motivates precision.** Students understand *why* they need to be specific — vague specs produce vague implementations, whether the implementer is human or AI.
- **Makes the skill transferable.** Writing clear specifications is useful whether students become developers, architects, product managers, or AI-assisted engineers.
- **Testable outcome.** A teacher (or student) can literally paste the LLD into an LLM and see if the generated code matches the spec. This makes quality tangible and self-assessable.
