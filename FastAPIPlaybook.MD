# Python FastAPI Execution Specification (Playbook)

> **Purpose**: This document defines the architectural standards, patterns, and constraints for all code generation and implementation in Python FastAPI projects. Feed this to any AI agent before implementation.

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
6. [FastAPI-Specific Patterns](#6-fastapi-specific-patterns)
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
- Routes are thin: handle HTTP, delegate to services
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
    async def process_payment(self, amount: float) -> bool:
        pass

class StripeGateway(PaymentGateway):
    async def process_payment(self, amount: float) -> bool:
        # Stripe implementation
        pass

class PayPalGateway(PaymentGateway):
    async def process_payment(self, amount: float) -> bool:
        # PayPal implementation
        pass
```

### Liskov Substitution Principle (LSP)

- Subclasses can replace parent classes without breaking functionality
- Don't override methods in ways that violate parent contracts

**Example**:

```python
# ✅ GOOD: Duck can replace Bird
class Bird:
    def fly(self) -> str:
        return "Flying"

class Duck(Bird):
    def fly(self) -> str:
        return "Duck flying"

# ❌ BAD: Penguin can't fly but inherits from Bird
class Penguin(Bird):
    def fly(self) -> str:
        raise NotImplementedError("Penguins can't fly")
```

### Interface Segregation Principle (ISP)

- Break large interfaces into smaller, focused ones
- Clients shouldn't depend on methods they don't use

**Example**:

```python
# ❌ BAD: One giant interface
class PaymentInterface(ABC):
    @abstractmethod
    def process_credit_card(self): pass

    @abstractmethod
    def process_paypal(self): pass

    @abstractmethod
    def process_crypto(self): pass

# ✅ GOOD: Separate focused interfaces
class CreditCardPayment(ABC):
    @abstractmethod
    async def process(self): pass

class PayPalPayment(ABC):
    @abstractmethod
    async def process(self): pass

class CryptoPayment(ABC):
    @abstractmethod
    async def process(self): pass
```

### Dependency Inversion Principle (DIP)

- High-level modules don't depend on low-level modules
- Both depend on abstractions
- Use dependency injection everywhere

**Example**:

```python
# ✅ Service depends on abstraction, not concrete implementation
class PaymentService:
    def __init__(self, gateway: PaymentGateway):
        self.gateway = gateway

    async def charge_customer(self, amount: float) -> bool:
        return await self.gateway.process_payment(amount)
```

### Keep it Simple, Stupid (KISS)

- Prefer straightforward solutions over clever ones
- No premature optimization
- If you can't explain it in one sentence, it's too complex

### Don't Repeat Yourself (DRY)

- Avoid code duplication
- Extract repeated logic into helper functions or utilities
- Use abstractions to prevent tight coupling

**Example**:

```python
# ❌ BAD: Duplicated validation logic
def create_user(email: str):
    if not re.match(r'^[^@]+@[^@]+\.[^@]+$', email):
        raise ValueError("Invalid email")
    # ...

def update_user(email: str):
    if not re.match(r'^[^@]+@[^@]+\.[^@]+$', email):
        raise ValueError("Invalid email")
    # ...

# ✅ GOOD: Reusable validation
def validate_email(email: str) -> bool:
    return bool(re.match(r'^[^@]+@[^@]+\.[^@]+$', email))

def create_user(email: str):
    if not validate_email(email):
        raise ValueError("Invalid email")
    # ...
```

### You Ain't Gonna Need It (YAGNI)

- Only implement features when needed
- No premature optimization or unnecessary features
- Don't build for hypothetical future requirements

### Command-Query Separation (CQS)

- **Commands**: Mutate state, return None or success indicator
- **Queries**: Return data, never mutate state
- Name accordingly: `create_user()` vs `get_user_by_id()`

**Example**:

```python
# ✅ GOOD: Clear separation
async def create_user(data: UserCreate, db: DbDep) -> User:  # Command
    """Creates user (mutates state)"""
    user = User(**data.model_dump())
    db.add(user)
    await db.commit()
    return user

async def get_user_by_id(user_id: int, db: DbDep) -> User | None:  # Query
    """Retrieves user (no mutation)"""
    return await db.get(User, user_id)
```

### Separation of Concerns (SoC)

- **Routes**: HTTP layer (routing, request/response mapping) - THIN
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

async def process_order(
    order_id: int,
    user_id: int,
    items: List[Dict[str, Any]]
) -> Order:
    pass

# ❌ NEVER: No type hints
async def process_order(order_id, user_id, items):
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
async def find_by_id(user_id: UserId) -> User | None:
    pass

async def get_all_users(limit: int = 10) -> List[User]:
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
async def create_user(data: UserCreate, db: DbDep) -> User:
    """
    Creates a new user in the database.

    This function hashes the password, validates email uniqueness,
    and sends a welcome email to the user.

    Args:
        data: User creation data containing email and password
        db: Database session dependency

    Returns:
        User: The created user entity with hashed password

    Raises:
        DuplicateEmailError: If email already exists in database
        ValidationError: If data validation fails

    Example:
        >>> user_data = UserCreate(email="test@example.com", password="secret123")
        >>> user = await create_user(user_data, db)
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
    belong in the route/controller layer.

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

**Module Docstrings:**

```python
"""
User authentication and authorization services.

This module provides business logic for user authentication,
including login, logout, password reset, and token management.

Key components:
    - AuthService: Main authentication service
    - TokenService: JWT token generation and validation
    - PasswordService: Password hashing and verification
"""
```

---

## 4. Testing Standards

### Testing Framework: pytest

Use **pytest** as the primary testing framework.

Required packages:

- `pytest` - Core testing framework
- `pytest-asyncio` - For async tests
- `pytest-cov` - Coverage reporting
- `httpx` - For FastAPI TestClient
- `faker` or `factory-boy` - Test data generation

### Test Structure

```markdown
tests/
├── unit/ # Unit tests (business logic isolation)
│ ├── services/
│ │ ├── test_user_service.py
│ │ └── test_payment_service.py
│ ├── utils/
│ │ └── test_helpers.py
│ └── validators/
│ └── test_email_validator.py
├── integration/ # Integration tests (full flow)
│ ├── routes/
│ │ ├── test_user_routes.py
│ │ └── test_order_routes.py
│ └── database/
│ └── test_user_repository.py
├── e2e/ # End-to-end tests
│ └── test_user_workflow.py
├── conftest.py # Shared fixtures
└── pytest.ini # Pytest configuration
```

### Unit Tests

Test business logic in isolation. Mock all external dependencies.

**Naming convention:**

- `test_<function_name>_<scenario>`
- Example: `test_create_user_with_valid_data`, `test_create_user_raises_error_when_email_exists`

**Example:**

```python
import pytest
from unittest.mock import Mock, AsyncMock
from src.services.user_service import UserService
from src.schemas.user import UserCreate
from src.exceptions import DuplicateEmailError

@pytest.fixture
def mock_db():
    return Mock()

@pytest.fixture
def mock_email_service():
    service = Mock()
    service.send_welcome_email = AsyncMock()
    return service

@pytest.fixture
def user_service(mock_db, mock_email_service):
    return UserService(db=mock_db, email_service=mock_email_service)

class TestUserService:
    """Unit tests for UserService business logic"""

    @pytest.mark.asyncio
    async def test_create_user_hashes_password(self, user_service, mock_db):
        # Arrange
        data = UserCreate(email="test@example.com", password="plaintext123")
        mock_db.query().filter().first.return_value = None  # No existing user

        # Act
        user = await user_service.create_user(data)

        # Assert
        assert user.password != "plaintext123"  # Password was hashed
        assert len(user.password) > 50  # Hashed password is long

    @pytest.mark.asyncio
    async def test_create_user_raises_error_when_email_exists(self, user_service, mock_db):
        # Arrange
        data = UserCreate(email="existing@example.com", password="secret123")
        mock_db.query().filter().first.return_value = Mock()  # User exists

        # Act & Assert
        with pytest.raises(DuplicateEmailError):
            await user_service.create_user(data)

    @pytest.mark.asyncio
    async def test_create_user_sends_welcome_email(self, user_service, mock_email_service):
        # Arrange
        data = UserCreate(email="new@example.com", password="secret123")

        # Act
        await user_service.create_user(data)

        # Assert
        mock_email_service.send_welcome_email.assert_called_once()
```

### Integration Tests: Validation

Test input validation with FastAPI routes. No database hits (mock repository).

**Example:**

```python
import pytest
from httpx import AsyncClient
from src.app import app

@pytest.mark.asyncio
async def test_create_user_rejects_invalid_email():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/api/v1/users", json={
            "email": "invalid-email",
            "password": "secret123"
        })

        assert response.status_code == 422  # Validation error
        assert "email" in response.json()["detail"][0]["loc"]

