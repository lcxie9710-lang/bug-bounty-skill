# Bug Bounty Methodology — AI Agent Skill

## Acknowledgement

Special thanks to **Liyi Zhou** from the University of Sydney, whose course **INFO5995** greatly inspired the development of this skill. The structured approach to vulnerability research, non-code security auditing, and responsible disclosure practices presented in this methodology are largely derived from the frameworks and thinking introduced in that course.

---

## What Is This

A structured methodology skill for AI coding agents (Claude Code, OpenAI Codex, GitHub Copilot, Cursor, Kiro, etc.) that guides the agent through the complete bug bounty / VDP vulnerability research lifecycle:

1. **Target Selection** — OSINT-based non-code security audit to assess risk posture before investing time
2. **APK Reverse Engineering** — Decompile mobile apps to build system models, map API endpoints, and identify attack surfaces
3. **Attack Surface Prioritization** — Rank directions by exploitability using data flow and control flow analysis
4. **Verification & PoC** — Test each direction, build attack chains, bypass defenses
5. **Quantitative Analysis** — Latency measurement, throughput projection, probability calculation
6. **Reporting** — HackerOne-format reports with closed-loop evidence and honest caveats

The skill includes input/output schemas, hard constraints, execution contracts, and a report quality checklist — making it a drop-in configuration for any AI agent working on authorized security research.

---

## Files

| File | Purpose |
|------|---------|
| `skill-bbp.md` | Core methodology (the "brain") — complete workflow, schemas, constraints |
| `CLAUDE.md` | Entry shell for Claude Code — place in project root |
| `AGENTS.md` | Entry shell for Codex/Copilot/Cursor — place in project root |
| `README.md` | This file |

---

## Quick Start

### For Claude Code

```bash
cp methodology/CLAUDE.md ./CLAUDE.md
```

Claude Code will automatically read `CLAUDE.md` at session start and follow the methodology.

### For OpenAI Codex / GitHub Copilot / Cursor / Windsurf

```bash
cp methodology/AGENTS.md ./AGENTS.md
```

These tools read `AGENTS.md` from the project root automatically.

### For Kiro

Copy the skill to the steering directory:

```bash
mkdir -p .kiro/steering
cp methodology/skill-bbp.md .kiro/steering/skill-bbp.md
```

Add front-matter to enable auto-inclusion:

```yaml
---
inclusion: auto
---
```

### For any other agent

Simply include the content of `skill-bbp.md` in your system prompt or project context.

---

## Usage

Once installed, start a conversation with your AI agent and provide:

**Required inputs:**
- `program_url` — The HackerOne/Bugcrowd program URL
- `scope_definition` — Path to scope CSV or paste the scope text

**Optional inputs:**
- `apk_path` — Path to APK for reverse engineering
- `proxy` — Proxy address for geo-restricted targets
- `test_intensity` — low (default) / medium / high
- `output_language` — en (default) / zh

The agent will follow the 6-phase workflow and produce output in the fixed 7-step schema:

1. Target Summary
2. Prioritized Attack Surfaces
3. Verification Log
4. Findings (verified only)
5. Evidence
6. Caveats & Legal Boundaries
7. Next Actions

---

## Safety & Legal

This skill enforces hard constraints:

- Only test within authorized scope
- No DoS or destructive actions
- No testing third-party assets outside scope
- Stop and ask if scope is ambiguous
- Default to LOW test intensity
- Never claim unverified findings as confirmed

The methodology is designed for **authorized security research only** (HackerOne VDP/BBP programs with explicit Safe Harbor policies).

---

## License

This methodology is shared for educational purposes. Use responsibly and only within authorized testing programs.

