# Automated PR Reviewer

Fail-closed MVP for GitHub Pull Requests + Jira Cloud. It fetches a linked ticket and PR diff, evaluates acceptance criteria with an LLM, comments on the PR, and optionally requests a squash merge.

## Run locally
1. Install Python 3.11+.
2. Create and activate a virtual environment: `python -m venv .venv`.
3. Install dependencies: `pip install -r requirements.txt`.
4. Copy `.env.example` to `.env` and fill credentials locally. Never commit `.env`.
5. Export the variables from `.env` in your shell.
6. Start the API: `uvicorn app:app --reload --port 8000`.
7. Expose the service over HTTPS and configure GitHub Settings → Webhooks:
   - Payload URL: `https://YOUR-HOST/webhook/github`
   - Content type: `application/json`
   - Secret: the same value as `GITHUB_WEBHOOK_SECRET`
   - Subscribe to Pull requests.

## Safe demo
Keep `DRY_RUN=true` and `AUTO_MERGE=false`. Open a test PR with a Jira key (for example `PROJ-123`) in its title or branch. The webhook returns the verdict and intended comment without changing GitHub.

## GitHub permissions
The token needs pull-request read/write and commit-status write permissions. Jira credentials need issue read access.

To post comments and failure statuses, set `DRY_RUN=false`. Configure branch protection to require the `ai-pr-reviewer` status context; a failure status alone does not block merging.

Only consider `AUTO_MERGE=true` after validating in a disposable test repository. The merge API remains subject to GitHub rules and permissions. Keep a human approval gate for production.

## Fail-safe behavior
- Missing ticket ID: blocked; no merge.
- Missing/unavailable Jira ticket or empty criteria: blocked; no merge.
- Invalid JSON or invalid LLM response schema: blocked; no merge.
- Empty or oversized diff: blocked; no merge.
- API/network/runtime errors: blocked; no merge.
- Webhook HMAC signature is verified before processing.

## Scope and limitations
- GitHub + Jira Cloud only. Azure DevOps and GitLab adapters are not implemented.
- Posts a top-level PR comment, not inline line-level comments.
- Acceptance criteria are extracted from Jira description text; map a dedicated custom field for a real project.
- Oversized diffs are rejected rather than chunked.
- No persistent idempotency store yet; duplicate webhook deliveries can repeat work.
- The LLM sees only the diff; it does not run tests or inspect the full repository. This is an assistant to CI and human review, not a replacement.
- Add tests, durable event tracking, retries, audit logging, rate limits, secret management, and human approval before production.