@pytest.mark.asyncio
async def test_create_user_rejects_short_password():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/api/v1/users", json={
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
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession
from src.app import app
from src.database import get_db
from tests.fixtures import test_db_session

@pytest.mark.asyncio
async def test_create_user_and_verify_in_database(test_db_session: AsyncSession):
    """
    Integration test: Full user creation flow

    Tests:
    1. POST request creates user
    2. User is saved to database
    3. Password is hashed
    4. Welcome email is sent
    """
    async with AsyncClient(app=app, base_url="http://test") as client:
        # Override DB dependency
        app.dependency_overrides[get_db] = lambda: test_db_session

        # Act
        response = await client.post("/api/v1/users", json={
            "email": "integration@example.com",
            "password": "secret123"
        })

        # Assert HTTP response
        assert response.status_code == 201
        assert response.json()["email"] == "integration@example.com"

        # Verify database state
        user = await test_db_session.get(User, response.json()["id"])
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

```markdown
src/services/user_service.py # 180 lines - TOO LARGE
```

**Example - After (Modular):**

```markdown
src/services/UserService/
├── **init**.py # Barrel exports (10 lines)
├── create_user.py # 80 lines
├── update_user.py # 60 lines
├── delete_user.py # 40 lines
└── helpers.py # 50 lines
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

### Standard Project Structure

```markdown
project/
├── src/
│ ├── app.py # FastAPI app entry point
│ ├── config/ # Configuration management
│ │ ├── **init**.py
│ │ ├── settings.py # Pydantic settings
│ │ └── database.py # DB config
│ ├── routes/ # API routes (THIN)
│ │ ├── **init**.py
│ │ ├── users.py
│ │ └── orders.py
│ ├── services/ # Business logic (CORE)
│ │ ├── **init**.py
│ │ ├── UserService/ # Modular if > 100 lines
│ │ │ ├── **init**.py
│ │ │ ├── create_user.py
│ │ │ └── update_user.py
│ │ └── order_service.py
│ ├── models/ # SQLAlchemy models
│ │ ├── **init**.py
│ │ ├── user.py
│ │ └── order.py
│ ├── schemas/ # Pydantic schemas
│ │ ├── **init**.py
│ │ ├── user.py
│ │ └── order.py
│ ├── validators/ # Input validation logic
│ │ ├── **init**.py
│ │ └── email_validator.py
│ ├── middlewares/ # Custom middlewares
│ │ ├── **init**.py
│ │ ├── auth.py
│ │ └── logging.py
│ ├── utils/ # Generic utilities
│ │ ├── **init**.py
│ │ └── date_helpers.py
│ ├── helpers/ # Domain-specific helpers
│ │ ├── **init**.py
│ │ └── payment_helpers.py
│ ├── exceptions/ # Custom exceptions
│ │ ├── **init**.py
│ │ └── user_exceptions.py
│ └── types/ # Type definitions
│ ├── **init**.py
│ └── common.py
├── tests/ # Test suite
│ ├── unit/
│ ├── integration/
│ ├── e2e/
│ ├── conftest.py
│ └── pytest.ini
├── alembic/ # Database migrations
│ ├── versions/
│ └── env.py
├── docs/ # Documentation
│ ├── api_spec.md
│ └── database_schema.md
├── logs/ # Application logs
├── scripts/ # Utility scripts
│ ├── seed_database.py
│ └── run_migrations.py
├── .env.example # Environment variables template
├── .gitignore
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml # Poetry/project config
├── requirements.txt # Dependencies
└── README.md
```

### Module Organization Rules

1. **Group by feature/domain**, not by layer (if project is large)
2. Each module is self-contained
3. Use `__init__.py` for barrel exports
4. Keep related code together

**Example - Feature-based structure (large projects):**

```markdown
src/
├── users/ # User domain module
│ ├── **init**.py
│ ├── routes.py
│ ├── service.py
│ ├── models.py
│ ├── schemas.py
│ └── exceptions.py
└── orders/ # Order domain module
├── **init**.py
├── routes.py
├── service.py
├── models.py
└── schemas.py
```

---

## 6. FastAPI-Specific Patterns

### Routes (THIN Layer)

Routes handle ONLY HTTP concerns. Delegate all business logic to services.

**What routes SHOULD do:**

- Parse request data
- Call service methods
- Map responses to HTTP format
- Handle HTTP errors

**What routes SHOULD NOT do:**

- Business logic
- Database queries
- Complex validation
- Email sending

**Example:**

```python
from fastapi import APIRouter, status, Depends
from typing import List
from src.schemas.user import UserCreate, UserResponse
from src.services.user_service import UserService
from src.database import DbDep

