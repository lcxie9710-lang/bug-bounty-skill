# AGENTS.md — Bug Bounty Security Research

## Overview

This repository contains vulnerability research materials for authorized HackerOne VDP/BBP programs. AI agents working in this repo should follow the structured methodology defined in `methodology/skill-bbp.md`.

## Setup

- OS: Windows
- Shell: PowerShell
- Proxy: Clash on port 7897 (US exit node, required for geo-restricted APIs)
- APK decompiler: jadx 1.5.5 (if APK analysis needed)
- Python: 3.12+ with `requests` library

## Build / Test Commands

```bash
# Run OTP rate limit test
python wonder/scripts/otp-rate-limit-test.py --email <test_email> --proxy http://127.0.0.1:7897

# Run concurrent rate limit test
python wonder/scripts/otp-concurrent-test.py

# Run full attack chain PoC (interactive — requires manual OTP input)
python wonder/scripts/otp-full-chain.py --proxy http://127.0.0.1:7897

# Verify script syntax
python -c "import ast; ast.parse(open('<script_path>', encoding='utf-8').read())"
```

## Coding Conventions

- PoC scripts use `curl.exe` subprocess calls (Windows Schannel TLS) to bypass WAF fingerprinting
- JSON bodies passed via temp files to avoid PowerShell escaping issues
- All scripts support `--proxy` flag for geo-restricted targets
- Scripts pause for manual input at security-sensitive steps (OTP entry)
- UTF-8 encoding for all files

## Testing Conventions

- Append-only recon logs in markdown format
- Findings numbered sequentially: FINDING-001, FINDING-002, etc.
- Reports in HackerOne submission format
- Test intensity levels: low (30 req), medium (500 req), high (1000 req)
- Default to LOW — escalate only with explicit confirmation

## Security Rules

- ONLY test within authorized HackerOne scope
- ONLY interact with researcher-owned test accounts
- NO DoS, NO destructive actions, NO data exfiltration
- NO testing third-party assets discovered through in-scope targets
- STOP and ASK if scope boundaries are unclear
- RESPECT all VDP exclusion lists (check before each test category)
- CAP request volume at declared test_intensity level

## Context Files

| File | Purpose |
|------|---------|
| `methodology/skill-bbp.md` | Complete methodology with input/output schemas |
| `methodology/01-漏洞挖掘工作思路总结.md` | Lessons learned from prior projects |
| `wonder/wonder-recon-log.md` | Wonder Group recon log (append-only) |
| `wonder/grubhub-system-model.md` | Grubhub APK system model |
| `wonder/grubhub-attack-plan.md` | Attack surface prioritization |
| `wonder/reports/REPORT-002-hackerone-submission.md` | OTP vulnerability report |
