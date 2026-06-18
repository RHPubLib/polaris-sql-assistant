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

## Update 2026-06-18 — vendor + Clarivate responses evaluated

Both Bill (ILS) and Mike O'Connor (Lead Systems Engineer, Clarivate) replied. This closes the two gating
questions and surfaces one new security item.

### 1. Encrypted SIP2 = stunnel, and it is the *only* encryption path — RESOLVED, with a caveat

- **Clarivate (Mike):** Polaris has **no native TLS on the SIP service**. The kiosk **cannot** TLS directly
  to the SIP port. The only encrypted option is Clarivate standing up an **stunnel** proxy, with the vendor
  running a compatible stunnel/TLS client on their end. Clarivate sets it up and provides connection steps.
  **No fee.**
- **Bill (ILS):** Describes his TLS as "a point-to-point tunnel on a port(s)… no clear-text fallbacks" —
  which *is* the stunnel model. Both ends therefore support it; it's free; Bill explicitly disclaims cleartext
  fallback inside the tunnel.
- **Caveat (load-bearing):** because there is no native TLS, the non-stunnel state is **cleartext, not
  "encrypted by default."** PIN/barcode protection rests entirely on stunnel being correctly stood up and the
  device never touching a raw SIP port. Bill's tone is mildly indifferent ("if they don't, no issues for us
  anyway… it's up to Hosting"), so **we must drive this**: make stunnel a contractual go-live gate, confirm
  Bill's client is stunnel-compatible per Clarivate's steps, and verify no plaintext SIP listener is reachable.

### 2. Static IP is now a HARD requirement — confirmed by both vendors, for two independent reasons

- **Clarivate:** the stunnel/SIP2 endpoint is **source-IP allowlisted**; Mike says a dynamic cellular IP
  "would **definitely** cause future problems."
- **Bill:** staff reach the web-based staff page at the device's **WAN address**, which rolls constantly on a
  non-static cellular link.

This is the cleanest resolution of our biggest open risk: the **"pure cellular, no VPN" architecture survives
intact.** A static carrier IP is still just the device's own WAN — it does **not** reintroduce RHPL network
involvement. Risk → procurement line item. Residual: confirm the carrier plan can provide a static IP and at
what monthly cost. (Bill floated allowlisting a whole **Class C /24** as an alternative — skip it: Mike's
"definitely cause problems" plus a /24 being a far broader trust surface than a single /32 both argue for the
static IP.)

### 3. NEW security finding — inbound staff-web-interface exposure on the public cellular IP

We had modeled the device as **outbound-only** to Clarivate. Bill's static-IP rationale reveals the
**web-based staff interface is reached at the device's public cellular IP** — i.e., an **inbound web service
exposed on the open internet.** This is the most important new item.

Questions for Bill before we're comfortable:
- Is the staff interface **HTTPS-only with a valid cert**, or plain HTTP?
- Auth: per-user accounts, lockout/brute-force protection, MFA? Any default credentials to change?
- Can inbound access be **IP-restricted (device-side firewall allowlist)** to RHPL/OT staff source IPs?
- Can staff functions be reached **through the vendor remote-access portal instead**, so **no inbound port is
  exposed at all**?

**Recommendation:** do not leave the staff page open to the whole internet. Either firewall it to allowlisted
staff IPs, or disable the public staff page and rely on (a) the on-screen physical admin UI for restocking and
(b) the vendor remote-access portal for remote work. Closing the staff page removes the *staff-access* reason
for a static IP, but Clarivate's allowlist reason still mandates it — **static IP stays either way.**

### 4. Vendor "Remote Access" RMM dashboard — understand and bound it

Bill noted ILS's support/maintenance includes a **free account on the same Remote Access dashboard they use**,
offered to RHPL IT, separate from the staff web interface. Almost certainly a **cloud-brokered outbound tunnel**
(device dials vendor cloud, admins log into a portal) — if so, no inbound exposure, which is fine. Confirm it's
broker/outbound, what it can do (likely full control of the Win10 IoT box), and the auth model. Under the
segmentation decision this is a vendor↔vendor responsibility, but the board should understand a vendor plane can
fully control a device transacting our patron data. **Taking the free IT account is worth it for visibility.**

### 5. Other confirmations from Bill

