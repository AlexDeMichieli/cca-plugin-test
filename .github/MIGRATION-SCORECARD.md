# Jenkins → GitHub Actions Migration Scorecard

**Source:** `Jenkinsfile` (declarative pipeline) → **Target:** `.github/workflows/ci.yml`
**Date:** 2026-08-04 · **Original archived at:** `.github/ci-archive/Jenkinsfile`

## Conversion summary

| Jenkins construct | GitHub Actions equivalent | Status |
|---|---|---|
| `agent any` | `runs-on: ubuntu-latest` | ✅ Converted |
| `environment { NODE_VERSION, DOCKER_REGISTRY }` | workflow-level `env:` | ✅ Converted |
| `stage('Checkout')` / `checkout scm` | `actions/checkout` | ✅ Converted |
| `stage('Install')` / `npm ci` | `run: npm ci` (+ `setup-node` with npm cache) | ✅ Converted |
| `stage('Test')` / `npm test` | `run: npm test` | ✅ Converted |
| `post { always { junit 'test-results/*.xml' } }` | `upload-artifact` with `if: always()` | ⚠️ Adapted |
| `stage('Build')` / `npm run build` | `run: npm run build` | ✅ Converted |
| `archiveArtifacts 'dist/**'` | `actions/upload-artifact` | ✅ Converted |
| `when { branch 'main' }` | job-level `if: github.ref == 'refs/heads/main'` | ✅ Converted |
| `withCredentials(usernamePassword 'docker-registry')` | `docker/login-action` + `secrets.REG_USER` / `secrets.REG_PASS` | ⚠️ Action required |
| `$BUILD_NUMBER` | `$GITHUB_RUN_NUMBER` | ✅ Converted |
| `post { always { cleanWs() } }` | Not needed — runners are ephemeral | ✅ Dropped |

**Automated conversion: 11/12 constructs (92%).**

## Manual follow-up required

1. **Secrets** — create repository secrets `REG_USER` and `REG_PASS` mapping to the
   Jenkins credential `docker-registry`. Consider OIDC federation instead of static creds.
2. **JUnit reporting** — Jenkins' `junit` step published test trend graphs. GitHub has no
   native equivalent; results are uploaded as artifacts. To restore rich reporting, add a
   test-reporter action (e.g. `dorny/test-reporter`) with `checks: write` permission.
3. **Docker image build** — the Jenkinsfile pushed `myapp:$BUILD_NUMBER` but never built or
   tagged it (an implicit dependency on prior Jenkins state). Add an explicit
   `docker build -t $DOCKER_REGISTRY/myapp:$GITHUB_RUN_NUMBER .` step before the push.
4. **Action SHA verification** — network access to the GitHub API was unavailable during
   migration, so pinned SHAs come from known release commits. Verify each pin resolves to
   the commented tag before merging (`git ls-remote https://github.com/<owner>/<repo> <tag>`).

## Security & hardening

- `permissions:` set explicitly at workflow level to `contents: read` (least privilege).
- All actions pinned to full 40-character commit SHAs with tag comments.
- No secrets hard-coded; credentials referenced via `secrets.*` only.
- `concurrency` group added to cancel superseded in-progress runs.
- Publish job gated to `push` events on `main`, preventing fork-PR credential exposure.

## Validation

- `actionlint` v1.7.7 — **0 errors, 0 warnings**.
## 2026-08-04T14:57:01Z
- session: toolu_01WEeijciYymQFdyoKMWyGc3
- reason: complete
- workflows: total=1, clean=1, with_issues=0

| workflow | status |
|---|---|
| ci.yml | clean |

