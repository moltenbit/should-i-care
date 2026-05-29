# Trusted Sources

This is a non-exhaustive heuristic for what "established" means in condition research. It is NOT a whitelist and it excludes nothing. An unknown source is not rejected: it is carried transparently as unconfirmed until a second corroborating anchor appears. The two-anchor assignment test from `condition-extraction.md` applies to every source, established or not.

Group by what the source is consulted FOR, not by an authority ranking. Different roles, different uses.

## Condition deep-analysis

Reverse engineering, exact preconditions, technical depth on what makes the bug actually fire. For freshly dropped CVEs these sources are often AHEAD of the vendor and more precise about the discriminating condition. Co-equal with vendor / PSIRT advisories. Do not rank below the vendor by default.

- watchTowr Labs
- Assetnote (Searchlight Cyber)
- Horizon3.ai
- Google Project Zero
- Trend Micro Zero Day Initiative (ZDI)
- Rapid7

## Active-exploitation telemetry

Is it exploited in the wild, under what circumstances. Sharpens freshness and urgency. These sources NEVER carry an applicability verdict: observing exploitation does not tell you whether YOUR environment meets the precondition. They feed the urgency tag and freshness language, nothing more.

- GreyNoise
- Google Threat Intelligence Group / Mandiant
- SANS Internet Storm Center

## Incident-response origin

CVEs first discovered from real incidents, where the actual exploitation condition is often documented precisely because it was observed in the wild on a real victim.

- Orange Cyberdefense CSIRT
- Mandiant

## Aggregators (signposts, not evidence)

Use ONLY to find the primary source. NEVER cite as the source. A CVE-ID templated into an aggregator page is not analysis.

- Sploitus
- cvemon
- Similar index sites

## Security boundary

Read the analysis text. Extract conditions. Never download or run linked artifacts (PoC repos, Nuclei templates, exploit binaries). Never contact named infrastructure from a writeup (no callbacks, no test requests, no verification URLs, no C2 endpoints, however harmlessly framed). Communication is restricted to authoritative data sources: CVE.org, vendor security pages, CISA KEV, FIRST EPSS, GHSA, OSV.dev.

Reading a writeup to extract a precondition is legitimate. Treating any linked artifact as runnable or any named host as contactable is not.
