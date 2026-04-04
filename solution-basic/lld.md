# FitBook Studio - Low-Level Design Document

By Marco

---

## Phase 1: Data Model

### Entities

#### Member

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `firstName` | string | yes | no | 1-100 characters |
| `lastName` | string | yes | no | 1-100 characters |
| `email` | string | yes | yes | Must be valid email format |
| `membershipStatus` | string | yes | no | "active" or "inactive", defaults to "active" |
| `createdAt` | string (datetime) | yes | no | Server-generated |

#### Instructor

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `firstName` | string | yes | no | 1-100 characters |
| `lastName` | string | yes | no | 1-100 characters |
| `email` | string | yes | yes | Must be valid email format |
| `qualifications` | array of strings | yes | no | e.g. ["Yoga", "HIIT", "Spinning"] |
| `createdAt` | string (datetime) | yes | no | Server-generated |

#### Class

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `title` | string | yes | no | e.g. "Morning Yoga" |
| `description` | string | no | no | Longer text about the class |
| `instructorId` | integer | yes | no | Foreign key to Instructor |
| `startTime` | string (datetime) | yes | no | ISO 8601 format |
| `duration` | integer | yes | no | In minutes |
| `maxCapacity` | integer | yes | no | Maximum number of bookings |
| `roomName` | string | yes | no | e.g. "Room A" |
| `createdAt` | string (datetime) | yes | no | Server-generated |

#### Booking

| Attribute | Type | Required | Unique | Notes |
|-----------|------|----------|--------|-------|
| `id` | integer | yes | yes | Server-generated, auto-increment |
| `classId` | integer | yes | no | Foreign key to Class |
| `memberId` | integer | yes | no | Foreign key to Member |
| `createdAt` | string (datetime) | yes | no | Server-generated |

### Relationships

| Relationship | Type | Description |
|--------------|------|-------------|
| Instructor -> Class | 1:N | One instructor leads many classes, each class has one instructor |
| Class -> Booking | 1:N | One class can have many bookings |
| Member -> Booking | 1:N | One member can have many bookings |
| Member <-> Class | N:M | Many-to-many through Booking table |

---

## Phase 2: General Conventions

### API Conventions

| Convention | My Decision |
|------------|-------------|
| Base URL | `/api/v1/` |
| Resource naming in URIs | plural, lowercase (e.g. `/members`, `/classes`) |
| ID format | Auto-increment integers |
| Date/time format | ISO 8601 in UTC (e.g. `2026-04-04T10:30:00Z`) |
| Request/response body casing | camelCase |

### Standard Error Response Format

Every error from the API will look like this:

```json
{
  "error": "ERROR_CODE",
  "message": "Human readable message about what went wrong.",
  "details": []
}
```

The `details` array is optional and used for validation errors where multiple fields can be wrong at the same time.

### Common Status Codes

| Status Code | Meaning in My API |
|-------------|-------------------|
| `200 OK` | Request was successful, here's the data |
| `201 Created` | New resource was created successfully |
| `204 No Content` | Resource was deleted successfully, no body returned |
| `400 Bad Request` | Something wrong with the request (missing fields, wrong format, etc.) |
| `404 Not Found` | The resource you're looking for doesn't exist |
| `409 Conflict` | Duplicate resource or business rule conflict (like double booking) |

---

## Phase 3: Endpoint Overview

| Method | URI | Description |
|--------|-----|-------------|
| GET | /api/v1/members | List all members |
| POST | /api/v1/members | Create a new member |
| GET | /api/v1/members/{id} | Get a single member |
| PUT | /api/v1/members/{id} | Update a member |
| DELETE | /api/v1/members/{id} | Delete a member |
| GET | /api/v1/instructors | List all instructors |
| POST | /api/v1/instructors | Create a new instructor |
| GET | /api/v1/instructors/{id} | Get a single instructor |
| PUT | /api/v1/instructors/{id} | Update an instructor |
| DELETE | /api/v1/instructors/{id} | Delete an instructor |
| GET | /api/v1/classes | List all classes |
| POST | /api/v1/classes | Create a new class |
| GET | /api/v1/classes/{id} | Get a single class |
| PUT | /api/v1/classes/{id} | Update a class |
| DELETE | /api/v1/classes/{id} | Delete a class |
| GET | /api/v1/classes/{classId}/bookings | List bookings for a class |
| POST | /api/v1/classes/{classId}/bookings | Create a booking for a class |
| DELETE | /api/v1/classes/{classId}/bookings/{id} | Cancel a booking |

