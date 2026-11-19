# Linux runner architecture standard

Status: staged rollout. No organization runner variable or paid budget change is implied by this document.

## Intent

Prefer the standard GitHub-hosted `ubuntu-24.04-arm` image for compatible Linux jobs across Labyrinth Foundation. Preserve x64 and macOS for native tools, container images, visual baselines, platform-specific signing, and release requirements that cannot yet be validated on ARM64.

This is an operating-system architecture preference, **not** a mandate to move every job to ARM64 or to a self-hosted runner. GitHub-hosted ARM64 still consumes the organization's included Actions compute allowance.

## Shared setting

Use organization Actions variable `LF_LINUX_RUNNER` for jobs that have passed the ARM64 compatibility gate. During preparation, set it to `ubuntu-24.04` to preserve today's behavior. After validation, change it to `ubuntu-24.04-arm`.

In eligible jobs, use:

```yaml
runs-on: ${{ vars.LF_LINUX_RUNNER || 'ubuntu-24.04' }}
```

The fallback prevents an unset variable from silently activating unvalidated ARM64 jobs. Configuration variables cannot override a workflow's literal `runs-on`; workflows must opt in to the variable. Keep jobs with documented exceptions on explicit labels.

An organization owner must create and scope this variable in **Organization Settings > Secrets and variables > Actions > Variables**. This document and its PR do not create that setting.

## Initial compatibility pilot

Use isolated, non-deploying PR validation jobs in these private repositories:

* `steam-ci`: standard-library Python and shell syntax tests only; leave Godot build and Steam upload jobs unchanged
* `research`: `validate-library.yml` only; leave Pages publishing jobs unchanged
* `ghostnote-levels-backend`: `ci.yml` only; do not change deployment or catalog refresh
* `ghostnote-levels-content`: `pr-validate.yml` only; do not change publishing
* `donations-api`: `validation.yml` only; Wrangler dry runs must not deploy

The first pilot PRs explicitly select `ubuntu-24.04-arm` to establish actual compatibility. Once they pass, migrate those jobs to the shared variable above. Avoid introducing variable-based defaults across the other repos until this evidence exists.

## Follow-up candidates

* Ordinary Node/Python checks in `google-ads-local-reports`, `rondoodles`, `less-is-more-content`, and `labyrinth-infrastructure`, after dependency and network/auth checks. Do not reenable retired tracking or dormant diagnostics.
* WebKit/Chromium and build workflows in `website`, `less-is-more-web`, and `labyrinth-events`, after verifying exact Playwright Docker digest architecture, browser installations, screenshot baselines, and reproducible generated outputs.
* Godot and Steam tooling only after checking ARM64 distribution, SHA checksums, and compatibility.

## Exceptions

* `GhostNote` validation downloads an x86_64 Godot binary.
* `steam-ci`'s toolchain manifest provides Linux/X64 and macOS/ARM64, not Linux/ARM64; macOS signing/notarization must remain on macOS.
* `ghostnote-creator-godot` currently downloads Linux SteamCMD and performs Steam release tasks that must not be switched speculatively.
* `Arcade` uses `barichello/godot-ci:4.7`, whose ARM64 platform must be verified.
* `labyrinth-infrastructure` secret scan uses a checksum-pinned Linux x64 Gitleaks download.
* `website` uses digest-pinned Playwright containers and canonical visual baselines; preserve their architecture until verified.

Leave archived repositories unchanged.

## Acceptance and rollback

1. Confirm the job runs on Linux ARM64 using `runner.arch` or `uname -m`. Do not log secrets.
2. Preserve workflow triggers, job IDs/names, required status checks, permissions, caches, release gates, and artifact shapes.
3. Pass the existing checks on ARM64, including browser/visual validation where relevant. Compare generated build artifacts, not just exit codes.
4. Compare actual GitHub billed usage, job duration, retries, and reliability with x64. ARM64's lower per-minute rate does not guarantee savings if jobs slow down.
5. Keep the existing $0 Actions stop-usage safeguard and do not bypass branch protection.
6. If a workflow cannot run reliably on ARM64, restore `ubuntu-24.04` for that job and record the exception.

On November 19, 2026, the organization has exhausted its hosted Actions allowance; issue [website #57](https://github.com/Labyrinth-Foundation/website/issues/57) records the expected December 1 reset. Do not repeatedly trigger hosted jobs before the reset solely to test this migration. Do not enable auto-merge of unvalidated migration PRs.

This policy establishes the intended direction. The variable and migration are complete only when GitHub configuration and individual job results are verified.
