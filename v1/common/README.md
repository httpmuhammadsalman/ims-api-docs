# Common Information

## Response Format

All successful API responses follow this format:

```json
{
  "success": "boolean",
  "message": "string",
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
  "success": "boolean",
  "message": "string",
  "errors": {
    "field_name": ["string"]
  }
}
```

### Authentication Errors (401)
```json
{
  "success": "boolean",
  "message": "string"
}
```

### Business Logic Errors (400)
```json
{
  "success": "boolean",
  "message": "string"
}
```

### Server Errors (500)
```json
{
  "success": "boolean",
  "message": "string"
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
      "is_default": "boolean",
      ...
    }
  ],
  "created_at": "string (datetime)",
  "updated_at": "string (datetime)"
}
```

### Company Object
```json
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
```

### Invitation Object
```json
{
  "id": "integer",
  "email": "string (email)",
  "company_id": "integer",
  "invited_user_id": "null|integer",
  "is_user_registered": "boolean",
  "expires_at": "string (datetime)"
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
