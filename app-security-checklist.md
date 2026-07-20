# Universal Security Standard for AI-Assisted Applications

> **Document version:** 1.0
> **Intended location:** Store this file in the repository so every human and AI developer can review it before design, implementation, security-sensitive changes, and release.

> **Purpose:** This file is a reusable, technology-neutral security baseline for applications built or modified with AI coding agents. It applies to web apps, APIs, mobile/desktop clients, SaaS products, serverless systems, AI features, background workers, integrations, and supporting infrastructure.
>
> **Primary instruction to the developer agent:** For every control in this document, determine whether it applies to the application currently under development. For each applicable control, inspect the real implementation, fix the weakness, and verify the mitigation with evidence. Do not merely recommend a fix, add a comment, hide a UI element, or mark an item complete based on assumption. Security controls must be enforced at the correct trust boundary—normally the backend, database, identity provider, storage layer, infrastructure, or external provider configuration.
>
> **This is a baseline, not a ceiling.** It does not replace application-specific threat modeling, secure design review, code review, automated security testing, dependency and secret scanning, monitoring, incident response, privacy review, or professional penetration testing for high-risk systems.

---

## 1. Mandatory rules for the AI developer agent

The following rules govern how this checklist must be used.

### 1.1 Required status values

Use one of these values for every reviewed control:

- `[ ]` **OPEN** — applicable but not yet implemented or verified.
- `[x]` **VERIFIED** — implemented and verified with reproducible evidence.
- `[~]` **PARTIAL** — partially implemented, incompletely tested, or dependent on unfinished work.
- `[N/A]` **NOT APPLICABLE** — demonstrably irrelevant to this application. One compact justification may cover a contiguous ID range or an entire control family when the same verified absence makes every control in the group inapplicable.
- `[BLOCKED]` — cannot be completed because of a concrete dependency or access limitation; identify the blocker and the safest temporary state.
- `[RISK ACCEPTED]` — an authorized human owner explicitly accepted the residual risk, with scope, reason, compensating controls, owner, and expiration date. An AI agent must never accept risk on behalf of a human.

### 1.2 Token-efficient applicability and reporting protocol

The agent must evaluate every control, but it must not narrate every evaluation. Exhaustive review and concise output are compatible. Use the following protocol unless a human explicitly requests a full control-by-control audit artifact.

#### A. Build one scope map, then gate controls

Inspect the repository, manifests, routes, schemas, infrastructure, deployment files, and available provider configuration once. Record a compact capability map such as:

```text
auth=on; browser=on; api=on; tenants=off; files=off; outbound_fetch=off;
payments=off; personal_data=on; ai=off; native_client=off; cloud=unknown
```

Use positive evidence to turn a capability `on`, verified absence to turn it `off`, and `unknown` when the available evidence is insufficient. `unknown` never justifies `N/A`; inspect further or mark affected controls `OPEN`/`BLOCKED`. A section heading, framework convention, missing documentation, or lack of an obvious file is not by itself proof of absence.

Common grouping gates are listed below. They are shortcuts for applicability only, not permission to skip inspection.

| If this capability is absent | Controls that may share one grouped `N/A` decision | Minimum absence claim to verify |
|---|---|---|
| Account authentication and identity lifecycle | `AUTHN-*` | No login, accounts, privileged identity, registration, recovery, linking, or impersonation flow |
| Browser-delivered UI or content | `WEB-*` | No web page, WebView, browser extension surface, or browser-consumed response |
| Network API or streaming interface | `API-*` | No HTTP/RPC/GraphQL/WebSocket/streaming API, including internal or management endpoints |
| File ingestion or delivery | `FILE-*` | No upload, import, attachment, archive extraction, user-influenced file parsing, generated download, or file serving |
| AI/model capability | `AI-*` | No model call, RAG/vector retrieval, model-controlled tool, agent memory, generated-code execution, or AI provider |
| Installed mobile/desktop client | `CLIENT-*` | No shipped native, hybrid, Electron/Tauri, or mobile/desktop client |
| Cloud/container/orchestrator resources | `CLOUD-*` | No cloud account resources, serverless functions, containers, clusters, or cloud-managed data/control plane in scope |

Mixed families such as `SESS-*`, `DATA-*`, `NET-*`, `BIZ-*`, `PRIV-*`, `SUPPLY-*`, and `TEST-*` usually need narrower ranges because one absent feature does not disable the whole family. For example:

```text
N/A SESS-02..06 — no cookies or server-side sessions; auth uses scoped service credentials (routes/auth config E2).
N/A SESS-14 — no SAML dependency, metadata, endpoint, or provider configuration (manifest + route inventory E1).
N/A FILE-* — no file intake, generated downloads, file parsers, or object/file serving (entry-point inventory E3).
```

Range notation such as `SESS-02..06` is inclusive. Do not combine noncontiguous IDs under a range; list them with commas or use separate rows.

Before grouping `N/A`, check for indirect implementations: framework middleware, provider dashboards, CI jobs, admin/support paths, preview environments, generated exports, logging/telemetry, transitive parsers, and dormant or feature-flagged code. If one control in a proposed range has a different trigger, split the range.

#### B. Default to delta reporting

Use **DELTA mode** by default:

- do not restate control text or produce one paragraph per control;
- do not list every `VERIFIED` control in conversation or in the completion report;
- report security findings, incomplete work, blockers, risk acceptances, changed controls, and release blockers;
- collapse `N/A` controls by shared cause using IDs/ranges plus one evidence-backed absence claim;
- place each item in only one report location; do not repeat grouped exclusions in the findings table or restate findings in the release section;
- summarize successful automated checks once, and cite an evidence file, command result, test name, or provider setting instead of repeating its output;
- keep progress updates to newly discovered risk, material scope changes, blockers, and completed milestones.

Use **FULL mode** only when explicitly requested for compliance, external audit, or a complete traceability matrix. FULL mode may emit one row per control. A normal implementation task or release review does not by itself require FULL mode.

For a review limited to a code change, first inspect the changed surfaces and their trust-boundary neighbors, then re-evaluate any capability gates affected by the change. Report only deltas. A production release still requires coverage of all controls.

#### C. Prove coverage without printing the checklist back

Maintain a terse internal or repository evidence ledger when durable evidence is needed. One evidence item may support multiple controls. Use control IDs plus pointers, for example:

```text
E1 routes+manifests: app/routes.ts, package-lock.json
E2 authz tests: tests/authz.test.ts::cross_tenant_denied (pass)
AUTHZ-01..04,07..08 [x] E2
FILE-* [N/A] E1 — no file capability found
```

In the completion report, provide coverage arithmetic rather than a list of passes:

```text
VERIFIED=<n>; PARTIAL=<n>; OPEN=<n>; N/A=<n>; BLOCKED=<n>; RISK_ACCEPTED=<n>; TOTAL=<sum>
```

Expand grouped ranges when calculating counts. In the unmodified version 1.0 of this document, `TOTAL` must equal **347**; update that expected total if project-specific controls are added or an explicitly approved baseline revision changes the list. This checksum detects accidentally omitted controls without spending output tokens enumerating them.

### 1.3 Evidence is required

A control is not `VERIFIED` unless the agent records evidence appropriate to that control, such as:

- affected files, modules, routes, database policies, infrastructure resources, or provider settings;
- tests added and their results;
- commands or scans run and summarized results;
- negative/adversarial test cases that failed safely;
- screenshots or configuration exports for external dashboards when code alone is insufficient;
- migration, rollback, and deployment notes where applicable.

Do not expose secrets, private keys, tokens, personal data, or exploitable production details in the evidence.

### 1.4 Prohibited agent behavior

- Do not mark a control complete because a framework “usually handles it.” Verify the actual configuration and version.
- Do not delete, weaken, rewrite, or mark controls `N/A` merely to reduce implementation work. Project-specific copies may add stricter controls; removing or relaxing a baseline control requires explicit human approval and documentation.
- Do not infer `N/A` from a product description alone. Verify the absence claim against the available implementation and configuration, and use `unknown`, `OPEN`, or `BLOCKED` when scope cannot be established.
- Do not rely on frontend validation, hidden buttons, route guards, prompt instructions, or client state as a security boundary.
- Do not weaken authentication, authorization, TLS, certificate validation, sandboxing, rate limits, security headers, logging, or validation merely to make the app work or make tests pass.
- Do not add fake controls, placeholder middleware, nonfunctional checks, hardcoded “temporary” secrets, or comments such as `TODO: secure later` in a release candidate.
- Do not silently ignore security errors. Fail safely, log appropriately, and preserve a usable operational signal.
- Do not invent custom cryptography, password storage, token formats, authentication protocols, or session systems when maintained standards and libraries exist.
- Do not assume development, preview, staging, demo, admin, internal, or test environments are harmless. Secure them and isolate them from production.
- Do not make irreversible or high-impact security changes without a rollback path and explicit documentation.
- Do not run load tests, fuzzing, exploit payloads, destructive tests, unsolicited scanning, or third-party/provider probes outside an explicitly authorized scope. Prefer isolated non-production targets with synthetic data and inert credentials; test production only with specific approval, bounded impact, monitoring, and a stop plan.

### 1.5 Release-blocking severity

Classify every discovered issue from demonstrated or reasonably supported **impact, exploitability, exposure, required privileges/user interaction, blast radius, and existing compensating controls**. Vulnerability names alone do not determine severity.

