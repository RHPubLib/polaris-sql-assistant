# Abbreviated summary — Autolend kiosk (for Juliane)

*Draft for Derek. Updated 2026-06-30 after the first reference (Jason Bedsaul / ODIN–North Dakota) replied
and after the PAPI/BibGet finding firmed up. Supersedes the 6/18 six-bullet version. Full report (with the
complete vendor + Clarivate dialog) is under Director Reports in the internal portal.*

---

**Subject:** Autolend kiosk — updated read

Juliane,

Updating last week's summary now that a reference has finally weighed in and we've dug deeper on the
integration. The bottom line hasn't changed, but it's better supported now:

- **Still technically workable — and still entirely on the vendor's word.** It integrates with our Polaris
  and we can secure it to our standard, but every capability and security assurance is the vendor describing
  their own product.

- **First real reference is lukewarm.** Jason Bedsaul (ODIN — the North Dakota library consortium, a site the
  vendor pointed us to) gave his initial read: *"they're okay, but I wouldn't go out of my way to bend
  procurement rules to go with them over evaluating all your options."* It was a State Library purchase forced
  on his team without their input; they run it on SIP2 plus a hand-built SQL report for holdings and have
  never revisited it. He found the vendor difficult during implementation and "very sure of themselves." He
  hears no reliability complaints, but wants to confirm with their IT before a formal endorsement next week.

- **A competence flag, not just a preference.** The vendor runs patron transactions over the older **SIP2**
  protocol and told us the modern, more secure **PAPI** "wasn't possible" for this. But Polaris has supported
  PAPI for these transactions since **2024 (v7.6)**, and patron authentication has been in PAPI far longer —
  and I verified the key call works against our own catalog. Tellingly, ODIN was given the *same* "the API
  can't do it" story and had to fall back to a SQL workaround. The vendor has had a safer path available for
  years and hasn't adopted it.

- **That's why securing it is extra work for us.** Because they insist on SIP2, protecting patron card numbers
  and PINs means we'd have to **build a small dedicated secure network** around the device. And the vendor
  themselves admit they've rarely — if ever, outside a couple of academic sites — set up a connection this
  hardened.

- **The one risk we can't engineer away stands:** it's a vendor-managed Windows device the vendor can remote
  into for support. We reduce that exposure; we can't eliminate it.

- **Recommendation unchanged, now better grounded: take about one more month.** Get Jason's formal reference
  in hand and put at least one PAPI-native vendor (e.g., Lyngsoe) side-by-side before committing. Nothing here
  is a showstopper — but nothing argues for committing on vendor-word alone, and the first independent voice
  says the same thing: evaluate your options first.

Full report is in Director Reports if you or the board want the detail. Happy to walk through it.

Derek
