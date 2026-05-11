# Bug Bounty Methodology Skill

This is a comprehensive methodology for AI-assisted vulnerability research, combining non-code security auditing with APK-driven technical testing.

---

## When To Use

- User requests: VDP/Bug Bounty target selection, APK reverse engineering vulnerability discovery, report writing/polishing
- Only activate when an authorized testing scope is defined (HackerOne program, VDP policy, or explicit user confirmation)
- Do NOT use for unauthorized targets or destructive testing
- Applicable when user mentions: "bug bounty", "VDP", "vulnerability", "APK reverse", "OTP", "rate limit", "IDOR", "account takeover", "pentest", "scope"

---

## Input Schema

**Required:**
- `program_url`: string — HackerOne/Bugcrowd program URL or target description
- `scope_definition`: string or file path — Scope CSV, policy text, or asset list

**Optional:**
- `apk_path`: string — Path to APK file for reverse engineering
- `time_budget_minutes`: number (default: 120) — How long to spend on testing
- `test_intensity`: enum[low, medium, high] (default: low) — Controls request volume and concurrency
- `output_language`: enum[en, zh] (default: en) — Report output language
- `proxy`: string — Proxy address if geo-restriction applies (e.g., http://127.0.0.1:7897)
- `test_account_email`: string — Researcher's own test account for closed-loop verification

---

## Output Schema

Every execution MUST produce output in this order:

1. **Target Summary** — Organization, assets in scope, risk score, key signals
2. **Prioritized Attack Surfaces** — Ranked list with rationale for each priority
3. **Verification Log** — What was tested, what was not tested, what was eliminated and why
4. **Findings** — Only verified vulnerabilities (never speculative claims)
5. **Evidence** — Raw requests, responses, timing data, screenshots/logs
6. **Caveats & Legal Boundaries** — Test limitations, unverified assumptions, scope concerns
7. **Next Actions** — Recommended follow-up steps, ordered by expected value

---

## Hard Constraints

- **Only test within authorized scope** — Verify every target domain/endpoint against scope before testing
- **No DoS / no destructive actions** — No high-volume floods, no data deletion, no service disruption
- **No testing on third-party assets outside scope** — Even if discovered through in-scope assets
- **Stop and report if scope ambiguity is found** — Ask user before proceeding when boundaries are unclear
- **Only interact with accounts owned by the researcher** — Never test against other users' data
- **Respect VDP exclusion lists** — Check exclusions before every test category
- **Rate limit testing capped at test_intensity level:**
  - low: max 30 sequential requests, no concurrency
  - medium: max 500 sequential, max 5 concurrent sessions
  - high: max 1000 sequential, max 50 concurrent sessions (requires explicit user confirmation)

---

## Execution Contract

- If required input missing → ask only for the missing fields, do not guess
- If APK not provided → run non-code audit branch only (Part A)
- If APK provided → run full workflow (Part A + Part B)
- If evidence insufficient → mark finding as "unverified/suspected", never claim confirmed vulnerability
- If WAF/defense blocks testing → document the defense, analyze bypass feasibility, do NOT brute-force through it without user approval
- If test_intensity escalation needed → explain why and ask user permission before proceeding
- Always produce output in Output Schema order
- Always maintain append-only recon log throughout execution
- Always check legal boundaries before each new test category

---

## PART A: Non-Code Security Audit (Target Selection)

Before investing time in technical testing, spend 30 minutes assessing the target's overall risk posture using OSINT.

### Quick Screening (5 minutes)

1. HackerOne program page: When was the last report resolved? How many reports in 90 days?
2. Target domains: Accessible? TLS certificates valid?
3. GitHub repos (if any): Last commit? Archived?
4. News search: Recent layoffs, M&A, data breaches?

If any of these show severe anomalies, evaluate whether to continue before going deeper.

### Four-Dimension Risk Assessment

#### Dimension 1: HR & Culture (Weight: 15%)
- Employee review sentiment analysis (Fear, Anxiety, Low Joy are predictive)
- Core team stability and security team turnover
- Source: Glassdoor, LinkedIn

#### Dimension 2: Financial Pressure (Weight: 15%)
- Layoff ratio, M&A intensity, revenue/EBITDA decline
- Project/product active status
- Source: News, financial reports

#### Dimension 3: IT Operations / Network Hygiene (Weight: 40%)
- DNS hygiene: dangling records, open resolvers
- TLS certificates: expired, untrusted, version
- HTTP security headers: HSTS, CSP, X-Frame-Options, etc.
- Email security: SPF, DMARC, DKIM policies
- Known breach history and repeat attack vectors
- Zombie/abandoned assets
- Source: DNS queries, HTTP probes, crt.sh, AbuseIPDB

#### Dimension 4: Engineering Effectiveness (Weight: 30%)
- HackerOne response metrics: TTFR, TTT, TTR, resolution ratio
- Dependency health (if source available)
- CI/CD maturity (if source available)
- Source: HackerOne page, GitHub

### Risk Score Interpretation

| Score | Probability | Meaning |
|-------|------------|---------|
| 0-2 | <10% | Good security posture, low-hanging fruit already picked |
| 3-4 | 10-30% | Room for improvement but basically controlled |
| 5-6 | 30-60% | Systemic risk, worth deep testing |
| 7-8 | 60-85% | High probability of undiscovered serious vulnerabilities |
| 9-10 | >85% | Almost certain exploitable security flaws exist |

---

## PART B: APK-Driven Vulnerability Discovery Workflow

### Phase 1: Scope Investigation & Asset Enumeration

Input: HackerOne VDP page
Output: Priority-ranked asset list

1. Download scope CSV, classify all assets (URL/API/App/Wildcard)
2. Identify newly added assets (timestamp sort → new assets may have incomplete security audits)
3. Read exclusion rules, extract tech stack intelligence (exclusion lists reveal services used)
4. Quick HTTP probe each asset (status code, Server header, security headers)
5. Flag high-value signals: GraphQL endpoints, new assets, auth services, API gateways

### Phase 2: APK Reverse Engineering & System Model

Input: Google Play APK
Output: Complete system model document

1. Decompile APK (jadx)
2. Extract strings.xml → all hardcoded Keys/Tokens/URLs/IDs
3. Locate Retrofit Service interfaces → complete API endpoint list + auth levels
4. Identify authentication architecture:
   - Auth level enum (NONE / ANONYMOUS / PREFERRED / REQUIRED)
   - Session lifecycle (create → refresh → logout)
   - Token types and TTLs
5. Build data flow diagram:
   - Sensitive data models (UserAuth, CreditCard, Address class fields)
   - Data flow: App → API Gateway → Third-party services
   - Local storage: SharedPreferences, Room DB sensitive data
6. Build control flow diagram:
   - Auth flows (password / OTP / OAuth / SSO)
   - Payment flows (tokenization → checkout)
   - Password reset flow (trigger → verify → exchange → change)
7. Mark attack surfaces:
   - All NONE/ANONYMOUS auth level endpoints
   - All endpoints with user ID path parameters (IDOR candidates)
   - All Deep Link schemes (hijack candidates)
   - Third-party SDK integration points

### Phase 3: Attack Surface Prioritization

Input: System model + data/control flow diagrams
Output: Ranked attack direction list

Evaluation dimensions:
- Auth strength: NONE > ANONYMOUS > PREFERRED > REQUIRED
- Data sensitivity: Payment > PII > Orders > Public content
- Operation severity: Password change > Payment ops > Data read
- Defense layers: None > Single WAF > Multi-layer
- Input controllability: Path params > Body params > Headers

Priority target: Combinations of HIGH-sensitivity operations + LOW-auth entry points

### Phase 4: Direction-by-Direction Verification

Input: Ranked attack directions
Output: Verified vulnerabilities + eliminated directions

For each direction:
1. Construct minimal verification request (single curl)
2. Observe response: status code, error messages, response time
3. Determine if defenses exist (WAF/Rate Limit/Auth Check)
4. If defended → analyze defense mechanism, seek bypass
5. If undefended → expand test scale (multiple attempts, concurrent, cross-session)
6. Record results to append-only log

### Phase 5: Attack Chain Construction & Closed-Loop Verification

Input: Verified single-point vulnerabilities
Output: End-to-end PoC

1. Chain single-point vulnerabilities into complete attack chain
2. Identify blocking points in the chain (e.g., WAF interception)
3. For each blocking point, research bypass:
   - TLS fingerprint analysis (JA3/JA4)
   - Request identity spoofing (Web vs App)
   - Policy gap exploitation (endpoint × identity combinations not covered)
4. Execute complete chain against own test account
5. Verify final effect (password change → login confirm → data access)

### Phase 6: Quantitative Analysis & Reporting

Input: Closed-loop PoC + test data
Output: Submittable vulnerability report

1. Latency analysis: separate network latency vs server processing time
2. Throughput projection: single-thread → multi-thread → attacker perspective
3. Probability calculation: per-window hit rate × windows × targets
4. Scale attack economics: cost vs revenue
5. Threat model: minimum attacker capability assumptions
6. Honestly label caveats: test limitations, unverified assumptions
7. Write report + external expert review + corrections

---

## PART C: Key Principles

1. **System model first**: Don't test blindly. Understand the entire auth architecture and data flow before attacking.
2. **Focus on boundaries**: Vulnerabilities often appear at intersections of different auth levels, identity types, and defense policies.
3. **Combination attacks**: A single weakness may not be a vulnerability. Multiple weaknesses stacked together form an attack chain.
4. **Defense ≠ Security**: WAF existence doesn't mean effective protection. Always verify defenses actually block the attack, not just exist.
5. **Math speaks**: Quantitative analysis is more persuasive than "could potentially be exploited."
6. **Append-only logging**: Every finding and exploration result gets logged with timestamps. Never modify past entries.
7. **Legal awareness**: Pause and assess legal boundaries whenever findings exceed expectations. Check VDP rules before escalating test intensity.
8. **External review**: Have another agent/expert critically review reports before submission to catch technical errors and severity inflation.

---

## PART D: Tool Preferences

| Task | Tool | Reason |
|------|------|--------|
| APK decompilation | jadx | Best Kotlin/Java decompilation quality |
| API testing (PX-protected) | Windows curl.exe | Schannel TLS bypasses PX fingerprinting |
| API testing (no PX) | Python requests or PowerShell | Faster scripting |
| Concurrent testing | Python threading + curl subprocess | Linear throughput scaling |
| Rate limit testing | Custom Python script with curl backend | Precise timing measurement |
| Latency analysis | curl -w with timing variables | Separates DNS/TCP/TLS/server processing |
| Recon logging | Append-only markdown | Preserves timeline integrity |

---

## PART E: Report Quality Checklist

Before submitting any report, verify:

- [ ] Complete reproduction steps with copy-pasteable commands
- [ ] Closed-loop verification (not just "this might work")
- [ ] Honest caveats section (what was NOT tested)
- [ ] Quantitative feasibility analysis (not just "theoretically possible")
- [ ] Threat model with minimum attacker capability
- [ ] CVSS score with per-metric justification
- [ ] Remediation recommendations (specific, actionable)
- [ ] No severity inflation (defer to security team when uncertain)
- [ ] UTF-8 encoding, no garbled characters
- [ ] PoC scripts attached and tested