- **Critical:** exploitation is practical and could cause catastrophic or system-wide compromise, such as broad production control, signing-key compromise, cross-tenant compromise at scale, or unauthenticated access to highly sensitive data. **Release is blocked.**
- **High:** exploitation is practical and could cause major unauthorized access, privilege gain, account takeover, sensitive-data exposure, code execution, financial harm, or repeatable high-cost abuse. **Release is blocked until remediated and retested.**
- **Medium:** meaningful confidentiality, integrity, availability, privacy, or abuse impact, but exploitation requires significant preconditions or has a limited blast radius. Fix before release when practical; otherwise require documented, time-bounded human risk acceptance.
- **Low:** limited-impact or defense-in-depth weakness with no credible path to a higher impact under the assessed conditions. Track with an owner and due date.

If evidence is incomplete, record the uncertainty and use the higher plausible severity temporarily; do not inflate severity after evidence rules out the higher impact.

### 1.6 Required execution workflow

Before declaring the application secure enough to release, the agent must:

1. **Inventory the system:** entry points, users, roles, tenants, data, APIs, clients, databases, storage, queues, jobs, third parties, AI models/tools, cloud resources, and deployment environments.
2. **Identify trust boundaries:** browser-to-server, service-to-service, app-to-provider, tenant-to-tenant, user-to-admin, model-to-tool, public-to-private network, and CI/CD-to-production.
3. **Create or update a concise threat model:** assets, attackers, abuse cases, likely failure modes, and high-impact actions.
4. **Determine applicability:** build the capability map, evaluate every checklist section, and group only controls made inapplicable by the same verified absence; never skip a section only because its title appears unrelated.
5. **Implement controls:** prefer simple, maintained, secure-by-default mechanisms.
6. **Verify adversarially:** test how controls fail, not only how normal use succeeds.
7. **Run automated checks:** tests, dependency scanning, secret scanning, static analysis, infrastructure scanning, and relevant dynamic tests.
8. **Produce a concise security completion report:** use DELTA mode and the template near the end of this document unless FULL mode was explicitly requested.
9. **Re-run the review after material changes:** new authentication, new data, new provider, new AI tool, new upload type, new payment flow, new tenant model, or infrastructure change.

---

## 2. Security scope, architecture, and threat modeling

- [ ] **ARCH-01 — Document the architecture.** Maintain a current diagram or structured description of clients, services, databases, storage, queues, external providers, trust boundaries, data flows, administrative paths, and deployment environments.
- [ ] **ARCH-02 — Inventory all entry points.** Include web routes, APIs, GraphQL, WebSockets, webhooks, uploads, deep links, CLI commands, scheduled jobs, message consumers, admin panels, support tools, preview deployments, and legacy endpoints.
- [ ] **ARCH-03 — Identify protected assets.** Include credentials, sessions, API keys, personal data, financial records, uploaded files, proprietary data, source code, model prompts, vector stores, signing keys, logs, backups, and cloud control-plane access.
- [ ] **ARCH-04 — Identify actors and roles.** Include unauthenticated visitors, ordinary users, tenant admins, platform admins, support staff, developers, CI/CD identities, background workers, third parties, and AI agents.
- [ ] **ARCH-05 — Model abuse cases.** Consider account takeover, IDOR/BOLA, privilege escalation, tenant crossover, scraping, automation, fraud, cost exhaustion, data poisoning, prompt injection, insider misuse, supply-chain compromise, and denial of service.
- [ ] **ARCH-06 — Define security assumptions explicitly.** Examples: trusted identity provider, private network, provider webhook authenticity, immutable audit store, or single-tenant deployment. Verify each assumption or remove it.
- [ ] **ARCH-07 — Choose a security target appropriate to risk.** Raise assurance for systems handling payments, health, identity, children, regulated data, destructive actions, public uploads, privileged automation, or multi-tenant data.
- [ ] **ARCH-08 — Minimize attack surface.** Remove unused routes, ports, services, packages, permissions, features, sample accounts, debug tools, legacy APIs, and unnecessary data collection.
- [ ] **ARCH-09 — Design secure defaults.** New accounts and resources containing non-public data or capabilities must be private and least-privileged. Public repositories, buckets, projects, profiles, or tenant data require a deliberate product decision and verification that no sensitive material is exposed.
- [ ] **ARCH-10 — Define data classification.** At minimum classify public, internal, confidential, sensitive personal, authentication secret, and cryptographic key material, with handling rules for each.

---

## 3. Identity, registration, authentication, and account recovery

- [ ] **AUTHN-01 — Use a mature identity solution.** Prefer a maintained identity provider or framework. Do not create custom password hashing, OAuth, OIDC, SAML, passkey, MFA, session, or token-verification logic without a compelling reviewed need.
- [ ] **AUTHN-02 — Verify identity on the server.** Derive the authenticated principal from a validated server-side session or token, never from client-provided user IDs, email addresses, roles, tenant IDs, or headers that an untrusted client can set.
- [ ] **AUTHN-03 — Require phishing-resistant authentication for privileged access.** Prefer FIDO2/WebAuthn passkeys or security keys for platform admins, production access, financial operations, support impersonation, and other high-impact roles. If a system outside the team's control cannot support them, document the limitation and use the strongest available MFA plus compensating controls.
- [ ] **AUTHN-04 — Offer strong MFA to users where risk warrants.** Provide recovery codes or secure recovery mechanisms; protect MFA enrollment, replacement, and removal with recent authentication and notifications.
- [ ] **AUTHN-05 — Use safe password policy when passwords exist.** Permit long passwords and password managers, block commonly breached passwords, avoid composition rules that reduce usability, do not silently truncate, and do not require periodic rotation without compromise or policy need.
- [ ] **AUTHN-06 — Store passwords with an approved adaptive password hash.** Use the identity provider's secure implementation or a current configuration of Argon2id, scrypt, bcrypt, or PBKDF2 when platform/FIPS requirements call for it, with unique salts and reviewed work factors. Account for algorithm input limits without silent truncation. Never use fast general-purpose hashes or reversible encryption.
- [ ] **AUTHN-07 — Prevent account enumeration.** Use appropriately generic responses and similar timing for login, registration, recovery, invitation, and verification flows, while retaining useful internal logs.
- [ ] **AUTHN-08 — Rate-limit authentication flows.** Combine per-account, per-IP/network, device, and global controls as appropriate. Protect login, signup, email/SMS verification, recovery, MFA, magic links, and invitation acceptance.
- [ ] **AUTHN-09 — Defend against credential stuffing and brute force.** Use progressive delays, anomaly detection, breached-password checks, risk-based challenges, and safe lockout behavior that cannot be weaponized for denial of service.
- [ ] **AUTHN-10 — Secure email and phone verification.** Use single-purpose, short-lived, one-time tokens; bind them to the intended account and action; invalidate older tokens after success or resend.
- [ ] **AUTHN-11 — Secure password reset and account recovery.** Recovery must not be weaker than normal authentication. Do not use knowledge-based questions. Invalidate reset tokens and relevant sessions after recovery.
- [ ] **AUTHN-12 — Protect account identifier changes.** Require recent authentication for email/phone changes, confirm the new identifier, notify the old identifier, and provide a recovery path for unauthorized changes.
- [ ] **AUTHN-13 — Prevent unsafe account linking.** When linking social, enterprise, or password identities, prove control of both identities; never link solely because email strings match unless the provider's verification semantics are trusted and reviewed.
- [ ] **AUTHN-14 — Handle disabled or deleted accounts consistently.** Revoke active sessions, refresh tokens, API keys, device tokens, background jobs, invitations, and privileged access.
- [ ] **AUTHN-15 — Prevent insecure default or shared accounts.** Remove vendor defaults and sample users; prohibit shared administrator accounts except tightly controlled break-glass procedures.
- [ ] **AUTHN-16 — Protect support impersonation.** Require explicit privilege, reason, time limit, prominent indication, immutable audit logging, and restrictions on especially sensitive actions.

---

## 4. Authorization, permissions, and multi-tenancy

- [ ] **AUTHZ-01 — Default deny.** Access is denied unless a specific server-side rule grants it.
- [ ] **AUTHZ-02 — Enforce object-level authorization (IDOR/BOLA).** On every operation accepting an object ID, slug, path, key, or foreign key, verify ownership, tenant membership, delegated access, or explicit permission server-side. Test by substituting another user's identifier.
- [ ] **AUTHZ-03 — Enforce function-level authorization.** Protect admin, staff, moderation, billing, export, support, impersonation, audit, internal, and feature-management actions with server-side permission checks.
- [ ] **AUTHZ-04 — Enforce property-level authorization.** Allowlist fields a caller may read or modify. Prevent clients from setting ownership, role, verification, balance, price, tenant, moderation, entitlement, or internal-state fields.
- [ ] **AUTHZ-05 — Use a centralized authorization model.** Define roles, permissions, resource relationships, and policy evaluation consistently; avoid scattered ad hoc checks that drift over time.
- [ ] **AUTHZ-06 — Separate platform and tenant administration.** A tenant administrator must not gain platform-wide access; platform support access should be narrowly scoped and audited.
- [ ] **AUTHZ-07 — Make tenant scope mandatory.** Scope every query, mutation, cache key, search index, vector retrieval, file path, background job, queue message, analytics event, export, and log correlation field by authorized tenant/account where applicable.
- [ ] **AUTHZ-08 — Do not trust client-supplied tenant identifiers.** Resolve tenant context from verified identity and authorized membership; validate any requested tenant transition.
- [ ] **AUTHZ-09 — Prevent confused-deputy behavior.** A privileged service must authorize the initiating user and action, not merely trust that another service called it.
- [ ] **AUTHZ-10 — Re-check authorization at execution time.** Jobs, approvals, scheduled actions, downloads, signed URLs, and queued work must verify that access is still valid when executed.
- [ ] **AUTHZ-11 — Require step-up authentication for sensitive actions.** Apply recent authentication and, where justified, MFA or explicit confirmation before ownership transfer, role changes, payouts, API-key creation, recovery changes, bulk export, or destructive operations.
- [ ] **AUTHZ-12 — Protect invitation workflows.** Bind invitations to intended tenant, role, recipient, and expiry; make them single-use; prevent privilege escalation and reuse after revocation.
- [ ] **AUTHZ-13 — Test authorization matrices.** Cover each role/action/resource combination, anonymous access, stale memberships, disabled accounts, two distinct users, and two distinct tenants.

