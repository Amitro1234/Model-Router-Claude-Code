# Task Examples by Model Tier

## Tier 1 — claude-haiku-4-5-20251001 (Light / Execution-only)

### File Operations
- "Read `requirements.txt` and list all dependencies"
- "Find all `.py` files that import `os.path`"
- "Show me the contents of `docker-compose.yml`"
- "List all environment variables used across the project"
- "How many lines of code are in `services/payment.py`?"

### Running Tools (output only, no interpretation)
- "Run `ruff check .` and show output"
- "Run `black --check .` and tell me which files would change"
- "Run `pytest --collect-only` and list all test names"
- "Run `mypy src/` and show the errors"
- "Run `git diff main` and show the raw diff"
- "Run `pre-commit run --all-files`"

### Simple Lookups
- "Does this project use SQLAlchemy?"
- "What Python version is in `.python-version`?"
- "Is there a `Makefile`? What targets does it have?"
- "What's the DB host in `config.yaml`?"
- "Are there any TODO comments in the codebase?"
- "Add a docstring to this one-liner function"

---

## Tier 2 — claude-sonnet-4-6 (Standard / All reasoning tasks)

### Writing New Code
- "Add input validation to the `create_user` endpoint"
- "Write a helper to paginate SQLAlchemy queries"
- "Add retry logic with exponential backoff to the API client"
- "Create a Pydantic model for the `OrderEvent` schema"
- "Add a CLI command using Click that accepts a `--dry-run` flag"

### Bug Fixing
- "This function returns `None` sometimes — it should always return a list"
- "Fix the `KeyError` on line 42 of `handlers.py`"
- "The `calculate_tax` function uses the wrong rate for EU orders"
- "The tests pass locally but fail in CI — investigate across the stack"
- "Our API returns 500 on some POST requests but not others — no clear pattern"
- "Memory usage grows over time — help diagnose where the leak might be"

### Refactoring
- "Extract the email-sending logic into a separate service class"
- "Replace `dict` with typed dataclasses in `models.py`"
- "Convert this callback-style code to async/await"
- "Add type hints throughout `utils.py`"
- "Refactor the auth module to use dependency injection across the whole service"
- "Migrate all raw SQL queries in the repo to SQLAlchemy ORM"
- "Replace the custom logging setup with structlog, consistently across all modules"

### Writing Tests
- "Write unit tests for `calculate_discount`"
- "Add a pytest fixture for a test database session"
- "Write parametrized tests for the `validate_email` function"

### Code Explanation & Review
- "Explain what this decorator does and how it works"
- "Walk me through the data flow in `process_payment()`"
- "Why does this list comprehension use a walrus operator?"
- "Review this PR — is the approach correct? Are there better alternatives?"
- "Is this abstraction layer warranted, or is it over-engineering?"
- "What are the failure modes of this implementation under load?"

### Integration Work
- "Integrate Celery into our existing FastAPI app for background tasks"
- "Add OpenTelemetry tracing to our service without breaking existing behavior"
- "Migrate from `requests` to `httpx` with full async support"

### Feature Design (within existing system)
- "Design the data flow for adding webhook support to our existing event system"
- "How should we add multi-tenancy to the current user model?"

---

## Tier 3 — claude-opus-4-7 (Heavy / Deep reasoning)

### Architecture
- "Design the data model for a multi-tenant SaaS platform from scratch"
- "How should we structure microservices to handle 10x current load?"
- "Should we use CQRS for the reporting module, and what are the trade-offs?"
- "Design an event-driven order processing system"

### Security
- "Review our authentication flow for vulnerabilities"
- "What are the OWASP risks in this API design?"
- "Audit our use of user-supplied input throughout the codebase"
- "Is our token handling compliant with OAuth 2.0 best practices?"

### Strategic/Architectural Decisions
- "Should we migrate from REST to GraphQL? Walk me through the trade-offs"
- "Evaluate Kafka vs. RabbitMQ for our use case"
- "Is it worth switching from SQLAlchemy to async-native Tortoise ORM?"
- "How should we approach breaking this monolith into services?"

### Deep Investigations (no clear starting point)
- "Performance degrades past ~500 concurrent users — we have no idea why"
- "Intermittent 500s with no clear pattern, across multiple services"
- "We have a race condition somewhere in the order pipeline — help investigate"
