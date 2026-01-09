# 📦 KhataSync - API Documentation

> Complete API documentation for the KhataSync (IMS)

## 🚀 Quick Start

**Base URL:** `http://your-domain.com/api/v1`

**Authentication:** Use Bearer token in `Authorization` header

```http
Authorization: Bearer {your_access_token}
```

## 📚 Available Versions

| Version | Status | Documentation |
|---------|--------|---------------|
| **v1** | ✅ Stable | [View Documentation](./v1/README.md) |

## 📖 Documentation Structure

```
docs/
├── README.md                    # This file - Overview
└── v1/
    ├── README.md                # Main v1 API documentation index
    ├── auth/
    │   └── README.md           # Authentication APIs
    ├── invitations/
    │   └── README.md          # Invitation APIs
    └── common/
        └── README.md           # Common information, error handling, data models
```

## 🔐 Authentication APIs

Complete authentication and user management APIs.

| Endpoint | Method | Description |
|----------|--------|-------------|
| [Register](./v1/auth/README.md#register) | `POST` | Register a new user account |
| [Login](./v1/auth/README.md#login) | `POST` | Authenticate user and get access token |
| [Logout](./v1/auth/README.md#logout) | `POST` | Logout user and revoke tokens |
| [Delete Account](./v1/auth/README.md#delete-account) | `DELETE` | Soft delete user account |

📖 [View Full Authentication Documentation](./v1/auth/README.md)

## 👥 Invitation APIs

Team invitation and company access management.

| Endpoint | Method | Description |
|----------|--------|-------------|
| [Invite User](./v1/invitations/README.md#invite-user) | `POST` | Invite user to company |
| [Accept Invitation](./v1/invitations/README.md#accept-invitation) | `POST` | Accept invitation and join company |

📖 [View Full Invitation Documentation](./v1/invitations/README.md)

## 🏢 Multi-Company System

The system supports a multi-company architecture:

- **Owners**: Can create and manage multiple companies
- **Team Members**: Can have access to multiple companies
- **Default Company**: Each user has a default company
- **Company Access**: Controlled via invitations

## 🚦 Getting Started

### Step 1: Register or Login

**Register a new user:**
```json
POST /api/v1/auth/register
Content-Type: application/json

{
  "name": "string",
  "email": "string (email)",
  "password": "string",
  "password_confirmation": "string",
  "company_name": "string"
}
```

**Or login with existing credentials:**
```json
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "string (email)",
  "password": "string"
}
```

### Step 2: Get Access Token

**Note:** Register endpoint does not return a token. You must login after registration to get an access token.

Login response will include an access token:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": { ... },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }
}
```

### Step 3: Use Token in Requests

Include token in Authorization header:
```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

### Step 4: Invite Team Members

```bash
POST /api/v1/auth/invite
Authorization: Bearer {token}
Content-Type: application/json

{
  "email": "team@example.com",
  "company_id": 1
}
```

## 📋 Common Information

- [Error Handling](./v1/common/README.md#error-handling)
- [Response Format](./v1/common/README.md#response-format)
- [Company System](./v1/common/README.md#company-system)
- [Invitation Flow](./v1/common/README.md#invitation-flow)

📖 [View Common Documentation](./v1/common/README.md)

## 🔗 Documentation Links

- [📘 v1 Main Documentation](./v1/README.md)
- [🔐 Authentication APIs](./v1/auth/README.md)
- [👥 Invitation APIs](./v1/invitations/README.md)
- [ℹ️ Common Information](./v1/common/README.md)

## 📝 Notes

- All timestamps are in ISO 8601 format (UTC)
- All endpoints return JSON responses
- Use HTTPS in production
- Tokens expire after configured time (default: 1 year)

## 🤝 Support

For issues or questions, please refer to the project repository or contact the development team.