---

## 5. Sessions, cookies, API keys, OAuth, OIDC, SAML, and tokens

- [ ] **SESS-01 — Use HTTPS for all authenticated traffic.** Never send credentials, sessions, or bearer tokens over plaintext transport.
- [ ] **SESS-02 — Secure cookies.** Set `Secure`, `HttpOnly`, an appropriate `SameSite` value, narrow `Domain`/`Path`, and sensible expiration. Prefer host-only cookies and cookie prefixes where supported.
- [ ] **SESS-03 — Prevent session fixation.** Rotate the session identifier after login, privilege elevation, account linking, and other trust changes.
- [ ] **SESS-04 — Set idle and absolute expiry.** Choose durations based on risk and user context; do not create effectively permanent privileged sessions.
- [ ] **SESS-05 — Revoke sessions correctly.** Revoke on logout, password or recovery changes, suspected compromise, account disablement, and administrator action. Provide session/device management where useful.
- [ ] **SESS-06 — Protect session stores.** Store only necessary data, encrypt sensitive server-side state when needed, and restrict access to session storage.
- [ ] **SESS-07 — Validate JWTs rigorously.** Verify signature, trusted key, allowed algorithm, issuer, audience, expiry, not-before, token type, and required claims. Reject `alg=none`, algorithm confusion, malformed tokens, and unexpected issuers.
- [ ] **SESS-08 — Keep access tokens narrowly scoped.** Restrict audience, resources, actions, tenant, lifetime, and privileges. A token valid for one service must not automatically authorize another.
- [ ] **SESS-09 — Protect refresh tokens.** Use rotation or sender-constraining for public clients, detect reuse, bind tokens to the client/session, and revoke token families on compromise.
- [ ] **SESS-10 — Treat API keys as credentials.** Generate with sufficient entropy, display once, store only a secure hash where practical, support prefixes for identification, scope permissions, enforce expiry/rotation, and log use without logging the key.
- [ ] **SESS-11 — Use current OAuth/OIDC best practice.** Use authorization-code flow with PKCE and exact redirect URI matching. Bind the browser transaction and apply appropriate CSRF protection (PKCE and, where needed, `state`); validate OIDC `nonce` when used. Do not use the resource-owner-password grant, and avoid flows that return access tokens in the authorization response.
- [ ] **SESS-12 — Prevent OAuth mix-up and code injection.** Validate issuer and authorization response binding; use transaction-specific PKCE challenges and approved metadata discovery.
- [ ] **SESS-13 — Do not place secrets or bearer tokens in URLs.** Avoid query strings, fragments, referrers, browser history, logs, analytics, and error reports.
- [ ] **SESS-14 — Validate SAML safely when used.** Verify XML signatures against trusted metadata, audience, recipient, destination, issuer, time conditions, and replay; prevent signature-wrapping and XML parsing weaknesses.
- [ ] **SESS-15 — Rotate signing and encryption keys safely.** Support overlap, key identifiers, revocation, audit trails, and rollback. Never silently fall back to an untrusted key.

---

## 6. Secrets, cryptography, and key management

- [ ] **CRYPTO-01 — Keep secrets out of client code.** Never embed private API keys, database credentials, signing keys, service-role keys, webhook secrets, or cloud credentials in frontend bundles, mobile/desktop binaries, public repositories, source maps, or client-readable environment variables.
- [ ] **CRYPTO-02 — Use a secrets manager.** Store production secrets in a managed vault or equivalent protected system; restrict access by workload identity and environment.
- [ ] **CRYPTO-03 — Use short-lived workload credentials.** Prefer workload identity, federation, or CI OIDC over long-lived static cloud keys.
- [ ] **CRYPTO-04 — Separate secrets by environment and purpose.** Development, preview, staging, and production must not share credentials or sensitive datasets.
- [ ] **CRYPTO-05 — Rotate and revoke secrets.** Define rotation procedures, ownership, expiry where possible, compromise response, and a way to identify every consumer before rotation.
- [ ] **CRYPTO-06 — Scan for secrets continuously.** Scan source, history, build artifacts, container layers, logs, tickets, and generated files. Treat a committed secret as compromised even after deletion.
- [ ] **CRYPTO-07 — Use approved cryptographic libraries and modes.** Do not design custom algorithms or protocols. Prefer authenticated encryption and current platform primitives.
- [ ] **CRYPTO-08 — Manage encryption keys separately from encrypted data.** Restrict key access, record key versions, support rotation, and protect backups of key material.
- [ ] **CRYPTO-09 — Use cryptographically secure randomness.** Use operating-system or platform CSPRNGs for tokens, keys, reset links, nonces, identifiers requiring unpredictability, and invitation codes.
- [ ] **CRYPTO-10 — Use constant-time comparison for secrets where applicable.** Compare MACs, signatures, webhook secrets, and authentication tokens using safe library functions.
- [ ] **CRYPTO-11 — Do not misuse hashing.** Use password hashing for passwords, keyed MACs for integrity/authenticity, and ordinary cryptographic hashes only for non-secret integrity or identifiers where appropriate.
- [ ] **CRYPTO-12 — Apply encryption according to data risk.** Encrypt sensitive data in transit and at rest as required by its classification and threat model, but do not treat encryption as a substitute for authorization, minimization, deletion, or secure key management.
- [ ] **CRYPTO-13 — Protect signing operations.** Keep signing keys non-exportable where feasible; restrict which service can sign; validate the exact payload and context before signing.

---

## 7. Input validation, parsing, injection, and unsafe execution

- [ ] **INPUT-01 — Validate at every trust boundary.** Server-side allowlist expected types, formats, ranges, lengths, encodings, enums, relationships, and state transitions; reject unexpected fields.
- [ ] **INPUT-02 — Canonicalize carefully.** Parse and decode once with the component that will consume the value, reject invalid or ambiguous forms, normalize to one defined representation where appropriate, and make security comparisons in that same representation. Prevent double decoding and discrepancies between validators and consumers.
- [ ] **INPUT-03 — Prevent query injection.** Use parameterized SQL/procedure calls and safe structured query APIs. For NoSQL, search, LDAP, XPath, and similar query languages, allowlist operators and fields, enforce schemas, and prevent untrusted input from changing query structure.
- [ ] **INPUT-04 — Prevent command injection.** Avoid invoking shells. Use structured process APIs with fixed executables and separately passed arguments; apply allowlists and sandboxing.
- [ ] **INPUT-05 — Prevent server-side template injection.** Never evaluate user input as a template, expression, or code. Use non-evaluating templates and strict context-aware escaping.
- [ ] **INPUT-06 — Prevent unsafe deserialization.** Do not deserialize attacker-controlled native objects. Use simple data formats, explicit schemas, type allowlists, size/depth limits, and integrity checks where needed.
- [ ] **INPUT-07 — Prevent path traversal.** Resolve and verify canonical paths, use generated storage keys, restrict roots, and reject absolute paths, traversal sequences, device names, and dangerous encodings.
- [ ] **INPUT-08 — Prevent XXE and unsafe XML behavior.** Disable external entities, DTD processing, external schema fetching, and unrestricted expansion; limit size and nesting.
- [ ] **INPUT-09 — Prevent ReDoS.** Avoid catastrophic-backtracking patterns; cap input length and execution time; prefer safe regex engines or parsers for complex grammars.
- [ ] **INPUT-10 — Defend against prototype pollution and object injection.** Reject dangerous property names, use safe merge utilities, avoid trusting object prototypes, and update vulnerable libraries.
- [ ] **INPUT-11 — Validate URLs as structured data.** Enforce allowed schemes, hosts, ports, paths, credentials, and destination behavior; do not rely on string-prefix or substring checks.
- [ ] **INPUT-12 — Validate headers and metadata.** Treat `Host`, forwarded headers, filenames, content types, locale, device metadata, and client IP headers as untrusted unless set by a trusted proxy and configured explicitly.
- [ ] **INPUT-13 — Prevent response splitting and header injection.** Reject control characters and use framework APIs for headers, cookies, downloads, and redirects.
- [ ] **INPUT-14 — Prevent CSV/formula injection.** Escape or neutralize cells beginning with formula-triggering characters when exporting untrusted content for spreadsheet use.
- [ ] **INPUT-15 — Sandbox unavoidable code execution.** Generated or user-supplied code must run in an isolated, resource-limited, non-privileged environment with no default secrets, host access, or unrestricted network.

---

## 8. Output handling, browser, frontend, and content security

