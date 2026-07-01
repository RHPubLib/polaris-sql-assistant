# Note to Jim Kiefer (OTLB) — verbiage options + our offer to help

*Drafted 2026-07-01 (rev 2, after Derek & Jim's ~1-hr phone call; Jim asked Derek to look over the quote verbiage).
Audience: Jim Kiefer, Oakland Township Library Board (non-technical). Purpose: give Jim the two ways he can go on the
PAPI wording so the board can weigh them, plus RHPL's offer to assist ILS directly. Not sent — awaiting Derek's review.
Companion analysis: `autolend-oakland-township-evaluation.md` and the IT internal report.*

---

**Subject:** AutoLend quote — the PAPI wording, two ways to go

Jim,

Good talking with you. As promised, I looked over the PAPI language in the ILS quote. Short version: the technical
path is solid — we checked every step of a transaction (patron sign-in, check-out, check-in, holds, and pulling book
info and cover art) against Polaris's own documentation, and every one is supported over the secure Polaris API
("PAPI"). We've even run the catalog piece live against our own system. So there's no technical roadblock. The one
piece we'll want to prove hands-on during setup is holds — a pickup locker has its own workflow — which is exactly
what the end-to-end test in Option 2 is for.

The only open question is how tightly you want the *quote* to commit to it. Here are the two ways you can go:

**Option 1 — Accept the quote as written.** It already says the machine will use PAPI "for all transaction data,"
which is the right direction. The catch: it doesn't spell out the specific transactions (sign-in, check-out,
check-in, holds), it doesn't say the older, less-secure "SIP2" method won't be used, and it ties payment to
*installation* rather than to a *working* system. Workable, but it leaves a little to trust.

**Option 2 — Ask ILS to add one explicit paragraph** (my recommendation). It names each transaction, rules out
SIP2, and ties final acceptance/payment to a verified working integration — which matters because RHPL would be the
first library running these transactions live over PAPI with ILS. Suggested language to hand them:

> *"All AutoLend transactions with the host library's Polaris system — patron authentication (barcode + PIN), item
> check-out, item check-in, hold pickup, and catalog/book content — are performed exclusively over the Polaris API
> (PAPI). No SIP2 or other protocol is used for any patron or material transaction. ILS will validate these
> transactions end-to-end against the library's Clarivate-hosted Polaris system before go-live, and final acceptance
> and payment are contingent on that successful validation."*

Either option can work — it's really a question of how much you want nailed down on paper up front.

One more thing I want to put on the table: **RHPL is more than happy to help make this work, whatever form that
takes.** We can work with ILS directly on the PAPI setup, help coordinate with Clarivate support on our end, and do
the technical legwork — whatever gets to the best result and a smooth experience for your patrons. We're invested in
this going well, not just in reviewing it.

Happy to jump on a call with ILS together, or talk any of this through whenever works for you.

Thanks,

Derek Brown
Director of IT
Rochester Hills Public Library
Office: 248-650-7123
