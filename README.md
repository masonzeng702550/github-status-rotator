# github-status-rotator

Rotate your GitHub profile status on a schedule, for free, with zero infrastructure — just GitHub Actions.

Picks a random `(emoji, message)` from a pool you control and writes it to your profile via the `changeUserStatus` GraphQL mutation. Default cadence is hourly; each status auto-expires 65 minutes later so a missed run never leaves stale text behind.

## Why

Your "What's happening" status is the most underused real-estate on your profile. Setting it once and forgetting it makes it lie; setting it manually is friction. A small rotator splits the difference: it always says *something*, the something stays fresh, and you only edit a JSON file when you want a new line in circulation.

## How it works

1. `.github/workflows/status.yml` runs on a cron schedule (default `0 * * * *`).
2. `scripts/set_status.py` loads `scripts/pool.json`, picks one entry at random, and calls the GitHub GraphQL `changeUserStatus` mutation with your PAT.
3. Each status is set with `expiresAt = now + 65min` so a failed/skipped run self-heals.

GitHub Actions cron is best-effort and can be delayed by 5–15 minutes under load — that's fine for this use case.

## Setup

### 1. Use this template

Click **Use this template** (top of the repo on GitHub) → create a new repo under your account. A fork also works, but a template keeps your history clean.

### 2. Create a PAT with the `user` scope

Open https://github.com/settings/tokens/new and create a classic PAT:

- **Note:** `status-rotator`
- **Expiration:** whatever you're comfortable with
- **Scopes:** check **only** `user` (least privilege — this is all the mutation needs)

Copy the token.

### 3. Store it as a repo secret

In your new repo: **Settings → Secrets and variables → Actions → New repository secret**

- **Name:** `STATUS_TOKEN`
- **Value:** the PAT you just created

Or via `gh`:

```sh
gh secret set STATUS_TOKEN -R <you>/<your-repo>
# paste the PAT when prompted
```

### 4. Trigger a test run

```sh
gh workflow run rotate-status -R <you>/<your-repo>
```

Then check your GitHub profile — the status should change within a few seconds. After this, the hourly cron takes over.

## Customizing

### The status pool

Edit [`scripts/pool.json`](scripts/pool.json). Each entry is:

```json
{ "emoji": ":coffee:", "message": "caffeinating" }
```

- `emoji` uses GitHub shortcode syntax (`:rocket:`, `:fire:`, etc.) — the same syntax issues and PRs accept.
- `message` is plain text, max 80 characters.

The pool ships ~38 generic developer statuses. Replace them with whatever fits you — interests, in-jokes, current projects.

### Cadence

Edit the `cron:` line in `.github/workflows/status.yml`. Examples:

| Cron                 | Frequency       | Approx. Actions minutes / month |
| -------------------- | --------------- | ------------------------------- |
| `0 * * * *`          | hourly          | ~12                             |
| `*/30 * * * *`       | every 30 min    | ~24                             |
| `*/15 * * * *`       | every 15 min    | ~96                             |
| `*/5 * * * *`        | every 5 min     | ~290                            |

Free accounts get 2,000 Actions minutes/month on private repos and unlimited on public repos, so any of these are comfortably within budget.

If you change the cadence, also bump `STATUS_EXPIRY_MIN` in the workflow env so the safety-net expiry stays slightly longer than the interval.

### Busy flag

Set `STATUS_BUSY: "true"` in the workflow to mark yourself as having limited availability — GitHub will then warn people who @ or assign you. (It doesn't block notifications, just nudges.)

### Custom pool location

`STATUS_POOL_FILE` env var lets you point at a different JSON file if you want to keep your real pool out of a public repo. Stage a private pool as a workflow artifact or commit-on-branch.

## Local use

Want to test before pushing?

```sh
export STATUS_TOKEN=ghp_xxx   # PAT with `user` scope
python3 scripts/set_status.py
```

## License

MIT — see [LICENSE](LICENSE).