- [ ] **WEB-01 — Escape output by context.** Use framework auto-escaping and context-specific encoding for HTML, attributes, JavaScript, CSS, URLs, and headers.
- [ ] **WEB-02 — Sanitize user-controlled rich content.** Use a maintained allowlist sanitizer. Treat SVG, MathML, embedded media, Markdown extensions, and pasted HTML as potentially active content.
- [ ] **WEB-03 — Avoid dangerous browser APIs.** Minimize `innerHTML`, `outerHTML`, `document.write`, dynamic script creation, `eval`, `Function`, string-based timers, and unsafe URL assignments.
- [ ] **WEB-04 — Deploy a restrictive Content Security Policy.** Prefer nonces or hashes, avoid `unsafe-inline` and `unsafe-eval`, restrict `script-src`, `connect-src`, `frame-src`, `object-src`, and `base-uri`, and use report-only mode before enforcement when needed.
- [ ] **WEB-05 — Prevent CSRF.** For cookie-authenticated state changes, use same-site cookies plus synchronizer tokens or equivalent protections; validate `Origin` and, where appropriate, `Referer`; never use GET for state changes.
- [ ] **WEB-06 — Prevent clickjacking.** Use CSP `frame-ancestors` and, where useful for legacy support, `X-Frame-Options`; allow framing only by explicitly trusted origins.
- [ ] **WEB-07 — Set browser security headers.** Include `X-Content-Type-Options: nosniff`, a restrictive `Referrer-Policy`, and a suitable `Permissions-Policy`. Enable HSTS only on HTTPS hosts after confirming all affected subdomains and preload implications are safe.
- [ ] **WEB-08 — Prevent open redirects.** Use fixed destinations or exact allowlists; do not redirect to arbitrary user-provided URLs.
- [ ] **WEB-09 — Restrict cross-origin communication.** Validate `postMessage` sender origin and message schema; specify exact target origins; do not use `*` for sensitive messages.
- [ ] **WEB-10 — Use Subresource Integrity or self-hosting where appropriate.** Protect externally hosted scripts/styles and minimize third-party browser code.
- [ ] **WEB-11 — Minimize sensitive client storage.** Avoid bearer tokens and sensitive data in `localStorage`, IndexedDB, caches, service workers, or persisted state when safer server-managed sessions are available.
- [ ] **WEB-12 — Secure service workers.** Scope narrowly, prevent cache poisoning and sensitive response caching, handle updates safely, and avoid serving stale authorization-sensitive content.
- [ ] **WEB-13 — Control browser caching.** Mark sensitive responses appropriately, prevent shared-cache leakage, include authorization-relevant cache keys, and clear sensitive state on logout.
- [ ] **WEB-14 — Protect source maps and build metadata.** Do not expose private source, internal paths, secrets, or debugging data in production artifacts.
- [ ] **WEB-15 — Prevent tabnabbing and opener abuse.** Use safe link attributes and explicitly control new-window behavior.

---

## 9. API, GraphQL, WebSocket, and service-to-service security

- [ ] **API-01 — Maintain an API inventory.** Include versions, hosts, authentication schemes, owners, data classifications, clients, deprecation dates, and public/internal status.
- [ ] **API-02 — Authenticate and authorize every API operation.** Do not assume an API is protected because it is undocumented, on an unusual path, or called only by the frontend.
- [ ] **API-03 — Validate request and response schemas.** Enforce types, required fields, additional-property rules, size, depth, and semantic constraints at runtime.
- [ ] **API-04 — Prevent HTTP parameter pollution.** Define how duplicate query, form, cookie, and header fields are handled; reject ambiguous duplicates for security-sensitive parameters.
- [ ] **API-05 — Minimize response data.** Return only required fields; prevent excessive data exposure, internal flags, stack traces, identifiers, and cross-tenant references.
- [ ] **API-06 — Use endpoint-specific rate limits and quotas.** Limit requests, cost, concurrency, payload size, result size, pagination, and expensive filters per identity, tenant, IP/network, and globally as appropriate.
- [ ] **API-07 — Make pagination bounded.** Set safe defaults and hard maximums; prevent unbounded exports or scans through ordinary list endpoints.
- [ ] **API-08 — Use idempotency for retryable mutations.** Bind keys to caller and operation, store results safely, set expiry, and reject key reuse with conflicting payloads.
- [ ] **API-09 — Protect API versioning and deprecation.** Authenticate legacy versions, patch them until removal, monitor usage, and remove abandoned endpoints.
- [ ] **API-10 — Restrict CORS.** Allow only explicit trusted origins, methods, and headers; never combine credentialed requests with wildcard origins.
- [ ] **API-11 — Protect GraphQL complexity.** Apply depth, breadth, alias, batching, recursion, and cost limits; authorize each resolver; cap pagination; prevent batching from bypassing rate limits.
- [ ] **API-12 — Control GraphQL introspection and errors.** Decide deliberately by environment and audience; do not expose secrets, stack traces, or internal schema details unnecessarily.
- [ ] **API-13 — Secure WebSockets and streaming connections.** Authenticate the handshake, authorize each subscription/action, re-check long-lived permissions, enforce origin policy, bound message size/rate, and close revoked sessions.
- [ ] **API-14 — Authenticate service-to-service calls.** Use workload identity, scoped tokens, mTLS, or signed requests; do not trust network location alone.
- [ ] **API-15 — Prevent replay.** Use timestamps, nonces, sequence rules, short validity, and idempotency where signed or high-impact requests can be captured and resent.
- [ ] **API-16 — Do not expose management interfaces publicly.** Protect metrics, health details, admin APIs, debug endpoints, database consoles, queues, dashboards, and service discovery.
- [ ] **API-17 — Use safe error contracts.** Return stable, non-sensitive errors to clients and retain detailed correlated diagnostics only in protected logs.

---

## 10. Database, cache, storage, queues, and background processing

- [ ] **DATA-01 — Use least-privilege database identities.** Separate runtime, migration, reporting, backup, and administrative permissions.
- [ ] **DATA-02 — Enable and correctly design row-level security where applicable.** Protect every exposed user/tenant table with least-privilege policies that explicitly cover `SELECT`, `INSERT`, `UPDATE`, and `DELETE`, including both row-access predicates and write checks where the database distinguishes them.
- [ ] **DATA-03 — Audit RLS bypass paths.** Review joins, views, RPC/functions, security-definer functions, service-role clients, ownership columns, and related tables. Use both access predicates and write checks.
- [ ] **DATA-04 — Prevent client control of ownership fields.** Set owner and tenant identifiers from trusted server context, not request bodies.
- [ ] **DATA-05 — Use transactions and constraints for invariants.** Enforce uniqueness, nonnegative balances, valid transitions, referential integrity, and one-time actions at the database layer where possible.
- [ ] **DATA-06 — Protect against race conditions.** Use atomic operations, locking, optimistic concurrency, serializable transactions, or compare-and-set patterns for sensitive state.
- [ ] **DATA-07 — Secure database transport and exposure.** Require TLS, private networking or strict allowlists, certificate validation, and no public administrative access.
- [ ] **DATA-08 — Protect database logs and replicas.** Apply the same data classification, access control, encryption, retention, and deletion requirements.
- [ ] **DATA-09 — Scope caches by authorization context.** Include tenant, user, role, locale, and other relevant dimensions; never cache private data under a public or incomplete key.
- [ ] **DATA-10 — Prevent cache poisoning.** Normalize and constrain cache keys, trusted headers, hostnames, and variants; avoid caching unvalidated error or redirect responses.
- [ ] **DATA-11 — Treat queue messages as untrusted.** Authenticate producers where possible, validate schemas, authorize the represented action, bound size, and reject malformed or stale messages.
- [ ] **DATA-12 — Make consumers idempotent.** Handle duplicate, delayed, reordered, and retried messages without duplicate charges, emails, grants, deletions, or state corruption.
- [ ] **DATA-13 — Protect dead-letter queues.** Restrict access, avoid sensitive payload leakage, monitor growth, and define safe replay procedures.
- [ ] **DATA-14 — Re-authorize background jobs.** Store minimum necessary context and verify permissions/state at execution, especially after role or tenant changes.
- [ ] **DATA-15 — Prevent job parameter tampering.** Sign or store server-side sensitive job parameters; do not trust client-supplied callback URLs, file paths, roles, prices, or recipients.
- [ ] **DATA-16 — Set retention and deletion for caches, queues, replicas, and derived stores.** Data deletion must propagate beyond the primary database.

---

## 11. File upload, download, media, document, and archive security

- [ ] **FILE-01 — Decide whether uploads are necessary.** Prefer not accepting files when structured input is sufficient.
- [ ] **FILE-02 — Allowlist file types.** Validate extension, MIME type, and file signature/magic bytes; account for polyglots and format ambiguity.
- [ ] **FILE-03 — Limit upload size, count, dimensions, duration, pages, nesting, and compression ratio.** Enforce limits before or during streaming, not only after full ingestion.
- [ ] **FILE-04 — Use generated storage names.** Do not use untrusted filenames as filesystem paths or public object keys; preserve display names separately after sanitization.
- [ ] **FILE-05 — Store uploads outside executable roots.** Prevent execution, server-side includes, dynamic import, and content-type sniffing.
- [ ] **FILE-06 — Keep sensitive objects private.** Authorize read, list, upload, overwrite, rename, and delete independently. Use short-lived signed URLs where appropriate.
- [ ] **FILE-07 — Scan risky uploads.** Use malware scanning, content disarm/reconstruction, or sandboxing based on file type and threat model; quarantine until processing completes.
- [ ] **FILE-08 — Treat image/document parsers as attack surfaces.** Use maintained libraries, resource limits, process isolation where warranted, and safe handling of SVG, PDF, office documents, fonts, media codecs, and metadata.
- [ ] **FILE-09 — Strip unnecessary metadata.** Remove EXIF/GPS, embedded scripts, comments, hidden content, macros, and author data where product requirements permit.
- [ ] **FILE-10 — Defend against archive bombs and traversal.** Limit total expanded size, file count, depth, links, special files, and paths; extract only into isolated temporary directories.
- [ ] **FILE-11 — Re-encode untrusted media when appropriate.** Decode and encode into a known-safe format rather than serving original active content.
- [ ] **FILE-12 — Force safe download behavior.** Set an explicit content type, `Content-Disposition`, safe filename, `nosniff`, and appropriate sandbox/CSP headers for browser-viewable content.
- [ ] **FILE-13 — Prevent unauthorized sharing by URL.** Do not treat an unguessable URL as the sole authorization mechanism for sensitive files unless the signed capability is deliberate, scoped, short-lived, and revocable where required.
- [ ] **FILE-14 — Clean temporary files.** Use isolated directories, restrictive permissions, quotas, and reliable cleanup on success, error, cancellation, and timeout.

