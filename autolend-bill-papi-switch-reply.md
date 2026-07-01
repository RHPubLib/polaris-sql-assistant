# Reply to Bill (ILS) — friendly: what's possible now vs. future, PAPI is the priority — SENT 2026-07-01

*SENT 2026-07-01 (v2, friendlier; final sent text below — minor edits from draft: "Where my head is now:" prefix
dropped, item 2 dev-time ask tightened, closing folded into the "Validating across sites" paragraph, "our board"
→ "OT's board"). Context: Bill's 6/30 thread where framing shifted from "SIP2 only /
barcode lookup not possible" to "we do PAPI in all products — a misunderstanding." Softened after Derek
recalled ODIN originally *asked* for PAPI (so there may be genuine crossed wires on all sides). OT board vote
likely happened the night of 6/30. Key technical lever: Clarivate-hosted Polaris + SIP2 requires special
secure stunnel tunnels stood up by Clarivate, and CloudOps ticket replies didn't indicate ILS/AutoLend has
those — so PAPI is both the security path and the seamless-integration path RHPL already runs with other
vendors. Related: [autolend-bill-bibget-v2-thread.md](autolend-bill-bibget-v2-thread.md).*

---

**Subject:** Re: PAPI use with the ILS Autolend, and all ILS products

No worries at all, and thanks for taking the time to write back while you're at ALA.

Honestly, re-reading everything, I think there's been some genuine crossed wires on all sides here. When I
spoke with the folks at ODIN it actually sounded like they'd asked for PAPI from the start too, so I don't
think anyone was trying to muddy the water. We've just been coming at it from different angles across a couple
weeks and a dozen or so emails. I only flagged it because I want to make sure you and I are working from the
same picture.

The Oakland Township board most likely voted last night, so what I'm really trying to sort out is simply what's
possible today versus what would be possible down the road.

The reason PAPI matters so much to us isn't only the security upside; it's the seamless experience. We're
already comfortable supporting our third-party vendors against our Clarivate-hosted Polaris over PAPI. And
candidly, I have a hard time picturing many Clarivate-hosted Polaris customers running AutoLend over SIP2 only,
because Clarivate would have had to stand up dedicated secure (stunnel) tunnels for each of them. When we were
trading ticket replies with Clarivate's CloudOps team, that didn't appear to be something already in place for
ILS/AutoLend that they had seen before. So PAPI ends up being both the cleaner security fit and the more
natural fit with how we already operate.

PAPI-only onboarding: if a customer wants AutoLend running entirely on PAPI (patron auth, holds, and
checkout/checkin all through the API, with no SIP2 at all), what does bringing that customer on actually look
like on your end?

Development time: reading your last note, it sounds like the patron/hold and charge-discharge PAPI calls have
been tested but aren't yet running in production anywhere, which would make RHPL an early (maybe first)
production site on that path. No problem at all; I just want to understand it plainly. What kind of development
effort and timeline are we looking at to stand the transactional side up against Clarivate-hosted Polaris? Your
barcode-search testing came together in about ten minutes once provisioning was added, which is a great sign;
I'd just like the same picture for the transaction side.

Validating across sites: rather than proving it out at RHPL alone, would it make sense to work with a small
group of your Polaris customers who are also AutoLend customers, so we can confirm the transactional PAPI calls
hold up across the board and not just on our system? You've been really responsive through all of this and I
appreciate it; I just want to land on a clear "here's now, here's later" so I can give OT's board an accurate
read.

Thanks!

Derek Brown
Director of IT
Rochester Hills Public Library
500 Olde Towne Road, Rochester, MI 48307
Office: 248-650-7123

---

## Bill's reply (received 2026-07-01) — his answers to the three questions above

*Bill McClendon (International Library Services), replying to the email above. Captured
verbatim for the record. His numbered items 1–3 answer Derek's "PAPI-only onboarding,"
"Development time," and "Validating across sites" questions, in order.*

**From:** Bill McClendon (International Library Services)
**Date:** 2026-07-01

Derek,

Well North Dakota's installation  (ODIN) predates the release of the ability to search for catalog data by barcode on the Clarivate site PAPI documents.

