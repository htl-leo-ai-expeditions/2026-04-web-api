# FitBook Studio — Low-Level Design Document

**Author:** Sofia M.
**Date:** 2026-04-04

---

## Phase 1: Data Model

### Entities and Attributes

#### Member

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `firstName` | string | yes | no | 1-100 characters, non-blank |
| `lastName` | string | yes | no | 1-100 characters, non-blank |
| `email` | string | yes | yes | Valid email format |
| `membershipStatus` | string | yes | no | Either `"active"` or `"inactive"`, defaults to `"active"` |
| `createdAt` | string (datetime) | yes | no | Server-generated, ISO 8601 UTC |

#### Instructor

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `firstName` | string | yes | no | 1-100 characters, non-blank |
| `lastName` | string | yes | no | 1-100 characters, non-blank |
| `email` | string | yes | yes | Valid email format |
| `qualifications` | array of strings | yes | no | At least one required, e.g. `["Yoga", "HIIT"]` |
| `createdAt` | string (datetime) | yes | no | Server-generated, ISO 8601 UTC |

#### Class

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `title` | string | yes | no | 1-150 characters, non-blank |
| `description` | string | no | no | Up to 1000 characters |
| `instructorId` | integer | yes | no | Foreign key to Instructor |
| `startTime` | string (datetime) | yes | no | ISO 8601 UTC, must be in the future when creating |
| `duration` | integer | yes | no | In minutes, must be > 0 |
| `maxCapacity` | integer | yes | no | Must be >= 1 |
| `roomName` | string | yes | no | 1-100 characters, non-blank |
| `createdAt` | string (datetime) | yes | no | Server-generated, ISO 8601 UTC |

#### Booking

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `classId` | integer | yes | no | Foreign key to Class |
| `memberId` | integer | yes | no | Foreign key to Member |
| `bookedAt` | string (datetime) | yes | no | Server-generated, ISO 8601 UTC |

### Relationships

| Relationship | Type | Description |
|--------------|------|-------------|
| Instructor -> Class | 1:N | An instructor can lead many classes. Each class has exactly one instructor. |
| Class -> Booking | 1:N | A class can have many bookings (up to maxCapacity). |
| Member -> Booking | 1:N | A member can have many bookings across different classes. |
| Member <-> Class | N:M | Many-to-many through the Booking entity. A member can book many classes, and a class can have many members. |

### Design Decisions

**Bookings as a resource:** I decided to make bookings a nested resource under classes (`/api/v1/classes/{classId}/bookings`). This makes sense because a booking is always in the context of a specific class. To look up all bookings for a member, I'll add a query parameter on the GET endpoint to filter by memberId. You could also add a convenience endpoint like `GET /api/v1/members/{memberId}/bookings` but I'll keep it simple for now.

**Modeling qualifications:** I chose to store qualifications as a free-text array of strings on the instructor. This is the simplest approach. When a class is created, the server checks whether the instructor's qualifications list contains the class title (or a matching keyword). The downside is there's no centralized list of valid qualifications, so typos could happen, but it keeps things simple.

**Cancellation vs. deletion:** I'm going with DELETE for cancelling bookings. When a member wants to cancel, we just delete the booking record. This is simpler but it does mean we lose the history of who cancelled. For a small studio this seems fine.

**Time zone handling:** All times are stored and returned in UTC (ISO 8601 format with the `Z` suffix). The client is responsible for converting to local time for display. This avoids confusion about time zones.

---

## Phase 2: General Conventions

| Convention | Decision |
|------------|----------|
| Base URL | `/api/v1/` |
| Resource naming in URIs | Plural, lowercase (e.g. `/members`, `/classes`) |
| ID format | Auto-increment integers |
| Date/time format | ISO 8601 in UTC (e.g. `2026-04-04T10:30:00Z`) |
| Request/response body casing | camelCase |
| Pagination | Not implemented (stretch goal) |

### Standard Error Response Format

All error responses follow this structure:

```json
{
  "error": "ERROR_CODE",
  "message": "A human-readable description of what went wrong.",
  "details": [
    { "field": "fieldName", "issue": "What's wrong with this field." }
  ]
}
```

