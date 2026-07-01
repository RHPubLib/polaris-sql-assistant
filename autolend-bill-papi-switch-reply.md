# Reply to Bill (ILS) — accept PAPI switch, pin down "first production site," ask for test plan

*Drafted 2026-07-01. Context: Bill's 6/30 thread ("PAPI use with the ILS Autolend, and all ILS
products") where the tone pivoted from "SIP2 only / PAPI barcode lookup not possible" to "we do PAPI
in all products — this was a misunderstanding," then walked back to "we've tested PAPI transactions
but have no production sites using it." Related: [autolend-bill-bibget-v2-thread.md](autolend-bill-bibget-v2-thread.md).*

---

**Subject:** Re: PAPI use with the ILS Autolend, and all ILS products

Bill,

Thanks for clarifying — and I appreciate you responding while you're at ALA.

I want to be candid, because I think it will save us both time. We've traded roughly a dozen emails
over the last two weeks on exactly this topic, and I went back through the whole chain before writing
this. Through most of it the consistent message was that SIP2 handles the patron and material
transactions, that PAPI was used read-only for catalog/cover-art, and — on the barcode→bib lookup
specifically — that it "had not succeeded" on your sites and might be new to 8.1.x. So I'll admit I
was surprised by how much the framing shifted in the span of a single afternoon, from that position to
"we do PAPI in all our products, this was a misunderstanding." I'm not trying to assign blame; I just
want us both working from the same facts, because my recommendation to our board depends on getting
this exactly right.

Here's where I land, and what I'd need to move forward:

1. **We'll take you up on the PAPI switch.** For RHPL, all patron and material transactions on the
   AutoLend unit would run through PAPI against our (Clarivate-hosted) Polaris — not SIP2. Your note
   says you can switch "whenever we are asked," so consider this the ask.

2. **I need one thing stated plainly:** your last email said you've *tested* the patron/hold and
   charge/discharge PAPI calls but have "never had anyone in production request to use it" and have "no
   production sites using it so far." Reading that directly — **RHPL would be your first production
   deployment running AutoLend transactions over PAPI, correct?** I'm not treating that as a
   dealbreaker, but I have to represent it accurately to the board, so I need a clear yes/no rather
   than an inference.

3. **Before any commitment, I'd want a written test plan and timeline** for the PAPI transactional
   path (patron auth, holds, checkout/checkin), including what you'd validate in a test environment
   against Clarivate-hosted Polaris and roughly how long you'd expect that to take. Your barcode-search
   testing came together in about ten minutes once provisioning was added, which is encouraging — I'd
   like the same clarity on the transaction calls.

None of this changes that we've found you responsive and easy to work with. I just want the
production-readiness question answered on the record before we go further.

Thanks,

Derek Brown
Director of IT
Rochester Hills Public Library
500 Olde Towne Road, Rochester, MI 48307
Office: 248-650-7123
