---
name: "before-sunrise"
description: "Use before releasing a system to production: read-only audit of security (secrets, rate limits, access control) and state machines. Triggers: pre-release audit, production readiness, yayın öncesi kontrol, canlıya almadan önce."
---

# Before Sunrise: Pre-Release Audit

Run this audit before a system goes live. It is read-only and applies to any language, framework, or stack, including web, API, desktop, and mobile applications. Detect the stack first and apply each check using that stack's own mechanisms.

The audit has two phases, reported together:

- Phase 1: Security and release readiness
- Phase 2: State machine discovery and analysis

Examine the entire codebase systematically. Do not limit the analysis to specific modules, entities, workflows, or business processes.

# Phase 1: Security and Release Readiness

If a check does not apply to this system, say so briefly instead of skipping it silently.

## 1.1 Secrets

- Check whether environment files or other local configuration containing secrets are tracked in version control.
- Search the full version control history, not only the current state. A secret that was committed and later removed is still exposed and must be rotated.
- Search source code, configuration, scripts, test fixtures, seed data, documentation, and CI/CD files for hardcoded API keys, tokens, passwords, private keys, signing keys, and connection strings.
- Check everything distributed to users or third parties for embedded secrets: browser bundles and source maps, mobile app packages, desktop binaries, container images, and published packages. Anything shipped can be extracted, and obfuscation does not protect it. Such secrets must move behind a server.
- Check whether ignore rules cover environment files, keys, logs, and database dumps.
- Check whether logs or error responses can expose secrets or personal data.
- Always mask secret values in the report. Show only the type, the location, and at most the first few characters.

## 1.2 Rate Limiting and Abuse Protection

- Check whether limits exist on login, registration, password reset, OTP or verification codes, public APIs, webhooks, file uploads, outbound email or SMS triggers, search, exports, and other expensive endpoints.
- Check whether each limit uses the right key (IP, user, API key, tenant, or a combination) and whether its values suit the endpoint.
- Check whether the real client IP is resolved correctly behind proxies, load balancers, or CDNs. A wrong setup can put all users in one bucket or let clients spoof the header.
- Check whether the limit store works across multiple instances.
- Check for brute-force protection and account enumeration through differing responses.

## 1.3 Authentication and Authorization

- Check every route, endpoint, and handler for missing authentication or authorization.
- Check for direct access to other users' records by identifier (IDOR).
- In multi-tenant systems, check tenant isolation in queries, background jobs, caches, file storage, and exports.
- Check for privilege escalation through user-controlled fields such as role or owner.
- Check session and token handling: expiry, revocation, rotation, and invalidation on password change.
- Check that authorization is enforced on the server, not only hidden in the interface or client application.

## 1.4 Input and Integrations

- Check for injection risks: SQL or other query injection, command injection, template injection, and path traversal.
- Check for XSS and missing CSRF protection where applicable.
- Check for mass assignment or over-posting of unintended fields.
- Check file uploads: type and size validation, storage location, whether uploaded files can execute, and public access.
- Check for SSRF where the system fetches user-supplied URLs.
- Check signature verification, replay protection, and duplicate handling on incoming webhooks.
- Check CORS rules for overly broad origins, especially combined with credentials.

## 1.5 Production Configuration

- Check that debug mode and verbose error output are disabled for production.
- Check that development tools, profilers, test routes, seeders, and sample accounts are unreachable in production.
- Check for unprotected admin panels, health or metrics endpoints, API documentation, and directory listings.
- Check HTTPS enforcement, secure cookie flags, and security headers.
- Check for default credentials and publicly exposed sensitive files.
- Check that required environment variables are documented and that the application fails clearly when they are missing.

## 1.6 Dependencies

- Run the read-only audit tool of each package ecosystem in use, and report known vulnerabilities by severity.
- Identify abandoned or end-of-life runtimes and packages.
- Check that lock files are committed.
- If an audit tool cannot be run, document this as a coverage limitation.

## 1.7 Operational Readiness

- Check whether backups, database migrations, queues, and scheduled tasks are configured for production, as far as the repository shows.
- Check whether errors are logged and reported somewhere that will be seen.
- Label anything that cannot be verified from the repository as requiring manual verification.

## 1.8 Mobile and Desktop Clients

Apply this section only when the system includes a mobile or desktop client.

- Check whether tokens and sensitive data use the platform's secure storage (such as iOS Keychain, Android Keystore, or the operating system's credential store) instead of plain preferences, files, or local databases.
- Check that release builds disable debugging and remove verbose logging.
- Check that release builds do not allow unencrypted network traffic.
- Check whether TLS certificate validation has been disabled or weakened anywhere.
- Check whether requested permissions are all actually needed.
- Check that deep links, custom URL schemes, and exported components validate their input and cannot trigger sensitive actions unauthenticated.
- Check whether older client versions keep working or can be forced to update when the API changes.

# Phase 2: State Machine Discovery and Analysis

