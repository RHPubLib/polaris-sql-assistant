# Email to Jason (ODIN) — Autolend reference / gut-check — draft

*Draft for Derek to send. Tone: peer-to-peer, candid, IUG-colleague familiar. Two jobs in one note:
(1) lay out my honest worries about the product/vendor, and (2) ask Jason how ODIN actually finds it in
production. Jason = ODIN (Online Dakota Information Network), Polaris consortium; Derek knows him well
from IUG. Product = Autolend holds-locker kiosk from International Library Services (ILS), Bill McClendon CTO.*

---

**To:** Jason <…@odin…>
**Subject:** Picking your brain on the Autolend lockers

Jason,

Hope you're doing well — overdue for a real catch-up since the last IUG.

I'm reaching out because I learned ODIN is running the **Autolend** holds lockers from International
Library Services, and we (Rochester Hills, near Detroit) are evaluating one for a township we serve.
Our board is weighing it, and I'd trust your read on this far more than anything I can get from a sales
thread — so I wanted to come to you straight, both with what's making me hesitate and a few honest
questions.

**Where my head's at — the worries.** A few things have been nagging at me:

- It's been hard to get any **customer references**. The vendor's been responsive and candid otherwise,
  but I still don't have a list of Polaris libraries actually running this that I can call — I found out
  ODIN uses it on my own, not from them. That alone makes me want to talk to a real user.
- So far **every assurance is the vendor describing their own product** — no independent site to confirm
  it holds up day to day.
- And one specific thing gave me pause on how deep their Polaris integration knowledge runs: I was told
  flatly that **Polaris/PAPI can't return a Bib record from an item barcode** — that you'd have to feed
  them a barcode→Bib ID dump. But when I tested it against our own catalog, the PAPI **BibGetByType v2**
  call with `?type=barcode` handed back the Bib ID just fine (it comes through as the Control Number /
  ElementID 11, off the MARC 001). It works cleanly for us. I may be missing a wrinkle on their end, but
  it left me wondering how current their PAPI knowledge is versus leaning on SIP2 for everything.

None of that is a verdict — the product may be perfectly solid — but it's exactly the kind of thing a
real user can settle in five minutes.

**So, if you have a few minutes, what I'd love your honest take on:**

- **The company** — how's International Library Services to deal with? Responsive, straight with you,
  good on support after the sale?
- **The product** — how's the Autolend unit actually performing? Reliability, patron experience, anything
  that's bitten you?
- **Polaris integration** — how well does it really tie into Polaris in practice? Holds routing,
  notifications, check-in/inventory — does it behave the way you'd want?
- **Your Polaris hosting** — is ODIN **self-hosted, or hosted by Clarivate?** (I can't remember which
  you landed on, and it matters for how closely your setup mirrors ours — we're Clarivate-hosted.)
- **SIP2 vs. PAPI** — does the locker talk to you over **SIP2 only, or a mix of SIP2 and PAPI?** And if
  you hit that barcode→Bib lookup limitation, how did you end up handling item lookup?

Anything else you'd want to know if you were in my seat, I'm all ears — good or bad. I'd genuinely rather
hear the warts now than after a board vote.

Thanks, Jason — really appreciate it. Happy to return the favor anytime, and let's grab time at the next
IUG regardless.

Best,
Derek

Derek Brown
Director of IT, Rochester Hills Public Library
</content>
</invoke>
