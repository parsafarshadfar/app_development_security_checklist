# 🛡️ Universal Security Standard for AI-Assisted Projects (App development and AI Security Checklist)

> **Don’t let an AI coding agent accidentally build your startup’s bankruptcy.** "A single prompt injection can drain your LLM API wallet overnight. A single forgotten trust boundary can leak your entire database." 

Welcome to the ultimate **AI coding agent security checklist** and **LLM security baseline**. AI coders (like Cursor, Claude, Codex, Antigravity, etc) are incredibly fast and powerful, but they don't have common sense about security. They write code that "works," but they'll gladly skip validation, ignore rate limits, or link your LLM to destructive tools if you ask them to. 

This repository contains **app-security-checklist.md**—a lightweight, token-efficient security standard for AI-assisted software development, designed to prevent prompt injection vulnerabilities, DDOS cost abuse, data leaks, and API bill exhaustion.

---

## ⚡ The Nightmare (Why this exists)

If you let an AI write features without a security framework, you risk:
* **The Infinite Loop Bill:** An attacker triggers a loop in your LLM features, running up $10,000 in API costs in a few hours.
* **The "Works on my machine" Auth Bypass:** AI builds a sleek UI but forgets to verify permissions on the backend API.
* **Open Backdoors:** Sandbox escape, prompt injections, or debug ports left open in production.

By dropping this file into your repo, you give your AI agent a strict set of rules it *must* check and prove with evidence before you ship.

---

## 🚀 How to use it in 10 seconds

1. **Copy the checklist:** Drop **app-security-checklist.md** into the root of your project.
2. **Tell your AI agent to read it:** Before any release or major feature, copy-paste this prompt:
   > *"Read app-security-checklist.md. Audit our changes against it."*

---

## 📊 How it reports (The Delta Protocol)

AI tokens cost money, and no one wants to read a long report. The checklist defaults to **DELTA mode**, meaning the agent only reports the critical stuff:

* **🚨 Release Blockers & Critical Findings:** The scary stuff (missing API auth, hardcoded secrets) that must be fixed before deploying.
* **⚠️ Security Findings:** Medium/low issues that need tracking.
* **⏳ Incomplete Work:** Stuff that was started but not finished.
* **🚫 Blockers:** Things the AI couldn't check (e.g., missing API access or environment variables).
* **✍️ Risk Acceptances:** Active decisions where a *human* (never the AI!) explicitly **signed off on a risk**, noting why and who approved it.
* **🔄 Changed Controls:** Security mechanisms touched by recent edits that must be re-tested.

---

## 🔒 What it covers

security checks grouped by following domains:
* **Authentication & Identity (`AUTHN-*`)**: MFA, secure password recovery, rate-limiting, and session management.
* **Authorization (`AUTHZ-*`)**: Preventing IDOR/BOLA, property-level filtering, and multi-tenant leaks.
* **AI & LLM Security (`AI-*`)**: Hard walls around model-controlled tools, vector database sanitization, and strict API cost caps.
* **Data & Privacy (`DATA-*`, `PRIV-*`)**: Encryption, PII protection, and file upload safety.
* **Infrastructure (`CLOUD-*`, `SUPPLY-*`)**: Docker/Cloud configurations and dependency scanning.

---

## 🛠️ Prove it: Evidence is required

The AI can't just say "it's secure." To mark a control as `VERIFIED`, it has to log actual proof:
* Which files or code blocks enforce the check.
* The test case (including how it handles bad inputs/attacks).
* Command outputs or scan results.

## 📈 The Security Checklist Checksum

At the end of an audit, you get a clean scorecard to see where you stand:
```text
VERIFIED=<n> | PARTIAL=<n> | OPEN=<n> | N/A=<n> | BLOCKED=<n> | RISK_ACCEPTED=<n> | TOTAL=347
```

Keep your app safe, your credentials private, and your cloud bill under control. 🛡️