---

## 12. Network, SSRF, DNS, proxies, webhooks, and third-party integrations

- [ ] **NET-01 — Use secure transport everywhere.** Require current TLS for clients, services, databases, storage, webhooks, and provider APIs; validate certificates; never disable TLS verification.
- [ ] **NET-02 — Prevent SSRF.** Prefer allowlisted destinations. Validate scheme, hostname, resolved IP, port, and final destination on every redirect. Block loopback, link-local, multicast, reserved, cloud-metadata, and private ranges unless a narrowly defined private destination is an intentional requirement protected by a separate allowlist and network isolation.
- [ ] **NET-03 — Defend against DNS rebinding and resolution changes.** Resolve safely, validate every address that may be selected, and ensure the connection uses an approved address; repeat validation after redirects and do not let the network client silently re-resolve to an unapproved destination.
- [ ] **NET-04 — Restrict outbound network access.** Give services only the egress they need; isolate URL fetchers and document processors from internal networks and metadata services.
- [ ] **NET-05 — Bound external requests.** Apply connect/read/total timeouts, response-size limits, redirect limits, decompression limits, protocol allowlists, and concurrency limits.
- [ ] **NET-06 — Use safe proxy configuration.** Trust forwarded headers only from known proxies; define the trusted proxy chain; prevent spoofed client IP, host, scheme, and secure-cookie decisions.
- [ ] **NET-07 — Prevent HTTP request smuggling/desynchronization.** Keep proxy, load balancer, CDN, and origin HTTP parsing consistent; reject ambiguous transfer framing, conflicting length headers, and unsupported protocol downgrades.
- [ ] **NET-08 — Verify inbound webhooks.** Validate signatures/MACs with the raw body, timestamp and freshness, expected source/context, schema, and replay protection.
- [ ] **NET-09 — Make webhook handlers idempotent.** Store provider event IDs or equivalent deduplication state and process state transitions transactionally.
- [ ] **NET-10 — Do not trust third-party payloads.** Revalidate identifiers, amounts, URLs, filenames, statuses, and referenced resources before privileged use.
- [ ] **NET-11 — Use least-privilege integration credentials.** Scope provider tokens to required resources/actions; separate environments and tenants where supported.
- [ ] **NET-12 — Handle provider failure safely.** Use bounded retries with backoff and jitter, circuit breakers, idempotency, dead-letter handling, and reconciliation jobs.
- [ ] **NET-13 — Validate callback destinations.** Prevent arbitrary callback, webhook, image proxy, import, or export URLs unless intentionally supported with strong SSRF controls.
- [ ] **NET-14 — Review third-party scripts and SDKs.** Minimize browser permissions and data access; pin versions; monitor compromise and ownership changes; provide kill switches.
- [ ] **NET-15 — Protect domain and DNS administration.** Use MFA, registrar lock where appropriate, restricted accounts, monitored changes, secure DNS configuration, and documented recovery.
- [ ] **NET-16 — Protect outbound email identity.** Configure SPF, DKIM, and DMARC as appropriate; restrict sending credentials; prevent header injection; monitor spoofing and delivery-domain changes.

---

## 13. Business logic, workflows, payments, abuse, and cost controls

- [ ] **BIZ-01 — Make the server authoritative.** Calculate prices, taxes, discounts, currency, inventory, balances, entitlements, quotas, roles, approval state, and feature access from trusted server-side records.
- [ ] **BIZ-02 — Model workflows as explicit state machines.** Define legal transitions, required actor, preconditions, side effects, retry behavior, cancellation, and terminal states.
- [ ] **BIZ-03 — Validate state transitions transactionally.** Prevent skipping steps, replaying completed steps, approving one's own request where prohibited, or acting on stale state.
- [ ] **BIZ-04 — Prevent race conditions and double-spend.** Use transactions, idempotency keys, uniqueness constraints, locks/atomic updates, and reconciliation.
- [ ] **BIZ-05 — Verify payment events server-side.** Validate provider signature, timestamp, event identity, expected customer/order, amount, currency, environment, and final status before granting value.
- [ ] **BIZ-06 — Do not trust browser payment success.** Treat redirects and frontend SDK callbacks as user experience signals, not proof of payment.
- [ ] **BIZ-07 — Handle refunds, disputes, reversals, and partial payments.** Reconcile entitlements and financial records consistently and audit every adjustment.
- [ ] **BIZ-08 — Protect donations, credits, coupons, referrals, and promotional balances.** Prevent self-referral, repeated redemption, negative values, race abuse, and cross-account transfer not explicitly supported.
- [ ] **BIZ-09 — Apply layered rate limits.** Use per-user, tenant, token, IP/network, device, endpoint, and global limits based on abuse risk.
- [ ] **BIZ-10 — Set hard cost and resource ceilings.** Bound AI tokens, generations, image size, compute time, uploads, email/SMS, exports, geocoding, storage, and third-party spend.
- [ ] **BIZ-11 — Use concurrency limits and queues.** Prevent one user, tenant, or anonymous source from exhausting workers, database pools, provider quotas, or memory.
- [ ] **BIZ-12 — Protect pre-authentication flows.** Prevent account farming, verification/SMS bombing, unlimited trials, anonymous cost exhaustion, and IP rotation bypasses.
- [ ] **BIZ-13 — Use abuse-resistant identifiers.** Do not expose sequential identifiers where they enable enumeration without strong authorization and rate controls.
- [ ] **BIZ-14 — Detect scraping and enumeration.** Monitor velocity, coverage patterns, failed authorization, search behavior, and bulk access; use product-appropriate mitigation.
- [ ] **BIZ-15 — Protect exports and bulk operations.** Require authorization, recent authentication where warranted, bounded scope, asynchronous processing, audit logs, expiration, and download protection.
- [ ] **BIZ-16 — Add kill switches.** Be able to disable expensive providers, vulnerable features, compromised integrations, or abusive tenants without a full redeploy.
- [ ] **BIZ-17 — Fail safely under provider or budget exhaustion.** Do not grant unpaid entitlements, expose fallback credentials, or bypass controls to maintain availability.

---

## 14. Privacy, personal data, retention, and user rights

- [ ] **PRIV-01 — Collect the minimum data required.** Do not gather personal, device, location, contact, biometric, or behavioral data without a defined product need.
- [ ] **PRIV-02 — Document purpose and lawful/authorized use.** Track why each sensitive field exists, who can access it, where it flows, and how long it is retained.
- [ ] **PRIV-03 — Use privacy-protective defaults.** Do not enable public profiles, tracking, personalization, model training, or data sharing by default unless clearly appropriate and disclosed.
- [ ] **PRIV-04 — Provide clear consent and choices where required.** Avoid dark patterns; distinguish required processing from optional analytics, marketing, personalization, and training.
- [ ] **PRIV-05 — Limit internal access.** Use least privilege, purpose limitation, audited access, and masking for support, analytics, and operations.
- [ ] **PRIV-06 — Keep sensitive data out of URLs and logs.** Also exclude it from analytics, crash reports, traces, screenshots, referrers, cache keys, filenames, and third-party monitoring unless explicitly protected and necessary.
- [ ] **PRIV-07 — Protect data in non-production.** Use synthetic or properly de-identified data; do not copy production personal data into local development, demos, or public previews.
- [ ] **PRIV-08 — Define retention schedules.** Apply automatic deletion or archival to primary data, files, logs, analytics, vector stores, caches, queues, replicas, and backups.
- [ ] **PRIV-09 — Implement account and data deletion.** Verify deletion reaches all active systems and that backup expiration follows documented policy; record exceptions that must be retained.
- [ ] **PRIV-10 — Implement data access and export safely.** Authenticate strongly, authorize scope, avoid cross-tenant data, protect generated archives, and expire download links.
- [ ] **PRIV-11 — Handle minors and sensitive categories deliberately.** Add age, consent, safety, minimization, and regulatory controls appropriate to the jurisdiction and product.
- [ ] **PRIV-12 — Review international and vendor data transfers.** Know provider regions, subprocessors, retention, training use, and breach-notification commitments.
- [ ] **PRIV-13 — Prevent re-identification.** Treat pseudonymous identifiers, embeddings, telemetry, and “anonymized” datasets as potentially identifiable unless robustly assessed.
- [ ] **PRIV-14 — Include privacy in incident response.** Define who assesses notification obligations, affected data, jurisdictions, users, and third parties.

---

## 15. AI, LLM, RAG, agents, MCP, and generated-content security

