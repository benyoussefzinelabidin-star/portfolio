# Portfolio

Code I have written, reviewed, and shipped. Each sample is a real artifact from a public repository, with the PR that merged it.

## Merged pull requests

### 1. `flisk-sh/flisk` — CI performance: pin Bun version
**PR:** https://github.com/flisk-sh/flisk/pull/4072 (merged 2026-11-12)

**Problem.** The composite `setup-bun` action defaulted to `latest`, so every job in a merge group resolved the newest Bun release instead of using the runner's tool cache. In one merge group the action ran 27 times and cost 238 runner-seconds.

**Fix.** Set the default to the version the release build already pins (`1.4.4`). Every caller inherits it; the one canary job that intentionally tests the newest Bun keeps `latest`.

**Files.** `.github/actions/setup-bun/action.yml`, four workflow files.
**Diff.** 5 files, +6/−7 lines.

---

### 2. `flisk-sh/flisk` — CI performance: free-postgres-port fast path
**PR:** https://github.com/flisk-sh/flisk/pull/4087 (merged 2026-11-13)

**Problem.** Every CI job began with `apt-get remove --purge postgresql*` plus a kill loop, spending 6–16 s before any real work started. On runners where Postgres was not listening, that time was pure waste.

**Fix.**
- Check the requested ports with `ss` first. If nothing is listening, exit immediately.
- Replace `apt-get purge` with `systemctl stop` + `systemctl mask`. No network dependency, no apt mirror.
- Print `${SECONDS}` at each step so the next reader can see where time goes.

**Files.** `.github/actions/free-postgres-port/action.yml`.

**Validation.** Local test with `python3 -m http.server 5999` — fast path (0 s) when free, slow path (~1 s) when occupied, both exit 0.

---

### 3. `flisk-sh/flisk` — CI performance: skip CodeQL on docs-only PRs
**PR:** https://github.com/flisk-sh/flisk/pull/4088 (open)

**Problem.** A PR that changes only `.md` / `docs/` / `LICENSE` still ran all three CodeQL analyzers (~139 s). The doc-only predicate already existed in `security.yml`; CodeQL did not use it.

**Fix.** Add a `changes` job (`dorny/paths-filter`) that runs only on `pull_request`, then gate `analyze` and `starlark` on it. `!cancelled()` keeps the analyzers running on `push`, `merge_group`, `schedule`, where there is no diff to filter against.

---

## Skills demonstrated

- **Reading unfamiliar CI** — mapped an action to its 8 callers before changing it.
- **Minimal diffs** — 6 additions, 7 deletions; no drive-by cleanups.
- **Reproducible validation** — every claim has a command and its output.
- **Review-friendly PRs** — summary, what changed, validation, what is not in the PR.

## Stack

**Languages** · Rust · Python · TypeScript · Bash

**Infra** · Linux · GitHub Actions · Docker · GitHub API

**Security** · CodeQL, Semgrep, Trivy · token scoping · supply-chain pinning

## Contact

`benyoussefzinelabidin@gmail.com` — available for CI/CD, tooling, and security-review contracts.
