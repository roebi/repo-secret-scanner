# repo-secret-scanner

Scheduled GitHub Actions workflow that scans all `roebi/*` repositories for leaked secrets using [TruffleHog](https://github.com/trufflesecurity/trufflehog).

## How it works

1. Reads `repos.txt` to build a dynamic scan matrix
2. Resolves the newest TruffleHog release that is **≥ 3 days old** (supply chain hygiene)
3. Verifies the TruffleHog binary via SHA256 before executing it
4. Checks out each repo with **full git history** via anonymous HTTPS (no token)
5. Severs all git remote info before scanning — fully offline from that point
6. Scans via `file://` protocol — no outbound traffic during scan phase
7. Cleans up all scan directories on every run, including on failure
8. On findings: creates a **private draft Security Advisory** in this repo — visible only to the maintainer, never public

## Schedule

Runs every **Monday at 06:00 UTC**. Trigger manually anytime via `workflow_dispatch`.  
Re-runs automatically when `repos.txt` is changed.

## Setup

### 1. Enable Private Vulnerability Reporting

In `repo-secret-scanner` → Settings → Code security → enable **Private vulnerability reporting**.

### 2. Create `ADVISORY_PAT`

Go to GitHub → Settings → Developer Settings → **Fine-grained personal access tokens** → Generate new token.

- **Repository access**: Only `roebi/repo-secret-scanner`
- **Permissions**: `Repository security advisories` → **Read and write**
- Nothing else

Store it in this repo → Settings → Secrets → Actions → `ADVISORY_PAT`.

> The `GITHUB_TOKEN` built into Actions cannot create security advisories — that is why a PAT is needed. This PAT is scoped to this repo only and has no access to any scanned repository.

### 3. Add repos to scan

Edit `repos.txt` — one repo name per line (no `roebi/` prefix). Pushing triggers an immediate scan.

## Security design

| Concern | How it is addressed |
|---|---|
| No permissions on scanned repos | Anonymous HTTPS clone, no token, scan jobs have `permissions: {}` |
| No public disclosure of findings | Private draft Security Advisory — maintainer-only, never auto-published |
| Container/runner leftovers | `mktemp -d` + `rm -rf` in `if: always()` cleanup step |
| TruffleHog supply chain | Newest release ≥ 3 days old, SHA256 verified before execution |
| Token leakage during scan | `unset GITHUB_TOKEN GH_TOKEN GITHUB_ACTIONS CI` before TruffleHog runs |
| No network during scan | `file://` protocol — local only after clone |
| Advisory scope | `ADVISORY_PAT` scoped to this repo only, `repository_advisories: write` only |
| Advisory content | Only repo name (no `roebi/`), finding count, run URL — never raw secret values |
| Scanner repo itself | `dependabot.yml` keeps action pins current, scheduled Sunday before Monday scan |

## On a finding

You will see a new entry under **Security → Advisories** (draft, private) in this repo.

1. Rotate the exposed credential **immediately** at the provider
2. Revoke the old token
3. Remove from git history: `git filter-repo --path <file> --invert-paths`
4. Force-push the affected repo
5. Contact GitHub Support to purge any cached views of the commit
6. Close / dismiss the advisory when resolved

## License

MIT
