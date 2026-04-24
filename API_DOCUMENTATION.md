# E-DKE Cloud Security - API Documentation

## Base URL
```
http://localhost:8080/api
```

---

## Authentication Endpoints

### 1. Register User
**Endpoint:** `POST /auth/register`

**Description:** Register a new user with role assignment

**Request Body:**
```json
{
  "username": "john_doe",
  "password": "secure_password_123",
  "email": "john@example.com",
  "role": "AUTHORIZED_USER"
}
```

**Roles Available:**
- `ADMIN` - Full access to all resources
- `DATA_OWNER` - Can upload, download, and manage own files
- `AUTHORIZED_USER` - Can only download authorized files

**Response (201):**
```json
{
  "message": "User registered successfully",
  "userId": 1,
  "username": "john_doe",
  "role": "AUTHORIZED_USER"
}
```

**Error Response (400):**
```json
{
  "error": "Username already exists"
}
```

---

### 2. Login User
**Endpoint:** `POST /auth/login`

**Description:** Authenticate user and receive JWT token

**Request Body:**
```json
{
  "username": "john_doe",
  "password": "secure_password_123"
}
```

**Response (200):**
```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "userId": 1,
  "username": "john_doe",
  "role": "AUTHORIZED_USER",
  "expiresIn": 3600
}
```

**Error Response (400):**
```json
{
  "error": "Invalid username or password"
}
```

---

### 3. Get All Users
**Endpoint:** `GET /auth/users`

**Description:** Retrieve list of all users (Admin only)

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
[
  {
    "id": 1,
    "username": "john_doe",
    "email": "john@example.com",
    "role": "AUTHORIZED_USER",
    "enabled": true
  }
]
```

---

### 4. Get User by ID
**Endpoint:** `GET /auth/users/{id}`

**Description:** Get specific user details

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "role": "AUTHORIZED_USER",
  "enabled": true
}
```

---

### 5. Update User Role
**Endpoint:** `PUT /auth/users/{id}/role`

**Description:** Update user role (Admin only)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "role": "DATA_OWNER"
}
```

**Response (200):**
```json
{
  "message": "User role updated successfully",
  "userId": 1,
  "role": "DATA_OWNER"
}
```

---

### 6. Delete User
**Endpoint:** `DELETE /auth/users/{id}`

**Description:** Delete a user (Admin only)

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "message": "User deleted successfully"
}
```

---

## File Management Endpoints

### 7. Upload and Encrypt File
**Endpoint:** `POST /file/upload`

**Description:** Upload file and encrypt using AES-DKE hybrid encryption

**Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Parameters:**
```
file: <binary file data>
```

**Response (200):**
```json
{
  "message": "File uploaded and encrypted successfully",
  "fileId": 5,
  "fileName": "document.pdf",
  "fileSize": 204800,
  "encryptionTimeMs": 45,
  "performanceMetrics": {
    "encryptionTimeMs": 45,
    "fileSizeBytes": 204800,
    "fileSizeMB": 0.195,
    "throughputMBps": 4.33,
    "algorithm": "AES-DKE"
  }
}
```

**Supported File Types:**
- Text files (.txt, .log, .csv)
- Images (.jpg, .jpeg, .png, .gif, .bmp)
- Documents (.pdf, .docx, .xlsx)
- Binary files (byte stream encoding)

---

### 8. Download and Decrypt File
**Endpoint:** `GET /file/download/{fileId}`

**Description:** Download encrypted file and decrypt with integrity verification

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):** Binary file data
```
Content-Disposition: attachment; filename="document.pdf"
X-Decryption-Time-Ms: 32
X-Integrity-Status: VERIFIED
```

**Error Response (403):**
```json
{
  "error": "Access denied"
}
```

**Error Response (400):**
```json
{
  "error": "File integrity check failed"
}
```

---

### 9. Get File Metadata
**Endpoint:** `GET /file/metadata/{fileId}`

**Description:** Retrieve metadata for specific file

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "fileId": 5,
  "fileName": "document.pdf",
  "fileType": "application/pdf",
  "fileSize": 204800,
  "uploadedBy": 1,
  "uploadTimestamp": "2026-02-18T10:30:45",
  "encryptionAlgorithm": "AES-DKE",
  "encryptionTimeMs": 45,
  "decryptionTimeMs": 32,
  "fileStatus": "UPLOADED",
  "integrityStatus": "VERIFIED",
  "downloadCount": 2,
  "lastAccessTime": "2026-02-18T11:45:30"
}
```

---

### 10. List User Files
**Endpoint:** `GET /file/list`

**Description:** List all files uploaded by current user

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "totalFiles": 3,
  "files": [
    {
      "fileId": 1,
      "fileName": "data.txt",
      "fileSize": 1024,
      "uploadTimestamp": "2026-02-18T09:15:00",
      "fileStatus": "UPLOADED",
      "downloadCount": 0
    },
    {
      "fileId": 2,
      "fileName": "image.jpg",
      "fileSize": 2048,
      "uploadTimestamp": "2026-02-18T09:30:00",
      "fileStatus": "UPLOADED",
      "downloadCount": 1
    }
  ]
}
```