router = APIRouter(prefix="/api/v1/users", tags=["users"])

@router.post(
    "/",
    response_model=UserResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Create a new user",
    description="Creates a new user account with email and password"
)
async def create_user(
    data: UserCreate,
    db: DbDep,
    user_service: UserService = Depends()
) -> UserResponse:
    """
    Create a new user.

    - **email**: Valid email address (unique)
    - **password**: Minimum 8 characters with at least one digit
    """
    # THIN: Only call service and map response
    user = await user_service.create_user(data, db)
    return UserResponse.model_validate(user)

@router.get("/", response_model=List[UserResponse])
async def get_users(
    limit: int = 10,
    offset: int = 0,
    db: DbDep = Depends(),
    user_service: UserService = Depends()
) -> List[UserResponse]:
    """Get list of users with pagination"""
    users = await user_service.get_all_users(limit, offset, db)
    return [UserResponse.model_validate(u) for u in users]
```

### Services (Business Logic - CORE)

Services contain all business logic. Framework-agnostic and reusable.

**What services SHOULD do:**

- Business logic
- Data validation (business rules)
- Database operations (via repositories or direct)
- Side effects (emails, events, cache invalidation)
- Transaction management

**What services SHOULD NOT do:**

- HTTP concerns (status codes, headers)
- Request parsing
- Response formatting

**Example:**

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from typing import List
from src.models.user import User
from src.schemas.user import UserCreate
from src.exceptions import DuplicateEmailError
from src.utils.password import hash_password
from src.services.email_service import EmailService

class UserService:
    """
    User service containing all user-related business logic.

    This service is framework-agnostic and can be used from
    HTTP routes, CLI commands, background tasks, etc.
    """

    def __init__(self, email_service: EmailService):
        self.email_service = email_service

    async def create_user(self, data: UserCreate, db: AsyncSession) -> User:
        """
        Creates a new user with hashed password.

        Business rules:
        - Email must be unique
        - Password must be hashed before storage
        - Welcome email must be sent

        Args:
            data: User creation data
            db: Database session

        Returns:
            User: Created user entity

        Raises:
            DuplicateEmailError: If email already exists
        """
        # Check email uniqueness (business rule)
        existing = await self._get_by_email(data.email, db)
        if existing:
            raise DuplicateEmailError(f"Email {data.email} already exists")

        # Hash password (security requirement)
        hashed_password = hash_password(data.password)

        # Create user
        user = User(
            email=data.email,
            password=hashed_password,
            is_active=True
        )

        db.add(user)
        await db.commit()
        await db.refresh(user)

        # Side effect: Send welcome email
        await self.email_service.send_welcome_email(user.email, user.id)

        return user

    async def get_all_users(
        self,
        limit: int,
        offset: int,
        db: AsyncSession
    ) -> List[User]:
        """
        Retrieves paginated list of users.

        Args:
            limit: Maximum number of users to return
            offset: Number of users to skip
            db: Database session

        Returns:
            List[User]: List of user entities
        """
        result = await db.execute(
            select(User)
            .limit(limit)
            .offset(offset)
            .order_by(User.created_at.desc())
        )
        return list(result.scalars().all())

    async def _get_by_email(self, email: str, db: AsyncSession) -> User | None:
        """Private helper to find user by email"""
        result = await db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
```

