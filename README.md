# 📦 Setup

Please **fork** the repo. You can work locally using a code editor, or open it in GitHub Codespaces.

---

# 🟢 Task 1 (5 pts) — CI Quality Gate: Tests + Lint + Formatting

## 🎯 Goal
Set up a **GitHub Actions CI workflow** that runs automatically for every **Push** and **Pull Request** and blocks merging if:
- unit tests fail
- linting fails
- formatting check fails

You should get a ✅/❌ status check on the PR.

---

## ✅ Requirements
1. Create a workflow file:  
   **`.github/workflows/ci.yml`**

2. The workflow must run on:
- `pull_request` events (PR opened / updated)

3. The workflow must run **all of the following** checks:
- **Tests**: `pytest` (or `python manage.py test`)
- **Lint**: `ruff check .`
- **Formatting**: `black --check .`

4. If any check fails, the workflow must fail.

---

## 💡 Hints
- Use `actions/checkout` to fetch the repository code.
- Use `actions/setup-python` to set up Python on the runner.
- Install dependencies from `requirements-dev.txt` (recommended).
- Keep it simple: one workflow file with **two jobs** is fine (e.g. `lint` and `test`).

Optional (nice):
- Use caching for pip to speed up runs (e.g. via `actions/setup-python` cache option).

---

## 📦 How to show your work?
- Change something in the code on a new branch and push it.
- Open a Pull Request containing your workflow file.
- Ensure the PR shows the CI checks and they are ✅ green.
- Break the tests. Push the changes and see that the checks are ❌.

---

# 🟡 Task 2 (10 pts) — DevSecOps + PR Automation (CodeQL + Dependabot + PR Comment)

## 🎯 Goal
Enhance your repository with **security automation** and **PR feedback** by implementing:
- **CodeQL code scanning** (results visible in the repository **Security** tab)
- **Dependabot** dependency update configuration
- An **automatic PR comment** posted when CI is ✅ green (tests + lint + formatting passed)

---

## ✅ Requirements

### (A) CodeQL (Code Scanning → Security tab)
1. Create a workflow file:  
   **`.github/workflows/codeql.yml`**

2. The workflow must run on:
- `push` to `main`
- `pull_request` targeting `main`

3. The workflow must upload CodeQL results so that they appear in:
- **Security → Code scanning alerts**

**Hint:** Use the CodeQL GitHub Action steps (advanced setup style) located in Security → Code scanning alerts → Set up code scanning:
- `github/codeql-action/init`
- `github/codeql-action/autobuild`
- `github/codeql-action/analyze`

**Hint (permissions):** CodeQL upload requires `security-events: write` in that workflow.

---

### (B) Dependabot (automatic dependency update PRs)
1. Create a config file:
   **`.github/dependabot.yml`**

2. Configure Dependabot for:
- `pip` ecosystem (Python dependencies)
- schedule interval: **weekly** (recommended)

---

### (C) Automatic PR comment when CI succeeds
1. Update your CI workflow (**`.github/workflows/ci.yml`**) to add a job that:
- runs **only for Pull Requests**
- runs **only after** `lint` and `test` jobs succeed (`needs: [lint, test]`)
- posts a PR comment like:  
  `✅ CI passed: tests + lint + formatting`

2. The comment should **not spam** the PR:
- either update the same comment each run (recommended)
- or ensure only one comment is created

**Hint (how):**
- Use `actions/github-script` to call the GitHub API and create/update a comment  
  (PRs are treated as “issues” in the API)
- Or use a marketplace action for PR comments

**Hint (permissions):**
- For commenting you will likely need: `issues: write` (or `pull-requests: write`) for that job.

---

## 📦 How to show your work?
- Open a PR with the new files:
  - `.github/workflows/codeql.yml`
  - `.github/dependabot.yml`
  - updated `.github/workflows/ci.yml` (adds PR comment job)
- Ensure:
  - CodeQL workflow runs successfully
  - Dependabot config is present
  - PR gets a ✅ success comment after green CI

---

# 🔴 Task 3 (15 pts) — Build → Scan → Push Docker image (GitHub Secrets required)

## 🎯 Goal
Create a GitHub Actions workflow that works like a **delivery pipeline**:

1) **Build** the Docker image from this repository  
2) **Scan** the image with a **non-GitHub** container security scanner (e.g., Trivy)  
3) **Push** the image to **Docker Hub**  
4) Authenticate using **GitHub Secrets** (no credentials in code)

This task focuses on secure automation and artifact delivery (image in a registry), not server deployment.

---

## ✅ Requirements

### (A) Docker Hub auth via GitHub Secrets
1. In your forked repository, add GitHub Actions secrets:
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

> The token will be provided during class (whiteboard).  
> Put it in GitHub → **Settings → Secrets and variables → Actions**.

2. Your workflow must log in to Docker Hub **using these secrets** (no plaintext secrets in yml).

---

### (B) Workflow file and triggers
1. Create a workflow file:
- **`.github/workflows/docker.yml`**

2. The workflow must run on:
- `push` to `main`

3. Important security rule:
- **Do not push images to Docker Hub on `pull_request` events**  
  (PR workflows should never require registry secrets).

---

### (C) Image naming convention (must include your identity)
To avoid collisions (everyone pushing to the same Docker Hub account), your tag must include your **FirstName-LastName**.

✅ Required tag pattern (example):
- `DOCKERHUB_USERNAME/<shared-repo>:firstname-lastname-<shortsha>`

Example:
- `shareduser/tsd-cicd:pawel-kelar-a1b2c3d`

---

### (D) Build & Push using official Docker actions
Your workflow must use GitHub Actions that are designed for Docker workflows (recommended set):
- Docker login action
- Buildx setup
- Build and push action

---

### (E) Container scanning (non-GitHub scanner)
Add a container security scan step **before pushing**, using a tool like **Trivy**.

Requirements for scanning:
- Scan the **built image**
- Fail the workflow when vulnerabilities of selected severities are found (choose at least `HIGH,CRITICAL`)
- Scanning must happen **before** push (as a quality gate)

*(This is your “DevSecOps in CD” part.)*

---

## 💡 Hints
- Use `docker/setup-buildx-action` for modern builds (Buildx)
- Use `docker/login-action` for registry authentication (Docker Hub)
- Use `docker/build-push-action` for building + pushing the image
- Use `aquasecurity/trivy-action` to scan your image
- You can generate `shortsha` from `${{ github.sha }}` in a step and reuse it later
- Consider conditional push logic (push only if scan passed)

---

## 📦 How to show your work?
- Merge your workflow into `main` (in your fork).
- Push a commit to `main`.
- In the **Actions** tab, show a successful run that:
  1) builds the image
  2) scans it
  3) pushes it to Docker Hub
- Verify on Docker Hub that your image tag exists and includes your name.

---