- [ ] **AI-01 — Assume prompt injection will occur.** Treat user prompts, retrieved documents, web pages, emails, files, tool output, model memory, and third-party content as untrusted data—not authority.
- [ ] **AI-02 — Do not use prompts as a security boundary.** Authorization, data isolation, policy enforcement, secrets, spending limits, and action approval must exist outside the model.
- [ ] **AI-03 — Apply least agency.** Give the model only the minimum tools, permissions, data, duration, and action scope necessary for the current task.
- [ ] **AI-04 — Authorize every tool call server-side.** Validate the authenticated user, tenant, permission, resource, action, and current state independently of model-generated arguments.
- [ ] **AI-05 — Use narrow tool schemas.** Prefer explicit enums and typed fields over free-form commands, SQL, shell, URLs, paths, recipients, or arbitrary JSON.
- [ ] **AI-06 — Require confirmation for consequential actions.** Before sending, publishing, purchasing, deleting, transferring, changing permissions, executing code, or revealing sensitive data, show the exact action and obtain informed user confirmation unless a narrowly defined pre-authorization exists.
- [ ] **AI-07 — Bind confirmation to the exact action.** A confirmation must not authorize materially changed recipients, amounts, resources, content, or permissions.
- [ ] **AI-08 — Treat model output as untrusted.** Schema-validate before tool use and sanitize before rendering. Never pass generated SQL, shell, code, templates, URLs, or HTML directly to a privileged interpreter or sensitive sink; use narrow structured operations or the isolated execution controls in `AI-09` when execution is an explicit feature.
- [ ] **AI-09 — Isolate generated code execution.** Use disposable sandboxes, non-root execution, strict CPU/memory/time/process/file limits, no host mounts, no default secrets, and restricted or disabled networking.
- [ ] **AI-10 — Secure RAG authorization.** Filter retrieval by verified user and tenant permissions before similarity search and again before returning source content.
- [ ] **AI-11 — Prevent vector-store tenant crossover.** Namespace/index/partition embeddings, metadata, cache, and retrieval logs by tenant and authorization context.
- [ ] **AI-12 — Limit retrieved content.** Bound document size, chunk count, context length, recency, source trust, and sensitivity; avoid retrieving secrets merely because they are semantically relevant.
- [ ] **AI-13 — Defend against indirect prompt injection.** Separate instructions from data, label source boundaries, constrain tools, use policy checks, and avoid allowing retrieved content to redefine system authority.
- [ ] **AI-14 — Protect agent memory.** Validate what may be stored, scope memory by user/tenant, prevent untrusted content from becoming durable instructions, support review/deletion, and expire stale memory.
- [ ] **AI-15 — Prevent sensitive-information disclosure.** Redact prompts, traces, evaluations, tool output, and logs; avoid placing secrets in model context; test extraction attempts.
- [ ] **AI-16 — Control model and tool resource consumption.** Set limits on tokens, turns, recursion, parallelism, tool calls, file size, generation count, latency, and spend; stop loops deterministically.
- [ ] **AI-17 — Detect and contain excessive agency.** Restrict tool chaining, delegation, autonomous retries, and cross-system actions; maintain an allowlisted action graph where practical.
- [ ] **AI-18 — Review MCP/tool servers and plugins.** Treat them as privileged dependencies; pin versions/endpoints, authenticate them, scope credentials, review schemas, monitor changes, and disable untrusted discovery.
- [ ] **AI-19 — Protect model supply chain.** Verify model, adapter, dataset, prompt package, evaluation set, and runtime sources; record immutable versions and hashes where available; scan artifacts; review licenses and provenance.
- [ ] **AI-20 — Address data and model poisoning.** Authenticate data sources, validate provenance, separate untrusted feedback, monitor anomalous changes, and provide rollback for indexes, prompts, models, and fine-tunes.
- [ ] **AI-21 — Test model-specific attacks.** Include direct and indirect prompt injection, data exfiltration, cross-tenant retrieval, tool-argument manipulation, jailbreaks, denial of wallet/service, unsafe output handling, and memory poisoning.
- [ ] **AI-22 — Provide human override and auditability.** Record the user request, model/tool decisions, approvals, actions, and results in a privacy-conscious audit trail sufficient for incident review.
- [ ] **AI-23 — Use safe fallback behavior.** When policy, identity, tool, retrieval, or validation fails, refuse or degrade safely rather than guessing or bypassing controls.
- [ ] **AI-24 — Do not overstate AI assurance.** Clearly distinguish generated content from verified facts and require deterministic validation for high-impact decisions.

---

## 16. Mobile and desktop client security (when applicable)

- [ ] **CLIENT-01 — Assume the client device is hostile.** Do not store server secrets or rely on client code, obfuscation, certificate pins, or local flags to enforce authorization.
- [ ] **CLIENT-02 — Use platform secure storage.** Store refresh tokens, private keys, and sensitive credentials in Keychain/Keystore/credential vaults with appropriate access controls.
- [ ] **CLIENT-03 — Minimize local sensitive data.** Encrypt where appropriate, exclude from backups, clear on logout, and protect temporary files, caches, clipboard, and notifications.
- [ ] **CLIENT-04 — Secure deep links and custom URL schemes.** Validate origin, route, parameters, authorization, and state; prefer verified universal/app links; prevent link hijacking and open redirects.
- [ ] **CLIENT-05 — Harden WebViews.** Avoid loading untrusted content with privileged bridges, disable unnecessary JavaScript/file access, restrict navigation, and validate messages between native and web contexts.
- [ ] **CLIENT-06 — Protect inter-process communication.** Restrict exported components, intents, URL handlers, local sockets, named pipes, and custom protocols.
- [ ] **CLIENT-07 — Validate update mechanisms.** Require signed updates, trusted distribution, integrity verification, anti-downgrade protection, and safe rollback.
- [ ] **CLIENT-08 — Control screenshots and screen recording where risk warrants.** Prevent sensitive data appearing in app switchers, notifications, logs, or share sheets.
- [ ] **CLIENT-09 — Handle device compromise realistically.** Root/jailbreak detection may be a signal but not a sole control; keep critical authorization server-side.
- [ ] **CLIENT-10 — Use certificate pinning only with an operational plan.** If used, include backup pins/keys, rotation, telemetry, and recovery; never disable normal certificate validation.
- [ ] **CLIENT-11 — Test against the current OWASP mobile verification/testing guidance.** Cover storage, crypto, auth, network, platform, code, resilience, and privacy controls appropriate to the app.

---

## 17. Cloud, serverless, containers, orchestration, and infrastructure

- [ ] **CLOUD-01 — Manage infrastructure as code where practical.** Review, version, scan, and approve cloud, network, DNS, identity, storage, and deployment configuration.
- [ ] **CLOUD-02 — Use least-privilege cloud IAM.** Separate human and workload identities; scope resources and actions; avoid wildcard permissions and shared credentials.
- [ ] **CLOUD-03 — Protect the cloud control plane.** Require MFA, strong federation, restricted administrator roles, short-lived access, audit logs, and break-glass procedures.
- [ ] **CLOUD-04 — Keep private resources private.** Databases, queues, object storage, metadata, internal services, dashboards, and orchestrator APIs should not be internet-exposed unless required and strongly protected.
- [ ] **CLOUD-05 — Block public storage by default.** Use organization/account-level preventive controls where available; continuously detect public buckets and objects.
- [ ] **CLOUD-06 — Protect instance and workload metadata.** Use current metadata protections, restrict network access, and prevent SSRF paths to credentials.
- [ ] **CLOUD-07 — Segment networks.** Separate public ingress, application, data, administration, build, and untrusted processing zones; restrict east-west and egress traffic.
- [ ] **CLOUD-08 — Harden containers.** Use minimal trusted images, run as non-root, drop capabilities, avoid privileged mode and host mounts, use read-only filesystems where possible, and apply resource limits and runtime profiles.
- [ ] **CLOUD-09 — Pin images and artifacts by immutable digest.** Scan before deployment, sign/verify provenance where supported, and remove outdated images from registries.
- [ ] **CLOUD-10 — Protect orchestration secrets and service accounts.** Disable automatic token mounting where unnecessary; scope namespaces and RBAC; encrypt sensitive control-plane data.
- [ ] **CLOUD-11 — Secure ingress and load balancers.** Enforce TLS, trusted proxy settings, size/time limits, safe headers, origin restrictions, and protected administrative endpoints.
- [ ] **CLOUD-12 — Set serverless limits.** Configure concurrency, timeout, memory, payload, retry, event age, dead-letter behavior, and least-privilege execution roles.
- [ ] **CLOUD-13 — Prevent cross-environment access.** Production resources and secrets must not be reachable by preview or untrusted branch deployments.
- [ ] **CLOUD-14 — Detect configuration drift.** Monitor changes made outside approved infrastructure workflows and restore secure configuration.
- [ ] **CLOUD-15 — Plan secure teardown.** Revoke credentials, delete data, DNS, storage, backups, queues, logs, and provider integrations when environments or features are retired.

---

## 18. Dependencies, packages, build systems, CI/CD, and software supply chain

- [ ] **SUPPLY-01 — Use lockfiles and reproducible build inputs.** Pin direct and transitive dependencies where supported, review lockfile changes, and make builds reproducible or otherwise attestable where practical.
- [ ] **SUPPLY-02 — Minimize dependencies.** Remove unused packages, plugins, build tools, images, GitHub Actions, and SDKs.
- [ ] **SUPPLY-03 — Verify package identity.** Detect typosquatting, dependency confusion, malicious install scripts, unexpected registries, abandoned packages, and suspicious maintainer changes.
- [ ] **SUPPLY-04 — Configure registries safely.** Use scoped/private registries where needed, explicit sources, integrity verification, and protections against public-package shadowing.
- [ ] **SUPPLY-05 — Generate and retain an SBOM.** Include production components, versions, dependency relationships, images, and major services in a machine-readable format.
- [ ] **SUPPLY-06 — Run software composition analysis.** Scan dependencies and container images continuously; prioritize exploitability, exposure, and reachability, not severity score alone.
- [ ] **SUPPLY-07 — Patch supported vulnerabilities promptly.** Define service objectives for critical/high findings and emergency updates; remove unsupported components.
- [ ] **SUPPLY-08 — Scan source and artifacts for secrets.** Include commit history, pull requests, generated bundles, images, release archives, and logs.
- [ ] **SUPPLY-09 — Use static analysis and secure linters.** Configure rules for the languages/frameworks in use and review suppressions as security decisions.
- [ ] **SUPPLY-10 — Scan infrastructure and policy.** Check IaC, container configuration, cloud permissions, Kubernetes manifests, and deployment descriptors.
- [ ] **SUPPLY-11 — Protect branches and releases.** Require review for sensitive code, status checks, restricted force-push, protected tags, and controlled deployment approvals.
- [ ] **SUPPLY-12 — Protect CI secrets from untrusted code.** Do not expose production secrets to forks, arbitrary pull requests, unreviewed scripts, or mutable third-party actions.
- [ ] **SUPPLY-13 — Pin CI actions and build images.** Prefer immutable references or verified publishers; review changes before updates.
- [ ] **SUPPLY-14 — Use isolated, ephemeral build runners where possible.** Prevent persistence, credential theft, cross-project contamination, and access to internal networks.
- [ ] **SUPPLY-15 — Separate build and deploy authority.** A compromised build step should not automatically gain unrestricted production administration.
- [ ] **SUPPLY-16 — Sign and verify release artifacts where practical.** Record provenance, source revision, builder, dependency state, and environment.
- [ ] **SUPPLY-17 — Protect package publishing.** Require MFA, scoped tokens, trusted publishing, review, and rapid revocation procedures.
- [ ] **SUPPLY-18 — Review AI-generated dependencies and code.** Verify package names, APIs, licenses, maintenance, security posture, and whether the generated code actually enforces the intended control.