### Dependency Injection Pattern

Use FastAPI's `Depends()` for dependency injection.

**Common dependencies:**

```python
from typing import Annotated
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from src.database import get_db

# Database session dependency
DbDep = Annotated[AsyncSession, Depends(get_db)]

# Current user dependency
from src.auth import get_current_user
CurrentUser = Annotated[User, Depends(get_current_user)]

# Service dependencies
def get_user_service() -> UserService:
    return UserService(email_service=EmailService())

UserServiceDep = Annotated[UserService, Depends(get_user_service)]
```

**Usage in routes:**

```python
@router.get("/me", response_model=UserResponse)
async def get_current_user_profile(
    current_user: CurrentUser,  # Injected
    db: DbDep                    # Injected
) -> UserResponse:
    return UserResponse.model_validate(current_user)
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

**Global exception handler:**

```python
# src/app.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from src.exceptions.base import AppException

app = FastAPI()

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    """Handle all custom application exceptions"""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "message": exc.message,
            "status": exc.status_code
        }
    )

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    """Handle value errors as 400 Bad Request"""
    return JSONResponse(
        status_code=status.HTTP_400_BAD_REQUEST,
        content={
            "success": False,
            "message": str(exc),
            "status": 400
        }
    )
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

```markdown
✅ GOOD:
GET /api/v1/users # List users
GET /api/v1/users/123 # Get specific user
POST /api/v1/users # Create user
PUT /api/v1/users/123 # Update entire user
PATCH /api/v1/users/123 # Update partial user
DELETE /api/v1/users/123 # Delete user

GET /api/v1/users/123/orders # Get user's orders (nested)
GET /api/v1/orders?user_id=123 # Alternative: filter by query param

❌ BAD:
GET /api/v1/getUsers # Don't use verbs
POST /api/v1/user # Use plural
GET /api/v1/Users # Don't capitalize
GET /api/v1/user_list # Don't use underscores
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
from fastapi import status

@router.post("/users", status_code=status.HTTP_201_CREATED)
async def create_user(data: UserCreate):
    # Returns 201 on success
    pass

@router.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(user_id: int):
    # Returns 204 on success (no body)
    pass
```

