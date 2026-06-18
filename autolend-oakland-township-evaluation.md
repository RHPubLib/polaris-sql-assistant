# Autolend Holds-Locker Kiosk — Technical Evaluation for Oakland Township

**Prepared by:** Derek Brown, Director of IT, Rochester Hills Public Library
**Date:** 2026-06-17
**Decision context:** Oakland Township Library Board (OTLB) votes on/about **2026-06-25**
**Vendor:** International Library Services (ILS) — **Bill McClendon, CTO**, Billm@internationallibraryservices.com
**Product:** Autolend 24/7 holds-pickup locker kiosk
**Our environment:** Polaris 8.1, hosted through Clarivate

> **Scope disclaimer:** This is a **technical-integration evaluation only.** It does **not** assess vendor
> uptime, real-world reliability, configuration effort, or day-to-day patron experience. RHPL has no prior
> track record with this company; those factors are explicitly out of scope and should be weighed separately
> by the board.

---

## Bottom line

Bill's reply (2026-06-16) is substantively reassuring and resolves most of the red flags from our initial
scoping. The architecture is a self-contained, vendor-managed device that fits *into* our network rather than
pulling us into a vendor cloud, and it **does not add meaningful ongoing load to either circ staff or IT.**
Remaining work is one-time setup plus a physical restocking cadence that needs an assigned owner.

**On a purely technical basis, this unit will work for Oakland Township** — subject to confirming encrypted
SIP2 transport and the open Clarivate/vendor items below.

> **Update 2026-06-18 — both vendors answered the two gating questions; see the dedicated section below.**
> Encryption path is settled (stunnel, free, both ends support it), static IP is now a confirmed hard
> requirement (the segmentation plan survives), and Bill's reply surfaced one **new** item: an inbound
> staff-web-interface exposure on the device's public cellular IP.

### Scorecard vs. original red flags

| Red flag | Verdict |
|---|---|
| SIP2-only, no PAPI | **Confirmed, but mitigated** — SIP2 for circ (incl. holds); PAPI used read-only for catalog/cover-art only, never to modify patron/item records |
| Separate notification system | **Resolved favorably** — fires the *native* Polaris notice when the item hits inventory (LX Starter / Vega experience, identical to a branch pickup) |
| Windows appliance vs. our Chrome OS shop | **Confirmed, but mitigated** — staff/admin UI is 100% web-based; the Windows 10 IoT box is vendor-managed remotely and never touches our endpoint fleet |
| No Clarivate-hosted reference | **Still open** — promised customer names once he has permission |

---

## Security assessment

**Posture is good, with one hard requirement and a few items to nail down.**

Favorable:
- **No vendor cloud.** Everything self-contained on the device; ILS/patron data is not in a third-party cloud.
- **Minimal PII handling.** Vendor deliberately avoids PAPI write paths to patron/item records. **Nothing is
  cached locally except catalog content** (cover art, summaries) for browse — confirms the no-local-patron-data
  assumption.
- **Supported, hardened OS.** Windows 10 IoT LTSC, supported through **Jan 2032**, industrial-grade; AV/security
  updates auto-applied off-hours; choice of Defender / Malwarebytes / our own toolkit.

**The one non-negotiable — encrypted transport.** TLS is supported by the vendor at no extra cost but is **not**
the default. Cleartext SIP2 carries patron barcode + PIN. We will **require TLS-only with no cleartext fallback**
for SIP2/PAPI. This is partly a Clarivate question — their SIP endpoint must accept the encrypted connection.

### Deployment decision: full cellular segmentation

RHPL's intended posture: run the unit on **its own cellular WAN, no VPN/VLAN back to RHPL.** The device talks
directly to our Clarivate-hosted Polaris, never touching the RHPL/OT network.

- **Benefit:** The vulnerability layer becomes device↔Clarivate, not through RHPL. RMM/update relationship is a
  **vendor-to-vendor contract** the board signs off on — *not* RHPL IT's responsibility. (Contrast: our mobile
  branches — kids bus, bookmobile — funnel through our Peplink Fusion network, which we monitor and control. We
  consciously give up that visibility here in exchange for full segmentation.)
