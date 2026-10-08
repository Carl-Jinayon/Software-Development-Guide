# Python Code Patterns

Working reference code for a production-shaped Python service. Every pattern here is a pattern you should be able to write from memory.

---

## Table of Contents

- [1. Project structure](#1-project-structure)
- [2. pyproject.toml](#2-pyprojecttoml)
- [3. Configuration](#3-configuration)
- [4. Application factory](#4-application-factory)
- [5. Errors](#5-errors)
- [6. Logging](#6-logging)
- [7. Database models](#7-database-models)
- [8. Repositories](#8-repositories)
- [9. Pydantic schemas](#9-pydantic-schemas)
- [10. Routing](#10-routing)
- [11. Dependencies](#11-dependencies)
- [12. Authentication](#12-authentication)
- [13. Migrations](#13-migrations)
- [14. Testing](#14-testing)
- [15. Docker](#15-docker)
- [16. Type hints](#16-type-hints)
- [17. Async](#17-async)
- [18. Decorators](#18-decorators)
- [19. Performance](#19-performance)
- [20. Code quality tooling](#20-code-quality-tooling)

---

# 1. Project structure

```
myproject/
├── pyproject.toml
├── README.md
├── .env.example              # template only — never the real .env
├── .gitignore
├── alembic.ini
├── app/
│   ├── main.py               # app factory
│   ├── config.py             # settings
│   ├── container.py          # composition root
│   ├── domain/               # pure logic — zero technology imports
│   │   ├── entities.py
│   │   ├── scheduling.py
│   │   └── errors.py
│   ├── api/
│   │   ├── routes.py
│   │   ├── schemas.py
│   │   └── deps.py
│   ├── repositories/
│   │   ├── card_repo.py
│   │   └── user_repo.py
│   ├── services/             # use-case orchestration
│   ├── security/
│   │   ├── auth.py
│   │   ├── passwords.py
│   │   └── tokens.py
│   └── middleware/
│       ├── tracing.py
│       └── errors.py
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   ├── test_scheduling.py
│   │   └── test_scheduling_properties.py
│   ├── integration/
│   │   ├── test_decks_api.py
│   │   └── test_security.py
│   └── e2e/
└── docs/
```

## The dependency rule

```
domain/     imports nothing internal. Pure Python only.
    ↑
services/   imports domain.
    ↑
api/        imports services, domain.
    ↑
main.py     imports everything. The only place they meet.
```

**If `domain/` imports FastAPI or SQLAlchemy, the architecture is wrong.** That rule is what makes the domain testable in microseconds.

---

# 2. pyproject.toml

```toml
[project]
name = "myapp"
version = "0.1.0"
description = ""
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.115",
    "uvicorn[standard]>=0.32",
    "sqlalchemy>=2.0",
    "alembic>=1.14",
    "pydantic>=2.9",
    "pydantic-settings>=2.6",
    "psycopg[binary]>=3.2",
    "structlog>=24.4",
    "argon2-cffi>=23.1",
    "pyjwt>=2.9",
    "redis>=5.2",
    "slowapi>=0.1",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.3",
    "pytest-cov>=6.0",
    "pytest-asyncio>=0.24",
    "pytest-mock>=3.14",
    "httpx>=0.28",
    "hypothesis>=6.113",
    "ruff>=0.7",
    "mypy>=1.13",
    "bandit>=1.8",
    "pip-audit>=2.7",
    "pre-commit>=4.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B", "A", "S", "C4", "SIM", "RUF"]
ignore = ["S101"]                      # assert is fine in tests

[tool.ruff.lint.per-file-ignores]
"tests/**/*" = ["S", "N802"]

[tool.mypy]
python_version = "3.12"
strict = true
warn_unused_ignores = true
disallow_any_generics = true
plugins = ["pydantic.mypy"]

[[tool.mypy.overrides]]
module = ["argon2.*", "slowapi.*"]
ignore_missing_imports = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra --strict-markers --strict-config"
markers = [
    "slow: takes more than one second",
    "integration: needs external services",
    "e2e: full system",
    "security: security regression",
]
asyncio_mode = "auto"
filterwarnings = ["error", "ignore::DeprecationWarning"]

[tool.coverage.run]
source = ["app"]
branch = true

[tool.coverage.report]
show_missing = true
skip_covered = true
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
    "@overload",
]
```

---

# 3. Configuration

```python
# app/config.py
from functools import lru_cache
from typing import Literal

from pydantic import Field, SecretStr, field_validator
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    """
    Typed configuration read from the environment.
    Imported values are SecretStr so they never appear in logs or tracebacks.
    """

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
        case_sensitive=False,
    )

    # Application
    app_name: str = "myapp"
    environment: Literal["development", "staging", "production"] = "development"
    debug: bool = False
    api_prefix: str = "/api/v1"

    # Server
    host: str = "0.0.0.0"
    port: int = Field(default=8000, ge=1, le=65535)
    cors_origins: list[str] = Field(default_factory=list)

    # Database
    database_url: str
    db_pool_size: int = Field(default=10, ge=1, le=100)
    db_max_overflow: int = Field(default=20, ge=0)
    db_echo: bool = False

    # Security
    jwt_secret: SecretStr = Field(min_length=32)
    jwt_refresh_secret: SecretStr = Field(min_length=32)
    jwt_algorithm: str = "HS256"
    access_token_ttl_minutes: int = Field(default=15, ge=1, le=1440)
    refresh_token_ttl_days: int = Field(default=30, ge=1, le=365)
    encryption_key: SecretStr | None = None

    # Redis
    redis_url: str | None = None
    cache_ttl_seconds: int = Field(default=300, ge=1)

    # Observability
    log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"
    log_json: bool = True
    sentry_dsn: str | None = None

    # Rate limits
    login_rate_limit: str = "5 per minute"
    api_rate_limit: str = "60 per minute"

    @field_validator("cors_origins", mode="before")
    @classmethod
    def split_origins(cls, value):
        if isinstance(value, str):
            return [origin.strip() for origin in value.split(",") if origin.strip()]
        return value

    @field_validator("debug")
    @classmethod
    def no_debug_in_production(cls, value: bool, info) -> bool:
        if value and (info.data or {}).get("environment") == "production":
            raise ValueError("debug must be False in production")
        return value

    @property
    def is_production(self) -> bool:
        return self.environment == "production"


@lru_cache
def get_settings() -> Settings:
    """
    Cached, so the environment is read once. Import this rather than
    instantiating Settings directly.
    """
    return Settings()
```

```bash
# .env.example — committed. The real .env is gitignored.
ENVIRONMENT=development
DEBUG=true
DATABASE_URL=postgresql+psycopg://app:app@localhost:5432/appdb
JWT_SECRET=change-me-to-at-least-32-characters-in-production
JWT_REFRESH_SECRET=change-me-to-a-different-32-character-secret
CORS_ORIGINS=http://localhost:3000
REDIS_URL=redis://localhost:6379/0
LOG_LEVEL=DEBUG
```

---

# 4. Application factory

```python
# app/main.py
import logging
import uuid
from collections.abc import Awaitable, Callable
from contextlib import asynccontextmanager

import structlog
from fastapi import FastAPI, Request, Response
from fastapi.middleware.cors import CORSMiddleware
from starlette.middleware.httpsredirect import HTTPSRedirectMiddleware

from app.api.routes import router
from app.config import get_settings
from app.middleware.errors import install_exception_handlers

settings = get_settings()


def configure_logging() -> None:
    logging.basicConfig(level=settings.log_level.upper(), format="%(message)s")
    structlog.configure(
        processors=[
            structlog.contextvars.merge_contextvars,
            structlog.processors.add_log_level,
            structlog.processors.TimeStamper(fmt="iso", utc=True),
            structlog.processors.StackInfoRenderer(),
            structlog.processors.format_exc_info,
            structlog.processors.JSONRenderer() if settings.log_json
            else structlog.dev.ConsoleRenderer(),
        ],
        wrapper_class=structlog.make_filtering_bound_logger(
            logging.getLevelName(settings.log_level)
        ),
    )


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Startup and shutdown. Close pooled resources here."""
    configure_logging()
    log = structlog.get_logger()
    log.info("startup", environment=settings.environment, app=settings.app_name)
    yield
    await close_resources()
    log.info("shutdown")


def create_app() -> FastAPI:
    """
    Factory rather than a module-level instance: tests can build an app
    with different settings, and lifespan is properly handled.
    """
    app = FastAPI(
        title=settings.app_name,
        version="0.1.0",
        lifespan=lifespan,
        docs_url="/docs" if not settings.is_production else None,
        redoc_url=None,
        openapi_url="/openapi.json" if not settings.is_production else None,
    )

    if settings.is_production:
        app.add_middleware(HTTPSRedirectMiddleware)

    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.cors_origins,     # never "*" with credentials
        allow_credentials=True,
        allow_methods=["GET", "POST", "PATCH", "DELETE"],
        allow_headers=["Content-Type", "Authorization", "X-Request-ID"],
        max_age=3600,
    )

    @app.middleware("http")
    async def request_context(request: Request, call_next: Callable) -> Response:
        request_id = request.headers.get("X-Request-ID") or uuid.uuid4().hex
        request.state.request_id = request_id
        structlog.contextvars.clear_contextvars()
        structlog.contextvars.bind_contextvars(
            request_id=request_id, path=request.url.path, method=request.method
        )

        with structlog.contextvars.bound_contextvars():
            try:
                response = await call_next(request)
            except Exception:
                structlog.get_logger().exception("request.failed")
                raise

            response.headers["X-Request-ID"] = request_id
            structlog.get_logger().info(
                "request.completed", status=response.status_code
            )
            return response

    @app.middleware("http")
    async def security_headers(request: Request, call_next: Callable) -> Response:
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Permissions-Policy"] = "geolocation=(), microphone=(), camera=()"
        if settings.is_production:
            response.headers["Strict-Transport-Security"] = (
                "max-age=31536000; includeSubDomains; preload"
            )
        return response

    app.include_router(router, prefix=settings.api_prefix)

    @app.get("/api/health", tags=["ops"])
    async def health() -> dict[str, str]:
        return {"status": "ok"}

    install_exception_handlers(app)
    return app


app = create_app()
```

---

# 5. Errors

```python
# app/domain/errors.py
class DomainError(Exception):
    """Base for all domain errors. Carries a stable machine-readable code."""

    code: str = "DOMAIN_ERROR"
    status_code: int = 400
    message: str = "A domain rule was violated"

    def __init__(self, message: str | None = None, *, details: dict | None = None):
        super().__init__(message or self.message)
        self.details = details or {}


class NotFoundError(DomainError):
    code = "NOT_FOUND"
    status_code = 404
    message = "The requested resource does not exist"


class ValidationError(DomainError):
    code = "VALIDATION_FAILED"
    status_code = 422
    message = "The input failed validation"


class PermissionDeniedError(DomainError):
    code = "PERMISSION_DENIED"
    status_code = 403
    message = "You do not have permission to perform this action"


class AuthenticationError(DomainError):
    code = "AUTHENTICATION_FAILED"
    status_code = 401
    message = "Authentication failed"


class ConflictError(DomainError):
    code = "CONFLICT"
    status_code = 409
    message = "The resource already exists or is in a conflicting state"


class RateLimitError(DomainError):
    code = "RATE_LIMITED"
    status_code = 429
    message = "Too many requests"
```

```python
# app/middleware/errors.py
import structlog
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from sqlalchemy.exc import SQLAlchemyError

from app.domain.errors import DomainError

log = structlog.get_logger()


def install_exception_handlers(app: FastAPI) -> None:
    @app.exception_handler(DomainError)
    async def domain_error_handler(request: Request, exc: DomainError):
        log.info("domain_error", code=exc.code, detail=str(exc))
        return JSONResponse(
            status_code=exc.status_code,
            content={
                "type": f"https://api.example.com/errors/{exc.code.lower()}",
                "title": exc.code.replace("_", " ").title(),
                "status": exc.status_code,
                "detail": str(exc),
                "trace_id": getattr(request.state, "request_id", None),
                **({"errors": exc.details} if exc.details else {}),
            },
        )

    @app.exception_handler(RequestValidationError)
    async def validation_handler(request: Request, exc: RequestValidationError):
        """Return a stable, parseable error contract. Never echo the input."""
        return JSONResponse(
            status_code=422,
            content={
                "type": "https://api.example.com/errors/validation",
                "title": "Validation failed",
                "status": 422,
                "detail": "The request body failed validation",
                "errors": [
                    {
                        "field": ".".join(str(p) for p in err["loc"]),
                        "message": err["msg"],
                    }
                    for err in exc.errors()
                ],
                "trace_id": getattr(request.state, "request_id", None),
            },
        )

    @app.exception_handler(SQLAlchemyError)
    async def database_handler(request: Request, exc: SQLAlchemyError):
        """
        Log the detail, return a generic message.
        SQLAlchemy errors leak table and column names — never return them.
        """
        log.error("database_error", error=str(exc), exc_info=True)
        return JSONResponse(
            status_code=503,
            content={
                "type": "https://api.example.com/errors/service_unavailable",
                "title": "Service Unavailable",
                "status": 503,
                "detail": "A temporary error occurred. Please retry.",
                "trace_id": getattr(request.state, "request_id", None),
            },
        )

    @app.exception_handler(Exception)
    async def unhandled_handler(request: Request, exc: Exception):
        log.exception("unhandled_error", error=type(exc).__name__)
        return JSONResponse(
            status_code=500,
            content={
                "type": "https://api.example.com/errors/internal",
                "title": "Internal Server Error",
                "status": 500,
                "detail": "An unexpected error occurred.",
                "trace_id": getattr(request.state, "request_id", None),
            },
        )
```

```python
def test_error_responses_never_leak_internals(client):
    """Regression test: no stack traces, no SQL, no file paths."""
    response = client.get("/api/v1/decks/not-an-int")
    body = response.text

    assert response.status_code == 422
    for leak in ("Traceback", "sqlalchemy", "psycopg", "SELECT", "/app/", "File \""):
        assert leak not in body
```

---

# 6. Logging

```python
# app/logging_config.py
import logging
import sys
import uuid

import structlog

_request_id: str = ""


def get_logger(name: str | None = None) -> structlog.stdlib.BoundLogger:
    return structlog.get_logger(name)


class RequestIdFilter(logging.Filter):
    def filter(self, record: logging.LogRecord) -> bool:
        record.request_id = _request_id or "-"
        return True


def configure_logging(level: str = "INFO", json_output: bool = True) -> None:
    logging.basicConfig(
        format="%(message)s",
        stream=sys.stdout,
        level=level.upper(),
        force=True,
    )

    structlog.configure(
        processors=[
            structlog.contextvars.merge_contextvars,
            structlog.stdlib.add_log_level,
            structlog.stdlib.add_logger_name,
            structlog.processors.TimeStamper(fmt="iso", utc=True),
            structlog.processors.StackInfoRenderer(),
            structlog.processors.format_exc_info,
            structlog.processors.UnicodeDecoder(),
            structlog.processors.JSONRenderer() if json_output
            else structlog.dev.ConsoleRenderer(),
        ],
        logger_factory=structlog.stdlib.LoggerFactory(),
        wrapper_class=structlog.stdlib.BoundLogger,
        cache_logger_on_first_use=True,
    )
```

**Log the identifier and the outcome, never the payload.**

```python
# ✓ GOOD
log.info("deck.created", deck_id=deck.id, user_id=user.id, card_count=12)
log.warning("login.failed", email=email, reason="bad_password", ip=ip)

# ✗ BAD
log.info(f"User {user.email} created deck {deck.title} with cards {deck.cards}")
log.debug(f"Request body: {request_body}")       # may contain passwords and PII
```

---

# 7. Database models

```python
# app/models.py
from __future__ import annotations

import enum
from datetime import datetime

from sqlalchemy import (
    Boolean, CheckConstraint, DateTime, Enum, Float, ForeignKey, Index,
    Integer, String, Text, UniqueConstraint, func,
)
from sqlalchemy.dialects.postgresql import TSVECTOR
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship


class Base(DeclarativeBase):
    pass


class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), nullable=False
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False,
    )


class SoftDeleteMixin:
    deleted_at: Mapped[datetime | None] = mapped_column(
        DateTime(timezone=True), nullable=True, index=True
    )

    @property
    def is_deleted(self) -> bool:
        return self.deleted_at is not None


class Role(str, enum.Enum):
    OWNER = "owner"
    EDITOR = "editor"
    VIEWER = "viewer"


class User(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False, index=True)
    password_hash: Mapped[str] = mapped_column(String(255), nullable=False)
    display_name: Mapped[str | None] = mapped_column(String(120))
    role: Mapped[Role] = mapped_column(
        Enum(Role, name="user_role"), default=Role.VIEWER, nullable=False
    )
    is_active: Mapped[bool] = mapped_column(Boolean, default=True, nullable=False)

    decks: Mapped[list[Deck]] = relationship(
        back_populates="owner", cascade="all, delete-orphan", lazy="selectin"
    )

    __table_args__ = (
        CheckConstraint("length(email) <= 255", name="ck_users_email_length"),
    )


class Deck(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "decks"

    id: Mapped[int] = mapped_column(primary_key=True)
    owner_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"), nullable=False, index=True
    )
    title: Mapped[str] = mapped_column(String(120), nullable=False)
    description: Mapped[str | None] = mapped_column(Text)

    owner: Mapped[User] = relationship(back_populates="decks")
    cards: Mapped[list[Card]] = relationship(
        back_populates="deck", cascade="all, delete-orphan", lazy="selectin"
    )

    __table_args__ = (
        # Unique among live rows only — a soft-deleted title can be reused
        Index(
            "uq_decks_owner_title",
            "owner_id", "title",
            unique=True,
            postgresql_where=(deleted_at.is_(None)),
        ),
    )


class Card(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "cards"

    id: Mapped[int] = mapped_column(primary_key=True)
    deck_id: Mapped[int] = mapped_column(
        ForeignKey("decks.id", ondelete="CASCADE"), nullable=False, index=True
    )
    front: Mapped[str] = mapped_column(Text, nullable=False)
    back: Mapped[str] = mapped_column(Text, nullable=False)

    due_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), nullable=False, server_default=func.now()
    )
    ease: Mapped[float] = mapped_column(Float, default=2.5, nullable=False)
    interval_days: Mapped[int] = mapped_column(Integer, default=0, nullable=False)
    repetitions: Mapped[int] = mapped_column(Integer, default=0, nullable=False)
    lapses: Mapped[int] = mapped_column(Integer, default=0, nullable=False)

    deck: Mapped[Deck] = relationship(back_populates="cards")
    reviews: Mapped[list[ReviewLog]] = relationship(
        back_populates="card", cascade="all, delete-orphan", lazy="noload"
    )

    __table_args__ = (
        # The main query: due cards for a deck, in order.
        # Equality column first, range column last.
        Index("idx_cards_deck_due", "deck_id", "due_at",
              postgresql_where=(deleted_at.is_(None))),
        CheckConstraint("ease >= 1.3", name="ck_cards_ease_min"),
        CheckConstraint("interval_days >= 0", name="ck_cards_interval_nonneg"),
    )


class ReviewLog(Base):
    __tablename__ = "review_logs"

    id: Mapped[int] = mapped_column(primary_key=True)
    card_id: Mapped[int] = mapped_column(
        ForeignKey("cards.id", ondelete="CASCADE"), nullable=False, index=True
    )
    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"), nullable=False, index=True
    )
    was_correct: Mapped[bool] = mapped_column(Boolean, nullable=False)
    quality: Mapped[int] = mapped_column(Integer, nullable=False)
    time_taken_ms: Mapped[int | None] = mapped_column(Integer)
    reviewed_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), nullable=False
    )

    card: Mapped[Card] = relationship(back_populates="reviews")

    __table_args__ = (
        Index("idx_reviews_card_date", "card_id", "reviewed_at"),
        CheckConstraint("quality BETWEEN 0 AND 5", name="ck_reviews_quality_range"),
    )
```

**Database constraints are the last line of defence.** An application bug cannot violate a `CHECK` constraint.

---

# 8. Repositories

```python
# app/repositories/base.py
from collections.abc import Sequence
from typing import Generic, TypeVar

from sqlalchemy.orm import Session

from app.models import Base

ModelT = TypeVar("ModelT", bound=Base)


class Repository(Generic[ModelT]):
    def __init__(self, session: Session, model: type[ModelT]):
        self._session = session
        self._model = model

    def add(self, entity: ModelT) -> ModelT:
        self._session.add(entity)
        self._session.flush()
        return entity

    def get(self, entity_id: int) -> ModelT | None:
        return self._session.get(self._model, entity_id)

    def list(self, limit: int = 100, offset: int = 0) -> Sequence[ModelT]:
        stmt = select(self._model).limit(limit).offset(offset)
        return self._session.execute(stmt).scalars().all()
```

```python
# app/repositories/card_repo.py
from datetime import datetime

from sqlalchemy import func, select
from sqlalchemy.orm import Session, selectinload

from app.models import Card, Deck


class CardRepository:
    """
    Ownership is part of every query, not a check someone can forget.
    """

    def __init__(self, session: Session):
        self._session = session

    def get_for_user(self, card_id: int, user_id: int) -> Card | None:
        return self._session.execute(
            select(Card)
            .join(Deck, Card.deck_id == Deck.id)
            .where(
                Card.id == card_id,
                Deck.owner_id == user_id,       # ← ownership in the query
                Card.deleted_at.is_(None),
                Deck.deleted_at.is_(None),
            )
        ).scalar_one_or_none()

    def list_due(self, user_id: int, now: datetime, limit: int = 200) -> list[Card]:
        return list(
            self._session.execute(
                select(Card)
                .join(Deck, Card.deck_id == Deck.id)
                .where(
                    Deck.owner_id == user_id,
                    Card.due_at <= now,
                    Card.deleted_at.is_(None),
                    Deck.deleted_at.is_(None),
                )
                .order_by(Card.due_at.asc())
                .limit(limit)
            ).scalars().all()
        )

    def create(self, deck_id: int, front: str, back: str, due_at: datetime) -> Card:
        card = Card(deck_id=deck_id, front=front, back=back, due_at=due_at)
        self._session.add(card)
        self._session.flush()
        return card

    def update_schedule(self, card_id: int, due_at: datetime, ease: float,
                        interval_days: int, repetitions: int) -> None:
        """Atomic update — never read-modify-write."""
        self._session.execute(
            update(Card)
            .where(Card.id == card_id)
            .values(due_at=due_at, ease=ease,
                    interval_days=interval_days, repetitions=repetitions)
        )

    def count_by_deck(self, deck_id: int) -> int:
        return self._session.execute(
            select(func.count()).select_from(Card)
            .where(Card.deck_id == deck_id, Card.deleted_at.is_(None))
        ).scalar_one()

    def soft_delete(self, card_id: int, user_id: int) -> bool:
        result = self._session.execute(
            update(Card)
            .join(Deck, Card.deck_id == Deck.id)
            .where(Card.id == card_id, Deck.owner_id == user_id)
            .values(deleted_at=func.now())
        )
        return result.rowcount > 0
```

---

# 9. Pydantic schemas

```python
# app/api/schemas.py
from datetime import datetime
from typing import Annotated, Literal

from pydantic import BaseModel, ConfigDict, EmailStr, Field, field_validator


class APIModel(BaseModel):
    """Base for all response models. Blocks mass assignment."""

    model_config = ConfigDict(
        from_attributes=True,     # read from ORM objects
        extra="forbid",           # reject unknown fields
        str_strip_whitespace=True,
    )


class ORMBase(APIModel):
    model_config = ConfigDict(from_attributes=True, extra="ignore", populate_by_name=True)


# ── Requests ────────────────────────────────────────────────
Title = Annotated[str, Field(min_length=1, max_length=120)]
HexColour = Annotated[str, Field(pattern=r"^#[0-9a-fA-F]{6}$")]


class CreateDeckRequest(APIModel):
    title: Title
    description: str | None = Field(default=None, max_length=1000)
    colour: HexColour | None = None

    @field_validator("title")
    @classmethod
    def reject_control_characters(cls, value: str) -> str:
        cleaned = "".join(ch for ch in value if ch.isprintable()).strip()
        if not cleaned:
            raise ValueError("Title must contain printable characters")
        return cleaned


class UpdateDeckRequest(APIModel):
    """All fields optional for PATCH. No `id` or `owner_id` — they must never be settable."""

    title: Title | None = None
    description: str | None = Field(default=None, max_length=1000)
    colour: HexColour | None = None


class CreateCardRequest(APIModel):
    deck_id: int = Field(gt=0)
    front: str = Field(min_length=1, max_length=5000)
    back: str = Field(min_length=1, max_length=5000)

    @field_validator("front", "back")
    @classmethod
    def reject_control_characters(cls, value: str) -> str:
        cleaned = "".join(ch for ch in value if ch.isprintable()).strip()
        if not cleaned:
            raise ValueError("Content must contain printable characters")
        return cleaned


class RecordReviewRequest(APIModel):
    was_correct: bool
    quality: int = Field(ge=0, le=5)
    time_taken_ms: int | None = Field(default=None, ge=0, le=3_600_000)


class LoginRequest(APIModel):
    email: EmailStr
    password: str = Field(min_length=1, max_length=128)


# ── Responses ───────────────────────────────────────────────
class DeckSummary(ORMBase):
    id: int
    title: str
    description: str | None = None
    card_count: int = 0
    due_count: int = 0
    created_at: datetime


class DeckDetail(DeckSummary):
    updated_at: datetime


class CardResponse(ORMBase):
    id: int
    deck_id: int
    front: str
    back: str
    due_at: datetime
    ease: float
    interval_days: int


class ReviewResult(ORMBase):
    card: CardResponse
    next_review: datetime
    interval_days: int
    repetitions: int


class PaginatedResponse(APIModel):
    items: list
    page: int
    limit: int
    total: int


class TokenResponse(APIModel):
    access_token: str
    refresh_token: str
    token_type: Literal["bearer"] = "bearer"
    expires_in: int


class UserResponse(ORMBase):
    """Never include password_hash. This model is the only thing serialised."""

    id: int
    email: EmailStr
    display_name: str | None = None
    created_at: datetime
```

---

# 10. Routing

```python
# app/api/routes.py
from fastapi import APIRouter, Depends, Query, Response, status

from app.api.deps import CurrentUser, get_current_user, get_session
from app.api.schemas import (
    CardResponse, CreateCardRequest, CreateDeckRequest, DeckDetail,
    DeckSummary, PaginatedResponse, RecordReviewRequest, ReviewResult,
    UpdateDeckRequest,
)
from app.container import Container
from app.domain.errors import NotFoundError, PermissionDeniedError

router = APIRouter(tags=["decks"])


@router.get("/decks", response_model=PaginatedResponse)
async def list_decks(
    response: Response,
    page: int = Query(default=0, ge=0),
    limit: int = Query(default=20, ge=1, le=100),
    sort: str = Query(default="created_at", pattern="^(created_at|title|updated_at)$"),
    session=Depends(get_session),
    user: CurrentUser = Depends(get_current_user),
) -> PaginatedResponse:
    """List decks owned by the authenticated user."""
    container = Container(session)
    decks, total = container.list_decks(user.id, page, limit, sort)

    response.headers["X-Total-Count"] = str(total)
    return PaginatedResponse(
        items=[DeckSummary.model_validate(d) for d in decks],
        page=page, limit=limit, total=total,
    )


@router.post("/decks", response_model=DeckDetail, status_code=status.HTTP_201_CREATED)
async def create_deck(
    payload: CreateDeckRequest,
    response: Response,
    session=Depends(get_session),
    user: CurrentUser = Depends(get_current_user),
) -> DeckDetail:
    """Routes contain no business logic — they validate, delegate, and serialise."""
    container = Container(session)
    deck = container.create_deck(user.id, payload.title, payload.description)

    response.headers["Location"] = f"/api/v1/decks/{deck.id}"
    return DeckDetail.model_validate(deck)


@router.get("/decks/{deck_id}", response_model=DeckDetail)
async def get_deck(
    deck_id: int,
    session=Depends(get_session),
    user: CurrentUser = Depends(get_current_user),
) -> DeckDetail:
    container = Container(session)
    deck = container.get_deck_for_user(deck_id, user.id)

    if deck is None:
        # 403 rather than 404 leaks less about what exists
        raise PermissionDeniedError("Deck not found or not accessible")

    return DeckDetail.model_validate(deck)


@router.patch("/decks/{deck_id}", response_model=DeckDetail)
async def update_deck(
    deck_id: int,
    payload: UpdateDeckRequest,
    session=Depends(get_session),
    user: CurrentUser = Depends(get_current_user),
) -> DeckDetail:
    container = Container(session)
    deck = container.update_deck(deck_id, user.id, payload.model_dump(exclude_unset=True))
    return DeckDetail.model_validate(deck)


@router.delete("/decks/{deck_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_deck(
    deck_id: int,
    session=Depends(get_session),
    user: CurrentUser = Depends(get_current_user),
) -> Response:
    container = Container(session)
    container.delete_deck(deck_id, user.id)
    return Response(status_code=status.HTTP_204_NO_CONTENT)


@router.post("/decks/{deck_id}/cards", response_model=CardResponse,
             status_code=status.HTTP_201_CREATED)
async def create_card(
    deck_id: int,
    payload: CreateCardRequest,
    session=Depends(get_session),
    user: CurrentUser = Depends(get_current_user),
) -> CardResponse:
    container = Container(session)
    card = container.create_card(deck_id, user.id, payload.front, payload.back)
    return CardResponse.model_validate(card)


@router.post("/cards/{card_id}/reviews", response_model=ReviewResult)
async def record_review(
    card_id: int,
    payload: RecordReviewRequest,
    session=Depends(get_session),
    user: CurrentUser = Depends(get_current_user),
) -> ReviewResult:
    container = Container(session)
    result = container.record_review(
        card_id, user.id, payload.was_correct, payload.quality, payload.time_taken_ms
    )
    return ReviewResult(
        card=CardResponse.model_validate(result.card),
        next_review=result.next_review,
        interval_days=result.interval_days,
        repetitions=result.repetitions,
    )
```

---

# 11. Dependencies

```python
# app/api/deps.py
from collections.abc import Generator
from typing import Annotated

from fastapi import Depends, HTTPException, status
from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

from app.config import get_settings
from app.models import User
from app.security.tokens import decode_token

settings = get_settings()

engine = create_engine(
    settings.database_url,
    pool_size=settings.db_pool_size,
    max_overflow=settings.db_max_overflow,
    pool_pre_ping=True,
    pool_recycle=1800,
    echo=settings.db_echo,
)

SessionLocal = sessionmaker(bind=engine, autoflush=False, expire_on_commit=False)


def get_session() -> Generator[Session, None, None]:
    """
    One session per request. Commits on success, rolls back on exception.
    """
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()


def get_current_user(
    authorization: str | None = None,
    session: Session = Depends(get_session),
) -> User:
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Missing or malformed Authorization header",
            headers={"WWW-Authenticate": "Bearer"},
        )

    token = authorization.removeprefix("Bearer ").strip()

    try:
        user_id, token_type = decode_token(token, expected_type="access")
    except TokenError as exc:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED, detail=str(exc)
        ) from exc

    user = session.get(User, user_id)
    if user is None or not user.is_active or user.is_deleted:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED, detail="Account is not active"
        )

    return user


CurrentUser = Annotated[User, Depends(get_current_user)]
DbSession = Annotated[Session, Depends(get_session)]
```

---

# 12. Authentication

```python
# app/security/passwords.py
import argon2

# Memory-hard and tunable. Raise time_cost as hardware improves.
_hasher = argon2.PasswordHasher(
    time_cost=3,
    memory_cost=65536,      # 64 MB per hash
    parallelism=4,
)


def hash_password(plaintext: str) -> str:
    """Argon2id embeds its own salt and parameters in the output."""
    return _hasher.hash(plaintext)


def verify_password(stored_hash: str, plaintext: str) -> bool:
    """
    Never raises for an incorrect password — returns False.
    Never compares with == — that leaks timing information.
    """
    try:
        return _hasher.verify(stored_hash, plaintext)
    except (argon2.exceptions.VerifyMismatchError,
            argon2.exceptions.InvalidHashError,
            argon2.exceptions.VerificationError):
        return False


def needs_rehash(stored_hash: str) -> bool:
    """True when the parameters have been increased since this hash was made."""
    return _hasher.check_needs_rehash(stored_hash)
```

```python
# app/security/tokens.py
import uuid
from datetime import datetime, timedelta, timezone

import jwt

from app.config import get_settings

settings = get_settings()


class TokenError(Exception):
    pass


def create_token_pair(user_id: int, session_id: str | None = None) -> dict[str, str | int]:
    now = datetime.now(timezone.utc)
    session_id = session_id or str(uuid.uuid4())

    common = {"sub": str(user_id), "sid": session_id, "iat": now}

    access = jwt.encode(
        {**common, "type": "access", "exp": now + timedelta(minutes=settings.access_token_ttl_minutes)},
        settings.jwt_secret.get_secret_value(),
        algorithm=settings.jwt_algorithm,
    )
    refresh = jwt.encode(
        {**common, "type": "refresh", "exp": now + timedelta(days=settings.refresh_token_ttl_days)},
        settings.jwt_refresh_secret.get_secret_value(),
        algorithm=settings.jwt_algorithm,
    )

    return {
        "access_token": access,
        "refresh_token": refresh,
        "session_id": session_id,
        "expires_in": settings.access_token_ttl_minutes * 60,
    }


def decode_token(token: str, expected_type: str = "access") -> tuple[int, str]:
    """
    Validate fully. The allowed algorithm is hardcoded — NEVER read it from
    the token, or `alg: none` forgeries succeed.
    """
    secret = (
        settings.jwt_secret if expected_type == "access"
        else settings.jwt_refresh_secret
    )

    try:
        payload = jwt.decode(
            token,
            secret.get_secret_value(),
            algorithms=[settings.jwt_algorithm],
            audience="myapp",
            issuer="myapp",
            options={"require": ["exp", "iat", "sub", "type", "sid"]},
        )
    except jwt.ExpiredSignatureError as exc:
        raise TokenError("Token has expired") from exc
    except jwt.InvalidTokenError as exc:
        raise TokenError("Invalid token") from exc

    if payload["type"] != expected_type:
        raise TokenError(f"Expected a {expected_type} token")

    return int(payload["sub"]), payload["sid"]
```

```python
# app/services/auth.py
import structlog

from app.api.schemas import TokenResponse
from app.domain.errors import AuthenticationError, RateLimitError
from app.repositories.user_repo import UserRepository
from app.security.passwords import hash_password, needs_rehash, verify_password
from app.security.tokens import create_token_pair

log = structlog.get_logger()


class AuthService:
    def __init__(self, users: UserRepository):
        self._users = users

    def register(self, email: str, password: str, display_name: str | None) -> int:
        email = email.strip().lower()

        if self._users.exists_by_email(email):
            # Do not reveal that the email is taken — see the login path below
            raise AuthenticationError("Unable to create the account")

        user = self._users.create(
            email=email,
            password_hash=hash_password(password),   # never store plaintext
            display_name=display_name,
        )
        return user.id

    def login(self, email: str, password: str) -> TokenResponse:
        email = email.strip().lower()
        user = self._users.get_by_email(email)

        # ONE generic error for every failure mode — no user enumeration
        if user is None:
            # Hash anyway so timing does not reveal whether the account exists
            verify_password(_DUMMY_HASH, password)
            log.warning("login.failed", email=email, reason="no_such_user")
            raise AuthenticationError("Incorrect email or password")

        if not verify_password(user.password_hash, password):
            log.warning("login.failed", email=email, reason="bad_password",
                        user_id=user.id)
            raise AuthenticationError("Incorrect email or password")

        if not user.is_active or user.is_deleted:
            raise AuthenticationError("Incorrect email or password")

        # Opportunistic upgrade when the hashing parameters have increased
        if needs_rehash(user.password_hash):
            user.password_hash = hash_password(password)
            self._users.commit()

        return TokenResponse.model_validate(create_token_pair(user.id))
```

---

# 13. Migrations

```bash
alembic init alembic
alembic revision --autogenerate -m "create users, decks, cards, review_logs"
alembic upgrade head
alembic downgrade -1
alembic current
alembic history --verbose
```

```python
# alembic/versions/0001_initial.py
"""create users, decks, cards, review_logs

Revision ID: 0001
Revises:
Create Date: 2026-10-02
"""
from collections.abc import Sequence

import sqlalchemy as sa
from alembic import op

revision: str = "0001"
down_revision: str | None = None
branch_labels: str | Sequence[str] | None = None
depends_on: str | Sequence[str] | None = None


def upgrade() -> None:
    op.create_table(
        "users",
        sa.Column("id", sa.Integer(), primary_key=True),
        sa.Column("email", sa.String(255), nullable=False),
        sa.Column("password_hash", sa.String(255), nullable=False),
        sa.Column("display_name", sa.String(120)),
        sa.Column("role", sa.Enum("OWNER", "EDITOR", "VIEWER", name="user_role"),
                  server_default="VIEWER", nullable=False),
        sa.Column("is_active", sa.Boolean(), server_default=sa.true(), nullable=False),
        sa.Column("created_at", sa.DateTime(timezone=True),
                  server_default=sa.func.now(), nullable=False),
        sa.Column("updated_at", sa.DateTime(timezone=True),
                  server_default=sa.func.now(), nullable=False),
        sa.Column("deleted_at", sa.DateTime(timezone=True)),
    )
    op.create_index("uq_users_email", "users", ["email"], unique=True)

    # Partial unique index: only live decks need a unique title per owner
    op.create_index(
        "uq_decks_owner_title", "decks", ["owner_id", "title"],
        unique=True,
        postgresql_where=sa.text("deleted_at IS NULL"),
    )

    # Composite index: equality column first, range column last
    op.create_index(
        "idx_cards_deck_due", "cards", ["deck_id", "due_at"],
        postgresql_where=sa.text("deleted_at IS NULL"),
    )


def downgrade() -> None:
    op.drop_index("idx_cards_deck_due", table_name="cards")
    op.drop_index("uq_decks_owner_title", table_name="decks")
    op.drop_index("uq_users_email", table_name="users")
    op.drop_table("review_logs")
    op.drop_table("cards")
    op.drop_table("decks")
    op.drop_table("users")
    sa.Enum(name="user_role").drop(op.get_bind(), checkfirst=True)
```

**Migrations that break running code:**

| Migration | Safe? |
|---|---|
| `ADD COLUMN` nullable | Yes |
| `ADD COLUMN NOT NULL` with default | Yes |
| `ADD COLUMN NOT NULL` without default | **No** — old INSERTs fail |
| `DROP COLUMN` | **No** — old SELECTs break |
| `RENAME COLUMN` | **No** |
| `CREATE INDEX` (non-concurrent) | Locks writes. Use `CONCURRENTLY` on Postgres |
| Adding a `UNIQUE` constraint | **No** if duplicates exist |

---

# 14. Testing

```python
# tests/conftest.py
from collections.abc import Generator
from datetime import datetime, timezone

import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

from app.api.deps import get_current_user, get_session
from app.main import create_app
from app.models import Base, Card, Deck, User
from app.repositories.user_repo import UserRepository
from app.security.passwords import hash_password

VALID_PASSWORD = "correct-horse-battery-staple"


@pytest.fixture(scope="session")
def engine():
    test_url = "postgresql+psycopg://app:app@localhost:5432/testdb"
    eng = create_engine(test_url, poolclass=StaticPool)
    Base.metadata.create_all(eng)
    yield eng
    Base.metadata.drop_all(eng)
    eng.dispose()


@pytest.fixture
def session(engine) -> Generator[Session, None, None]:
    """
    One transaction per test, rolled back afterwards.
    Guarantees isolation without manual cleanup.
    """
    connection = engine.connect()
    transaction = connection.begin()
    test_session = Session(bind=connection, join_transaction_mode="create_savepoint")

    yield test_session

    test_session.close()
    transaction.rollback()
    connection.close()


@pytest.fixture
def users(session) -> UserRepository:
    return UserRepository(session)


@pytest.fixture
def user_factory(session):
    """Factory rather than fixture, so each test creates exactly what it needs."""
    def _make(email="test@example.com", role="viewer", password=VALID_PASSWORD, **kwargs):
        user = User(
            email=email,
            password_hash=hash_password(password),
            display_name=kwargs.pop("display_name", "Test User"),
            role=role,
            **kwargs,
        )
        session.add(user)
        session.commit()
        session.refresh(user)
        return user
    return _make


@pytest.fixture
def deck_factory(session, user_factory):
    def _make(owner=None, title="Test Deck", **kwargs):
        owner = owner or user_factory()
        deck = Deck(owner_id=owner.id, title=title, **kwargs)
        session.add(deck)
        session.commit()
        session.refresh(deck)
        return deck
    return _make


@pytest.fixture
def card_factory(session, deck_factory):
    def _make(deck=None, front="Question?", back="Answer", due_at=None, **kwargs):
        deck = deck or deck_factory()
        card = Card(
            deck_id=deck.id,
            front=front,
            back=back,
            due_at=due_at or datetime(2026, 1, 1, tzinfo=timezone.utc),
            **kwargs,
        )
        session.add(card)
        session.commit()
        session.refresh(card)
        return card
    return _make


@pytest.fixture
def app(session, user):
    application = create_app()
    application.dependency_overrides[get_session] = lambda: session
    application.dependency_overrides[get_current_user] = lambda: user
    yield application
    application.dependency_overrides.clear()      # MANDATORY


@pytest.fixture
def client(app) -> TestClient:
    return TestClient(app)


@pytest.fixture
def anon_client(session) -> TestClient:
    """No auth override — for testing 401 behaviour."""
    application = create_app()
    application.dependency_overrides[get_session] = lambda: session
    yield TestClient(application)
    application.dependency_overrides.clear()
```

```python
# tests/unit/test_scheduling.py — instant, no infrastructure
from datetime import datetime, timedelta, timezone

import pytest

from app.domain.scheduling import ReviewOutcome, Schedule, schedule_next


@pytest.fixture
def now() -> datetime:
    return datetime(2026, 1, 1, 12, 0, tzinfo=timezone.utc)


def make_schedule(**kwargs) -> Schedule:
    defaults = {
        "next_review": datetime(2026, 1, 1, tzinfo=timezone.utc),
        "interval": timedelta(days=1),
        "ease": 2.5,
        "repetitions": 0,
    }
    return Schedule(**{**defaults, **kwargs})


def test_failure_resets_to_one_day(now):
    current = make_schedule(interval=timedelta(days=90), repetitions=8)

    result = schedule_next(current, ReviewOutcome(quality=1, reviewed_at=now))

    assert result.interval == timedelta(days=1)
    assert result.repetitions == 0


def test_first_success_is_one_day(now):
    result = schedule_next(make_schedule(), ReviewOutcome(quality=5, reviewed_at=now))
    assert result.interval == timedelta(days=1)
    assert result.repetitions == 1


def test_second_success_is_six_days(now):
    current = make_schedule(interval=timedelta(days=1), repetitions=1)

    result = schedule_next(current, ReviewOutcome(quality=5, reviewed_at=now))

    assert result.interval == timedelta(days=6)
    assert result.repetitions == 2


def test_next_review_is_always_in_the_future(now):
    for quality in range(6):
        result = schedule_next(
            make_schedule(interval=timedelta(days=30), repetitions=5),
            ReviewOutcome(quality=quality, reviewed_at=now),
        )
        assert result.next_review > now


def test_ease_never_drops_below_floor(now):
    current = make_schedule(ease=1.3, interval=timedelta(days=1), repetitions=1)

    for _ in range(20):
        current = schedule_next(current, ReviewOutcome(quality=0, reviewed_at=now))

    assert current.ease >= 1.3
```

```python
# tests/unit/test_scheduling_properties.py — Hypothesis
from datetime import timedelta

from hypothesis import given, settings
import hypothesis.strategies as st

from app.domain.scheduling import ReviewOutcome, Schedule, schedule_next


@given(
    ease=st.floats(min_value=1.3, max_value=3.5),
    reps=st.integers(min_value=0, max_value=30),
    quality=st.integers(min_value=0, max_value=5),
    days=st.integers(min_value=1, max_value=365),
)
@settings(max_examples=300)
def test_interval_never_exceeds_one_year(ease, reps, quality, days):
    current = Schedule(next_review=None, interval=timedelta(days=days),
                       ease=ease, repetitions=reps)

    result = schedule_next(current, ReviewOutcome(quality=quality, reviewed_at=None))

    assert result.interval <= timedelta(days=365)


@given(reps=st.integers(min_value=0, max_value=50),
       quality=st.integers(min_value=0, max_value=5))
def test_repetitions_only_reset_on_failure(reps, quality):
    current = Schedule(None, timedelta(days=1), 2.5, reps)

    result = schedule_next(current, ReviewOutcome(quality=quality, reviewed_at=None))

    if quality < 3:
        assert result.repetitions == 0
    else:
        assert result.repetitions == reps + 1
```

```python
# tests/integration/test_security.py — the four tests every app needs
def test_cannot_read_another_users_deck(client, other_user_deck):
    """Broken access control is the most exploited vulnerability. Test it everywhere."""
    response = client.get(f"/api/v1/decks/{other_user_deck.id}")

    assert response.status_code in (403, 404)


def test_cannot_modify_another_users_deck(client, other_user_deck):
    response = client.patch(
        f"/api/v1/decks/{other_user_deck.id}", json={"title": "Hacked"}
    )
    assert response.status_code in (403, 404)


def test_mass_assignment_cannot_escalate(client, current_user):
    response = client.patch("/api/v1/users/me", json={"display_name": "New", "role": "owner"})

    assert response.status_code in (200, 422)
    assert current_user.role.value == "viewer"      # unchanged


def test_unknown_fields_are_rejected(client):
    response = client.post("/api/v1/decks",
                           json={"title": "Valid", "owner_id": 999, "is_admin": True})
    assert response.status_code == 422


def test_requires_authentication(anon_client):
    assert anon_client.get("/api/v1/decks").status_code == 401


def test_sql_injection_attempts_are_rejected(client):
    for payload in ["' OR '1'='1", "'; DROP TABLE users; --", "1' UNION SELECT * FROM users--"]:
        response = client.post("/api/v1/cards",
                               json={"deck_id": 1, "front": payload, "back": "x"})
        assert response.status_code in (403, 404, 422)


def test_responses_never_contain_secrets(client):
    for path in ["/api/v1/decks", "/api/v1/users/me", "/api/v1/health"]:
        body = client.get(path).text.lower()
        for secret in ("password", "password_hash", "argon2", "jwt_secret"):
            assert secret not in body
```

---

# 15. Docker

```dockerfile
# Multi-stage: build with full toolchain, ship with a minimal image
FROM python:3.12-slim AS builder

WORKDIR /build

RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY pyproject.toml README.md ./
COPY app ./app

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir --prefix=/install .


FROM python:3.12-slim AS runtime

# Never run as root
RUN groupadd --system app && useradd --system --gid app --create-home app

COPY --from=builder /install /usr/local

WORKDIR /app
COPY --chown=app:app . .

USER app

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONPATH=/app \
    PORT=8000

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=20s --retries=3 \
    CMD python -c "import urllib.request,sys; sys.exit(0 if urllib.request.urlopen('http://localhost:8000/api/health').status==200 else 1)"

CMD ["sh", "-c", "uvicorn app.main:app --host 0.0.0.0 --port ${PORT} --workers 4"]
```

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-localdev}
      POSTGRES_DB: appdb
    ports: ["5432:5432"]
    volumes: ["pgdata:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d appdb"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      retries: 5

  api:
    build:
      context: .
      target: runtime
    environment:
      ENVIRONMENT: development
      DEBUG: "true"
      DATABASE_URL: postgresql+psycopg://app:${POSTGRES_PASSWORD:-localdev}@db:5432/appdb
      REDIS_URL: redis://redis:6379/0
      JWT_SECRET: local-development-secret-at-least-32-chars
      JWT_REFRESH_SECRET: local-development-refresh-secret-32-chars
    ports: ["8000:8000"]
    depends_on:
      db:    { condition: service_healthy }
      redis: { condition: service_healthy }
    volumes: [".:/app"]
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

volumes:
  pgdata:
```

---

# 16. Type hints

```python
from collections.abc import Callable, Iterable, Mapping, Sequence
from typing import Any, Literal, Protocol, TypeVar

T = TypeVar("T")

# Prefer concrete generics over Any
def process(items: list[dict[str, object]], *, strict: bool = False) -> list[str]:
    ...


# New union syntax (3.10+)
def find(user_id: int | None) -> User | None:
    ...


# Callable types
Handler = Callable[[Request, dict[str, str]], Response]
Middleware = Callable[[Request, Callable], Awaitable[Response]]


# TypeVar for generics
def first(items: Sequence[T]) -> T | None:
    return items[0] if items else None


# Literal for closed sets
SortField = Literal["created_at", "title", "updated_at"]


# Protocol for structural typing (no inheritance needed)
class PasswordHasher(Protocol):
    def hash(self, plaintext: str) -> str: ...
    def verify(self, stored: str, plaintext: str) -> bool: ...


# Annotated for metadata
from typing import Annotated
from pydantic import Field

UserId = Annotated[int, Field(gt=0, description="The user identifier")]


# TypedDict for JSON-ish structures
class TokenPayload(TypedDict):
    sub: str
    sid: str
    type: Literal["access", "refresh"]
    iat: datetime
    exp: datetime


# When you must use Any, constrain it
def load(raw: object) -> Card:                 # object, not Any
    if not isinstance(raw, dict):
        raise TypeError("expected a mapping")
    ...
```

---

# 17. Async

```python
import asyncio
import httpx
from collections.abc import Awaitable


async def fetch_many(urls: list[str], concurrency: int = 10) -> list[httpx.Response]:
    """Bounded concurrency — never gather unbounded."""
    semaphore = asyncio.Semaphore(concurrency)

    async with httpx.AsyncClient(timeout=httpx.Timeout(5.0, connect=2.0)) as client:
        async def fetch(url: str) -> httpx.Response:
            async with semaphore:            # bounded
                return await client.get(url)

        return await asyncio.gather(*(fetch(url) for url in urls),
                                    return_exceptions=True)


async def with_timeout(awaitable: Awaitable[T], seconds: float = 5.0) -> T:
    return await asyncio.wait_for(awaitable, timeout=seconds)


async def retry_with_backoff(fn, attempts: int = 3, base_delay: float = 0.5):
    for attempt in range(attempts):
        try:
            return await fn()
        except (httpx.TimeoutException, httpx.HTTPStatusError) as exc:
            if attempt == attempts - 1:
                raise
            # Exponential backoff with jitter — without jitter this
            # synchronises every client and becomes a thundering herd
            delay = base_delay * (2 ** attempt) * (0.5 + random.random())
            await asyncio.sleep(delay)
```

**Note:** SQLAlchemy's async support is less mature than its sync API. A synchronous database call inside an async endpoint **blocks the event loop**. For most applications, sync SQLAlchemy with FastAPI's thread pool is the simpler, more reliable choice.

---

# 18. Decorators

```python
import functools
import logging
import time
from collections.abc import Callable
from typing import Any, ParamSpec, TypeVar

P = ParamSpec("P")
R = TypeVar("R")

log = logging.getLogger(__name__)


def timed(logger: logging.Logger = log) -> Callable:
    """Log the duration and outcome of every call."""
    def decorator(fn: Callable[P, R]) -> Callable[P, R]:
        @functools.wraps(fn)                    # preserves __name__, __doc__, signature
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            start = time.perf_counter()
            try:
                return fn(*args, **kwargs)
            except Exception:
                logger.exception("%s failed", fn.__qualname__)
                raise
            finally:
                elapsed = (time.perf_counter() - start) * 1000
                logger.info("%s took %.1fms", fn.__qualname__, elapsed)
        return wrapper
    return decorator


def retry(attempts: int = 3, exceptions: tuple[type[Exception], ...] = (Exception,)):
    def decorator(fn: Callable[P, R]) -> Callable[P, R]:
        @functools.wraps(fn)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            last: Exception | None = None
            for attempt in range(attempts):
                try:
                    return fn(*args, **kwargs)
                except exceptions as exc:
                    last = exc
                    if attempt < attempts - 1:
                        time.sleep(0.5 * (2 ** attempt))
            assert last is not None
            raise last
        return wrapper
    return decorator


def require_permission(permission: str):
    """Authorisation as a decorator — impossible to forget at the call site."""
    def decorator(fn: Callable[P, R]) -> Callable[P, R]:
        @functools.wraps(fn)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            user = kwargs.get("user")
            if user is None:
                raise PermissionDeniedError("Authentication required")
            if permission not in PERMISSIONS.get(user.role, set()):
                raise PermissionDeniedError(f"Requires '{permission}'")
            return fn(*args, **kwargs)
        return wrapper
    return decorator
```

**Always use `functools.wraps`.** Without it the wrapper loses its name, docstring, and signature — which breaks FastAPI's dependency injection and every introspection tool.

---

# 19. Performance

```python
import time
from functools import lru_cache
from itertools import groupby
from operator import itemgetter


# ── The N+1 fix: eager loading ─────────────────────────────
from sqlalchemy import select
from sqlalchemy.orm import selectinload, joinedload

# WRONG — 1 + N queries
decks = session.execute(select(Deck).where(Deck.owner_id == uid)).scalars().all()
for deck in decks:
    count = len(deck.cards)                  # ← fires a SELECT per deck

# RIGHT — 2 queries regardless of N
decks = session.execute(
    select(Deck).options(selectinload(Deck.cards)).where(Deck.owner_id == uid)
).scalars().all()


# ── Keyset pagination, not OFFSET ──────────────────────────
def list_cards(deck_id: int, after_id: int | None = None, limit: int = 50):
    """O(limit) regardless of how deep you are. OFFSET degrades linearly."""
    stmt = select(Card).where(Card.deck_id == deck_id, Card.deleted_at.is_(None))
    if after_id is not None:
        stmt = stmt.where(Card.id > after_id)
    return list(session.execute(stmt.order_by(Card.id).limit(limit)).scalars().all())


# ── Aggregate in the database, not in Python ───────────────
# WRONG — pull 100k rows into memory to count them
count = len(session.execute(select(Card).where(...)).scalars().all())

# RIGHT
from sqlalchemy import func
count = session.execute(
    select(func.count()).select_from(Card).where(Card.deck_id == deck_id)
).scalar_one()


# ── Batch inserts ──────────────────────────────────────────
session.execute(insert(Card), [
    {"deck_id": deck_id, "front": f"Q{i}", "back": f"A{i}", "due_at": now}
    for i in range(10_000)
])
session.commit()                               # ONE round trip, not 10,000


# ── Caching pure functions ──────────────────────────────────
@lru_cache(maxsize=1000)
def compute_schedule(key: str) -> Schedule:
    """Only safe for pure functions of the arguments."""
    ...


# ── Never build SQL by string concatenation ────────────────
# WRONG
session.execute(text(f"SELECT * FROM cards WHERE front = '{front}'"))
# RIGHT
session.execute(text("SELECT * FROM cards WHERE front = :front"), {"front": front})


# ── Never use `in` on a large list ─────────────────────────
ids = [1, 2, 3, 50_000]
# WRONG — a full scan
session.execute(select(Card).where(Card.id.in_(ids)))
# RIGHT — a hash lookup
session.execute(select(Card).where(Card.id.in_(set(ids))))
```

```python
def test_list_decks_query_count_is_constant(session, user, monkeypatch):
    """The regression test that stops the N+1 coming back."""
    from sqlalchemy import event
    from sqlalchemy.engine import Engine

    queries = []
    event.listen(Engine, "before_cursor_execute",
                 lambda conn, cursor, stmt, params, ctx, many:
                 queries.append(stmt))

    for i in range(20):
        Deck(owner_id=user.id, title=f"Deck {i}")
    session.commit()
    queries.clear()

    list_decks_for(session, user.id)

    assert len(queries) == 2, f"N+1 detected: {len(queries)} queries for 20 decks"
```

---

# 20. Code quality tooling

```bash
# Format and lint
ruff format .
ruff check . --fix
ruff check . --output-format=github        # for CI annotations

# Type check
mypy app/

# Tests
pytest -x -q --cov --cov-report=html --cov-report=term-missing

# Security
bandit -r app/ -lll
pip-audit --strict
gitleaks detect --source . --redact

# Pre-commit
pre-commit install
pre-commit run --all-files
```

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12", cache: pip }
      - run: pip install -e ".[dev]"
      - run: ruff format --check .
      - run: ruff check .
      - run: mypy app/

  test:
    runs-on: ubuntu-latest
    needs: quality
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: app
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 5s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12", cache: pip }
      - run: pip install -e ".[dev]"
      - run: pytest -q --cov --cov-report=xml --cov-report=term
      - run: coverage report --fail-under=80
      - run: pip install bandit pip-audit && bandit -r app/ -lll && pip-audit --strict
```

---

## What to internalise

The value of these patterns is not that you will copy them verbatim. It is that you will **stop being surprised** by them — so that when you open an unfamiliar codebase, the layout, the dependency direction, the error handling, and the test fixtures are all things you have seen and can navigate immediately.

Learn the shapes. Then adapt them to whatever the problem actually needs, and delete whatever the problem does not need.
