# Note to Jim Kiefer (OTLB) — PAPI confirmed; one wording item before signing

*Drafted 2026-07-01. Audience: Jim Kiefer, Oakland Township Library Board (non-technical). Purpose: give the board
(1) the good news that the technical path checks out, and (2) the one thing to firm up in the ILS quote before a
purchase — with drop-in language to hand ILS. Not sent — awaiting Derek's review. Companion analysis in
`autolend-oakland-township-evaluation.md` and the internal report.*

---

**Subject:** AutoLend + Polaris — good news, and one wording item for the quote

Jim,

Quick update from the technical side, and one thing worth firming up before the board signs anything.

**The good news:** the connection this machine needs to our Polaris catalog is solid. We checked every step of a
patron's transaction — signing in with their card and PIN, checking a book out, checking one back in ("smart
returns"), pulling holds, and showing the book's info and cover — against Polaris's own documentation. **Every one
of those is supported by the secure Polaris API ("PAPI"),** and we've already run the catalog-lookup piece live
against our own system. So there's no technical roadblock to running this the secure way.

**The one thing to firm up:** ILS's quote says the machine will use PAPI "for all transaction data," which is exactly
the right direction. But as written it doesn't actually spell out the specific transactions (patron sign-in,
check-out, check-in, holds), it doesn't say SIP2 — the older, less-secure method — won't be used, and it ties
payment to *installation* rather than to a *working* integration. That matters here because **RHPL would be the
first library running these transactions live over PAPI with ILS** — so we'd want it proven before go-live, not
taken on faith.

**Suggested fix:** ask ILS to add language like this to the quote or agreement —

> *"All AutoLend transactions with the host library's Polaris system — patron authentication (barcode + PIN),
> item check-out, item check-in, hold pickup, and catalog/book content — are performed exclusively over the Polaris
> API (PAPI). No SIP2 or other protocol is used for any patron or material transaction. ILS will validate these
> transactions end-to-end against the library's Clarivate-hosted Polaris system before go-live, and final acceptance
> and payment are contingent on that successful validation."*

That one paragraph turns a good intention into a commitment and protects the township. Happy to hop on a call or
talk it through with ILS directly if that's easier.

Thanks,

Derek Brown
Director of IT
Rochester Hills Public Library
Office: 248-650-7123
