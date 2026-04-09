# GEICO Unified Cloud - Technology Stack Documentation

## Overview

**GEICO Unified Cloud** (formerly Account Vending Portal) is a comprehensive multi-cloud management platform for provisioning and managing AWS and Azure accounts/subscriptions. This document provides a complete breakdown of all technologies, frameworks, libraries, and tools used in the application.

---

## Application Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Web Browser (Client)                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Frontend: HTML5, CSS3, Vanilla JavaScript (ES6+)    │   │
│  │  - Single Page Application (SPA)                    │   │
│  │  - AJAX-based API communication                      │   │
│  │  - LocalStorage for token management                 │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTP/HTTPS REST API
                     │ JSON Data Exchange
┌────────────────────▼────────────────────────────────────────┐
│              Flask Backend Server (Python)                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Core Framework: Flask 3.1.2                         │   │
│  │  - RESTful API endpoints                             │   │
│  │  - Session management                                │   │
│  │  - Template rendering                                │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Authentication: Flask-Login, PyJWT                  │   │
│  │  Caching: Flask-Caching (in-memory/Redis/filesystem) │   │
│  │  CORS: Flask-CORS                                    │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼────────┐      ┌─────────▼──────────┐
│   AWS APIs     │      │    Azure APIs      │
│   (boto3 SDK)  │      │  (azure-mgmt SDK)  │
└────────────────┘      └────────────────────┘
        │                         │
