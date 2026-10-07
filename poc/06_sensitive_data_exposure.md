# Finding 6: Sensitive Data Exposure (backup file world-readable)

**Flag:** `FLAG{exposure_616c8bdf1d10}`

## Steps

1. Checked `/robots.txt`:
   ```
   curl http://127.0.0.1:5050/robots.txt
   ```
   Response:
   ```
   User-agent: *
   Disallow: /backups/
   ```
   `robots.txt` only works by voluntary compliance — it isn't an access control,
   and disallowing a path there is a strong hint that something sensitive lives
   under it and is reachable by anyone who simply requests it.

2. Guessed a plausible filename for a database backup and requested it directly
   (no authentication, no session needed):
   ```
   curl http://127.0.0.1:5050/backups/vulnhub.sql.bak
   ```

3. Server returned the raw `.sql.bak` file (HTTP 200) containing a SQL `INSERT`
   statement with the flag in a `secrets` table:
   ```
   -- database backup (should NOT be world-readable)
   -- left in /backups/ by mistake
   INSERT INTO secrets VALUES ('FLAG{exposure_616c8bdf1d10}');
   ```

## Root cause
A database backup file was placed inside a directory served statically by the
web app, with no authentication check on that path. `robots.txt` additionally
advertised the directory's existence to anyone who reads it.

## Remediation
- Never store backups, source, or config files inside a web-served directory.
- Store backups outside the web root, or behind authentication.
- Don't rely on `robots.txt` for access control — it is advisory only, visible
  to anyone, and ignored by non-compliant clients.
- Add automated scanning (e.g. in CI) for common sensitive file patterns
  (`*.bak`, `*.sql`, `.env`, `.git/`) inside deployed static directories.
