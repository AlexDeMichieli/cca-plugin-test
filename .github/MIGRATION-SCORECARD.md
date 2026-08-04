# Jenkins to GitHub Actions Migration Scorecard

## Scope
- Source: `/workspaces/Jenkinsfile`
- Target: `/workspaces/.github/workflows/ci.yml`
- Archive: `/workspaces/.github/ci-archive/Jenkinsfile`

## Migration Summary
- ✅ Converted declarative Jenkins pipeline stages: Checkout, Install, Test, Build, Publish.
- ✅ Preserved pipeline environment variables (`NODE_VERSION`, `DOCKER_REGISTRY`).
- ✅ Mapped `when { branch 'main' }` to `if: github.ref == 'refs/heads/main'`.
- ✅ Mapped Jenkins build number to `${{ github.run_number }}`.
- ✅ Replaced `archiveArtifacts` with `actions/upload-artifact`.
- ✅ Replaced `junit` publishing with artifact upload of `test-results/*.xml` (minimal, no extra reporter action).
- ✅ Replaced Jenkins credentials binding with GitHub secrets:
  - `DOCKER_REGISTRY_USERNAME`
  - `DOCKER_REGISTRY_PASSWORD`
- ✅ Added explicit `permissions` block.
- ✅ Pinned all referenced actions to commit SHAs.

## Notes / Behavior Differences
- Jenkins `cleanWs()` has no direct equivalent; GitHub-hosted runners are ephemeral, so workspace cleanup is implicit per job.
- Docker push step assumes image/tag exists locally as in original pipeline behavior; no additional image build step was introduced to keep migration minimal.

## Validation
- Workflow syntactically generated and ready for repository validation (e.g., actionlint in CI).

## Completion Checklist
- [x] Workflow created at `.github/workflows/ci.yml`
- [x] Jenkinsfile archived at `.github/ci-archive/Jenkinsfile`
- [x] Scorecard created at `.github/MIGRATION-SCORECARD.md`
- [x] Actions pinned to SHAs
- [x] Explicit permissions configured
## 2026-08-04T15:14:42Z
- session: call_N3cSahqezSeIFlQHVpM02KTO
- reason: complete
- workflows: total=1, clean=1, with_issues=0

| workflow | status |
|---|---|
| ci.yml | clean |

