# Common Information

## Response Format

All successful API responses follow this format:

```json
{
  "success": true,
  "message": "Success message",
  "data": {
    // Response data
  }
}
```

---

## Error Handling

All API endpoints follow a consistent error response format:

### Validation Errors (422)
```json
{
  "success": false,
  "message": "Validation errors",
  "errors": {
    "field_name": ["Error message 1", "Error message 2"]
  }
}
```

### Authentication Errors (401)
```json
{
  "success": false,
  "message": "Unauthenticated"
}
```

### Business Logic Errors (400)
```json
{
  "success": false,
  "message": "Error message describing what went wrong"
}
```

### Server Errors (500)
```json
{
  "success": false,
  "message": "Internal Server Error"  // In production
}
```

In development mode, server errors include additional debugging information:
```json
{
  "success": false,
  "message": "Error message",
  "file": "/path/to/file.php",
  "line": 123,
  "trace": "..."
}
```

---

## Company System

### Default Company

Each user has a default company which is determined by the `is_default` flag in the `user_companies` pivot table. The default company is automatically set when:
- User creates a new company (becomes default)
- User accepts an invitation and has no other companies (becomes default)

### Company Access

Users can have access to multiple companies through the `user_companies` pivot table. The `is_default` flag indicates which company is the user's default company.

### Owner vs Team Member

- **Owner**: User who created the company (`owner_id` in companies table)
- **Team Member**: User who has been invited and given access to a company

---

## Data Models

### User Object
```json
{
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
      "is_default": true,
      ...
    }
  ],
  "created_at": "2024-01-01T12:00:00.000000Z",
  "updated_at": "2024-01-01T12:00:00.000000Z"
}
```

### Company Object
```json
{
  "id": 1,
  "name": "My Company",
  "email": "company@example.com",
  "phone": "+1234567890",
  "address": "123 Main St",
  "logo": "https://example.com/logo.png",
  "status": "active",
  "is_default": true,
  "created_at": "2024-01-01T12:00:00.000000Z",
  "updated_at": "2024-01-01T12:00:00.000000Z"
}
```

### Invitation Object
```json
{
  "id": 1,
  "email": "user@example.com",
  "company_id": 1,
  "invited_user_id": null,
  "is_user_registered": false,
  "expires_at": "2024-01-08T12:00:00.000000Z"
}
```

---

## Token Expiration

- **Access Tokens**: Valid until revoked (via logout or delete account)
- **Invitation Tokens**: Valid for 7 days from creation

---

## Notes

- All timestamps are in ISO 8601 format (UTC)
- All IDs are integers
- Soft deleted users cannot login
- Soft deleted companies are excluded from queries
- Email addresses must be unique across the system
- Company names are not required to be unique
