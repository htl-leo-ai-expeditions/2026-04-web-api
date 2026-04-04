# Exercise: Designing a RESTful Web API — FitBook Studio

## Introduction

In this exercise, you will design a RESTful web API for **FitBook**, a fictional fitness studio management system. Your task is **not** to write code — it is to produce a **Low-Level Design (LLD) document** that specifies the API completely and unambiguously.

Your LLD should be so precise that a competent developer — or a frontier AI model — could implement a fully working API from your document alone, without needing to ask a single clarifying question. Think of your LLD as a prompt: the clearer and more complete it is, the better the result.

### What You Will Practice

- Identifying resources and modeling data for a REST API
- Designing endpoints with appropriate HTTP methods and URIs
- Specifying request/response schemas, status codes, and validation rules
- Thinking about edge cases and error handling before writing any code
- Writing structured, unambiguous technical specifications

### Estimated Effort

4-8 hours of focused work. You do not need to finish in one sitting.

---

## The Scenario: FitBook Studio

FitBook is a small fitness studio that offers group classes (e.g. Yoga, Spinning, HIIT) led by instructors. Members sign up for the studio and book spots in upcoming classes. The studio needs an API to manage its operations.

### The Domain at a Glance

The studio has **members**, **instructors**, and **classes**. Here is how they relate:

```
┌──────────┐       leads        ┌──────────┐
│Instructor├────────────────────┤  Class   │
└──────────┘       1 : N        └────┬─────┘
                                     │
                                     │ bookings
                                     │ N : M
                                     │
                                ┌────┴─────┐
                                │  Member  │
                                └──────────┘
```

- An **instructor** can lead many classes, but each class has exactly one instructor.
- A **member** can book spots in many classes, and a class can have many members booked.
- Each class has a **maximum capacity** — once it is full, no more bookings are accepted.

### Core Business Rules

These rules define how FitBook works. Your API design must enforce all of them.

1. **Members** have a first name, last name, email (unique), and a membership status (active or inactive). Only active members can book classes.
2. **Instructors** have a first name, last name, email (unique), and a list of qualifications (e.g. "Yoga", "HIIT", "Spinning").
3. **Classes** have a title, description, an assigned instructor, a start time, a duration (in minutes), a maximum capacity, and a room name.
4. An instructor can only be assigned to a class if they hold a matching qualification (e.g. a "Yoga" class requires the instructor to have the "Yoga" qualification).
5. Classes must not overlap in the same room (no two classes in the same room at the same time).
6. Classes must not overlap for the same instructor (an instructor cannot lead two classes at the same time).
7. A member cannot book the same class twice.
8. A booking can only be made if the class has not yet reached its maximum capacity.
9. A booking can only be made for classes that start in the future (not in the past).
10. When a class is deleted, all associated bookings are also removed.
11. An instructor cannot be deleted if they have upcoming classes assigned.
12. A member cannot be deleted if they have upcoming bookings.

---

## Your Task: Write the LLD

Your LLD must cover all sections listed below. Work **top-down** — start with the big picture and progressively add detail. Resist the urge to jump straight into endpoint details.

### Recommended Workflow

Follow these phases in order. Each phase builds on the previous one.

#### Phase 1: Big Picture

Identify the main resources (entities) of the API. Sketch their attributes and relationships. Think about which attributes are required vs. optional, which must be unique, and what data types they have.

**Deliverable:** A data model section with entities, attributes, types, and relationships (an ER diagram is encouraged but not required).

#### Phase 2: General Conventions

Before designing individual endpoints, establish rules that apply across the entire API:

- **Base URL structure** — e.g. `/api/v1/...`
- **Naming conventions** — how are resources named in URIs? Plural or singular? camelCase or kebab-case?
- **Standard error response format** — what does every error response look like? (e.g. a JSON object with `error`, `message`, `details` fields)
- **Date/time format** — how are dates and times represented? (e.g. ISO 8601)
- **ID format** — auto-increment integers, UUIDs, or something else?
- **Common HTTP status codes** — define which status codes you use and what they mean in your API

**Deliverable:** A conventions section that a developer can reference when implementing any endpoint.

#### Phase 3: Endpoint Overview

List every endpoint your API exposes: HTTP method, URI, and a one-line description. No request/response bodies yet — just the map.

Example format:

| Method | URI | Description |
|--------|-----|-------------|
| GET | /api/v1/members | List all members |
| POST | /api/v1/members | Create a new member |
| ... | ... | ... |

**Deliverable:** A complete endpoint table covering all resources and operations.

#### Phase 4: Endpoint Details

Now specify each endpoint fully:

