# Email to Samantha Quell (Polaris Product Mgmt, Clarivate) — PAPI capability sanity-check

*Drafted 2026-07-01. Purpose: get Sam's expert read on why a vendor (International Library Services /
"AutoLend") was repeatedly told by Polaris/PAPI support that barcode catalog search "isn't offered,"
when RHPL has verified it first-hand and Phoenix/Lyngsoe are doing it in production. Two pointed
questions: (1) are we missing something / why would Polaris say it can't be done; (2) why would ILS
believe PAPI patron verification wasn't supported when it's been out since 5.2 — is that somehow not
surfaced to vendors on the Clarivate side. Tone: friendly/casual — Derek & Sam have an established
joking rapport (per prior Gmail threads); Derek is coming to her for her expertise. Jokes left for
Derek to add. Includes the verbatim quote from Bill's last email. Sources:
[autolend-bill-papi-switch-reply.md](autolend-bill-papi-switch-reply.md),
[autolend-bill-bibget-v2-thread.md](autolend-bill-bibget-v2-thread.md) (our live call + result),
[autolend-jim-otlb-papi-verbiage.md](autolend-jim-otlb-papi-verbiage.md) (contract language). Not sent
— awaiting Derek's review.*

---

**To:** Samantha Quell <Samantha.Quell@clarivate.com>
**Subject:** A PAPI head-scratcher — picking your brain

Sam,

I'm coming to you on this one because you know PAPI and where Polaris is headed better than just about
anyone, and I've got a genuine head-scratcher I can't reconcile.

Quick background: RHPL is helping a neighboring library (Oakland Township) evaluate an automated
lending/locker vendor — International Library Services ("AutoLend"). We're recommending, and want in
the contract, that the device talk to the host library's Clarivate-hosted Polaris **exclusively over
PAPI** — patron authentication (barcode + PIN), check-out, check-in, hold pickup, and catalog/book
content — with **no SIP2** for any patron or material transaction.

Here's where I'm stuck. The vendor tells us that Polaris/PAPI support told them — more than once, as
recently as last August — that searching for catalog data by item barcode simply isn't offered. In
his words:

> *"I even had this conversation with Polaris again last August when [we] were starting installation
> for a different Polaris site (hosted). I again used the new site as an opportunity to communicate
> with the PAPI support org to ask about barcode searches. I got 'not currently offered'. So that site
> implemented a barcode and Bib ID setup for us, so we could lookup the barcode and get the Bib ID.
> Then, we could use BibGet — and did."*

That just doesn't match what I'm seeing. We ran the calls ourselves against our live Polaris and
verified the whole path a locker would use — patron auth, check-out, check-in, and item→bib matching —
and it all works. The barcode→bib call in particular (BibGetByType v2, `?type=barcode`) we exercised
across ~20 items and it returned the right bib every time; an invalid barcode even came back with the
proper error, so it's genuinely validating input, not just echoing. And you almost certainly already
know Eric Young at Phoenix Public Library — he and Lyngsoe are wrapping up a fully-PAPI locker/vending
setup using that exact barcode→bib call. So this is clearly live and in production, not theoretical.

So my two real questions for you:

**1. Are we missing something?** Is there any reason Polaris would tell a vendor that barcode catalog
search (BibGetByType v2) can't be done — some licensing, hosted-instance provisioning, version, or
public-method auth-level caveat I'm not seeing — when Phoenix is doing it with a vendor and I can
reproduce it with our own PAPI calls? I'd genuinely rather find out I'm wrong now than after it's in a
contract.

**2. Why would ILS's team believe the capability wasn't there?** Patron verification/authentication
over PAPI (`AuthenticatePatron`) has been around for ages — since 5.2 — so it's odd that a vendor would
think it wasn't supported. Is that something that gets downplayed, or just doesn't get surfaced, when
your support / dev-program folks talk to third-party vendors? I'm trying to figure out whether this is
crossed wires on their end, or something on the Clarivate side that quietly steers vendors toward SIP2.

No fire drill at all — I just want the picture I hand the board to be accurate, and to make sure the
PAPI-only contract language actually matches what Polaris supports today. If there's someone on the
PAPI / dev-program side you'd rather I bug instead, point me their way. And if it's easier to just talk
it through, I'm around.

Thanks Sam,

Derek Brown
Director of IT
Rochester Hills Public Library
500 Olde Towne Road, Rochester, MI 48307
Office: 248-650-7123
