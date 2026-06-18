# Autolend evaluation — published report, access, and troubleshooting

*Saved 2026-06-18. Companion to `autolend-oakland-township-evaluation.md` (full working analysis),
`autolend-director-cover-email.md` (Juliane's 6-bullet summary), and `autolend-bill-reply-email.md`.*

## The published report (internal reports — Director Reports, managers tier)

- **What:** Full technical & security evaluation for OT, with a **Director's summary** (6 bullets) at the
  top and the **complete vendor + Clarivate dialog as an appendix** (Bill's June 18 reply verbatim).
- **Repo source:** `/var/opt/rhpl/internalreports/portal/community-survey/autolend-kiosk-evaluation/index.html`
- **Tier:** managers (`internalreports-mgt.rhpl.org`), IAP-restricted to **`managers@rhpl.org`**.
- **UNLISTED:** intentionally *not* linked from the Director Reports landing page
  (`portal/community-survey/index.html`) — reachable only by direct URL so Derek hands it to Juliane.
- **Direct link (custom domain):**
  `https://internalreports-mgt.rhpl.org/community-survey/autolend-kiosk-evaluation/`
- **Direct link (Cloud Run, bypasses the rhpl.org name):**
  `https://internal-reports-mgr-379661259922.us-central1.run.app/community-survey/autolend-kiosk-evaluation/`
- **Deployed revision:** `internal-reports-mgr-00020-dl2` (100% traffic), 2026-06-18.

## How it was published (managers tier = reauth-walled gcloud path)

1. `! gcloud auth login` (Derek, in-session — the staff/mgr tiers hit the interactive-reauth wall; only the
   it@ tier is keyless).
2. `cd /var/opt/rhpl/internalreports && bash cloud-build/assemble.sh` → builds `/tmp/ir-ctx/{staff,mgr,it}`.
3. **Inject the new report folder** (assemble only overlays the *index* page, not new subfolders):
   `cp -r portal/community-survey/autolend-kiosk-evaluation /tmp/ir-ctx/mgr/html/community-survey/`
4. `~/google-cloud-sdk/bin/gcloud run deploy internal-reports-mgr --source /tmp/ir-ctx/mgr --region
   us-central1 --iap --project rhpl-internal-reports --quiet`

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
