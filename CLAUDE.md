# CLAUDE.md - GTM Copilot Project Guide

> **Last Updated**: 2025-11-23
> **Project Phase**: Phase 0 - Infrastructure Skeleton
> **Status**: Greenfield starter template with Docker infrastructure ready

This document provides comprehensive guidance for AI assistants working on the GTM Copilot codebase.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture & Technology Stack](#architecture--technology-stack)
3. [Directory Structure](#directory-structure)
4. [Development Workflow](#development-workflow)
5. [Key Conventions](#key-conventions)
6. [Database & Models](#database--models)
7. [API Structure](#api-structure)
8. [Frontend Structure](#frontend-structure)
9. [Environment Configuration](#environment-configuration)
10. [Testing Guidelines](#testing-guidelines)
11. [Common Tasks](#common-tasks)
12. [What's Implemented vs. What's Planned](#whats-implemented-vs-whats-planned)

---

## Project Overview

**GTM Copilot** is a full-stack application designed for Go-To-Market automation with AI capabilities. The project is currently at Phase 0 - a minimal starter template with Docker infrastructure established but minimal business logic implemented.

### Current State
- ✅ Docker Compose multi-service setup
- ✅ FastAPI backend skeleton with CORS
- ✅ PostgreSQL with pgvector extension
- ✅ Basic health check endpoint
- ❌ Frontend (Next.js) - empty directory
- ❌ Database models and migrations
- ❌ API routes and business logic
- ❌ Testing infrastructure

---

## Architecture & Technology Stack

### Three-Tier Microservices Architecture

```
┌─────────────────┐
│   Next.js Web   │  Port 3000 (not implemented)
│   (Frontend)    │
└────────┬────────┘
         │ HTTP
         ▼
┌─────────────────┐
│  FastAPI API    │  Port 8000
│   (Backend)     │
└────────┬────────┘
         │ SQLAlchemy
         ▼
┌─────────────────┐
│  PostgreSQL 16  │  Port 5432
│   + pgvector    │
└─────────────────┘
```

### Technology Stack

#### Backend (`/api`)
- **Language**: Python 3.12
- **Framework**: FastAPI (async, automatic OpenAPI docs)
- **ASGI Server**: Uvicorn with standard extras
- **ORM**: SQLAlchemy (planned, not implemented)
- **Database Driver**: psycopg (binary) - modern psycopg3
- **Migrations**: Alembic (planned, not configured)
- **Validation**: Pydantic
- **Configuration**: python-dotenv

#### Frontend (`/web`)
- **Framework**: Next.js (planned, not implemented)
- **Status**: Empty directory

#### Database
- **Engine**: PostgreSQL 16
- **Extension**: pgvector (for vector embeddings/semantic search)
- **Purpose**: Enables AI/ML features with vector similarity search

#### Infrastructure
- **Containerization**: Docker + Docker Compose
- **Services**: 3 containers (db, api, web)
- **Development**: Hot reload enabled on both API and web

---

## Directory Structure

```
/home/user/GTM_Copilot/
├── .env.example              # Environment variable template
├── .gitignore               # Git ignore patterns
├── README.md                # Project readme (minimal)
├── docker-compose.yml       # Multi-service orchestration
│
├── api/                     # FastAPI Backend
│   ├── Dockerfile          # Python 3.12 container
│   ├── requirements.txt    # Python dependencies
│   └── app/
│       └── main.py         # FastAPI entry point (8 lines)
│
└── web/                     # Next.js Frontend (EMPTY)
```

### Expected API Structure (Not Yet Implemented)

```
api/app/
├── main.py              # ✅ Application entry (exists)
├── config.py            # ❌ Configuration settings
├── database.py          # ❌ Database connection/session
├── models/              # ❌ SQLAlchemy ORM models
│   └── __init__.py
├── schemas/             # ❌ Pydantic request/response schemas
│   └── __init__.py
├── routers/             # ❌ API route handlers
│   └── __init__.py
├── services/            # ❌ Business logic layer
│   └── __init__.py
├── dependencies.py      # ❌ FastAPI dependencies
└── alembic/             # ❌ Database migrations
    └── versions/
```

### Expected Frontend Structure (Not Yet Implemented)

```
web/
├── Dockerfile           # ❌ Node.js container
├── package.json         # ❌ Node dependencies
├── next.config.js       # ❌ Next.js configuration
├── tsconfig.json        # ❌ TypeScript config (if using TS)
├── app/                 # ❌ Next.js 13+ app directory
│   ├── layout.tsx
│   ├── page.tsx
│   └── api/
└── components/          # ❌ React components
```

---

## Development Workflow

### Starting the Development Environment

```bash
# 1. Copy environment template
cp .env.example .env

# 2. Edit .env with your configuration (if needed)
nano .env

# 3. Start all services
docker-compose up

# 4. Start in detached mode
docker-compose up -d

# 5. View logs
docker-compose logs -f

# 6. Stop services
docker-compose down
```

### Container Names
- `gc_db` - PostgreSQL database
- `gc_api` - FastAPI backend
- `gc_web` - Next.js frontend

### Service Dependencies
1. **Database starts first** with health checks (20 retries, 5s interval)
2. **API starts after database** is healthy
3. **Web starts after API** is available

### Hot Reload Configuration
- **API**: `uvicorn --reload` watches `/api` directory
- **Web**: `npm run dev` watches `/web` directory (when implemented)
- **Volumes**: Local directories mounted for instant code changes

### Accessing Services
- **Frontend**: http://localhost:3000 (when implemented)
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs (Swagger UI)
- **API Redoc**: http://localhost:8000/redoc
- **Health Check**: http://localhost:8000/health

### Database Access

```bash
# Connect to PostgreSQL
docker exec -it gc_db psql -U gtm_user -d gtm_local

# Run SQL commands
docker exec -it gc_db psql -U gtm_user -d gtm_local -c "SELECT version();"
```

---

## Key Conventions

### Naming Conventions

#### Containers & Services
- **Prefix**: `gc_` (GTM Copilot)
  - `gc_db` - database container
  - `gc_api` - API container
  - `gc_web` - web container

#### Database
- **Prefix**: `gtm_` for database objects
  - Database: `gtm_local`
  - User: `gtm_user`
  - Password: `gtm_pass` (local dev only)

#### Python Code Style
Follow PEP 8 and FastAPI conventions:
- **Modules**: lowercase with underscores (`user_service.py`)
- **Classes**: PascalCase (`UserModel`, `UserSchema`)
- **Functions**: lowercase with underscores (`get_user()`)
- **Constants**: UPPERCASE (`DATABASE_URL`)

#### API Endpoints
- Use RESTful conventions
- Plural nouns for resources (`/users`, `/products`)
- Version endpoints if needed (`/v1/users`)

### Code Organization Patterns

#### Separation of Concerns
When implementing features, follow this structure:

1. **Models** (`models/`) - Database schema (SQLAlchemy)
2. **Schemas** (`schemas/`) - Request/response validation (Pydantic)
3. **Routers** (`routers/`) - HTTP endpoints (FastAPI)
4. **Services** (`services/`) - Business logic
5. **Dependencies** (`dependencies.py`) - Shared dependencies

#### Example Flow
```
HTTP Request → Router → Service → Model → Database
                 ↓         ↓        ↓
              Schema   Business   ORM
                      Validation  Logic
```

### Docker Best Practices

#### Volume Mounting
- **Source code**: Mount for hot reload
- **node_modules**: Separate volume to avoid conflicts
- **Database**: Persistent volume `db_data`

#### Health Checks
Always configure health checks for database dependencies:
```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 5s
  timeout: 5s
  retries: 20
```

#### Service Dependencies
Use `depends_on` with health conditions:
```yaml
depends_on:
  db:
    condition: service_healthy
```

### CORS Configuration

Current setup in `/api/app/main.py:6-17`:
- Reads allowed origins from `ALLOWED_ORIGINS` env var
- Comma-separated list support
- Defaults to `http://localhost:3000`
- Wide-open in development (credentials, all methods, all headers)

**When modifying CORS**:
- Keep localhost:3000 for local development
- Add production domains to `.env`
- Consider tightening for production

---

## Database & Models

### PostgreSQL + pgvector

The database uses **pgvector extension** for vector similarity search, indicating planned AI/ML features.

#### Connection String
```python
# From .env.example
DATABASE_URL=postgresql+psycopg://gtm_user:gtm_pass@db:5432/gtm_local
```

**Important**: Uses Docker service name `db`, not `localhost`

### Alembic Migrations (Not Yet Configured)

When implementing database migrations:

```bash
# Initialize Alembic (inside API container)
docker exec -it gc_api alembic init alembic

# Create migration
docker exec -it gc_api alembic revision --autogenerate -m "description"

# Apply migrations
docker exec -it gc_api alembic upgrade head

# Rollback
docker exec -it gc_api alembic downgrade -1
```

### Expected Database Structure

When implementing models, create:

```python
# api/app/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Vector Search Example (When Implementing)

```python
# Example using pgvector for semantic search
from pgvector.sqlalchemy import Vector
from sqlalchemy import Column, Integer, Text

class Document(Base):
    __tablename__ = "documents"

    id = Column(Integer, primary_key=True)
    content = Column(Text)
    embedding = Column(Vector(1536))  # OpenAI embeddings

    # Query similar documents
    # session.query(Document).order_by(
    #     Document.embedding.cosine_distance(query_embedding)
    # ).limit(5)
```

---

## API Structure

### Current Implementation

**File**: `/api/app/main.py`

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import os

app = FastAPI(title="GTM Copilot API")

# CORS configuration
allowed = os.getenv("ALLOWED_ORIGINS", "http://localhost:3000")
origins = [o.strip() for o in allowed.split(",") if o.strip()]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins or ["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/health")
def health():
    return {"status": "ok"}
```

### Current Endpoints
- `GET /health` - Health check (returns `{"status": "ok"}`)

### Adding New Endpoints

When implementing new routes, follow this pattern:

1. **Create router** (`api/app/routers/users.py`):
```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from ..database import get_db
from ..schemas import UserCreate, UserResponse
from ..services import user_service

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserResponse)
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    return user_service.create_user(db, user)
```

2. **Register router** in `main.py`:
```python
from app.routers import users

app.include_router(users.router)
```

3. **Create schemas** (`api/app/schemas/user.py`):
```python
from pydantic import BaseModel, EmailStr

class UserBase(BaseModel):
    email: EmailStr
    name: str

class UserCreate(UserBase):
    password: str

class UserResponse(UserBase):
    id: int

    class Config:
        from_attributes = True  # For SQLAlchemy compatibility
```

4. **Create service** (`api/app/services/user_service.py`):
```python
from sqlalchemy.orm import Session
from ..models import User
from ..schemas import UserCreate

def create_user(db: Session, user: UserCreate):
    db_user = User(**user.dict())
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

### FastAPI Features to Utilize
- **Automatic OpenAPI docs**: Available at `/docs` and `/redoc`
- **Async support**: Use `async def` for I/O-bound operations
- **Dependency injection**: Use `Depends()` for shared logic
- **Background tasks**: Use `BackgroundTasks` for async operations
- **WebSockets**: Supported natively for real-time features

---

## Frontend Structure

### Current Status
The `/web` directory is **empty** and needs complete setup.

### Setting Up Next.js Frontend

When implementing the frontend, create these files:

#### 1. Dockerfile (`web/Dockerfile`)
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev", "--", "-p", "3000", "--hostname", "0.0.0.0"]
```

#### 2. Package.json (`web/package.json`)
```json
{
  "name": "gtm-copilot-web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev -p 3000 --hostname 0.0.0.0",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  }
}
```

**Note**: The `--hostname 0.0.0.0` flag is required for Docker networking (mentioned in `/api/Dockerfile:11`).

#### 3. Next.js Configuration (`web/next.config.js`)
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // API base URL from environment
  env: {
    API_BASE_URL: process.env.NEXT_PUBLIC_API_BASE || 'http://localhost:8000',
  },
}

module.exports = nextConfig
```

### API Communication

The frontend should communicate with the backend using the `NEXT_PUBLIC_API_BASE` environment variable (set in `docker-compose.yml:46`).

Example API client:
```typescript
// lib/api.ts
const API_BASE = process.env.NEXT_PUBLIC_API_BASE || 'http://localhost:8000';

export async function fetchAPI(endpoint: string, options?: RequestInit) {
  const response = await fetch(`${API_BASE}${endpoint}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  });

  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }

  return response.json();
}
```

---

## Environment Configuration

### Environment Variables

**File**: `.env.example` (copy to `.env`)

```bash
# Ports
WEB_PORT=3000
API_PORT=8000
POSTGRES_PORT=5432

# Local DB credentials
POSTGRES_DB=gtm_local
POSTGRES_USER=gtm_user
POSTGRES_PASSWORD=gtm_pass

# API config
ALLOWED_ORIGINS=http://localhost:3000

# SQLAlchemy URL for the API service
DATABASE_URL=postgresql+psycopg://gtm_user:gtm_pass@db:5432/gtm_local
```

### Configuration Best Practices

1. **Never commit `.env`** - It's gitignored
2. **Use `.env.example`** as template with dummy values
3. **Document all variables** in `.env.example`
4. **Docker service names** - Use `db` not `localhost` in `DATABASE_URL`
5. **Production secrets** - Use proper secret management (not `.env`)

### Adding New Environment Variables

1. Add to `.env.example` with documentation
2. Add to `docker-compose.yml` `env_file` (already configured)
3. Access in Python: `os.getenv("VAR_NAME")`
4. Access in Next.js: Prefix with `NEXT_PUBLIC_` for client-side

---

## Testing Guidelines

### Current Status
**No testing infrastructure implemented yet.**

### Recommended Testing Setup

#### Backend Testing (pytest)

Add to `api/requirements.txt`:
```
pytest
pytest-asyncio
httpx  # For async test client
```

Create test structure:
```
api/tests/
├── __init__.py
├── conftest.py          # Shared fixtures
├── test_main.py         # Test health endpoint
├── test_models.py       # Test database models
├── test_routers.py      # Test API endpoints
└── test_services.py     # Test business logic
```

Example test (`api/tests/test_main.py`):
```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_health_endpoint():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

Run tests:
```bash
docker exec -it gc_api pytest
```

#### Frontend Testing (Jest + React Testing Library)

Add to `web/package.json`:
```json
"devDependencies": {
  "jest": "^29.0.0",
  "@testing-library/react": "^14.0.0",
  "@testing-library/jest-dom": "^6.0.0"
}
```

---

## Common Tasks

### Task 1: Adding a New API Endpoint

```bash
# 1. Create model (if needed)
# Edit: api/app/models/resource.py

# 2. Create schema
# Edit: api/app/schemas/resource.py

# 3. Create service
# Edit: api/app/services/resource_service.py

# 4. Create router
# Edit: api/app/routers/resource.py

# 5. Register router in main.py
# Edit: api/app/main.py

# 6. Test at http://localhost:8000/docs
```

### Task 2: Database Migration

```bash
# 1. Create or modify model
# Edit: api/app/models/

# 2. Generate migration
docker exec -it gc_api alembic revision --autogenerate -m "add resource table"

# 3. Review migration
# Edit: api/alembic/versions/xxxxx_add_resource_table.py

# 4. Apply migration
docker exec -it gc_api alembic upgrade head

# 5. Verify in database
docker exec -it gc_db psql -U gtm_user -d gtm_local -c "\dt"
```

### Task 3: Adding a Frontend Page

```bash
# 1. Create page component
# Create: web/app/page-name/page.tsx

# 2. Create necessary components
# Create: web/components/PageComponent.tsx

# 3. Add API integration
# Edit: web/lib/api.ts

# 4. Test at http://localhost:3000/page-name
```

### Task 4: Viewing Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f api
docker-compose logs -f web
docker-compose logs -f db

# Last 100 lines
docker-compose logs --tail=100 api
```

### Task 5: Restarting Services

```bash
# Restart all
docker-compose restart

# Restart specific service
docker-compose restart api

# Rebuild and restart (after dependency changes)
docker-compose up --build
```

### Task 6: Database Backup/Restore

```bash
# Backup
docker exec gc_db pg_dump -U gtm_user gtm_local > backup.sql

# Restore
cat backup.sql | docker exec -i gc_db psql -U gtm_user gtm_local
```

---

## What's Implemented vs. What's Planned

### ✅ Implemented (Phase 0)

#### Infrastructure
- [x] Docker Compose configuration
- [x] PostgreSQL with pgvector
- [x] FastAPI application skeleton
- [x] Python 3.12 Docker container
- [x] Hot reload for API
- [x] Health check endpoint
- [x] CORS middleware
- [x] Environment configuration
- [x] Git repository with .gitignore

#### Dependencies Installed
- [x] FastAPI, Uvicorn
- [x] SQLAlchemy, psycopg
- [x] Alembic
- [x] Pydantic, python-dotenv

### ❌ Not Yet Implemented

#### Backend
- [ ] Database models (SQLAlchemy)
- [ ] Alembic migrations setup
- [ ] API routers beyond health check
- [ ] Pydantic schemas for requests/responses
- [ ] Business logic services
- [ ] Database session management
- [ ] Authentication/authorization
- [ ] Error handling middleware
- [ ] Logging configuration
- [ ] Testing infrastructure (pytest)

#### Frontend
- [ ] Next.js application setup (completely empty)
- [ ] Dockerfile for web service
- [ ] package.json and dependencies
- [ ] React components
- [ ] API integration
- [ ] UI/UX design
- [ ] State management
- [ ] Frontend testing

#### Features
- [ ] User management
- [ ] GTM-specific functionality
- [ ] AI/ML features (pgvector usage)
- [ ] Vector search implementation
- [ ] Any business logic

#### DevOps
- [ ] CI/CD pipeline
- [ ] Production deployment config
- [ ] Environment-specific configs (dev/staging/prod)
- [ ] Monitoring and logging
- [ ] Security hardening

---

## Git Workflow

### Current Branch
Working on: `claude/claude-md-mibtqebcqwvgj9re-01Df9JwRHYg1D6n8JaRjxkTi`

### Commit Message Conventions

Follow conventional commits:
```
feat: Add user authentication endpoint
fix: Resolve CORS issue with production domain
docs: Update API documentation
refactor: Simplify database connection logic
test: Add tests for user service
chore: Update dependencies
```

### Making Changes

```bash
# 1. Make changes to code

# 2. Stage changes
git add .

# 3. Commit with descriptive message
git commit -m "feat: Add user registration endpoint"

# 4. Push to branch
git push -u origin claude/claude-md-mibtqebcqwvgj9re-01Df9JwRHYg1D6n8JaRjxkTi
```

### Important Git Notes
- Use retry logic (4 attempts with exponential backoff) for network issues
- Branch names should start with `claude/` and end with session ID
- Never force push to main/master

---

## Troubleshooting

### Common Issues

#### Database Connection Fails
```bash
# Check database is healthy
docker-compose ps

# View database logs
docker-compose logs db

# Ensure DATABASE_URL uses 'db' not 'localhost'
DATABASE_URL=postgresql+psycopg://gtm_user:gtm_pass@db:5432/gtm_local
```

#### API Not Accessible
```bash
# Check API logs
docker-compose logs api

# Verify port mapping
docker-compose ps

# Ensure uvicorn binds to 0.0.0.0 not 127.0.0.1
```

#### CORS Errors
```bash
# Check ALLOWED_ORIGINS in .env
ALLOWED_ORIGINS=http://localhost:3000

# For multiple origins (comma-separated)
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:3001
```

#### Changes Not Reflecting
```bash
# For Python dependency changes
docker-compose up --build api

# For code changes (should auto-reload)
docker-compose logs -f api  # Check for reload messages

# For Docker configuration changes
docker-compose down
docker-compose up --build
```

---

## Additional Resources

### FastAPI Documentation
- Official Docs: https://fastapi.tiangolo.com/
- Tutorial: https://fastapi.tiangolo.com/tutorial/
- Async: https://fastapi.tiangolo.com/async/

### Next.js Documentation
- Official Docs: https://nextjs.org/docs
- Learn: https://nextjs.org/learn

### PostgreSQL & pgvector
- PostgreSQL: https://www.postgresql.org/docs/
- pgvector: https://github.com/pgvector/pgvector

### SQLAlchemy & Alembic
- SQLAlchemy: https://docs.sqlalchemy.org/
- Alembic: https://alembic.sqlalchemy.org/

---

## Quick Reference

### Ports
- **3000**: Next.js frontend
- **8000**: FastAPI backend
- **5432**: PostgreSQL database

### Container Names
- **gc_web**: Frontend container
- **gc_api**: Backend container
- **gc_db**: Database container

### Important Files
- **docker-compose.yml**: Service orchestration
- **.env**: Environment variables (gitignored)
- **api/app/main.py**: FastAPI entry point
- **api/requirements.txt**: Python dependencies

### Key Commands
```bash
# Start everything
docker-compose up

# Stop everything
docker-compose down

# Rebuild after changes
docker-compose up --build

# View logs
docker-compose logs -f [service]

# Execute in container
docker exec -it [container] [command]

# Database access
docker exec -it gc_db psql -U gtm_user -d gtm_local

# Run tests (when implemented)
docker exec -it gc_api pytest
```

---

## For AI Assistants: Key Reminders

1. **Project is Phase 0**: Infrastructure only, minimal implementation
2. **Frontend is empty**: Need complete Next.js setup before building features
3. **Use Docker service names**: `db` not `localhost` in connection strings
4. **Hot reload enabled**: Code changes reflect immediately
5. **CORS configured**: Remember to update ALLOWED_ORIGINS for new domains
6. **pgvector available**: Database ready for AI/ML vector operations
7. **Follow conventions**: RESTful APIs, separation of concerns, PEP 8
8. **Test endpoints**: Use http://localhost:8000/docs for API testing
9. **Check dependencies**: Build container after changing requirements.txt
10. **Commit frequently**: Clear messages, push to correct branch

---

**End of CLAUDE.md**
