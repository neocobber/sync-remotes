# TelDrive Sync

Scheduled and manually triggered one-way synchronization from TelDrive to another rclone remote. The workflow starts `ghcr.io/tgdrive/teldrive:v2` with River workers and Telegram rate limiting disabled, configures up to 20 download bots, and uses the TelDrive-enabled rclone build from [`rclone-v1.75.1`](https://github.com/divyam234/nix-pkgs/releases/tag/rclone-v1.75.1).

## Warning

The workflow runs `rclone sync`. Files in the destination that are absent from the selected TelDrive path are deleted. Use a dedicated destination path and test with non-critical data first.

## Required secrets

Configure these GitHub Actions repository secrets:

| Secret | Purpose |
| --- | --- |
| `POSTGRES_USER` | Username for the TelDrive PostgreSQL database |
| `POSTGRES_PASSWORD` | Password for the TelDrive PostgreSQL database |
| `TELDRIVE_SECURITY_SIGNING_KEY` | Signing key matching the existing TelDrive deployment |
| `TELDRIVE_SECURITY_DATA_KEY` | Data key matching the existing TelDrive deployment |
| `TELDRIVE_API_KEY` | TelDrive API token used by rclone |
| `RCLONE_CONFIG` | Complete rclone configuration containing a remote named `destination` |
| `TS_OAUTH_CLIENT_ID` | Tailscale OAuth client ID with the `auth_keys` scope |
| `TS_OAUTH_CLIENT_SECRET` | Tailscale OAuth client secret |

The workflow joins the tailnet with `tag:nixos,tag:github-deploy`, waits for `netcup.tail69fe7a.ts.net`, and connects to PostgreSQL at `netcup.tail69fe7a.ts.net:6432/postgres`. The Tailscale OAuth client must be permitted to issue both tags. Prefer a restricted PostgreSQL user.

Example `RCLONE_CONFIG` secret for an S3 destination:

```ini
[destination]
type = s3
provider = Other
access_key_id = example
secret_access_key = example
endpoint = https://s3.example.com
```

Do not add the TelDrive source to this file. The workflow configures the `teldrive:` remote through masked environment variables.

## Operation

- Scheduled sync: every three hours on the hour UTC, leaving a one-hour gap after the two-hour run limit.
- Each workflow run is limited to two hours.
- Manual sync: **Actions > Sync TelDrive to destination > Run workflow**.
- Manual runs can select paths within `teldrive:` and `destination:`.
- Scheduled runs synchronize both remote roots.
- GitHub concurrency prevents overlapping syncs.

## Local validation

```bash
actionlint .github/workflows/sync.yml
```
