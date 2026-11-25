---
name: "before-sunrise"
description: "Use before releasing a system to production: read-only audit of security (secrets, rate limits, access control, hidden behavior and backdoors, AI usage, billing) and state machines. Triggers: pre-release audit, production readiness, yayın öncesi kontrol, canlıya almadan önce."
---

# Before Sunrise: Pre-Release Audit

Run this audit before a system goes live. It is read-only and applies to any language, framework, or stack, including web, API, desktop, and mobile applications. Detect the stack first and apply each check using that stack's own mechanisms.

The audit has two phases, reported together:

- Phase 1: Security and release readiness
- Phase 2: State machine discovery and analysis

Examine the entire codebase systematically. Do not limit the analysis to specific modules, entities, workflows, or business processes.

## Modes

- **Full audit (default):** both phases.
- **Quick audit:** Phase 1 only. Use it only when the user asks for a quick, security-only, or Phase 1 audit. State in the report that Phase 2 was not run.

## Working on Large Codebases

A large codebase can exceed what you can hold in context at once. Do not let the end of the audit get weaker than the start.

- If your environment allows writing files, write findings to a report file as you go, instead of keeping them all in memory until the end. Use `before-sunrise-report.md` in the project root unless the user names another location. If a report already exists there, read it completely first (see Previous Reports) before writing over it. This report file is the only file you may create or change; never edit application files.
- Work through the codebase area by area, and record each area as covered in the report before moving to the next.
- If you cannot finish, say exactly which areas were covered and which were not.

# Coverage and Evidence

A shallow or partial audit is the main way this skill fails. These rules make every gap visible in the report. Follow them in both modes.

## File Inventory

Before any check, build the inventory with a command, not from memory or impression: use the version control file list (such as `git ls-files`), or a recursive directory listing if there is no version control. Group the files by top-level directory and module, with file counts. Leave out generated, vendored-by-package-manager, and binary files, and list what you left out and why.

Put the inventory in the report. By the end, every group must be marked **Analyzed** or **Not analyzed** with a reason. A group cannot simply be missing.

## Required Searches

Reading files can miss things; a search over the whole codebase cannot. Run these searches with the stack's own tools and record, for each one, the exact command or pattern, the number of matches, and how the matches were reviewed. Adapt the patterns to the languages in use.

- Secrets in the current files: common key and token formats, `password`, `secret`, `token`, `api_key`, private key headers, and connection strings.
- Secrets in the full version control history, for example with `git log -p` filtered by the same patterns, and the list of environment or key files ever committed.
- Dynamic code execution and obfuscation: `eval`, dynamic function calls, shell execution, and base64 or hex decoding.
- Outbound network calls: HTTP client calls and hardcoded external URLs and domains.
- Raw queries and string-built queries.
- Unescaped output in templates.
- AI provider SDKs, AI API endpoints, and AI gateways.
- Payment provider SDKs, payment webhooks, and plan, price, coupon, credit, quota, and usage fields.
- Debug flags, test routes, and default credentials.

If a search cannot be run, record that as a coverage limitation.

## Endpoint Inventory

List every route, endpoint, and handler from the framework's own route list or route definitions, including API routes, webhooks, admin routes, and anything registered by packages. Do not sample.

For each one, record in a table: method and path, handler, authentication, authorization (policy, role, or ownership check), and rate limit. Mark each value **Yes**, **No**, or **Unclear**. Every **No** or **Unclear** on a route that needs protection is a finding or needs verification.

For systems without routes (desktop, CLI, background workers), list the equivalent entry points instead: commands, message handlers, IPC handlers, and scheduled tasks.

## Check Ledger

Keep a ledger with one row for every bullet in every Phase 1 section, and one row for each Phase 2 step. In a quick audit, mark the Phase 2 rows **Not run (quick audit)**. Each row has:

- The check, by section number and a short name.
- The result: **Issue found**, **No issue found**, or **Does not apply**.
- The evidence: the commands or searches run, the files read, and the match counts.

