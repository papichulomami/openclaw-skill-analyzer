# OpenClaw Skill Analyzer

A security audit tool that scans OpenClaw skills for malicious patterns before you install them.

## Why this exists

ClawHub has thousands of skills. Most are fine. Some aren't. In February 2026, security researchers found that roughly 12% of audited skills on ClawHub contained malicious code — things like credential theft, data exfiltration, and hidden backdoors disguised as useful tools.

I got tired of manually reading through skill source code every time I wanted to try something new, so I built this. It's a static analyzer that checks for 40+ known bad patterns across 12 security categories and gives you a clear pass/fail before you install anything.

## What it catches

- **Prompt injection** — exec/eval with user input, shell injection, template manipulation
- **Data exfiltration** — HTTP calls to external endpoints, DNS tunneling, encoded transmissions
- **Privilege escalation** — sudo/root usage, SUID bit manipulation, container escapes
- **Hidden file access** — reading .ssh, .aws, .env, SSH keys, home directory snooping
- **Dangerous execution** — os.system, subprocess with shell=True, unquoted bash variables
- **Code obfuscation** — base64 payloads, hex-encoded strings, reversed command strings
- **Credential theft** — environment variable harvesting, reading credential files, hardcoded secrets
- **Supply chain risks** — runtime package installs, curl-pipe-bash patterns
- **Sandbox escape** — Docker socket access, container breakout patterns, kernel module loading
- **Filesystem abuse** — path traversal, access to /etc/passwd, recursive deletes
- **Network access** — raw sockets, hardcoded IPs and URLs
- **Information disclosure** — system enumeration, debug output leaking secrets

## Setup

No dependencies. Just Python 3. Clone the repo and run it.

```bash
git clone https://github.com/papichulomami/openclaw-skill-analyzer.git
cd openclaw-skill-analyzer
```

## Usage

Scan a skill before installing it:

```bash
python scripts/analyze_skill.py --skill_path /path/to/skill
```

Save a JSON report:

```bash
python scripts/analyze_skill.py --skill_path /path/to/skill --output report.json
```

JSON output to console:

```bash
python scripts/analyze_skill.py --skill_path /path/to/skill --format json
```

## Reading the results

The tool gives you a risk level based on what it finds:

| Risk | What it means |
|---|---|
| SAFE | Nothing found. Good to go. |
| LOW | Minor stuff. Probably fine, but read the findings. |
| MEDIUM | Some concerns. Review before installing. |
| HIGH | Serious red flags. Don't install without understanding every finding. |
| CRITICAL | Walk away. Or at least read every line of code yourself first. |

Exit codes for CI/CD or scripting:
- `0` = SAFE or LOW
- `1` = MEDIUM or HIGH
- `2` = CRITICAL

## Scoring

Each finding has a severity level with a point value:

- CRITICAL = 10 points
- HIGH = 7 points
- MEDIUM = 4 points
- LOW = 2 points
- INFO = 0 points

One CRITICAL finding = overall CRITICAL. Three or more HIGH findings = overall HIGH. The tool is intentionally strict — better to flag something harmless than miss something dangerous.

## Limitations

This is static analysis. It reads code and matches patterns. It doesn't run anything, doesn't sandbox anything, and can't catch everything. A sufficiently clever attacker could evade regex-based detection. This tool is meant to catch the obvious and the common, not to replace a full manual code review on skills you plan to use in production.

It also flags its own detection patterns if you scan it against itself (it sees references to `exec`, `eval`, `.env` in its own regex rules). That's expected behavior, not a bug.

## License

MIT — use it however you want.
