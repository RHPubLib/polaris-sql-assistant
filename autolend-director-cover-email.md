# Abbreviated cover email — Autolend kiosk (for Juliane)

*Draft for Derek to send. Plain-language, short. Full report is in the Director Reports section of the
internal portal (managers tier): Director Reports → "Autolend Holds-Locker Kiosk — Technical & Security
Evaluation." It includes the complete vendor/Clarivate dialog as an appendix.*

---

**Subject:** Autolend kiosk — IT's read for the OT board

Juliane,

Here's the short version of IT's evaluation of the Autolend holds locker for Oakland Township. The full
report (with the complete vendor and Clarivate email exchange) is posted under **Director Reports** in the
internal portal if you or the board want the detail.

**Bottom line:** On a technical basis, **yes — it works with our Polaris**, and we'd run it in a
locked-down configuration ("Model 2") that meets RHPL's security standard. The vendor answered every
question and agreed to every safeguard we asked for.

**The one honest caveat (not a knock on the product):** everything we know is the **vendor describing their
own device** — we haven't tested it, and we have no customer references yet. So our "yes" is "yes, per the
vendor, pending a hands-on test." We are **not** telling OT to reject it; we're being transparent about what
we can and can't vouch for. The fix is a short proof-of-integration test before go-live, plus references.

**A few points the board may care about:**

- **Patron card + PIN are encrypted** on the connection — non-negotiable, and confirmed by both the vendor
  and Clarivate.
- **The device never touches RHPL's internal network** — it talks only to Clarivate, who hosts our data.
- **Staff loading stays simple** (a limited "loader" card); the **remote admin page gets RHPL sign-in +
  security key** in front of it.
- **One risk we can't fully remove:** it's a vendor-managed Windows computer the vendor can remote into for
  support. We reduce that exposure but can't eliminate it — it's true of any vendor-run device, and we name
  it openly.
- **Worth knowing:** the vendor was candid that most of their sites **don't** lock down this much. The
  hardening we want is achievable, but it's somewhat first-time for them too — another reason to test before
  go-live. It's a reason to verify, not to distrust.
- **A PAPI-native alternative exists (Lyngsoe)** — larger vendor, but its hold-locker-over-PAPI option is
  just as unproven today. Noted in case the board wants a broader comparison.

**Out of scope for IT:** vendor reliability/track record and cost — the board should weigh those separately.

Happy to walk through any of it, or join the board discussion if useful.

Derek