- **Important nuance:** Network segmentation ≠ data segmentation. The device still transacts *our* patron records
  against *our* Clarivate Polaris, so RHPL/OT retain a **data-stewardship interest** even while off our network.
  This is exactly why TLS-only is load-bearing: with no VPN backstop, SIP2-over-TLS is the *only* thing protecting
  the patron PIN across the public cellular path.
- **Offline mode:** Lock to **fail-closed / out-of-service screen, offline mode OFF, no patron data on the unit.**
  (Bill's "end-to-end, nothing stored" claim contradicts an offline mode that buffers transactions locally — we
  want it disabled and confirmed in writing.)

---

## Workload impact

**Circ / librarians — LOW.**
- Holds route to the locker **exactly like any other pickup branch**; staff trap/transit/fulfill using today's
  workflow with no new policies or training. The device handles ILS-state logic internally.
- Patron notification is the **normal Polaris notice** — no parallel system.
- Expired/abandoned/re-routed holds: device **traps the state automatically** and shows staff color-coded counts;
  staff pull when convenient.
- **The one genuinely new task is physical restocking** (loading material via the on-screen admin UI). For a
  close-by OT site this is a recurring courier/staff trip. **Open logistical question: who owns the restock trips
  and at what cadence.**

**IT — LOW ongoing, modest one-time.**
- *One-time:* provision SIP2 login + least-privilege read-only PAPI; configure the locker as a Polaris **pickup
  branch/location with hold-pickup = True** (the "mini-branch" model — confirmed); set up the cellular + TLS path.
- *Ongoing: near-zero.* Vendor manages OS/patching/AV remotely and sends **active notifications** on issues. **No
  endpoint-management hooks on our side; it does not enter our Chrome OS / Google MDM footprint.**

**Accessibility (a strength worth telling the board — we serve the Oakland Talking Book Service):** full
side-access reach to 54"; large-button/high-contrast mode; text-to-speech with amplified audio; "browse only
ADA-reachable items" checkbox.

**Site note (Bill raised it):** a Michigan-winter install needs cover/vestibule — patrons won't use a metal/glass
surface in a blizzard; the device itself operates to −25°F. The proposed retail vestibule fits.

---

## What we're least confident about (ranked)

1. **Cellular plan vs. Clarivate SIP2 access controls.** Clarivate-hosted Polaris commonly gates SIP2 by **source-IP
   allowlist**; a cellular link typically gets a **dynamic/CGNAT IP**. If Clarivate requires a static source IP, the
   "pure cellular, no VPN" plan needs either a paid **static carrier IP** or a small **VPN to a fixed endpoint**
   (which reintroduces network involvement). **This is a Clarivate question** — submitted (below).
2. **Whether Clarivate enables/charges for third-party SIP2 + PAPI** on a hosted instance, and lead time. Also a
   Clarivate question — submitted.
3. **OT-vs-RHPL Polaris scoping** (whose branch code / collection / SIP login stocks the locker) — internal open item.
4. **Notification on items already resident in the locker** that Polaris later elects to fill a new hold — taken at
   face value, not independently verified.
5. **PIN handling in transit/at-rest** during normal online operation; any management interface reachable over the
   cellular link that should be closed.

Items 1–2 should resolve **before** the vote; 3–5 can be settled on the follow-up call.

---

## Clarivate Supportal ticket — SUBMITTED 2026-06-17

> **Status:** Acknowledged; Clarivate to respond within 1–2 business days.
> **Title:** Hosted Polaris — TLS-encrypted SIP2 from a third-party kiosk over cellular

**Summary of what we asked:**
- **TLS-encrypted SIP2:** supported/permitted into our hosted instance? requirements (endpoint/port, certs,
  session settings)? any non-TLS fallback to disable?
- **Source-IP / allowlisting:** does the hosted SIP2 service restrict by source-IP allowlist? can it accommodate a
  **dynamic/CGNAT cellular IP**, or do we need a **static/dedicated carrier IP** to stay on the allowlist?
