# 📦 Inventory Management System - API v1 Documentation

> Complete API documentation for version 1 of the Inventory Management System

## 🌐 Base URL

```
http://your-domain.com/api/v1
```

## 🔐 Authentication

All protected endpoints require authentication using Bearer token in the `Authorization` header:

```http
Authorization: Bearer {access_token}
```

**Getting a Token:**
1. Register a new user via `/auth/register` (no token returned)
2. Login via `/auth/login` to get access token
3. Extract `token` from the login response
4. Include it in all subsequent requests

## 📦 API Modules

### 🔐 Authentication APIs
Complete user authentication and account management.

- **Register** - Create new user account
- **Login** - Authenticate and get access token
- **Logout** - Revoke all access tokens
- **Delete Account** - Soft delete user account

📖 [View Authentication Documentation](./auth/README.md)

### 👥 Invitation APIs
Team invitation and company access management.

- **Invite User** - Send invitation to join company
- **Accept Invitation** - Accept invitation and join company

📖 [View Invitation Documentation](./invitations/README.md)

### ℹ️ Common Information
Shared information across all APIs.

- Error handling and response formats
- Company system architecture
- Invitation flow
- Data models

📖 [View Common Documentation](./common/README.md)

## 🚀 Quick Start Guide

### 1. Register a New User

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

**Response:**
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": { ... }
  }
}
```

**Note:** After registration, login via `/auth/login` to get an access token.

### 2. Login

```json
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "string (email)",
  "password": "string"
}
```

### 3. Use Token in Protected Endpoints

```bash
POST /api/v1/auth/invite
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
  "email": "team@example.com",
  "company_id": 1
}
```

## 📋 Endpoint Summary

| Endpoint | Method | Auth Required | Description |
|----------|--------|---------------|-------------|
| `/auth/register` | `POST` | ❌ | Register new user |
| `/auth/login` | `POST` | ❌ | Login user |
| `/auth/logout` | `POST` | ✅ | Logout user |
| `/auth/account` | `DELETE` | ✅ | Delete account |
| `/auth/invite` | `POST` | ✅ | Invite user to company |
| `/auth/invite/accept` | `POST` | ✅ | Accept invitation |

## 🔗 Quick Links

- [🔐 Register User](./auth/README.md#register)
- [🔑 Login](./auth/README.md#login)
- [🚪 Logout](./auth/README.md#logout)
- [🗑️ Delete Account](./auth/README.md#delete-account)
- [📧 Invite User](./invitations/README.md#invite-user)
- [✅ Accept Invitation](./invitations/README.md#accept-invitation)

## 📝 Response Format

All API responses follow a consistent format:

**Success Response:**
```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... }
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "Error message",
  "errors": { ... }
}
```

## ⚠️ Error Codes

| Code | Description |
|------|-------------|
| `200` | Success |
| `201` | Created |
| `400` | Bad Request |
| `401` | Unauthenticated |
| `403` | Forbidden |
| `404` | Not Found |
| `422` | Validation Error |
| `500` | Server Error |

## 📚 Additional Resources

- [Error Handling Guide](./common/README.md#error-handling)
- [Company System Overview](./common/README.md#company-system)
- [Invitation Flow](./common/README.md#invitation-flow)