- **HTTP method and URI** (from Phase 3)
- **Request body** (if applicable) — with field names, types, required/optional, and validation rules
- **Response body** — with field names and types for success responses
- **Status codes** — for success and each error case, with a description of when each occurs
- **Business rules enforced** — reference the rules from the scenario that apply to this endpoint
- **Example request and response** (JSON) — at least one success and one error case per endpoint

**Deliverable:** Detailed specification for every endpoint.

---

## Worked Example

To show you the level of detail expected, here is a complete specification for **one** endpoint. Use this as a template for your own work.

### `POST /api/v1/members` — Create a new member

**Description:** Registers a new member in the system.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Must be a valid email format, unique across all members |

**Success Response:** `201 Created`

```json
{
  "id": "m-1024",
  "firstName": "Jane",
  "lastName": "Rivera",
  "email": "jane.rivera@example.com",
  "membershipStatus": "active",
  "createdAt": "2026-04-04T10:30:00Z"
}
```

Note: `membershipStatus` defaults to `"active"` on creation. `id` and `createdAt` are generated by the server.

**Error Responses:**

| Status Code | Condition | Example Response |
|-------------|-----------|------------------|
| `400 Bad Request` | Missing or invalid fields (e.g. blank name, malformed email) | See below |
| `409 Conflict` | A member with this email already exists | See below |

**Error example — validation failure (`400`):**

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Request body contains invalid fields.",
  "details": [
    { "field": "email", "issue": "Must be a valid email address." },
    { "field": "firstName", "issue": "Must not be blank." }
  ]
}
```

**Error example — duplicate email (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "A member with email 'jane.rivera@example.com' already exists."
}
```

**Business Rules Enforced:**
- Rule 1: Email must be unique. Membership status defaults to active.

> **Important:** This example illustrates the format and depth — you are expected to produce similar detail for *every* endpoint in your LLD. Your conventions, field names, and error format may differ from this example, as long as they are consistent throughout your document.

---

## Requirements for Your LLD

### Must Include (Core)

These sections are **required** for every student:

- [ ] **Data Model**: Entities, attributes (with types), relationships, and constraints
- [ ] **Conventions**: Base URL, naming, error format, date format, ID format, common status codes
- [ ] **Endpoint Overview**: Complete table of all endpoints
- [ ] **Endpoint Details**: Full specification for every endpoint (request/response schemas, status codes, validation, examples)
- [ ] **Business Rule Coverage**: Every business rule from the scenario must be reflected in at least one endpoint's specification

### May Include (Advanced)

These topics are **optional** and intended for students who want an extra challenge. If you include them, integrate them into your LLD — don't just mention them in passing.

- [ ] **Pagination**: How does the API handle large lists? Define query parameters (e.g. `page`, `pageSize`), response metadata (total count, links), and limits.
- [ ] **Filtering and Sorting**: Can clients filter classes by date, instructor, or room? Can they sort results? Define the query parameter conventions.
- [ ] **Concurrency Handling**: What happens if two members try to book the last spot at the same time? Consider ETags and optimistic locking for updates.
- [ ] **Idempotency**: Are your POST/PUT/PATCH operations safe to retry? How do you prevent double-bookings on network retries?
- [ ] **Conditional Requests**: Use of `If-Match`, `If-None-Match`, `If-Modified-Since` headers for cache validation and conflict detection.
- [ ] **Bulk Operations**: Can an admin create multiple classes at once? How would a bulk endpoint work?
- [ ] **HATEOAS / Hypermedia**: Do your responses include links to related resources or available actions?
- [ ] **Rate Limiting**: How does the API protect itself from abuse? What headers communicate rate limit status?
- [ ] **OpenAPI Specification**: Write an OpenAPI (Swagger) YAML/JSON specification for a subset of your endpoints.

---

## Submission

Submit your LLD as a single document (Markdown, PDF, or Word). Structure it clearly with headings and use tables, diagrams, and JSON examples generously. Remember: your document should speak for itself.

### Self-Check Before Submission

Ask yourself these questions:

1. Could someone implement this API without asking me a single question?
2. Is every business rule from the scenario addressed in at least one endpoint?
3. Are my error cases complete — what happens when things go wrong?
4. Are my conventions consistent across all endpoints?
5. Did I include concrete JSON examples, not just abstract descriptions?

---

## Tips

- **Be specific.** "Returns an error" is not enough. Specify *which* status code, *what* the error response body looks like, and *when* it occurs.
- **Be consistent.** If you decide on a naming convention, use it everywhere. If you define an error format, every error should follow it.
- **Think about the consumer.** A frontend developer will read your API spec. Will they know exactly what to send and what to expect?
- **Steal from the best.** Look at well-designed public APIs (Stripe, GitHub, Spotify) for inspiration on structure and conventions.
- **Don't overcomplicate.** A clean, consistent API with 15 well-designed endpoints is better than a messy one with 30.
