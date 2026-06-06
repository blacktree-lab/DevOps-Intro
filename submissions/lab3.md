# Lab 3 Submission — CI/CD: A PR-Gated Pipeline for QuickNotes

**CI Path:** GitHub Actions

---

## Task 1 - Write the PR Gate

### 1.1 CI Configuration

CI config lives at `.github/workflows/ci.yml`. It runs three independent jobs:
- `vet` runs `go vet ./...`
- `test` runs `go test -race -count=1 ./...`
- `lint` runs `golangci-lint v2.5.0`

### 1.2 Design Questions

**a) Why pin the runner version (`ubuntu-24.04`) instead of `ubuntu-latest`?**
`ubuntu-latest` is a floating alias that GitHub reassigns when a new LTS is released. When it happens, pre-installed tool versions, kernel behaviour, and available system libraries can change overnight, silently breaking the pipeline. Pinning `ubuntu-24.04` guarantees the same environment on every run.

**b) Why split vet + test + lint into separate jobs?**
With a single combined job, a failure in `vet` cancels the entire job and you never learn whether `test` or `lint` would have passed. Three separate jobs run in parallel, finish faster, signalling exactly which check failed and why.

**c) What real attack does SHA pinning prevent?**
SHA pinning prevents a **tag-hijacking / supply-chain attack**. If a third-party action is referenced by tag (e.g. `@v4`), a compromised maintainer can silently push malicious code to that tag. The most prominent example is the **tj-actions/changed-files incident (March 2025)**: attackers pushed a backdoored commit to the tag used by thousands of repositories. Pinning by full 40-character commit SHA means your workflow runs exactly the code you reviewed.

**d) What is `permissions:` and what principle is behind it?**
`permissions:` restricts what the GitHub Actions token (`GITHUB_TOKEN`) is allowed to do in a workflow run. By default the token has broad write access to the repository. Declaring `permissions: contents: read` ensures the workflow only gets the minimum access it needs.

### 1.3 Green CI Run

[green_run_1.png]

Link to green CI run: https://github.com/blacktree-lab/DevOps-Intro/actions/runs/27049565155

### 1.4 Deliberate Failure + Fix

**What was broken:**
Changed `http.StatusCreated` to `http.StatusOK` in `app/handlers_test.go` to make the test expect 200 instead of 201.

[failed_run.png]

**Fix commit:** Reverted `http.StatusOK` back to `http.StatusCreated`.

[green_run_2.png]

### 1.5 Branch Protection

[main_rules.png]

---

## Task 2 — Cache + Matrix + Path Filter

### Timing Table

| Scenario | Wall-clock |
|----------|-----------|
| Baseline (no cache, single Go version, no path filter) | 25s |
| With cache | XX s |
| With cache + matrix | XX s |