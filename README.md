# Digital Heirloom Application

A self-hostable platform for securely storing digital assets that can be inherited by loved ones after death. Built with Django, PostgreSQL, and enterprise-grade encryption.

## Overview

The Digital Heirloom Application allows users to:
- Securely store documents, photos, videos, and messages
- Designate beneficiaries for digital inheritance
- Implement time-based access controls
- Ensure privacy through multi-layer encryption
- Manage digital estate with legal compliance

### Key Features

- **Zero-Knowledge Architecture**: Server never accesses unencrypted user data
- **Multi-Layer Encryption**: AES-256-GCM with PBKDF2 key derivation
- **Death Verification**: Automated and manual verification systems
- **Beneficiary Management**: Secure asset distribution to designated heirs
- **Legal Compliance**: GDPR, CCPA, and digital estate law compliance
- **Self-Hostable**: Complete control over your data and infrastructure

## Technology Stack

- **Backend**: Django 5.x
- **Database**: PostgreSQL 15+
- **Package Manager**: uv (Python)
- **Styling**: TailwindCSS
- **Encryption**: AES-256-GCM, PBKDF2-SHA256
- **Authentication**: Multi-factor authentication
- **Storage**: Encrypted local filesystem
- **Deployment**: Docker, Docker Compose
- **Architecture**: Server-side rendered (no client-side frontend)

## Development Setup

### Prerequisites

- Python 3.11+
- uv (Python package manager)
- PostgreSQL 15+
- Redis (for caching and sessions)
- Node.js 18+ (for TailwindCSS compilation)

### Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Heirloom
   ```

2. **Set up Python environment**
   ```bash
   # Install uv if not already installed
   curl -LsSf https://astral.sh/uv/install.sh | sh
   
   # Create virtual environment and install dependencies
   uv venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   uv pip install -r requirements.txt
   ```

3. **Configure environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. **Set up TailwindCSS**
   ```bash
   npm install
   npm run build-css
   ```

5. **Set up database**
   ```bash
   createdb heirloom_dev
   python manage.py migrate
   python manage.py createsuperuser
   ```

6. **Run development server**
   ```bash
   # Terminal 1: Start TailwindCSS watcher
   npm run watch-css
   
   # Terminal 2: Start Django server
   python manage.py runserver
   ```

### Environment Variables

Create a `.env` file with the following variables:

```env
# Django Settings
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/heirloom_dev

# Redis
REDIS_URL=redis://localhost:6379/0

# Email Configuration
EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_HOST_USER=your-email@example.com
EMAIL_HOST_PASSWORD=your-email-password
EMAIL_USE_TLS=True

# Encryption Keys
MASTER_ENCRYPTION_KEY=base64-encoded-key
DATABASE_ENCRYPTION_KEY=base64-encoded-key

# File Storage
MEDIA_ROOT=/path/to/encrypted/storage
MAX_FILE_SIZE=100MB
```

### Docker Development

```bash
# Build and start services
docker-compose up -d

# Run migrations
docker-compose exec web python manage.py migrate

# Create superuser
docker-compose exec web python manage.py createsuperuser
```

### Running Tests

```bash
# Run all tests
python manage.py test

# Run with coverage
coverage run --source='.' manage.py test
coverage report
coverage html
```

## Deploy to Production

### Server Requirements

- **Minimum**: 4GB RAM, 2 CPU cores, 100GB storage
- **Recommended**: 8GB RAM, 4 CPU cores, 500GB SSD
- **OS**: Ubuntu 22.04 LTS or similar
- **SSL**: Valid SSL certificate (Let's Encrypt recommended)

### Production Deployment

1. **Server Setup**
   ```bash
   # Update system
   sudo apt update && sudo apt upgrade -y
   
   # Install dependencies
   sudo apt install -y python3-pip postgresql postgresql-contrib redis-server nginx
   ```

2. **Database Setup**
   ```bash
   sudo -u postgres createuser --createdb heirloom
   sudo -u postgres createdb heirloom_prod
   sudo -u postgres psql -c "ALTER USER heirloom WITH PASSWORD 'secure-password';"
   ```

3. **Application Deployment**
   ```bash
   # Clone and setup
   git clone <repository-url> /opt/heirloom
   cd /opt/heirloom
   
   # Install uv and setup environment
   curl -LsSf https://astral.sh/uv/install.sh | sh
   uv venv
   source .venv/bin/activate
   uv pip install -r requirements.txt
   
   # Build TailwindCSS for production
   npm install
   npm run build-css
   
   # Configure production settings
   cp .env.production .env
   # Edit .env with production values
   
   # Run migrations
   python manage.py migrate
   python manage.py collectstatic --noinput
   ```

4. **Systemd Service**
   ```bash
   sudo cp deploy/heirloom.service /etc/systemd/system/
   sudo systemctl enable heirloom
   sudo systemctl start heirloom
   ```

5. **Nginx Configuration**
   ```bash
   sudo cp deploy/nginx.conf /etc/nginx/sites-available/heirloom
   sudo ln -s /etc/nginx/sites-available/heirloom /etc/nginx/sites-enabled/
   sudo systemctl reload nginx
   ```

### Docker Production Deployment

```bash
# Production docker-compose
docker-compose -f docker-compose.prod.yml up -d

