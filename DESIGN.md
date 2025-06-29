# Digital Heirloom Application - Design Document

## Overview

The Digital Heirloom Application is a self-hostable platform that allows users to securely store digital assets (documents, photos, videos, messages, etc.) for their relatives to discover and access after their death. The system prioritizes security, privacy, and long-term data preservation.

## Architecture Overview

### Technology Stack
- **Backend**: Django 5.x
- **Database**: PostgreSQL 15+
- **Package Manager**: uv (Python)
- **Frontend**: Server-side rendered Django templates
- **Styling**: TailwindCSS
- **Encryption**: AES-256-GCM (symmetric encryption)
- **Key Derivation**: PBKDF2 with SHA-256
- **File Storage**: Local filesystem with encrypted storage
- **Authentication**: Django's built-in auth + custom security layers

### Core Components
1. **User Management System**
2. **Asset Storage Engine**
3. **Encryption/Decryption Service**
4. **Inheritance Management**
5. **Access Control System**
6. **Notification Service**

## Data Model

### Core Entities

#### User
```
- id (UUID)
- email (unique)
- password_hash
- master_key_salt
- created_at
- last_login
- is_deceased (boolean)
- death_verification_date
```

#### Heirloom (Digital Asset)
```
- id (UUID)
- owner_id (FK to User)
- title
- description
- asset_type (document, photo, video, message, etc.)
- encrypted_content_key
- file_path (encrypted file location)
- metadata_encrypted
- created_at
- updated_at
- access_after_date (optional)
```

#### Beneficiary
```
- id (UUID)
- heirloom_id (FK to Heirloom)
- email
- relationship
- access_key_encrypted
- notification_sent
- access_granted_at
```

#### AccessLog
```
- id (UUID)
- heirloom_id (FK to Heirloom)
- accessor_email
- access_timestamp
- ip_address
- user_agent
```

## Encryption Architecture

### Multi-Layer Encryption Strategy

#### Layer 1: User Master Key
- **Purpose**: Root encryption key for user's data
- **Generation**: Derived from user password using PBKDF2
- **Formula**: `master_key = PBKDF2(password, salt, 100000, SHA-256)`
- **Storage**: Never stored directly; regenerated from password

#### Layer 2: Content Encryption Keys (CEK)
- **Purpose**: Individual encryption keys for each heirloom
- **Generation**: Random 256-bit keys using cryptographically secure RNG
- **Encryption**: CEK encrypted with user's master key
- **Storage**: Encrypted CEK stored in database

#### Layer 3: File Encryption
- **Algorithm**: AES-256-GCM
- **Process**: Files encrypted with their respective CEK
- **Storage**: Encrypted files stored on filesystem
- **Integrity**: GCM provides authentication and integrity

#### Layer 4: Database Encryption
- **Sensitive Fields**: All PII and metadata encrypted at rest
- **Key Management**: Separate database encryption keys
- **Implementation**: Field-level encryption using Django's encryption utilities

### Encryption Flow

#### Asset Upload Process
1. User uploads file
2. Generate random CEK for this asset
3. Encrypt file with CEK using AES-256-GCM
4. Encrypt CEK with user's master key
5. Store encrypted file and encrypted CEK
6. Encrypt metadata with CEK

#### Asset Access Process
1. User authenticates and derives master key
2. Decrypt CEK using master key
3. Decrypt file using CEK
4. Serve decrypted content (never stored decrypted)

#### Beneficiary Access Process
1. Beneficiary receives access notification
2. Beneficiary creates account or uses temporary access
3. System decrypts beneficiary-specific access key
4. Access key used to decrypt relevant CEKs
5. Assets decrypted and presented

## Security Model

### Authentication & Authorization
- **Multi-factor Authentication**: Required for all users
- **Password Requirements**: Minimum 12 characters, complexity rules
- **Session Management**: Secure session tokens with short expiration
- **Rate Limiting**: Prevent brute force attacks

### Key Management
- **Key Rotation**: Annual rotation of database encryption keys
- **Key Escrow**: Optional secure key backup for account recovery
- **Zero-Knowledge**: Server never has access to unencrypted user data
- **Forward Secrecy**: Old keys cannot decrypt new data

### Access Control
- **Principle of Least Privilege**: Users only access their own data
- **Time-based Access**: Assets can be time-locked
- **Beneficiary Verification**: Email verification + optional identity verification
- **Audit Trail**: All access attempts logged

## Inheritance Management

### Death Verification Process
1. **Automated Triggers**:
   - Inactivity detection (configurable period)
   - Third-party death verification services
   - Manual reporting by family members

2. **Verification Methods**:
   - Death certificate upload
   - Government database verification (where available)
   - Multiple family member confirmation

