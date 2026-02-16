# Backend Patterns Reference

Detailed patterns for the Python + FastAPI + SQLAlchemy backend.

---

## Model Definitions

### Standard Model Template

```python
from datetime import datetime, timezone
from uuid import uuid4
from sqlalchemy import String, Boolean, DateTime, UUID, ForeignKey, UniqueConstraint
from sqlalchemy.orm import mapped_column, relationship
from app.database import Base


class Item(Base):
    __tablename__ = "items"
    __table_args__ = (
        UniqueConstraint("owner_id", "name", name="uq_item_owner_name"),
    )

    id = mapped_column(UUID, primary_key=True, default=uuid4)
    owner_id = mapped_column(UUID, ForeignKey("users.id"), nullable=False)
    name = mapped_column(String(255), nullable=False)
    description = mapped_column(String, nullable=True)
    is_active = mapped_column(Boolean, default=True)
    created_at = mapped_column(DateTime(timezone=True), default=lambda: datetime.now(timezone.utc))
    updated_at = mapped_column(DateTime(timezone=True), onupdate=lambda: datetime.now(timezone.utc))

    # Relationships
    owner = relationship("User", back_populates="items")
```

### Conventions

- **UUID primary keys**: `default=uuid4` — auto-generated, no sequence conflicts
- **Timezone-aware datetimes**: Always `DateTime(timezone=True)` with UTC default
- **Soft deletes**: `is_active` boolean instead of actual deletion
- **Unique constraints**: Defined in `__table_args__` — enforced at DB level
- **Explicit nullability**: Always specify `nullable=True` or `nullable=False`
- **String lengths**: Use `String(255)` for short text, `String` (unlimited) for long text

### Numeric Fields

For financial or decimal data, use `Numeric`:

```python
from sqlalchemy import Numeric

price = mapped_column(Numeric(precision=12, scale=2), default=0)
shares = mapped_column(Numeric(precision=16, scale=8), default=0)  # Supports fractional
```

Never use `Float` for financial data — floating-point arithmetic causes rounding errors.

### JSON Fields

For flexible or list-like data stored in a single column:

```python
from sqlalchemy import JSON

allowed_items = mapped_column(JSON, nullable=True)  # NULL = no restrictions
settings = mapped_column(JSON, default=dict)
```

Use when the data doesn't need relational queries. For data you need to filter/join on, use a proper table.

---

## Router Patterns

### Standard CRUD Router

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app.services.auth_service import get_current_user

router = APIRouter()


@router.get("/")
async def list_items(
    db: AsyncSession = Depends(get_db),
    user=Depends(get_current_user),
):
    return await item_service.list_items(db, user.id)


@router.post("/", status_code=201)
async def create_item(
    req: CreateItemRequest,
    db: AsyncSession = Depends(get_db),
    user=Depends(get_current_user),
):
    return await item_service.create_item(db, user.id, req)