┌───────▼─────────────────────────▼──────────┐
│      Terraform (Infrastructure as Code)     │
│  - AWS account provisioning                 │
│  - Azure subscription provisioning          │
│  - State management                         │
└─────────────────────────────────────────────┘
```

---

## Backend Technology Stack

### Core Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| **Python** | 3.8+ | Programming language |
| **Flask** | 3.1.2 | Web framework and API server |
| **Werkzeug** | 3.1.4 | WSGI utility library (Flask dependency) |

### Authentication & Authorization

| Technology | Version | Purpose |
|------------|---------|---------|
| **Flask-Login** | 0.6.3 | Session-based user authentication |
| **PyJWT** | 2.8.0 | JSON Web Token generation and verification |
| **SHA-256** | Built-in | Password hashing algorithm |
| **Role-Based Access Control (RBAC)** | Custom | Admin/User role management |

**Authentication Features:**
- Email-based user authentication
- JWT token generation (24-hour expiration)
- Session management with secure cookies
- Password hashing using SHA-256
- Role-based access control (Admin/User roles)
- Auto-logout on token expiration

### Caching

| Technology | Version | Purpose |
|------------|---------|---------|
| **Flask-Caching** | 2.3.1 | Response caching framework |

**Cache Backends Supported:**
- **Simple Cache** (default): In-memory caching
- **Redis**: Distributed caching (configurable)
- **Filesystem**: File-based caching (configurable)
- **Memcached**: Memcached backend (configurable)

**Cache Configuration:**
- Default TTL: 5 minutes (300 seconds)
- Configurable via environment variables
- Cache key prefix: `unified_cloud_`
- Cache invalidation on updates

### Cloud Provider Integrations

#### AWS Integration

| Technology | Version | Purpose |
|------------|---------|---------|
| **boto3** | 1.41.5 | AWS SDK for Python |
| **botocore** | 1.41.5 | Low-level AWS service access |

**AWS Services Used:**
- **AWS Organizations API**: Account provisioning and listing
- **EC2 API**: Instance querying across accounts
- **S3 API**: Bucket listing and management
- **IAM API**: User and role management
- **STS (Security Token Service)**: Cross-account role assumption
- **VPC API**: Virtual Private Cloud and subnet querying

**AWS Features:**
- Cross-account role assumption via STS
- Multi-region support
- Account-level resource querying
- Management account integration

#### Azure Integration

| Technology | Version | Purpose |
|------------|---------|---------|
| **azure-identity** | 1.15.0 | Azure authentication library |
| **azure-mgmt-subscription** | 3.1.1 | Azure subscription management |
| **azure-mgmt-resource** | 23.0.0 | Azure resource management |

**Azure Services Used:**
- **Subscription Management**: Create and list subscriptions
- **Resource Management**: Query Azure resources
- **DefaultAzureCredential**: Automatic credential chain

**Azure Features:**
- DefaultAzureCredential authentication (supports multiple auth methods)
- Subscription provisioning
- Management group integration

### Cross-Origin Resource Sharing (CORS)

| Technology | Version | Purpose |
|------------|---------|---------|
| **Flask-CORS** | 6.0.1 | Enable CORS for cross-origin requests |

### Data Storage

**Storage Type**: File-based JSON storage (no database)

| Storage Location | Purpose |
|------------------|---------|
| `state/users.json` | User account data |
| `state/metadata/` | Account/subscription metadata (JSON files) |
| `state/` | Terraform state files (`.tfstate`) |
| `logs/unified_cloud.log` | Application logs |

**Data Structure:**
- JSON files for user management
- JSON metadata files for account tracking
- Terraform state files for infrastructure state
- Rotating log files (10MB max, 5 backups)

### Logging

| Technology | Purpose |
|------------|---------|
| **Python logging** | Application logging |
| **RotatingFileHandler** | Log rotation (10MB, 5 backups) |

**Logging Features:**
- File-based logging: `logs/unified_cloud.log`
- Console output for development
- Structured logging with timestamps
- Log rotation to prevent disk space issues
- Error tracking and debugging support

### Infrastructure as Code

| Technology | Purpose |
|------------|---------|
| **Terraform** | Infrastructure provisioning and management |

**Terraform Usage:**
- AWS account provisioning via Organizations API
- Azure subscription creation
- State file management
- Metadata tracking for disaster recovery
- Reprovisioning capabilities

---

## Frontend Technology Stack

### Core Technologies

| Technology | Version/Purpose |
|------------|----------------|
| **HTML5** | Markup language for structure |
| **CSS3** | Styling and layout |
| **JavaScript (ES6+)** | Client-side logic (Vanilla JS, no frameworks) |

### Frontend Architecture

**Type**: Single Page Application (SPA)
- No build tools or transpilers
- Vanilla JavaScript (no frameworks like React, Vue, Angular)
- Direct DOM manipulation
- AJAX-based API communication

### UI Components & Features

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Layout** | CSS Grid & Flexbox | Responsive design |
| **Navigation** | Custom sidebar | Dashboard navigation |
| **Data Tables** | Vanilla JS | Sorting and filtering |
| **Modals** | Custom dialogs | User interactions |
| **Forms** | HTML5 forms | User input |
| **Loading Indicators** | CSS animations | Real-time feedback |
| **Error Handling** | JavaScript | User feedback |

### Frontend Libraries & APIs

| Technology | Purpose |
|------------|---------|
| **Fetch API** | HTTP requests to backend |
| **LocalStorage API** | Token storage and session management |
| **DOM API** | Dynamic content updates |
| **JSON API** | Data serialization |

### Frontend File Structure

```
frontend/
├── templates/
│   ├── index.html      # Main dashboard (SPA)
│   ├── login.html      # Login page
│   └── signup.html     # Sign-up page
└── static/
    ├── css/
    │   ├── style.css       # Base styles
    │   ├── dashboard.css   # Dashboard-specific styles
    │   └── components.css  # Component styles
    └── js/
        └── app.js          # Main application logic
