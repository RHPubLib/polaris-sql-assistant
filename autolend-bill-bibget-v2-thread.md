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
