# Reply to Bill (ILS) — friendly: what's possible now vs. future, PAPI is the priority

*Drafted 2026-07-01 (v2, friendlier). Context: Bill's 6/30 thread where framing shifted from "SIP2 only /
barcode lookup not possible" to "we do PAPI in all products — a misunderstanding." Softened after Derek
recalled ODIN originally *asked* for PAPI (so there may be genuine crossed wires on all sides). OT board vote
likely happened the night of 6/30. Key technical lever: Clarivate-hosted Polaris + SIP2 requires special
secure stunnel tunnels stood up by Clarivate, and CloudOps ticket replies didn't indicate ILS/AutoLend has
those — so PAPI is both the security path and the seamless-integration path RHPL already runs with other
vendors. Related: [autolend-bill-bibget-v2-thread.md](autolend-bill-bibget-v2-thread.md).*

---

**Subject:** Re: PAPI use with the ILS Autolend, and all ILS products

Bill,

No worries at all — and thanks for taking the time to write back while you're at ALA.

Honestly, re-reading everything, I think there's been some genuine crossed wires on all sides here, and
that's okay. When I spoke with the folks at ODIN it actually sounded like they'd asked for PAPI from the
start too, so I don't think anyone was trying to muddy the water — we've just been coming at it from
different angles across a couple weeks and a dozen or so emails. I only flagged it because I want to make
sure you and I are working from the same picture before I bring anything back to the table.

Where my head is now: the Oakland Township board most likely voted last night, so what I'm really trying to
sort out is simply **what's possible today versus what would be possible down the road.** Two lanes, and I'd
love your read on each.

The reason PAPI matters so much to us isn't only the security upside — it's the seamless experience. We're
already comfortable supporting several third-party vendors against our Clarivate-hosted Polaris over PAPI, so
it's a well-worn path on our end. And candidly, I have a hard time picturing many Clarivate-hosted Polaris
customers running AutoLend over SIP2 *only*, because Clarivate would have had to stand up dedicated secure
(stunnel) tunnels for each of them — and when we were trading ticket replies with Clarivate's CloudOps team,
that didn't appear to be something already in place for ILS/AutoLend. So PAPI ends up being both the cleaner
security story and the more natural fit with how we already operate.

With that in mind, a few friendly questions:

1. **Right now:** for a Clarivate-hosted Polaris site like ours, what would an AutoLend deployment realistically
   look like *today* — PAPI for patron auth, holds, and checkout/checkin, or is SIP2 still the piece that's
   actually running in production at your live sites?

2. Related to that — reading your last note, it sounds like the patron/hold and charge-discharge PAPI calls
   have been tested but aren't yet running in production anywhere. Is that right? If so, no problem at all — I'd
   just want to understand it plainly, since it'd mean RHPL would be an early (or first) production site on that
   path.

3. **Future:** if we go the full-PAPI route, what would the switch look like — a rough test plan and timeline for
   validating the transactional calls against Clarivate-hosted Polaris? Your barcode-search testing came together
   in about ten minutes once provisioning was added, which is a great sign; I'd just like the same picture for the
   transaction side.

You've been really responsive through all of this and I appreciate it — I just want to land on a clear
"here's now, here's later" so I can give our board an accurate read.

Thanks,

Derek Brown
Director of IT
Rochester Hills Public Library
500 Olde Towne Road, Rochester, MI 48307
Office: 248-650-7123