- **Offline mode:** confirmed — if not enabled, no browse screen; displays a **configurable "Out of Service"**
  message. Matches our fail-closed requirement. (Resolves open item #6.)
- **Notification edge case:** confirmed — "the ILS is left to send whichever notice you have it configured to
  send," i.e., the **standard Polaris notice**, not a device-generated one. (Resolves open item #7.)
- **References:** still pending — Bill awaiting permission to share names. (#8 open.)

### Must-haves before the 6/25 vote (all now tractable — no showstoppers)

1. Confirm a **static cellular IP** is obtainable and its monthly cost.
2. Get **Clarivate and Bill to agree the stunnel setup**, with us enforcing no-cleartext (cert exchange,
   endpoint/port, the static source IP to allowlist).
3. Decide the **staff-interface exposure** question (IP-restrict vs. disable-in-favor-of-portal).

Still unanswered by Clarivate: **read-only PAPI enablement/cost** and **lead time** (Mike addressed only
stunnel + allowlisting).

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

## Second follow-up to Bill — SENT 2026-06-18

After folding in Clarivate's stunnel/allowlist answers, a second email went to Bill. Questions asked:

- **stunnel setup:** how to configure the AutoLend to ride Clarivate's stunnel tunnel, and how to guarantee
  the device only ever talks over it and never falls back to a cleartext SIP port.
- **Connection method / stable IP:** asked Bill to recommend how to connect the unit given Clarivate's
  source-IP allowlist (a changing cellular IP won't work); deferring to what's worked at his hosted-Polaris
  cellular installs. Floated the RHPL **Peplink + FusionHub** design (below) and asked the gating question:
  **can the AutoLend run off an Ethernet WAN handoff instead of its internal cellular box?**
- **Identity provider / SSO:** does the staff interface support **Google OAuth 2.0 / OIDC or SAML** so staff
  authenticate with RHPL accounts and inherit our **YubiKey FIDO2 MFA** (as we run LEAP)? If not native, can it
  sit behind an IdP-aware gateway? If local accounts are unavoidable — individual named accounts (password
  policy + lockout) and FIDO2/second-factor support on them, not one shared login.
- **Staff interface exposure:** served directly from the device's public IP over HTTPS (valid cert) and
  IP-restrictable to our ranges, or reachable via the vendor remote-access portal so no inbound port is exposed.
- **Remote Access dashboard:** taking the no-charge IT account; can it authenticate via our IdP too.
- **Catalog content channel:** re-confirm it's the read-only PAPI (bib/catalog only, no patron/item writes).
- **References:** nudge — ideally a hosted-Polaris site.

### Candidate connectivity design — Peplink + FusionHub (RHPL-managed)

Verified approach for giving Clarivate one stable allowlisted IP over cellular (RHPL already runs Peplink
SpeedFusion on the bookmobile and kids' bus):

```
AutoLend --Ethernet--> Peplink router (2 SIMs, 2 carriers)
        --SpeedFusion bonded/encrypted tunnel--> FusionHub (RHPL-hosted VM, static public IP)
        --stunnel--> Clarivate Polaris SIP2
```

- The **static IP comes from the FusionHub**, not the cellular links — Clarivate allowlists that fixed IP; the
  cellular WAN IPs underneath can churn freely. Dual-carrier = link redundancy.
- **stunnel still rides end-to-end inside the path** (SpeedFusion encrypts device→FusionHub; stunnel encrypts the
  SIP2 session and the FusionHub→Clarivate leg). The two layers stack.
- **FusionHub Solo is free, no throughput cap**, runs on VMware (host on existing ESXi); needs a static public
  IP at the host (RHPL HQ link works). Alternative: paid SpeedFusion Cloud with a dedicated IP.
- **Tradeoff vs. the original 6/17 "fully segmented, vendor-to-vendor" posture:** this puts RHPL gear (Peplink +
  FusionHub) back in the data path — so it becomes **RHPL-managed connectivity**, not off-our-network. That's a
  *stronger* control/redundancy story for the board, but a different framing of responsibility. Present to the
  board as a choice: (A) vendor-managed cellular + purchased static carrier IP (hands-off), or (B) RHPL-managed
  Peplink + FusionHub (more control/redundancy, leverages gear/skills we already have). **(B) is the likely
  recommendation.** Hinges on Bill confirming the unit accepts an Ethernet WAN.
- Sources: Peplink SpeedFusion Bonding tech; FusionHub Solo (free, no throughput limit) — peplink.com /
  Peplink Community.

---

## Bill reply #2 (2026-06-18) — second follow-up answered; deployment models for the board

Bill answered the second follow-up. The remaining **technical** gates are cleared, so we can present the
board a clean **two-model choice** rather than a list of open questions.

What his answers resolved:
- **Ethernet / WiFi WAN accepted (#17 RESOLVED).** *"We can take a wired WAN connection or a WiFi WAN
  connection, and everything else."* This unblocks the Peplink design, and he endorses it: *"If it works
  for your bookmobile, it should work for us too… the only difference would be the stunnel setup."*
- **stunnel (#14).** ~30 min to configure, 5 to test; both ends support it; toggled by a background
  service; he insists on a **live holds-pickup test** ("every site's holds can be different").
- **Staff web portal (#13 RESOLVED).** Defaults to **HTTP**, but **no PII is stored or displayed at all**.
  Hardening options: our cert → HTTPS (~1 hr, we supply the cert), IP-restrict, self-signed cert in a
  closed network, or front it with **our own auth/MFA proxy**. Every option except a public cert assumes a
  **LAN / closed network** — which only **Model 2** provides.
- **Admin screen (#16).** On-device, **local-only** card/number (can be a dummy card unconnected to the
  ILS), role-scoped; nothing leaves the box. OAuth/MFA available (from their Academic sites) if we ever
  want it. Web-portal SSO + FIDO2 is achieved via our proxy under Model 2.
- **RMM = Splashtop (#15 RESOLVED).** Cloud-brokered (outbound, no inbound port), full encrypted VPN, free
  IT account, its **own MFA** (not our IdP). It is a vendor cloud **control plane** (can fully control the
  Win10 box) even though the **data path** has no vendor cloud. Take the free account for visibility;
  document it.
- **Catalog channel (#18 RESOLVED).** *"Read/search only for item/catalog records. No calls to modify any
  data of any kind."*
- **References (#8).** Bill expects to hear back with names **today (6/18)**; chasing.

**Clarivate (understood — no open question remaining for the design):** we provide a **dedicated/static
source IP** for their allowlist; their **stunnel setup is no-cost**; and they will **assist standing up the
tunnel between our Peplink/FusionHub edge and their SIP endpoint** — that is where the encrypted tunnel
terminates. Only residual is generic **lead time**.

### The two models we'll put to the Director and OTLB

**Model 1 — Vendor-default ("it works," lower security; *not* RHPL's normal standard).**
- Connectivity: the unit's **own cellular line** with a **purchased static carrier IP** (required for
  Clarivate's allowlist). Single carrier = single point of failure.
- SIP2 patron barcode/PIN encrypted via **stunnel** (baseline; free on both ends).
- **Staff HTTP portal left at vendor default** — not locked to RHPL standards (no reverse proxy, no
  FIDO2). Bounded by Bill's "no PII displayed," but below what we would normally deploy.
- RMM via Splashtop. **No RHPL gear, no SpeedFusion VPN, no redundancy.**
- Cost: monthly **static carrier IP + one cellular line**; **no** Peplink hardware/software.

**Model 2 — RHPL-managed, locked down (RECOMMENDED).**
- Path: **AutoLend → Ethernet → Peplink → SpeedFusion VPN → FusionHub (our static IP) → stunnel →
  Clarivate.** SpeedFusion encrypts the entire device→RHPL path; **stunnel still rides inside** for the
  SIP2 session and the FusionHub→Clarivate leg (defense in depth).
- The **static IP Clarivate allowlists is our FusionHub's**, not a churning cellular IP; Clarivate assists
  standing up the tunnel at our edge.
- **Staff portal locked to RHPL standards:** reachable only through our network, behind our **reverse proxy
  with Google OAuth + YubiKey FIDO2**; never exposed to the public internet.
- Availability: **one cellular SIM, or two different carriers bonded** via SpeedFusion for best
  uptime/redundancy.
- Cost: **Peplink hardware + software/license + monthly fee for the cellular line(s)** (two lines if
  dual-carrier).

Both models satisfy the non-negotiable — encrypted patron credentials in transit. The difference is
**staff-portal hardening, network control/visibility, and redundancy**, all of which Model 2 adds, at the
cost of RHPL gear + (optionally) a second cellular line. **Scoping (#9) and restock cadence (#10) are
deferred — not material to this decision right now.**

---

## Concerns we are keeping visible (not closed by vendor assurances) — 2026-06-18

These stay in the write-up deliberately. Most of our "resolved" items rest on the **vendor's description of
its own closed system**; a fast, articulate vendor thread is not the same as verified behavior. Ranked:

> **Basis-of-confidence note (and what it is NOT).** Everything in this evaluation about how the device
> behaves and protects data comes from **the vendor (Bill/ILS) describing his own product** — not from
> RHPL testing, a third party, or reference customers (we have **none yet**). We are **taking the vendor at
> his word** that he understands his system's logic and is being honest. **This is not a recommendation
> against the locker, and we are not telling OT to reject it** — it is simply an honest statement that our
> technical "yes" is **"yes, per the vendor, pending verification."** The remedy is a hands-on
> proof-of-integration test (packet capture + a test patron) as a go-live gate, plus real references.

1. **Load-bearing claims are vendor assertions, not observed behavior.** "No PII stored," "no cleartext
   fallback," "read/search-only PAPI, no writes," "offline-off = nothing written," "loader role can't
   escalate" — all are Bill describing his own box. **Treat as *claimed*, not *confirmed*, until verified**
   (packet capture + a test patron, watching the live SIP2/PAPI traffic). This should be a written
   **go-live gate**, not a leap of faith.
2. **"No PII at all" does not survive first principles for a *holds* locker.** A holds locker must know
   *which patron's* item sits in *which bin* to release it correctly — so the device necessarily handles
   patron-identifying hold data. The claim is almost certainly scoped to the *staff web pages*, not the
   device itself. **Pin down exactly what patron data lives on the box, where, for how long, and whether
   it is encrypted at rest.**
3. **Vendor-managed Windows box + vendor remote SYSTEM access + locally-stored SIP2 creds + no RHPL track
   record with this vendor.** The residual we cannot fully fix (R3): a vendor/Splashtop compromise yields
   SIP2 creds and a pre-authenticated path to patron data. Least-privilege + audit reduce it, not remove it.
4. **The exact integration is unproven *to us*, and no hosted-Clarivate reference is in hand.** We have
   never watched this product run in the specific Clarivate-hosted-Polaris-over-stunnel configuration we
   are proposing, from a vendor we have never worked with. If references don't arrive — or none are
   hosted-Clarivate — that gap stands directly under the recommendation.
5. **The security model is fragile by construction.** Polaris has no native SIP TLS, so the *entire*
   patron-credential protection is a bolted-on stunnel proxy both ends must configure correctly, with a
   cleartext fallback if misconfigured. It works, but a PAPI-only design would not have this failure mode.

## Alternative vendor noted — Lyngsoe (PAPI-native, no SIP2) — 2026-06-18

Recorded for the Director/board because it directly addresses our **standing reservation** (we'd prefer a
PAPI-only product over SIP2). **At least one alternative — Lyngsoe — can run all traffic through PAPI, with
no SIP2 tunneling.** Lyngsoe is a large, established locker vendor with **known Polaris libraries** running
its hardware — a real track-record advantage over ILS. **Sourcing caveat:** most detail below is
**secondhand via a Phoenix-area Polaris library (security-focused), not confirmed by Lyngsoe to us
directly.**

What we've been told (via that Polaris library):
- **Two hardware lines.** Lyngsoe's own model is **hold-lockers only**; a **separate partner locker
  vendor** provides **"Library of Things" (LoT)** lending — staff stage items for **browsable checkout**,
  patrons select and check out directly (reportedly glass / mergeable doors for flexible sizes). So the
  browsable/vending capability we'd want comes via that **partner**, not Lyngsoe's own hold-locker unit.
- **PAPI maturity is the catch.** PAPI support **launched only this past March (2026), and only for LoT
  customers.** **No one is yet running Lyngsoe *hold lockers* over PAPI** — that library would be the
  **first**. They saw an in-person Lyngsoe PAPI demo (Feb 2026) and hope to move their hold lockers to
  PAPI "soon." So PAPI-for-holds is **real but not production-proven** — not a drop-in mature option.
  *Added nuance from RHPL's own PAPI experience:* our PAPI use to date is **read-only** (catalog + patron
  lookup; see `docs/polaris-api-claude.md`, Freegal), and patron/item **write** operations have been
  limited via PAPI — consistent with why holds lockers have historically used SIP2 and why PAPI-for-holds
  is genuinely new **across the board**, not just for Lyngsoe. Worth verifying that RHPL's Clarivate-hosted
  Polaris PAPI even exposes the hold/checkout write operations a PAPI-native locker would require.
- **Staff auth: same gap as ILS.** Lyngsoe currently uses **local accounts hosted by Lyngsoe**, with
  **MFA still in development** ("soon"); that library volunteered to partner on MFA setup/testing.

**Net:** Lyngsoe is a credible, larger, Polaris-proven, PAPI-capable alternative that fits our protocol
preference — but for the **hold-locker + PAPI** combination specifically it is **as unproven as ILS**, the
browsable option rides a partner vendor, and its staff MFA has also not shipped. It belongs in the
write-up as evidence that a PAPI-only path **exists in the market** (reinforcing that a fuller procurement
should compare vendors) — **not** as a turnkey, readier-than-ILS substitute for this specific use case.

## Open items tracker

| # | Item | Owner | Status |
|---|------|-------|--------|
| 1 | TLS SIP2 supported into hosted instance | Clarivate | **RESOLVED 6/18** — yes, via stunnel; free; vendor must run compatible stunnel client |
| 2 | Cellular dynamic IP vs. allowlist / need for static IP | Clarivate | **ANSWERED 6/18** — dynamic IP "will definitely cause problems" → static IP required |
| 3 | Clarivate setup/recurring cost + lead time | Clarivate | **PARTIAL** — stunnel free; lead time still unknown |
| 4 | Read-only PAPI enablement + cost | Clarivate | **Open** — not addressed in Mike's reply; follow up |
| 5 | TLS enforceable, no cleartext fallback | Bill (vendor) | **RESOLVED 6/18** — no cleartext fallback in tunnel; enforce that Polaris's cleartext SIP is never the fallback |
| 6 | Offline mode OFF, no patron data on device | Bill (vendor) | **RESOLVED 6/18** — configurable "Out of Service"; fail-closed confirmed |
| 7 | Standard Polaris notice for already-resident copies | Bill (vendor) | **RESOLVED 6/18** — ILS sends the configured (standard Polaris) notice |
| 8 | Polaris customer references for board submission | Bill (vendor) | **Open** — Bill expects names back **today (6/18)**; chasing |
| 9 | OT-vs-RHPL Polaris scoping (branch/collection/SIP login) | RHPL internal | **Deferred** — not material to this decision now |
| 10 | Restock cadence + owner | RHPL/OT | **Deferred** — not material to this decision now |
| 11 | SIP2-vs-PAPI reassurance note for Director (Juliane) | RHPL internal | Optional / pending |
| 12 | Stable allowlisted source IP — Peplink+FusionHub (preferred) vs. carrier static IP | RHPL / Bill | **RESOLVED 6/18** — folded into the two-model choice (Model 2 = FusionHub static IP) |
| 13 | Inbound staff-web-interface exposure (HTTPS? auth? IP-restrict or via portal?) | Bill (vendor) / RHPL | **RESOLVED 6/18** — HTTP default, no PII; harden via cert / IP-restrict / our auth-proxy (proxy under Model 2) |
| 14 | stunnel setup guidance + guarantee no cleartext-SIP fallback | Bill (vendor) | **RESOLVED 6/18** — ~30 min setup, both ends; live holds-pickup test required |
| 15 | Vendor Remote Access RMM: broker/outbound? capabilities? IdP auth? accept free IT account | Bill (vendor) / RHPL | **RESOLVED 6/18** — Splashtop, cloud-brokered/outbound, own MFA (not our IdP); take free IT acct |
| 16 | Staff SSO via Google OAuth/OIDC or SAML + FIDO2 MFA; named accounts vs. shared | Bill (vendor) | **RESOLVED 6/18** — admin screen local-card only; web-portal FIDO2 via our proxy (Model 2) |
| 17 | AutoLend accepts Ethernet WAN handoff (gates the Peplink design) | Bill (vendor) | **RESOLVED 6/18** — wired or WiFi WAN accepted; Peplink design endorsed |
| 18 | Read-only PAPI content channel re-confirm (bib only, no patron/item writes) | Bill (vendor) | **RESOLVED 6/18** — read/search only, no writes of any kind |
