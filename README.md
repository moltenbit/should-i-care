# should-i-care

[![skills.sh](https://skills.sh/b/moltenbit/should-i-care)](https://skills.sh/moltenbit/should-i-care)

Single-CVE applicability triage for security researchers and defenders. Ask "am I affected by CVE-X?" and get a reasoned verdict against your environment, with every condition cited to its source, every assumption named, and a full audit of what was consulted.

## What the output looks like

A real assessment, against an environment profile holding one Debian 13 server running nginx:

> **CVE-2026-9256: not affected (conditional) · nginx OSS 1.26.3 (Debian 13)**
> 
> The overflow is reachable only through rewrite directives whose regex uses overlapping PCRE captures and whose replacement references multiple captures in a redirect/args context. With no such rules in use, the vulnerable path is not exercised. This is a config-level reprieve, not a patched system; every other gate is open.
>
> | Condition | state | Sources |
> |---|---|---|
> | version in range 0.1.17-1.31.0 | 1.26.3: in range, not exonerating | [CVE.org](https://www.cve.org/CVERecord?id=CVE-2026-9256) · [NVD](https://nvd.nist.gov/vuln/detail/CVE-2026-9256) |
> | Debian backport fixed | no, trixie still vulnerable | [Debian tracker](https://security-tracker.debian.org/tracker/CVE-2026-9256) |
> | multi-capture rewrite rule (redirect/args) | not present | [CVE.org](https://www.cve.org/CVERecord?id=CVE-2026-9256) |
> | attacker prerequisites | unauth, network-reachable, no further gate | [CVE.org](https://www.cve.org/CVERecord?id=CVE-2026-9256) |
> 
> *Rests on the confirmed absence of multi-capture rewrite rules (volatile: flips to affected if such a rule is added). CVE published 2026-05-22; package still unpatched in trixie, so patch when Debian ships the fix.*
> 
> Consulted: [CVE.org record](https://www.cve.org/CVERecord?id=CVE-2026-9256) · [Debian security tracker](https://security-tracker.debian.org/tracker/CVE-2026-9256) · [NVD](https://nvd.nist.gov/vuln/detail/CVE-2026-9256) · [openwall oss-security](https://www.openwall.com/lists/oss-security/)

## What this is not

- **Not a scanner replacement.** It replaces the manual step *after* a scanner hit: deciding whether a CPE match actually applies given your specific environment.
- **Not a decision engine.** It exposes the condition chain so you can verify. The reasoning is the product, not the verdict.
- **Not a free pass.** A "not affected" is a reasoned assessment with cited conditions, not a clean bill of health. The verdict is held to a higher evidentiary bar precisely so it does not become one.
- **Not a compliance record.** It is designed to produce an auditable triage note, not an authoritative compliance record.

## Requirements

Built and tested for **Claude Code**, which is the recommended and verified environment. It is also a standard agent skill following the open agent-skills standard (SKILL.md plus references, no Claude-exclusive mechanics), so it installs on other compatible coding agents (e.g. Codex, Cursor, Gemini CLI, and others that support the skills standard) via the skills CLI. Those other agents are untested: behavior there depends on the agent and the model, use at your own discretion, feedback welcome.

The actual requirements are generic, not Claude-specific:

- **An agent with a local filesystem.** The environment file at `~/.config/should-i-care/environment.md` has to persist between runs for the write-back to work and the profile to accumulate across sessions. Any terminal or IDE coding agent provides this; a pure browser chat does not.
- **Runtime web access.** The skill reads the canonical CVE record and vendor / research sources at the time of the assessment.
- **No API keys.** Fully keyless.

## Install

Install with the skills CLI (recommended):

```
npx skills add moltenbit/should-i-care -g
```

The CLI auto-detects your coding agent and installs into that agent's skills directory. Drop the `-g` flag to install into the current project instead of globally. Restart the agent so the skill is picked up.

Update later with:

```
npx skills update should-i-care
```

As a manual fallback, clone the repository into the Claude Code skills directory:

```
git clone https://github.com/moltenbit/should-i-care ~/.claude/skills/should-i-care
git clone https://github.com/moltenbit/should-i-care .claude/skills/should-i-care
```

The first path is for personal / global use, the second for project scope. Restart Claude Code after a manual clone. This manual path is Claude-Code-specific; on other agents use the skills CLI above, which knows your agent's skills directory.

## Quick start

1. Ask:

   > Am I affected by CVE-2026-XXXX?

2. On the first CVE the skill asks the minimum facts it needs, confirms them with you, and creates the environment file at `~/.config/should-i-care/environment.md` from your answers, printing the exact path on first creation. This location is fixed: it persists across skill updates and does not depend on which directory you launch Claude Code from. From then on it maintains the file itself: when it learns a relevant fact, it asks you, you confirm, and it writes the entry back. You should rarely if ever hand-edit the file. The profile is not a source of truth, it is a store of dated, sourced assumptions: each fact carries `last_verified`, `source`, and a volatility tag, and a not-affected verdict that rests on a stale or unconfirmed fact is surfaced through the assumption-disclosure step rather than relied on silently.

Because that path sits outside any repository, the environment file is not normally anywhere git would see it. The `.gitignore` shipped with this skill is kept only as a safety net for the unusual case where someone places an environment file inside a repo: it keeps `environment.md` and its variants from being committed.

`environment.example.md` is an annotated read-only reference: it shows the format the skill writes: Layer 1 (inventory), Layer 2 (conditions), the completeness flag, the volatility tag, so you can read `environment.md` and understand what is in it. Do not copy it; the skill creates the real file for you.

## The three verdicts

- **affected**: the CVE applies; conditions are met or no exonerating condition exists.
- **not affected**: a named exonerating condition excludes you. Held to a higher evidentiary bar than the other two; on thin sourcing it degrades to needs verification.
- **needs verification**: no clean basis to call it either way. The default in doubt.

The skill separates **identity sources** (the canonical CVE record: what the CVE *is*), **condition sources** (vendor advisories AND established research, co-equal: *under what condition* it applies), and **urgency sources** (KEV, EPSS: how *urgent*, never *whether*). Each condition is cited inline; the full reading audit, including rejected sources, lives in a footer line. Depth on the verdict logic, the two-anchor assignment test, the freshness modulator and the condition archetypes lives in the `references/` directory.

## Security boundary

- **Read text only.** Analyses, writeups, advisories, the CVE record.
- **Never run artifacts.** No PoC repos, Nuclei templates, or exploit binaries are downloaded or executed.
- **Never contact named infrastructure.** No callbacks, no test requests, no verification URLs from a writeup. Communication is restricted to authoritative data sources (CVE.org, vendor pages, KEV, EPSS, GHSA, OSV).

## Batch input

Multiple CVEs in one call are processed strictly sequentially, one fully complete before the next. Soft limit ~5; if many survive the inventory gate, the skill will recommend individual calls. The gold standard is **one per call when accuracy matters**.

## Author

`moltenbit`

- Website: <https://moltenbit.net>
- GitHub: <https://github.com/moltenbit>
- Mastodon: <https://infosec.exchange/@moltenbit>
- Bluesky: <https://bsky.app/profile/moltenbit.bsky.social>

## License

MIT. See `LICENSE`.
