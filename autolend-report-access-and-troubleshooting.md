# Autolend evaluation — published report, access, and troubleshooting

*Saved 2026-06-18. Companion to `autolend-oakland-township-evaluation.md` (full working analysis),
`autolend-director-cover-email.md` (Juliane's 6-bullet summary), and `autolend-bill-reply-email.md`.*

## The published report (internal reports — IT Internal tier, moved here 2026-07-01)

- **What:** Full technical & security evaluation for OT, with a **Director's summary** at the top and the
  **complete vendor + Clarivate dialog as an appendix**.
- **Repo source:** `/var/opt/rhpl/internalreports/portal/it-internal/autolend-kiosk-evaluation/index.html`
- **Tier:** IT Internal (`internalreports-it.rhpl.org`), IAP-restricted to **`it@rhpl.org`** — **plus**
  `juliane.morian@rhpl.org` as a per-service one-off grant (added 2026-06-15), so the Director keeps access.
- **LISTED:** now linked from the IT Internal Reports index (`portal/it-internal/index.html`, under
  *Infrastructure & Devices*). (On the old managers-tier home it was deliberately unlisted.)
- **Direct link:** `https://internalreports-it.rhpl.org/autolend-kiosk-evaluation/`
- **Deployed revision:** `internal-reports-it-00037-czc`, 2026-07-01.

> **Location history:** originally published on the **managers** tier at
> `…-mgt.rhpl.org/community-survey/autolend-kiosk-evaluation/` (2026-06-18, unlisted); briefly re-filed to
> the managers **IT Reports** section (`…-mgt.rhpl.org/it-reports/…`) on 2026-07-01; then moved to the **IT
> Internal** tier (above) the same day. Both managers-tier URLs now **404** (stale copies were purged from the
> mgr run-sources base archive so they don't linger).

## How it is published now (IT tier = KEYLESS, no reauth)

`cd /var/opt/rhpl/internalreports && python3 cloud-build/deploy_it_via_cloudbuild.py` — tars the repo, submits
`cloud-build/it.cloudbuild.yaml` to Cloud Build impersonating `report-deployer@` via local ADC; assembles the
it context from `portal/it-internal/` and deploys `internal-reports-it` entirely in GCP. **No gcloud CLI, so
no interactive-reauth wall.** The it tier is fully repo-authored, so committing/staging the report + its index
card in the repo is all that's needed.

## Access troubleshooting (observed 2026-06-18)

**"It just sits spinning" (as derek.brown@rhpl.org):** the new report path returns the **exact same**
`HTTP 302 → accounts.google.com` (IAP sign-in) as the existing, working Director Reports — so the deploy is
healthy; the stall is in the **IAP/Google sign-in**, not the page. Usual causes/fixes: multiple Google
accounts in the browser (use Incognito, sign in only as `…@rhpl.org`); stale IAP cookie; confirm membership
in `managers@rhpl.org`.

**"The site isn't secure":** the cert is genuinely valid — verified from outside:
- DNS: `internalreports-mgt.rhpl.org` → `ghs.googlehosted.com` (Google), same as the working staff domain.
- Cert: `CN=internalreports-mgt.rhpl.org`, issuer **Google Trust Services (WR3)**, valid **Jun 4 → Sep 2,
  2026**, hostname matches.
- Conclusion: the warning is **client-side** (matches the known `*.rhpl.org`-from-Chromebook quirk), not a
  server cert problem. Diagnose by the Chrome error code:
  - `NET::ERR_CERT_DATE_INVALID` → device **clock** skew (most common).
  - `NET::ERR_CERT_AUTHORITY_INVALID` → network/SSL-inspection re-signing with an untrusted CA.
  - `NET::ERR_CERT_COMMON_NAME_INVALID` → internal DNS resolving the name to the wrong host.
- **Fallback that bypasses the rhpl.org name entirely:** use the **run.app URL** above. If that loads, the
  issue is the rhpl.org name on Derek's network/Chromebook; if it also fails, it's the device (clock/CA).

**Optional one-off IAP grant (keyless, no reauth):** if `juliane.morian@` or `derek.brown@` aren't in
`managers@rhpl.org`, add them directly to the `internal-reports-mgr` IAP policy via derek@ ADC + the IAP
REST API (same pattern as the it-tier one-off in `/var/opt/rhpl/CLAUDE.md`).

## Recommendation as published (Derek's real call, after talking to Juliane)

Technically feasible and securable (Model 2), **but recommend holding ~one month** to obtain references
(especially a **hosted-Clarivate** customer to speak to) and compare a couple of other vendors (incl.
PAPI-native **Lyngsoe**) before committing — because the whole picture rests on the vendor's word with no
references/customers yet. A timing recommendation, **not** a verdict against the product.
