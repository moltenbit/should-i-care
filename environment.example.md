# Environment profile (example template)

<!--
DO NOT copy this file.

The skill creates `environment.md` for you on the first CVE assessment: it
asks the minimum facts it needs, you confirm, and it writes them back to the
file. From then on every change goes through the same write-back-with-
confirmation path. You should rarely if ever hand-edit `environment.md`.

This file (`environment.example.md`) exists as a read-only reference. It
shows the format the skill writes, so when you open the real
`environment.md` you can recognize what is in it. The example entries below
are illustrative; they are not your environment and must not be copied into
it.

The format has two layers:
  Layer 1 (Inventory): products, versions, deployment model, exposure.
  Layer 2 (Conditions): discriminating settings the skill has learned about.

Plus per-section the optional `complete: true` flag (see Layer 1 below) and
per-Layer-2-entry the volatility tag (`stable` | `volatile`).
-->

## Layer 1: Inventory

<!--
Each section may carry an optional completeness flag at the top of the section:
  complete: true

If a section is marked complete and a product is absent from it, the skill may
verdict "not affected". If the section is NOT marked complete, absence leads
to "needs verification". You do not have to mark anything complete; leave the
flag off until you are sure the section is exhaustive.

The optional `exposure:` attribute (internet-facing | internal | unsure) is
only meaningful on entries that accept inbound connections: server services
and network/appliance gear. It is never attached to client OSes, laptops, or
mobile devices, and it only sharpens urgency; it never changes applicability.
-->

### Mail

- **Exchange**: Exchange Online only (EXO). All mailboxes migrated; no on-prem Exchange servers remain.
  - deployment: cloud
  - notes: flagship Layer-1 fact, makes any on-prem-only Exchange CVE (e.g. OWA frontend bugs) not-applicable

### Directory services

- **Active Directory**: on-prem, single forest, two domain controllers.
  - version: Windows Server 2022, fully patched as of 2026-05-10
  - exposure: internal only
  <!--
    The skill derives Kerberos, NTLM, LDAP, AD CS and similar subsystems from
    "Active Directory" at the gate; you do not need to list those underlying
    components yourself.
  -->

### Endpoints

<!--
  No exposure attribute on any entry here: these are client devices and mobile
  endpoints, they do not listen for inbound connections, so exposure is not a
  meaningful property for them.
-->

- **MacBook Pro**: 8 units, macOS 15.4
- **iPhone 17**: 12 units, managed via MDM
  <!--
    The skill derives iOS from "iPhone 17" at the gate, so an iOS-specific CVE
    is recognized even though "iOS" is not written here literally.
  -->

### Web / proxy

- **nginx**: 1.25.3
  - role: TLS-terminating reverse proxy
  - deployment: bare metal, 2 nodes
  - exposure: internet-facing (source: user-confirmed, 2026-05-20)  <!-- written back when a network-reachable CVE made it relevant -->
  - cpe: cpe:2.3:a:nginx:nginx:1.25.3:*:*:*:*:*:*:*  <!-- optional power-user field, never required -->

### Network appliances
complete: true
<!--
  This section is marked complete: absence of a product here will support a
  "not affected" verdict for a CVE that targets an appliance not listed below.
-->

- **FortiGate 100F**: FortiOS 7.4.3
  - role: perimeter firewall
  - exposure: internet-facing on WAN

### Server services
<!--
  This section is NOT marked complete: absence of a product here leads to
  "needs verification", because the inventory could still be missing things.
-->

- **PostgreSQL**: 16.3
  - role: application database
  - exposure: internal only

---

## Layer 2: Conditions

<!--
The skill writes Layer 2 as a single Markdown table, one row per condition,
columns: Condition | value | last_verified (YYYY-MM-DD) | source |
volatility (stable | volatile) | notes. It appends and updates rows as it
learns conditions. Examples below.
-->

| Condition | value | last_verified | source | volatility | notes |
|---|---|---|---|---|---|
| RC4 allowed as Kerberos etype | no | 2026-05-01 | user-confirmed | stable | domain policy enforces AES only |
| OWA in use (on-prem) | no | 2026-05-15 | user-confirmed | stable | derived from EXO-only Exchange deployment in Layer 1 |
| Exchange Online build | 24.192.xx | 2026-05-07 | user-confirmed | volatile | cloud service, build changes frequently; re-confirm at next assessment that touches it |

---

## Assessment log

<!--
Illustrative entries, not your environment. The skill appends one row per
assessment via write-back-with-confirmation: date, CVE ID, verdict, and a
one-line basis. Entries are appended, never silently rewritten; this is the
audit trail and the "why was this cleared" memory for later assessments.
-->

| date | CVE | verdict | basis |
|---|---|---|---|
| 2026-05-18 | CVE-2026-XXXX | not affected | Exchange OWA frontend bug; all mailboxes EXO-only, on-prem OWA vector not present |
| 2026-05-21 | CVE-2026-YYYY | not affected | nginx HTTP/3 handler overflow; HTTP/3 not enabled on the proxy, vector not reachable |
