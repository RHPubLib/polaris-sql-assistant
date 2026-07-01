# Email to Samantha Quell (Polaris Product Owner) — verify PAPI capabilities for the AutoLend evaluation

*Drafted 2026-07-01. Purpose: confirm with Clarivate what PAPI does/doesn't support, because the
vendor (International Library Services / "AutoLend") has told us Polaris/PAPI support said barcode
catalog search is "not currently offered" — which contradicts the PAPI docs and RHPL's live testing.
We are also making PAPI-only the contractual integration requirement, so we want Clarivate's read
before it's finalized. Sources: [autolend-bill-papi-switch-reply.md](autolend-bill-papi-switch-reply.md)
(Bill's account) and [autolend-jim-otlb-papi-verbiage.md](autolend-jim-otlb-papi-verbiage.md) (contract
language). Corroboration from Eric Young (Phoenix Public Library), who is completing a fully-PAPI
locker/vending integration with **Lyngsoe** using the same BibGetByType v2 barcode→bib call. Not sent
— awaiting Derek's review.*

---

**To:** Samantha Quell (Product Owner, Polaris — Clarivate)
**Subject:** Quick confirmation on PAPI capabilities for a hosted-Polaris kiosk integration

Sam,

Hope you're well. I could use your read as the product owner on a couple of PAPI questions, because
we're leaning on the API for a vendor evaluation and some of what we're being told doesn't quite
square with the documentation.

**The short context:** RHPL is helping a neighboring library (Oakland Township) evaluate an automated
lending/pickup-locker vendor — International Library Services ("AutoLend"). We're recommending, and
asking to put in the contract, that the device integrate with the host library's Clarivate-hosted
Polaris **exclusively over PAPI** — patron authentication (barcode + PIN), item check-out, item
check-in, hold pickup, and catalog/book content — with **no SIP2** used for any patron or material
transaction, validated end-to-end before go-live.

**Where I'd value your confirmation:** the vendor has told us that Polaris/PAPI support informed them,
more than once (as recently as last August), that searching for **catalog data by item barcode** is
"not currently offered." Because of that, their existing installs (e.g., ODIN / North Dakota, live
since Jan 2023) built a custom barcode → Bib ID lookup and then used BibGet. But our own reading of
the PAPI Revision History shows **BibGetByType v2 (barcode → Bib, `?type=barcode`) was introduced in
Polaris 7.6**, and — importantly — we've verified it ourselves, hands-on. We ran the actual PAPI calls
against RHPL's live Polaris and checked the responses against real material, covering the full path a
locker/kiosk would use: **patron authentication, item check-out, item check-in, and item→bib
matching**. Every one of them worked. The barcode→bib call in particular (BibGetByType v2, unsigned) we
exercised across **~20 items** and it returned the correct bib every time — and a deliberately invalid
barcode came back with the proper "invalid barcode" error, so it's genuinely validating input, not
just echoing. This is first-hand and reproducible on our end, not secondhand. Other sites run it in
production too (SILS is one), and patron barcode authentication (`AuthenticatePatron`) goes back to
**Polaris 5.2** — so none of this is new. That's exactly what makes the vendor's account of being told
barcode search is "not currently offered" (most recently last August) hard to square, and the gap I'd
like to close.

And it isn't only us — though you may well know this already, since you know Eric Young at **Phoenix
Public Library** as well as I do. He and **Lyngsoe** are wrapping up the last pieces of running their
lockers and vending **fully over PAPI**, including the item-record and bib-record matching calls. Eric
confirmed
you can pass an *item barcode* to PAPI and get the items tied to the same bib (BibGetByType v2 — the
very call in question), calling it "a heavy lift, but it's what Lyngsoe is using"; the one remaining
gap he flagged is that the hold-request GET needs to return the **Pickup Area**. (For completeness:
Lyngsoe's *current* model still runs the transaction side over **SIP2** from their hosted SaaS to
Innovative via a Lyngsoe/Innovative-provisioned tunnel — precisely the SIP2 dependency we're trying to
design out by requiring PAPI-only.)

Could you help me confirm three things:

1. **Barcode catalog lookup.** When did BibGetByType v2 (`?type=barcode`) become generally available,
   and is it a supported, current PAPI method? I'm trying to understand whether there was ever a
   period when "barcode catalog search is not offered" would have been accurate, versus simply crossed
   wires.

2. **Patron + circulation over PAPI.** Is PAPI fully supported for a third-party self-service/kiosk
   vendor to perform patron authentication (barcode + PIN), holds, and item check-out/check-in
   (charge/discharge) against a **Clarivate-hosted** Polaris instance? Any known limitations, or sites
   doing this in production today?

3. **Recommended path for hosted customers.** For Clarivate-hosted Polaris, is PAPI the
   supported/recommended integration path for a vendor like this (as opposed to SIP2, which for hosted
   would require Clarivate to stand up dedicated secure tunnels)? Anything we should build into the
   access provisioning or public-method authentication level (e.g., PWS signing)?

I'm not looking to put anyone on the spot — I just want the picture I hand the board to be accurate,
and to make sure our contract language matches what PAPI actually supports today. If there's someone
on the PAPI / developer-program side you'd rather I loop in, I'm glad to.

Thanks very much,

Derek Brown
Director of IT
Rochester Hills Public Library
500 Olde Towne Road, Rochester, MI 48307
Office: 248-650-7123
