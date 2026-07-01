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
