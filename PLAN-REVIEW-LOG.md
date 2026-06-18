# PLAN-REVIEW-LOG — Autolend board/director memo

## Act 1 — complete (2026-06-18)

Interview resolved 6 branches with the user (Derek Brown):
1. **Decision shape:** Option A — OTLB votes purchase yes/no; RHPL delivers a feasibility+security memo
   and picks the network model. Model 1 is contrast only.
2. **Costs:** out of the board memo (RHPL operational; Director brief only).
3. **RHPL ownership:** Model 2 recommended; memo states RHPL becomes the **administrator of that network
   path** (ongoing ownership) — Director's decision to accept.
4. **Clarivate fact:** Clarivate allowlists *our* static IP (FusionHub/carrier) + free stunnel setup; the
   IP is ours to provide.
5. **References:** technical rec not gated on them; note pending, especially a **hosted-Clarivate** ref;
   reliability out of scope.
6. **Tone on Model 1:** blunt — "below RHPL standard, we wouldn't run it that way"; Model 2 = baseline,
   not luxury; plus plain-English "PIN must be encrypted, non-negotiable."

Plan written to `PLAN.md`.

**Config:** REVIEWERS = gemini,local · GEMINI_MODEL = gemini-3.1-pro-preview (global, project
rhpl-vertex-sandbox) · LOCAL = 127.0.0.1:8000 · MAX_ROUNDS = 5.

---

## Act 2 — adversarial review

### Round 1 — Gemini: REVISE · local: APPROVED

**Gemini objections (all valid, all incorporated):**
1. **Splashtop + locally-stored SIP2 creds = remote backdoor.** Vendor has remote SYSTEM access; the
   Win10 box must store SIP2 terminal creds locally. Splashtop compromise → SIP2 creds + pre-authed path
   to Clarivate → patron scrape via SIP2. "Document it" is not a mitigation.
2. **VPN blast radius / lateral movement.** Untrusted vendor Win box on our SpeedFusion VPN; plan omits
   egress ACLs. Compromised kiosk could pivot into RHPL internal or hit Clarivate via our trusted IP.
3. **Reverse-proxy "security theater."** Portal runs locally on the kiosk; anyone with Splashtop/localhost
   access bypasses our proxy + FIDO2. Proxy secures *our path* to the portal, not the portal itself —
   FIDO2 claim overstated.
4. **stunnel termination ambiguity.** `FusionHub→stunnel→Clarivate` wording leaves cleartext-PIN exposure
   undefined; must pin down where stunnel terminates.

**Local (Qwen):** bare `VERDICT: APPROVED`, no content — low signal; deferred to Gemini's conservative reading.

**Revisions made:** rewrote the core-security-fact to specify **stunnel client on the kiosk → Clarivate
stunnel server** (cleartext SIP2 exists only transiently inside the kiosk app, never on a wire; SpeedFusion
is an independent outer layer). Reframed the Model 2 staff-portal claim to "secures the **network path**,
not the box." Upgraded R3 (Splashtop) to real mitigations + honest residual. Added R7 (egress isolation /
blast radius) and R8 (local proxy bypass). Added a "residuals we will not hide from the board" note.

### Round 2 — Gemini: REVISE · local: APPROVED

**Gemini objections (all valid operational fixes, incorporated):**
1. **Egress ACL over-constrained → breaks the box.** "Only Clarivate + RMM" blocks **NTP** (stunnel/TLS
   needs accurate time), **DNS** (resolving Clarivate/Splashtop), and **Windows Update/Defender/CRL-OCSP**.
   The allowlist must include foundational OS/time/name/update/revocation services; the *deny* target is
   lateral access to RHPL internal, not all internet.
2. **Reverse-proxy routing contradicts "no route to RHPL internal."** Needed to define the ingress path:
   isolation constrains **kiosk-initiated** egress; RHPL-initiated admin/proxy access to the portal over
   the SpeedFusion management tunnel is a *separate, tightly-scoped* inbound ACL (only the proxy host may
   reach the portal port).
3. **stunnel MITM.** Must mandate **certificate verification** (`verifyPeer`/`verifyChain` vs. Clarivate's
   cert/CA) in the go-live gate — encryption without cert validation is MITM-able.

**Local (Qwen):** bare `VERDICT: APPROVED` again — low signal.

