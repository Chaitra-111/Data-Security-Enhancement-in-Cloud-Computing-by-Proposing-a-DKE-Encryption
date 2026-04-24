# Enhanced Data Key Encryption (EDKE) for Cloud Security 🔐

A robust cloud security application implementing the **Dual Key Encryption (DKE) Protocol** for secure file storage in cloud environments. This project combines AES-256 encryption with a proprietary Dual Key Encryption mechanism to provide enhanced data security for cloud computing environments.

## 🚀 Overview

EDKE (Enhanced Data Key Encryption) is an advanced encryption protocol designed for secure cloud data storage. It uses a hybrid encryption approach that combines:
- **AES-256** for fast file data encryption
- **Dual Key Encryption (DKE)** for securing the AES key using two independent keys (VM Key + Server Key)
- **SHA-256 hashing** for data integrity verification

This architecture ensures that even if one key is compromised, the data remains secure.

## ✨ Features

- **Dual Key Encryption Protocol**: Implements DKE using VM-generated and Server-generated keys
- **AES-256 Encryption**: Military-grade symmetric encryption for file data
- **Data Integrity Verification**: SHA-256 hash verification ensures data hasn't been tampered with
- **Role-Based Access Control**: Three-tier user roles (ADMIN, DATA_OWNER, AUTHORIZED_USER)
- **JWT Authentication**: Secure token-based authentication
- **Cloud Storage Integration**: Supports both local storage and Backblaze B2 cloud storage
- **Performance Analytics**: Built-in performance monitoring for encryption/decryption operations
- **RESTful API**: Complete REST API for file operations

## 🚀 Tech Stack

- **Backend**: Spring Boot 3.2.5, Java 21
- **Security**: Spring Security, JWT (jjwt)
- **Database**: MySQL, JPA/Hibernate
- **Cloud Storage**: Backblaze B2
- **Build Tool**: Apache Maven
- **Testing**: JUnit, Spring Security Test

## 🏗️ Architecture

### Dual Key Encryption (DKE) Protocol

The DKE protocol works as follows:

1. **Key Generation**:
   - Generate a random AES-256 session key
   - Generate VM (Virtual Machine) key
   - Generate Server key

2. **Encryption Process**:
   - Encrypt file data using AES-256
   - Encrypt the AES key using DKE: `EncryptedKey = (AESKey XOR VMKey) XOR ServerKey`

3. **Storage**:
   - Store encrypted file data
   - Store encrypted AES key
   - Store VM key and Server key (for decryption)
   - Store SHA-256 hash for integrity verification

4. **Decryption Process**:
   - Decrypt AES key: `AESKey = (EncryptedKey XOR ServerKey) XOR VMKey`
   - Decrypt file data using AES-256
   - Verify integrity using SHA-256 hash

```
┌─────────────┐     ┌─────────────┐     ┌──────────────────┐
│  File Data  │────▶│  AES-256    │────▶│  Encrypted Data  │
└─────────────┘     └─────────────┘     └──────────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  AES Key    │
                    └─────────────┘
                           │
           ┌───────────────┴───────────────┐
           ▼                               ▼
    ┌─────────────┐                ┌─────────────┐
    │   VM Key    │────XOR────────▶│  DKE Layer  │
    └─────────────┘                └─────────────┘
           │                               ▲
           │                        ┌─────────────┐
           └───────────XOR─────────│  Server Key │
                          └─────────────┘
                                        │
                                        ▼
                               ┌──────────────────┐
                               │ Encrypted AES Key│
                               └──────────────────┘
```

## 📋 Prerequisites

Before running this project, ensure you have:

- Java Development Kit (JDK) 21 or higher
- Apache Maven 3.8+
- MySQL Server 8.0+
- IDE (IntelliJ IDEA, Eclipse, or VS Code)

## 🛠️ Installation

1. **Clone the repository**
   
```
bash
   git clone <repository-url>
   cd cloudsecurity
   
```

2. **Configure MySQL Database**
   
   Create a MySQL database named `edke_cloud`:
   
```
sql
   CREATE DATABASE edke_cloud;
   
```

3. **Configure Application Properties**
   
   Edit `src/main/resources/application.properties`:
   
```
properties
   # Database Configuration
   spring.datasource.url=jdbc:mysql://localhost:3306/edke_cloud
   spring.datasource.username=your_username
   spring.datasource.password=your_password
   
   # JWT Configuration
   jwt.secret=your_jwt_secret_key
   
   # Storage Mode: local or backblaze
   storage.mode=local
   
```

4. **Build the Project**
   
```
bash
   mvn clean install
   
```

5. **Run the Application**
   
```
bash
   mvn spring-boot:run
   
```
   
   The application will be available at `http://localhost:8080`

## 📡 API Endpoints

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register a new user |
| POST | `/api/auth/login` | Login and get JWT token |