**No issue found** requires evidence. Without evidence, the result is **Not checked**, which counts as a coverage gap. **Does not apply** requires a one-line reason.

## Final Self-Check

Before writing the final report:

1. Compare the ledger with this skill and add any missing row.
2. Find every row marked **Not checked** or without evidence, go back, and complete it.
3. Find every inventory group marked **Not analyzed**, and analyze it or state why it cannot be analyzed.
4. If anything still cannot be completed, list it in the Executive Summary as a coverage gap. A release verdict of **ready** is not allowed while a check that could block the release is **Not checked**.

## Independent Verification

An agent checking its own work tends to confirm it. If your tool can start a separate agent (a sub-agent or task tool), run this step after the self-check and before the final report. Give the verifier only the code locations and claims below, not your reasoning or conclusions. The verifier is read-only and returns its results to you; only you write the report.

1. **Try to disprove every Critical and High finding.** For each one, the verifier reads the cited code and looks for anything that prevents the issue: a guard, middleware, policy, configuration, or framework default. Record each result as **Confirmed**, **Downgraded** (with the new severity and why), or **Rejected** (with the code that disproves it).
2. **Re-check a sample of "No issue found" rows.** The verifier independently re-checks at least 10 rows, or all of them if there are fewer, chosen from the ones that could block the release: authentication, authorization, tenant isolation, secrets, payments, and AI spending limits. Include rows from the endpoint inventory. Any issue it finds is a new finding, and the original ledger row is marked **Corrected by verification**.
3. If several re-checked rows turn out wrong, the original pass was not reliable for that area: re-check the rest of that area before giving a verdict.

Report the verification results in their own section. If your tool cannot start a separate agent, say so in the report: "Independent verification not run: no sub-agent support." Do not pretend to verify independently in the same context.

## Previous Reports

Before starting, look for an earlier report at the report location (`before-sunrise-report.md` in the project root, or the location the user names). If one exists, read it completely before writing anything, because this run will replace it.

- Use it to compare, never as evidence. Re-check every earlier finding against the current code; do not carry a finding or a "No issue found" result forward without checking it again.
- In the new report, classify every earlier finding as **Fixed** (with the code that fixes it), **Still open**, or **Changed** (different severity or location), and list new findings separately.
- Note whether the earlier report covered the same scope and mode. Results from a quick audit or a partial run say nothing about what it did not check.

If no earlier report exists, say so in the report.

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

## 1.9 Hidden Behavior and Backdoors

Apply this section to every audit and to the whole codebase, including vendored code copied into the repository. Do not wait to be told where the code came from: purchased scripts, templates, starter kits, AI-generated code, and code written in-house can all contain these problems.

- Search for outbound calls to servers the system does not need for its documented features: license checks, update checks, telemetry, analytics, or calls to an author's or vendor's domain. Report what data each call sends and whether it can be turned off. A call that sends user data, configuration, or secrets is critical.
- Search for obfuscated or encoded code: `eval`, dynamic code execution, base64 or hex decoding followed by execution, packed or minified server-side code, and encrypted source files.
- Check for remote code or configuration loading: code that downloads and runs scripts, plugins, or settings from an external URL at runtime.
- Check seed data, installers, and migrations for default admin users, demo accounts, known passwords, and test API keys that would reach production.
- Check for hidden routes, backdoor parameters, master passwords, or authentication bypasses that are not part of the documented features.
- Check whether the installer or setup wizard can be run again after installation, and whether it is removed or locked.

## 1.10 AI and LLM Integrations

Apply this section to every audit. Search the whole codebase for calls to language models or other AI services, such as provider SDKs, HTTP calls to AI APIs, AI gateways, embeddings, and vector stores. Do not wait to be told the system uses AI. If you find none, state in the report that this section does not apply and that no AI usage was found.

