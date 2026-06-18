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
   that is **non-negotiable** and is delivered the same way in both models via **stunnel**. *Termination,
   stated precisely:* the **stunnel client runs on the kiosk** and connects to **Clarivate's stunnel
   server** — so SIP2 is encrypted from the moment it leaves the kiosk application; cleartext barcode/PIN
   exists only transiently **inside the kiosk's own process**, never on any wire or LAN segment. Under
   Model 2, **SpeedFusion is an independent outer transport layer** (device→FusionHub) that the already-
   stunnel-encrypted SIP2 rides inside — the two layers stack, neither replaces the other. Clarivate
   stands up its stunnel endpoint at no cost; both ends support it; no cleartext fallback inside the
   tunnel; enforced as a **go-live gate** with a live holds-pickup test and verification that **no
   plaintext SIP listener is reachable** from the kiosk.

4. **The two models — framed as floor vs. standard.**
   - **Model 1 — vendor-default ("it works," lower security).** Device on its own cellular line with a
     **static carrier IP** (Clarivate allowlists it). SIP2 encrypted via stunnel. **Staff HTTP portal
     left at vendor default** — no reverse proxy, no FIDO2, not locked to RHPL standards. Splashtop RMM.
     No RHPL gear, no SpeedFusion VPN, no redundancy. **Stated bluntly: this is the vendor's floor;
     it falls below the security standard RHPL applies to any system touching patron data, and RHPL
     would not operate it this way.**
   - **Model 2 — RHPL-managed, locked down (RECOMMENDED, = RHPL's baseline, not a luxury).**
     `AutoLend (stunnel client) → Ethernet → Peplink → SpeedFusion VPN → FusionHub (our static IP) →
     internet → Clarivate (stunnel server).` The stunnel-encrypted SIP2 rides **inside** the SpeedFusion
     tunnel (two stacked layers). The allowlisted static IP is **our FusionHub's** (cellular IPs
     underneath can churn). Availability: one cellular SIM, or **two carriers bonded** for redundancy.
     Three lockdowns that distinguish Model 2:
     - **Egress isolation (blast-radius control).** The kiosk is treated as an **untrusted device on its
       own isolated segment**; Peplink/FusionHub firewall rules restrict its egress to **only the
       Clarivate stunnel IP:port** (plus the vendor RMM endpoint) — **no route to any other RHPL internal
       system**, and the FusionHub is not a shared-trust hub. A compromised kiosk cannot pivot inward or
       abuse our trusted IP against anything but the one allowlisted Clarivate endpoint.
     - **Staff portal — network-path hardening (stated honestly).** The portal is served *locally on the
       kiosk*. Our reverse proxy + Google OAuth + YubiKey FIDO2 secures the **remote/network path** staff
       use to reach it and keeps it **off the public internet**; it does **not** secure the box against
       someone with on-device or vendor-remote (Splashtop) access — that's governed by the vendor remote
       plane (R3) and the on-device local-card admin model, not by our proxy. We do not claim the proxy
       "locks down the portal" absolutely; it removes public/internet exposure of a default-HTTP page.
     - **Operating Model 2 commits RHPL to being the administrator of that network path** (ongoing
       ownership/responsibility) — surfaced as the Director's decision to accept.

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
- **R3 — Vendor remote plane + locally-stored SIP2 creds = the sharpest residual risk.** The kiosk is a
  **vendor-managed Windows box** that must hold Polaris **SIP2 terminal credentials locally**, and the
  vendor retains full remote control via **Splashtop** (its own MFA, not our IdP). A Splashtop/vendor
  compromise therefore yields the SIP2 creds *and* a pre-authenticated path through our allowlisted IP to
  Clarivate — i.e., potential patron-record scraping over SIP2. **Neither model eliminates this; it is
  inherent to any vendor-managed appliance** (true of Model 1 too). *Mitigations:* (a) provision the SIP2
  account at **least privilege** — only the operations the kiosk needs (checkout/holds), nothing broader;
  (b) take the free IT Splashtop account for **visibility/audit** of vendor sessions; (c) the egress ACL
  (R7) bounds what a compromised box can reach; (d) **state this plainly to the board** — a vendor plane
  can control a device transacting our patron data, and that trust dependency is accepted, not hidden.
  *Honest limit:* "document it" alone is not a fix; (a)+(c)+monitoring reduce blast radius but the trust
  in the vendor's remote-access hygiene remains a real, named dependency.
- **R4 — Vendor track record unproven; references (esp. hosted-Clarivate) not yet in hand.**
  *Mitigation:* flagged as out-of-scope + named as a pending item; board weighs separately.
- **R5 — SIP2 dependency is a structural limitation** vs. a PAPI-only design. *Mitigation:* captured as
  Derek's explicit reservation in Scope + Director brief (not buried).
- **R6 — Board misreads scope** as a competitive "best kiosk" recommendation or as a reliability
  endorsement. *Mitigation:* scope disclaimer up front; "this vendor's product is technically workable"
  wording.
- **R7 — Lateral movement / VPN blast radius.** Putting an untrusted vendor Windows box on our
  SpeedFusion VPN risks pivot into RHPL internal systems or abuse of our Clarivate-allowlisted IP.
  *Mitigation:* kiosk on an **isolated segment**; Peplink/FusionHub **egress ACL** locks it to *only* the
  Clarivate stunnel endpoint (+ RMM); no route to other RHPL infrastructure; FusionHub is not shared-trust.
- **R8 — Local bypass of the reverse proxy.** The staff portal runs *on the kiosk*, so on-device or
  Splashtop access bypasses our proxy + FIDO2 entirely — the proxy only secures the network path, not the
  box. *Mitigation:* don't overstate the control (done in the memo wording); rely on the on-device
  **local-card admin model** for physical access and R3 mitigations for the vendor plane; the proxy's job
  is strictly removing **public-internet exposure** of a default-HTTP page and gating remote staff access.

**Residuals we will NOT hide from the board:** (1) the vendor remote plane (R3) can control a device that
transacts our patron data; (2) vendor reliability/track record is unproven by RHPL (R4); (3) the product
is SIP2-dependent, not PAPI-only (R5). Model 2 reduces *network* exposure and blast radius; it does not
turn a vendor-managed appliance into an RHPL-trusted endpoint, and the memo says so.

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
