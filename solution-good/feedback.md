# Experience Report -- FitBook Studio Exercise

**Student:** Lena M.  
**Date:** 2026-04-04

---

## 1. What parts of the exercise were clear and helpful?

- **The phased workflow (1-2-3-4) was excellent.** It genuinely prevented me from diving into endpoint details too early. I started with the data model, and by the time I got to Phase 4, I already had answers for 80% of the design questions. The exercise practices what it preaches about top-down design.

- **The worked example (POST /api/v1/members) set a clear quality bar.** Without it, I would have been guessing whether my spec was detailed enough. Having a concrete reference made it obvious what "good enough" looks like -- field tables, error examples, business rule references. I used it as a template.

- **The time budget was realistic and helpful for pacing.** Knowing that Phase 4 would take 3-5 hours prevented me from spending 2 hours perfecting my ER diagram. I appreciated the honesty that "Phase 4 is where the bulk of the work lives."

- **The business rules were crisp and numbered.** Being able to reference "Rule 7" in my endpoint specs instead of re-explaining the rule was efficient. The numbered list also made the coverage checklist straightforward.

- **The "Design Decisions to Think About" section was my favorite part.** It flagged exactly the trade-offs I would have stumbled into anyway, but framed them as choices rather than requirements. That framing felt respectful of my ability to reason through trade-offs rather than just follow instructions.

- **The templates for each phase** saved time. I didn't have to invent my own table structure or wonder what columns to include.

## 2. What parts were confusing or overwhelming?

- **Qualification matching for Rule 4 is underspecified.** The rule says "a Yoga class requires the instructor to have the Yoga qualification," but the class entity doesn't have an explicit "type" or "category" field -- it has a "title." So does "Morning Yoga Flow" count as a Yoga class? I had to invent my own matching logic (substring matching). I think the exercise should either (a) add an explicit class type/category field, or (b) explicitly state that students need to decide how title-to-qualification matching works. Right now it feels like it should be obvious, but it isn't.

- **The relationship between "design decisions" and "business rules" could be clearer.** The exercise lists 12 business rules and separately lists 6 design decisions. Some design decisions (like "what happens when capacity is reduced below current bookings") feel like they should be business rules too, but they're not numbered. I wasn't always sure whether I was expected to handle these edge cases or just think about them.

- **The "May Include (Advanced)" section lists 9 optional topics.** That's a lot, and they vary wildly in complexity. Pagination is a 20-minute addition; an OpenAPI spec is a multi-hour project. Some indication of relative effort would help students choose wisely.

## 3. Where did I get stuck and why?

- **Qualification modeling took me the longest to decide.** The exercise asks you to think about whether qualifications should be a separate entity, an enum, or free text -- but this decision cascades into how Rule 4 validation works, how the instructor endpoint looks, and whether you need extra CRUD endpoints. I went back and forth several times. I eventually chose a separate entity with its own endpoints, which added complexity but felt like the cleanest solution. A brief note about how this decision affects scope/complexity would have helped.

- **Nested vs. top-level bookings.** I wanted both a nested creation endpoint (POST under classes) and a top-level query endpoint (GET /bookings with filters). But the exercise presents this as an either/or decision. It took me a while to realize I could do both, and that doing both was actually the right call for the use cases described.

- **Concurrency handling.** I wanted to address the "two members book the last spot" scenario, but the exercise doesn't give much guidance on how deep to go. I ended up describing the database-level solution (conditional UPDATE) and optional ETag support, but I wasn't sure if that level of detail was appropriate or overkill.

- **Deciding between PUT and PATCH.** The exercise doesn't mention PATCH at all. I went with PUT (full replacement) for all updates because it's simpler and the resources are small. But I kept wondering if the exercise wanted me to consider PATCH for partial updates (e.g., just changing membership status).

## 4. What would you change about the exercise to make it better for students like me?

1. **Clarify the qualification-to-class matching mechanism.** Either add a `classType` field to the class entity, or explicitly call out that students must decide how to match a class to a qualification. This is a non-obvious design decision that the current text glosses over.

2. **Rank the advanced topics by effort.** Something like: "Quick wins: Pagination, Filtering/Sorting (~30 min each). Medium: Concurrency, Idempotency (~1 hour). Deep dive: OpenAPI Spec, HATEOAS (~2+ hours)." This helps students budget their remaining time.

3. **Add a short note about PUT vs. PATCH.** Even a one-liner like "You may choose PUT, PATCH, or both -- document your choice and why" would help. Right now the worked example uses POST and the templates mention PATCH in the coverage checklist, which sent mixed signals.

4. **Consider providing a sample OpenAPI snippet.** The advanced section mentions OpenAPI but doesn't show what it looks like. For students who have never seen an OpenAPI spec, a 10-line example would lower the barrier to trying it.

5. **The "Tips" section at the end is great but buried.** I'd move it closer to the top, maybe right after the scenario. By the time I reached it, I'd already finished most of my work.

6. **One more worked example would help** -- specifically for an endpoint with multiple business rules (like booking creation). The member creation example is useful but relatively simple. Seeing how to document Rule 7 + Rule 8 + Rule 9 all on one endpoint would have saved me some trial and error.

## 5. How long do you think this would realistically take you?

**My estimate: 6-7 hours for a thorough LLD with some advanced features.**

Breakdown:
- Phase 1 (data model + design decisions): ~1 hour. I spent extra time on the qualification modeling decision.
- Phase 2 (conventions): ~30 minutes. Straightforward once you've seen a few real APIs.
- Phase 3 (endpoint overview): ~20 minutes. Quick since the data model already implied most endpoints.
- Phase 4 (endpoint details): ~4 hours. The booking and class creation endpoints took the most time because of overlapping business rules. Repetitive endpoints (GET by ID, list endpoints) were fast once I had the pattern.
- Advanced features (pagination, concurrency, idempotency): ~45 minutes. Pagination was quick. Concurrency required some thinking.
- Review and polish: ~30 minutes.

For an average student, I'd estimate 7-9 hours. The design decisions section alone could stall someone for a long time if they're not used to making trade-offs without being told the "right" answer.

## 6. Difficulty Rating

**7 out of 10.**

**Why not lower:** The exercise requires genuine design thinking, not just following a recipe. The business rules interact in non-obvious ways (e.g., Rule 4 depends on how you model qualifications, which depends on a design decision that isn't a rule). The "Design Decisions" section forces you to make and justify trade-offs, which is harder than implementing code someone else designed. Also, the sheer number of endpoints (I ended up with 21) means Phase 4 is a marathon.

**Why not higher:** The domain is familiar and well-scoped. The phased approach provides enough scaffolding that you're never truly lost. The templates prevent "blank page" syndrome. The business rules are clearly stated, even if their interactions are subtle. And the exercise is honest about the time investment, which helps manage expectations.

**Bottom line:** This is one of the best-designed exercises I've encountered in school. It felt like real work -- the kind of document I'd actually write at a job -- rather than an artificial homework problem. The top-down approach and the "your LLD is the prompt" framing made the whole thing feel purposeful.