### File Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/file/upload` | Upload and encrypt a file |
| GET | `/api/file/download/{fileId}` | Download and decrypt a file |
| GET | `/api/file/metadata/{fileId}` | Get file metadata |
| GET | `/api/file/list` | List user's files |
| DELETE | `/api/file/{fileId}` | Delete a file |
| GET | `/api/file/performance/{fileId}` | Get performance metrics |

### Admin Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/file/admin/all` | Get all files (admin only) |

## 📱 Usage

### Register a New User

```
bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "johndoe",
    "email": "john@example.com",
    "password": "password123",
    "role": "AUTHORIZED_USER"
  }'
```

### Login

```
bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "johndoe",
    "password": "password123"
  }'
```

### Upload a File

```
bash
curl -X POST http://localhost:8080/api/file/upload \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -F "file=@/path/to/file.txt"
```

### Download a File

```
bash
curl -X GET http://localhost:8080/api/file/download/{fileId} \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -o downloaded_file.txt
```

## 🔧 Configuration

### Storage Modes

The application supports two storage modes:

1. **Local Storage** (Default):
   
```
properties
   storage.mode=local
   storage.local.path=C:/edke-uploads
   
```

2. **Backblaze B2 Cloud Storage**:
   
```
properties
   storage.mode=backblaze
   backblaze.keyId=your_key_id
   backblaze.applicationKey=your_application_key
   backblaze.bucketName=your_bucket_name
   backblaze.endpoint=https://s3.us-east-005.backblazeb2.com
   
```

### User Roles

| Role | Description |
|------|-------------|
| ADMIN | Full access to all files and operations |
| DATA_OWNER | Can manage their own files and grant access |
| AUTHORIZED_USER | Can upload/download their own files |

## 🔐 Security Features

- **JWT Token Authentication**: Secure stateless authentication
- **Dual Key Encryption**: Two-layer key protection for AES key
- **Password Hashing**: BCrypt password encryption
- **Role-Based Access Control**: Granular permission system
- **Data Integrity Verification**: SHA-256 hash verification
- **Secure Key Management**: Session-based key generation
- **CORS Configuration**: Controlled cross-origin requests

## 📊 Database Schema

### Users Table
```
sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    role VARCHAR(50) NOT NULL,
    enabled BOOLEAN DEFAULT TRUE
);
```

### File Metadata Table
```
sql
CREATE TABLE file_metadata (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    file_name VARCHAR(255),
    file_type VARCHAR(255),
    file_size BIGINT,
    hash_value VARCHAR(255),
    encrypted_aes_key BLOB,
    vm_key BLOB,
    server_key BLOB,
    encryption_algorithm VARCHAR(50),
    encryption_time_ms BIGINT,
    decryption_time_ms BIGINT,
    file_status VARCHAR(50),
    integrity_status VARCHAR(50),
    storage_location VARCHAR(50),
    storage_path VARCHAR(500),
    uploaded_by BIGINT,
    download_count INT DEFAULT 0,
    upload_timestamp TIMESTAMP,
    last_access_time TIMESTAMP,
    accessible BOOLEAN DEFAULT TRUE,
    requires_role_auth BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (uploaded_by) REFERENCES users(id)
);
```

## 🧪 Testing

Run the tests:
```
bash
mvn test
```

The project includes tests for:
- Encryption Service
- Decryption Service
- Performance Analyzer
- User Service
- User Controller Integration

## 🚀 Deployment

### Production Deployment

1. **Build the JAR**:
   
```
bash
   mvn clean package -DskipTests
   
```

2. **Run the JAR**:
   
```
bash
   java -jar target/cloudsecurity-0.0.1-SNAPSHOT.jar
   
```

### Docker Deployment

Create a `Dockerfile`:
```
dockerfile
FROM eclipse-temurin:21-jdk-alpine
WORKDIR /app
COPY target/cloudsecurity-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build and run:
```
bash
docker build -t edke-cloudsecurity .
docker run -p 8080:8080 edke-cloudsecurity
```

## 📈 Performance

The application includes built-in performance monitoring that tracks:

- Encryption time (ms)
- Decryption time (ms)
- File size
- Throughput (MB/s)
- Performance metrics per file

Access performance data via:
```
GET /api/file/performance/{fileId}
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Commit changes: `git commit -am 'Add feature'`
4. Push to branch: `git push origin feature-name`
5. Submit a pull request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- [Spring Boot](https://spring.io/projects/spring-boot) for the framework
- [Backblaze](https://www.backblaze.com) for cloud storage
- [JWT](https://github.com/jwtk/jjwt) for token authentication

## 📞 Support

If you encounter any issues or have questions:

1. Check the Issues section
2. Create a new issue with detailed information
3. Contact the maintainer

---

**Enhanced Data Key Encryption (DKE) Protocol for Secure Cloud Data Storage**

*Made with ❤️ by Shashikanth*
