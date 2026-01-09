# Authentication APIs

## Register

Register a new user account. If `company_name` is provided, the user will be created as an owner with a new company. If `invitation_token` is provided, the user will be linked to the invited company.

**Endpoint:** `POST /api/v1/auth/register`

**Authentication:** Not required

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123",
  "password_confirmation": "password123",
  "company_name": "My Company",  // Optional - for owner registration
  "invitation_token": "abc123..."  // Optional - for invited user registration
}
```

**Validation Rules:**
- `name`: required, string, max:255
- `email`: required, string, email, max:255, unique:users
- `password`: required, string, min:8, confirmed
- `company_name`: optional, string, max:255
- `invitation_token`: optional, string, exists:invitations,token

**Success Response (201):**
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "email_verified_at": null,
      "company": {
        "id": 1,
        "name": "My Company",
        "email": null,
        "phone": null,
        "address": null,
        "logo": null,
        "status": "active",
        "is_default": true,
        "created_at": "2024-01-01T12:00:00.000000Z",
        "updated_at": "2024-01-01T12:00:00.000000Z"
      },
      "companies": [
        {
          "id": 1,
          "name": "My Company",
          "email": null,
          "phone": null,
          "address": null,
          "logo": null,
          "status": "active",
          "is_default": true,
          "created_at": "2024-01-01T12:00:00.000000Z",
          "updated_at": "2024-01-01T12:00:00.000000Z"
        }
      ],
      "created_at": "2024-01-01T12:00:00.000000Z",
      "updated_at": "2024-01-01T12:00:00.000000Z"
    }
  }
}
```

**Note:** After successful registration, you need to login via `/api/v1/auth/login` to get an access token.

**Error Response (422) - Validation Error:**
```json
{
  "success": false,
  "message": "Validation errors",
  "errors": {
    "email": ["The email has already been taken."],
    "password": ["The password must be at least 8 characters."]
  }
}
```

**Error Response (400) - Invalid Invitation Token:**
```json
{
  "success": false,
  "message": "Invalid or expired invitation token."
}
```

**Error Response (400) - Email Mismatch:**
```json
{
  "success": false,
  "message": "Email does not match the invitation."
}
```

---

## Login

Authenticate a user and get access token.

**Endpoint:** `POST /api/v1/auth/login`

**Authentication:** Not required

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

**Validation Rules:**
- `email`: required, string, email
- `password`: required, string

**Success Response (200):**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
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
    },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }
}
```

**Error Response (401) - Invalid Credentials:**
```json
{
  "success": false,
  "message": "Invalid credentials"
}
```

**Error Response (422) - Validation Error:**
```json
{
  "success": false,
  "message": "Validation errors",
  "errors": {
    "email": ["The email field is required."],
    "password": ["The password field is required."]
  }
}
```

---

## Logout

Logout the authenticated user and revoke all access tokens.

**Endpoint:** `POST /api/v1/auth/logout`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:** None

**Success Response (200):**
```json
{
  "success": true,
  "message": "Logout successful"
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

## Delete Account

Soft delete the authenticated user account. This will also soft delete all companies owned by the user and revoke all access tokens.

**Endpoint:** `DELETE /api/v1/auth/account`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:** None

**Success Response (200):**
```json
{
  "success": true,
  "message": "Account deleted successfully"
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

## Registration Scenarios

### Scenario 1: Owner Registration
Register as a company owner by providing `company_name`:

```json
POST /api/v1/auth/register
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123",
  "password_confirmation": "password123",
  "company_name": "My Company"
}
```

Result: User is created as owner, company is created, and user is linked to company with `is_default = true`.

### Scenario 2: Invited User Registration (New User)
Register with invitation token (user not yet registered):

```json
POST /api/v1/auth/register
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "password123",
  "password_confirmation": "password123",
  "invitation_token": "abc123xyz..."
}
```

Result: User is created, linked to the company from invitation, and invitation is marked as accepted.

### Scenario 3: Regular Registration (No Company)
Register without company or invitation:

```json
POST /api/v1/auth/register
{
  "name": "Bob Smith",
  "email": "bob@example.com",
  "password": "password123",
  "password_confirmation": "password123"
}
```

Result: User is created but has no company access. They can be invited later.

---

## Related Documentation

- [Common Information](../common/README.md)
- [Invitation APIs](../invitations/README.md)