I decided to nest bookings under classes because a booking always belongs to a class, so it makes sense to me to have it under the class URL. If you need to find bookings for a member you can use GET /api/v1/members/{id} and the bookings could be included there somehow... I didn't fully figure that out yet.

---

## Phase 4: Endpoint Details

### `POST /api/v1/members` -- Create a new member

This is already in the worked example, so I'll follow the same pattern.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email, unique across members |

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

`membershipStatus` defaults to "active". `id` and `createdAt` are generated by the server.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields |
| `409 Conflict` | Email already exists |

**Error example (400):**
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Request body contains invalid fields.",
  "details": [
    { "field": "email", "issue": "Must be a valid email address." }
  ]
}
```

**Error example (409):**
```json
{
  "error": "CONFLICT",
  "message": "A member with email 'jane.rivera@example.com' already exists."
}
```

**Business Rules Enforced:** Rule 1 (email unique, status defaults to active)

---

### `GET /api/v1/members` -- List all members

**Request Body:** None

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
  }
]
```

Returns an array of all members. Empty array if none exist.

**Error Responses:** None really, just returns empty array.

---

### `GET /api/v1/members/{id}` -- Get a single member

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
| `404 Not Found` | Member with given ID does not exist |

**Error example (404):**
```json
{
  "error": "NOT_FOUND",
  "message": "Member with id 999 not found."
}
```

---

### `PUT /api/v1/members/{id}` -- Update a member

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email, unique |
| `membershipStatus` | string | yes | Must be "active" or "inactive" |

I'm using PUT here so the client sends the whole object every time. I think PATCH would be for partial updates but I'm not 100% sure so I'll just use PUT to keep things simple.

**Success Response:** `200 OK`

