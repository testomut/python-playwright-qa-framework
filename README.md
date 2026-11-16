# Python Playwright QA Framework

A Python automation portfolio sample for a **fictional CRM / messaging application**.
It demonstrates page objects, layered pytest fixtures, API-assisted setup and cleanup,
role-based parameterization, typed test data and per-worker account isolation.

The public sample uses generic application names, illustrative API routes and synthetic
account data. It does not include an application server, real service credentials, customer
data or the original repository history. App-specific selectors and routes must be adapted
to a disposable test application before the full UI suite can run.

## What is verified

| Check | Scope |
|---|---|
| 15 offline unit cases | token expiry, typed data, phone formatting, REST requests and role switching |
| 2 local Chromium cases | shared Playwright fill/click commands and masked password logging |
| 50 collected E2E cases | contacts, tags, custom fields and saved replies; opt-in, requires an adapted app |
| Ruff | lint and formatting |
| CI template | prepared GitHub Actions workflow in `ci/github-actions.yml`; not enabled |

**Collection is not an E2E pass.** The 50 application scenarios are skipped by default.
The two local-browser cases use `page.set_content`; the API checks use `httpx.MockTransport`.

## Stack

- Python 3.11+, pytest, Playwright sync API, pytest-playwright
- pytest-xdist for worker isolation; pytest-timeout
- httpx for authentication, setup and cleanup
- Pydantic v2 for account and test-data validation
- Optional Allure steps and reporting
- Ruff and a GitHub Actions workflow template

## Quick start

```bash
python -m venv .venv
# Windows PowerShell: .\.venv\Scripts\Activate.ps1
# Linux/macOS: source .venv/bin/activate
pip install -r requirements-dev.txt
python -m playwright install chromium
python -m pytest tests/unit tests/demo -q
```

No `.env` or provisioned account is needed for these checks.

```bash
python -m ruff check .
python -m ruff format --check .
python -m pytest --collect-only -m e2e -q
python -m pytest tests/unit tests/demo -n 2
```

On Linux CI, install Chromium with `python -m playwright install --with-deps chromium`.

## Architecture

```text
Tests
  -> layered pytest fixtures (accounts / cleanup / seeded data / page objects)
     -> Playwright page objects -> shared interaction commands
     -> httpx REST client       -> illustrative API contract
     -> Pydantic models         -> validated account and entity data
```

| Location | Purpose |
|---|---|
| `src/pages/` | base, settings, tags, contacts, custom fields, saved replies, conversations |
| `src/utils/custom_commands.py` | reusable interactions, logging and optional Allure steps |
| `src/utils/api_requests.py` | authentication, token refresh, setup and cleanup |
| `src/utils/parametrization_helpers.py` | re-authentication for Owner/Admin/Manager/Member |
| `src/types/models.py` | account aliases, custom-field types and typed test entities |
| `fixtures/` | worker accounts, API client lifecycle and cleanup dependencies |
| `tests/unit/` | executable offline framework checks |
| `tests/demo/` | executable local-browser checks |
| `tests/test_*.py` | illustrative application E2E scenarios |

The preferred flow is **arrange through the API, assert through the UI**. Mutable accounts
are assigned per xdist worker. Seed fixtures explicitly depend on cleanup; role switching
re-authenticates after changing permissions. JWT payload decoding checks cache expiry only;
it is not signature verification or an authorization decision.

## Adapting the application scenarios

1. Provide your own disposable test application with matching features.
2. Adapt page-object selectors and the illustrative `/api/v1/` routes/payloads.
3. Copy `.env.example` to `.env` and set the explicit UI and API base URLs.
4. Provision isolated worker accounts using `variables/accounts/account.example.json` as a
   shape reference. Save them as `account0.json`, `account1.json`, etc. Set `WORKERS` to match.
5. Configure the separate recipient account for scenarios that need seeded conversations.
6. Opt into the application suite:

```bash
python -m pytest --run-e2e -m e2e --headed
python -m pytest --run-e2e -m e2e -n 2
```

Use disposable accounts: cleanup removes tags and custom fields and restores role permissions.
A pinned `ACCOUNT` is for single-worker debugging. No registration, billing, email/2FA service
or live messaging provider is included. Some legacy helper methods need an adapter before use.
Cleanup failures are logged by the current sample; stricter teardown failure policy is a
possible extension when integrating with a real test environment.

## Reports and debugging

```bash
python -m pytest tests/demo --headed --slowmo 200
python -m pytest tests/unit tests/demo --junitxml=test-results/results.xml
pip install allure-pytest
python -m pytest tests/unit tests/demo --alluredir=allure-results
```

Generated traces, logs, reports, local account files and token caches are ignored by Git.
Only synthetic examples are included. The image attachment is generated test data.

## Portfolio scope

This repository demonstrates Python QA automation architecture. It is a portfolio sample,
not a commercial product and not evidence that the adapted application suite passed against
a real service. The CI template runs the reusable checks that are executable here. To enable it, copy
`ci/github-actions.yml` to `.github/workflows/checks.yml` using credentials with workflow
write permission. The template has not been run on GitHub; the 17 checks passed locally.
