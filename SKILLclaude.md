# Bug Bounty Security Research Assistant

## Project Overview

This workspace contains vulnerability research materials for HackerOne VDP/BBP programs. The AI assistant follows a structured methodology combining non-code security auditing with APK-driven technical testing.

## Core Methodology

See `methodology/skill-bbp.md` for the complete workflow specification including:
- Input/Output schemas
- Hard constraints and execution contract
- Six-phase testing workflow
- Tool preferences and report quality checklist

## Commands

- `start audit` — Begin non-code security audit (Part A of skill-bbp.md)
- `start apk analysis` — Begin APK reverse engineering workflow (Part B, Phase 2)
- `prioritize attacks` — Run attack surface prioritization (Part B, Phase 3)
- `verify direction [name]` — Test a specific attack direction (Part B, Phase 4)
- `build chain` — Construct and verify full attack chain (Part B, Phase 5)
- `write report` — Generate submission-ready report (Part B, Phase 6)

## Conventions

- All recon logs are append-only markdown files
- Findings are numbered: FINDING-001, FINDING-002, etc.
- Reports follow HackerOne format: Summary, Steps to Reproduce, Impact, Supporting Material
- Test intensity defaults to LOW unless explicitly escalated by user
- All testing limited to researcher-owned accounts and authorized scope

## Key Rules

1. Always verify target is in scope before testing
2. Never exceed test_intensity without user confirmation
3. Mark unverified findings as "suspected" not "confirmed"
4. Stop and ask if scope boundaries are ambiguous
5. Produce output in the fixed 7-step Output Schema order
6. Use curl.exe (Windows Schannel) for PX-protected endpoints
7. Use Python/PowerShell for non-protected endpoints
8. Maintain append-only logs — never modify past entries

## File Structure

```
methodology/          — Workflow docs and skill definitions
wonder/               — Wonder Group VDP workspace
  reports/            — Vulnerability reports
  scripts/            — PoC scripts
  scope/              — Scope definitions
lovable/              — Lovable VDP workspace
  reports/
```

## Legal Boundaries

- Only test within HackerOne authorized scope
- No DoS, no destructive actions
- No testing third-party assets outside scope
- Respect all VDP exclusion rules
- Cap testing at declared intensity level
