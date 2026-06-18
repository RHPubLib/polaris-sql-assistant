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
touchpoint → kicked to an **OPEN DECISION** (B1 on-device+vendor-portal recommended; B2 DMZ-proxy; B3
device IP-restrict). Confirming B1 with Derek before resuming the loop.