Discover all explicit and implicit state machines across the application.

## 2.1 Full Application Discovery

- Identify all modules, packages, services, components, and subsystems.
- Analyze backend, frontend, background jobs, scheduled tasks, event handlers, integrations, and database structures.
- Examine relationships and dependencies between modules.
- Include shared infrastructure, common services, and cross-module workflows.
- Do not assume that state machines are explicitly defined or named.

## 2.2 State Machine Identification

Discover all explicit and implicit state machines, including those represented by:

- Status fields, enums, constants, and configuration values.
- Boolean flags and combinations of multiple fields.
- Conditional business logic and validation rules.
- Database records and relationships.
- Events, listeners, observers, hooks, and callbacks.
- Background jobs, queues, and scheduled processes.
- User interactions and frontend component states.
- Authentication, authorization, and session lifecycles.
- External integrations and asynchronous operations.
- Multi-step workflows and distributed processes.

Identify state machines even when their logic is scattered across multiple files, layers, or modules.

Distinguish genuine lifecycle/state machines from ordinary variables and temporary UI state.

## 2.3 Detailed Analysis

For every discovered state machine, document:

1. Name and purpose.
2. Owning module or subsystem.
3. All known states and their definitions.
4. Initial and terminal states, if applicable.
5. All valid state transitions.
6. Actions, events, or conditions triggering each transition.
7. Guards, permissions, validations, and business rules.
8. Side effects associated with transitions.
9. Relevant files, classes, methods, and line numbers.
10. Dependencies on other state machines or modules.

Distinguish confirmed behavior from inferred behavior.

## 2.4 Risk and Consistency Analysis

Identify potential problems, including:

- Invalid or unrestricted state transitions.
- Missing transition guards or authorization checks.
- Unreachable or orphaned states.
- Dead-end workflows.
- Contradictory or overlapping states.
- Inconsistent state representations.
- Race conditions and concurrent updates.
- Missing database transactions or atomicity guarantees.
- Partial failures and inconsistent data.
- Non-idempotent transitions.
- Duplicate event processing.
- Missing rollback, retry, or recovery mechanisms.
- Frontend/backend state inconsistencies.
- Cross-module state synchronization problems.
- Business rules implemented inconsistently across different locations.

Only report risks supported by code evidence. Clearly label anything that requires further verification.

## 2.5 Architecture Evaluation

Evaluate how state transitions are currently implemented.

Identify:

- State logic duplicated across the application.
- Transitions scattered across controllers, services, models, jobs, or other layers.
- Modules that would benefit from centralized transition management.
- State machines that are already sufficiently structured.
- Opportunities to improve consistency, testability, maintainability, and reliability.

Do not recommend introducing a state machine library or abstraction unless there is a clear technical benefit.

Prefer solutions compatible with the application's existing architecture and conventions.

# Required Report

Produce a structured Markdown report containing:

### A. Executive Summary

Overall findings, architectural observations, and major risks. Give a release verdict (ready, ready with conditions, or not ready) and list every finding that blocks the release.

### B. Application Coverage

All discovered modules and subsystems, including which areas were analyzed and any coverage limitations.

### C. Security and Release Readiness Findings

Results for each Phase 1 area, with severity, code references, impact, and recommended solutions. Mask all secret values.

### D. State Machine Inventory

A comprehensive table containing:

- Module
- State machine
- States
- Transition count
- Implementation locations
- Risk level
- Confidence level

### E. Detailed State Machine Documentation

Individual analysis of every discovered state machine, including transition tables and Mermaid state diagrams where useful.

### F. Cross-Module Dependencies

State transitions that affect other modules, services, or shared resources.

### G. State Machine Problems and Risks

Prioritized findings with severity, code references, impact, and recommended solutions.

### H. Architecture Recommendations

Practical improvements aligned with the existing application architecture.

### I. Implementation Roadmap

Prioritized recommendations from both phases, divided into:

- Critical corrections (must be resolved before release)
- High-priority improvements
- Optional architectural refinements

### J. Test Coverage Recommendations

Identify missing or insufficient tests for important transitions, invalid transitions, concurrency, failure recovery, cross-module consistency, and security controls.

# Important Constraints

- Analyze the entire application, not a predefined subset of modules.
- Do not stop after discovering the first few findings or state machines.
- Do not modify, refactor, or generate application code.
- Do not modify database schemas or configuration.
- Do not introduce new dependencies.
- Do not send secrets or code to external services.
- Do not test against live production systems.
- Do not assume that every state machine needs refactoring.
- Reference actual code locations for all findings.
- Do not invent states, transitions, business rules, or vulnerabilities.
- Clearly distinguish verified findings from hypotheses.
- If a module or check cannot be fully analyzed, explicitly document the limitation.
- Continue systematically until all discoverable modules and subsystems have been examined.

The objective is to create a complete, evidence-based picture of the system's release readiness and identify genuine weaknesses without making unnecessary changes.