# Verdict Taxonomy

## The three verdicts

### affected
The CVE applies to the environment. At least one inventory product (or a component derived from one, see `condition-archetypes.md` and SKILL.md section on component derivation) intersects the CVE's target, and every discriminating condition that gates exploitation resolves in favor of exposure (or no exonerating condition exists). Sourcing must include the identity anchor and at least one condition source that satisfies the two-anchor assignment test.

### not affected
The CVE does not apply because at least one named exonerating condition (a configuration, deployment model, feature state, build range) clearly excludes the environment. This is the dangerous verdict: a false negative means a real exposure is silently ignored.

Held to a higher evidentiary bar:

- The exonerating condition must come from an authoritative source (the identity anchor itself, a vendor / PSIRT advisory, or established research as defined in `trusted-sources.md`), OR
- The product is absent from an inventory section the user has explicitly marked complete.

On thin sourcing (a single unconfirmed claim, an aggregator-only mention, a writeup that does not pass the two-anchor test), the verdict degrades to needs-verification. Never a silent all-clear.

#### Mitigation vs. non-applicability

Not everything that reduces risk makes a CVE not-applicable. Non-reachability of the vulnerable path can support not-affected: if the vulnerable code is genuinely not reachable in the environment, the exonerating condition holds. A compensating control is different. A WAF rule, a network ACL, a SELinux or AppArmor profile, or enforced authentication lowers urgency, but it supports not-affected ONLY when an authoritative source states that this specific mitigation eliminates the vulnerable code path. Absent that, the control is a hardening factor, not exoneration: the verdict stays affected or needs-verification, with the mitigation noted on the urgency line.

### needs verification
There is no clean basis to call it either way. The product is in the environment (or derives from something that is), but a discriminating condition cannot be resolved with the current information. This is the default when in doubt.

Common reasons:

- The deciding condition is volatile (e.g. a specific build number) and the inventory value is stale.
- The inventory is silent on a relevant section and that section is not marked complete.
- Sources conflict on the condition.
- The CVE is too fresh and the affected range or condition statement is still in motion.

Every needs-verification verdict MUST name the single smallest concrete next check that would resolve it, phrased as closed as possible. Never a generic "inventory incomplete" or "could not determine". Bad: "needs verification: inventory incomplete." Good: "needs verification. I need exactly one fact: do you run any on-prem Exchange Server with OWA enabled? If no and your Mail section is complete, this becomes not affected." If a single closed question would resolve it, prefer asking it up front (SKILL.md Layout B); if the verdict is still issued as needs-verification, it must carry that smallest next check.

## Asymmetry rule

Affected and needs-verification are tolerable defaults. not-affected is the verdict that can silently leave the user exposed, so it has a higher bar than the other two. Symmetry between verdicts would be the wrong design: the cost of the two error modes is not equal.

## Sourcing minimums per verdict

| Verdict | Identity anchor | Condition sourcing |
|---|---|---|
| affected | required | at least one source passing the two-anchor test for the asserting condition |
| not affected | required | authoritative source (identity anchor / vendor / established research) for the exonerating condition, OR a complete-marked inventory section |
| needs verification | required | not strictly required, but the open question must be named |

A verdict that rests only on urgency sources (NVD CVSS, KEV, EPSS) or only on ecosystem sources (GHSA, OSV) for applicability is not permitted. Those sources do not carry applicability; they carry urgency or version ranges. Such a verdict defaults to needs-verification.

## Freshness as confidence modulator

Freshness is not a footer tag. It is part of the verdict. A fresh CVE has three axes that can still move:

1. The affected range (often corrected or expanded post-publication).
2. The exploitation status (PoC drop, KEV listing, in-the-wild reports).
3. Sometimes the condition itself (vendor refines guidance days later).

A fresh not-affected verdict is structurally weaker than the same verdict on a two-year-old CVE and must be phrased accordingly. Example: "not affected: as of now, affected-range still in motion, re-check in a few days."

### NVD enrichment trap

NIST prioritizes enrichment for selected CVEs (KEV, government-context and critical software) since around April 2026; industry analyses estimate this leaves only roughly 15-20% of incoming CVEs with timely full enrichment in NVD. "No CPE match in NVD" therefore means nothing for a fresh CVE and must NEVER support a not-affected verdict on its own. Time-to-exploit has been trending negative for some classes of vulnerability (exploitation preceding disclosure), which sharpens the caution on fresh not-affected verdicts.

## Absence of evidence

If a product is absent from the inventory, the default verdict is needs-verification, never a silent not-affected.

### The completeness flag

A user may mark an inventory section as complete. If a section is marked complete and the product is absent from it, absence may become not-affected. The verdict text must always state what it rests on, so an inventory gap can never silently carry a false all-clear.

When a verdict lands on needs-verification specifically because a section is not marked complete, append one contextual line:

> Verdict is needs-verification because your inventory in this area is not marked complete; if your list of [appliances / servers / ...] is complete, mark the section and this becomes not affected.

The user never edits the file by hand. If they confirm completeness in chat, propose the flag, get confirmation, write it back via the write-back mechanism.

### Scope caveat for not-affected

Absence of evidence covers a product missing from a section. One granularity coarser is the case where whole systems are missing from the inventory entirely. A user asking "am I affected by CVE-X" is usually thinking of their whole infrastructure, but the assessment can only evaluate what the profile records. If the profile holds one host and the user runs several, a bare not-affected invites the most expensive misread of a security verdict: reading a host-scoped clear as an environment-wide one. The product-scoped header (naming the specific host) already binds the verdict to what was checked, but it does not actively flag that other, unrecorded systems may exist and were not assessed. Because the cost of the too-broad reading is high and the cost of one extra sentence is low, the verdict states its scope.

Append a one-line scope caveat only when ALL three hold:

- the verdict is not-affected. Affected already compels action and needs-verification is already non-reassuring, so neither gets the caveat.
- the inventory section the verdict relied on is NOT marked complete.
- the CVE targets a product or derived component class a user plausibly runs in multiple instances (an operating system, a kernel, a web server, a database), not something inherently singular.

When it applies, the caveat names the scope and offers the self-disabling exit in one line:

> Scoped to the systems recorded in your profile (here: the one Debian 13 host). If you run other Linux systems, they are not assessed here. Tell me once your inventory is complete and this note drops.

It is self-disabling: once the relevant inventory section is marked complete (via the same write-back-with-confirmation flow as the completeness flag above), completeness has been affirmed and the caveat no longer appears. It is not a generic per-verdict disclaimer and must not appear on every verdict; the three triggers gate it. It never changes the verdict. Not-affected stays not-affected; the caveat only bounds what "not affected" ranges over, and must not be used to hedge the verdict into something mushier. It teaches the completeness flag at the moment the flag would change the answer, rather than explaining it upfront.

The scope caveat and the absence-of-evidence rule are two granularities of one principle: never let an inventory gap carry a silent all-clear. Absence of evidence guards the case where a product is missing from a section; the scope caveat guards the case where whole systems are missing from the inventory.

## Conflict rule

### Identity conflict
Two identity sources disagree (CVE.org vs. cvelistV5 disagree on affected ranges, or the record status disagrees). No verdict. Flag the conflict, show both, note in footer.

### Condition conflict
Vendor says "only under condition Y", established research demonstrates exploitability without Y. Do NOT silently resolve in favor of the all-clear. Flag as conflict, show both statements, the verdict goes conservative (affected or needs-verification). Vendor "not affected" vs. researcher "affected" must NEVER silently resolve to the reassuring side.
