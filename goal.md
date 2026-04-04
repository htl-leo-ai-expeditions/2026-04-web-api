# Goal: RESTful Web API Design Exercise

## Context

This exercise is part of a 4th-year fullstack development course at a vocational college for Computer Science. Students have three years of programming experience (C#, TypeScript, Java, some C), understand relational databases and SQL, and have solid CS fundamentals.

## Primary Learning Objectives

1. **API Design Thinking**: Students learn to systematically design a RESTful web API — thinking about resources, URIs, HTTP methods, status codes, request/response schemas, validation, error handling, and data relationships.

2. **Writing a Low-Level Design Document (LLD)**: Students produce a structured document that fully specifies an API's behavior. The LLD should be precise, complete, and unambiguous — detailed enough that someone (or an LLM) could implement it without asking clarifying questions.

3. **AI Prompting through Specification**: By writing a high-quality LLD, students implicitly practice the skill of giving clear, structured instructions to an AI. The LLD *is* the prompt. The better the LLD, the better an LLM can implement it. This connection should be made explicit in the exercise.

## Secondary Learning Objectives

- Thinking about edge cases and error scenarios before writing code
- Understanding the relationship between data model and API surface
- Appreciating the value of upfront design work

## What the Exercise is NOT About

- Writing code (no implementation required)
- Learning a specific framework (no Express, ASP.NET, Spring, etc.)
- Deployment, CI/CD, or infrastructure
- Authentication/authorization in depth (may be mentioned but is not the focus)

## Deliverable

The student produces an LLD document covering (at minimum):
- Domain description and context
- Data model (entities, attributes, relationships)
- Endpoint specification (URI, method, request/response bodies, status codes)
- Validation rules
- Error response format
- Any constraints or business rules
- OpenAPI Specification for a subset of the API (optional but encouraged)

## Validation Criterion

A good LLD should pass this test: *"Could a competent developer (or frontier LLM) implement a working API from this document alone, without needing to ask any clarifying questions?"*

## Differentiation by Skill Level

Students in the class have varying abilities. The exercise must support this:

**Core (required for all students):**
- Resource identification and data modeling
- CRUD endpoints with proper HTTP methods
- Meaningful status codes
- Request/response body schemas
- Basic validation rules
- Standard error response format

**Advanced (stretch goals for stronger students):**
- Pagination, filtering, and sorting
- Idempotency considerations (e.g. PUT vs. PATCH semantics, retry-safe operations)
- Concurrency / conflict handling (e.g. ETags, optimistic locking)
- Bulk operations
- HATEOAS or hypermedia hints
- Conditional requests (If-Match, If-None-Match)
- Rate limiting considerations in the design
- Versioning strategy

The exercise should clearly mark which parts are core and which are advanced, so average students are not overwhelmed and strong students are not bored.

## Exercise Format

- Self-contained written exercise (handout)
- Written in English
- Scenario-based: students work with a concrete domain
- Estimated effort: 4-8 hours
- Individual work

## Domain Choice (to be decided)

The exercise needs a concrete scenario. Criteria for a good domain:
- Familiar enough that students don't need domain expertise
- Complex enough to require multiple resources with relationships
- Rich enough for interesting validation rules and edge cases
- Not so large that the exercise becomes overwhelming

Candidates:
- Library management system (books, members, loans)
- Event/ticket booking system
- Small inventory/warehouse management
- Fitness studio membership and class booking
- Restaurant reservation and menu management
