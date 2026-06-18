# PLAN — Autolend Kiosk: what RHPL presents to the Director and OTLB (vote ~2026-06-25)

**Artifact being hardened:** the technical-feasibility + security **memo** RHPL (Derek Brown, IT
Director) delivers into the Oakland Township Library Board decision, plus the separate verbal/written
**Director brief** to Juliane Morian. This plan defines the *content and stance* of that memo — no code
or config is built here.

Source of record: `/var/opt/rhpl/polarissql/autolend-oakland-township-evaluation.md` (full scorecard,
vendor/Clarivate threads, open-items tracker).

---

## Goal

Give OTLB a clear, honest **technical-feasibility + security** answer so they can take an informed
**purchase yes/no** vote on ~2026-06-25, and give Juliane the RHPL operational picture behind it.

- The board decision is **Option A**: they vote to *purchase the Autolend kiosk* (OT-owned). They are
  **not** asked to pick a network model — RHPL tells them which one it will run and why.
- The memo's answer: **on a purely technical basis, the Autolend integrates cleanly with RHPL's
  Clarivate-hosted Polaris 8.1, and RHPL recommends operating it under Model 2 (locked down).**
- Honest positioning: this is **"is *this* vendor's kiosk technically workable,"** *not* "this is the
  best kiosk on the market." (See Scope — PAPI-only reservation.)

## Approach — what the memo says, in order

1. **Bottom line up front.** Technically feasible with our Polaris; RHPL recommends purchase *from a
   technical standpoint*, to be operated under **Model 2**. Two non-technical caveats the board must
   weigh themselves: (a) vendor track record is unproven by RHPL, and (b) references still pending.

2. **Scope disclaimer.** Technical-integration + security only. Explicitly excludes vendor
   uptime/reliability, real-world usability, configuration effort, and cost (cost is an RHPL operational
   matter handled with the Director separately, **not** in this memo).

3. **The core security fact, in plain English.** Patron **barcode + PIN must be encrypted in transit**;
   that is **non-negotiable** and is delivered the same way in both models via **stunnel** (Clarivate
   stands up the stunnel endpoint at no cost; both ends support it; no cleartext fallback inside the
   tunnel; enforced as a go-live gate with a live holds-pickup test).

4. **The two models — framed as floor vs. standard.**
   - **Model 1 — vendor-default ("it works," lower security).** Device on its own cellular line with a
     **static carrier IP** (Clarivate allowlists it). SIP2 encrypted via stunnel. **Staff HTTP portal
     left at vendor default** — no reverse proxy, no FIDO2, not locked to RHPL standards. Splashtop RMM.
     No RHPL gear, no SpeedFusion VPN, no redundancy. **Stated bluntly: this is the vendor's floor;
     it falls below the security standard RHPL applies to any system touching patron data, and RHPL
     would not operate it this way.**
   - **Model 2 — RHPL-managed, locked down (RECOMMENDED, = RHPL's baseline, not a luxury).**
     `AutoLend → Ethernet → Peplink → SpeedFusion VPN → FusionHub (our static IP) → stunnel →
     Clarivate.` SpeedFusion encrypts the whole device→RHPL path; stunnel still rides inside (defense in
     depth). The allowlisted static IP is **our FusionHub's** (cellular IPs underneath can churn). Staff
     portal reachable **only through our network, behind our reverse proxy with Google OAuth + YubiKey
     FIDO2**; never exposed to the public internet. Availability: one cellular SIM, or **two carriers
     bonded** for redundancy. **Operating Model 2 commits RHPL to being the administrator of that
     network path** (ongoing ownership/responsibility) — surfaced as the Director's decision to accept.

5. **What's confirmed (confidence anchors).** Read/search-only catalog access (no patron/item writes);
   no patron data stored on the device (fail-closed "Out of Service", offline mode off); native Polaris
   notice fires for holds (no parallel notification system); web-based staff UI / self-contained onboard
   PC means **no Windows endpoints in RHPL's Chrome OS fleet**; ADA reach/large-button/TTS (relevant to
   the Oakland Talking Book Service).

