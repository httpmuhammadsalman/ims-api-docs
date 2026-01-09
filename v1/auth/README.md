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
  "success": "boolean",
  "message": "string",
  "data": {
    "user": {
      "id": "integer",
      "name": "string",
      "email": "string (email)",
      "email_verified_at": "null|string (datetime)",
      "company": {
        "id": "integer",
        "name": "string",
        "is_default": "boolean",
        ...
      },
      "companies": [
        {
          "id": "integer",
          "name": "string",
          "is_default": "boolean",
          ...
        }
      ],
      "created_at": "string (datetime)",
      "updated_at": "string (datetime)"
    },
    "token": "string (token)"
  }
}
```

**Error Response (401) - Invalid Credentials:**
```json
{
  "success": "boolean",
  "message": "string"
}
```

**Error Response (422) - Validation Error:**
```json
{
  "success": "boolean",
  "message": "string",
  "errors": {
    "email": ["string"],
    "password": ["string"]
  }
}
```

---

## Forgot Password

Send a password reset link to the user's email address.

**Endpoint:** `POST /api/v1/auth/forgot-password`

**Authentication:** Not required

**Request Body:**
```json
{
  "email": "string (email)"
}
```

**Validation Rules:**
- `email`: required, string, email, must exist in users table

**Success Response (200):**
```json
{
  "success": "boolean",
  "message": "string"
}
```

**Error Response (400) - User Not Found:**
```json
{
  "success": "boolean",
  "message": "string"
}
```

**Error Response (422) - Validation Error:**
```json
{
  "success": "boolean",
  "message": "string",
  "errors": {
    "email": ["string"]
  }
}
```

**Note:** A password reset email will be sent to the provided email address if it exists in the system. The reset link will expire in 60 minutes.

---

## Reset Password

Reset user password using the token received via email.

**Endpoint:** `POST /api/v1/auth/reset-password`

**Authentication:** Not required

**Request Body:**
```json
{
  "email": "string (email)",
  "token": "string",
  "password": "string",
  "password_confirmation": "string"
}
```

**Validation Rules:**
- `email`: required, string, email, must exist in users table
- `token`: required, string, must be valid reset token
- `password`: required, string, min:8, must be confirmed
- `password_confirmation`: required, string, must match password

**Success Response (200):**
```json
{
  "success": "boolean",
  "message": "string"
}
```

**Error Response (400) - Invalid Token:**
```json
{
  "success": "boolean",
  "message": "string"
}
```

**Error Response (400) - Expired Token:**
```json
{
  "success": "boolean",
  "message": "string"
}
```

**Error Response (422) - Validation Error:**
```json
{
  "success": "boolean",
  "message": "string",
  "errors": {
    "email": ["string"],
    "token": ["string"],
    "password": ["string"]
  }
}
```

**Complete Example (cURL):**
```bash
curl -X POST http://your-domain.com/api/v1/auth/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "token": "abc123xyz789def456ghi012jkl345mno678pqr901stu234vwx567yz",
    "password": "NewSecurePass123!",
    "password_confirmation": "NewSecurePass123!"
  }'
```

**Note:** 
- The token is received via email after requesting password reset
- Token expires after 60 minutes
- Token is single-use and will be deleted after successful password reset

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
  "success": "boolean",
  "message": "string"
}
```

**Error Response (401) - Unauthenticated:**
```json
{
  "success": "boolean",
  "message": "string"
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
  "success": "boolean",
  "message": "string"
}
```

**Error Response (401) - Unauthenticated:**
```json
{
  "success": "boolean",
  "message": "string"
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
  "success": "boolean",
  "message": "string",
  "data": {
    "user": {
      "id": "integer",
      "name": "string",
      "email": "string (email)",
      "email_verified_at": "null|string (datetime)",
      "company": {
        "id": "integer",
        "name": "string",
        "email": "null|string (email)",
        "phone": "null|string",
        "address": "null|string",
        "logo": "null|string (url)",
        "status": "string",
        "is_default": "boolean",
        "created_at": "string (datetime)",
        "updated_at": "string (datetime)"
      },
      "companies": [
        {
          "id": "integer",
          "name": "string",
          "email": "null|string (email)",
          "phone": "null|string",
          "address": "null|string",
          "logo": "null|string (url)",
          "status": "string",
          "is_default": "boolean",
          "created_at": "string (datetime)",
          "updated_at": "string (datetime)"
        }
      ],
      "created_at": "string (datetime)",
      "updated_at": "string (datetime)"
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
  "success": "boolean",
  "message": "string",
  "data": {
    "user": {
      "id": "integer",
      "name": "string",
      "email": "string (email)",
      "email_verified_at": "null|string (datetime)",
      "company": {
        "id": "integer",
        "name": "string",
        "email": "null|string (email)",
        "phone": "null|string",
        "address": "null|string",
        "logo": "null|string (url)",
        "status": "string",
        "is_default": "boolean",
        "created_at": "string (datetime)",
        "updated_at": "string (datetime)"
      },
      "companies": [
        {
          "id": "integer",
          "name": "string",
          "email": "null|string (email)",
          "phone": "null|string",
          "address": "null|string",
          "logo": "null|string (url)",
          "status": "string",
          "is_default": "boolean",
          "created_at": "string (datetime)",
          "updated_at": "string (datetime)"
        }
      ],
      "created_at": "string (datetime)",
      "updated_at": "string (datetime)"
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
  "success": "boolean",
  "message": "string",
  "data": {
    "user": {
      "id": "integer",
      "name": "string",
      "email": "string (email)",
      "email_verified_at": "null|string (datetime)",
      "company": "null|object",
      "companies": "array",
      "created_at": "string (datetime)",
      "updated_at": "string (datetime)"
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
  "success": "boolean",
  "message": "string",
  "errors": {
    "name": ["string"],
    "email": ["string"],
    "password": ["string"],
    "company_name": ["string"]
  }
}
```

**Error Response (400) - Invalid Invitation Token:**
```json
{
  "success": "boolean",
  "message": "string"
}
```

**Error Response (400) - Email Mismatch:**
```json
{
  "success": "boolean",
  "message": "string"
}
```

---

## Related Documentation

- [Common Information](../common/README.md)
- [Invitation APIs](../invitations/README.md)