### Pagination (REQUIRED for Lists)

Use **query parameters** for pagination:

```python
@router.get("/users")
async def get_users(
    page: int = 1,
    limit: int = 20,
    db: DbDep = Depends()
):
    """
    Get paginated list of users.

    Query params:
    - page: Page number (default: 1)
    - limit: Items per page (default: 20, max: 100)
    """
    offset = (page - 1) * limit
    users = await user_service.get_all_users(limit, offset, db)
    total = await user_service.count_users(db)

    return {
        "data": users,
        "pagination": {
            "page": page,
            "limit": limit,
            "total": total,
            "pages": (total + limit - 1) // limit
        }
    }
```

### API Versioning (MANDATORY)

Use **URL versioning**:

```python
# ✅ GOOD: Version in URL
router = APIRouter(prefix="/api/v1/users")

# ❌ BAD: No versioning
router = APIRouter(prefix="/users")
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

class Article(Base):
    __tablename__ = "articles"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    slug: Mapped[str] = mapped_column(unique=True, index=True)

    def __init__(self, title: str, **kwargs):
        super().__init__(**kwargs)
        self.title = title
        self.slug = slugify(title)

# Route
@router.get("/articles/{slug}")
async def get_article(slug: str, db: DbDep):
    article = await db.execute(
        select(Article).where(Article.slug == slug)
    )
    return article.scalar_one_or_none()
```

---

## 8. Database & ORM Standards

### ORM Stack: Pydantic + SQLAlchemy + Alembic

**Required packages:**

- `pydantic` (v2+) - Data validation and serialization
- `sqlalchemy` (2.0+) - ORM
- `alembic` - Database migrations
- `asyncpg` - PostgreSQL async driver (or `aiosqlite` for SQLite)

### Separation of Models and Schemas (CRITICAL)

**NEVER mix SQLAlchemy models with Pydantic schemas.**

**SQLAlchemy Models** (Database layer):

```python
# src/models/user.py
from sqlalchemy.orm import Mapped, mapped_column, relationship
from sqlalchemy import String, Boolean, DateTime
from datetime import datetime
from src.database import Base

class User(Base):
    """SQLAlchemy User model - represents database table"""
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
    created_at: Mapped[datetime] = mapped_column(
        DateTime,
        default=datetime.utcnow
    )

    # Relationships
    orders: Mapped[list["Order"]] = relationship(back_populates="user")
```

**Pydantic Schemas** (Validation/Serialization layer):

```python
# src/schemas/user.py
from pydantic import BaseModel, EmailStr, Field
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

    model_config = {"from_attributes": True}  # Enable ORM mode
```

### SQLAlchemy 2.0 Async Pattern (MANDATORY)

Use async SQLAlchemy 2.0 style:

```python
# src/database.py
from sqlalchemy.ext.asyncio import (
    create_async_engine,
    async_sessionmaker,
    AsyncSession
)
from sqlalchemy.orm import DeclarativeBase
from typing import AsyncGenerator

class Base(DeclarativeBase):
    """Base class for all models"""
    pass

# Database URL (from config)
DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/dbname"

# Create async engine
engine = create_async_engine(
    DATABASE_URL,
    echo=True,  # Log SQL queries (disable in production)
    future=True
)

# Create session factory
async_session = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)

# Dependency for FastAPI
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """Database session dependency"""
    async with async_session() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

### Database Queries (Best Practices)

**Use SQLAlchemy 2.0 select() style:**

```python
from sqlalchemy import select, and_, or_
from sqlalchemy.ext.asyncio import AsyncSession

# ✅ GOOD: SQLAlchemy 2.0 style
async def get_active_users(db: AsyncSession) -> list[User]:
    result = await db.execute(
        select(User)
        .where(User.is_active == True)
        .order_by(User.created_at.desc())
    )
    return list(result.scalars().all())

# ✅ GOOD: Complex query with joins
async def get_user_with_orders(user_id: int, db: AsyncSession) -> User | None:
    result = await db.execute(
        select(User)
        .options(selectinload(User.orders))
        .where(User.id == user_id)
    )
    return result.scalar_one_or_none()

# ❌ BAD: Legacy SQLAlchemy 1.x style
users = db.query(User).filter(User.is_active == True).all()
```

### Database Migrations with Alembic

**Setup Alembic:**

```bash
# Initialize Alembic
alembic init alembic

