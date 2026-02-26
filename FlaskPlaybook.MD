# Python Flask Execution Specification (Playbook)

> **Purpose**: This document defines the architectural standards, patterns, and constraints for all code generation and implementation in Python Flask projects. Feed this to any AI agent before implementation.

**Version**: 1.0  
**Last Updated**: February 2026  
**Status**: Production-Ready  

---

## Table of Contents

1. [Core Architecture Principles](#1-core-architecture-principles)
2. [Python & Type Safety Standards](#2-python--type-safety-standards)
3. [Code Documentation Standards](#3-code-documentation-standards)
4. [Testing Standards](#4-testing-standards)
5. [Code Organization & File Structure](#5-code-organization--file-structure)
6. [Flask-Specific Patterns](#6-flask-specific-patterns)
7. [RESTful API Design Standards](#7-restful-api-design-standards)
8. [Database & ORM Standards](#8-database--orm-standards)
9. [Configuration Management](#9-configuration-management)
10. [Feature Switches (Feature Flags)](#10-feature-switches-feature-flags)
11. [Security Standards](#11-security-standards)
12. [Logging Standards](#12-logging-standards)
13. [Caching Strategies](#13-caching-strategies)
14. [Rate Limiting](#14-rate-limiting)
15. [CI/CD & Deployment](#15-cicd--deployment)
16. [Docker & Containerization](#16-docker--containerization)
17. [Things to NEVER Do](#17-things-to-never-do)
18. [When Implementing New Features](#18-when-implementing-new-features)
19. [Agent Behavior Constraints](#19-agent-behavior-constraints)
20. [Summary Checklist](#20-summary-checklist)

---

## 1. Core Architecture Principles

### Single Responsibility Principle (SRP)

- Each class/module/function handles **one concern only**
- Blueprints are thin: handle HTTP, delegate to services
- Services contain business logic
- Models handle data structure
- Validators handle input validation
- Repositories handle data access (if using repository pattern)

**Example**:

```python
# ❌ BAD: User class doing too much
class User:
    def authenticate(self): pass
    def send_welcome_email(self): pass
    def process_payment(self): pass

# ✅ GOOD: Separated concerns
class AuthService:
    def authenticate(self): pass

class NotificationService:
    def send_welcome_email(self): pass

class PaymentService:
    def process_payment(self): pass
```

### Open/Closed Principle (OCP)

- Extend functionality via composition, inheritance, or strategy patterns
- Don't modify existing tested code for new features
- Use abstract base classes (ABC) or protocols for extensibility

**Example**:

```python
from abc import ABC, abstractmethod

# ✅ Service depends on abstraction
class PaymentGateway(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> bool:
        pass

class StripeGateway(PaymentGateway):
    def process_payment(self, amount: float) -> bool:
        # Stripe implementation
        pass

class PayPalGateway(PaymentGateway):
    def process_payment(self, amount: float) -> bool:
        # PayPal implementation
        pass
```

### Liskov Substitution Principle (LSP)

- Subclasses can replace parent classes without breaking functionality
- Don't override methods in ways that violate parent contracts

### Interface Segregation Principle (ISP)

- Break large interfaces into smaller, focused ones
- Clients shouldn't depend on methods they don't use

### Dependency Inversion Principle (DIP)

- High-level modules don't depend on low-level modules
- Both depend on abstractions
- Use dependency injection (Flask-Injector or manual)

### Keep it Simple, Stupid (KISS)

- Prefer straightforward solutions over clever ones
- No premature optimization
- If you can't explain it in one sentence, it's too complex

### Don't Repeat Yourself (DRY)

- Avoid code duplication
- Extract repeated logic into helper functions or utilities
- Use abstractions to prevent tight coupling

### You Ain't Gonna Need It (YAGNI)

- Only implement features when needed
- No premature optimization or unnecessary features

### Command-Query Separation (CQS)

- **Commands**: Mutate state, return None or success indicator
- **Queries**: Return data, never mutate state
- Name accordingly: `create_user()` vs `get_user_by_id()`

### Separation of Concerns (SoC)

- **Blueprints**: HTTP layer (routing, request/response mapping) - THIN
- **Services**: Business logic - CORE LOGIC HERE
- **Models**: Database schema (SQLAlchemy)
- **Schemas**: Request/response validation (Pydantic)
- **Validators**: Input validation logic
- **Middlewares**: Cross-cutting concerns (logging, auth)
- **Utils/Helpers**: Reusable utilities
- **Config**: Environment and configuration management

---

## 2. Python & Type Safety Standards

### Type Hints - MANDATORY

```python
# ✅ ALWAYS: Explicit typing everywhere
from typing import Optional, List, Dict, Any
from datetime import datetime

def process_order(
    order_id: int, 
    user_id: int,
    items: List[Dict[str, Any]]
) -> Order:
    pass

# ❌ NEVER: No type hints
def process_order(order_id, user_id, items):
    pass
```

### Strict Typing Rules

- Use type hints for ALL function parameters and return types
- Use `Optional[T]` or `T | None` for nullable values
- Use `List[T]`, `Dict[K, V]` for collections
- Use `Any` only when absolutely justified (document why)
- Use `Protocol` or `ABC` for interfaces
- Enable mypy strict mode for type checking

### Type Definitions

```python
from typing import NewType, TypeAlias
from datetime import datetime

# ✅ Type aliases for clarity
UserId = NewType('UserId', int)
OrderId = NewType('OrderId', int)
Email = NewType('Email', str)

# ✅ Complex type aliases
UserDict: TypeAlias = Dict[str, str | int | bool]
ResponseData: TypeAlias = Dict[str, Any]

# ✅ Explicit return types
def find_by_id(user_id: UserId) -> User | None:
    pass

def get_all_users(limit: int = 10) -> List[User]:
    pass
```

### Use Pydantic Models for Data Validation

- Use Pydantic `BaseModel` for request/response schemas
- Use dataclasses for internal data structures (when Pydantic is overkill)
- NEVER use plain dictionaries for structured data

**Example**:

```python
from pydantic import BaseModel, EmailStr, Field, validator
from datetime import datetime

# ✅ GOOD: Pydantic model with validation
class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8)
    age: int = Field(..., ge=18, le=120)
    
    @validator('password')
    def validate_password(cls, v):
        if not any(char.isdigit() for char in v):
            raise ValueError('Password must contain at least one digit')
        return v

# ❌ BAD: Plain dictionary
user_data = {
    "email": "test@example.com",
    "password": "secret",
    "age": 25
}
```

### Use Enums for Constants

```python
from enum import Enum

# ✅ GOOD: Type-safe enums
class UserRole(str, Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"

class OrderStatus(str, Enum):
    PENDING = "pending"
    PROCESSING = "processing"
    COMPLETED = "completed"
    CANCELLED = "cancelled"

# Usage
user.role = UserRole.ADMIN  # Type-safe
```

---

## 3. Code Documentation Standards

### Inline Comments

- **WHY over WHAT**: Explain business rules, not syntax
- Comment complex algorithms, edge cases, or non-obvious behavior
- No redundant comments

```python
# ❌ BAD: States the obvious
# Loop through users
for user in users:
    pass

# ✅ GOOD: Explains business context
# Users without confirmed emails can't access premium features
# per security policy SEC-2024-15
active_users = [u for u in users if u.email_confirmed]
```

### Google-Style Docstrings (MANDATORY)

Use Google-style docstrings for all public functions, classes, and modules.

**Function/Method Docstrings:**

```python
def create_user(data: UserCreate, db: Session) -> User:
    """
    Creates a new user in the database.
    
    This function hashes the password, validates email uniqueness,
    and sends a welcome email to the user.
    
    Args:
        data: User creation data containing email and password
        db: Database session
        
    Returns:
        User: The created user entity with hashed password
        
    Raises:
        DuplicateEmailError: If email already exists in database
        ValidationError: If data validation fails
        
    Example:
        >>> user_data = UserCreate(email="test@example.com", password="secret123")
        >>> user = create_user(user_data, db)
        >>> user.email
        'test@example.com'
    """
    pass
```

**Class Docstrings:**

```python
class UserService:
    """
    Service layer for user-related business logic.
    
    This service handles user creation, authentication, profile updates,
    and account management. It does NOT handle HTTP concerns - those
    belong in the blueprint/route layer.
    
    Business logic is framework-agnostic and can be reused across
    different interfaces (HTTP, CLI, GraphQL, etc.).
    
    Attributes:
        db: Database session
        email_service: Service for sending emails
        cache: Redis cache instance
    """
    
    def __init__(self, db: Session, email_service: EmailService):
        self.db = db
        self.email_service = email_service
```

---

## 4. Testing Standards

### Testing Framework: pytest

Use **pytest** as the primary testing framework.

Required packages:
- `pytest` - Core testing framework
- `pytest-flask` - Flask-specific fixtures
- `pytest-cov` - Coverage reporting
- `faker` or `factory-boy` - Test data generation

### Test Structure

```
tests/
├── unit/                       # Unit tests (business logic isolation)
│   ├── services/
│   │   ├── test_user_service.py
│   │   └── test_payment_service.py
│   ├── utils/
│   │   └── test_helpers.py
│   └── validators/
│       └── test_email_validator.py
├── integration/                # Integration tests (full flow)
│   ├── blueprints/
│   │   ├── test_user_routes.py
│   │   └── test_order_routes.py
│   └── database/
│       └── test_user_repository.py
├── e2e/                        # End-to-end tests
│   └── test_user_workflow.py
├── conftest.py                 # Shared fixtures
└── pytest.ini                  # Pytest configuration
```

### Unit Tests

Test business logic in isolation. Mock all external dependencies.

**Naming convention:**
- `test_<function_name>_<scenario>`

**Example:**

```python
import pytest
from unittest.mock import Mock, patch
from src.services.user_service import UserService
from src.schemas.user import UserCreate
from src.exceptions import DuplicateEmailError

@pytest.fixture
def mock_db():
    return Mock()

@pytest.fixture
def mock_email_service():
    service = Mock()
    service.send_welcome_email = Mock()
    return service

@pytest.fixture
def user_service(mock_db, mock_email_service):
    return UserService(db=mock_db, email_service=mock_email_service)

class TestUserService:
    """Unit tests for UserService business logic"""
    
    def test_create_user_hashes_password(self, user_service, mock_db):
        # Arrange
        data = UserCreate(email="test@example.com", password="plaintext123")
        mock_db.query().filter().first.return_value = None  # No existing user
        
        # Act
        user = user_service.create_user(data)
        
        # Assert
        assert user.password != "plaintext123"  # Password was hashed
        assert len(user.password) > 50  # Hashed password is long
    
    def test_create_user_raises_error_when_email_exists(self, user_service, mock_db):
        # Arrange
        data = UserCreate(email="existing@example.com", password="secret123")
        mock_db.query().filter().first.return_value = Mock()  # User exists
        
        # Act & Assert
        with pytest.raises(DuplicateEmailError):
            user_service.create_user(data)
    
    def test_create_user_sends_welcome_email(self, user_service, mock_email_service):
        # Arrange
        data = UserCreate(email="new@example.com", password="secret123")
        
        # Act
        user_service.create_user(data)
        
        # Assert
        mock_email_service.send_welcome_email.assert_called_once()
```

### Integration Tests: Validation

Test input validation with Flask routes. No database hits (mock repository).

**Example:**

```python
import pytest
from src import create_app

@pytest.fixture
def client():
    app = create_app('testing')
    with app.test_client() as client:
        yield client

def test_create_user_rejects_invalid_email(client):
    response = client.post('/api/v1/users', json={
        "email": "invalid-email",
        "password": "secret123"
    })
    
    assert response.status_code == 422  # Validation error
    data = response.get_json()
    assert "email" in str(data)

def test_create_user_rejects_short_password(client):
    response = client.post('/api/v1/users', json={
        "email": "test@example.com",
        "password": "123"  # Too short
    })
    
    assert response.status_code == 422
```

### Integration Tests: Execution

Test full request/response cycle with test database.

**Example:**

```python
import pytest
from src import create_app, db
from src.models.user import User

@pytest.fixture
def app():
    app = create_app('testing')
    with app.app_context():
        db.create_all()
        yield app
        db.session.remove()
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

def test_create_user_and_verify_in_database(client, app):
    """
    Integration test: Full user creation flow
    
    Tests:
    1. POST request creates user
    2. User is saved to database
    3. Password is hashed
    4. Welcome email is sent
    """
    # Act
    response = client.post('/api/v1/users', json={
        "email": "integration@example.com",
        "password": "secret123"
    })
    
    # Assert HTTP response
    assert response.status_code == 201
    data = response.get_json()
    assert data["email"] == "integration@example.com"
    
    # Verify database state
    with app.app_context():
        user = db.session.get(User, data["id"])
        assert user is not None
        assert user.email == "integration@example.com"
        assert user.password != "secret123"  # Hashed
```

### Test Coverage Requirements

- **Minimum 80% coverage** for services and business logic
- Test both happy paths and error cases
- Test edge cases (empty inputs, null values, boundary conditions)
- Test side effects (emails sent, events emitted, database state)

---

## 5. Code Organization & File Structure

### File Line Cap: 100 Lines

**CRITICAL RULE**: No file should exceed **100 lines** of code.

If a file exceeds 100 lines:
1. Convert it into a **self-contained module**
2. Use **CamelCased module folder names** (e.g., `UserService/`, `PaymentProcessing/`)
3. Split into multiple focused files (each ≤ 100 lines)
4. Use **barrel exports** at `__init__.py`

**Example - Before (180 lines):**

```
src/services/user_service.py  # 180 lines - TOO LARGE
```

**Example - After (Modular):**

```
src/services/UserService/
├── __init__.py              # Barrel exports (10 lines)
├── create_user.py           # 80 lines
├── update_user.py           # 60 lines
├── delete_user.py           # 40 lines
└── helpers.py               # 50 lines
```

**Barrel Export Pattern (`__init__.py`):**

```python
"""
User service module.

Provides business logic for user management including
creation, updates, deletion, and authentication.
"""

from .create_user import create_user
from .update_user import update_user
from .delete_user import delete_user

__all__ = [
    'create_user',
    'update_user',
    'delete_user',
]
```

### Standard Flask Project Structure

```
project/
├── src/
│   ├── __init__.py                 # Flask app factory
│   ├── app.py                      # Create app entry point
│   ├── config/                     # Configuration management
│   │   ├── __init__.py
│   │   ├── settings.py             # Pydantic settings
│   │   └── database.py             # DB config
│   ├── blueprints/                 # Flask blueprints (THIN routes)
│   │   ├── __init__.py
│   │   ├── users.py
│   │   └── orders.py
│   ├── services/                   # Business logic (CORE)
│   │   ├── __init__.py
│   │   ├── UserService/            # Modular if > 100 lines
│   │   │   ├── __init__.py
│   │   │   ├── create_user.py
│   │   │   └── update_user.py
│   │   └── order_service.py
│   ├── models/                     # SQLAlchemy models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── order.py
│   ├── schemas/                    # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── order.py
│   ├── validators/                 # Input validation logic
│   │   ├── __init__.py
│   │   └── email_validator.py
│   ├── middlewares/                # Custom middlewares
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── logging.py
│   ├── utils/                      # Generic utilities
│   │   ├── __init__.py
│   │   └── date_helpers.py
│   ├── helpers/                    # Domain-specific helpers
│   │   ├── __init__.py
│   │   └── payment_helpers.py
│   ├── exceptions/                 # Custom exceptions
│   │   ├── __init__.py
│   │   └── user_exceptions.py
│   ├── types/                      # Type definitions
│   │   ├── __init__.py
│   │   └── common.py
│   └── extensions.py               # Flask extensions (db, migrate, etc.)
├── tests/                          # Test suite
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   ├── conftest.py
│   └── pytest.ini
├── migrations/                     # Flask-Migrate (Alembic) migrations
│   └── versions/
├── docs/                           # Documentation
│   ├── api_spec.md
│   └── database_schema.md
├── logs/                           # Application logs
├── scripts/                        # Utility scripts
│   ├── seed_database.py
│   └── run_migrations.py
├── .env.example                    # Environment variables template
├── .gitignore
├── docker-compose.yml
├── Dockerfile
├── requirements.txt                # Dependencies
├── run.py                          # Application entry point
└── README.md
```

### Module Organization Rules

1. **Group by feature/domain**, not by layer (if project is large)
2. Each module is self-contained
3. Use `__init__.py` for barrel exports
4. Keep related code together

**Example - Feature-based structure (large projects):**

```
src/
├── users/                  # User domain module
│   ├── __init__.py
│   ├── routes.py          # Blueprint
│   ├── service.py
│   ├── models.py
│   ├── schemas.py
│   └── exceptions.py
└── orders/                 # Order domain module
    ├── __init__.py
    ├── routes.py
    ├── service.py
    ├── models.py
    └── schemas.py
```

---

## 6. Flask-Specific Patterns

### Application Factory Pattern (MANDATORY)

Always use the application factory pattern for testability and flexibility.

**Example:**

```python
# src/__init__.py
from flask import Flask
from src.extensions import db, migrate, cors, limiter, jwt, ma
from src.config.settings import get_settings

def create_app(config_name: str = 'development') -> Flask:
    """
    Flask application factory.
    
    Args:
        config_name: Configuration environment ('development', 'production', 'testing')
        
    Returns:
        Flask: Configured Flask application
    """
    app = Flask(__name__)
    
    # Load configuration
    settings = get_settings(config_name)
    app.config.from_object(settings)
    
    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    cors.init_app(app, origins=settings.CORS_ORIGINS)
    limiter.init_app(app)
    jwt.init_app(app)
    ma.init_app(app)
    
    # Register blueprints
    from src.blueprints.users import users_bp
    from src.blueprints.orders import orders_bp
    
    app.register_blueprint(users_bp)
    app.register_blueprint(orders_bp)
    
    # Register error handlers
    register_error_handlers(app)
    
    # Register CLI commands (optional)
    register_cli_commands(app)
    
    return app

def register_error_handlers(app: Flask) -> None:
    """Register global error handlers"""
    from src.exceptions.base import AppException
    
    @app.errorhandler(AppException)
    def handle_app_exception(error):
        return {
            "success": False,
            "message": error.message,
            "status": error.status_code
        }, error.status_code
    
    @app.errorhandler(404)
    def handle_not_found(error):
        return {
            "success": False,
            "message": "Resource not found",
            "status": 404
        }, 404
    
    @app.errorhandler(500)
    def handle_internal_error(error):
        return {
            "success": False,
            "message": "Internal server error",
            "status": 500
        }, 500

def register_cli_commands(app: Flask) -> None:
    """Register custom CLI commands"""
    @app.cli.command()
    def seed_db():
        """Seed the database with sample data"""
        from scripts.seed_database import seed
        seed()
        print("Database seeded successfully!")
```

**Application entry point:**

```python
# run.py
from src import create_app
from src.config.settings import settings

app = create_app(settings.ENVIRONMENT)

if __name__ == '__main__':
    app.run(
        host='0.0.0.0',
        port=5000,
        debug=settings.DEBUG
    )
```

### Flask Extensions Pattern

Centralize extension initialization in a separate file.

```python
# src/extensions.py
"""
Flask extensions.

Initialize all Flask extensions here and import them in app factory.
"""

from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_cors import CORS
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
from flask_jwt_extended import JWTManager
from flask_marshmallow import Marshmallow

# Database
db = SQLAlchemy()
migrate = Migrate()

# CORS
cors = CORS()

# Rate limiting
limiter = Limiter(
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"],
    storage_uri="redis://localhost:6379"
)

# JWT authentication
jwt = JWTManager()

# Marshmallow (optional - if not using Pydantic everywhere)
ma = Marshmallow()
```

### Blueprints (THIN Layer)

Blueprints handle ONLY HTTP concerns. Delegate all business logic to services.

**What blueprints SHOULD do:**
- Parse request data
- Call service methods
- Map responses to HTTP format
- Handle HTTP errors

**What blueprints SHOULD NOT do:**
- Business logic
- Database queries
- Complex validation
- Email sending

**Example:**

```python
# src/blueprints/users.py
from flask import Blueprint, request, jsonify, g
from http import HTTPStatus
from pydantic import ValidationError
from src.schemas.user import UserCreate, UserResponse
from src.services.user_service import UserService
from src.extensions import db
from src.exceptions import DuplicateEmailError, UserNotFoundError

users_bp = Blueprint('users', __name__, url_prefix='/api/v1/users')

# Dependency injection helper
def get_user_service() -> UserService:
    """Get UserService instance with dependencies"""
    if 'user_service' not in g:
        from src.services.email_service import EmailService
        g.user_service = UserService(
            db=db.session,
            email_service=EmailService()
        )
    return g.user_service

@users_bp.route('/', methods=['POST'])
def create_user():
    """
    Create a new user.
    
    Request body:
        {
            "email": "user@example.com",
            "password": "secret123"
        }
    
    Returns:
        201: User created successfully
        422: Validation error
        409: Email already exists
    """
    try:
        # Validate request data with Pydantic
        data = UserCreate(**request.get_json())
    except ValidationError as e:
        return jsonify({
            "success": False,
            "message": "Validation failed",
            "errors": e.errors()
        }), HTTPStatus.UNPROCESSABLE_ENTITY
    
    try:
        # THIN: Only call service and map response
        user_service = get_user_service()
        user = user_service.create_user(data)
        
        # Map to response schema
        response = UserResponse.model_validate(user)
        
        return jsonify({
            "success": True,
            "message": "User created successfully",
            "data": response.model_dump()
        }), HTTPStatus.CREATED
        
    except DuplicateEmailError as e:
        return jsonify({
            "success": False,
            "message": str(e)
        }), HTTPStatus.CONFLICT

@users_bp.route('/', methods=['GET'])
def get_users():
    """
    Get list of users with pagination.
    
    Query params:
        - limit: Maximum number of users (default: 10)
        - offset: Number of users to skip (default: 0)
    
    Returns:
        200: List of users
    """
    # Parse query parameters
    limit = request.args.get('limit', 10, type=int)
    offset = request.args.get('offset', 0, type=int)
    
    # Call service
    user_service = get_user_service()
    users = user_service.get_all_users(limit, offset)
    
    # Map to response schemas
    users_data = [UserResponse.model_validate(u).model_dump() for u in users]
    
    return jsonify({
        "success": True,
        "data": users_data,
        "pagination": {
            "limit": limit,
            "offset": offset,
            "total": user_service.count_users()
        }
    }), HTTPStatus.OK

@users_bp.route('/<int:user_id>', methods=['GET'])
def get_user(user_id: int):
    """Get user by ID"""
    try:
        user_service = get_user_service()
        user = user_service.get_by_id(user_id)
        
        if not user:
            raise UserNotFoundError(user_id)
        
        response = UserResponse.model_validate(user)
        return jsonify({
            "success": True,
            "data": response.model_dump()
        }), HTTPStatus.OK
        
    except UserNotFoundError as e:
        return jsonify({
            "success": False,
            "message": str(e)
        }), HTTPStatus.NOT_FOUND

@users_bp.route('/<int:user_id>', methods=['DELETE'])
def delete_user(user_id: int):
    """Delete user by ID"""
    user_service = get_user_service()
    user_service.delete_user(user_id)
    
    return '', HTTPStatus.NO_CONTENT

# Teardown: Close database session
@users_bp.teardown_app_request
def shutdown_session(exception=None):
    """Remove database session after each request"""
    db.session.remove()
```

### Alternative: Flask-Pydantic for Automatic Validation

For cleaner validation, use **Flask-Pydantic**:

```python
from flask_pydantic import validate

@users_bp.route('/', methods=['POST'])
@validate()
def create_user(body: UserCreate):
    """
    Create user with automatic validation.
    
    Flask-Pydantic handles validation automatically.
    """
    user_service = get_user_service()
    user = user_service.create_user(body)
    
    response = UserResponse.model_validate(user)
    return jsonify({
        "success": True,
        "data": response.model_dump()
    }), HTTPStatus.CREATED
```

### Services (Business Logic - CORE)

Services contain all business logic. Framework-agnostic and reusable.

**What services SHOULD do:**
- Business logic
- Data validation (business rules)
- Database operations
- Side effects (emails, events, cache invalidation)
- Transaction management

**What services SHOULD NOT do:**
- HTTP concerns (status codes, headers)
- Request parsing
- Response formatting

**Example:**

```python
# src/services/user_service.py
from sqlalchemy.orm import Session
from sqlalchemy import select
from typing import List
from src.models.user import User
from src.schemas.user import UserCreate
from src.exceptions import DuplicateEmailError
from src.utils.password import hash_password
from src.services.email_service import EmailService
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class UserService:
    """
    User service containing all user-related business logic.
    
    This service is framework-agnostic and can be used from
    Flask blueprints, CLI commands, background tasks, etc.
    """
    
    def __init__(self, db: Session, email_service: EmailService):
        self.db = db
        self.email_service = email_service
    
    def create_user(self, data: UserCreate) -> User:
        """
        Creates a new user with hashed password.
        
        Business rules:
        - Email must be unique
        - Password must be hashed before storage
        - Welcome email must be sent
        
        Args:
            data: User creation data
            
        Returns:
            User: Created user entity
            
        Raises:
            DuplicateEmailError: If email already exists
        """
        logger.info(f"Creating user with email: {data.email}")
        
        # Check email uniqueness (business rule)
        existing = self._get_by_email(data.email)
        if existing:
            logger.warning(f"Duplicate email attempt: {data.email}")
            raise DuplicateEmailError(f"Email {data.email} already exists")
        
        # Hash password (security requirement)
        hashed_password = hash_password(data.password)
        
        # Create user
        user = User(
            email=data.email,
            password=hashed_password,
            is_active=True
        )
        
        self.db.add(user)
        self.db.commit()
        self.db.refresh(user)
        
        logger.info(f"User created successfully: {user.id}")
        
        # Side effect: Send welcome email
        self.email_service.send_welcome_email(user.email, user.id)
        
        return user
    
    def get_all_users(self, limit: int, offset: int) -> List[User]:
        """
        Retrieves paginated list of users.
        
        Args:
            limit: Maximum number of users to return
            offset: Number of users to skip
            
        Returns:
            List[User]: List of user entities
        """
        result = self.db.execute(
            select(User)
            .limit(limit)
            .offset(offset)
            .order_by(User.created_at.desc())
        )
        return list(result.scalars().all())
    
    def get_by_id(self, user_id: int) -> User | None:
        """Get user by ID"""
        return self.db.get(User, user_id)
    
    def count_users(self) -> int:
        """Count total users"""
        return self.db.query(User).count()
    
    def delete_user(self, user_id: int) -> None:
        """Delete user by ID"""
        user = self.get_by_id(user_id)
        if user:
            self.db.delete(user)
            self.db.commit()
            logger.info(f"User deleted: {user_id}")
    
    def _get_by_email(self, email: str) -> User | None:
        """Private helper to find user by email"""
        result = self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
```

### Dependency Injection

#### Option 1: Manual Dependency Injection (Simple)

```python
# src/blueprints/users.py
from flask import g

def get_user_service() -> UserService:
    """Get UserService instance with dependencies"""
    if 'user_service' not in g:
        from src.services.email_service import EmailService
        g.user_service = UserService(
            db=db.session,
            email_service=EmailService()
        )
    return g.user_service

@users_bp.route('/', methods=['POST'])
def create_user():
    user_service = get_user_service()
    # ...
```

#### Option 2: Flask-Injector (Recommended for Large Projects)

```python
# src/dependencies.py
from injector import Module, provider, singleton
from flask_sqlalchemy import SQLAlchemy
from src.services.user_service import UserService
from src.services.email_service import EmailService

class AppModule(Module):
    """Dependency injection configuration"""
    
    @provider
    @singleton
    def provide_email_service(self) -> EmailService:
        return EmailService()
    
    @provider
    def provide_user_service(
        self,
        db: SQLAlchemy,
        email_service: EmailService
    ) -> UserService:
        return UserService(db=db.session, email_service=email_service)

# In app factory
from flask_injector import FlaskInjector
from src.dependencies import AppModule

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    # ... setup app ...
    
    # Initialize dependency injection
    FlaskInjector(app=app, modules=[AppModule])
    
    return app

# In blueprints
from injector import inject

@users_bp.route('/', methods=['POST'])
@inject
def create_user(user_service: UserService):
    """Dependencies injected automatically"""
    # user_service is injected
    user = user_service.create_user(data)
    # ...
```

### Database Session Management

```python
# src/extensions.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

# In blueprints (automatic session management)
@users_bp.teardown_app_request
def shutdown_session(exception=None):
    """Remove database session after each request"""
    db.session.remove()

# Manual transaction management (when needed)
from src.extensions import db

def complex_operation():
    try:
        # Multiple operations in one transaction
        user = User(...)
        db.session.add(user)
        
        profile = Profile(user_id=user.id)
        db.session.add(profile)
        
        db.session.commit()
    except Exception as e:
        db.session.rollback()
        raise
```

### Exception Handling (Centralized)

Create custom exceptions and handle them globally.

**Custom exceptions:**

```python
# src/exceptions/base.py
class AppException(Exception):
    """Base exception for all application errors"""
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)

# src/exceptions/user_exceptions.py
class DuplicateEmailError(AppException):
    """Raised when email already exists"""
    def __init__(self, email: str):
        super().__init__(
            message=f"Email {email} already exists",
            status_code=409
        )

class UserNotFoundError(AppException):
    """Raised when user is not found"""
    def __init__(self, user_id: int):
        super().__init__(
            message=f"User with ID {user_id} not found",
            status_code=404
        )
```

**Global exception handlers (in app factory):**

```python
# In create_app()
def register_error_handlers(app: Flask) -> None:
    """Register global error handlers"""
    from src.exceptions.base import AppException
    from pydantic import ValidationError
    
    @app.errorhandler(AppException)
    def handle_app_exception(error: AppException):
        """Handle all custom application exceptions"""
        return jsonify({
            "success": False,
            "message": error.message,
            "status": error.status_code
        }), error.status_code
    
    @app.errorhandler(ValidationError)
    def handle_validation_error(error: ValidationError):
        """Handle Pydantic validation errors"""
        return jsonify({
            "success": False,
            "message": "Validation failed",
            "errors": error.errors(),
            "status": 422
        }), 422
    
    @app.errorhandler(404)
    def handle_not_found(error):
        return jsonify({
            "success": False,
            "message": "Resource not found",
            "status": 404
        }), 404
    
    @app.errorhandler(500)
    def handle_internal_error(error):
        app.logger.error(f"Internal error: {error}")
        return jsonify({
            "success": False,
            "message": "Internal server error",
            "status": 500
        }), 500
```

---

## 7. RESTful API Design Standards

### REST Principles (MANDATORY)

1. **Stateless**: Server does NOT store client state between requests
2. **Resource-Oriented**: Operate on resources (users, orders, products)
3. **Uniform Interface**: Consistent structure following HTTP standards
4. **Layered System**: Separation of concerns (business logic, database, client)
5. **Cacheable**: Mark responses as cacheable or non-cacheable

### HTTP Methods (Use Correctly)

- **GET**: Retrieve data (idempotent, safe)
- **POST**: Create new data (non-idempotent)
- **PUT**: Update entire resource (idempotent)
- **PATCH**: Update partial resource (idempotent)
- **DELETE**: Remove data (idempotent)

### Resource Naming Conventions

**RULES:**
- Use **nouns**, NOT verbs (e.g., `/users`, not `/getUsers`)
- Use **plural names** for collections (e.g., `/products`, `/users`)
- Use **lowercase with hyphens** for multi-word resources (e.g., `/order-items`)
- Use **nesting** for relationships (limit to 2 levels)

**Examples:**

```
✅ GOOD:
GET    /api/v1/users              # List users
GET    /api/v1/users/123          # Get specific user
POST   /api/v1/users              # Create user
PUT    /api/v1/users/123          # Update entire user
PATCH  /api/v1/users/123          # Update partial user
DELETE /api/v1/users/123          # Delete user

GET    /api/v1/users/123/orders   # Get user's orders (nested)
GET    /api/v1/orders?user_id=123 # Alternative: filter by query param

❌ BAD:
GET    /api/v1/getUsers           # Don't use verbs
POST   /api/v1/user               # Use plural
GET    /api/v1/Users              # Don't capitalize
GET    /api/v1/user_list          # Don't use underscores
```

### HTTP Status Codes (Use Correctly)

**Success (2xx):**
- `200 OK` - Request successful (GET, PUT, PATCH, DELETE)
- `201 Created` - New resource created (POST)
- `204 No Content` - Success with no response body (DELETE)

**Client Errors (4xx):**
- `400 Bad Request` - Invalid request data
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource not found
- `409 Conflict` - Resource conflict (e.g., duplicate email)
- `422 Unprocessable Entity` - Validation failed

**Server Errors (5xx):**
- `500 Internal Server Error` - Unexpected server error
- `503 Service Unavailable` - Server temporarily unavailable

**Example usage:**

```python
from http import HTTPStatus

@users_bp.route('/', methods=['POST'])
def create_user():
    # Returns 201 on success
    return jsonify(data), HTTPStatus.CREATED

@users_bp.route('/<int:user_id>', methods=['DELETE'])
def delete_user(user_id: int):
    # Returns 204 on success (no body)
    user_service.delete_user(user_id)
    return '', HTTPStatus.NO_CONTENT
```

### Pagination (REQUIRED for Lists)

Use **query parameters** for pagination:

```python
@users_bp.route('/', methods=['GET'])
def get_users():
    """
    Get paginated list of users.
    
    Query params:
    - page: Page number (default: 1)
    - limit: Items per page (default: 20, max: 100)
    """
    page = request.args.get('page', 1, type=int)
    limit = request.args.get('limit', 20, type=int)
    limit = min(limit, 100)  # Cap at 100
    
    offset = (page - 1) * limit
    
    user_service = get_user_service()
    users = user_service.get_all_users(limit, offset)
    total = user_service.count_users()
    
    return jsonify({
        "success": True,
        "data": [UserResponse.model_validate(u).model_dump() for u in users],
        "pagination": {
            "page": page,
            "limit": limit,
            "total": total,
            "pages": (total + limit - 1) // limit
        }
    }), HTTPStatus.OK
```

### API Versioning (MANDATORY)

Use **URL versioning**:

```python
# ✅ GOOD: Version in URL
users_bp = Blueprint('users', __name__, url_prefix='/api/v1/users')

# ❌ BAD: No versioning
users_bp = Blueprint('users', __name__, url_prefix='/users')
```

**Version upgrade strategy:**
- `/api/v1/users` - Current stable version
- `/api/v2/users` - New version (breaking changes)
- Deprecate old versions with notice period (e.g., 6 months)

### Consistent Error Response Format

**Standard error response:**

```python
{
    "success": false,
    "message": "User with email test@example.com already exists",
    "status": 409,
    "errors": [  # Optional: For validation errors
        {
            "field": "email",
            "message": "Email already exists"
        }
    ]
}
```

**Success response:**

```python
{
    "success": true,
    "message": "User created successfully",
    "status": 201,
    "data": {
        "id": 123,
        "email": "test@example.com"
    }
}
```

### Use Slugs for Human-Readable URLs (Recommended)

```python
# ✅ GOOD: Slug-based URL
GET /api/v1/articles/how-to-learn-python

# ❌ ACCEPTABLE but less user-friendly
GET /api/v1/articles/12345
```

**Implementation:**

```python
from sqlalchemy import Column, String, Integer
from slugify import slugify
from src.extensions import db

class Article(db.Model):
    __tablename__ = "articles"
    
    id = Column(Integer, primary_key=True)
    title = Column(String(255), nullable=False)
    slug = Column(String(255), unique=True, index=True, nullable=False)
    
    def __init__(self, title: str, **kwargs):
        super().__init__(**kwargs)
        self.title = title
        self.slug = slugify(title)

# Route
@articles_bp.route('/<slug>', methods=['GET'])
def get_article(slug: str):
    article = Article.query.filter_by(slug=slug).first_or_404()
    return jsonify(ArticleResponse.model_validate(article).model_dump())
```

---

## 8. Database & ORM Standards

### ORM Stack: Pydantic + SQLAlchemy + Flask-Migrate

**Required packages:**
- `pydantic` (v2+) - Data validation and serialization
- `sqlalchemy` (2.0+) - ORM
- `flask-sqlalchemy` - Flask integration for SQLAlchemy
- `flask-migrate` - Database migrations (Alembic wrapper)
- `psycopg2-binary` - PostgreSQL driver (or `pymysql` for MySQL)

### Database Configuration

```python
# src/config/database.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

db = SQLAlchemy()
migrate = Migrate()

# In app factory (src/__init__.py)
from src.config.database import db, migrate

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    
    # Database configuration
    app.config['SQLALCHEMY_DATABASE_URI'] = settings.DATABASE_URL
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    app.config['SQLALCHEMY_ECHO'] = settings.DEBUG  # Log SQL queries in debug
    
    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    
    return app
```

### Separation of Models and Schemas (CRITICAL)

**NEVER mix SQLAlchemy models with Pydantic schemas.**

**SQLAlchemy Models** (Database layer):

```python
# src/models/user.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from datetime import datetime
from src.extensions import db

class User(db.Model):
    """SQLAlchemy User model - represents database table"""
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False, index=True)
    password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # Relationships
    orders = db.relationship('Order', back_populates='user', lazy='dynamic')
    
    def __repr__(self):
        return f'<User {self.email}>'
```

**Pydantic Schemas** (Validation/Serialization layer):

```python
# src/schemas/user.py
from pydantic import BaseModel, EmailStr, Field, ConfigDict
from datetime import datetime

class UserBase(BaseModel):
    """Base schema with common fields"""
    email: EmailStr

class UserCreate(UserBase):
    """Schema for user creation (input)"""
    password: str = Field(..., min_length=8)

class UserUpdate(BaseModel):
    """Schema for user updates (partial)"""
    email: EmailStr | None = None
    password: str | None = Field(None, min_length=8)

class UserResponse(UserBase):
    """Schema for user response (output)"""
    id: int
    is_active: bool
    created_at: datetime
    
    model_config = ConfigDict(from_attributes=True)  # Enable ORM mode
```

### SQLAlchemy Queries (Best Practices)

**Use SQLAlchemy 2.0 select() style (preferred):**

```python
from sqlalchemy import select
from src.models.user import User
from src.extensions import db

# ✅ GOOD: SQLAlchemy 2.0 style
def get_active_users() -> list[User]:
    result = db.session.execute(
        select(User)
        .where(User.is_active == True)
        .order_by(User.created_at.desc())
    )
    return list(result.scalars().all())

# ✅ ACCEPTABLE: Flask-SQLAlchemy query interface
def get_active_users_legacy() -> list[User]:
    return User.query.filter_by(is_active=True).order_by(User.created_at.desc()).all()
```

**Complex queries with joins:**

```python
from sqlalchemy.orm import selectinload

def get_user_with_orders(user_id: int) -> User | None:
    result = db.session.execute(
        select(User)
        .options(selectinload(User.orders))
        .where(User.id == user_id)
    )
    return result.scalar_one_or_none()
```

### Database Migrations with Flask-Migrate

**Setup Flask-Migrate:**

```bash
# Initialize migrations
flask db init

# Create migration
flask db migrate -m "Create users table"

# Apply migrations
flask db upgrade

# Rollback migration
flask db downgrade
```

**Migration best practices:**
- ALWAYS review auto-generated migrations before applying
- Use descriptive migration messages
- Never edit applied migrations (create new ones)
- Keep migrations small and focused
- Test migrations on staging before production

### Database Constraints & Indexes

Add constraints for data integrity:

```python
from sqlalchemy import Column, Integer, String, CheckConstraint, Index
from src.extensions import db

class User(db.Model):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False, index=True)
    age = Column(Integer, nullable=False)
    
    # Table-level constraints
    __table_args__ = (
        CheckConstraint('age >= 18', name='check_age_minimum'),
        Index('idx_email_active', 'email', 'is_active'),
    )
```

### Relationships (One-to-Many, Many-to-Many)

**One-to-Many:**

```python
# models/user.py
class User(db.Model):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    orders = db.relationship('Order', back_populates='user', lazy='dynamic')

# models/order.py
class Order(db.Model):
    __tablename__ = "orders"
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, db.ForeignKey('users.id'), nullable=False)
    user = db.relationship('User', back_populates='orders')
```

**Many-to-Many:**

```python
# Association table
user_roles = db.Table('user_roles',
    Column('user_id', Integer, db.ForeignKey('users.id'), primary_key=True),
    Column('role_id', Integer, db.ForeignKey('roles.id'), primary_key=True)
)

class User(db.Model):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    roles = db.relationship('Role', secondary=user_roles, back_populates='users')

class Role(db.Model):
    __tablename__ = "roles"
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)
    users = db.relationship('User', secondary=user_roles, back_populates='roles')
```

---

## 9. Configuration Management

### Use Pydantic Settings (MANDATORY)

**Config structure:**

```python
# src/config/settings.py
from pydantic_settings import BaseSettings
from pydantic import Field
from typing import List, cast
from pathlib import Path
from functools import lru_cache
import os

BASE_DIR = Path(__file__).resolve().parent.parent.parent

class BaseConfig(BaseSettings):
    """Base configuration with common settings"""
    
    # App metadata
    APP_NAME: str = Field("My Flask API")
    APP_VERSION: str = Field("1.0.0")
    DEBUG: bool = Field(False)
    
    # Database
    DATABASE_URL: str = Field(..., env="DATABASE_URL")
    
    # Security
    SECRET_KEY: str = Field(..., env="SECRET_KEY")
    JWT_SECRET_KEY: str = Field(..., env="JWT_SECRET_KEY")
    JWT_ALGORITHM: str = Field("HS256")
    JWT_ACCESS_TOKEN_EXPIRES: int = Field(3600)  # 1 hour in seconds
    
    # CORS
    CORS_ORIGINS: List[str] = Field(default_factory=list)
    
    # Redis
    REDIS_URL: str = Field("redis://localhost:6379/0")
    
    # Email
    MAIL_SERVER: str = Field("localhost", env="MAIL_SERVER")
    MAIL_PORT: int = Field(587, env="MAIL_PORT")
    MAIL_USERNAME: str | None = Field(None, env="MAIL_USERNAME")
    MAIL_PASSWORD: str | None = Field(None, env="MAIL_PASSWORD")
    MAIL_USE_TLS: bool = Field(True)
    
    # File upload
    UPLOAD_FOLDER: Path = Field(default=BASE_DIR / "uploads")
    MAX_CONTENT_LENGTH: int = Field(16 * 1024 * 1024)  # 16MB
    
    model_config = {
        "env_file": ".env",
        "env_file_encoding": "utf-8",
        "case_sensitive": True
    }

class DevelopmentConfig(BaseConfig):
    """Development environment configuration"""
    DEBUG: bool = True
    CORS_ORIGINS: List[str] = Field(default_factory=lambda: ["*"])
    SECRET_KEY: str = Field("dev-secret-key-change-in-production")
    JWT_SECRET_KEY: str = Field("dev-jwt-secret")

class ProductionConfig(BaseConfig):
    """Production environment configuration"""
    DEBUG: bool = False
    CORS_ORIGINS: List[str] = Field(..., env="CORS_ORIGINS")
    SECRET_KEY: str = Field(..., env="SECRET_KEY")
    JWT_SECRET_KEY: str = Field(..., env="JWT_SECRET_KEY")

class TestingConfig(BaseConfig):
    """Testing environment configuration"""
    TESTING: bool = True
    DEBUG: bool = True
    DATABASE_URL: str = Field("sqlite:///:memory:")
    SECRET_KEY: str = Field("test-secret-key")
    JWT_SECRET_KEY: str = Field("test-jwt-secret")

@lru_cache()
def get_settings(env: str | None = None) -> BaseConfig:
    """
    Get environment-specific settings.
    
    Cached to avoid re-reading .env file on every call.
    
    Args:
        env: Environment name ('development', 'production', 'testing')
             If None, reads from FLASK_ENV or PYTHON_ENV environment variable
    
    Returns:
        BaseConfig: Configuration instance
    """
    if env is None:
        env = os.getenv("FLASK_ENV") or os.getenv("PYTHON_ENV", "development")
    
    configs = {
        "development": DevelopmentConfig,
        "production": ProductionConfig,
        "testing": TestingConfig,
    }
    
    if env not in configs:
        raise ValueError(f"Invalid environment: {env}. Must be one of: {list(configs.keys())}")
    
    return configs[env]()

# Default settings instance
settings = get_settings()
```

**Using settings in Flask app:**

```python
# src/__init__.py
from src.config.settings import get_settings

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    
    # Load configuration from Pydantic settings
    settings = get_settings(config_name)
    
    # Convert Pydantic model to dict and load into Flask config
    app.config.update(settings.model_dump())
    
    # Or set individual values
    app.config['SQLALCHEMY_DATABASE_URI'] = settings.DATABASE_URL
    app.config['SECRET_KEY'] = settings.SECRET_KEY
    app.config['DEBUG'] = settings.DEBUG
    
    return app
```

### Environment Variables (.env)

**Example `.env` file:**

```env
FLASK_ENV=development
PYTHON_ENV=development

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# Security
SECRET_KEY=your-super-secret-key-change-in-production
JWT_SECRET_KEY=your-jwt-secret-key

# CORS (comma-separated for production)
CORS_ORIGINS=http://localhost:3000,https://example.com

# Redis
REDIS_URL=redis://localhost:6379/0

# Email
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
```

**CRITICAL**: Never commit `.env` to git. Use `.env.example` as template.

---

## 10. Feature Switches (Feature Flags)

### What Are Feature Switches?

Feature switches (flags/toggles) allow enabling/disabling functionality at runtime without deploying new code.

**Types:**
1. **Release toggles** - Control rollout of incomplete features
2. **Experiment toggles** - A/B testing and experiments
3. **Ops toggles** - Circuit breakers, kill switches
4. **Permission toggles** - Premium features, role-based access

### When to Use Feature Switches

**ALWAYS use for:**
- Features > 2 days development time
- Changes to payment, auth, or data integrity systems
- Features requiring staged rollout or A/B testing
- Experimental or potentially risky functionality
- Large refactorings or architectural changes

### Implementation Pattern

**Basic implementation:**

```python
# src/config/feature_flags.py
from enum import Enum
from functools import lru_cache
from typing import Optional

class FeatureFlag(str, Enum):
    NEW_CHECKOUT_FLOW = "feature.payments.new_checkout_flow"
    ADVANCED_SEARCH = "feature.search.advanced_search"
    BETA_DASHBOARD = "feature.ui.beta_dashboard"

class FeatureFlagService:
    """Service for managing feature flags"""
    
    def __init__(self):
        # In production, use LaunchDarkly, Split.io, Redis, or database
        self._flags = {
            FeatureFlag.NEW_CHECKOUT_FLOW: False,  # Default OFF
            FeatureFlag.ADVANCED_SEARCH: True,
            FeatureFlag.BETA_DASHBOARD: False,
        }
    
    def is_enabled(self, flag: FeatureFlag, user_id: Optional[int] = None) -> bool:
        """
        Check if feature flag is enabled.
        
        Args:
            flag: Feature flag to check
            user_id: Optional user ID for user-specific flags
            
        Returns:
            bool: True if enabled, False otherwise
        """
        # Basic implementation - can be enhanced with user-based logic
        return self._flags.get(flag, False)
    
    def enable(self, flag: FeatureFlag):
        """Enable a feature flag"""
        self._flags[flag] = True
    
    def disable(self, flag: FeatureFlag):
        """Disable a feature flag"""
        self._flags[flag] = False

@lru_cache()
def get_feature_flag_service() -> FeatureFlagService:
    return FeatureFlagService()
```

**Usage in code:**

```python
from src.config.feature_flags import FeatureFlag, get_feature_flag_service

def checkout(user_id: int):
    """Process checkout with feature flag"""
    flag_service = get_feature_flag_service()
    
    if flag_service.is_enabled(FeatureFlag.NEW_CHECKOUT_FLOW, user_id):
        return new_checkout_experience(user_id)
    else:
        return legacy_checkout_experience(user_id)
```

### Best Practices

1. **Naming convention**: `feature.<team>.<feature-name>`
2. **Default to OFF** for new features
3. **Document** in code or config file with owner and expiration date
4. **Test both paths** when feature flag exists
5. **Clean up** after rollout completes (avoid toggle debt)
6. **Log** toggle state changes for auditing

---

## 11. Security Standards

### Password Hashing (MANDATORY)

Use **bcrypt** or **argon2** for password hashing. NEVER store plain text passwords.

```python
# src/utils/password.py
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    """Hash a password using bcrypt"""
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verify a password against its hash"""
    return pwd_context.verify(plain_password, hashed_password)
```

### JWT Authentication with Flask-JWT-Extended

```python
# Install: pip install flask-jwt-extended

# src/extensions.py
from flask_jwt_extended import JWTManager

jwt = JWTManager()

# In app factory
from src.extensions import jwt
from src.config.settings import settings

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    
    # JWT configuration
    app.config['JWT_SECRET_KEY'] = settings.JWT_SECRET_KEY
    app.config['JWT_ACCESS_TOKEN_EXPIRES'] = settings.JWT_ACCESS_TOKEN_EXPIRES
    
    jwt.init_app(app)
    
    return app

# Create tokens
from flask_jwt_extended import create_access_token

@auth_bp.route('/login', methods=['POST'])
def login():
    data = LoginSchema(**request.get_json())
    
    # Verify credentials
    user = user_service.authenticate(data.email, data.password)
    
    if not user:
        return jsonify({"message": "Invalid credentials"}), 401
    
    # Create JWT token
    access_token = create_access_token(identity=user.id)
    
    return jsonify({
        "access_token": access_token,
        "user": UserResponse.model_validate(user).model_dump()
    }), 200

# Protected routes
from flask_jwt_extended import jwt_required, get_jwt_identity

@users_bp.route('/me', methods=['GET'])
@jwt_required()
def get_current_user():
    """Get current authenticated user"""
    current_user_id = get_jwt_identity()
    user = user_service.get_by_id(current_user_id)
    
    return jsonify({
        "data": UserResponse.model_validate(user).model_dump()
    }), 200
```

### CORS Configuration

```python
# Install: pip install flask-cors

# src/extensions.py
from flask_cors import CORS

cors = CORS()

# In app factory
from src.extensions import cors
from src.config.settings import settings

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    
    # CORS configuration
    cors.init_app(
        app,
        origins=settings.CORS_ORIGINS,
        supports_credentials=True,
        allow_headers=["Content-Type", "Authorization"],
        methods=["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"]
    )
    
    return app
```

### Input Validation (Automatic with Pydantic)

Pydantic handles input validation automatically when used in blueprints.

```python
from pydantic import BaseModel, EmailStr, Field, validator

class UserCreate(BaseModel):
    email: EmailStr  # Validates email format
    password: str = Field(..., min_length=8)  # Minimum length
    age: int = Field(..., ge=18, le=120)  # Range validation
    
    @validator('password')
    def validate_password_strength(cls, v):
        """Custom password validation"""
        if not any(char.isdigit() for char in v):
            raise ValueError('Password must contain at least one digit')
        if not any(char.isupper() for char in v):
            raise ValueError('Password must contain at least one uppercase letter')
        return v

# In blueprint
@users_bp.route('/', methods=['POST'])
def create_user():
    try:
        data = UserCreate(**request.get_json())
    except ValidationError as e:
        return jsonify({"errors": e.errors()}), 422
    # ...
```

### SQL Injection Protection (Automatic with SQLAlchemy)

SQLAlchemy parameterizes queries automatically. NEVER use string formatting for queries.

```python
# ✅ GOOD: Parameterized query (safe)
user = User.query.filter_by(email=email).first()

# ✅ GOOD: SQLAlchemy 2.0 style
result = db.session.execute(
    select(User).where(User.email == email)
)

# ❌ BAD: String formatting (SQL injection risk)
query = f"SELECT * FROM users WHERE email = '{email}'"  # NEVER DO THIS
db.session.execute(query)
```

### Environment Variables for Secrets

NEVER hardcode secrets in code. Use environment variables.

```python
# ❌ BAD: Hardcoded secret
SECRET_KEY = "my-secret-key"

# ✅ GOOD: From environment
SECRET_KEY = os.getenv("SECRET_KEY")
```

### HTTPS Redirect in Production

```python
from flask_talisman import Talisman

# In app factory (production only)
if not app.config['DEBUG']:
    Talisman(app, force_https=True)
```

---

## 12. Logging Standards

### Use Python's logging Module

```python
# src/utils/logger.py
import logging
from src.config.settings import settings

def setup_logger(name: str) -> logging.Logger:
    """
    Setup logger with appropriate level and format.
    
    Args:
        name: Logger name (typically __name__)
        
    Returns:
        logging.Logger: Configured logger
    """
    logger = logging.getLogger(name)
    
    # Set level based on environment
    level = logging.DEBUG if settings.DEBUG else logging.INFO
    logger.setLevel(level)
    
    # Avoid duplicate handlers
    if logger.handlers:
        return logger
    
    # Create handler
    handler = logging.StreamHandler()
    handler.setLevel(level)
    
    # Create formatter
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    handler.setFormatter(formatter)
    
    logger.addHandler(handler)
    
    return logger
```

### Usage in Services

```python
# src/services/user_service.py
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

class UserService:
    def create_user(self, data: UserCreate) -> User:
        logger.info(f"Creating user with email: {data.email}")
        
        try:
            # Business logic
            user = User(**data.model_dump())
            db.session.add(user)
            db.session.commit()
            
            logger.info(f"User created successfully: {user.id}")
            return user
            
        except Exception as e:
            logger.error(f"Failed to create user: {str(e)}", exc_info=True)
            db.session.rollback()
            raise
```

### Flask Request Logging Middleware

```python
# src/middlewares/logging.py
from flask import request, g
import time
from src.utils.logger import setup_logger

logger = setup_logger(__name__)

def setup_request_logging(app):
    """Setup request/response logging middleware"""
    
    @app.before_request
    def log_request():
        """Log incoming request"""
        g.start_time = time.time()
        logger.info(f"{request.method} {request.path} - Started")
    
    @app.after_request
    def log_response(response):
        """Log outgoing response"""
        if hasattr(g, 'start_time'):
            duration = time.time() - g.start_time
            logger.info(
                f"{request.method} {request.path} - "
                f"Status: {response.status_code} - "
                f"Duration: {duration:.3f}s"
            )
        return response

# In app factory
from src.middlewares.logging import setup_request_logging

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    # ...
    setup_request_logging(app)
    return app
```

### What to Log

**DO log:**
- API requests/responses (INFO)
- Errors with stack traces (ERROR)
- Business-critical events (INFO)
- Security events (WARNING/ERROR)
- Performance bottlenecks (DEBUG/INFO)

**DON'T log:**
- Passwords, tokens, API keys
- PII (Personal Identifiable Information) unless necessary and encrypted
- Sensitive business data

---

## 13. Caching Strategies

### Use Redis for Distributed Caching

```python
# Install: pip install redis flask-caching

# src/extensions.py
from flask_caching import Cache

cache = Cache()

# In app factory
from src.extensions import cache
from src.config.settings import settings

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    
    # Cache configuration
    app.config['CACHE_TYPE'] = 'redis'
    app.config['CACHE_REDIS_URL'] = settings.REDIS_URL
    app.config['CACHE_DEFAULT_TIMEOUT'] = 300  # 5 minutes
    
    cache.init_app(app)
    
    return app
```

### Caching Routes

```python
from src.extensions import cache

@users_bp.route('/<int:user_id>', methods=['GET'])
@cache.cached(timeout=300, key_prefix='user_%s')
def get_user(user_id: int):
    """Get user by ID with 5-minute cache"""
    user = user_service.get_by_id(user_id)
    
    if not user:
        return jsonify({"message": "User not found"}), 404
    
    return jsonify({
        "data": UserResponse.model_validate(user).model_dump()
    }), 200
```

### Caching Service Methods

```python
from src.extensions import cache

class UserService:
    @cache.memoize(timeout=300)
    def get_by_id(self, user_id: int) -> User | None:
        """Get user by ID with caching"""
        return db.session.get(User, user_id)
    
    def update_user(self, user_id: int, data: UserUpdate) -> User:
        """Update user and invalidate cache"""
        user = self.get_by_id(user_id)
        
        for key, value in data.model_dump(exclude_unset=True).items():
            setattr(user, key, value)
        
        db.session.commit()
        
        # Invalidate cache
        cache.delete_memoized(self.get_by_id, user_id)
        
        return user
```

### What to Cache

**DO cache:**
- Expensive database queries
- External API calls
- Computed results (reports, analytics)
- Static content

**DON'T cache:**
- User-specific data (unless keyed properly)
- Real-time data
- Small, fast queries
- Frequently changing data

---

## 14. Rate Limiting

### Use Flask-Limiter

```python
# Install: pip install flask-limiter

# src/extensions.py
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"],
    storage_uri="redis://localhost:6379"
)

# In app factory
from src.extensions import limiter

def create_app(config_name: str = 'development') -> Flask:
    app = Flask(__name__)
    
    limiter.init_app(app)
    
    return app
```

### Apply Rate Limits to Routes

```python
from src.extensions import limiter

@users_bp.route('/', methods=['POST'])
@limiter.limit("5/minute")
def create_user():
    """Create user - limited to 5 requests per minute"""
    # ...

@users_bp.route('/', methods=['GET'])
@limiter.limit("100/hour")
def get_users():
    """Get users - limited to 100 requests per hour"""
    # ...
```

### Custom Rate Limiting by User

```python
from flask import request
from flask_jwt_extended import get_jwt_identity, verify_jwt_in_request

def get_user_id_for_rate_limit():
    """Get user ID from JWT for rate limiting"""
    try:
        verify_jwt_in_request(optional=True)
        user_id = get_jwt_identity()
        if user_id:
            return f"user:{user_id}"
    except:
        pass
    return get_remote_address()

limiter = Limiter(key_func=get_user_id_for_rate_limit)

@orders_bp.route('/', methods=['POST'])
@limiter.limit("10/minute")  # 10 orders per minute per user
def create_order():
    """Create order with per-user rate limiting"""
    # ...
```

---

## 15. CI/CD & Deployment

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-flask pytest-cov ruff mypy
      
      - name: Lint with Ruff
        run: ruff check src/
      
      - name: Type check with mypy
        run: mypy src/
      
      - name: Run tests with coverage
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379/0
          SECRET_KEY: test-secret-key
          JWT_SECRET_KEY: test-jwt-secret
        run: pytest --cov=src --cov-report=xml tests/
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
  
  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t myapp:latest .
      
      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push myapp:latest
```

### Pre-commit Hooks (Recommended)

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.0
    hooks:
      - id: ruff
        args: [--fix]
  
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]
  
  - repo: https://github.com/psf/black
    rev: 23.11.0
    hooks:
      - id: black
```

---

## 16. Docker & Containerization

### Multi-stage Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim as builder

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Runtime stage
FROM python:3.11-slim

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /root/.local

# Copy application code
COPY src/ ./src/
COPY run.py .

# Create non-root user
RUN useradd -m flaskuser && chown -R flaskuser:flaskuser /app
USER flaskuser

# Add local bin to PATH
ENV PATH=/root/.local/bin:$PATH

# Expose port
EXPOSE 5000

# Run application
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "run:app"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
      - REDIS_URL=redis://redis:6379/0
      - SECRET_KEY=dev-secret-key
      - JWT_SECRET_KEY=dev-jwt-secret
    depends_on:
      - db
      - redis
    volumes:
      - ./src:/app/src  # Hot reload in development
  
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: dbname
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### .dockerignore

```
# .dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.venv/
.git/
.gitignore
.pytest_cache/
.mypy_cache/
.ruff_cache/
*.log
.env
.env.local
tests/
docs/
README.md
```

---

## 17. Things to NEVER Do

- ❌ Don't use plain dictionaries for structured data (use Pydantic models or dataclasses)
- ❌ Don't put business logic in blueprints (use services)
- ❌ Don't skip type hints (mandatory everywhere)
- ❌ Don't exceed 100 lines per file without modularizing
- ❌ Don't mix SQLAlchemy models with Pydantic schemas
- ❌ Don't use string formatting for SQL queries (SQL injection risk)
- ❌ Don't hardcode secrets in code (use environment variables)
- ❌ Don't store plain text passwords (always hash)
- ❌ Don't skip validation (use Pydantic)
- ❌ Don't use verbs in resource URLs (use nouns)
- ❌ Don't return 200 for all responses (use correct status codes)
- ❌ Don't skip pagination for list endpoints
- ❌ Don't version APIs in headers (use URL versioning)
- ❌ Don't skip database migrations (use Flask-Migrate)
- ❌ Don't log sensitive data (passwords, tokens, PII)
- ❌ Don't commit .env files to git
- ❌ Don't skip tests (minimum 80% coverage for services)
- ❌ Don't use global state excessively (use dependency injection)
- ❌ Don't optimize prematurely (simple working code > clever code)
- ❌ Don't forget to close database sessions (use teardown handlers)

---

## 18. When Implementing New Features

Follow this workflow:

1. **Define types first** (Pydantic schemas, SQLAlchemy models)
2. **Create database migration** (Flask-Migrate)
3. **Write failing tests** (TDD approach preferred)
4. **Implement service layer** (business logic)
5. **Add blueprint/route layer** (HTTP mapping)
6. **Add validation** (Pydantic handles automatically)
7. **Add integration tests** (validation + execution)
8. **Document with docstrings** (Google-style)
9. **Check file line count** (modularize if > 100 lines)
10. **Run linter and type checker** (ruff, mypy)
11. **Review and commit**

---

## 19. Agent Behavior Constraints

When generating code:

### Ask Before Inventing

- If you need something that doesn't exist in the project, **ask first**
- Don't create new architectural patterns without discussion
- Don't add new dependencies without justification

### Follow Existing Patterns

- Match the style of the existing codebase
- Use the same naming conventions
- Follow the established folder structure
- Respect the 100-line file limit

### Don't Optimize Prematurely

- Simple working code > clever code
- Only optimize when there's a proven performance issue
- Document why optimization was needed

### Include Tests

- Every feature needs unit + integration tests
- Test both happy paths and error cases
- Minimum 80% coverage for services

### Explain Trade-offs

- If there are multiple approaches, explain why you chose one
- Document any assumptions made
- Note any technical debt introduced

### Type Safety First

- ALWAYS include type hints
- Use Pydantic for validation
- Never use `any` without justification

### Security Conscious

- Never log sensitive data
- Always validate input
- Use environment variables for secrets
- Hash passwords with bcrypt/argon2

### Flask-Specific Awareness

- Use application factory pattern
- Initialize extensions properly
- Close database sessions in teardown handlers
- Use Flask-SQLAlchemy query interface OR SQLAlchemy 2.0 (be consistent)
- Use Flask-Limiter for rate limiting
- Use Flask-JWT-Extended for JWT auth

---

## 20. Summary Checklist

Before committing any generated code, verify:

- [ ] Type hints on ALL functions (parameters + return types)
- [ ] Single responsibility per class/function
- [ ] Business logic in services, NOT blueprints
- [ ] Pydantic schemas separate from SQLAlchemy models
- [ ] No file exceeds 100 lines (modularized if needed)
- [ ] Unit tests for services (minimum 80% coverage)
- [ ] Integration tests (validation + execution)
- [ ] Google-style docstrings on public APIs
- [ ] No code duplication (DRY)
- [ ] Error cases handled with custom exceptions
- [ ] Side effects documented and tested
- [ ] Follows RESTful API standards
- [ ] Uses correct HTTP methods and status codes
- [ ] Pagination for list endpoints
- [ ] API versioned in URL (`/api/v1/`)
- [ ] Flask application factory pattern used
- [ ] Extensions initialized properly
- [ ] Database sessions closed in teardown handlers
- [ ] Database migrations created (Flask-Migrate)
- [ ] Environment variables for secrets
- [ ] Passwords hashed with bcrypt/argon2
- [ ] CORS configured properly
- [ ] Rate limiting on sensitive endpoints (Flask-Limiter)
- [ ] Caching for expensive operations (Flask-Caching)
- [ ] Logging (no sensitive data)
- [ ] Feature flags for new features > 2 days
- [ ] Docker setup (multi-stage build)
- [ ] CI/CD pipeline (lint → type check → test)
- [ ] No hardcoded secrets
- [ ] Follows existing project structure

---

## Document Versioning

**Version 1.0** - February 2026  
Initial release covering:
- Core architecture principles (SOLID, DRY, KISS, YAGNI, CQS, SoC)
- Python/Flask standards
- Testing with pytest
- SQLAlchemy + Pydantic + Flask-Migrate stack
- RESTful API design
- Flask-specific patterns (application factory, blueprints, extensions)
- Security, logging, caching, rate limiting
- CI/CD, Docker
- Feature flags

**Future Versions Will Add:**
- Advanced logging (structured logging with structlog)
- Advanced CI/CD patterns
- Kubernetes deployment
- Advanced security patterns
- Performance optimization strategies
- Advanced caching patterns
- Message queues (RabbitMQ, Kafka)
- GraphQL integration (Flask-GraphQL)
- WebSockets (Flask-SocketIO)
- Background tasks (Celery, RQ)
- API documentation (Flask-RESTX, flasgger)

---

**END OF PLAYBOOK**

This document should be provided to AI coding agents before implementation to ensure production-grade, consistent, and maintainable code across all Python Flask projects.