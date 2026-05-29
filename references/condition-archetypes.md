# Condition Archetypes

A catalog of the condition types that gate CVE applicability. When reading the canonical record and secondary sources, this is the shape of the question being asked: under what circumstance does the bug actually fire?

Each archetype names the condition and what to check in the environment file (or ask the user for via write-back).

## Component derivation reminder

Before any of the archetypes below, remember the component-derivation rule from SKILL.md: an inventory product implies its platform and direct subsystems.

- iPhone → iOS
- Active Directory → Kerberos, NTLM, LDAP, AD CS, AD roles
- Exchange Server → OWA, transport, mailbox role
- Windows Server → its installable roles

Derivation decides PRESENCE at the gate ("is the affected thing even here?"). The archetypes below decide whether, given presence, the condition then actually FIRES under the user's specific settings. Derivation does not answer the discriminating condition: that AD is present does not tell you whether RC4 is still allowed as a Kerberos etype.

Keep derivation conservative (immediate components and direct subsystems, not deep transitive chains). When in doubt, let the CVE through the gate and resolve it in the condition step. The dangerous error is dropping an applicable CVE on the gate, not carrying a non-applicable one into condition analysis.

## Archetypes

### feature-gated
The CVE fires only when a specific feature or role is enabled.
- **Check:** is the feature/role listed in the inventory as enabled? If silent, ask.
- **Examples:** only when OWA frontend is in use; only when the IIS WebDAV module is enabled; only when a specific MDM policy is active.

### config-gated
The CVE fires only under a non-default configuration value.
- **Check:** is the setting in Layer 2 (conditions)? If absent, ask and write back.
- **Examples:** only when `AllowAnonymous = true`; only when verbose logging exposes secrets; only when an admin override has loosened a default.

### protocol-in-use
The CVE fires only if a specific protocol or algorithm is still permitted.
- **Check:** Layer 2 protocol entries (RC4 allowed, NTLMv1 allowed, SSLv3 not yet disabled, etc.).
- **Examples:** Kerberos RC4 downgrade requires RC4 still being an allowed etype; SMBv1 attacks require SMBv1 still bound.

### build-range
The most common discriminant: only specific builds / patch levels are affected.
- **Check:** Layer 1 version/build entries. If the build is volatile (frequently updated cloud product) confirm freshness; if stable (on-prem with a known cadence) treat accordingly.
- **Examples:** Exchange CU range; iOS build-specific bug; a fixed range of nginx patch versions.

### exposure
The CVE applies in principle, but exposure (internet-facing vs. internal) decides priority and, sometimes, applicability.
- **Check:** Layer 1 exposure entries on the product. Internet-facing strongly raises urgency; internal-only may keep applicability while collapsing urgency.
- **Examples:** an unauthenticated RCE on an internet-facing appliance versus the same product internal-only with strict ACLs.

### auth / privilege precondition
The CVE requires a particular pre-state from the attacker (no auth, authenticated, admin, SYSTEM-local).
- **Check:** who could plausibly meet that pre-state in the user's setup. Pre-auth on an internet-facing surface is the high-leverage case.
- **Examples:** requires Domain User; requires SYSTEM-local; requires an authenticated session token; requires Domain Admin (rarely interesting since they could do it anyway).

### network-adjacency
The CVE requires the attacker to occupy a specific network position (adjacent network, local network only, link-local).
- **Check:** is that position plausible given exposure and segmentation? Layer 1 exposure plus any segmentation notes.
- **Examples:** must be on the same Wi-Fi segment; must reach an internal management VLAN; same broadcast domain.

### non-default configuration
A general bucket for "fires only outside default state". Use when no more specific archetype fits.
- **Check:** identify the deviation from default, store as a Layer 2 condition.
- **Examples:** a hardening setting that was relaxed for compatibility; a debug flag left on in production.

### platform / OS specifics
The CVE applies only on a given OS or architecture.
- **Check:** Layer 1 platform entries, including derived components (iPhone implies iOS, so an iOS-specific CVE applies; the iPhone listing does not have to literally say "iOS").
- **Examples:** only on Windows Server 2022; only on aarch64; only on iOS 17.x; only on a specific glibc range.
