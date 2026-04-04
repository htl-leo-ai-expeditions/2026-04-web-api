# FitBook Studio API -- Low-Level Design Document

**Author:** Lena M.  
**Date:** 2026-04-04  
**Version:** 1.0

---

## Table of Contents

1. [Design Decisions](#1-design-decisions)
2. [Data Model](#2-data-model)
3. [General Conventions](#3-general-conventions)
4. [Endpoint Overview](#4-endpoint-overview)
5. [Endpoint Details](#5-endpoint-details)
6. [Business Rule Coverage Checklist](#6-business-rule-coverage-checklist)
7. [Advanced Features](#7-advanced-features)

---

## 1. Design Decisions

Before diving into the spec, here are the key design trade-offs I considered and the choices I made. I'm documenting these up front so that any implementer understands the *why* behind the structure.

### 1.1 Bookings: Nested vs. Top-Level Resource

**Decision:** Bookings are modeled as a **sub-resource of classes** for creation (`POST /api/v1/classes/{classId}/bookings`) but also queryable as a top-level collection (`GET /api/v1/bookings`) for cross-cutting queries.

**Reasoning:** The primary action is "book a spot in a class," which maps naturally to a nested route. However, the system also needs to answer "what are all bookings for member X?" -- a query that cuts across classes. A top-level `GET /api/v1/bookings?memberId=...` endpoint handles that without forcing the client to iterate over every class. Deletion also goes through the nested path (`DELETE /api/v1/classes/{classId}/bookings/{bookingId}`) since a booking only makes sense in the context of a class.

### 1.2 Qualifications: Separate Resource with Endpoints

**Decision:** Qualifications are a **managed set** -- a separate entity with its own CRUD endpoints (`/api/v1/qualifications`). Instructors reference qualifications by ID.

**Reasoning:** Free-text strings would lead to inconsistencies ("yoga" vs. "Yoga" vs. "YOGA"). A fixed enum baked into code is inflexible -- the studio might add "Pilates" next month. A managed set gives us a canonical list, enables server-side validation of Rule 4, and allows the frontend to populate dropdowns. The trade-off is an extra entity to manage, but it's a simple one.

### 1.3 Cancellation vs. Deletion of Bookings

**Decision:** Cancelling a booking **deletes** the resource (`DELETE`). There is no "cancelled" status.

**Reasoning:** The exercise scope is a small studio. There's no stated requirement for booking history or analytics on cancellations. Soft-deletion adds complexity (every query must filter out cancelled bookings, capacity calculations become trickier). If booking history were needed, I'd add a `status` field with `active`/`cancelled` states instead. But for this scope, hard delete is simpler and cleaner. The creation timestamp is lost, but that's acceptable here.

### 1.4 Updating a Class with Existing Bookings

**Decision:**
- **Instructor reassignment:** Allowed. Existing bookings stay. The new instructor must hold the matching qualification (Rule 4 still enforced).
- **Capacity reduction:** Rejected if the new capacity would be lower than the current number of bookings. The API returns `409 Conflict`.

**Reasoning:** Silently orphaning bookings or kicking members out would be a terrible user experience. It's better to force the admin to resolve overbooking explicitly (e.g., cancel some bookings first, then reduce capacity).

### 1.5 Time Zone Handling

**Decision:** All times are in **UTC**, formatted as ISO 8601 strings (e.g., `"2026-04-10T14:00:00Z"`). The server stores and returns UTC exclusively.

**Reasoning:** UTC avoids daylight-saving-time ambiguities and is the industry standard for APIs. The frontend/client can convert to the studio's local time zone for display. Rule 9 (no booking past classes) is evaluated against the server's UTC clock.

### 1.6 Server-Generated vs. Client-Provided Fields

**Server-generated (read-only):** `id`, `createdAt`, `updatedAt`, `currentBookingCount` (on classes).

**Client-provided:** All other fields. If the client sends `id` or `createdAt` in a request body, the server silently ignores them.

---

## 2. Data Model

### 2.1 Entities

#### Qualification

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | string | yes | yes | Server-generated, prefixed: `q-{number}` |
| `name` | string | yes | yes | 1-100 characters, non-blank (e.g., "Yoga", "HIIT", "Spinning") |
| `createdAt` | string (ISO 8601) | yes | no | Server-generated |

#### Instructor

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | string | yes | yes | Server-generated, prefixed: `i-{number}` |
| `firstName` | string | yes | no | 1-100 characters, non-blank |
| `lastName` | string | yes | no | 1-100 characters, non-blank |
| `email` | string | yes | yes | Valid email format, unique across all instructors |
| `qualificationIds` | array of string | yes | no | References `Qualification.id`, at least one required |
| `createdAt` | string (ISO 8601) | yes | no | Server-generated |
| `updatedAt` | string (ISO 8601) | yes | no | Server-generated |

#### Member

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | string | yes | yes | Server-generated, prefixed: `m-{number}` |
| `firstName` | string | yes | no | 1-100 characters, non-blank |
| `lastName` | string | yes | no | 1-100 characters, non-blank |
| `email` | string | yes | yes | Valid email format, unique across all members |
| `membershipStatus` | string (enum) | yes | no | `"active"` or `"inactive"`. Defaults to `"active"` on creation |
| `createdAt` | string (ISO 8601) | yes | no | Server-generated |
| `updatedAt` | string (ISO 8601) | yes | no | Server-generated |

#### Class

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | string | yes | yes | Server-generated, prefixed: `c-{number}` |
| `title` | string | yes | no | 1-150 characters, non-blank |
| `description` | string | no | no | 0-1000 characters. Defaults to `""` if omitted |
| `instructorId` | string | yes | no | References `Instructor.id`, must exist |
| `startTime` | string (ISO 8601) | yes | no | Must be in the future at creation time |
| `durationMinutes` | integer | yes | no | Positive integer, 15-480 |
| `maxCapacity` | integer | yes | no | Positive integer, 1-100 |
| `roomName` | string | yes | no | 1-100 characters, non-blank |
| `currentBookingCount` | integer | yes | no | Server-managed, starts at 0 |
| `createdAt` | string (ISO 8601) | yes | no | Server-generated |
| `updatedAt` | string (ISO 8601) | yes | no | Server-generated |

#### Booking

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | string | yes | yes | Server-generated, prefixed: `b-{number}` |
| `classId` | string | yes | no | References `Class.id` |
| `memberId` | string | yes | no | References `Member.id` |
| `createdAt` | string (ISO 8601) | yes | no | Server-generated |

Uniqueness constraint: The combination of (`classId`, `memberId`) must be unique -- a member cannot book the same class twice.

### 2.2 Relationships

| Relationship | Type | Description |
|--------------|------|-------------|
| Qualification -> Instructor | M:N | An instructor holds one or more qualifications; a qualification can be held by many instructors. Stored as `qualificationIds` array on Instructor. |
| Instructor -> Class | 1:N | An instructor leads many classes; each class has exactly one instructor. |
| Class -> Booking | 1:N | A class has many bookings; each booking belongs to one class. |
| Member -> Booking | 1:N | A member has many bookings; each booking belongs to one member. |
| Member <-> Class | M:N | Many-to-many through the Booking entity. |

### 2.3 ER Diagram

```
┌─────────────────┐
│  Qualification  │
│─────────────────│
│ id (PK)         │
│ name (UNIQUE)   │
│ createdAt       │
└────────┬────────┘
         │
         │ M:N (via qualificationIds)
         │
┌────────┴────────┐         1:N          ┌─────────────────────┐
│   Instructor    │──────────────────────>│       Class         │
│─────────────────│                       │─────────────────────│
│ id (PK)         │                       │ id (PK)             │
│ firstName       │                       │ title               │
│ lastName        │                       │ description         │
│ email (UNIQUE)  │                       │ instructorId (FK)   │
│ qualificationIds│                       │ startTime           │
│ createdAt       │                       │ durationMinutes     │
│ updatedAt       │                       │ maxCapacity         │
│                 │                       │ roomName            │
│                 │                       │ currentBookingCount │
│                 │                       │ createdAt           │
│                 │                       │ updatedAt           │
└─────────────────┘                       └──────────┬──────────┘
                                                     │
                                                     │ 1:N
                                                     │
                                          ┌──────────┴──────────┐
┌─────────────────┐          M:N          │      Booking        │
│    Member       │<─────────────────────>│─────────────────────│
│─────────────────│  (via Booking)        │ id (PK)             │
│ id (PK)         │                       │ classId (FK)        │
│ firstName       │                       │ memberId (FK)       │
│ lastName        │                       │ createdAt           │
│ email (UNIQUE)  │                       │                     │
│ membershipStatus│                       │ UNIQUE(classId,     │
│ createdAt       │                       │        memberId)    │
│ updatedAt       │                       └─────────────────────┘
└─────────────────┘
```

---

## 3. General Conventions

### 3.1 Base URL

All endpoints are prefixed with:

```
/api/v1/
```

The `v1` prefix enables future versioning without breaking existing clients.

### 3.2 Naming Conventions

| Convention | Decision |
|------------|----------|
| Base URL | `/api/v1/` |
| Resource naming in URIs | Plural, lowercase, no special casing (e.g., `/members`, `/classes`, `/bookings`) |
| ID format | Prefixed string: `{entity-prefix}-{auto-increment number}` (e.g., `m-1`, `i-42`, `c-7`, `b-100`, `q-3`) |
| Date/time format | ISO 8601, always in UTC with `Z` suffix (e.g., `"2026-04-10T14:00:00Z"`) |
| Request/response body casing | camelCase for all JSON field names |
| Query parameter casing | camelCase (e.g., `?memberId=m-1&page=2`) |

### 3.3 Standard Error Response Format

Every error response follows this structure:

```json
{
  "error": "ERROR_CODE",
  "message": "A human-readable description of what went wrong.",
  "details": [
    {
      "field": "fieldName",
      "issue": "Specific problem with this field."
    }
  ]
}
```

- `error` (string, always present): A machine-readable error code in UPPER_SNAKE_CASE.
- `message` (string, always present): A human-friendly summary.
- `details` (array, present only for validation errors): One entry per invalid field.

Error codes used across the API:

| Error Code | Used For |
|------------|----------|
| `VALIDATION_ERROR` | Request body has missing or invalid fields |
| `NOT_FOUND` | Referenced resource does not exist |
| `CONFLICT` | Action violates a uniqueness constraint or business rule conflict |
| `BUSINESS_RULE_VIOLATION` | Action violates a domain rule (e.g., capacity full, inactive member) |

### 3.4 Common HTTP Status Codes

| Status Code | Meaning in This API |
|-------------|---------------------|
| `200 OK` | Successful retrieval or update. Response body contains the resource(s). |
| `201 Created` | Resource successfully created. Response body contains the new resource. |
| `204 No Content` | Successful deletion. No response body. |
| `400 Bad Request` | Request body is malformed, missing required fields, or fields fail validation. |
| `404 Not Found` | The specified resource (by ID) does not exist. |
| `409 Conflict` | The request conflicts with existing state (duplicate email, room overlap, instructor overlap, capacity reduction below current bookings, etc.). |
| `422 Unprocessable Entity` | The request is syntactically valid but violates a business rule (inactive member booking, past class booking, instructor lacks qualification). |

**Note on 409 vs. 422:** I use `409 Conflict` for state conflicts where the resource *state* prevents the action (duplicates, scheduling overlaps, capacity conflicts). I use `422 Unprocessable Entity` for business rule violations where the request is well-formed but semantically invalid. This distinction helps clients handle errors programmatically.

### 3.5 Pagination

List endpoints (`GET` collections) support pagination via query parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | `1` | Page number (1-indexed) |
| `pageSize` | integer | `20` | Items per page. Minimum 1, maximum 100 |

Paginated response wrapper:

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 57,
    "totalPages": 3
  }
}
```

All list endpoints return this wrapper. Single-resource endpoints (GET by ID) return the resource object directly (no wrapper).

### 3.6 Filtering and Sorting

List endpoints support filtering and sorting via query parameters. Filters and sort fields are specific to each resource and documented in the endpoint details. General format:

- **Filtering:** `?fieldName=value` (e.g., `?membershipStatus=active`, `?instructorId=i-5`)
- **Sorting:** `?sort=fieldName` for ascending, `?sort=-fieldName` for descending (e.g., `?sort=-startTime`)

Invalid filter field names or sort values are ignored (the endpoint returns results as if the invalid parameter were not present).

---

## 4. Endpoint Overview

| Method | URI | Description |
|--------|-----|-------------|
| **Qualifications** | | |
| GET | `/api/v1/qualifications` | List all qualifications |
| POST | `/api/v1/qualifications` | Create a new qualification |
| GET | `/api/v1/qualifications/{qualificationId}` | Get a qualification by ID |
| PUT | `/api/v1/qualifications/{qualificationId}` | Update a qualification |
| DELETE | `/api/v1/qualifications/{qualificationId}` | Delete a qualification |
| **Members** | | |
| GET | `/api/v1/members` | List all members |
| POST | `/api/v1/members` | Create a new member |
| GET | `/api/v1/members/{memberId}` | Get a member by ID |
| PUT | `/api/v1/members/{memberId}` | Update a member (including status change) |
| DELETE | `/api/v1/members/{memberId}` | Delete a member |
| **Instructors** | | |
| GET | `/api/v1/instructors` | List all instructors |
| POST | `/api/v1/instructors` | Create a new instructor |
| GET | `/api/v1/instructors/{instructorId}` | Get an instructor by ID |
| PUT | `/api/v1/instructors/{instructorId}` | Update an instructor |
| DELETE | `/api/v1/instructors/{instructorId}` | Delete an instructor |
| **Classes** | | |
| GET | `/api/v1/classes` | List all classes |
| POST | `/api/v1/classes` | Create a new class |
| GET | `/api/v1/classes/{classId}` | Get a class by ID |
| PUT | `/api/v1/classes/{classId}` | Update a class |
| DELETE | `/api/v1/classes/{classId}` | Delete a class (cascades to bookings) |
| **Bookings** | | |
| GET | `/api/v1/bookings` | List all bookings (supports filtering by memberId, classId) |
| POST | `/api/v1/classes/{classId}/bookings` | Create a booking for a class |
| GET | `/api/v1/classes/{classId}/bookings` | List all bookings for a class |
| GET | `/api/v1/classes/{classId}/bookings/{bookingId}` | Get a specific booking |
| DELETE | `/api/v1/classes/{classId}/bookings/{bookingId}` | Cancel (delete) a booking |

**Total: 21 endpoints.**

---

## 5. Endpoint Details

### 5.1 Qualifications

---

#### `POST /api/v1/qualifications` -- Create a new qualification

**Description:** Creates a new qualification that can be assigned to instructors.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `name` | string | yes | 1-100 characters, non-blank, unique across all qualifications |

**Success Response:** `201 Created`

```json
{
  "id": "q-1",
  "name": "Yoga",
  "createdAt": "2026-04-04T08:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | `name` is missing, blank, or exceeds 100 characters |
| `409 Conflict` | A qualification with the same name already exists |

**Error example -- duplicate name (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "A qualification with name 'Yoga' already exists."
}
```

**Business Rules Enforced:** None directly (supporting entity for Rule 4).

---

#### `GET /api/v1/qualifications` -- List all qualifications

**Description:** Returns all qualifications. Supports pagination.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `pageSize` | integer | 20 | Items per page (1-100) |
| `sort` | string | `name` | Sort field. Supported: `name`, `createdAt`. Prefix with `-` for descending. |

**Success Response:** `200 OK`

```json
{
  "data": [
    { "id": "q-1", "name": "HIIT", "createdAt": "2026-04-04T08:00:00Z" },
    { "id": "q-2", "name": "Spinning", "createdAt": "2026-04-04T08:01:00Z" },
    { "id": "q-3", "name": "Yoga", "createdAt": "2026-04-04T08:02:00Z" }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 3,
    "totalPages": 1
  }
}
```

**Error Responses:** None specific (invalid pagination parameters are clamped to valid range).

---

#### `GET /api/v1/qualifications/{qualificationId}` -- Get qualification by ID

**Description:** Returns a single qualification.

**Success Response:** `200 OK`

```json
{
  "id": "q-1",
  "name": "Yoga",
  "createdAt": "2026-04-04T08:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No qualification with this ID exists |

---

#### `PUT /api/v1/qualifications/{qualificationId}` -- Update a qualification

**Description:** Replaces the qualification's name. If the name changes, all instructors holding this qualification remain associated (they reference the ID, not the name).

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `name` | string | yes | 1-100 characters, non-blank, unique |

**Success Response:** `200 OK` -- returns the updated qualification.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | `name` is missing, blank, or exceeds 100 characters |
| `404 Not Found` | No qualification with this ID exists |
| `409 Conflict` | Another qualification with the same name already exists |

---

#### `DELETE /api/v1/qualifications/{qualificationId}` -- Delete a qualification

**Description:** Deletes a qualification. Fails if any instructor currently holds this qualification.

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No qualification with this ID exists |
| `409 Conflict` | One or more instructors hold this qualification. They must be updated first. |

**Error example -- qualification in use (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot delete qualification 'Yoga' because it is assigned to 2 instructor(s). Remove the qualification from those instructors first."
}
```

---

### 5.2 Members

---

#### `POST /api/v1/members` -- Create a new member

**Description:** Registers a new member in the system. Membership status defaults to `"active"`.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email format, unique across all members |

**Success Response:** `201 Created`

```json
{
  "id": "m-1",
  "firstName": "Jane",
  "lastName": "Rivera",
  "email": "jane.rivera@example.com",
  "membershipStatus": "active",
  "createdAt": "2026-04-04T10:30:00Z",
  "updatedAt": "2026-04-04T10:30:00Z"
}
```

Note: `id`, `createdAt`, `updatedAt` are server-generated. `membershipStatus` defaults to `"active"`.

**Error Responses:**

| Status Code | Condition | Example Response |
|-------------|-----------|------------------|
| `400 Bad Request` | Missing or invalid fields | See below |
| `409 Conflict` | A member with this email already exists | See below |

**Error example -- validation failure (`400`):**

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

**Error example -- duplicate email (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "A member with email 'jane.rivera@example.com' already exists."
}
```

**Business Rules Enforced:**
- Rule 1: Email must be unique. Membership status defaults to active.

---

#### `GET /api/v1/members` -- List all members

**Description:** Returns a paginated list of members.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `pageSize` | integer | 20 | Items per page (1-100) |
| `membershipStatus` | string | (none) | Filter by status: `active` or `inactive` |
| `sort` | string | `lastName` | Sort field. Supported: `lastName`, `firstName`, `email`, `createdAt`. Prefix with `-` for descending. |

**Success Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "m-1",
      "firstName": "Jane",
      "lastName": "Rivera",
      "email": "jane.rivera@example.com",
      "membershipStatus": "active",
      "createdAt": "2026-04-04T10:30:00Z",
      "updatedAt": "2026-04-04T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

---

#### `GET /api/v1/members/{memberId}` -- Get member by ID

Follows the same pattern as `GET /api/v1/qualifications/{qualificationId}`.

**Differences:**
- Response fields: `id`, `firstName`, `lastName`, `email`, `membershipStatus`, `createdAt`, `updatedAt`
- Returns `404 Not Found` if no member with this ID exists

**Success Response:** `200 OK`

```json
{
  "id": "m-1",
  "firstName": "Jane",
  "lastName": "Rivera",
  "email": "jane.rivera@example.com",
  "membershipStatus": "active",
  "createdAt": "2026-04-04T10:30:00Z",
  "updatedAt": "2026-04-04T10:30:00Z"
}
```

---

#### `PUT /api/v1/members/{memberId}` -- Update a member

**Description:** Full replacement update of a member's data. This endpoint is also used to change membership status (active/inactive).

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email format, unique across all members (excluding this member) |
| `membershipStatus` | string | yes | Must be `"active"` or `"inactive"` |

**Success Response:** `200 OK` -- returns the full updated member object. `updatedAt` is refreshed.

```json
{
  "id": "m-1",
  "firstName": "Jane",
  "lastName": "Rivera-Smith",
  "email": "jane.rivera@example.com",
  "membershipStatus": "inactive",
  "createdAt": "2026-04-04T10:30:00Z",
  "updatedAt": "2026-04-04T12:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields, invalid status value |
| `404 Not Found` | No member with this ID exists |
| `409 Conflict` | Another member already uses this email |

**Business Rules Enforced:**
- Rule 1: Email must be unique. Status can only be `"active"` or `"inactive"`.

**Note:** Changing a member to `"inactive"` does **not** automatically cancel their existing bookings. It prevents new bookings from being made. The admin should cancel bookings separately if desired.

---

#### `DELETE /api/v1/members/{memberId}` -- Delete a member

**Description:** Permanently deletes a member. Fails if the member has bookings for classes that start in the future.

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No member with this ID exists |
| `409 Conflict` | Member has upcoming bookings (classes with startTime in the future) |

**Error example -- member has upcoming bookings (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot delete member 'm-1' because they have 3 upcoming booking(s). Cancel those bookings first."
}
```

**Business Rules Enforced:**
- Rule 12: A member cannot be deleted if they have upcoming bookings.

---

### 5.3 Instructors

---

#### `POST /api/v1/instructors` -- Create a new instructor

**Description:** Registers a new instructor with their qualifications.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email format, unique across all instructors |
| `qualificationIds` | array of string | yes | Non-empty array. Each ID must reference an existing qualification. No duplicate IDs in the array. |

**Success Response:** `201 Created`

```json
{
  "id": "i-1",
  "firstName": "Carlos",
  "lastName": "Mendes",
  "email": "carlos.mendes@example.com",
  "qualifications": [
    { "id": "q-1", "name": "Yoga" },
    { "id": "q-2", "name": "Spinning" }
  ],
  "createdAt": "2026-04-04T09:00:00Z",
  "updatedAt": "2026-04-04T09:00:00Z"
}
```

**Note on response shape:** The request sends `qualificationIds` (array of IDs). The response returns `qualifications` (array of objects with `id` and `name`). This is intentional -- the response embeds the full qualification objects for convenience, so the client doesn't need a follow-up request.

**Error Responses:**

| Status Code | Condition | Example |
|-------------|-----------|---------|
| `400 Bad Request` | Missing/invalid fields, empty qualificationIds array, or duplicate IDs in the array | Standard validation error |
| `404 Not Found` | One or more `qualificationIds` do not reference existing qualifications | See below |
| `409 Conflict` | An instructor with this email already exists | Standard conflict error |

**Error example -- invalid qualification ID (`404`):**

```json
{
  "error": "NOT_FOUND",
  "message": "The following qualification IDs do not exist: q-99, q-100."
}
```

**Business Rules Enforced:**
- Rule 2: Instructors have a unique email and a list of qualifications.

---

#### `GET /api/v1/instructors` -- List all instructors

Follows the same pattern as `GET /api/v1/members`.

**Differences:**
- Response fields: `id`, `firstName`, `lastName`, `email`, `qualifications` (array of `{id, name}`), `createdAt`, `updatedAt`
- Filter parameter: `qualificationId` -- filter instructors who hold a specific qualification
- Sort fields: `lastName`, `firstName`, `email`, `createdAt`

**Success Response:** `200 OK` with paginated wrapper.

---

#### `GET /api/v1/instructors/{instructorId}` -- Get instructor by ID

Follows the same pattern as `GET /api/v1/members/{memberId}`.

**Differences:**
- Response fields: `id`, `firstName`, `lastName`, `email`, `qualifications` (array of `{id, name}`), `createdAt`, `updatedAt`
- Returns `404 Not Found` if no instructor with this ID exists

---

#### `PUT /api/v1/instructors/{instructorId}` -- Update an instructor

**Description:** Full replacement update of an instructor's data, including qualifications.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email format, unique (excluding this instructor) |
| `qualificationIds` | array of string | yes | Non-empty. Each must reference an existing qualification. |

**Success Response:** `200 OK` -- returns the full updated instructor object with embedded qualifications.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing/invalid fields |
| `404 Not Found` | Instructor not found, or one or more qualificationIds don't exist |
| `409 Conflict` | Another instructor already uses this email |
| `422 Unprocessable Entity` | Removing a qualification that is required by an upcoming class this instructor leads |

**Error example -- removing a needed qualification (`422`):**

```json
{
  "error": "BUSINESS_RULE_VIOLATION",
  "message": "Cannot remove qualification 'Yoga' (q-1) from instructor 'i-1' because they lead upcoming Yoga class(es): c-5, c-12."
}
```

**Business Rules Enforced:**
- Rule 2: Email unique, qualifications maintained.
- Rule 4 (indirectly): If an instructor's qualifications are updated, any upcoming classes they lead must still have a matching qualification. This prevents an instructor from losing a qualification while still assigned to classes requiring it.

---

#### `DELETE /api/v1/instructors/{instructorId}` -- Delete an instructor

**Description:** Permanently deletes an instructor. Fails if the instructor has upcoming classes.

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No instructor with this ID exists |
| `409 Conflict` | Instructor has upcoming classes (classes with startTime in the future) |

**Error example (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot delete instructor 'i-1' because they have 2 upcoming class(es). Reassign or delete those classes first."
}
```

**Business Rules Enforced:**
- Rule 11: An instructor cannot be deleted if they have upcoming classes assigned.

---

### 5.4 Classes

---

#### `POST /api/v1/classes` -- Create a new class

**Description:** Creates a new fitness class. This is one of the most complex endpoints in the API due to the number of business rules it enforces.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | yes | 1-150 characters, non-blank |
| `description` | string | no | 0-1000 characters. Defaults to `""` |
| `instructorId` | string | yes | Must reference an existing instructor |
| `startTime` | string (ISO 8601) | yes | Must be a valid ISO 8601 datetime in UTC. Must be in the future. |
| `durationMinutes` | integer | yes | Positive integer, 15-480 |
| `maxCapacity` | integer | yes | Positive integer, 1-100 |
| `roomName` | string | yes | 1-100 characters, non-blank |

**Success Response:** `201 Created`

```json
{
  "id": "c-1",
  "title": "Morning Yoga",
  "description": "A gentle start to your day.",
  "instructorId": "i-1",
  "startTime": "2026-04-10T07:00:00Z",
  "durationMinutes": 60,
  "maxCapacity": 20,
  "roomName": "Studio A",
  "currentBookingCount": 0,
  "createdAt": "2026-04-04T11:00:00Z",
  "updatedAt": "2026-04-04T11:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing/invalid fields (blank title, duration out of range, invalid datetime format, etc.) |
| `404 Not Found` | The referenced `instructorId` does not exist |
| `409 Conflict` | Room overlap: another class already occupies the same room at the same time |
| `409 Conflict` | Instructor overlap: the instructor already leads another class at the same time |
| `422 Unprocessable Entity` | The instructor does not hold a qualification matching the class title |
| `422 Unprocessable Entity` | The startTime is in the past |

**Overlap detection logic:** Two classes overlap if they are in the same room (or have the same instructor) AND their time ranges intersect. A class occupies the time range `[startTime, startTime + durationMinutes)`. Two ranges `[A_start, A_end)` and `[B_start, B_end)` overlap if `A_start < B_end AND B_start < A_end`. Classes that are exactly back-to-back (one ends at the moment another starts) do **not** overlap.

**Qualification matching logic (Rule 4):** The class title is matched against the instructor's qualifications. Specifically, the instructor must hold at least one qualification whose `name` is a **case-insensitive substring** of the class title. For example:
- Class title "Morning Yoga" -- instructor needs the "Yoga" qualification.
- Class title "HIIT Extreme" -- instructor needs the "HIIT" qualification.
- Class title "Spinning" -- instructor needs the "Spinning" qualification.

This matching approach is flexible enough to handle descriptive titles while still enforcing that the instructor is qualified.

**Error example -- room overlap (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Room 'Studio A' is already booked from 2026-04-10T07:00:00Z to 2026-04-10T08:00:00Z by class 'c-3' ('Power Yoga')."
}
```

**Error example -- instructor overlap (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Instructor 'i-1' already leads class 'c-3' ('Power Yoga') from 2026-04-10T07:00:00Z to 2026-04-10T08:00:00Z."
}
```

**Error example -- qualification mismatch (`422`):**

```json
{
  "error": "BUSINESS_RULE_VIOLATION",
  "message": "Instructor 'i-1' (Carlos Mendes) does not hold a qualification matching class title 'Pilates Fusion'. Their qualifications: Yoga, Spinning."
}
```

**Business Rules Enforced:**
- Rule 3: All required fields present.
- Rule 4: Instructor must hold matching qualification.
- Rule 5: No room overlap.
- Rule 6: No instructor time overlap.

---

#### `GET /api/v1/classes` -- List all classes

**Description:** Returns a paginated list of classes with filtering and sorting.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `pageSize` | integer | 20 | Items per page (1-100) |
| `instructorId` | string | (none) | Filter by instructor |
| `roomName` | string | (none) | Filter by room name (exact match) |
| `fromDate` | string (ISO 8601) | (none) | Only classes starting at or after this time |
| `toDate` | string (ISO 8601) | (none) | Only classes starting before this time |
| `sort` | string | `startTime` | Sort field. Supported: `startTime`, `title`, `createdAt`. Prefix with `-` for descending. |

**Success Response:** `200 OK` with paginated wrapper. Each item contains all class fields.

```json
{
  "data": [
    {
      "id": "c-1",
      "title": "Morning Yoga",
      "description": "A gentle start to your day.",
      "instructorId": "i-1",
      "startTime": "2026-04-10T07:00:00Z",
      "durationMinutes": 60,
      "maxCapacity": 20,
      "roomName": "Studio A",
      "currentBookingCount": 5,
      "createdAt": "2026-04-04T11:00:00Z",
      "updatedAt": "2026-04-04T11:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

---

#### `GET /api/v1/classes/{classId}` -- Get class by ID

Follows the same pattern as `GET /api/v1/members/{memberId}`.

**Differences:**
- Response fields: all class fields (`id`, `title`, `description`, `instructorId`, `startTime`, `durationMinutes`, `maxCapacity`, `roomName`, `currentBookingCount`, `createdAt`, `updatedAt`)
- Returns `404 Not Found` if no class with this ID exists

---

#### `PUT /api/v1/classes/{classId}` -- Update a class

**Description:** Full replacement update of a class. All the same business rules as creation apply, plus additional constraints when the class already has bookings.

**Request Body:** Same as `POST /api/v1/classes`.

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | yes | 1-150 characters, non-blank |
| `description` | string | no | 0-1000 characters |
| `instructorId` | string | yes | Must reference an existing instructor |
| `startTime` | string (ISO 8601) | yes | Must be in the future |
| `durationMinutes` | integer | yes | 15-480 |
| `maxCapacity` | integer | yes | 1-100 |
| `roomName` | string | yes | 1-100 characters, non-blank |

**Success Response:** `200 OK` -- returns the full updated class. `updatedAt` is refreshed.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing/invalid fields |
| `404 Not Found` | Class not found, or referenced instructorId doesn't exist |
| `409 Conflict` | Room overlap with another class (excluding this class itself) |
| `409 Conflict` | Instructor overlap with another class (excluding this class itself) |
| `409 Conflict` | `maxCapacity` reduced below `currentBookingCount` |
| `422 Unprocessable Entity` | New instructor does not hold matching qualification |
| `422 Unprocessable Entity` | startTime is in the past |

**Error example -- capacity reduction conflict (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot reduce maxCapacity to 5 because class 'c-1' already has 8 booking(s). Cancel some bookings first."
}
```

**Note on overlap checks:** When updating, the overlap check excludes the class being updated. This ensures that changing a non-time field (like `title`) doesn't trigger a false overlap conflict with itself.

**Business Rules Enforced:**
- Rule 3: All required fields present.
- Rule 4: Instructor must hold matching qualification (also for reassignment).
- Rule 5: No room overlap (excluding self).
- Rule 6: No instructor time overlap (excluding self).
- Design Decision 1.4: Capacity cannot be reduced below current bookings.

---

#### `DELETE /api/v1/classes/{classId}` -- Delete a class

**Description:** Permanently deletes a class and all associated bookings.

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No class with this ID exists |

**Business Rules Enforced:**
- Rule 10: When a class is deleted, all associated bookings are also removed. This is a cascade delete. The response does not list the deleted bookings.

---

### 5.5 Bookings

---

#### `POST /api/v1/classes/{classId}/bookings` -- Create a booking

**Description:** Books a spot in a class for a member. This is the most business-rule-heavy endpoint in the API.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `memberId` | string | yes | Must reference an existing member |

**Success Response:** `201 Created`

```json
{
  "id": "b-1",
  "classId": "c-1",
  "memberId": "m-1",
  "createdAt": "2026-04-04T12:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | `memberId` missing or invalid format |
| `404 Not Found` | The class (`classId` from path) does not exist |
| `404 Not Found` | The member (`memberId` from body) does not exist |
| `409 Conflict` | The member already has a booking for this class (duplicate booking) |
| `409 Conflict` | The class has reached maximum capacity |
| `422 Unprocessable Entity` | The member's membershipStatus is `"inactive"` |
| `422 Unprocessable Entity` | The class startTime is in the past |

**Error example -- inactive member (`422`):**

```json
{
  "error": "BUSINESS_RULE_VIOLATION",
  "message": "Member 'm-1' has an inactive membership and cannot book classes."
}
```

**Error example -- class full (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Class 'c-1' ('Morning Yoga') has reached its maximum capacity of 20."
}
```

**Error example -- duplicate booking (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Member 'm-1' already has a booking for class 'c-1'."
}
```

**Error example -- class in the past (`422`):**

```json
{
  "error": "BUSINESS_RULE_VIOLATION",
  "message": "Cannot book class 'c-1' because it started at 2026-04-01T07:00:00Z, which is in the past."
}
```

**Side effects:** On successful booking, the class's `currentBookingCount` is incremented by 1.

**Business Rules Enforced:**
- Rule 1: Only active members can book.
- Rule 7: A member cannot book the same class twice.
- Rule 8: Booking only if capacity not reached.
- Rule 9: Booking only for future classes.

---

#### `GET /api/v1/classes/{classId}/bookings` -- List bookings for a class

**Description:** Returns all bookings for a specific class, paginated.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `pageSize` | integer | 20 | Items per page (1-100) |
| `sort` | string | `createdAt` | Sort field. Supported: `createdAt`. Prefix with `-` for descending. |

**Success Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "b-1",
      "classId": "c-1",
      "memberId": "m-1",
      "createdAt": "2026-04-04T12:00:00Z"
    },
    {
      "id": "b-2",
      "classId": "c-1",
      "memberId": "m-2",
      "createdAt": "2026-04-04T12:05:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 2,
    "totalPages": 1
  }
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | The class (`classId` from path) does not exist |

---

#### `GET /api/v1/bookings` -- List all bookings (cross-cutting query)

**Description:** Returns bookings across all classes. Supports filtering by member and/or class. This is the top-level query endpoint that complements the nested endpoints.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `pageSize` | integer | 20 | Items per page (1-100) |
| `memberId` | string | (none) | Filter bookings by member |
| `classId` | string | (none) | Filter bookings by class |
| `sort` | string | `createdAt` | Sort field. Supported: `createdAt`. Prefix with `-` for descending. |

**Success Response:** `200 OK` with paginated wrapper. Same booking object structure as above.

```json
{
  "data": [
    {
      "id": "b-1",
      "classId": "c-1",
      "memberId": "m-1",
      "createdAt": "2026-04-04T12:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

**Use case:** A member wants to see all their bookings: `GET /api/v1/bookings?memberId=m-1`.

---

#### `GET /api/v1/classes/{classId}/bookings/{bookingId}` -- Get a specific booking

**Description:** Returns a single booking.

**Success Response:** `200 OK`

```json
{
  "id": "b-1",
  "classId": "c-1",
  "memberId": "m-1",
  "createdAt": "2026-04-04T12:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Class does not exist, or booking does not exist, or booking does not belong to this class |

**Note:** If the `bookingId` exists but belongs to a different class, the API returns `404` -- not `400`. From the client's perspective, the booking does not exist within the given class scope.

---

#### `DELETE /api/v1/classes/{classId}/bookings/{bookingId}` -- Cancel (delete) a booking

**Description:** Cancels a booking by deleting it permanently.

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Class does not exist, or booking does not exist, or booking does not belong to this class |

**Side effects:** On successful deletion, the class's `currentBookingCount` is decremented by 1.

**Business Rules Enforced:** None specific -- any booking can be cancelled regardless of class timing or member status. This is intentional: if a member becomes inactive or a class is in the past, they should still be able to clean up their bookings.

---

## 6. Business Rule Coverage Checklist

| Rule # | Rule Summary | Covered in Endpoint(s) |
|--------|-------------|------------------------|
| 1 | Members: unique email, active/inactive status, only active can book | `POST /api/v1/members`, `PUT /api/v1/members/{memberId}`, `POST /api/v1/classes/{classId}/bookings` |
| 2 | Instructors: unique email, qualifications | `POST /api/v1/instructors`, `PUT /api/v1/instructors/{instructorId}` |
| 3 | Classes: title, instructor, time, capacity, room | `POST /api/v1/classes`, `PUT /api/v1/classes/{classId}` |
| 4 | Instructor must hold matching qualification | `POST /api/v1/classes`, `PUT /api/v1/classes/{classId}`, `PUT /api/v1/instructors/{instructorId}` |
| 5 | No room overlap | `POST /api/v1/classes`, `PUT /api/v1/classes/{classId}` |
| 6 | No instructor time overlap | `POST /api/v1/classes`, `PUT /api/v1/classes/{classId}` |
| 7 | No duplicate booking | `POST /api/v1/classes/{classId}/bookings` |
| 8 | Capacity limit on bookings | `POST /api/v1/classes/{classId}/bookings` |
| 9 | No booking for past classes | `POST /api/v1/classes/{classId}/bookings` |
| 10 | Deleting class removes bookings | `DELETE /api/v1/classes/{classId}` |
| 11 | Cannot delete instructor with upcoming classes | `DELETE /api/v1/instructors/{instructorId}` |
| 12 | Cannot delete member with upcoming bookings | `DELETE /api/v1/members/{memberId}` |

All 12 business rules are covered.

---

## 7. Advanced Features

### 7.1 Pagination

Covered in Section 3.5. All list endpoints use consistent pagination with `page`/`pageSize` query parameters and a standardized response wrapper with `totalItems` and `totalPages`.

### 7.2 Filtering and Sorting

Covered in Section 3.6 and in individual endpoint details. Each list endpoint documents its supported filter and sort parameters.

### 7.3 Concurrency Handling

**The problem:** Two members try to book the last spot in a class simultaneously. Without concurrency control, both requests could read `currentBookingCount = 19` (with `maxCapacity = 20`), both see there's room, and both create a booking -- resulting in 21 bookings for a 20-capacity class.

**Solution: Optimistic concurrency with version checking.**

Each class has an `updatedAt` field that acts as a version marker. The booking creation flow:

1. Read the class and check capacity.
2. Attempt to increment `currentBookingCount` with a conditional update: `UPDATE classes SET currentBookingCount = currentBookingCount + 1, updatedAt = NOW() WHERE id = ? AND currentBookingCount < maxCapacity`.
3. If the conditional update affects 0 rows, the class is full -- return `409 Conflict`.

This approach handles the race condition at the database level without requiring distributed locks. The key insight is that the capacity check and the count increment happen in a single atomic database operation.

For class updates (PUT), the API supports `If-Match` headers with ETag values:

- **ETag:** Each class response includes an `ETag` header with the value of `updatedAt` as an opaque string (e.g., `ETag: "2026-04-04T11:00:00Z"`).
- **If-Match:** When updating a class via PUT, the client can send `If-Match: "2026-04-04T11:00:00Z"`. If the class has been modified since (i.e., `updatedAt` doesn't match), the server returns `412 Precondition Failed`.
- **If `If-Match` is not sent:** The update proceeds without concurrency checks (last-write-wins). This keeps the basic flow simple while allowing clients that care about concurrency to opt in.

### 7.4 Idempotency

**The problem:** A client sends `POST /api/v1/classes/c-1/bookings` with `memberId: m-1`. The request succeeds, but the client's network drops before it receives the `201` response. The client retries. Without idempotency handling, should this fail?

**Solution:** For bookings specifically, **natural idempotency** is built in via Rule 7. The unique constraint on `(classId, memberId)` means the retry will hit the `409 Conflict` response. The client should treat `409` on a booking POST as "already booked, probably my earlier request succeeded."

For other POST endpoints, idempotency is not built-in. If needed, an `Idempotency-Key` header pattern could be added:
- Client sends `Idempotency-Key: <uuid>` with the request.
- Server stores the key and the response for a period (e.g., 24 hours).
- On retry with the same key, the server returns the stored response without re-executing.

This is documented here as a design consideration but not fully specified, as the exercise scope doesn't require it for all endpoints.

---

## Appendix: Summary of All Endpoints

For quick reference, here is the complete endpoint map with business rules:

```
QUALIFICATIONS
  POST   /api/v1/qualifications                          [create]
  GET    /api/v1/qualifications                          [list]
  GET    /api/v1/qualifications/{qualificationId}        [read]
  PUT    /api/v1/qualifications/{qualificationId}        [update]
  DELETE /api/v1/qualifications/{qualificationId}        [delete]

MEMBERS
  POST   /api/v1/members                                 [create, Rule 1]
  GET    /api/v1/members                                 [list]
  GET    /api/v1/members/{memberId}                      [read]
  PUT    /api/v1/members/{memberId}                      [update, Rule 1]
  DELETE /api/v1/members/{memberId}                      [delete, Rule 12]

INSTRUCTORS
  POST   /api/v1/instructors                             [create, Rule 2]
  GET    /api/v1/instructors                              [list]
  GET    /api/v1/instructors/{instructorId}               [read]
  PUT    /api/v1/instructors/{instructorId}               [update, Rule 2, 4]
  DELETE /api/v1/instructors/{instructorId}               [delete, Rule 11]

CLASSES
  POST   /api/v1/classes                                 [create, Rules 3-6]
  GET    /api/v1/classes                                  [list]
  GET    /api/v1/classes/{classId}                        [read]
  PUT    /api/v1/classes/{classId}                        [update, Rules 3-6]
  DELETE /api/v1/classes/{classId}                        [delete, Rule 10]

BOOKINGS
  POST   /api/v1/classes/{classId}/bookings              [create, Rules 1,7-9]
  GET    /api/v1/classes/{classId}/bookings               [list for class]
  GET    /api/v1/bookings                                 [list cross-cutting]
  GET    /api/v1/classes/{classId}/bookings/{bookingId}  [read]
  DELETE /api/v1/classes/{classId}/bookings/{bookingId}  [delete/cancel]
```