**Revisions made (reconciled with Derek's correction below):** added **stunnel cert-verification** to the
go-live gate (Approach §3 + R2). Reframed the egress story: the box keeps **normal cellular WAN** for its
own OS/time/name/update needs (per Bill, "let the network layer work normally"); only the **SIP2 session**
rides the dedicated stunnel tunnel — so Gemini's "ACL starves the OS" concern is avoided by *not* imposing
a single restrictive ACL on the box. The reverse-proxy routing contradiction (Gemini #2) is superseded by
Derek's correction → became the staff-portal OPEN DECISION.

### Derek correction (2026-06-18, mid-Act-2)

> "This won't EVER come back to RHPL. This will be a dedicated IP between the Peplink and Clarivate to
> build the tunnel. Clarivate hosts our Polaris data and not us."

**Effect:** R7 reframed from "lateral-movement risk" to a **design property** — dedicated isolated tunnel
endpoint (Peplink↔Clarivate), never bridged to RHPL internal, Clarivate (not RHPL) hosts the data → **no
RHPL-internal pivot surface.** This *satisfies* the kernel of Gemini's R1 concern (isolation) by design.
**Side effect:** the earlier RHPL-reverse-proxy/FIDO2 staff-portal idea would reintroduce an RHPL-internal
touchpoint → kicked to an OPEN DECISION.

### Derek clarification #2 (2026-06-18) — two access surfaces; Tier 2 = layer to taste

Resolved the OPEN DECISION as a **hybrid by access surface**, not one global choice:
- **Physical loading (loader card): keep simple, no added security** — role-limited courier card, local-
  only, can't touch records/patron data. Don't gate it.
- **Remote admin panel: gate via Google** (OAuth/IAP, **cloud layer, not RHPL-internal** → still "nothing
  back to RHPL"); routes: our Google-fronted proxy *or* Bill's existing Academic OAuth/MFA. Open impl
  detail: how the Google gate reaches the device with **no inbound open port**.
- **Tier 2 (= Model 2) is explicitly "layer to taste"** — Bill confirms each layer is independent; RHPL
  can stack stunnel+cert / SpeedFusion / isolated endpoint / Google-auth admin / optional HTTPS / RMM-MFA
  without making restock harder.

Plan updated (replaced OPEN DECISION with "Two distinct access surfaces"; added layering list to Model 2).
Adversarial loop **paused at Derek's request** to discuss Bill's details / Tier 2 before resuming.

### Derek additions #3 (2026-06-18) — keep concerns 1–5; add Lyngsoe (PAPI alternative)

- **Concerns 1–5 stay in the write-up** (vendor-assertion-not-verified; "no PII" implausible for a holds
  locker; vendor remote-plane residual; unproven-to-us + no hosted-Clarivate ref; SIP2+stunnel fragility).
  Added as "Concerns we are keeping visible" in the eval doc; summarized in PLAN scope + Approach §7b.
- **Lyngsoe** added as a concrete **PAPI-native (no SIP2)** alternative — larger vendor, known Polaris
  customers. Caveats (secondhand via a Phoenix-area Polaris library, not Lyngsoe directly): hold-locker
  PAPI **not yet in production** (we'd be first; PAPI shipped only for "Library of Things" line, Mar 2026),
  browsable option via a **partner** vendor, **staff MFA still in development**. Framed as "PAPI path
  exists in the market" → argues for fuller comparative procurement, NOT a readier drop-in. Added as eval-
  doc section + PLAN standing-reservation extension.

Loop still paused; material design + content changes since Round 2 → a fresh adversarial round is warranted
when Derek says go.

### Round 3 — Gemini only (Derek: "just run it by Gemini"): REVISE — 2 valid findings

1. **Egress/exfiltration self-contradiction.** §4 ("box keeps normal WAN") vs R3/R7 ("egress ACL bounds
   what a compromised box can reach"). With normal WAN, a compromised box can scrape via SIP2 and
   exfiltrate out its own internet link → isolation protects RHPL-internal only, NOT data confidentiality.
   **Fixed:** removed the false bound from R3; rescoped R7 to "no RHPL-internal pivot" (explicitly not a
   no-exfil claim); added the default-deny-egress-allowlist as an *optional, heavy/brittle* Tier-2 layer in
   §4 with the tradeoff named.
2. **Unverified Google-IAP control ("vaporware").** Promised board a Google-gated admin panel but buried
   the unverified dependency (does the vendor permit an outbound broker agent on their box?). **Fixed:**
   downgraded to TARGET-pending-verification; named RHPL's real pattern (Cloud Run + IAP fronts an
   RHPL-hosted app, not a device-local page → needs the broker); stated the realistic fallback (Splashtop +
   on-device admin) if neither broker nor Bill's OAuth is permitted.

### Project survey (localai /var/opt/rhpl) — RHPL real patterns folded in

- **Wildcard cert:** RHPL holds a `*.rhpl.org` cert (nginx, GoDaddy) → staff-page HTTPS is trivial. Added.
- **Staff-auth pattern:** Cloud Run + IAP by Google group (internal-reports it@/managers@/domain) — used to
  sharpen the admin-gate section + its feasibility caveat.
- **Peplink/SpeedFusion/FusionHub** confirmed in production (bookmobile, kids' bus) — Model 2 is genuinely
  RHPL-proven, not theoretical.
- **PAPI** = HMAC-SHA1 "PWS" signing; RHPL PAPI use is read-only, patron/item writes limited → reinforces
  that PAPI-for-holds is new across the board (added nuance to the Lyngsoe section).
- **Secrets:** `.env`+gitignore / keyless ADC; OS pattern is "never expose unauthenticated admin to the net."

### Derek framing #4 (2026-06-18)

> "I'm not saying to OT to NOT pick this branded locker. I just want to be honest about what we've pulled
> so far and it's all based on what a vendor is telling us... We are taking them at their word."

Added a **Basis-of-confidence statement** to PLAN (Goal) and the eval doc (Concerns intro): all findings are
vendor-sourced, unverified by third parties/references; we take the vendor at his word; **explicitly NOT a
recommendation against the product** — the technical "yes" is "yes, per the vendor, pending verification,"
remedied by a proof-of-integration go-live test + references.