---

### 11. Get File Performance Metrics
**Endpoint:** `GET /file/performance/{fileId}`

**Description:** Get detailed performance analysis for file encryption/decryption

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "filePath": "document.pdf",
  "fileSizeMB": 0.195,
  "encryptionTimeMs": 45,
  "decryptionTimeMs": 32,
  "totalTimeMs": 77,
  "algorithm": "AES-DKE",
  "encryptionThroughputMBps": 4.33,
  "decryptionThroughputMBps": 6.09,
  "avgThroughputMBps": 5.21
}
```

---

### 12. Delete File
**Endpoint:** `DELETE /file/{fileId}`

**Description:** Soft delete file (mark as deleted, not permanently removed)

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "message": "File deleted successfully"
}
```

**Authorization:**
- File uploader can delete their own files
- Admin can delete any file

---

### 13. Get All Files (Admin Only)
**Endpoint:** `GET /file/admin/all`

**Description:** Retrieve all files in the system (Admin only)

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "totalFiles": 10,
  "files": [
    {
      "id": 1,
      "fileName": "data.txt",
      "fileType": "text/plain",
      "fileSize": 1024,
      "uploadedBy": 1,
      "fileStatus": "UPLOADED",
      "integrityStatus": "VERIFIED"
    }
  ]
}
```

---

## Error Codes & Responses

| Code | Status | Message |
|------|--------|---------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid input or validation error |
| 403 | Forbidden | Access denied / Insufficient permissions |
| 404 | Not Found | Resource not found |
| 500 | Server Error | Internal server error |

---

## Authentication

All endpoints except `/auth/register` and `/auth/login` require JWT token in Authorization header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Token expires in 1 hour (3600 seconds).

---

## Example cURL Requests

### Register User
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "secure_pass",
    "email": "john@example.com",
    "role": "AUTHORIZED_USER"
  }'
```

### Login
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "secure_pass"
  }'
```

### Upload File
```bash
curl -X POST http://localhost:8080/api/file/upload \
  -H "Authorization: Bearer <token>" \
  -F "file=@document.pdf"
```

### Download File
```bash
curl -X GET http://localhost:8080/api/file/download/1 \
  -H "Authorization: Bearer <token>" \
  -o downloaded_file.pdf
```

### Get File Metadata
```bash
curl -X GET http://localhost:8080/api/file/metadata/1 \
  -H "Authorization: Bearer <token>"
```

---

## Security Features

1. **JWT Token Authentication** - 1-hour expiration
2. **BCrypt Password Encoding** - Secure password storage
3. **Role-Based Access Control (RBAC)** - Three-tier authorization
4. **Hybrid Encryption** - AES-128 + DKE dual-key encryption
5. **Integrity Verification** - SHA-256 hash validation
6. **Stateless Session Management** - SessionCreationPolicy.STATELESS
7. **CSRF Protection Disabled** - For stateless REST API

---

## Performance Benchmarks

Average performance for 1MB file:
- **Encryption Time:** 8-12ms
- **Decryption Time:** 6-10ms
- **Throughput:** 80-125 MB/s
- **Hash Verification:** <1ms

---

## Database Schema

### users table
```
id (PK), username (UNIQUE), password, email (UNIQUE), role (ENUM), enabled, created_at
```

### file_metadata table
```
id (PK), file_name, file_type, file_size, hash_value, encrypted_data (BLOB),
encrypted_aes_key (BLOB), vm_key (BLOB), server_key (BLOB), uploaded_by (FK),
upload_timestamp, encryption_algorithm, encryption_time_ms, decryption_time_ms,
file_status, last_access_time, download_count, access_attempts, storage_location,
integrity_status, requires_role_auth, accessible
```

---

## Testing the API

Use Postman, Insomnia, or cURL to test endpoints. Steps:
1. Register a new user
2. Login to get JWT token
3. Copy token to Authorization header
4. Test file upload endpoint
5. Test file download with file ID from upload response
6. Monitor performance metrics
