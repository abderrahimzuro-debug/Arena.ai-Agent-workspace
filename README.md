# Zuro Code — Project Workspace

> **Project status:** Planning / specification stage. This repository is the source of truth for the product blueprint and implementation instructions.

Zuro Code is a Windows desktop AI-assisted development environment. It aims to coordinate multiple AI models, local project tools, skills, web research, and Git workflows while keeping user control, security, and usage efficiency central.

## Start here

1. Read [the project blueprint](docs/blueprint/README.md).
2. Read [architecture and component boundaries](docs/blueprint/architecture.md).
3. Read [the implementation roadmap](docs/blueprint/roadmap.md).
4. Review [providers and integrations](docs/blueprint/providers-and-integrations.md).
5. Review [security and data handling](docs/blueprint/security-and-data.md).
6. Review [open decisions and risks](docs/blueprint/decisions-and-risks.md).

## Instructions for Arena.ai Agent

When inspecting this repository, treat `docs/blueprint/` as the product requirements and architectural reference. Before creating or changing application code:

- Read all blueprint documents.
- Preserve the distinction between confirmed decisions, proposed choices, and items requiring validation.
- Do not invent API free tiers, model capabilities, or provider limits. Verify them from official documentation and record the verification date.
- Keep secrets, API keys, GitHub tokens, and private user data out of source control.
- Implement a thin vertical slice first: desktop shell, local workspace, permission gateway, one provider adapter, and safe file read/write with diff review.
- Do not implement every planned integration at once. Follow the roadmap and acceptance criteria.
- If the repository gains application code, update the relevant architecture and decision documents when behavior changes.

## Current scope

This repository currently contains planning documents. It is not yet a completed or runnable Zuro Code application.