- Check for spending limits per user, tenant, and plan, and a global cap. Without them, one account can run up unlimited provider costs ("denial of wallet"). Check that limits are enforced on the server before the call, not counted after it.
- Check whether loops, retries, agents, or tool calls can multiply model calls without a bound.
- Check for prompt injection: whether user input, uploaded files, fetched web pages, emails, or other external content can override system instructions or trigger tools and actions the user is not allowed to perform.
- Check whether model output is treated as untrusted: escaped before rendering as HTML or Markdown, validated before use in queries, commands, file paths, or URLs.
- Check that AI provider keys stay on the server and never reach a browser bundle, mobile app, or desktop binary.
- Check whether prompts, model inputs, and outputs that contain personal or confidential data are logged, and where.
- Check whether one tenant's data can leak into another tenant's prompts through shared context, caches, embeddings, or vector stores.

## 1.11 Billing, Credits, and Quotas

Apply this section to every audit. Search the whole codebase for payments, plans, subscriptions, credits, quotas, and usage metering, such as payment provider SDKs and packages, payment webhooks, and plan, price, coupon, credit, or usage fields. Do not wait to be told the system charges users. If you find none, state in the report that this section does not apply and that no billing or quota logic was found.

- Check that plan limits and quotas are enforced on the server for every feature and endpoint they apply to, not only in the interface.
- Check that credit deduction and usage counting are atomic, so concurrent requests cannot spend the same credit twice or overrun a quota.
- Check that paid features are unlocked only after the payment is confirmed by a verified provider webhook or server-side check, not by a client redirect or a value sent from the client.
- Check that payment webhooks are idempotent, and that failed payments, refunds, chargebacks, cancellations, and downgrades remove access correctly.
- Check whether trial, coupon, and referral logic can be repeated or abused, for example by creating new accounts.
- Check that prices and amounts are calculated on the server and cannot be changed by the client.

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

Overall findings, architectural observations, and major risks. Give a release verdict (ready, ready with conditions, or not ready) and list every finding that blocks the release. List every coverage gap: inventory groups not analyzed and checks not checked.

### B. Application Coverage

The file inventory, with every group marked **Analyzed** or **Not analyzed** and a reason, plus all discovered modules and subsystems and any coverage limitations.

### C. Security and Release Readiness Findings

Results for each Phase 1 area, with severity and confidence (see Severity Scale), code references, impact, and recommended solutions. Mask all secret values. Include the endpoint inventory table and the results of the required searches.

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

### K. Check Ledger

The full ledger described in Coverage and Evidence, with a summary line: how many checks found an issue, found no issue, do not apply, and were not checked.

### L. Independent Verification

For each Critical and High finding: Confirmed, Downgraded, or Rejected, with the reason. For each re-checked "No issue found" row: whether it held. Or the statement that independent verification was not run, and why.

### M. Changes Since the Previous Report

Every earlier finding marked Fixed, Still open, or Changed, and the new findings listed separately. Or the statement that no earlier report was found.

# Important Constraints

- Analyze the entire application, not a predefined subset of modules.
- Do not stop after discovering the first few findings or state machines.
- Do not modify, refactor, or generate application code. The report file is the only file you may write.
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

# Severity Scale

Use these levels for every finding in both phases, so reports stay comparable across tools and runs:

- **Critical:** Exploitable now, or already exposed, with direct impact: leaked secrets, authentication bypass, access to other users' or tenants' data, remote code execution, unbounded spending, or paid access without payment. Blocks the release.
- **High:** Likely to cause a security incident, data loss, or financial loss under realistic conditions, but needs a specific situation or additional step. Blocks the release unless the user explicitly accepts the risk.
- **Medium:** A real weakness with limited impact or a hard path to exploit, or a gap that will cause incidents as the system grows. Fix soon after release.
- **Low:** Hardening, best practice, or a minor inconsistency with little direct impact.

Report confidence separately from severity: **Confirmed** (seen in code), **Likely** (strong evidence, one step unverified), or **Needs verification** (depends on something outside the repository).