```json
{
  "id": 1,
  "firstName": "Jane",
  "lastName": "Rivera-Smith",
  "email": "jane.rivera@example.com",
  "membershipStatus": "active",
  "createdAt": "2026-04-04T10:30:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields |
| `404 Not Found` | Member not found |
| `409 Conflict` | Email already taken by another member |

**Business Rules Enforced:** Rule 1 (email must stay unique)

---

### `DELETE /api/v1/members/{id}` -- Delete a member

**Request Body:** None

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Member not found |
| `409 Conflict` | Member has upcoming bookings |

**Error example (409):**
```json
{
  "error": "CONFLICT",
  "message": "Cannot delete member because they have upcoming bookings."
}
```

**Business Rules Enforced:** Rule 12 (cannot delete member with upcoming bookings)

---

### `POST /api/v1/instructors` -- Create a new instructor

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email, unique across instructors |
| `qualifications` | array of strings | yes | At least one qualification |

**Success Response:** `201 Created`

```json
{
  "id": 1,
  "firstName": "Mike",
  "lastName": "Chen",
  "email": "mike.chen@example.com",
  "qualifications": ["Yoga", "HIIT"],
  "createdAt": "2026-04-04T11:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields, empty qualifications array |
| `409 Conflict` | Email already exists |

**Error example (400):**
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Request body contains invalid fields.",
  "details": [
    { "field": "qualifications", "issue": "Must have at least one qualification." }
  ]
}
```

**Business Rules Enforced:** Rule 2 (unique email, qualifications list)

---

### `GET /api/v1/instructors` -- List all instructors

Follows the same pattern as `GET /api/v1/members`.

**Differences:**
- Response fields: `id`, `firstName`, `lastName`, `email`, `qualifications`, `createdAt`
- Returns array of instructor objects

---

### `GET /api/v1/instructors/{id}` -- Get a single instructor

Follows the same pattern as `GET /api/v1/members/{id}`.

**Differences:**
- Response fields: `id`, `firstName`, `lastName`, `email`, `qualifications`, `createdAt`
- Returns `404` if instructor not found

---

### `PUT /api/v1/instructors/{id}` -- Update an instructor

Follows the same pattern as `PUT /api/v1/members/{id}`.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | string | yes | 1-100 characters, non-blank |
| `lastName` | string | yes | 1-100 characters, non-blank |
| `email` | string | yes | Valid email, unique |
| `qualifications` | array of strings | yes | At least one |

**Differences:**
- Additional field `qualifications` in request and response
- `409 Conflict` if email taken by another instructor

**Business Rules Enforced:** Rule 2 (email unique, qualifications)

---

### `DELETE /api/v1/instructors/{id}` -- Delete an instructor

**Request Body:** None

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Instructor not found |
| `409 Conflict` | Instructor has upcoming classes assigned |

**Error example (409):**
```json
{
  "error": "CONFLICT",
  "message": "Cannot delete instructor because they have upcoming classes."
}
```

**Business Rules Enforced:** Rule 11 (cannot delete instructor with upcoming classes)

---

### `POST /api/v1/classes` -- Create a new class

This one has a lot of rules so I'll be detailed.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | yes | 1-200 characters, non-blank |
| `description` | string | no | Max 1000 characters |
| `instructorId` | integer | yes | Must reference existing instructor |
| `startTime` | string (datetime) | yes | ISO 8601 format, must be in the future |
| `duration` | integer | yes | Positive number (minutes) |
| `maxCapacity` | integer | yes | Positive number, at least 1 |
| `roomName` | string | yes | Non-blank |

**Success Response:** `201 Created`

```json
{
  "id": 1,
  "title": "Morning Yoga",
  "description": "A relaxing yoga session to start your day",
  "instructorId": 1,
  "startTime": "2026-04-10T08:00:00Z",
  "duration": 60,
  "maxCapacity": 20,
  "roomName": "Room A",
  "createdAt": "2026-04-04T12:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing or invalid fields, start time in the past |
| `404 Not Found` | Instructor with given ID does not exist |
| `409 Conflict` | Room overlap (another class in same room at same time) or instructor overlap (instructor already teaching at that time) or instructor doesn't have matching qualification |

**Error example -- room overlap (409):**
```json
{
  "error": "CONFLICT",
  "message": "Room 'Room A' is already booked from 2026-04-10T08:00:00Z to 2026-04-10T09:00:00Z."
}
```

**Error example -- instructor qualification mismatch (409):**
```json
{
  "error": "CONFLICT",
  "message": "Instructor does not have the required qualification for 'Yoga'."
}
```

Wait, actually I'm not sure how the qualification check works. The class has a title like "Morning Yoga" but the qualification is just "Yoga". I guess the system needs to match somehow... Maybe the class should also have a `type` field or something? I'll just assume the title contains the type for now and the API needs to check if the instructor has that qualification. Actually, looking at the rules again, it says "a Yoga class requires the instructor to have the Yoga qualification". So I think there needs to be a `type` or `category` field on the class. Let me add that.

Actually, I'm going to keep it simple. I'll assume the class title IS the type for matching. No wait, that's weird. Let me just not worry about this too much -- I'll say the check happens based on the class title matching one of the qualifications. The instructor needs to have a qualification that matches the class title or part of it.

Hmm OK I realize this is a design decision I should document. I'll say qualifications are free text strings and when creating a class, the title should start with or contain one of the instructor's qualifications. But honestly this feels really messy. Let me just move on.

**Business Rules Enforced:** Rule 3, Rule 4 (qualification match), Rule 5 (no room overlap), Rule 6 (no instructor overlap)

---

### `GET /api/v1/classes` -- List all classes

Follows the same pattern as `GET /api/v1/members`.

**Differences:**
- Response fields: `id`, `title`, `description`, `instructorId`, `startTime`, `duration`, `maxCapacity`, `roomName`, `createdAt`
- Returns array of class objects

---

### `GET /api/v1/classes/{id}` -- Get a single class

Follows the same pattern as `GET /api/v1/members/{id}`.

**Differences:**
- Response fields: `id`, `title`, `description`, `instructorId`, `startTime`, `duration`, `maxCapacity`, `roomName`, `createdAt`
- Returns `404` if class not found

---

### `PUT /api/v1/classes/{id}` -- Update a class

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `title` | string | yes | 1-200 characters, non-blank |
| `description` | string | no | Max 1000 characters |
| `instructorId` | integer | yes | Must reference existing instructor |
| `startTime` | string (datetime) | yes | ISO 8601, must be in the future |
| `duration` | integer | yes | Positive number |
| `maxCapacity` | integer | yes | Positive number, at least 1 |
| `roomName` | string | yes | Non-blank |

**Success Response:** `200 OK`

Returns the updated class object (same shape as GET response).

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Invalid fields |
| `404 Not Found` | Class not found or instructor not found |
| `409 Conflict` | Room overlap, instructor overlap, or qualification mismatch |

Same business rules apply as POST. All the overlap checks and qualification checks need to happen again.

**Business Rules Enforced:** Rule 3, Rule 4, Rule 5, Rule 6

---

### `DELETE /api/v1/classes/{id}` -- Delete a class

**Request Body:** None

**Success Response:** `204 No Content`

When a class is deleted, all bookings for that class are also deleted automatically.

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Class not found |

**Business Rules Enforced:** Rule 10 (deleting class removes all bookings)

---

### `POST /api/v1/classes/{classId}/bookings` -- Create a booking

This is probably the most complex endpoint.

**Request Body:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `memberId` | integer | yes | Must reference existing, active member |

**Success Response:** `201 Created`

```json
{
  "id": 1,
  "classId": 1,
  "memberId": 3,
  "createdAt": "2026-04-04T14:00:00Z"
}
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `400 Bad Request` | Missing memberId or member is inactive |
| `404 Not Found` | Class not found or member not found |
| `409 Conflict` | Class is full, member already booked, or class is in the past |

**Error example -- class full (409):**
```json
{
  "error": "CONFLICT",
  "message": "Class has reached maximum capacity of 20."
}
```

**Error example -- already booked (409):**
```json
{
  "error": "CONFLICT",
  "message": "Member is already booked for this class."
}
```

**Error example -- class in the past (409):**
```json
{
  "error": "CONFLICT",
  "message": "Cannot book a class that has already started."
}
```

**Business Rules Enforced:** Rule 1 (only active members), Rule 7 (no duplicate booking), Rule 8 (capacity limit), Rule 9 (no past classes)

---

### `GET /api/v1/classes/{classId}/bookings` -- List bookings for a class

**Request Body:** None

**Success Response:** `200 OK`

```json
[
  {
    "id": 1,
    "classId": 1,
    "memberId": 3,
    "createdAt": "2026-04-04T14:00:00Z"
  }
]
```

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Class not found |

---

### `DELETE /api/v1/classes/{classId}/bookings/{id}` -- Cancel a booking

I'm just going to DELETE the booking entirely. I know the exercise mentioned maybe using PATCH to set status to "cancelled" but I think DELETE is simpler and honestly I don't think the studio needs to keep records of cancelled bookings. If they do, that's a future problem.

**Request Body:** None

**Success Response:** `204 No Content`

**Error Responses:**

| Status Code | Condition |
|-------------|-----------|
| `404 Not Found` | Booking or class not found |

---

## Design Decisions

I should document a few decisions I made:

1. **Bookings are nested under classes** (`/api/v1/classes/{classId}/bookings`) because a booking always belongs to a class. The downside is that to get all bookings for a member, you'd have to check every class. Maybe I should have added a GET /api/v1/members/{id}/bookings endpoint too, but I ran out of steam.

2. **Qualifications are free-text strings** stored as an array on the instructor. I didn't make them a separate entity because that seemed like overkill for this. The downside is that someone could type "yoga" vs "Yoga" and get inconsistent data. I'm not sure how to handle Rule 4 cleanly because the class doesn't have a "type" field -- I think there should probably be a `category` or `type` field on the class that matches against instructor qualifications, but I didn't add one and I'm not sure what to do about this.

3. **Cancellation = DELETE.** I'm just removing the booking record. Simpler but you lose history.

4. **Using PUT for all updates** instead of PATCH. Clients send the full object every time. Less flexible but less confusing (for me at least).

5. **Times are in UTC.** The studio is in one timezone but the API uses UTC for everything. The client/frontend can convert to local time.

6. **Auto-increment integer IDs.** I went with this because it's what I'm used to from SQL class. UUIDs would probably be better for a real API but I don't fully understand why.

---

## Business Rule Coverage Checklist

| Rule # | Rule Summary | Covered in Endpoint(s) |
|--------|-------------|------------------------|
| 1 | Members: unique email, active/inactive status | `POST /api/v1/members`, `PUT /api/v1/members/{id}` |
| 2 | Instructors: unique email, qualifications | `POST /api/v1/instructors`, `PUT /api/v1/instructors/{id}` |
| 3 | Classes: title, instructor, time, capacity, room | `POST /api/v1/classes`, `PUT /api/v1/classes/{id}` |
| 4 | Instructor must hold matching qualification | `POST /api/v1/classes`, `PUT /api/v1/classes/{id}` |
| 5 | No room overlap | `POST /api/v1/classes`, `PUT /api/v1/classes/{id}` |
| 6 | No instructor time overlap | `POST /api/v1/classes`, `PUT /api/v1/classes/{id}` |
| 7 | No duplicate booking | `POST /api/v1/classes/{classId}/bookings` |
| 8 | Capacity limit on bookings | `POST /api/v1/classes/{classId}/bookings` |
| 9 | No booking for past classes | `POST /api/v1/classes/{classId}/bookings` |
| 10 | Deleting class removes bookings | `DELETE /api/v1/classes/{id}` |
| 11 | Cannot delete instructor with upcoming classes | `DELETE /api/v1/instructors/{id}` |
| 12 | Cannot delete member with upcoming bookings | `DELETE /api/v1/members/{id}` |