- `error`: A machine-readable error code in UPPER_SNAKE_CASE (e.g. `VALIDATION_ERROR`, `CONFLICT`, `NOT_FOUND`)
- `message`: A human-readable summary
- `details`: An optional array with field-level errors (used for validation errors)

### Common HTTP Status Codes

| Status Code | Meaning in This API |
|-------------|---------------------|
| `200 OK` | Request succeeded. Used for GET (returning data) and PATCH (returning updated resource). |
| `201 Created` | A new resource was successfully created. Used for POST. |
| `204 No Content` | Request succeeded but there's nothing to return. Used for DELETE. |
| `400 Bad Request` | The request body is invalid — missing required fields, wrong types, failed validation. |
| `404 Not Found` | The requested resource doesn't exist. |
| `409 Conflict` | The request conflicts with current state — e.g. duplicate email, booking already exists, room overlap. |

---

## Phase 3: Endpoint Overview

| Method | URI | Description |
|--------|-----|-------------|
| GET | `/api/v1/members` | List all members |
| POST | `/api/v1/members` | Create a new member |
| GET | `/api/v1/members/{memberId}` | Get a specific member |
| PATCH | `/api/v1/members/{memberId}` | Update a member |
| DELETE | `/api/v1/members/{memberId}` | Delete a member |
| GET | `/api/v1/instructors` | List all instructors |
| POST | `/api/v1/instructors` | Create a new instructor |
| GET | `/api/v1/instructors/{instructorId}` | Get a specific instructor |
| PATCH | `/api/v1/instructors/{instructorId}` | Update an instructor |
| DELETE | `/api/v1/instructors/{instructorId}` | Delete an instructor |
| GET | `/api/v1/classes` | List all classes |
| POST | `/api/v1/classes` | Create a new class |
| GET | `/api/v1/classes/{classId}` | Get a specific class |
| PATCH | `/api/v1/classes/{classId}` | Update a class |
| DELETE | `/api/v1/classes/{classId}` | Delete a class and its bookings |
| GET | `/api/v1/classes/{classId}/bookings` | List all bookings for a class |
| POST | `/api/v1/classes/{classId}/bookings` | Book a member into a class |
| DELETE | `/api/v1/classes/{classId}/bookings/{bookingId}` | Cancel (delete) a booking |

---

## Phase 4: Endpoint Details

### `POST /api/v1/members` — Create a new member

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Must be a valid email format, unique across all members |

**Success Response:** `201 Created`

```json
{
  "id": 1,
  "firstName": "Jane",
  "lastName": "Rivera",
  "email": "jane.rivera@example.com",
  "membershipStatus": "active",
  "createdAt": "2026-04-04T10:30:00Z"
}
```

Note: `membershipStatus` defaults to `"active"`. `id` and `createdAt` are server-generated.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields (e.g. blank name, malformed email) |
| `409 Conflict` | A member with this email already exists |

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

**Business Rules Enforced:** Rule 1 (unique email, defaults to active)

---

### `GET /api/v1/members` — List all members

**Request Body:** None

**Query Parameters:** None

**Success Response:** `200 OK`

```json
[
  {
    "id": 1,
    "firstName": "Jane",
    "lastName": "Rivera",
    "email": "jane.rivera@example.com",
    "membershipStatus": "active",
    "createdAt": "2026-04-04T10:30:00Z"
  },
  {
    "id": 2,
    "firstName": "Tom",
    "lastName": "Chen",
    "email": "tom.chen@example.com",
    "membershipStatus": "inactive",
    "createdAt": "2026-04-03T08:00:00Z"
  }
]
```

**Error Responses:** None specific — returns an empty array `[]` if no members exist.

**Business Rules Enforced:** None

---

### `GET /api/v1/members/{memberId}` — Get a specific member

**Request Body:** None

**Success Response:** `200 OK`

