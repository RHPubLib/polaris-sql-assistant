# Email to Jason (ODIN) — Autolend reference / security gut-check — draft

*Draft for Derek to send. Tone: peer-to-peer, candid, IUG/PIAC-colleague familiar. Three jobs in one note:
(1) lay out my honest worries about the product/vendor, (2) ask Jason how ODIN actually finds it in
production, and (3) go deep with a fellow IT person on how they secured the tunnel/connection — I share my
planned design and ask what they did. Security questions are NUMBERED so he can reply inline. Jason = ODIN
(Online Dakota Information Network), Polaris consortium; Derek knows him from IUG + shared committees.
Product = Autolend holds-locker kiosk from International Library Services (ILS), Bill McClendon CTO. RHPL is
Polaris 8.1 hosted by Clarivate. NOTE: Derek is reworking the personal/intro opening himself.*

---

**To:** Jason <…@odin…>
**Subject:** Picking your brain on the Autolend lockers (esp. the security side)

Jason,

Hope you're doing well — overdue for a real catch-up since the last IUG, and since we've sat on enough
committees together I figured you're exactly the person to be straight with me on this.

I'm reaching out because I learned ODIN is running the **Autolend** holds lockers from International
Library Services, and we (Rochester Hills, near Detroit) are evaluating one for a township we serve. Our
board votes soon, and I'd trust your read on this — especially the security/network side — far more than
anything I can get from a sales thread. So I wanted to come to you straight: here's what's making me
hesitate, a few honest questions, and then the part I'd most value your time on, which is how you actually
secured the connection.

**Where my head's at — the worries.** A few things have been nagging at me:

- It's been hard to get **customer references**. The vendor's been responsive and candid otherwise, but I
  still don't have a list of Polaris libraries running this that I can call — I found out ODIN uses it on my
  own, not from them. That alone made me want to talk to a real user.
- So far **every assurance is the vendor describing their own product** — no independent site to confirm it
  holds up day to day, and no hands-on look at the actual SIP2/PAPI traffic.
- One thing gave me pause on how deep their Polaris knowledge runs: I was told flatly that **PAPI can't
  return a Bib record from an item barcode** — that we'd have to feed them a barcode→Bib ID dump. But when I
  tested it on our own catalog, the PAPI **BibGetByType v2** call with `?type=barcode` handed the Bib ID
  right back (Control Number / ElementID 11, off the MARC 001). Works cleanly for us. Maybe I'm missing a
  wrinkle on their end, but it left me wondering how current their PAPI knowledge is versus leaning on SIP2
  for everything.

None of that is a verdict — the product may be perfectly solid — but it's exactly the stuff a real user can
settle in a few minutes.

**Quick honest take, if you have a minute on each:**

- **The company** — how's International Library Services to deal with? Responsive, straight with you, good on
  support after the sale?
- **The product** — how's the Autolend unit performing day to day? Reliability, patron experience, anything
  that's bitten you?
- **Polaris integration** — how well does it really tie in? Holds routing, notifications, check-in/inventory
  behaving the way you'd want?
- **Your hosting** — is ODIN **self-hosted, or Clarivate-hosted?** (matters a lot for how closely your setup
  mirrors ours — we're Clarivate-hosted.)
- **SIP2 vs. PAPI** — does the locker talk to you over **SIP2 only, or a mix?** And if you hit that
  barcode→Bib lookup limitation, how did you handle item lookup?

---

**The part I'd most value — how did you secure the connection?**

This is where I'd really love to compare notes IT-person to IT-person, because the way the data path is built
is doing most of the heavy lifting on security, and I want to know what's actually worked in the field versus
what I've reasoned out on paper. I've laid these out as numbered questions so it's easy to fire back quick
answers inline — even one-liners would help me a ton. Here's roughly where I've landed and what I'd love to
hear about ODIN's setup:

1. **Transport encryption / the tunnel.** Polaris hosted-SIP has no native TLS, so the only encrypted path
   Clarivate offered us is an **stunnel** proxy — their stunnel server, a compatible stunnel client on the
   Autolend, patron barcode+PIN encrypted end to end with no cleartext SIP fallback. **Did ODIN end up running
   stunnel (or something else) in front of SIP2? Did you enforce no-cleartext, and do you verify the server
   cert on the client side?** That last bit is what I'm fussiest about — encryption without cert validation is
   still MITM-able, and I can't tell yet whether the vendor's client does `verifyPeer` properly.

2. **On-network vs. off — the big one for us.** Here's where our situation likely differs from yours: **do any
   of your lockers sit *outside* your network, or are they all on ODIN-controlled sites/links?** Our unit would
   go in a township spot with no fiber/our network anywhere near it, so we'd have to **leverage cellular
   service** to reach it — the locker would effectively live out on a carrier link, not behind our edge. If all
   of yours are on-network, that's useful to know too; if any run remote/cellular, I'd love to hear how that's
   gone and what you did to make it safe.

3. **Connectivity + the source IP.** Tied to #2: Clarivate allowlists the SIP endpoint by **source IP**, and a
   plain cellular link rolls its IP constantly, so for our off-network case a static IP is effectively
   mandatory. My leaning is to put the unit behind **our own Peplink + SpeedFusion to a FusionHub that presents
   one static public IP** (cellular WANs underneath can churn, Clarivate only ever sees our fixed IP, stunnel
   still rides inside). The simpler alternative is just buying a **static carrier IP** and running it on the
   vendor's cellular box. **Which way did ODIN go — and does the Autolend take an Ethernet/WiFi WAN handoff
   cleanly for you, or are you on its internal connection?**

4. **Network segmentation + egress.** I'm planning to keep this thing **off our internal network** — its own
   segment, default-deny outbound, only Clarivate (via the tunnel) + the vendor RMM + Windows
   Update/Defender/NTP/DNS allowed and logged. **Did you segment it (own VLAN/WAN) and filter its egress, or
   does it sit on a flatter network?**

5. **The staff web interface.** My biggest surprise — the staff page defaults to **plain HTTP, served off the
   device's own IP**, i.e. potentially an inbound web service on the public internet. Vendor says no PII is
   shown there, and they'll do HTTPS with our cert / IP-restrict it / let us front it with our own **OAuth +
   FIDO2 reverse proxy**. **How is yours exposed — public, IP-restricted, behind a proxy? Did you put any
   auth/MFA in front of it, or leave it at the vendor default?**

6. **Vendor remote access (Splashtop) + credentials.** It's a vendor-managed Windows box with **Splashtop**
   remote access and SIP2 creds stored locally — so a vendor-side compromise is the residual I can't fully
   design away. I'm planning least-privilege SIP2 (checkout/holds only) + taking the free IT Splashtop account
   for visibility. **Are you comfortable with their remote-access setup? Did you scope the SIP2 account down,
   and have you ever had a security or access concern with the vendor plane?**

7. **Did you proof-test it?** Before or after go-live, did you ever **packet-capture the live traffic** with a
   test patron to confirm it's actually encrypted, no cleartext leaking, and the device isn't storing/sending
   anything it shouldn't? That hands-on verification is the gate I most want, and I'm curious whether you found
   it worth doing — or whether it just worked and you moved on.

Anything else you'd flag if you were in my seat — good or bad — I'm all ears. I'd genuinely rather hear the
warts now than after a board vote, and I'm happy to share my full write-up (network diagrams and all) if it's
useful to you.

Thanks, Jason — really appreciate it. Happy to return the favor anytime, and let's grab time at the next IUG
regardless.

Best,
Derek

Derek Brown
Director of IT, Rochester Hills Public Library
