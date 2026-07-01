# Autolend — BibGetByType v2 (barcode→Bib) thread with Bill (ILS)

*Email chain captured for the record. Context: RHPL confirmed PAPI **BibGetByType v2**
with `?type=barcode` returns the Bib ID (Control Number / ElementID 11, off MARC 001)
directly from an item barcode — contradicting ILS's earlier claim that PAPI can't do
this and that a barcode→Bib ID dump would be required. Below are Bill McClendon's
(International Library Services) two replies and Derek's follow-up question. Related:
[autolend-bill-reply-email.md](autolend-bill-reply-email.md),
[autolend-jason-odin-email.md](autolend-jason-odin-email.md).*

---

**From:** Bill McClendon (International Library Services)

Derek,

Excellent!

Interesting, this is not on the V2 links I have for the documentation section under the Developers program.

I'll being that up with our Polaris Dev support POC.

---

**From:** Bill McClendon (International Library Services)

Derek,

I have attempted this on at least 2 of our sites and it has not succeeded yet. It appears that URL is either not enabled for them, or it really is new in 8.1.x and they have not updated yet.

We will gladly switch to this for your implementation, if you choose the AutoLend.

BTW, it is still not listed as an option on the PAPI developers swagger pages.

---

**From:** Derek Brown (RHPL)

Any idea why we can test it and it function and same with SILS (another Library system in Canada) and Phoenix uses it with their book locker system.

---

**From:** Bill McClendon (International Library Services)
**Date:** Tue, Jun 30, 2026, 6:03 PM
**To:** Derek Brown; Jason

Derek,

Yes, we have used SIP2 for all material transactions.  But we can switch to PAPI whenever we are asked. We only need to test before production implementation.  I didn't mean to imply we couldn't do it.

I was talking about the Catalog record content requests. After your email response about barcode searches now working, we immediately began testing it for Lake Forest, North Dakota, and several other Polaris sites.  Testing was successfully completed within 10 minutes of getting the V2 barcode search access added for the sites in the PAPI provisioning.

We have already reached out to them to ask them to pick a time to be implemented on their devices.

NOTE: these notes are for the Barcode search to return catalog (MARC) data I communicated to you, to which you responded.

We have tested the PAPI patron, hold, and most of the other calls previously, and still do when a new release is identified.  We have never had anyone in production request to use it for charge or discharge transactions before.

It's a bit like NCIP.  We support it, and have since 2012.   But we don't have any production sites using it so far.

---

## Finding (2026-06-22) — it's NOT new to 8.1

Bill's "new in 8.1.x" theory doesn't hold up. Evidence:

- **Shipped in Polaris 7.6.** III's PAPI Revision History
  (`docs/papi-reference/PAPIRevisionHist.md`) lists BibGetByType as split into v1/v2 in
  **Polaris 7.6** — same release as the v2 PatronRegistration endpoints. The endpoint doc page
  is footered "Polaris Version 8.0." Not bleeding-edge.
- **IS in Swagger — under a separate "Polaris API 2.0" definition.** *(Corrected 2026-06-22 — an
  earlier note here wrongly said v2 was absent from Swagger; that was based on fetching only the v1
  definition.)* The Swagger UI loads two documents via the top-right **"Select a definition"
  dropdown**: `/swagger/v1/swagger.json` (title "Polaris API", 128 paths, all v1) and
  `/swagger/v2/swagger.json` (title "Polaris API 2.0"), which contains the v2 calls:
  `GET .../public/v2/.../bib/{Key}` **with the `type` query param** (= BibGetByType v2), plus v2
  patron POST/GET/PATCH. So `?type=barcode` IS documented — you just have to switch the dropdown
  from the default "Polaris API" (v1) to "Polaris API 2.0". Bill likely missed it on the default v1
  view, or his older sites don't generate the v2 definition.
- **Why it likely looked absent / failed for Bill:** if you stay on the default "Polaris API" (v1)
  definition, the only bib endpoint shown is `/public/v1/.../bib/{BibID}` (expects a Bib ID, no
  `?type=barcode`) — feed a barcode to that and it fails, looking like "URL not enabled." The v2
  call is `/public/v2/{LangID}/{AppID}/{OrgID}/bib/{barcode}?type=barcode` (visible once you switch
  the Swagger definition dropdown to "Polaris API 2.0"). If his older sites don't even list the v2
  definition, that's a build-age thing, not a missing method. Secondary check: PAPI public-method
  auth level — doc says auth required if set to ALL.
- **Multi-site confirmation:** Scott (SILS) was using this exact call at the **IUG 2024 hackathon
  in Denver**; the hackathon group routinely exercises v2 calls not in Swagger. Phoenix uses it
  in their locker system. RHPL, SILS, Phoenix → live and stable across multiple sites.

---

## Live verification (2026-06-22) — real call + result

Ran against the production catalog, **unsigned** (RHPL's PAPI public-method auth level is open;
no `PWS` header needed):

