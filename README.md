# infrlo-watchdog

GitHub Actions watchdog for the Infrlo app `lytest` behind the fixed Cloudflare
tunnel `infrloly.ttxyc12345.kdns.fr`.

The app itself lives in `ttxyc/Singbox-nodejs` (branch `infrlo`). This workflow
lives in a standalone repository because GitHub scheduled workflows are not
reliable in forks.

## What it does

Every 5 minutes:

1. Request `https://infrloly.ttxyc12345.kdns.fr/`; HTTP 200 means healthy.
2. If the check fails, wait 30 seconds and check again.
3. If it fails twice, call the Infrlo dashboard API to redeploy the app.
4. Skip if a deployment is already in progress or the latest deployment is
   less than 10 minutes old, to avoid a redeploy loop.
5. After triggering a redeploy, poll the tunnel for up to 4 minutes and fail
   the job if it does not recover.

## Required secret

`INFRLO_COOKIE` - a Cookie header copied from a logged-in
`https://dash.infrlo.com` session. It includes `infrlo_device`, `csrf_token`
and `infrlo_session`.

The current session expires around **2026-11-28**. When it expires, log in to
dash.infrlo.com again, extract the new cookie, and update this secret. Do not
log out of dash.infrlo.com while the watchdog is active.

## Manual test

Actions -> **Infrlo tunnel watchdog** -> **Run workflow**.

Tick `force_redeploy` to skip the health check and exercise the redeploy path.

## Security notes

- The cookie is stored only as an encrypted GitHub Actions secret and is masked
  in logs.
- Only trusted collaborators should have write access to this repository.
- The workflow never prints the cookie or the CSRF token.