```json
{
  "id": 1,
  "firstName": "Jane",
  "lastName": "Rivera",
  "email": "jane.rivera@example.com",
  "membershipStatus": "active",
  "createdAt": "2026-04-04T10:30:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No member with this ID exists |

**Business Rules Enforced:** None

---

### `PATCH /api/v1/members/{memberId}` — Update a member

**Request Body:** (all fields optional — only include what you want to change)

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | no | 1-100 characters, non-blank |
| `lastName` | string | no | 1-100 characters, non-blank |
| `email` | string | no | Valid email format, unique |
| `membershipStatus` | string | no | Must be `"active"` or `"inactive"` |

**Success Response:** `200 OK`

Returns the full updated member object.

```json
{
  "id": 1,
  "firstName": "Jane",
  "lastName": "Rivera-Smith",
  "email": "jane.rivera@example.com",
  "membershipStatus": "inactive",
  "createdAt": "2026-04-04T10:30:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Invalid field values |
| `404 Not Found` | No member with this ID exists |
| `409 Conflict` | Email already taken by another member |

**Business Rules Enforced:** Rule 1 (email must remain unique, status must be valid)

---

### `DELETE /api/v1/members/{memberId}` — Delete a member

**Request Body:** None

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No member with this ID exists |
| `409 Conflict` | Member has upcoming bookings (classes that start in the future) |

**Error example — has upcoming bookings (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot delete member because they have upcoming bookings."
}
```

**Business Rules Enforced:** Rule 12 (cannot delete member with upcoming bookings)

---

### `POST /api/v1/instructors` — Create a new instructor

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email format, unique across all instructors |
| `qualifications` | array of strings | yes | At least one qualification required, each non-blank |

**Success Response:** `201 Created`

```json
{
  "id": 1,
  "firstName": "Mike",
  "lastName": "Taylor",
  "email": "mike.taylor@example.com",
  "qualifications": ["Yoga", "HIIT"],
  "createdAt": "2026-04-04T09:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields (e.g. empty qualifications array) |
| `409 Conflict` | An instructor with this email already exists |

**Error example — validation failure (`400`):**

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Request body contains invalid fields.",
  "details": [
    { "field": "qualifications", "issue": "Must contain at least one qualification." }
  ]
}
```

**Business Rules Enforced:** Rule 2 (unique email, qualifications list)

---

### `GET /api/v1/instructors` — List all instructors

Follows the same pattern as `GET /api/v1/members`.

**Differences:**
- Response fields: `id`, `firstName`, `lastName`, `email`, `qualifications`, `createdAt`
- Returns an empty array if no instructors exist

**Business Rules Enforced:** None

---

### `GET /api/v1/instructors/{instructorId}` — Get a specific instructor

Follows the same pattern as `GET /api/v1/members/{memberId}`.

**Differences:**
- Response fields: `id`, `firstName`, `lastName`, `email`, `qualifications`, `createdAt`
- Returns `404` if instructor not found

**Business Rules Enforced:** None

---

### `PATCH /api/v1/instructors/{instructorId}` — Update an instructor

Follows the same pattern as `PATCH /api/v1/members/{memberId}`.

**Request Body:** (all fields optional)

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | no | 1-100 characters, non-blank |
| `lastName` | string | no | 1-100 characters, non-blank |
| `email` | string | no | Valid email format, unique |
| `qualifications` | array of strings | no | At least one qualification if provided |

**Differences:**
- Additional status code: `409 Conflict` if email is already taken by another instructor
- Response returns the full updated instructor object

**Business Rules Enforced:** Rule 2 (unique email, qualifications)

---

### `DELETE /api/v1/instructors/{instructorId}` — Delete an instructor

**Request Body:** None

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No instructor with this ID exists |
| `409 Conflict` | Instructor has upcoming classes (classes with a start time in the future) |

**Error example — has upcoming classes (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot delete instructor because they have upcoming classes assigned."
}
```

**Business Rules Enforced:** Rule 11 (cannot delete instructor with upcoming classes)

---

### `POST /api/v1/classes` — Create a new class

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | yes | 1-150 characters, non-blank |
| `description` | string | no | Up to 1000 characters |
| `instructorId` | integer | yes | Must reference an existing instructor |
| `startTime` | string (datetime) | yes | ISO 8601 UTC, must be in the future |
| `duration` | integer | yes | Positive integer (minutes), must be > 0 |
| `maxCapacity` | integer | yes | Must be >= 1 |
| `roomName` | string | yes | 1-100 characters, non-blank |

**Success Response:** `201 Created`

```json
{
  "id": 1,
  "title": "Morning Yoga",
  "description": "A relaxing morning yoga session for all levels.",
  "instructorId": 1,
  "startTime": "2026-04-10T08:00:00Z",
  "duration": 60,
  "maxCapacity": 20,
  "roomName": "Studio A",
  "createdAt": "2026-04-04T11:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields, startTime is in the past, duration <= 0, capacity < 1 |
| `404 Not Found` | The referenced instructorId does not exist |
| `409 Conflict` | Instructor does not have a matching qualification for this class title |
| `409 Conflict` | Room is already booked for an overlapping time slot |
| `409 Conflict` | Instructor already has a class at an overlapping time |

**Error example — instructor qualification mismatch (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Instructor 1 does not have the 'Morning Yoga' qualification required for this class."
}
```

**Wait — qualification matching note:** The qualification check compares the class title against the instructor's qualifications list. For example, if the class title is "Morning Yoga", the instructor needs to have "Yoga" in their qualifications. Honestly, I'm not 100% sure how the matching should work here. I'll say the class title must contain one of the instructor's qualifications as a substring, case-insensitive. So a class titled "Morning Yoga" would match an instructor with qualification "Yoga".

**Error example — room overlap (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Room 'Studio A' is already booked from 2026-04-10T07:30:00Z to 2026-04-10T08:30:00Z."
}
```

**Error example — instructor time overlap (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Instructor 1 already has a class scheduled from 2026-04-10T07:30:00Z to 2026-04-10T08:30:00Z."
}
```

**Overlap calculation:** Two classes overlap if they are in the same room (or same instructor) and the time ranges `[startTime, startTime + duration)` intersect. That is, class A overlaps with class B if `A.startTime < B.startTime + B.duration AND B.startTime < A.startTime + A.duration`.

**Business Rules Enforced:** Rule 3 (class attributes), Rule 4 (instructor qualification match), Rule 5 (no room overlap), Rule 6 (no instructor time overlap)

---

### `GET /api/v1/classes` — List all classes

**Request Body:** None

**Success Response:** `200 OK`

```json
[
  {
    "id": 1,
    "title": "Morning Yoga",
    "description": "A relaxing morning yoga session for all levels.",
    "instructorId": 1,
    "startTime": "2026-04-10T08:00:00Z",
    "duration": 60,
    "maxCapacity": 20,
    "roomName": "Studio A",
    "createdAt": "2026-04-04T11:00:00Z"
  }
]
```

Returns an empty array `[]` if no classes exist.

**Business Rules Enforced:** None

---

### `GET /api/v1/classes/{classId}` — Get a specific class

Follows the same pattern as `GET /api/v1/members/{memberId}`.

**Differences:**
- Response fields: `id`, `title`, `description`, `instructorId`, `startTime`, `duration`, `maxCapacity`, `roomName`, `createdAt`
- Returns `404` if class not found

**Business Rules Enforced:** None

---

### `PATCH /api/v1/classes/{classId}` — Update a class

**Request Body:** (all fields optional)

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | no | 1-150 characters, non-blank |
| `description` | string | no | Up to 1000 characters |
| `instructorId` | integer | no | Must reference an existing instructor |
| `startTime` | string (datetime) | no | ISO 8601 UTC |
| `duration` | integer | no | Must be > 0 |
| `maxCapacity` | integer | no | Must be >= 1 |
| `roomName` | string | no | 1-100 characters, non-blank |

**Success Response:** `200 OK`

Returns the full updated class object.

```json
{
  "id": 1,
  "title": "Morning Yoga",
  "description": "Updated description.",
  "instructorId": 2,
  "startTime": "2026-04-10T08:00:00Z",
  "duration": 60,
  "maxCapacity": 15,
  "roomName": "Studio A",
  "createdAt": "2026-04-04T11:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Invalid field values |
| `404 Not Found` | Class or referenced instructor not found |
| `409 Conflict` | New instructor doesn't have matching qualification |
| `409 Conflict` | Updated time/room would cause a room overlap |
| `409 Conflict` | Updated time/instructor would cause an instructor overlap |
| `409 Conflict` | New maxCapacity is less than the current number of bookings |

**Error example — capacity reduction (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot reduce capacity to 5 because there are already 8 bookings for this class."
}
```

**Note on updating instructor:** If the instructor is changed, existing bookings stay. The class still happens, just with a different instructor. The new instructor must have the matching qualification.

**Business Rules Enforced:** Rule 4 (qualification check on reassignment), Rule 5 (room overlap), Rule 6 (instructor overlap)

---

### `DELETE /api/v1/classes/{classId}` — Delete a class

**Request Body:** None

**Success Response:** `204 No Content`

When a class is deleted, all associated bookings are automatically removed.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | No class with this ID exists |

**Business Rules Enforced:** Rule 10 (deleting a class removes all associated bookings)

---

### `POST /api/v1/classes/{classId}/bookings` — Create a booking

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `memberId` | integer | yes | Must reference an existing member |

**Success Response:** `201 Created`

```json
{
  "id": 1,
  "classId": 1,
  "memberId": 3,
  "bookedAt": "2026-04-04T12:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing memberId or invalid format |
| `404 Not Found` | Class does not exist or member does not exist |
| `409 Conflict` | Member is not active (membershipStatus is "inactive") |
| `409 Conflict` | Class has already started or is in the past |
| `409 Conflict` | Class is at full capacity |
| `409 Conflict` | Member has already booked this class |

**Error example — inactive member (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Member 3 cannot book because their membership status is inactive."
}
```

**Error example — class full (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Class 1 has reached its maximum capacity of 20."
}
```

**Error example — already booked (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Member 3 has already booked class 1."
}
```

**Error example — class in the past (`409`):**

```json
{
  "error": "CONFLICT",
  "message": "Cannot book class 1 because it has already started."
}
```

**Business Rules Enforced:** Rule 1 (only active members can book), Rule 7 (no duplicate booking), Rule 8 (capacity limit), Rule 9 (no booking for past classes)

---

### `GET /api/v1/classes/{classId}/bookings` — List bookings for a class

**Request Body:** None

**Success Response:** `200 OK`

```json
[
  {
    "id": 1,
    "classId": 1,
    "memberId": 3,
    "bookedAt": "2026-04-04T12:00:00Z"
  },
  {
    "id": 2,
    "classId": 1,
    "memberId": 5,
    "bookedAt": "2026-04-04T12:05:00Z"
  }
]
```

Returns an empty array if no bookings exist for this class.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Class does not exist |

**Business Rules Enforced:** None

---

### `DELETE /api/v1/classes/{classId}/bookings/{bookingId}` — Cancel a booking

**Request Body:** None

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Class or booking does not exist |

**Note:** The bookingId must belong to the specified classId. If the booking exists but belongs to a different class, return `404`.

**Business Rules Enforced:** None directly (this is a cancellation action)

---

## Business Rule Coverage Checklist

| Rule # | Rule Summary | Covered in Endpoint(s) |
|--------|-------------|------------------------|
| 1 | Members: unique email, active/inactive status, only active can book | `POST /api/v1/members`, `PATCH /api/v1/members/{memberId}`, `POST /api/v1/classes/{classId}/bookings` |
| 2 | Instructors: unique email, qualifications | `POST /api/v1/instructors`, `PATCH /api/v1/instructors/{instructorId}` |
| 3 | Classes: title, instructor, time, capacity, room | `POST /api/v1/classes` |
| 4 | Instructor must hold matching qualification | `POST /api/v1/classes`, `PATCH /api/v1/classes/{classId}` |
| 5 | No room overlap | `POST /api/v1/classes`, `PATCH /api/v1/classes/{classId}` |
| 6 | No instructor time overlap | `POST /api/v1/classes`, `PATCH /api/v1/classes/{classId}` |
| 7 | No duplicate booking | `POST /api/v1/classes/{classId}/bookings` |
| 8 | Capacity limit on bookings | `POST /api/v1/classes/{classId}/bookings` |
| 9 | No booking for past classes | `POST /api/v1/classes/{classId}/bookings` |
| 10 | Deleting class removes bookings | `DELETE /api/v1/classes/{classId}` |
| 11 | Cannot delete instructor with upcoming classes | `DELETE /api/v1/instructors/{instructorId}` |
| 12 | Cannot delete member with upcoming bookings | `DELETE /api/v1/members/{memberId}` |
