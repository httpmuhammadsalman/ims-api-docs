# Invitation APIs

## Overview

The invitation system allows company owners to invite users to join their companies. The system handles both new users (not yet registered) and existing users (already registered).

---

## Invitation Flow

### For New Users (Not Registered)

1. Owner sends invitation via `/api/v1/auth/invite`
2. User receives invitation email with token
3. User registers via `/api/v1/auth/register` with `invitation_token`
4. Account is created and automatically linked to the company
5. Invitation status changes to `accepted`

### For Existing Users (Already Registered)

1. Owner sends invitation via `/api/v1/auth/invite`
2. System detects user is already registered and sets `invited_user_id`
3. User receives invitation email with token
4. User logs in via `/api/v1/auth/login`
5. User accepts invitation via `/api/v1/auth/invite/accept` with token
6. Company access is granted
7. Invitation status changes to `accepted`

---

## Invite User

Invite a user to join a company. If `company_id` is not provided, the owner's default company will be used. If the user is already registered, their `invited_user_id` will be set in the invitation.

**Endpoint:** `POST /api/v1/auth/invite`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "company_id": 1  // Optional - defaults to owner's default company
}
```

**Validation Rules:**
- `email`: required, string, email, max:255
- `company_id`: optional, integer, exists:companies,id

**Success Response (201):**
```json
{
  "success": true,
  "message": "Invitation sent successfully",
  "data": {
    "invitation": {
      "id": 1,
      "email": "user@example.com",
      "company_id": 1,
      "invited_user_id": null,  // null if user not registered, user_id if registered
      "is_user_registered": false,
      "expires_at": "2024-01-08T12:00:00.000000Z"
    }
  }
}
```

**Example Response (User Already Registered):**
```json
{
  "success": true,
  "message": "Invitation sent successfully",
  "data": {
    "invitation": {
      "id": 1,
      "email": "user@example.com",
      "company_id": 1,
      "invited_user_id": 5,  // User ID if already registered
      "is_user_registered": true,
      "expires_at": "2024-01-08T12:00:00.000000Z"
    }
  }
}
```

**Error Response (400) - Business Logic Error:**
```json
{
  "success": false,
  "message": "User already has access to this company."
}
```

**Error Response (400) - No Company Found:**
```json
{
  "success": false,
  "message": "No company found. Please provide company_id or set a default company."
}
```

**Error Response (400) - Permission Denied:**
```json
{
  "success": false,
  "message": "You do not have permission to invite users to this company."
}
```

**Error Response (400) - Duplicate Invitation:**
```json
{
  "success": false,
  "message": "An active invitation already exists for this email and company."
}
```

**Error Response (422) - Validation Error:**
```json
{
  "success": false,
  "message": "Validation errors",
  "errors": {
    "email": ["The email must be a valid email address."],
    "company_id": ["The selected company does not exist."]
  }
}
```

**Error Response (401) - Unauthenticated:**
```json
{
  "success": false,
  "message": "Unauthenticated"
}
```

---

## Accept Invitation

Accept an invitation for an already registered user. This will link the user to the company from the invitation.

**Endpoint:** `POST /api/v1/auth/invite/accept`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "token": "abc123xyz..."
}
```

**Validation Rules:**
- `token`: required, string, exists:invitations,token

**Success Response (200):**
```json
{
  "success": true,
  "message": "Invitation accepted successfully",
  "data": {
    "user": {
      "id": 2,
      "name": "Jane Doe",
      "email": "user@example.com",
      "email_verified_at": null,
      "company": {
        "id": 1,
        "name": "My Company",
        "is_default": true,
        ...
      },
      "companies": [
        {
          "id": 1,
          "name": "My Company",
          "is_default": true,
          ...
        }
      ],
      "created_at": "2024-01-01T12:00:00.000000Z",
      "updated_at": "2024-01-01T12:00:00.000000Z"
    }
  }
}
```

**Error Response (400) - Invalid Token:**
```json
{
  "success": false,
  "message": "Invalid or expired invitation token, or email mismatch."
}
```

**Error Response (422) - Validation Error:**
```json
{
  "success": false,
  "message": "Validation errors",
  "errors": {
    "token": ["The invitation token is invalid."]
  }
}
```

**Error Response (401) - Unauthenticated:**
```json
{
  "success": false,
  "message": "Unauthenticated"
}
```

---

## Invitation Status

Invitations have the following statuses:

- **pending**: Invitation is active and waiting to be accepted
- **accepted**: Invitation has been accepted
- **expired**: Invitation has expired (7 days from creation)

---

## Invitation Token

- **Format**: 64 character random string
- **Expiration**: 7 days from creation
- **Usage**: 
  - For new users: Used during registration
  - For existing users: Used to accept invitation

---

## Business Rules

1. **Owner Permission**: Only the company owner can invite users to their company
2. **Default Company**: If `company_id` is not provided, owner's default company is used
3. **Duplicate Prevention**: 
   - User cannot be invited if they already have access to the company
   - Active invitation cannot be created if one already exists for the same email and company
4. **Email Verification**: 
   - During registration with invitation, email must match invitation email
   - During accept invitation, logged-in user's email must match invitation email
5. **User ID Tracking**: 
   - If user is already registered, `invited_user_id` is set during invitation creation
   - If user is not registered, `invited_user_id` is set during registration

---

## Related Documentation

- [Common Information](../common/README.md)
- [Authentication APIs](../auth/README.md)
