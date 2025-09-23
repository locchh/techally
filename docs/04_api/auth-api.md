---
id: SVC-AUTH-001
title: Auth Service API
category: api
tags: [api, authentication, authorization, jwt, oauth]
version: 1.0.0
status: approved
created: 2025-09-23
updated: 2025-09-23
author: API Team
references:
  - id: API-001
    type: specification
    link: ./api-reference.md
  - id: SEC-001
    type: security
    link: ../03_architecture/security.md
---

# Auth Service API

## 1. Service Overview

The Auth Service handles all authentication and authorization operations for the TechAlly platform.

### Base URL
```
Production: https://api.techally.com/auth
Staging: https://staging-api.techally.com/auth
```

### Authentication
Most endpoints require JWT authentication. Include the token in the Authorization header:
```
Authorization: Bearer <jwt_token>
```

## 2. API Endpoints

### 2.1 Authentication

#### Register User
```http
POST /register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "acceptTerms": true
}
```

**Response (201 Created):**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "customer",
    "createdAt": "2024-01-01T00:00:00Z"
  },
  "tokens": {
    "accessToken": "jwt_access_token",
    "refreshToken": "jwt_refresh_token",
    "expiresIn": 3600
  }
}
```

#### Login
```http
POST /login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Response (200 OK):**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "role": "customer"
  },
  "tokens": {
    "accessToken": "jwt_access_token",
    "refreshToken": "jwt_refresh_token",
    "expiresIn": 3600
  }
}
```

#### Logout
```http
POST /logout
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "message": "Logged out successfully"
}
```

#### Refresh Token
```http
POST /refresh
Content-Type: application/json

{
  "refreshToken": "jwt_refresh_token"
}
```

**Response (200 OK):**
```json
{
  "accessToken": "new_jwt_access_token",
  "expiresIn": 3600
}
```

### 2.2 Password Management

#### Forgot Password
```http
POST /forgot-password
Content-Type: application/json

{
  "email": "user@example.com"
}
```

**Response (200 OK):**
```json
{
  "message": "Password reset email sent"
}
```

#### Reset Password
```http
POST /reset-password
Content-Type: application/json

{
  "token": "reset_token",
  "password": "NewSecurePass123!"
}
```

**Response (200 OK):**
```json
{
  "message": "Password reset successfully"
}
```

#### Change Password
```http
PUT /change-password
Authorization: Bearer <token>
Content-Type: application/json

{
  "currentPassword": "OldPass123!",
  "newPassword": "NewPass123!"
}
```

### 2.3 Email Verification

#### Send Verification Email
```http
POST /verify-email/send
Authorization: Bearer <token>
```

#### Verify Email
```http
POST /verify-email
Content-Type: application/json

{
  "token": "verification_token"
}
```

### 2.4 OAuth Integration

#### OAuth Login
```http
GET /oauth/{provider}
```

Providers: `google`, `facebook`, `github`

#### OAuth Callback
```http
GET /oauth/{provider}/callback?code={code}&state={state}
```

### 2.5 Two-Factor Authentication

#### Enable 2FA
```http
POST /2fa/enable
Authorization: Bearer <token>
```

**Response:**
```json
{
  "qrCode": "data:image/png;base64,...",
  "secret": "ABCDEFGHIJKLMNOP",
  "backupCodes": ["12345678", "87654321", ...]
}
```

#### Verify 2FA
```http
POST /2fa/verify
Authorization: Bearer <token>
Content-Type: application/json

{
  "code": "123456"
}
```

#### Disable 2FA
```http
POST /2fa/disable
Authorization: Bearer <token>
Content-Type: application/json

{
  "password": "UserPassword123!",
  "code": "123456"
}
```

## 3. Token Management

### 3.1 JWT Structure

```javascript
// Access Token Payload
{
  "sub": "user_id",
  "email": "user@example.com",
  "role": "customer",
  "permissions": ["read:products", "write:orders"],
  "iat": 1234567890,
  "exp": 1234571490,
  "iss": "techally.com",
  "aud": "techally-api"
}

