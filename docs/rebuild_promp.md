# Reusable prompt: multi-cloud account & resource dashboard

Use this document to recreate **similar or equivalent** applications in future projects. Replace the bracketed placeholders (`[APP_NAME]`, `[COMPANY]`, etc.) with your product name, organization, or constraints. Branding may change; **capabilities and patterns** should stay aligned unless you explicitly scope them down.

---

## Copy-paste master prompt

**Goal:** Build a web application named **"[APP_NAME]"** that behaves like a **multi-cloud account vending and operations dashboard**: provision and manage AWS accounts and Azure subscriptions, inspect resources across accounts, support DR-style reprovision, billing views, and admin tooling. The **branding and copy** may differ from any existing project, but **capabilities and UX patterns should match** the following.

### Architecture

- **Backend:** Python **Flask** (REST JSON API + server-rendered HTML templates for login/signup/dashboard shell).
- **Frontend:** **Vanilla JavaScript (ES6+)**, **no React/Vue build step**; single main dashboard page acting as a **tab/section SPA**; **Fetch API** + **`localStorage`** for JWT; `credentials: 'include'` for session cookies where used.
- **Auth:** Email login, password hashing, **JWT** (Bearer) with expiry, **Flask-Login**-style session support as applicable; **RBAC** with at least **admin** and **user**; protect API routes; **401 → redirect to `/login`**.
- **Caching:** **Flask-Caching** for expensive read endpoints (configurable TTL; optional Redis later).
- **CORS:** Flask-CORS where needed.
- **Persistence (v1):** **JSON files** under `state/` (e.g. `users.json`, metadata per account); **rotating file logs**; **Terraform** for provisioning with state under `state/` or equivalent.
- **Integrations:** **AWS** via **boto3** (Organizations, STS assume-role, EC2, S3, IAM, VPC, etc. as required); **Azure** via **azure-identity** (DefaultAzureCredential) and **azure-mgmt-*** for subscriptions/resources.

### UI / UX (console-style dashboard)

- **Layout:** Top bar (logo + app title, search, notifications/help placeholders, user menu with sign out), **left sidebar** with grouped nav, **main content** with page title/breadcrumb and **tab panels**.
- **Visual style:** Clean, cloud-console-inspired (similar to Google Cloud Console density and components: cards, tables, modals, loading states). Use separate CSS files: base, dashboard, components.
- **Responsive:** Works on desktop; reasonable behavior on smaller widths.
- **Tables:** Sort/filter where it helps; **CSV export** on the client for large lists where applicable.
- **Long operations:** Extended timeouts or progress messaging for slow APIs (e.g. aggregated CloudTrail-style queries).

### Functional areas (sidebar / sections)

1. **Dashboard** – Overview stats (total accounts, breakdown by provider, recent activity placeholders).
2. **Account management**
   - **Provision account** – Form: cloud provider (AWS/Azure), account/subscription name, region, org-specific options; calls provision API; surfaces success/errors.
   - **Manage accounts** – List provisioned accounts from app metadata/API; actions: detail, delete, reprovision as applicable.
   - **Existing accounts** – Discover/list accounts from cloud providers (management account / subscriptions).
3. **Resources**
   - **Global resources** – Selector for resource type (e.g. EC2 instances, S3 buckets, VPC/subnets, IAM users, Lambda, etc.); aggregate across accounts via assumed roles; export CSV.
   - **Account resources** – Scoped view for one account.
   - **EKS clusters** / **ECS clusters** – List/describe as applicable.
   - **Console logins** – Aggregated console sign-in style report with refresh + CSV export; tolerate long-running queries.
4. **Billing & costs** – Section for cost/billing summaries or links (integrate with provider APIs or placeholders with clear extension points).
5. **Disaster recovery** – **Reprovision** flow driven by stored metadata + Terraform (or equivalent IaC) to rebuild accounts/subscriptions.
6. **Identity & access (admin)** – e.g. assign permission sets / Identity Center–style workflows if using AWS SSO; gate nav for non-admins.
7. **Admin** – **User management** (list users, roles); show admin-only nav items only for admins.

### API surface (minimum)

- Auth: `POST /login`, `POST /logout`, `POST /signup`, `GET /api/auth/me`.
- Accounts: `GET /api/accounts`, `POST /api/accounts/{provider}/{name}`, `GET`/`DELETE` detail, `POST .../reprovision`.
- Resources: `GET /api/instances`, `/api/buckets`, `/api/subnets`, `/api/iam-users`, `/api/management-accounts`, plus any EKS/ECS/billing endpoints the UI needs.
- Ops: `GET /api/health`, `POST /api/cache/clear`, `GET /api/logs` (if exposed).

### Non-functional

- **Security:** Secrets via env vars (`SECRET_KEY`, `JWT_SECRET_KEY`, etc.); no credentials in repo; document production hardening (HTTPS, WSGI server, bcrypt/argon2 upgrade path).
- **Repo layout:** `backend/` (Flask, `requirements.txt`, `auth.py`), `frontend/templates/` (`index.html`, `login.html`, `signup.html`), `frontend/static/css|js/`, `terraform/aws|azure/`, `state/`, `logs/`, `scripts/` (setup, backup/restore state), `docs/` (tech stack/spec).
- **Deliverables:** Working local run instructions, `config.example.env`, and a short **TECH_STACK**-style doc listing dependencies and endpoints.

**Customization line for each new project:**

> Use **[APP_NAME]** as the product name and **[COMPANY]** styling/colors if specified; keep the same features and API semantics unless I list explicit differences.

---

## How to use this prompt

| Scenario | What to add |
|----------|-------------|
| Same app, new name | Replace `[APP_NAME]` / logo text / CSS tokens only. |
| Smaller scope | Add: "Implement only sections: …" |
| Another cloud | Add: "Add GCP with analogous flows." |
| Stricter security | Add MFA, OAuth2, or database requirements explicitly. |

---

## Related docs in this repo

- [TECH_STACK.md](./TECH_STACK.md) – Detailed stack, endpoints, and file layout for the current implementation.

---

*This file is a specification template for greenfield or clone builds; it does not need to match line-by-line with current code if you intentionally evolve the product.*
