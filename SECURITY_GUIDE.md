# E-DKE Cloud Security - Security & Validation Guide

## Phase 4: Security Best Practices & Hardening

### 1. Input Validation & Error Handling

#### File Upload Validation
- ✅ Validate file size (max 100MB)
- ✅ Validate file type (whitelist: .txt, .pdf, .jpg, .png, .docx)
- ✅ Scan for malware (optional: integrate VirusTotal API)
- ✅ Sanitize file names
- ✅ Validate MIME types

#### Authentication Input Validation
```java
- Username: 3-50 alphanumeric characters
- Password: minimum 8 characters, must contain uppercase, lowercase, number
- Email: valid email format
- Role: only ADMIN, DATA_OWNER, AUTHORIZED_USER
```

### 2. Security Headers

Add to SecurityConfig:
```java
http.headers(headers -> headers
    .contentSecurityPolicy(csp -> csp.policyDirectives("default-src 'self'"))
    .xssProtection()
    .frameOptions(frameOptions -> frameOptions.deny())
);
```

### 3. Rate Limiting

Implement rate limiting on authentication endpoints:
```
- Login: Max 5 attempts per 15 minutes per IP
- Register: Max 3 registrations per hour per IP
- File upload: Max 10 uploads per hour per user
```

### 4. Logging & Audit Trail

Audit all sensitive operations:
```
✅ User login attempts (success/failure)
✅ File uploads (user, file name, size, time)
✅ File downloads (user, file name, timestamp)
✅ File deletions (user, file name, timestamp)
✅ Role changes (admin, user, old role, new role)
✅ Failed authentication attempts
```

### 5. Cryptographic Security

#### Key Management
- ✅ Session keys renewed for each file upload
- ✅ Keys stored in database for decryption
- ✅ Implement key rotation policy (90 days)

#### Algorithm Verification
- ✅ AES-128 encryption (industry standard)
- ✅ SHA-256 hashing (cryptographically secure)
- ✅ DKE dual-key system (proprietary enhancement)

### 6. Database Security

#### SQL Injection Prevention
- ✅ Use JPA named parameters (not string concatenation)
- ✅ JpaRepository uses parameterized queries

#### Data Protection
- ✅ Hash passwords with BCrypt (rounds: 10)
- ✅ Encrypt sensitive files at rest
- ✅ Use HTTPS for all communication

### 7. RBAC Authorization

#### Role Permissions Matrix

| Operation | ADMIN | DATA_OWNER | AUTHORIZED_USER |
|-----------|-------|------------|-----------------|
| Register | ✅ | ✅ | ✅ |
| Login | ✅ | ✅ | ✅ |
| Upload File | ✅ | ✅ | ❌ |
| Download Own | ✅ | ✅ | ❌ |
| Download Other | ✅ | ✅ | Only if granted |
| Delete Own | ✅ | ✅ | ❌ |
| Delete Other | ✅ | ❌ | ❌ |
| Manage Users | ✅ | ❌ | ❌ |
| View All Files | ✅ | ❌ | ❌ |

### 8. JWT Token Security

#### Configuration
```properties
jwt.secret=<strong-random-256-bit-key>
jwt.expiration=3600000 (1 hour)
```

#### Token Validation
- ✅ Check signature on every request
- ✅ Verify expiration time
- ✅ Validate username against database
- ✅ Check token rotation if needed

### 9. HTTPS/TLS Configuration

```properties
server.port=8443
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=<password>
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=tomcat
```

### 10. Session Management

- ✅ Stateless sessions (SessionCreationPolicy.STATELESS)
- ✅ Token-based authentication only
- ✅ No session cookies
- ✅ Token expiration: 1 hour
- ✅ Refresh token endpoint (optional)

### 11. CORS Configuration

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://yourdomain.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

### 12. Response Security

#### Information Disclosure Prevention
- ✅ Generic error messages (no stack traces to clients)
- ✅ Log detailed errors server-side
- ✅ Return 404 instead of 403 for non-existent resources

#### Response Headers
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### 13. Dependency Vulnerabilities

Run security audit:
```bash
./mvnw dependency-check:aggregate
./mvnw org.owasp:dependency-check-maven:check
```

### 14. Code Quality & Security Scanning

#### SonarQube Integration
```bash
./mvnw clean verify sonar:sonar \
  -Dsonar.projectKey=edke-cloud-security \
  -Dsonar.sources=src/main/java \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=<token>
```

### 15. Compliance Checklist

- [ ] OWASP Top 10 compliance verified
- [ ] GDPR compliance (data privacy)
- [ ] PCI DSS compliance (if storing payment data)
- [ ] Encryption in transit (HTTPS)
- [ ] Encryption at rest (database)
- [ ] Regular security audits
- [ ] Penetration testing completed
- [ ] Security training for developers

### 16. Monitoring & Alerting

#### Setup ELK Stack
```
Elasticsearch -> Logstash -> Kibana
```

#### Alert Triggers
- Multiple failed login attempts (5+ in 15 min)
- File integrity check failures
- Unauthorized access attempts
- Large file downloads (>50MB)
- System error rate > 5%

### 17. Deployment Security

#### Production Checklist
- [ ] Change default passwords
- [ ] Disable debug mode
- [ ] Enable HTTPS with valid certificate
- [ ] Configure WAF rules
- [ ] Setup VPN/private network
- [ ] Implement backup & recovery
- [ ] Setup monitoring dashboard
- [ ] Configure log aggregation
- [ ] Document security procedures
- [ ] Create incident response plan

### 18. Vulnerability Disclosure

If security vulnerability is found:
1. Report to: security@edke.com
2. Do NOT disclose publicly
3. Allow 90 days for patch
4. Include: description, impact, PoC
5. Sign NDA if required

---

## Testing Commands

### Run All Tests
```bash
./mvnw clean test
```

### Run Specific Test Class
```bash
./mvnw test -Dtest=EncryptionServiceTest
./mvnw test -Dtest=UserServiceTest
./mvnw test -Dtest=UserControllerIntegrationTest
```

### Run with Coverage
```bash
./mvnw clean test jacoco:report
```

### View Coverage Report
```
target/site/jacoco/index.html
```

---

## Regular Security Maintenance

### Monthly
- Check for dependency updates
- Review access logs for anomalies
- Rotate any exposed credentials

### Quarterly
- Security assessment
- Penetration testing
- Update security documentation

### Annually
- Full security audit
- Training renewal
- Disaster recovery drill