# Configure alembic/env.py to use your models
# Then create migrations
alembic revision --autogenerate -m "Create users table"

# Apply migrations
alembic upgrade head
```

**Alembic best practices:**

- ALWAYS review auto-generated migrations before applying
- Use descriptive migration messages
- Never edit applied migrations (create new ones)
- Keep migrations small and focused
- Test migrations on staging before production

### Database Constraints & Indexes

Add constraints for data integrity:

```python
from sqlalchemy import UniqueConstraint, Index, CheckConstraint

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    age: Mapped[int]

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
class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    orders: Mapped[list["Order"]] = relationship(back_populates="user")

# models/order.py
class Order(Base):
    __tablename__ = "orders"
    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    user: Mapped["User"] = relationship(back_populates="orders")
```

**Many-to-Many:**

```python
# Association table
user_roles = Table(
    "user_roles",
    Base.metadata,
    Column("user_id", ForeignKey("users.id"), primary_key=True),
    Column("role_id", ForeignKey("roles.id"), primary_key=True)
)

class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    roles: Mapped[list["Role"]] = relationship(secondary=user_roles)

class Role(Base):
    __tablename__ = "roles"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
```

---

## 9. Configuration Management

### Use Pydantic Settings (MANDATORY)

**Config structure:**

```python
# src/config/settings.py
from pydantic_settings import BaseSettings
from pydantic import Field, computed_field
from typing import Literal
from pathlib import Path
from functools import lru_cache

class BaseConfig(BaseSettings):
    """Base configuration with common settings"""

    # App metadata
    APP_NAME: str = Field("My FastAPI App")
    APP_VERSION: str = Field("1.0.0")
    DEBUG: bool = Field(False)

    # Database
    DATABASE_URL: str = Field(..., env="DATABASE_URL")

    # Security
    JWT_SECRET: str = Field(..., env="JWT_SECRET")
    JWT_ALGORITHM: str = Field("HS256")
    ACCESS_TOKEN_EXPIRE_MINUTES: int = Field(30)

    # CORS
    CORS_ORIGINS: list[str] = Field(default_factory=list)

    # Redis
    REDIS_URL: str = Field("redis://localhost:6379/0")

    # AWS
    AWS_ACCESS_KEY_ID: str | None = Field(None, env="AWS_ACCESS_KEY_ID")
    AWS_SECRET_ACCESS_KEY: str | None = Field(None, env="AWS_SECRET_ACCESS_KEY")
    AWS_S3_BUCKET: str | None = Field(None, env="AWS_S3_BUCKET")

    model_config = {
        "env_file": ".env",
        "env_file_encoding": "utf-8",
        "case_sensitive": True
    }

class DevelopmentConfig(BaseConfig):
    """Development environment configuration"""
    DEBUG: bool = True
    CORS_ORIGINS: list[str] = Field(default_factory=lambda: ["*"])

class ProductionConfig(BaseConfig):
    """Production environment configuration"""
    DEBUG: bool = False
    CORS_ORIGINS: list[str] = Field(..., env="CORS_ORIGINS")

@lru_cache()
def get_settings() -> BaseConfig:
    """
    Get environment-specific settings.

    Cached to avoid re-reading .env file on every call.
    """
    env = os.getenv("PYTHON_ENV", "development")

    configs = {
        "development": DevelopmentConfig,
        "production": ProductionConfig,
    }

    if env not in configs:
        raise ValueError(f"Invalid PYTHON_ENV: {env}")

    return configs[env]()

settings = get_settings()
```

### Environment Variables (.env)

**Example `.env` file:**

```env
PYTHON_ENV=development

# Database
DATABASE_URL=postgresql+asyncpg://user:password@localhost:5432/dbname

# Security
JWT_SECRET=your-super-secret-key-change-in-production

# CORS (comma-separated for production)
CORS_ORIGINS=http://localhost:3000,https://example.com

# Redis
REDIS_URL=redis://localhost:6379/0

# AWS (optional)
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_S3_BUCKET=your-bucket-name

# Email (if using)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
```

**CRITICAL**: Never commit `.env` to git. Use `.env.example` as template.

### Dependency Injection for Settings

```python
from typing import Annotated
from fastapi import Depends
from src.config.settings import get_settings, BaseConfig

SettingsDep = Annotated[BaseConfig, Depends(get_settings)]

@router.get("/info")
async def get_app_info(settings: SettingsDep):
    return {
        "app_name": settings.APP_NAME,
        "version": settings.APP_VERSION,
        "environment": "production" if not settings.DEBUG else "development"
    }
