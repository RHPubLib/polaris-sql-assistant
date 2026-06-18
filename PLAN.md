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
   tunnel; enforced as a **go-live gate** with a live holds-pickup test, verification that **no
   plaintext SIP listener is reachable** from the kiosk, and confirmation that the kiosk's stunnel client
   **validates Clarivate's server certificate** (`verifyPeer`/`verifyChain`) — encryption without cert
   verification is still MITM-able, so cert validation is part of the gate, not optional.

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
     Distinguishing properties of Model 2:
     - **Dedicated, isolated tunnel endpoint — nothing comes back to RHPL.** The SIP2 path is a
       **dedicated IP / SpeedFusion tunnel built Peplink ↔ Clarivate**; the endpoint exists *only* to
       present the allowlisted static IP and pass SIP2 to Clarivate. It is **never bridged to RHPL
       internal/production systems**, and **Clarivate (not RHPL) hosts the Polaris data.** So the device's
       only privileged destination is Clarivate — the same place our Polaris already lives, behind
       Clarivate's controls. There is **no RHPL-internal pivot surface.**
     - **The box keeps normal WAN for its own OS needs (default) — with an exfiltration tradeoff to name.**
       Per Bill ("let the network layer work normally"), by default the Windows box uses ordinary WAN for
       DNS/NTP/time and vendor-managed OS/AV updates (via RMM); only the **SIP2 session** rides the
       dedicated stunnel tunnel. **Honest consequence:** normal WAN means a compromised box can exfiltrate
       (R3) — isolation protects RHPL internal, not data confidentiality. *Optional Tier-2 layer:* since
       Model 2 already routes the box through our Peplink, we **could** apply a **default-deny egress
       allowlist** (Clarivate + Windows Update/Defender + NTP + DNS + CRL/OCSP + Splashtop) to bound
       exfiltration — but those endpoints are CDN/rotating, so it is **operationally heavy and brittle**,
       and we own the maintenance. Present it as a deliberate choice (default: normal WAN + monitor;
       hardened: egress filtering), not a free win.
     - **stunnel with certificate verification** — see §3; encryption alone isn't enough, the kiosk's
       stunnel client must validate Clarivate's server cert (`verifyPeer`/`verifyChain`) to prevent MITM.
     - **Two access surfaces, treated oppositely** (see "Two distinct access surfaces"): physical
       **loading stays simple** (role-limited loader card, no added security); the **remote admin panel is
       gated via Google** (OAuth/IAP, cloud layer, not RHPL-internal).
     - **Tier 2 is "layer to taste."** Bill confirms each layer is independently available, so RHPL can
       stack as much defense-in-depth as it wants without making the loading task harder: (1) stunnel +
       cert-verify on SIP2, (2) SpeedFusion dedicated tunnel, (3) isolated tunnel endpoint, (4) Google
       auth on the remote admin panel, (5) optional HTTPS on the staff page (our cert / self-signed),
       (6) Splashtop RMM with its own MFA + our IT account for audit. None of these touches the loader-card
       restock flow.
     - **Operating Model 2 commits RHPL to being the administrator of that connectivity** (ongoing
       ownership of the Peplink + tunnel endpoint) — surfaced as the Director's decision to accept.

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

7b. **Honest caveats + the PAPI alternative.** Include the five "concerns kept visible" (above) so the
   recommendation isn't oversold, and note **Lyngsoe** as a real PAPI-native alternative (with its own
   not-yet-proven-for-holds caveat) — evidence the protocol preference is achievable in the market and an
   argument for a fuller comparative procurement if OT wants one.

8. **Director brief (separate from board memo).** RHPL's ongoing-ownership commitment under Model 2;
   cost structure; and Derek's standing reservation (below).

## Two distinct access surfaces (resolved — Derek 2026-06-18)

The kiosk has **two different ways in**, and they get **opposite** treatment:

1. **Physical loading at the kiosk (loader card) — keep it simple, no added security.** Restocking is "put
   material inside" with **no security implications**: Bill's on-device admin uses a **role-limited loader
   card** (courier role can *only* add/remove material — cannot modify records, read patron data, or
   escalate; the card is a local dummy checked only by the box). Loading stays **low-friction — tap card,
   load, done.** We deliberately add nothing here; gating it would only hurt staff for no security gain.
2. **Remote admin control panel — gate it ourselves via Google.** This is the surface worth securing.
   RHPL puts **Google authentication in front of remote admin access** — delivered through **Google's
   cloud layer (OAuth / Identity-Aware Proxy), NOT an RHPL-internal box**, so it does **not** reintroduce
   an RHPL-internal touchpoint and stays consistent with "nothing comes back to RHPL." Two viable routes:
   (a) our own Google-fronted proxy/IAP, or (b) leveraging **Bill's existing OAuth/MFA** (built for his
   Academic sites). Implementation detail to confirm: how the Google-auth gate reaches the device's local
   admin page **without an inbound open port** (outbound connector / broker + Google auth, or via the
   vendor portal fronted by Google SSO).

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
- **R2 — stunnel not actually enforced, or enforced without cert validation → cleartext/MITM'd PIN.**
  Polaris has no native SIP TLS; the non-stunnel state is cleartext, and an stunnel that doesn't validate
  the server cert is encrypted-but-MITM-able. *Mitigation:* stunnel is a contractual **go-live gate** +
  live holds-pickup test; verify no plaintext SIP listener is reachable **and** that the client enforces
  `verifyPeer`/`verifyChain` against Clarivate's cert/CA.
