# Mini Project 4 — Vuln Hub Penetration Test

Oumar Mballo — INF601 Advanced Programming in Python

Track: A

## What this is

An authorized penetration test of **Vuln Hub**, the sandboxed, intentionally
vulnerable Flask app provided as the Mini Project 4 Track A target. All 6
planted vulnerabilities were found and exploited; each `FLAG{...}` is backed
by a working proof-of-concept.

**Authorization / scope:** Only Vuln Hub, running locally on `127.0.0.1` with
`FLAG_SEED=o_mballo2`, was tested. No other host, service, or system was
touched. This is a closed, sandboxed, course-provided target — testing it is
explicitly authorized by the Mini Project 4 assignment.

## Repo contents

| Path | What it is |
|---|---|
| `app.py`, `templates/`, `static/`, `requirements.txt`, `Dockerfile`, `docker-compose.yml`, `check_flags.py`, `VULNHUB_APP_README.md` | The Vuln Hub target app, copied unmodified from the Mini Project 4 starter files, included so the engagement is reproducible. |
| `submission.txt` | All 6 captured flags, one per line. |
| `poc/` | One Markdown file per finding — the exact request/payload that produced each flag. |
| `pentest_report.md` | Full pentest report: scope, methodology, findings with severity, evidence, remediation. |

## How to reproduce

```bash
pip install -r requirements.txt
export FLAG_SEED=o_mballo2      # Windows PowerShell: $env:FLAG_SEED="o_mballo2"
python app.py                   # serves http://127.0.0.1:5000 (or set PORT=5050, etc.)
```

Then follow `pentest_report.md` / `poc/*.md` — every step is a literal `curl`
command that can be re-run against the freshly started app. Check flags with:

```bash
python check_flags.py submission.txt --username o_mballo2
```

## Agentic workflow

This project was done with Claude Code (via Claude's Cowork/Agent SDK
environment) driving the hands-on work: starting the target, running recon
and exploitation commands, verifying each flag, and writing the PoC/report
files. I directed the loop at each step (which challenge to go after, which
payload angle to try, when a result was "done") and reviewed every captured
flag and every claim in the report against the actual HTTP responses before
accepting it — nothing below was taken on faith from a draft.

Rough loop per finding: **goal** ("get the IDOR flag") → Claude ran the
recon/exploit commands (`curl` against the running app) → I reviewed the raw
HTTP response in each result → once a flag string appeared, it was verified
immediately with `check_flags.py` before being written into `submission.txt`
or any PoC file.

## AI Usage

- **Drafted by AI (Claude Code):** all `curl`/shell commands used for recon
  and exploitation; the wording of each `poc/*.md` write-up and of
  `pentest_report.md`; this README's structure.
- **Done/verified by me:** I decided which vulnerability to tackle next and
  in what order, confirmed every flag against `check_flags.py` before it was
  recorded anywhere, and reviewed each PoC file against the actual request/
  response output to confirm the write-up matches what really happened (no
  flag or claim here is AI-invented — each is a literal captured HTTP
  response).
- **Corrected/adjusted:** none of the technical content needed correcting —
  all 6 `check_flags.py` checks came back `VALID` on the first working
  payload per challenge. I did ask for the `.gitignore` to be adjusted so
  `submission.txt` (excluded in the starter's own `.gitignore`, since that
  file is meant to protect the starter repo from flag leaks) is tracked in
  *this* repo, since it's a required deliverable here.