@router.get("/{item_id}")
async def get_item(
    item_id: str,
    db: AsyncSession = Depends(get_db),
    user=Depends(get_current_user),
):
    item = await item_service.get_item(db, user.id, item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return item
```

### Conventions

- **Dependencies**: `db` and `user` via `Depends()` on every authenticated endpoint
- **Status codes**: 201 for creation, 200 for everything else (FastAPI defaults)
- **Error handling**: Raise `HTTPException` in routers, not in services
- **Return values**: Return service results directly — FastAPI serializes to JSON

### Admin-Only Endpoints

```python
@router.post("/admin/action")
async def admin_action(
    db: AsyncSession = Depends(get_db),
    user=Depends(get_current_user),
):
    if not user.is_admin:
        raise HTTPException(status_code=403, detail="Admin access required")
    return await admin_service.do_action(db)
```

---

## Service Patterns

### Atomic Operations

Wrap multi-step operations in a single commit:

```python
async def execute_trade(db: AsyncSession, user_id: UUID, req: TradeRequest):
    # 1. Validate
    player = await get_player(db, user_id, req.season_id)
    if not player:
        raise ValueError("Not in this season")

    # 2. Create immutable record
    transaction = Transaction(
        player_id=player.id,
        type=req.type,
        amount=req.amount,
    )
    db.add(transaction)

    # 3. Update derived state
    player.balance -= req.amount

    # 4. Single commit (atomic)
    await db.commit()
    return transaction
```

**Rule:** Either everything commits or nothing does. Never commit in the middle of a multi-step operation.

### Query Patterns

```python
from sqlalchemy import select
from sqlalchemy.orm import selectinload

# Simple select
result = await db.execute(select(User).where(User.id == user_id))
user = result.scalar_one_or_none()

# Select with eager loading (prevent N+1)
result = await db.execute(
    select(User)
    .options(selectinload(User.items))
    .where(User.id == user_id)
)

# Select with join filter
result = await db.execute(
    select(Item)
    .join(User)
    .where(User.id == user_id, Item.is_active == True)
    .order_by(Item.created_at.desc())
)
items = result.scalars().all()

# Aggregate
from sqlalchemy import func
result = await db.execute(
    select(func.count(Item.id)).where(Item.owner_id == user_id)
)
count = result.scalar()
```

### External API Calls

```python
import httpx

async def fetch_external_data(identifier: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f"https://api.example.com/data/{identifier}",
            headers={"X-API-Key": settings.external_api_key},
            timeout=10.0,
        )
        response.raise_for_status()
        return response.json()
```

**Rules:**
- Always use `httpx` (async) instead of `requests` (sync)
- Set explicit timeouts
- Use `raise_for_status()` to convert HTTP errors to exceptions
- API keys from settings, never hardcoded

---

## Pydantic Schemas

### Request/Response DTOs

```python
from pydantic import BaseModel, Field
from uuid import UUID
from datetime import datetime


class CreateItemRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=255)
    description: str | None = None


class ItemResponse(BaseModel):
    id: UUID
    name: str
    description: str | None
    is_active: bool
    created_at: datetime

    class Config:
        from_attributes = True  # Allows ORM model → Pydantic conversion
```

### Conventions

- **Request models**: Only include fields the client should send
- **Response models**: Only include fields the client should see (no token hashes, internal IDs)
- **Validation**: Use `Field()` constraints — FastAPI returns 422 with field-level errors automatically
- **`from_attributes = True`**: Required to create Pydantic models directly from SQLAlchemy results

---

## Alembic (Migrations)

### Async Environment Setup

```python
# alembic/env.py
from app.database import Base, engine
from app.models import *  # Import all models for autogenerate

def run_migrations_online():
    connectable = engine
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=Base.metadata)
    with context.begin_transaction():
        context.run_migrations()
```

### Commands

```bash
# Generate migration from model changes
alembic revision --autogenerate -m "add items table"

# Apply all pending migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1

# View migration history
alembic history
```

### Rules

- Always review autogenerated migrations before committing
- Migration messages should describe what changed: "add items table", "add index on items.name"
- Never edit a migration that has been applied to production — create a new one instead
- Test migrations: `upgrade head` then `downgrade -1` then `upgrade head` again

---

## Error Handling

### Approach

- **Services** raise `ValueError` or custom exceptions for business rule violations
- **Routers** catch exceptions and convert to `HTTPException` with appropriate status codes
- **FastAPI** handles Pydantic validation errors automatically (422)

```python
# service
async def create_item(db, user_id, req):
    if await name_taken(db, user_id, req.name):
        raise ValueError("Item name already exists")
    ...

# router
@router.post("/")
async def create_item(req, db=Depends(get_db), user=Depends(get_current_user)):
    try:
        return await item_service.create_item(db, user.id, req)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
```

### HTTP Status Codes

| Code | When |
|------|------|
| 200 | Success (default) |
| 201 | Resource created |
| 400 | Bad request (business rule violation) |
| 401 | Not authenticated |
| 403 | Not authorized (authenticated but insufficient permissions) |
| 404 | Resource not found |
| 422 | Validation error (Pydantic handles automatically) |
| 429 | Rate limited |
| 500 | Unexpected server error |