- **R3 — Vendor remote plane + locally-stored SIP2 creds = the sharpest residual risk.** The kiosk is a
  **vendor-managed Windows box** that must hold Polaris **SIP2 terminal credentials locally**, and the
  vendor retains full remote control via **Splashtop** (its own MFA, not our IdP). A Splashtop/vendor
  compromise therefore yields the SIP2 creds *and* a pre-authenticated path through our allowlisted IP to
  Clarivate — i.e., potential patron-record scraping over SIP2. **Neither model eliminates this; it is
  inherent to any vendor-managed appliance** (true of Model 1 too). *Mitigations:* (a) provision the SIP2
  account at **least privilege** — only the operations the kiosk needs (checkout/holds), nothing broader;
  (b) take the free IT Splashtop account for **visibility/audit** of vendor sessions; (c) **state this
  plainly to the board** — a vendor plane can control a device transacting our patron data, and that trust
  dependency is accepted, not hidden.
  *Honest limit — do NOT overstate isolation:* the network isolation (R7) protects **RHPL internal**; it
  does **not** bound **data exfiltration**. As long as the box has a normal internet WAN and the vendor
  has remote access, a compromised box can read patron data over SIP2 and **exfiltrate it out its own
  internet link** — our SpeedFusion/Clarivate isolation does nothing against that. Bounding exfiltration
  would mean routing *all* box traffic through our Peplink under a default-deny egress allowlist (see §4
  option) — heavy and brittle. Realistic posture: **least-privilege + audit + accept the residual**, not
  "prevented." This is the single hardest thing we cannot engineer away.
- **R4 — Vendor track record unproven; references (esp. hosted-Clarivate) not yet in hand.**
  *Mitigation:* flagged as out-of-scope + named as a pending item; board weighs separately.
- **R5 — SIP2 dependency is a structural limitation** vs. a PAPI-only design. *Mitigation:* captured as
  Derek's explicit reservation in Scope + Director brief (not buried).
- **R6 — Board misreads scope** as a competitive "best kiosk" recommendation or as a reliability
  endorsement. *Mitigation:* scope disclaimer up front; "this vendor's product is technically workable"
  wording.
- **R7 — No pivot surface into RHPL internal (scoped claim — NOT a no-exfiltration claim).** The reviewer
  raised lateral movement from an untrusted vendor box on our VPN. **Into RHPL internal, this cannot
  happen by design:** the **SpeedFusion tunnel** is a dedicated path Peplink ↔ Clarivate, the endpoint is
  **dedicated and never bridged to RHPL internal/production**, and **Clarivate hosts the Polaris data, not
  RHPL.** So over the *tunnel*, a compromised kiosk reaches only Clarivate (where our data already lives,
  under Clarivate's controls), never an RHPL LAN. **Scope of this claim, stated honestly:** it covers the
  *tunnel/RHPL-internal* surface only — it does **not** mean a compromised box can't reach the internet
  generally (it has its own WAN) or can't exfiltrate (see R3). The control to hold during build: the
  tunnel endpoint stays dedicated/single-purpose and is never bridged onto an RHPL LAN.
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
SIP2 dependency is its main structural limitation. **A concrete alternative exists:** **Lyngsoe** runs all
traffic through **PAPI (no SIP2)**, is larger, and has **known Polaris customers** — but (per a secondhand
Phoenix-area Polaris source, not Lyngsoe directly) its **PAPI-for-hold-lockers is not yet in production**
(we'd be first; PAPI shipped only for its "Library of Things" line in March 2026), the browsable option
rides a **partner** locker vendor, and its **staff MFA is still in development**. So Lyngsoe proves the
PAPI path exists in the market, but for *hold-locker + PAPI* specifically it is **as unproven as ILS** —
it's an argument for a fuller comparative procurement, not a readier drop-in. (Full detail in the eval doc.)

**Concerns kept visible (memo includes these verbatim-in-spirit — see eval doc "Concerns we are keeping
visible"):** (1) load-bearing security items are *vendor assertions, not verified behavior* → make
verification a go-live gate; (2) "no PII at all" can't be literally true for a holds locker → pin down what
patron data is actually on the box; (3) vendor-managed Windows box + vendor remote SYSTEM access + local
SIP2 creds + no track record = the unfixable residual; (4) integration unproven to us + no hosted-Clarivate
reference yet; (5) SIP2 + bolted-on stunnel is fragile by construction. These are presented as honest
caveats, not as reasons the technical "yes" fails.