```

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

class FeatureFlag(str, Enum):
    NEW_CHECKOUT_FLOW = "feature.payments.new_checkout_flow"
    ADVANCED_SEARCH = "feature.search.advanced_search"
    BETA_DASHBOARD = "feature.ui.beta_dashboard"

class FeatureFlagService:
    """Service for managing feature flags"""

    def __init__(self):
        # In production, use LaunchDarkly, Split.io, or database
        self._flags = {
            FeatureFlag.NEW_CHECKOUT_FLOW: False,  # Default OFF
            FeatureFlag.ADVANCED_SEARCH: True,
            FeatureFlag.BETA_DASHBOARD: False,
        }

    def is_enabled(self, flag: FeatureFlag, user_id: int | None = None) -> bool:
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

async def checkout(user_id: int):
    """Process checkout with feature flag"""
    flag_service = get_feature_flag_service()

    if flag_service.is_enabled(FeatureFlag.NEW_CHECKOUT_FLOW, user_id):
        return await new_checkout_experience(user_id)
    else:
        return await legacy_checkout_experience(user_id)
```

### Best Practices

1. **Naming convention**: `feature.<team>.<feature-name>`
2. **Default to OFF** for new features
3. **Document** in code or config file with owner and expiration date
4. **Test both paths** when feature flag exists
5. **Clean up** after rollout completes (avoid toggle debt)
6. **Log** toggle state changes for auditing

**Example documentation (toggles.yaml):**

```yaml
features:
  - name: feature.payments.new_checkout_flow
    owner: payments-team
    created: 2026-01-15
    expires: 2026-03-15
    description: New streamlined checkout flow with one-click payment
    status: testing

  - name: feature.search.advanced_search
    owner: search-team
    created: 2026-01-01
    expires: never # Permanent permission toggle
    description: Advanced search with filters (premium feature)
    status: enabled
```

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

### JWT Authentication

```python
# src/auth/jwt.py
from datetime import datetime, timedelta
from jose import jwt, JWTError
from src.config.settings import settings

def create_access_token(data: dict, expires_delta: timedelta | None = None) -> str:
    """
    Create JWT access token.

    Args:
        data: Payload data to encode
        expires_delta: Optional expiration time delta

    Returns:
        str: Encoded JWT token
    """
    to_encode = data.copy()

    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(
            minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES
        )

    to_encode.update({"exp": expire})

    encoded_jwt = jwt.encode(
        to_encode,
        settings.JWT_SECRET,
        algorithm=settings.JWT_ALGORITHM
    )

    return encoded_jwt

def verify_token(token: str) -> dict:
    """
    Verify and decode JWT token.

    Args:
        token: JWT token string

    Returns:
        dict: Decoded token payload

    Raises:
        JWTError: If token is invalid or expired
    """
    try:
        payload = jwt.decode(
            token,
            settings.JWT_SECRET,
            algorithms=[settings.JWT_ALGORITHM]
        )
        return payload
    except JWTError:
        raise
```

### CORS Configuration

```python
from fastapi.middleware.cors import CORSMiddleware
from src.config.settings import settings

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Input Validation (Automatic with Pydantic)

Pydantic handles input validation automatically. No additional work needed.

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
```

### SQL Injection Protection (Automatic with SQLAlchemy)

SQLAlchemy parameterizes queries automatically. NEVER use string formatting for queries.

```python
# ✅ GOOD: Parameterized query (safe)
result = await db.execute(
    select(User).where(User.email == email)
)

# ❌ BAD: String formatting (SQL injection risk)
query = f"SELECT * FROM users WHERE email = '{email}'"  # NEVER DO THIS
```

### Environment Variables for Secrets

NEVER hardcode secrets in code. Use environment variables.

```python
# ❌ BAD: Hardcoded secret
JWT_SECRET = "my-secret-key"

# ✅ GOOD: From environment
JWT_SECRET = os.getenv("JWT_SECRET")
```

### HTTPS Only in Production

```python
if not settings.DEBUG:
    # Enforce HTTPS in production
    app.add_middleware(HTTPSRedirectMiddleware)
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
    async def create_user(self, data: UserCreate, db: AsyncSession) -> User:
        logger.info(f"Creating user with email: {data.email}")

        try:
            # Business logic
            user = User(**data.model_dump())
            db.add(user)
            await db.commit()

            logger.info(f"User created successfully: {user.id}")
            return user

        except Exception as e:
            logger.error(f"Failed to create user: {str(e)}", exc_info=True)
            raise
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

### Log Levels

- **DEBUG**: Detailed diagnostic information
- **INFO**: General informational messages
- **WARNING**: Warning messages for potentially harmful situations
- **ERROR**: Error messages for failures
- **CRITICAL**: Critical messages for severe failures

---

## 13. Caching Strategies

### Use Redis for Distributed Caching

```python
# src/cache/redis_client.py
import redis.asyncio as redis
from src.config.settings import settings