// Refresh Token Payload
{
  "sub": "user_id",
  "tokenFamily": "family_id",
  "iat": 1234567890,
  "exp": 1234657890
}
```

### 3.2 Token Validation

```typescript
// Token validation middleware
export async function validateToken(req: Request): Promise<User> {
  const token = extractToken(req);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET, {
      issuer: 'techally.com',
      audience: 'techally-api'
    });
    
    // Check if token is blacklisted
    if (await isTokenBlacklisted(token)) {
      throw new UnauthorizedError('Token has been revoked');
    }
    
    // Check user still exists and is active
    const user = await getUserById(decoded.sub);
    if (!user || !user.isActive) {
      throw new UnauthorizedError('User not found or inactive');
    }
    
    return user;
  } catch (error) {
    throw new UnauthorizedError('Invalid token');
  }
}
```

## 4. Session Management

### 4.1 Get Active Sessions
```http
GET /sessions
Authorization: Bearer <token>
```

**Response:**
```json
{
  "sessions": [
    {
      "id": "session_id",
      "device": "Chrome on Windows",
      "ipAddress": "192.168.1.1",
      "location": "New York, US",
      "lastActive": "2024-01-01T00:00:00Z",
      "current": true
    }
  ]
}
```

### 4.2 Revoke Session
```http
DELETE /sessions/{sessionId}
Authorization: Bearer <token>
```

## 5. Role & Permissions

### 5.1 Get User Permissions
```http
GET /permissions
Authorization: Bearer <token>
```

**Response:**
```json
{
  "role": "customer",
  "permissions": [
    "read:products",
    "write:orders",
    "read:profile",
    "write:profile"
  ]
}
```

### 5.2 Check Permission
```http
POST /check-permission
Authorization: Bearer <token>
Content-Type: application/json

{
  "resource": "orders",
  "action": "write"
}
```

## 6. Error Responses

### Error Format
```json
{
  "error": {
    "code": "AUTH_ERROR",
    "message": "Authentication failed",
    "details": {
      "field": "email",
      "reason": "Invalid credentials"
    }
  }
}
```

### Common Error Codes

| Code | Status | Description |
|------|--------|-------------|
| `INVALID_CREDENTIALS` | 401 | Email or password incorrect |
| `ACCOUNT_LOCKED` | 403 | Too many failed attempts |
| `EMAIL_NOT_VERIFIED` | 403 | Email verification required |
| `TOKEN_EXPIRED` | 401 | JWT token has expired |
| `TOKEN_INVALID` | 401 | JWT token is invalid |
| `INSUFFICIENT_PERMISSIONS` | 403 | User lacks required permissions |
| `USER_EXISTS` | 409 | Email already registered |
| `WEAK_PASSWORD` | 400 | Password doesn't meet requirements |

## 7. Rate Limiting

| Endpoint | Limit | Window |
|----------|-------|--------|
| `/login` | 5 requests | 15 minutes |
| `/register` | 3 requests | 1 hour |
| `/forgot-password` | 3 requests | 1 hour |
| `/refresh` | 10 requests | 1 hour |
| Other endpoints | 100 requests | 1 minute |

## 8. Security Implementation

### 8.1 Password Requirements
```javascript
const passwordPolicy = {
  minLength: 8,
  maxLength: 128,
  requireUppercase: true,
  requireLowercase: true,
  requireNumbers: true,
  requireSpecialChars: true,
  preventCommonPasswords: true,
  preventUserInfo: true
};
```

### 8.2 Security Headers
```javascript
{
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'Content-Security-Policy': "default-src 'self'"
}
```

## 9. SDK Examples

### 9.1 JavaScript/TypeScript
```typescript
import { AuthClient } from '@techally/auth-sdk';

const auth = new AuthClient({
  baseURL: 'https://api.techally.com/auth'
});

// Login
const { user, tokens } = await auth.login({
  email: 'user@example.com',
  password: 'password'
});

// Use authenticated client
auth.setToken(tokens.accessToken);

// Get current user
const currentUser = await auth.getCurrentUser();

// Logout
await auth.logout();
```

### 9.2 Python
```python
from techally_sdk import AuthClient

auth = AuthClient(base_url='https://api.techally.com/auth')

# Login
response = auth.login(
    email='user@example.com',
    password='password'
)

# Set token for authenticated requests
auth.set_token(response['tokens']['accessToken'])

# Get current user
user = auth.get_current_user()
```

## 10. Webhooks

The Auth Service can send webhooks for the following events:

| Event | Description |
|-------|-------------|
| `user.registered` | New user registration |
| `user.verified` | Email verified |
| `user.login` | Successful login |
| `user.password_changed` | Password changed |
| `user.locked` | Account locked |
| `user.2fa_enabled` | 2FA enabled |

## 11. References

- [API Reference](./api-reference.md) - `API-001`
- [Security Architecture](../03_architecture/security.md) - `SEC-001`
- [Error Handling](./error-handling.md) - `ERR-001`

---
*This Auth Service API documentation is maintained by the API Team.*
