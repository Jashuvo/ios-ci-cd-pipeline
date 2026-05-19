# iOS CI/CD Pipeline — GitHub Actions

This repository demonstrates the full iOS build and release pipeline 
implemented in GitHub Actions, based on the documented iOS Build and 
Release Process.

## Pipeline Overview

| Trigger | What Happens |
|---|---|
| Push to `main` | Lint → Test → Build → Deploy to TestFlight (Internal) |
| Push of `v*.*.*` tag | Full pipeline → External Beta → App Store (with approval gates) |
| Manual dispatch | Choose target: internal / external / appstore |

## Pipeline Jobs

```
lint_and_test → build_and_sign → deploy_testflight_internal
                                       ↓ (on version tag)
                              deploy_testflight_external
                                       ↓ (manual approval)
                                 deploy_appstore
```

## Secrets Required

| Secret | Purpose |
|---|---|
| `MATCH_GIT_URL` | Private repo storing encrypted certificates |
| `MATCH_PASSWORD` | Encryption passphrase for the cert repo |
| `MATCH_GIT_PRIVATE_KEY` | SSH key to clone the cert repo |
| `APPLE_TEAM_ID` | Apple Developer Team ID |
| `ASC_KEY_ID` | App Store Connect API Key ID |
| `ASC_ISSUER_ID` | App Store Connect Issuer ID |
| `ASC_KEY_P8_CONTENT` | Contents of the `.p8` API key file |
| `SLACK_WEBHOOK_URL` | Slack notifications (optional) |

## Key Decisions

**Fastlane Match** — Used for code signing sync. All certificates and 
provisioning profiles are stored encrypted in a private Git repository 
so the CI server gets identical credentials to the developer's machine.

**App Store Connect API** — Used for authentication during artifact 
delivery. This bypasses Apple's Two-Factor Authentication (2FA) which 
would block automated uploads.

**GitHub Environments** — Three environments (`testflight-internal`, 
`testflight-external`, `appstore`) with required reviewers on the last 
two, acting as manual approval gates before any external deployment.

**Phased Release** — App Store submissions use Apple's 7-day phased 
rollout so a critical crash can be caught and the release halted before 
reaching 100% of users.

## Versioning

- `CFBundleShortVersionString` — Set from the git tag (e.g. `v1.2.3` → `1.2.3`)  
- `CFBundleVersion` — Set to the GitHub Actions run number (auto-increments)

To trigger a release:
```bash
git tag v1.0.0
git push origin v1.0.0
```