---

## 19. Logging, audit trails, detection, monitoring, and alerting

- [ ] **LOG-01 — Define security-relevant events.** Include login/recovery/MFA changes, session creation/revocation, authorization denials, admin actions, role changes, exports, payments, secret use, AI tool actions, and security-control failures.
- [ ] **LOG-02 — Use structured logs and correlation IDs.** Correlate user/session/request/job/provider events without exposing secrets.
- [ ] **LOG-03 — Prevent log injection.** Encode or structure untrusted values, normalize line breaks, and do not let attacker-controlled fields forge severity, identity, event boundaries, or audit records.
- [ ] **LOG-04 — Do not log sensitive values.** Redact passwords, tokens, authorization headers, session IDs, API keys, private keys, full payment data, and unnecessary personal content.
- [ ] **LOG-05 — Protect log integrity and access.** Restrict readers, separate duties, encrypt transport/storage, define retention, and make high-value audit records tamper-evident where warranted.
- [ ] **LOG-06 — Synchronize time.** Use reliable time sources so authentication, token, webhook, and incident timelines are trustworthy.
- [ ] **LOG-07 — Alert on actionable signals.** Examples: repeated auth failures, new admin creation, MFA removal, privilege escalation, cross-tenant denials, webhook failures, unusual exports, secret-scan hits, and cost spikes.
- [ ] **LOG-08 — Monitor abuse and resource anomalies.** Track per-user/tenant/provider rates, queue backlog, storage growth, token usage, generation cost, and error patterns.
- [ ] **LOG-09 — Monitor control health.** Detect disabled RLS, public storage, missing security headers, expired certificates, failing backups, disabled scanners, and log pipeline outages.
- [ ] **LOG-10 — Avoid alert flooding.** Tune thresholds, deduplicate, assign ownership, define severity, and test routing/escalation.
- [ ] **LOG-11 — Preserve privacy in observability.** Sample and redact carefully; restrict session replay and screen capture; exclude sensitive pages and fields.
- [ ] **LOG-12 — Provide secure diagnostic modes.** Time-bound and restrict verbose logging; never enable debug mode globally in production.

---

## 20. Reliability, availability, backup, recovery, and denial-of-service resistance

- [ ] **RES-01 — Set resource limits at every layer.** Bound request body, header count/size, form fields, JSON/XML depth, upload size, decompression, database rows, memory, CPU, process count, and execution time.
- [ ] **RES-02 — Use timeouts consistently.** Configure inbound, outbound, database, queue, lock, job, and model/tool timeouts; avoid unbounded waits.
- [ ] **RES-03 — Use bounded retries.** Apply exponential backoff and jitter; retry only safe/idempotent operations; prevent retry storms.
- [ ] **RES-04 — Use circuit breakers and load shedding.** Protect databases and providers; degrade nonessential features before core authentication or data integrity.
- [ ] **RES-05 — Protect against application-layer DoS.** Identify expensive searches, regex, joins, reports, image/document processing, GraphQL queries, exports, and AI operations.
- [ ] **RES-06 — Use quotas and fair scheduling.** Prevent one tenant or user from monopolizing shared workers, database pools, storage, or provider limits.
- [ ] **RES-07 — Design fail-safe behavior.** Authorization, payment, and integrity checks should fail closed; availability fallbacks must not bypass security.
- [ ] **RES-08 — Back up critical data and configuration.** Include databases, object metadata, infrastructure state, encryption-key strategy, and essential configuration.
- [ ] **RES-09 — Encrypt and restrict backups.** Separate backup credentials, accounts, and regions where appropriate; monitor access and deletion.
- [ ] **RES-10 — Test restoration.** Verify integrity, recovery time, permissions, tenant boundaries, key availability, and application consistency—not merely that backups exist.
- [ ] **RES-11 — Protect backups from ransomware and operator error.** Use immutability, versioning, separate administration, and tested retention where risk warrants.
- [ ] **RES-12 — Document recovery objectives.** Define acceptable data loss and downtime for critical components and align architecture accordingly.
- [ ] **RES-13 — Plan rollback.** Releases, migrations, policies, models, prompts, and provider changes need safe rollback or forward-recovery procedures.

---

## 21. Secure deployment and production configuration

- [ ] **DEPLOY-01 — Separate environments.** Use distinct accounts/projects, networks, credentials, databases, storage, domains, and provider modes for development, staging, preview, and production.
- [ ] **DEPLOY-02 — Disable debug and development behavior.** Remove stack traces, interactive consoles, hot reload, test endpoints, seeded credentials, permissive CORS, mock authentication, and unintended server-source or source-map exposure.
- [ ] **DEPLOY-03 — Remove default content and accounts.** Delete sample apps, default credentials, unused dashboards, documentation consoles, and installation files.
- [ ] **DEPLOY-04 — Set secure environment variables and flags.** Validate required settings at startup and fail safely when security-critical configuration is missing.
- [ ] **DEPLOY-05 — Harden web server and framework configuration.** Disable directory listing, unsafe methods, legacy protocols, unnecessary modules, and verbose server headers.
- [ ] **DEPLOY-06 — Use controlled database migrations.** Review privileges, data transformations, lock/availability impact, rollback, and tenant-safety implications.
- [ ] **DEPLOY-07 — Protect health and metrics endpoints.** Expose minimal public liveness information; restrict detailed readiness, build, dependency, and metrics data.
- [ ] **DEPLOY-08 — Use safe feature flags.** Authorize flag changes, separate environments, audit updates, prevent client-side privilege flags, and clean up stale flags.
- [ ] **DEPLOY-09 — Verify production domains and callbacks.** Review OAuth redirect URIs, webhook URLs, CORS origins, cookie domains, allowed hosts, email links, and payment environment.
- [ ] **DEPLOY-10 — Prevent preview-environment exposure.** Authenticate previews containing private features/data and never connect untrusted previews to production secrets or databases.
- [ ] **DEPLOY-11 — Run post-deployment smoke security tests.** Verify authentication, authorization, headers, TLS, storage privacy, rate limiting, provider modes, logging, and rollback readiness.

---

## 22. Security testing and verification

- [ ] **TEST-01 — Add security unit and integration tests.** Cover authorization, tenant isolation, validation, token verification, RLS, state transitions, rate limits, signed URLs, and webhook verification.
- [ ] **TEST-02 — Test with two users and two tenants.** Attempt read, write, update, delete, list, search, export, cache, file, job, and AI retrieval crossover.
- [ ] **TEST-03 — Test as an unauthenticated and least-privileged user.** Include direct API calls that bypass the UI.
- [ ] **TEST-04 — Add negative tests.** Verify malformed, expired, replayed, oversized, unauthorized, out-of-order, duplicate, and cross-origin requests fail safely.
- [ ] **TEST-05 — Run dependency, secret, static, and infrastructure scans.** Fail CI on policy-defined severe findings or require explicit reviewed exceptions.
- [ ] **TEST-06 — Run dynamic security testing where appropriate.** Scan deployed test environments and manually validate high-impact findings to reduce false confidence.
- [ ] **TEST-07 — Fuzz parsers and exposed schemas where useful.** Focus on file formats, URL parsing, serializers, protocol boundaries, and complex validators.
- [ ] **TEST-08 — Test rate-limit bypasses.** Try alternate endpoints, GraphQL batching, multiple accounts, token/IP changes, concurrency, retries, and distributed patterns.
- [ ] **TEST-09 — Test race conditions.** Use parallel requests against redemption, payment, invitation, role, balance, reservation, and one-time workflows.
- [ ] **TEST-10 — Test SSRF thoroughly.** Cover redirects, alternate IP representations, IPv6, DNS rebinding, userinfo, mixed encodings, metadata hosts, and internal ports.
- [ ] **TEST-11 — Test browser controls.** Verify XSS sinks, CSRF, CSP behavior, CORS, clickjacking, cache leakage, open redirects, and `postMessage` origin checks.
- [ ] **TEST-12 — Test uploads adversarially.** Include spoofed types, polyglots, traversal names, SVG/script content, decompression bombs, malformed media, and unauthorized access.
- [ ] **TEST-13 — Test recovery and revocation.** Verify logout, password reset, MFA removal, role removal, account disablement, key rotation, and provider revocation invalidate access.
- [ ] **TEST-14 — Test backups and incident procedures.** Perform restoration and at least tabletop exercises for likely high-impact incidents.
- [ ] **TEST-15 — Use independent review for high-risk systems.** Obtain a qualified security review or penetration test before major release and after material security-sensitive redesign.
- [ ] **TEST-16 — Retest fixes.** Confirm the original exploit no longer works and that the mitigation did not create a bypass or regression elsewhere.

---

## 23. Incident response, vulnerability handling, and post-release operations