- **Cost & lead time:** any Clarivate setup/config/recurring fee to enable TLS SIP2 and/or maintain an allowlist
  entry? typical lead time?
- **(Secondary) PAPI:** is read-only PAPI (catalog/bib content only) enabled at the account level, or does it need
  Clarivate provisioning, and at what cost?

We will handle branch/pickup-location creation and our own configuration; the ticket is strictly about Clarivate
support/enablement and cost.

---

## Follow-up email to Bill — SENT 2026-06-17

> Subject: Re: Autolend — Polaris integration questions

Thanks (that was fast and genuinely thorough), and the staff manual is a big help (which we'll keep confidential
per your note). It answered most of what we needed; our IT read is that this integrates cleanly with our Polaris
8.1 environment with low ongoing burden on our side. The fully web-based staff interface and self-contained
onboard PC in particular resolve our "no Windows endpoints" concern.

One thing that simplifies our deployment: we'd most likely run the unit fully network-segmented from us (its own
cellular connection, no VPN or VLAN back into our network) so the device talks directly to our Clarivate-hosted
Polaris rather than routing through RHPL. I've already opened a ticket with Clarivate to confirm the hosting-side
details: whether they support and permit TLS-encrypted SIP2 into our instance, and how their SIP2 endpoint handles
a dynamic/CGNAT cellular source IP (i.e., whether their allowlisting can accommodate a rotating carrier address or
whether we'd need to budget a static IP from the carrier). We'll fold their answer together with yours once we hear
back. You may even know how they have handled this with hosted customers in the past.

That leaves a short list, mostly confirmations on points you already touched...

**Reference sites:** A couple of Polaris customer references would be great to include in our board submission, so
whenever you have permission to share names, that'd be a big help.

**SIP2 encryption:** You noted you're TLS-compatible at no added cost. What we'd like to confirm is whether it can
be enforced as TLS-only, with no cleartext fallback, so the patron barcode/PIN is encrypted end-to-end across the
cellular path (we assume this is the case but it will help us answer any questions that may arise).

**Offline mode / local storage:** You said the out-of-service screen is the common choice and that offline
transactions are configurable. We'd want it set exactly that way: fail to out-of-service, offline mode off. Just
confirming that with offline mode disabled, nothing patron-related is written to the device, only the browse cache
(cover art/summaries) you described.

**Notification edge case:** You noted the device recognizes when Polaris elects a copy already inside the unit to
fill a hold, and that notification occurs. Just confirming that it fires the standard Polaris pickup notice (the
same one a patron gets when an item arrives via transit) rather than a device-generated message (my read from your
answer is again that is the standard Polaris method, but it's just to confirm).

Again I really appreciate all the information and how quickly you've been with the replies!

---

## Open items tracker

| # | Item | Owner | Status |
|---|------|-------|--------|
| 1 | TLS-only SIP2 supported into hosted instance | Clarivate | Ticket submitted 2026-06-17 |
| 2 | Cellular dynamic IP vs. allowlist / need for static IP | Clarivate | Ticket submitted 2026-06-17 |
| 3 | Clarivate setup/recurring cost + lead time | Clarivate | Ticket submitted 2026-06-17 |
| 4 | Read-only PAPI enablement + cost | Clarivate | Ticket submitted 2026-06-17 |
| 5 | TLS-only enforceable, no cleartext fallback | Bill (vendor) | Asked 2026-06-17 |
| 6 | Offline mode OFF, no patron data on device | Bill (vendor) | Asked 2026-06-17 |
| 7 | Standard Polaris notice for already-resident copies | Bill (vendor) | Asked 2026-06-17 |
| 8 | Polaris customer references for board submission | Bill (vendor) | Asked 2026-06-17 |
| 9 | OT-vs-RHPL Polaris scoping (branch/collection/SIP login) | RHPL internal | Open |
| 10 | Restock cadence + owner | RHPL/OT | Open |
| 11 | SIP2-vs-PAPI reassurance note for Director (Juliane) | RHPL internal | Optional / pending |