redis_client = redis.from_url(
    settings.REDIS_URL,
    encoding="utf-8",
    decode_responses=True
)
```

### Caching with aiocache

```python
from aiocache import cached
from aiocache.serializers import JsonSerializer

@cached(
    ttl=300,  # Cache for 5 minutes
    key_builder=lambda f, *args, **kwargs: f"user:{args[0]}",
    serializer=JsonSerializer()
)
async def get_user_by_id(user_id: int, db: AsyncSession) -> User | None:
    """
    Get user by ID with caching.

    Caches result for 5 minutes to reduce database load.
    """
    result = await db.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()
```

### Cache Invalidation

```python
from aiocache import Cache

cache = Cache(Cache.REDIS, endpoint="localhost", port=6379)

async def update_user(user_id: int, data: UserUpdate, db: AsyncSession):
    """Update user and invalidate cache"""

    # Update user
    user = await db.get(User, user_id)
    for key, value in data.model_dump(exclude_unset=True).items():
        setattr(user, key, value)

    await db.commit()

    # Invalidate cache
    await cache.delete(f"user:{user_id}")

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

### Use slowapi (Flask-Limiter for FastAPI)

```python
# src/app.py
from slowapi import Limiter
from slowapi.util import get_remote_address
from slowapi.middleware import SlowAPIMiddleware

limiter = Limiter(key_func=get_remote_address)

app = FastAPI()
app.state.limiter = limiter
app.add_middleware(SlowAPIMiddleware)

# Apply rate limit to route
@app.get("/users")
@limiter.limit("5/minute")
async def get_users():
    return {"users": []}
```

### Redis-backed Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(
    key_func=get_remote_address,
    storage_uri="redis://localhost:6379"
)
```

### Custom Rate Limiting by User

```python
def get_user_id(request: Request) -> str:
    """Extract user ID from JWT token for rate limiting"""
    token = request.headers.get("Authorization", "").replace("Bearer ", "")
    try:
        payload = verify_token(token)
        return f"user:{payload['sub']}"
    except:
        return get_remote_address(request)

limiter = Limiter(key_func=get_user_id)

@app.post("/orders")
@limiter.limit("10/minute")  # 10 orders per minute per user
async def create_order(current_user: CurrentUser):
    pass
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

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-asyncio pytest-cov ruff mypy

      - name: Lint with Ruff
        run: ruff check src/

      - name: Type check with mypy
        run: mypy src/

      - name: Run tests with coverage
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

# Create non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

# Add local bin to PATH
ENV PATH=/root/.local/bin:$PATH

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "src.app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### docker-compose.yml

```yaml
version: "3.8"

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - PYTHON_ENV=development
      - DATABASE_URL=postgresql+asyncpg://user:password@db:5432/dbname
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    volumes:
      - ./src:/app/src # Hot reload in development

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

```dockerfile
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
- ❌ Don't put business logic in routes/controllers (use services)
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
- ❌ Don't use legacy SQLAlchemy 1.x query style (use 2.0)
- ❌ Don't skip database migrations (use Alembic)
- ❌ Don't log sensitive data (passwords, tokens, PII)
- ❌ Don't commit .env files to git
- ❌ Don't skip tests (minimum 80% coverage for services)
- ❌ Don't use global state (use dependency injection)
- ❌ Don't optimize prematurely (simple working code > clever code)

---

## 18. When Implementing New Features

Follow this workflow:

1. **Define types first** (Pydantic schemas, SQLAlchemy models)
2. **Create database migration** (Alembic)
3. **Write failing tests** (TDD approach preferred)
4. **Implement service layer** (business logic)
5. **Add route/controller layer** (HTTP mapping)
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

---

## 20. Summary Checklist

Before committing any generated code, verify:

- [ ] Type hints on ALL functions (parameters + return types)
- [ ] Single responsibility per class/function
- [ ] Business logic in services, NOT routes
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
- [ ] SQLAlchemy 2.0 async style
- [ ] Database migrations created (Alembic)
- [ ] Environment variables for secrets
- [ ] Passwords hashed with bcrypt/argon2
- [ ] CORS configured properly
- [ ] Rate limiting on sensitive endpoints
- [ ] Caching for expensive operations
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
- Python/FastAPI standards
- Testing with pytest
- SQLAlchemy + Pydantic + Alembic stack
- RESTful API design
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
- GraphQL integration
- WebSockets
- Background tasks (Celery, ARQ)

---

## END OF PLAYBOOK