- [ ] **IR-01 — Maintain an incident response plan.** Define severity, roles, contacts, evidence handling, containment, eradication, recovery, communication, and lessons learned.
- [ ] **IR-02 — Prepare credential-revocation procedures.** Be able to rotate database, cloud, OAuth, webhook, signing, payment, email, AI-provider, and CI/CD credentials quickly.
- [ ] **IR-03 — Prepare containment controls.** Include account disablement, session revocation, tenant suspension, provider disablement, feature kill switches, network blocks, and read-only mode.
- [ ] **IR-04 — Define vulnerability intake.** Provide a monitored security contact or disclosure channel and a process for triage, acknowledgment, remediation, and coordinated disclosure; publish `security.txt` when appropriate.
- [ ] **IR-05 — Track vulnerability remediation.** Assign owner, severity, due date, affected versions, fix, verification, and release status.
- [ ] **IR-06 — Monitor newly disclosed vulnerabilities.** Watch dependencies, base images, frameworks, cloud services, identity providers, and critical integrations.
- [ ] **IR-07 — Plan breach notification.** Know contractual, legal, provider, insurer, and user-notification paths; preserve timelines and affected-data evidence.
- [ ] **IR-08 — Conduct post-incident review.** Identify root causes, control gaps, detection failures, and systemic fixes; update this checklist and regression tests.
- [ ] **IR-09 — Reassess dormant features and assets.** Remove unused domains, buckets, credentials, APIs, queues, preview environments, and provider apps.
- [ ] **IR-10 — Schedule recurring access reviews.** Review administrators, cloud roles, production access, service accounts, support roles, API keys, and third-party access.

---

## 24. Conditional risk modules

Apply additional specialist standards and review when any of the following are present:

- [ ] **RISK-01 — Payments or card data.** Minimize scope through hosted/tokenized provider flows; follow applicable payment-provider and PCI DSS requirements.
- [ ] **RISK-02 — Healthcare or highly sensitive personal data.** Perform legal/privacy/security review for applicable health-data obligations and breach handling.
- [ ] **RISK-03 — Financial advice, trading, lending, or stored value.** Add fraud controls, reconciliation, auditability, suitability/approval rules, and regulatory review.
- [ ] **RISK-04 — Children or education.** Add age-appropriate design, parental/guardian controls where required, content safety, and strict data minimization.
- [ ] **RISK-05 — Biometrics, identity documents, or age verification.** Use specialist providers and controls for spoofing, retention, irreversible identifiers, and human review.
- [ ] **RISK-06 — End-to-end encryption.** Obtain cryptographic protocol review; define identity verification, metadata exposure, key backup/recovery, device addition, and multi-device behavior.
- [ ] **RISK-07 — Public content, messaging, or marketplaces.** Add moderation, reporting, blocking, anti-harassment, spam/scam controls, evidence preservation, and appeals.
- [ ] **RISK-08 — Location, camera, microphone, contacts, or background permissions.** Request only when needed, explain use, minimize retention, and use platform permission best practices.
- [ ] **RISK-09 — Browser extensions.** Minimize host permissions, secure message passing, protect update/signing, isolate content scripts, and review store policies.
- [ ] **RISK-10 — IoT or hardware.** Add secure boot, signed updates, unique credentials, physical threat assumptions, device identity, safe reset, and support lifecycle.
- [ ] **RISK-11 — Blockchain or smart contracts.** Obtain specialist review, use explicit invariants, pause/upgrade governance, oracle protection, key custody, and adversarial economic testing.
- [ ] **RISK-12 — High-risk AI decisions.** Add human review, quality/fairness testing, appeal/correction, traceability, drift monitoring, and domain/legal governance.

Marking a specialist control `N/A` requires confirming that the related feature or data is absent. Controls may be grouped only when one evidence-backed absence claim applies to the whole group; otherwise decide them individually.

---

## 25. Release gate: minimum conditions before production

The developer agent must not recommend production release until all applicable conditions below are true. A condition may be grouped `N/A` only when every capability it names is demonstrably absent; otherwise record the unmet part explicitly.

- [ ] **RELEASE-01 —** No unresolved **Critical** or **High** security findings remain.
- [ ] **RELEASE-02 —** Authentication, recovery, authorization, and tenant-isolation tests pass.
- [ ] **RELEASE-03 —** Secrets are not present in client artifacts, repository history, images, logs, or public configuration.
- [ ] **RELEASE-04 —** Production uses correct provider modes, domains, callbacks, credentials, and data stores.
- [ ] **RELEASE-05 —** Database/storage policies and privileged service paths were reviewed and tested.
- [ ] **RELEASE-06 —** Rate limits, quotas, payload limits, timeouts, and cost circuit breakers exist for expensive or abusable operations.
- [ ] **RELEASE-07 —** Security headers, TLS, cookie settings, CORS, CSP, and cache behavior are verified in the deployed environment.
- [ ] **RELEASE-08 —** Dependency, secret, static, and infrastructure scans are complete and reviewed.
- [ ] **RELEASE-09 —** Logging and alerts cover high-impact events without recording secrets.
- [ ] **RELEASE-10 —** Backups, restoration, rollback, and incident contacts are documented and tested proportionate to risk.
- [ ] **RELEASE-11 —** Privacy, retention, deletion, export, and third-party data flows are documented.
- [ ] **RELEASE-12 —** Administrative and production access uses least privilege and MFA.
- [ ] **RELEASE-13 —** Every `PARTIAL`, `BLOCKED`, and `RISK ACCEPTED` item has a written justification, and every individual or grouped `N/A` decision has a compact evidence-backed absence claim.
- [ ] **RELEASE-14 —** The security completion report is generated and reviewed by the responsible human owner.

---

## 26. Required security completion report

At the end of implementation or review, produce a DELTA-mode report in this format. Omit empty optional sections and do not enumerate verified controls. Use FULL mode only when explicitly requested.

```markdown
# Security Completion Report

## Application and review scope
- Application/version/commit:
- Environment(s) reviewed:
- Capability map (compact `key=on/off/unknown` form):
- Sensitive data/high-impact actions:
- Review date:
- Report mode: DELTA

## Coverage
- Counts: VERIFIED=<n>; PARTIAL=<n>; OPEN=<n>; N/A=<n>; BLOCKED=<n>; RISK_ACCEPTED=<n>; TOTAL=<sum>
- Evidence ledger or key evidence pointers:

### Grouped exclusions
| IDs | Verified absence claim | Evidence |
|---|---|---|

## Findings
| ID | Severity | Finding/gap | Affected area | Fix or safest next action | Evidence | Status |
|---|---|---|---|---|---|---|

## Automated checks
| Check | Result | Evidence or command summary |
|---|---|---|

## Release decision
- Recommended: YES / NO / CONDITIONAL
- Release blockers:
- Required follow-ups (owner/date):
- Human risk acceptances (owner/expiry), if any:
```

A report containing only “all checks passed” is insufficient. A no-finding report may be short, but it still needs scope, coverage counts, grouped exclusions, evidence pointers, automated-check results, and a release decision. `unknown` gates and unavailable external configuration must be visible as gaps; concise output must not imply verification that did not occur.

---

## 27. Risk exception requirements

A security control may be deferred only when an authorized human explicitly approves a risk exception containing:

- the exact issue and affected assets;
- severity, likelihood, impact, and exposure;
- why immediate remediation is not feasible;
- compensating controls and monitoring;
- owner and accountable approver;
- target remediation date and exception expiration;
- rollback/containment plan if exploitation occurs.

Exceptions must be time-bounded and re-reviewed. “Low probability,” “internal only,” “temporary,” “the framework handles it,” or “we will fix it later” are not sufficient by themselves.

---

## 28. Completion and discovery rule

**This checklist is a minimum baseline, not an exhaustive catalog.** If the developer agent discovers, suspects, or can reasonably infer any vulnerability, insecure default, misconfiguration, privacy issue, fraud path, abuse path, data-integrity risk, supply-chain weakness, unsafe AI behavior, or operational security gap that is not explicitly listed here, the agent must:

1. investigate and validate the issue;
2. classify its severity and affected scope;
3. implement an appropriate defense-in-depth mitigation;
4. add regression tests and monitoring where applicable;
5. document the fix and verification evidence; and
6. block release if the unresolved risk is Critical or High.

The agent must not ignore an issue merely because it is absent from this document. If remediation cannot be completed, it must be surfaced clearly for a time-bounded, explicit human risk decision; the agent may not silently accept the risk.

---

## 29. Reference standards and guidance

Use the latest stable versions applicable to the technology and risk profile. This checklist is informed by, and should be supplemented with:

- [OWASP Application Security Verification Standard (ASVS)](https://owasp.org/www-project-application-security-verification-standard/), currently version 5.0.0
- [OWASP Top 10 for Web Applications](https://owasp.org/Top10/), currently the 2025 edition
- [OWASP API Security Top 10](https://owasp.org/API-Security/)
- [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Mobile Application Security Verification Standard (MASVS) and Testing Guide (MASTG)](https://mas.owasp.org/)
- [OWASP guidance for LLM, GenAI, and agentic application security](https://genai.owasp.org/)
- [NIST Secure Software Development Framework (SSDF)](https://csrc.nist.gov/Projects/ssdf), including [NIST SP 800-218A](https://csrc.nist.gov/pubs/sp/800/218/a/final) for generative AI and dual-use foundation models where applicable
- [NIST Cybersecurity Framework (CSF)](https://www.nist.gov/cyberframework)
- [IETF OAuth 2.0 Security Best Current Practice (RFC 9700)](https://www.rfc-editor.org/rfc/rfc9700.html) and relevant OIDC/SAML standards
- Provider-specific secure configuration guidance for the selected cloud, identity, database, payment, storage, and AI platforms
- Applicable legal, regulatory, contractual, and industry requirements

When this checklist conflicts with a newer authoritative security standard or a stricter applicable requirement, follow the newer or stricter requirement and document the decision.
