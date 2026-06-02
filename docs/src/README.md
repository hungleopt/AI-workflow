# FirstMile Internal Handbook

This book documents the current implementation of the FirstMile codebase as it exists in this repository. It is intended for engineers, QA, and support staff who need to understand how the solution is structured, how requests flow through the system, how frontend and backend delivery are connected, and what operational procedures already exist.

## Documentation principles

- Prefer implemented behavior over intended behavior.
- Call out open questions instead of inventing missing details.
- Treat application code, configuration, and pipeline YAML as the primary sources of truth.
- Keep architecture, feature behavior, and runbook content in the same navigation tree so onboarding and operations use the same handbook.

## Current coverage

This first implementation pass establishes the mdBook structure and documents:

- the solution and project layout
- the runtime bootstrap and configuration model
- service registration boundaries
- the main API and feature areas
- frontend build and integration packaging
- branching, deployment, DB export, and log retrieval workflows
- local development and team conventions

Subsequent passes should deepen feature-by-feature behavior, controller contracts, and operational troubleshooting.