NOTE: I even had this conversation with Polaris again last August when were starting installation for a different Polaris site (hosted). I again used the new site as an opportunity to communicate with the PAPI support org to ask about barcode searches.  I got "not currently offered".

So that site implemented a barcode and Bib ID setup for us, so we could lookup the barcode and get the Bib ID.  Then, we could use BibGet - and did.

ODIN has never had to deal with the AutoLend's again since installation in January of 2023.  The person assigned to the ND implementation  gave us the connection information we needed, we implemented, they have run ever since without issue.

Sure, PAPI is secure. But you can make anything secure, if you know what to do.  Its why I asked about a VPN tunnel - many sites in the last year have asked us if we support them (as I told you, we have for years) and they get setup and used when requested.

Without direct evidence, I suspect that most libraries use SIP2 because -  they always have, they know it (somewhat), they trust it, and they can get it configured and running.  API's are the future (especially web services like this) , only they all require a different codebase (library) for each vendor.  They are what all the ILS Apps run on.  I wrote the first one for Sirsi (SirsiDynix) in 2008

Regardless, we are doing this for your site.  Here are your responses below:

1 - It will look no different for you.  We will have tested and verified the PAPI calls for circulation transactions before an AutoLend is installed and implemented at your location (if you choose us) in a test environment.  We would only be left to test your items with PAPI for your instance.  That is a 30 minute task from our perspective.

If you prefer, we could arrange to test these calls before we arrive using test records in your system and using a designated item barcode or two and a designated test ID or two, so we can report on that before we ever arrive to implement. In reality, some sites are never really ready that early so we routinely test such connections and communications on-site, after we install and QC their device(s) before staff training the next day.  These two approaches are routine.

If you choose the AutoLend (or any of our devices), we will ask for 1-3 item barcodes and 1-2 patron barcodes that we can use for integration testing during our initial installation and QC.  When staff training occurs, we would request your team to bring 10 or more items to be used during that training, so that staff can practice the process of placing material into inventory inside the device(s). This staff training process takes about 30 minutes.  This is routine.

2 - Less than a week on our side.

Note: I should add that it makes no difference to our side if the ILS is hosted or not, or who is hosting it.  To date, we have not had any site (new or existing) that was Polaris/Clarivate hosted that presented a challenge beyond the initial access configuration.  I created, designed, and managed the Sirsi (later SirsiDynix) hosting setup, when I was there. I'm reasonably familiar with such implementations.

3 - We are already working on that with a consortial site with a test server setup mirroring their production system.  I'll let you know how that goes.

I am not downplaying the importance of your setup, connection, hosted system access, security concerns, security requirements, or any other aspect of same.  Please know that none of my responses should be construed in that way.  We are just that experienced with them.

---

## Open discrepancies to verify with Clarivate (→ email to Samantha Quell, Polaris Product Owner)

Bill's account has points that don't line up with the PAPI documentation; escalating to
Clarivate to confirm before the contract language is finalized:

1. **"Barcode catalog search not currently offered" (told to Bill by PAPI support as recently
   as ~Aug 2025).** Contradicts the PAPI Revision History, which introduced **BibGetByType v2
   (`?type=barcode`, barcode → Bib) in Polaris 7.6** — and RHPL has run it live/unsigned against
   production, with SILS and Phoenix also using it. Question for Clarivate: when did v2 go GA, and
   was there ever a window where "not offered" would have been accurate? (ODIN installed Jan 2023,
   so ODIN *may* genuinely predate v2 — but the repeated later "not offered" answers are the issue.)
2. **Patron barcode auth is long-standing.** `AuthenticatePatron` has supported barcode since
   **Polaris 5.2** — so patron authentication/status over PAPI is not new. Confirm PAPI is fully
   supported for a kiosk vendor's patron auth, holds, and charge/discharge on hosted Polaris.
3. **PAPI vs SIP2 for hosted customers.** Confirm PAPI is the supported/recommended path for a
   third-party self-service vendor against Clarivate-hosted Polaris (SIP2 would require Clarivate
   to stand up dedicated secure tunnels), plus any provisioning / auth-level (PWS signing) notes.

Draft email: [autolend-samantha-quell-papi-verification-email.md](autolend-samantha-quell-papi-verification-email.md).
