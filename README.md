# repo-secret-scanner

Scheduled GitHub Actions workflow that scans all `roebi/*` repositories for leaked secrets using [TruffleHog](https://github.com/trufflesecurity/trufflehog).

## How it works

1. Reads `repos.txt` to build a dynamic matrix
2. Checks out each repo with **full git history** (`fetch-depth: 0`)
3. Runs TruffleHog with `--only-verified` (reduces false positives)
4. Writes a summary to the GitHub Actions job summary
5. Opens an issue in **this** repo if verified secrets are found

## Schedule

Runs every **Monday at 06:00 UTC**. Trigger manually anytime via `workflow_dispatch`.

## Setup

### 1. Create a PAT

Go to GitHub → Settings → Developer Settings → Fine-grained tokens.  
Required permissions (for each target repo): **Contents: Read**, **Metadata: Read**.  
For auto-creating issues: **Issues: Write** on this repo.

### 2. Add the PAT as a secret

In this repo → Settings → Secrets → Actions → New secret:  
Name: `SCAN_PAT`

### 3. Add repos to scan

Edit `repos.txt` — one repo name per line. Pushing the change triggers an immediate scan.

## On a finding

1. Rotate the exposed credential **immediately**
2. Revoke the old token at the provider
3. Remove from git history: `git filter-repo --path <file> --invert-paths`
4. Force-push and contact GitHub Support to purge cached views
5. Close the auto-opened issue

## License

MIT