```
GET https://catalog.rhpl.org/PAPIService/REST/public/v2/1033/100/1/bib/33158013187361?type=barcode
```

Item barcode `33158013187361` = a copy of *The Hobbit: A Graphic Novel*. Response HTTP 200, 28 rows:

```
ElementID 11  Control Number:  1099993   ← Bib ID, returned straight from the item barcode
ElementID 35  Title:           Hobbit : a graphic novel of the fantasy classic
ElementID 18  Author:          Tolkien, J. R. R. (John Ronald Reuel), 1892-1973
ElementID 13  Call Number:     HOBBIT
```

Cross-checks: BibKeywordSearch (TI=hobbit) independently returns Control Number `1099993` for that
title, and `/bib/1099993/holdings` lists barcode `33158013187361` — so the barcode→Bib mapping is
confirmed both directions. A deliberately-invalid barcode returns `PAPIErrorCode -1000 "Invalid
barcode supplied"` (HTTP 200), confirming the v2 route is live and validating input.

**Note for the reply:** since RHPL serves this unsigned, sites where it "fails" may simply have the
PAPI public-method authentication level set to ALL (doc: *"Authorization Required? Yes, if
authentication level set to ALL"*), which would require a signed `PWS` header.

---

**Reply drafted to Bill 2026-06-22** (in Derek's Gmail drafts; subject *Re: Autolend —
BibGetByType v2 (barcode→Bib)*): conveys the 7.6 revision-history citation, the Swagger-omits-v2
point, the v1-vs-v2 URL gotcha, the SILS/Phoenix/hackathon corroboration, and the auth-level
check — closing that the barcode→Bib lookup works cleanly so no barcode→Bib ID dump is needed.

---

## Eric Young (Phoenix Public Library) — vendor-side corroboration (captured 2026-07-01)

*Eric Young at Phoenix PL, who runs Lyngsoe lockers on Polaris, on the barcode→Bib PAPI call and
Lyngsoe's move to PAPI. Verbatim, for the record. Confirms BibGetByType v2 (item barcode → items on
the same bib) is real and in use by a vendor, and that Lyngsoe's "SideEvent V2" now supports PAPI.*

**From:** Eric Young (Phoenix Public Library)

> "Sorry, got tied up, you can pass an item barcode with PAPI and get all items tied to the same bib 😛
> It is a heavy lift but it is what Lyngsoe [is] using...
> https://knowledge.ag-software.clarivate.com/polaris/PAPI/PAPIService/PAPIServiceBibGetByType_v2.htm"

> "Yeah control number is BibID, but you can also pass the item barcode and get details... just need
> the hold request get to include Pickup Area."

**From:** Eric Young (Phoenix Public Library) — follow-up (Lyngsoe SideEvent V2 / PAPI status)

> "We are testing the latest version of Lyngsoe's 'SideEvent Version 2' solution that supports PAPI
> now. I have a test locker if you want I can jump on a Teams meeting and give you a tour.
>
> Yes, they offer browsable lending options. To my knowledge, Lyngsoe offers two distinct hardware
> options. The model we have is designed exclusively for hold lockers. The other option, provided by a
> different physical locker vendor, supports what Lyngsoe refers to as 'Library of Things' lending.
> This system enables staff to stage items within the locker for browsable checkouts, allowing patrons
> to select and check out items directly. While I'm not entirely sure about the specifics, such as
> whether it uses glass doors, I have read that it allows for merging doors to create flexible locker
> sizes. Although I haven't seen this hardware firsthand, I recently began testing the new SideEvent V2
> software, which supports both LoT and Hold lockers.
>
> According to the developer, several Library of Things customers use only PAPI, but we would be the
> first to use it for Hold lockers. PAPI support was launched for those LoT customers just this past
> March, and I'm hoping our hold lockers will transition to PAPI soon. I was given an in-person demo of
> the PAPI support from Lyngsoe back in Feb 2026.
>
> Regarding staff access, the current setup uses local accounts hosted by Lyngsoe. When I met with them
> yesterday, I emphasized the need for MFA, and they assured me it is in development and should be
> available soon. I even volunteered to partner with them for setup and testing."
>
> — Thank you, Eric Young

**Why this matters for RHPL:** independent, vendor-side confirmation that (a) the barcode→Bib PAPI call
(BibGetByType v2) is real and used by Lyngsoe, (b) Lyngsoe's SideEvent V2 supports PAPI for both LoT
and Hold lockers, (c) LoT customers have run **PAPI-only since ~March 2026**, and (d) the only gap Eric
noted is the hold-request GET needing **Pickup Area**. Directly contradicts ILS's "barcode search not
currently offered" account. Feeds the Samantha Quell email
([autolend-samantha-quell-papi-verification-email.md](autolend-samantha-quell-papi-verification-email.md)).
Separate track (not a PAPI/catalog issue): Lyngsoe staff access is local accounts hosted by Lyngsoe —
Eric pushed them for **MFA** (in development).
