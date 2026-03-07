# Shared

This directory is intentionally minimal at project start (YAGNI).

Shared utilities are extracted here only when the same code is genuinely needed by two or
more projects. Do not add code here speculatively.

## Candidate additions (when needed)

- `shared/python/` — common Pydantic models, logging config, service client base classes
- `shared/dotnet/` — shared .NET library for API contracts consumed by multiple services
