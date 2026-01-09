# Authentication APIs

## Register

Register a new user account. The registration supports three different scenarios based on the provided fields.

**Endpoint:** `POST /api/v1/auth/register`

**Authentication:** Not required

**Validation Rules:**
- `name`: required, string, max:255
- `email`: required, string, email, max:255, unique:users
- `password`: required, string, min:8, confirmed
- `password_confirmation`: required, string, must match password
- `company_name`: optional, string, max:255 (for owner registration)
- `invitation_token`: optional, string, exists:invitations,token (for invited user registration)

**Note:** `company_name` and `invitation_token` are mutually exclusive. Provide only one or neither.

**Important:** After successful registration, you need to login via `/api/v1/auth/login` to get an access token. The register endpoint does not return a token.

---

## Login

Authenticate a user and get access token.

**Endpoint:** `POST /api/v1/auth/login`

**Authentication:** Not required

**Request Body:**
```json
{
  "email": "string (email)",
  "password": "string"
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

### Scenario 1: Owner Registration (With Company)

Register as a company owner by providing `company_name`. This creates a new company and makes the user the owner.

**Request Payload:**
```json
{
  "name": "string",
  "email": "string (email)",
  "password": "string",
  "password_confirmation": "string",
  "company_name": "string"
}
```

**Complete Example (cURL):**
```bash
curl -X POST http://your-domain.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "SecurePass123!",
    "password_confirmation": "SecurePass123!",
    "company_name": "My Company"
  }'
```

**Response (201):**
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

**Result:** User is created as owner, company is created, and user is linked to company with `is_default = true`.

---

### Scenario 2: Invited User Registration (New User)

Register with invitation token when the user is not yet registered. The email must match the invitation email.

**Request Payload:**
```json
{
  "name": "string",
  "email": "string (email)",
  "password": "string",
  "password_confirmation": "string",
  "invitation_token": "string (token)"
}
```

**Complete Example (cURL):**
```bash
curl -X POST http://your-domain.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Jane Doe",
    "email": "jane@example.com",
    "password": "SecurePass123!",
    "password_confirmation": "SecurePass123!",
    "invitation_token": "abc123xyz789def456ghi012jkl345mno678pqr901stu234vwx567yz"
  }'
```

**Response (201):**
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": 2,
      "name": "Jane Doe",
      "email": "jane@example.com",
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

**Result:** User is created, linked to the company from invitation, and invitation is marked as accepted.

**Important Notes:**
- The `email` in the request must match the email in the invitation
- The `invitation_token` must be valid and not expired
- Do not provide `company_name` when using `invitation_token`

---

### Scenario 3: Regular Registration (No Company)

Register without company or invitation. User will have no company access initially but can be invited later.

**Request Payload:**
```json
{
  "name": "string",
  "email": "string (email)",
  "password": "string",
  "password_confirmation": "string"
}
```

**Complete Example (cURL):**
```bash
curl -X POST http://your-domain.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Bob Smith",
    "email": "bob@example.com",
    "password": "SecurePass123!",
    "password_confirmation": "SecurePass123!"
  }'
```

**Response (201):**
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": 3,
      "name": "Bob Smith",
      "email": "bob@example.com",
      "email_verified_at": null,
      "company": null,
      "companies": [],
      "created_at": "2024-01-01T12:00:00.000000Z",
      "updated_at": "2024-01-01T12:00:00.000000Z"
    }
  }
}
```

**Result:** User is created but has no company access. They can be invited to join a company later.

---

### All Scenarios - Common Error Responses

**Error Response (422) - Validation Error:**
```json
{
  "success": false,
  "message": "Validation errors",
  "errors": {
    "name": ["The name field is required."],
    "email": ["The email has already been taken."],
    "password": ["The password must be at least 8 characters.", "The password confirmation does not match."],
    "company_name": ["The company name may not be greater than 255 characters."]
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

## Related Documentation

- [Common Information](../common/README.md)
- [Invitation APIs](../invitations/README.md)