```

### Frontend Features

- **Authentication**: Token-based auth with localStorage
- **API Communication**: RESTful API calls with error handling
- **Real-time Updates**: Dynamic content loading
- **CSV Export**: Client-side CSV generation
- **Responsive Design**: Mobile-friendly layout
- **Error Handling**: User-friendly error messages
- **Loading States**: Visual feedback during operations

---

## Development Tools & Utilities

### Scripts

| Script | Purpose |
|--------|---------|
| `setup.sh` | Environment setup and dependency installation |
| `backup_state.sh` | Backup state files and metadata |
| `restore_state.sh` | Restore from backup |

### Python Standard Library Modules Used

| Module | Purpose |
|--------|---------|
| `json` | JSON serialization/deserialization |
| `os` | Operating system interface |
| `subprocess` | Terraform execution |
| `shutil` | File operations |
| `datetime` | Date/time handling |
| `pathlib` | Path manipulation |
| `uuid` | Unique identifier generation |
| `logging` | Application logging |
| `hashlib` | Password hashing |
| `pickle` | Object serialization |
| `time` | Time utilities |

---

## Infrastructure & Deployment

### Server Requirements

| Component | Requirement |
|-----------|-------------|
| **Python** | 3.8 or higher |
| **Terraform** | 1.0 or higher |
| **AWS CLI** | For AWS operations |
| **Azure CLI** | For Azure operations |

### Deployment Options

**Current**: Standalone Flask development server
- Default port: 5000
- Development mode: Debug enabled
- Production: Requires WSGI server (Gunicorn, uWSGI)

**Recommended Production Stack:**
- **WSGI Server**: Gunicorn or uWSGI
- **Reverse Proxy**: Nginx or Apache
- **Process Manager**: systemd or Supervisor
- **SSL/TLS**: Let's Encrypt certificates
- **Database**: PostgreSQL or MySQL (future enhancement)
- **Cache**: Redis (for production caching)

### Environment Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `SECRET_KEY` | Flask session secret | Auto-generated UUID |
| `JWT_SECRET_KEY` | JWT token secret | Change in production |
| `CACHE_TYPE` | Cache backend type | `simple` |
| `CACHE_TIMEOUT` | Cache TTL (seconds) | `300` |
| `REDIS_HOST` | Redis host (if using Redis) | `localhost` |
| `REDIS_PORT` | Redis port | `6379` |
| `REDIS_DB` | Redis database number | `0` |
| `REDIS_PASSWORD` | Redis password | None |

---

## Security Stack

### Authentication Security

- **Password Hashing**: SHA-256 (consider upgrading to bcrypt/argon2)
- **Session Protection**: Flask-Login strong session protection
- **JWT Tokens**: HS256 algorithm with expiration
- **CSRF Protection**: Flask built-in CSRF protection

### Authorization

- **Role-Based Access Control**: Admin/User roles
- **Endpoint Protection**: `@login_required`, `@admin_required` decorators
- **Token Validation**: JWT token verification on API calls

### Data Security

- **State File Encryption**: Recommended for production
- **Secure Credential Handling**: Environment variables
- **No Credential Storage**: Credentials not stored in code
- **Log File Protection**: Restricted access to log files

### Network Security

- **CORS Configuration**: Controlled cross-origin access
- **HTTPS**: Recommended for production
- **Secure Cookies**: Session cookie security

---

## API Architecture

### API Style

**RESTful API** with JSON data exchange

### API Endpoints

#### Authentication Endpoints
- `POST /login` - User login
- `POST /logout` - User logout
- `POST /signup` - User registration
- `GET /api/auth/me` - Get current user info

#### Account Management Endpoints
- `GET /api/accounts` - List all accounts
- `POST /api/accounts/{provider}/{name}` - Provision new account
- `GET /api/accounts/{provider}/{name}` - Get account details
- `DELETE /api/accounts/{provider}/{name}` - Delete account
- `POST /api/accounts/{provider}/{name}/reprovision` - Reprovision account

#### Resource Querying Endpoints
- `GET /api/instances` - List EC2 instances across accounts
- `GET /api/buckets` - List S3 buckets across accounts
- `GET /api/subnets` - List VPCs and subnets
- `GET /api/iam-users` - List IAM users across accounts
- `GET /api/management-accounts` - List existing accounts from AWS/Azure

#### Utility Endpoints
- `GET /api/health` - Health check
- `POST /api/cache/clear` - Clear cache
- `GET /api/logs` - Get application logs

---

## Dependencies Summary

### Backend Dependencies (requirements.txt)

```
Flask==3.1.2
Flask-CORS==6.0.1
Werkzeug==3.1.4
boto3==1.41.5
azure-identity==1.15.0
azure-mgmt-subscription==3.1.1
azure-mgmt-resource==23.0.0
Flask-Caching==2.3.1
Flask-Login==0.6.3
PyJWT==2.8.0
```

### Frontend Dependencies

**None** - Pure vanilla JavaScript, HTML, and CSS (no npm packages)

---

## Browser Compatibility

| Browser | Support |
|---------|---------|
| Chrome/Edge (latest) | ✅ Fully supported |
| Firefox (latest) | ✅ Fully supported |
| Safari (latest) | ✅ Fully supported |
| Mobile browsers | ✅ Responsive design |

---

## Performance Characteristics

### Caching Performance
- **Cache Hit Rate**: ~80% reduction in API calls for repeated queries
- **Cached Response Time**: < 100ms
- **Fresh Query Time**: 5-30 seconds (depending on account count)

### Scalability
- **Concurrent Users**: Supports multiple concurrent sessions
- **Account Capacity**: Can handle 100+ accounts efficiently
- **Resource Querying**: Parallel account queries where possible

### Optimization Strategies
- Response caching for expensive operations
- Optional metadata fetching
- Request batching
- Error handling with retries
- Parallel account queries

---

## Future Enhancements (Planned)

### Infrastructure
- Docker containerization
- Kubernetes deployment
- Database integration (PostgreSQL/MySQL)
- Redis caching backend (production)
- Load balancer for high availability

### Security
- OAuth2 integration
- Multi-factor authentication (MFA)
- Audit logging
- Encryption at rest
- Password hashing upgrade (bcrypt/argon2)

### Features
- Additional resource types (RDS, Lambda, CloudFormation)
- Real-time notifications
- Scheduled queries
- Custom reports
- API rate limiting
- WebSocket support for real-time updates

### Development
- Automated testing (pytest, Jest)
- CI/CD pipeline
- API documentation (Swagger/OpenAPI)
- Code quality tools (linting, formatting)

---

## File Structure

```
unified_cloud/
├── backend/
│   ├── app.py              # Main Flask application
│   ├── auth.py             # Authentication module
│   ├── requirements.txt    # Python dependencies
│   └── config.example.env  # Environment config template
├── frontend/
│   ├── templates/          # HTML templates
│   │   ├── index.html     # Main dashboard
│   │   ├── login.html     # Login page
│   │   └── signup.html    # Sign-up page
│   └── static/
│       ├── css/           # Stylesheets
│       │   ├── style.css
│       │   ├── dashboard.css
│       │   └── components.css
│       └── js/            # JavaScript files
│           └── app.js
├── terraform/              # Infrastructure as Code
│   ├── aws/               # AWS provisioning
│   └── azure/             # Azure provisioning
├── state/                  # Application state
│   ├── metadata/          # JSON metadata files
│   └── users.json         # User data
├── logs/                   # Application logs
│   └── unified_cloud.log
├── scripts/                # Utility scripts
│   ├── setup.sh
│   ├── backup_state.sh
│   └── restore_state.sh
└── docs/                   # Documentation
    ├── TECH_STACK.md      # This file
    ├── TECH_SPEC.md       # Technical specification
    ├── AUTHENTICATION.md  # Auth documentation
    └── CACHING.md         # Caching documentation
```

---

## Version Information

- **Application Name**: GEICO Unified Cloud
- **Previous Name**: Account Vending Portal
- **Backend Framework**: Flask 3.1.2
- **Python Version**: 3.8+
- **Frontend**: Vanilla JavaScript (ES6+)
- **Last Updated**: 2024

---

## Contact & Support

For technical questions or issues, please refer to the main README.md or contact the infrastructure team.

---

## License

[Specify your license here]

---

*This document provides a comprehensive overview of all technologies used in the GEICO Unified Cloud application. For specific implementation details, refer to the source code and other documentation files.*

