---
name: openclaw-skill-analyzer
description: Advanced security audit tool for OpenClaw skills. Performs comprehensive static analysis to detect prompt injection, data exfiltration, privilege escalation, credential theft, sandbox escape, obfuscation, supply chain risks, and more. Use this skill to vet any third-party OpenClaw skill before installing it, or to audit your own skills for security best practices. Triggers on any request to analyze, scan, audit, review, or check the security of an OpenClaw skill.
---

# OpenClaw Skill Analyzer

A comprehensive security audit tool that performs advanced static analysis on OpenClaw skills to detect vulnerabilities and malicious patterns.

## What It Detects

The analyzer runs **40+ detection rules** across **12 security categories**:

| Category | What It Catches |
|---|---|
| **Prompt Injection** | exec/eval with user input, shell injection, unsanitized templates, dynamic code generation |
| **Data Exfiltration** | HTTP requests to external endpoints, DNS tunneling, encoded data transmission |
| **Privilege Escalation** | sudo/root usage, SUID/SGID manipulation, container escape attempts |
| **Hidden File Access** | Dotfile access (.ssh, .aws, .env), SSH key reading, home directory traversal |
| **Dangerous Execution** | os.system, subprocess, exec/eval, shell=True, unquoted bash variables |
| **Code Obfuscation** | Base64 payloads, hex-encoded strings, dynamic attribute access, reversed commands |
| **Network Access** | Raw socket creation, hardcoded IPs/URLs |
| **Filesystem Abuse** | Path traversal (../), access to /etc/passwd, recursive deletes, temp file leaks |
| **Credential Theft** | Environment variable harvesting (API_KEY, TOKEN), credential file access, hardcoded secrets |
| **Supply Chain** | Runtime package installs, remote script execution (curl | bash) |
| **Sandbox Escape** | Docker socket access, container breakout patterns, kernel module manipulation |
| **Information Disclosure** | System enumeration, debug output leaking secrets |

## Usage

### Basic Scan

```bash
python {baseDir}/scripts/analyze_skill.py --skill_path /path/to/openclaw/skill
```

### Save Report as JSON

```bash
python {baseDir}/scripts/analyze_skill.py --skill_path /path/to/skill --output report.json
```

### JSON Output to Console

```bash
python {baseDir}/scripts/analyze_skill.py --skill_path /path/to/skill --format json
```

### Quiet Mode (JSON file only, no console output)

```bash
python {baseDir}/scripts/analyze_skill.py --skill_path /path/to/skill --output report.json --quiet
```

## Output Format

### Console Report

The console output includes:
- **Scan summary** — files analyzed, lines scanned, file types found
- **Risk assessment** — overall risk level (SAFE / LOW / MEDIUM / HIGH / CRITICAL) with a numeric risk score
- **Finding counts** — breakdown by severity (Critical, High, Medium, Low, Info)
- **Category breakdown** — how many findings per security category
- **Detailed findings** — each finding with severity, rule ID, file, line number, matched text, and a remediation recommendation

### JSON Report

The JSON report contains structured data for programmatic use:

```json
{
  "scan_info": { "target": "...", "timestamp": "...", "files_analyzed": 5, "lines_analyzed": 200 },
  "risk_assessment": { "overall_risk": "HIGH", "risk_score": 28, "risk_percentage": 70, "finding_counts": { ... } },
  "category_breakdown": { "Prompt Injection": { "CRITICAL": 1 }, ... },
  "findings": [ { "severity": "CRITICAL", "category": "...", "title": "...", "file": "...", "line": 12, ... } ]
}
```

### Exit Codes

| Code | Meaning |
|---|---|
| `0` | SAFE or LOW risk — no critical issues found |
| `1` | MEDIUM or HIGH risk — review recommended |
| `2` | CRITICAL risk — do not install without remediation |

## Risk Scoring

Each finding has a severity with a point value:
- **CRITICAL** = 10 points
- **HIGH** = 7 points
- **MEDIUM** = 4 points
- **LOW** = 2 points
- **INFO** = 0 points

The overall risk level is determined by the highest severity findings present:
- Any CRITICAL finding → overall CRITICAL
- 3+ HIGH findings → overall HIGH
- Any HIGH finding → overall MEDIUM
- Only MEDIUM/LOW/INFO → overall LOW
- No findings → SAFE

## Best Practices for Skill Development

- **Principle of Least Privilege**: Only request the minimum necessary permissions and tools.
- **Input Validation**: Always sanitize user inputs before using them in commands or file paths.
- **No Dynamic Execution**: Avoid exec, eval, os.system — use safer alternatives.
- **No Hardcoded Secrets**: Use environment variables or secret managers, never embed credentials.
- **Sandbox Awareness**: Never attempt to access files or paths outside your skill's workspace.
- **Transparency**: All code should be readable — no obfuscation, encoding, or hidden payloads.
- **Minimal Network Access**: Only make network requests that are essential and documented.

## References

- [OpenClaw Security Documentation](https://docs.openclaw.ai/gateway/security)
- [OpenClaw Creating Skills Documentation](https://docs.openclaw.ai/tools/creating-skills)