# SSL with Let's Encrypt
docker-compose -f docker-compose.prod.yml exec certbot certbot --nginx
```

### Security Checklist

- [ ] SSL/TLS certificate installed and configured
- [ ] Firewall configured (ports 80, 443 only)
- [ ] Database access restricted to application
- [ ] Regular security updates scheduled
- [ ] Backup system configured and tested
- [ ] Monitoring and alerting configured
- [ ] Log rotation configured
- [ ] File permissions properly set

## Contribute

We welcome contributions to the Digital Heirloom Application! Please follow these guidelines:

### Getting Started

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Set up development environment (see Development Setup)
4. Make your changes
5. Write tests for new functionality
6. Ensure all tests pass
7. Submit a pull request

### Code Standards

- **Python**: Follow PEP 8 style guide
- **Django**: Follow Django best practices
- **Security**: All code must pass security review
- **Testing**: Minimum 90% test coverage required
- **Documentation**: Update docs for any new features

### Security Guidelines

- Never commit secrets or credentials
- All encryption code must be reviewed by security team
- Follow OWASP security guidelines
- Use parameterized queries to prevent SQL injection
- Validate and sanitize all user inputs
- Implement proper error handling without information leakage

### Pull Request Process

1. **Pre-submission Checklist**
   - [ ] All tests pass
   - [ ] Code follows style guidelines
   - [ ] Documentation updated
   - [ ] Security review completed
   - [ ] No sensitive data in commits

2. **Review Process**
   - Code review by at least 2 maintainers
   - Security review for encryption/auth changes
   - Performance review for database changes
   - UI/UX review for frontend changes

3. **Merge Requirements**
   - All CI checks pass
   - Approved by required reviewers
   - No merge conflicts
   - Branch up to date with main

### Reporting Issues

- **Security Issues**: Email security@heirloom-app.com (do not create public issues)
- **Bug Reports**: Use GitHub issues with bug template
- **Feature Requests**: Use GitHub issues with feature template
- **Questions**: Use GitHub discussions

### Development Workflow

```bash
# Start new feature
git checkout main
git pull origin main
git checkout -b feature/your-feature

# Setup environment
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt

# Start TailwindCSS watcher
npm run watch-css

# Make changes and test
python manage.py test
python manage.py check --deploy

# Build CSS for production
npm run build-css

# Commit changes
git add .
git commit -m "feat: add your feature description"

# Push and create PR
git push origin feature/your-feature
# Create pull request on GitHub
```

## Agent Instructions

When working with this codebase, AI agents should follow these guidelines:

### Code Generation Rules

1. **Security First**: Always prioritize security in code generation
   - Use parameterized queries for database operations
   - Implement proper input validation and sanitization
   - Follow encryption best practices
   - Never hardcode secrets or credentials

2. **Django Best Practices**
   - Use Django's built-in security features
   - Follow Model-View-Template pattern
   - Use Django forms for user input
   - Implement proper error handling
   - Use Django's authentication and authorization

3. **Encryption Implementation**
   - Always use the established encryption service layer
   - Never store unencrypted sensitive data
   - Implement proper key management
   - Use secure random number generation
   - Follow the multi-layer encryption architecture
4. **Commit to git**
   - If a task is successful, commit to git

### File Structure Guidelines

```
Heirloom/
├── apps/
│   ├── accounts/          # User management and authentication
│   ├── assets/            # Digital asset management
│   ├── beneficiaries/     # Beneficiary management
│   ├── encryption/        # Encryption services
│   ├── inheritance/       # Death verification and distribution
│   └── notifications/     # Notification system
├── config/                # Django configuration
├── static/                # Static files (CSS, JS, images)
│   ├── css/               # Compiled TailwindCSS
│   └── src/               # TailwindCSS source files
├── templates/             # Django HTML templates
├── tests/                 # Test files
├── utils/                 # Utility functions
├── package.json           # Node.js dependencies (TailwindCSS)
├── tailwind.config.js     # TailwindCSS configuration
└── pyproject.toml         # uv project configuration
```

### Testing Requirements

- Write unit tests for all new functions
- Include integration tests for user workflows
- Test encryption/decryption thoroughly
- Mock external services in tests
- Achieve minimum 90% code coverage

### Documentation Standards

- Document all public APIs
- Include docstrings for all functions and classes
- Update README for new features
- Create user guides for complex features
- Document security considerations

### Performance Considerations

- Optimize database queries (use select_related, prefetch_related)
- Implement proper caching strategies
- Handle large file uploads efficiently
- Use pagination for large datasets
- Monitor and optimize encryption performance

### Error Handling

- Use Django's logging framework
- Implement graceful error handling
- Never expose sensitive information in errors
- Provide user-friendly error messages
- Log security events appropriately

### Code Review Focus Areas

1. **Security**: Encryption implementation, input validation, authentication
2. **Performance**: Database queries, file handling, caching
3. **Maintainability**: Code structure, documentation, testing
4. **User Experience**: Error handling, UI/UX, accessibility
5. **Compliance**: GDPR, data protection, legal requirements

### Common Patterns

```python
# Encryption service usage
from apps.encryption.services import EncryptionService

# Always use the service layer
encrypted_data = EncryptionService.encrypt_field(data, user_key)
decrypted_data = EncryptionService.decrypt_field(encrypted_data, user_key)

# Database operations with encryption
class HeirloomManager(models.Manager):
    def create_encrypted(self, user, **kwargs):
        # Implement encrypted creation logic
        pass

# View security
from django.contrib.auth.decorators import login_required
from apps.accounts.decorators import require_mfa

@login_required
@require_mfa
def secure_view(request):
    # Implement secure view logic
    pass
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Documentation**: [docs.heirloom-app.com](https://docs.heirloom-app.com)
- **Community**: [GitHub Discussions](https://github.com/your-org/heirloom/discussions)
- **Issues**: [GitHub Issues](https://github.com/your-org/heirloom/issues)
- **Security**: security@heirloom-app.com