6. **Clarivate, stated correctly.** Clarivate does **not** issue us an IP — they **allowlist a static
   source IP we present** and **set up the stunnel at no cost**, and will **assist getting the tunnel
   working between our Peplink/FusionHub edge and their SIP endpoint**. Residual: generic lead time.

7. **Open items (named, not hidden).** References pending — **note we are specifically still waiting on a
   hosted-Clarivate customer reference**; vendor reliability out of our scope and the board's to weigh.

8. **Director brief (separate from board memo).** RHPL's ongoing-ownership commitment under Model 2;
   cost structure; and Derek's standing reservation (below).

## Tradeoffs (choices made, alternatives rejected)

- **Option A (purchase vote, RHPL picks the model)** over presenting Model 1/Model 2 as a board-selectable
  choice — the township board shouldn't vote on RHPL network plumbing; RHPL owns that call.
- **No dollars in the board memo** over firm cost figures — cost is RHPL operational, briefed to Juliane;
  keeps the board memo squarely technical/security and avoids quoting numbers we haven't finalized.
- **Blunt "below our standard" framing of Model 1** over neutral pros/cons — prevents the board from
  reading "Model 1 works" as "so why pay for Model 2?"; Model 2 is RHPL's baseline, not gold-plating.
- **Model 2 (RHPL gear in path)** over the earlier 6/17 "fully segmented, vendor-to-vendor" posture —
  trades hands-off segmentation for control, redundancy, and a real network context to lock down the
  HTTP staff portal. Accepted cost: RHPL becomes the network admin.
- **Recommendation not gated on references** over conditioning the technical "yes" on a reference —
  references speak to track record, which is out of technical scope.

## Risks (and mitigations)

- **R1 — RHPL silently inherits an OT operational obligation.** Model 2 = RHPL on the pager for OT's
  device link/proxy. *Mitigation:* memo states the admin-ownership commitment explicitly; it is the
  Director's decision to accept (locked Q3).
- **R2 — stunnel not actually enforced → cleartext PIN.** Polaris has no native SIP TLS; the non-stunnel
  state is cleartext. *Mitigation:* stunnel is a contractual **go-live gate** + live holds-pickup test;
  verify no plaintext SIP listener is reachable.
- **R3 — Splashtop is a vendor cloud control plane** that can fully control the Win10 box, with its own
  MFA (not our IdP), even though the data path has no vendor cloud. *Mitigation:* document it; take the
  free IT account for visibility; note it as a vendor↔vendor responsibility the board should understand.
- **R4 — Vendor track record unproven; references (esp. hosted-Clarivate) not yet in hand.**
  *Mitigation:* flagged as out-of-scope + named as a pending item; board weighs separately.
- **R5 — SIP2 dependency is a structural limitation** vs. a PAPI-only design. *Mitigation:* captured as
  Derek's explicit reservation in Scope + Director brief (not buried).
- **R6 — Board misreads scope** as a competitive "best kiosk" recommendation or as a reliability
  endorsement. *Mitigation:* scope disclaimer up front; "this vendor's product is technically workable"
  wording.

## Scope

**In:** technical integration with Clarivate-hosted Polaris 8.1; security posture (transport encryption,
staff-portal exposure, RMM, data-at-rest on device, access/auth); the Model-1-vs-Model-2 framing; the
RHPL admin-ownership commitment (named, decided by Director); plain-English security message for a
non-technical board.

**Out (this memo):** dollar costs/quotes (→ Director brief); vendor uptime/reliability/usability; OT-vs-
RHPL Polaris scoping (#9, deferred); restock cadence + owner (#10, deferred); competitive market
evaluation of other kiosk vendors.

**Standing reservation (recorded, not acted on now):** were RHPL leading a from-scratch procurement, the
preference would be a **PAPI-only** product (checkout, holds, and patron *My Account* portal all over
PAPI, no SIP2). This evaluation assesses the kiosk OT brought forward; it is not a market survey, and the
SIP2 dependency is its main structural limitation.