3. **Grace Period**: 30-day waiting period before asset release

### Asset Distribution
1. **Notification Phase**: Beneficiaries notified via email
2. **Access Provisioning**: Temporary access keys generated
3. **Secure Access**: Beneficiaries access through secure portal
4. **Download Window**: Limited time window for asset download

## Data Protection & Privacy

### Encryption at Rest
- **Database**: All sensitive data encrypted using AES-256
- **File System**: All uploaded files encrypted
- **Backups**: Encrypted backups with separate key management
- **Logs**: Sensitive information in logs encrypted

### Encryption in Transit
- **TLS 1.3**: All communications encrypted
- **Certificate Pinning**: Prevent man-in-the-middle attacks
- **HSTS**: Force HTTPS connections

### Data Minimization
- **Metadata Limitation**: Only essential metadata stored
- **Automatic Purging**: Old logs and temporary data purged
- **Anonymization**: Personal identifiers anonymized where possible

## Implementation Security Considerations

### Django Security Settings
```python
# Critical security settings
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'
SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'
```

### Database Security
- **Connection Encryption**: SSL/TLS for all database connections
- **User Privileges**: Minimal database user privileges
- **Query Parameterization**: Prevent SQL injection
- **Regular Updates**: Keep PostgreSQL updated

### File System Security
- **Directory Permissions**: Restricted access to encrypted files
- **File Naming**: Obfuscated file names (UUID-based)
- **Storage Isolation**: Separate storage per user/tenant

## Backup & Recovery

### Backup Strategy
- **Encrypted Backups**: All backups encrypted with separate keys
- **Geographic Distribution**: Backups stored in multiple locations
- **Regular Testing**: Backup restoration tested monthly
- **Retention Policy**: 7-year retention for legal compliance

### Disaster Recovery
- **Recovery Time Objective (RTO)**: 4 hours
- **Recovery Point Objective (RPO)**: 1 hour
- **Failover Process**: Automated failover to backup systems
- **Data Integrity**: Checksums verify backup integrity

## Compliance & Legal

### Data Protection Compliance
- **GDPR**: Right to erasure, data portability, consent management
- **CCPA**: California privacy rights compliance
- **HIPAA**: Healthcare data protection (if applicable)

### Legal Considerations
- **Digital Estate Laws**: Compliance with local inheritance laws
- **Data Retention**: Legal requirements for data retention
- **Cross-border**: International data transfer compliance

## Monitoring & Alerting

### Security Monitoring
- **Failed Login Attempts**: Alert on suspicious activity
- **Unusual Access Patterns**: ML-based anomaly detection
- **System Integrity**: File integrity monitoring
- **Encryption Key Access**: Alert on key access attempts

### Performance Monitoring
- **Response Times**: API and page load monitoring
- **Database Performance**: Query performance tracking
- **Storage Usage**: Disk space and growth monitoring
- **Error Rates**: Application error tracking

## Deployment Architecture

### Self-Hosting Requirements
- **Minimum Hardware**: 4GB RAM, 2 CPU cores, 100GB storage
- **Operating System**: Ubuntu 22.04 LTS or similar
- **Docker Support**: Containerized deployment option
- **SSL Certificate**: Let's Encrypt integration

### Scalability Considerations
- **Horizontal Scaling**: Load balancer support
- **Database Scaling**: Read replicas for performance
- **File Storage**: Distributed storage options
- **Caching**: Redis for session and query caching

## Development Roadmap

### Phase 1: Core Platform (Months 1-3)
- User authentication and management
- Basic asset upload and encryption
- Simple beneficiary management

### Phase 2: Enhanced Security (Months 4-6)
- Multi-factor authentication
- Advanced encryption features
- Audit logging and monitoring

### Phase 3: Inheritance Features (Months 7-9)
- Death verification system
- Automated asset distribution
- Legal compliance features

### Phase 4: Advanced Features (Months 10-12)
- Mobile applications
- API for third-party integrations
- Advanced analytics and reporting

## Risk Assessment

### High-Risk Areas
1. **Key Management**: Loss of encryption keys = permanent data loss
2. **Death Verification**: False positives could trigger premature access
3. **Beneficiary Authentication**: Unauthorized access to inherited assets
4. **Data Breaches**: Encrypted data exposure

### Mitigation Strategies
1. **Key Escrow**: Optional secure key backup system
2. **Multi-step Verification**: Multiple confirmation methods
3. **Strong Authentication**: MFA and identity verification
4. **Defense in Depth**: Multiple security layers

## Conclusion

This design provides a robust, secure foundation for a digital heirloom application that prioritizes user privacy and data security through comprehensive encryption at all levels. The multi-layered approach ensures that even in the event of a security breach, user data remains protected through strong cryptographic controls.