# Exercise: Designing a RESTful Web API — FitBook Studio

## Introduction

You are about to design a RESTful web API for **FitBook**, a fictional fitness studio management system. Here's the twist: **you won't write a single line of code.** Instead, you will produce a **Low-Level Design (LLD) document** — a complete blueprint that specifies every detail of the API.

Why does this matter? Imagine handing your LLD to a developer — or pasting it into a frontier AI model like ChatGPT or Claude. If your spec is precise enough, they should be able to build a fully working API without asking you a single question. Your LLD *is* the prompt. Vague specs produce broken implementations. Precise specs produce working software.

**Estimated effort:** 4-8 hours of focused work. You don't need to finish in one sitting.

---

## The Scenario: FitBook Studio

Picture a small fitness studio around the corner. They run group classes — Yoga on Monday mornings, Spinning at lunch, HIIT after work — each led by an instructor. People sign up as members and book spots in upcoming classes. The studio owner wants an API to manage all of this.

### The Domain at a Glance

Three things matter in this domain: **members**, **instructors**, and **classes**. Here is how they connect:

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

These are the rules of the FitBook world. Your API must enforce every single one — if a rule says "only active members can book," then your API needs to check that and reject the request if it's violated.

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

## Design Decisions to Think About

Here's where it gets interesting. As you design, you'll run into questions where there is no single "correct" answer — just trade-offs. Don't try to answer these upfront. Instead, keep them in the back of your mind and let your LLD reflect a deliberate choice on each.

- **Bookings as a resource:** Is a booking its own top-level resource (`/api/v1/bookings`) or is it nested under a class (`/api/v1/classes/{id}/bookings`)? What are the trade-offs? What if you need to look up all bookings for a specific member?
- **Modeling qualifications:** An instructor has qualifications like "Yoga" or "HIIT". Should these be a separate entity with their own endpoints, a fixed enum, or a free-text list stored on the instructor? What does your choice imply for validation of Rule 4?
- **Cancellation vs. deletion:** When a member cancels a booking, do you DELETE the booking resource or do you PATCH its status to "cancelled"? What information is lost in each case?
- **Updating a class:** If an instructor is reassigned to a class that already has bookings, do the bookings stay? What if the capacity is reduced below the current number of bookings?
- **Time zone handling:** The studio operates in a specific time zone. Should the API accept and return UTC times, local times, or both? How does this affect Rule 9 (no booking past classes)?
- **What the server generates vs. what the client sends:** Which fields should the server set automatically (e.g. `id`, `createdAt`)? Which fields should the client never be able to override?

> **There are no wrong answers here — only undocumented ones.** "Bookings are nested under classes *because...*" beats silently nesting them without explanation every time.

---

## Your Task: Write the LLD

Your LLD must cover all the sections below. The key principle: work **top-down**. Start with the big picture, then zoom in step by step. It's tempting to jump straight into endpoint details — resist that urge. You'll save yourself a lot of rework.

### Recommended Workflow

Follow these four phases in order. Each one builds on the previous.

#### Phase 1: Big Picture

Start by figuring out the "what." What are the main resources (entities) in this API? What attributes do they have? How do they relate to each other? Which attributes are required, which are unique, what types do they use?

**Deliverable:** A data model section with entities, attributes, types, and relationships. An ER diagram is a nice bonus but not required.

#### Phase 2: General Conventions

Before you touch a single endpoint, nail down the rules that apply everywhere. This saves you from the classic mistake of designing ten endpoints and then realizing they all use different naming styles:

- **Base URL structure** — e.g. `/api/v1/...`
- **Naming conventions** — how are resources named in URIs? Plural or singular? camelCase or kebab-case?
- **Standard error response format** — what does every error response look like? (e.g. a JSON object with `error`, `message`, `details` fields)
- **Date/time format** — how are dates and times represented? (e.g. ISO 8601)
- **ID format** — auto-increment integers, UUIDs, or something else?
- **Common HTTP status codes** — define which status codes you use and what they mean in your API

**Deliverable:** A conventions section that a developer can reference when implementing any endpoint.

#### Phase 3: Endpoint Overview

Now sketch out the full map of your API. List every endpoint: HTTP method, URI, and a one-line description. Don't worry about request/response bodies yet — this is just the bird's-eye view.

Example format:

| Method | URI | Description |
|--------|-----|-------------|
| GET | /api/v1/members | List all members |
| POST | /api/v1/members | Create a new member |
| ... | ... | ... |

**Deliverable:** A complete endpoint table covering all resources and operations.

#### Phase 4: Endpoint Details

This is the main event — now you go deep on every endpoint:

- **HTTP method and URI** (from Phase 3)
- **Request body** (if applicable) — with field names, types, required/optional, and validation rules
- **Response body** — with field names and types for success responses
- **Status codes** — for success and each error case, with a description of when each occurs
- **Business rules enforced** — reference the rules from the scenario that apply to this endpoint
- **Example request and response** (JSON) — at least one success and one error case per endpoint

**Deliverable:** Detailed specification for every endpoint.

---

## Worked Example

Wondering how detailed your specs should be? Here's a fully worked-out endpoint to set the bar. Use it as a template for your own work.

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

> **This is the level of detail you should aim for on every endpoint.** Your conventions, field names, and error format can differ from this example — what matters is that you're consistent across your whole API.

---

## Requirements for Your LLD

### Must Include (Core)

Everyone needs to cover these — they're the foundation of a solid LLD:

- [ ] **Data Model**: Entities, attributes (with types), relationships, and constraints
- [ ] **Conventions**: Base URL, naming, error format, date format, ID format, common status codes
- [ ] **Endpoint Overview**: Complete table of all endpoints
- [ ] **Endpoint Details**: Full specification for every endpoint (request/response schemas, status codes, validation, examples)
- [ ] **Business Rule Coverage**: Every business rule from the scenario must be reflected in at least one endpoint's specification

### May Include (Advanced)

Want to go further? These topics are **optional** and meant for those who want a real challenge. If you tackle any of them, make sure you integrate them properly into your LLD — don't just name-drop them.

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

Submit your LLD as a single document (Markdown, PDF, or Word). Use headings, tables, diagrams, and JSON examples generously — your document needs to speak for itself.

### Self-Check Before You Submit

Before you hand it in, run through these questions honestly:

1. Could someone implement this API without asking me a single question?
2. Is every business rule from the scenario addressed in at least one endpoint?
3. For each endpoint, have I specified what happens when things go wrong — not just the happy path?
4. Are my naming, error format, and conventions consistent across *all* endpoints?
5. Did I include concrete JSON examples, not just abstract descriptions?

---

## Tips

- **Be specific.** "Returns an error" tells nobody anything. *Which* status code? *What* does the body look like? *When* does it happen?
- **Think like a frontend dev.** Someone will consume your API. Could they build a UI against your spec without guessing?
- **Steal from the best.** Look at APIs from Stripe, GitHub, or Spotify for inspiration — they've spent years getting their conventions right.
- **Keep it clean.** 15 well-designed endpoints beat 30 messy ones. Every time.
