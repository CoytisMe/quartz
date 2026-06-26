---
category: research
date: 2026-06-11
publish: true
tags:
  - smart-lock
  - home
  - hardware
  - rfid
---
Yale Acc Password: Accountforlocklol@123
## Smart lock
Lockwood 001 Touch Plus
- [Their website](https://www.lockweb.com.au/au/en/products/keyless-smart-products/smart-products/001touch-plus-smart-deadlatch)
- [Bunnings](https://www.bunnings.com.au/lockwood-001touch-plus-smart-deadlatch_p0458564)
- [Amazon](https://www.amazon.com.au/Yale-YD-01-NOMOD-SN-Keyless-Connected/dp/B08ZL65PZQ/ref=asc_df_B07961CKWC?mcid=80cf051f2de23193a03135120e6dfc75&tag=googleshopdsk-22&linkCode=df0&hvadid=712242965703&hvpos=&hvnetw=g&hvrand=17324183012168162441&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9071446&hvtargid=pla-422524010623&hvocijid=17324183012168162441-B07961CKWC-&hvexpln=0&gad_source=1&th=1) - Same thing (I Think, pls confirm) but the parent brand, Yale. no smart module.

Basically the only one we can get because we have a 001 Deadlatch which this is built to replace. We have to keep the damage to the door to an absolutely minimum, no cutting holes, no chiseling etc.

Don't really want the smart module, Wi-Fi connected front doorlock dumb, we had RFID tags, PINs and not to get to locked out. The long distance fobs mentioned below sound cool but don't think we'll pay for the smart module for that.

Need to check into the RFID tags, if we can replicate them with the standard NFC Equipment we've got, ie make our own cards. If note, what do we need.

Amazon Link to the tags [sold seperately](https://www.amazon.com.au/Yale-P-YD-01-RFIDT-BL-Smart-Black/dp/B01N32UF3X?th=1)

I wrote down  `mifare classic 1k tag nfc tag` last night when I was looking on my phone but can't find that anywhere.

That there we go, Google AI says
`The Yale Smart Living Keyless Connected (YD-01-CON-BL) uses Mifare (13.56 MHz) passive RFID technology. Specifically, it uses standard MIFARE Classic 1K or MIFARE DESFire tags`

Apparently they can also a remote key fob, like a car from a distance. Is cool but does it require the smart module?

---

## Research Notes (added 2026-06-11)

### Is the Yale YD-01 on Amazon the same thing?

**Short answer: No, not the same physical product — but same family, same brand, worse fit for you.**

Both are ASSA ABLOY products (Yale and Lockwood are siblings under ASSA ABLOY). They share the same Yale Home app ecosystem and Bluetooth module. But the Amazon listing you linked (YD-01-CON-BL) is described as a **nightlatch**, not a deadlatch. A nightlatch has a spring bolt that latches automatically when the door closes — your existing 001 is a deadlatch (the bolt only moves when you turn the key/cylinder). These are different form factors. The Lockwood 001 Touch Plus is specifically designed to replace a Lockwood 001 deadlatch body for body — it uses the same 001 latch mechanism. The Yale YD-01 would likely not drop into your door prep without modification.

The "NOMOD" in the Amazon listing means "No Module" — i.e., sold without the smart/Bluetooth module. This is an older model configuration.

**Bottom line: The Lockwood 001 Touch Plus at Bunnings is the right product. Don't buy the Yale one off Amazon for this application.**

### Is it a good replacement for the 001 Deadlatch? Are there others?

**Yes, this is literally its purpose.** The 001 Touch Plus is explicitly designed to retrofit into an existing Lockwood 001 (and also 002, 100, 201, 211, 213) deadlatch installation. Same backset (60mm), same footprint. There is a note in the specs that "some door frame modifications may be necessary" — likely meaning the strike plate area, not cutting into the door itself. Given you have a 001 deadlatch already, this should be about as close to a direct swap as you'll get.

**Are there any others?** Basically no. The Australian residential door prep market is dominated by Lockwood/ASSA ABLOY, and the 001 is their dominant cylindrical deadlatch. No other brand makes a smart lock designed to drop directly into a 001 hole pattern. Smart locks from other brands (Schlage, August, etc.) are either designed for different prep styles (mortice), are rimlock add-ons that sit over the existing lock and motorise the thumb turn (messy, ugly), or require completely different cutouts. For a true no-modification drop-in replacement: **this is the only option.**

There is one older model — the Lockwood 001Touch (non-Plus) — which has been **officially discontinued by ASSA ABLOY**. Don't buy that one, even second-hand.

### What can you do with and without the "smart module"?

Quick terminology clarification first, because the marketing is a bit confusing:

The 001 Touch Plus **already has Bluetooth built in** — the touchpad unit IS the smart module. What you can optionally buy separately is the **Yale Connect Plus Wi-Fi Bridge** (model AYR-BDG-CB2-ANZ, ~$100–$150 when available), which adds internet-over-WiFi remote access. This bridge is currently **discontinued or very hard to get** in Australia — it shows as "No Longer Available" at multiple retailers as of May 2026. Yale's website mentions a newer hub model but that's mostly for the alarm ecosystem.

**Without the WiFi Bridge (i.e., the lock as sold at Bunnings):**
- PIN code entry (4–12 digits, up to 20 codes)
- RFID/NFC key tag entry (up to 20 tags, Mifare Classic 1K)
- Mechanical key override (from inside, deadlock mode)
- Bluetooth auto-unlock via Yale Home app (opens as you approach, requires phone)
- Activity log (who entered and when, viewable via Bluetooth when in range)
- Auto-lock after preset time (up to 30 minutes)
- DoorSense™ (tells the app if door is open/closed, Bluetooth only)
- Visitor codes with time-limited expiry
- Passage mode (hold unlocked for tradespeople etc.)

**Requires the WiFi Bridge (you've ruled this out, wisely):**
- Remote unlock/lock from anywhere over the internet
- Push notifications when away from home
- Voice assistant control (Alexa, Google Home, Samsung SmartThings)

**So for your use case (PIN + RFID tags, no remote control):** the lock as sold at Bunnings is everything you need. You're not losing anything meaningful by skipping the bridge.

### The Remote "Car Key" Style Fob

The fob you remember reading about is most likely the **Yale Home app auto-unlock** feature — which uses Bluetooth + phone GPS to unlock the door as you walk up. It's not a physical fob. This does NOT require the WiFi Bridge. It's built into the Bluetooth module already in the lock.

There are also RFID-format key fobs (the RFKF10 — a Mifare 1K card in a keyring plastic housing, not a clicky RF remote) which are just proximity fobs you wave at the reader. These are enrolled like any other key card.

There is no traditional 433MHz car-key style remote fob sold for this lock. The "distance" unlock is the phone app over Bluetooth, range ~10m.

### Can you make your own key cards?

**Yes. This is easy. Here's exactly how:**

The lock officially uses **Mifare Classic 1K** at **13.56MHz** (ISO 14443A) — this is confirmed in the official ASSA ABLOY product specs. When you enrol a card, the lock stores that card's UID (unique identifier number). It does NOT do a full cryptographic challenge-response — it just reads the UID and compares it to the stored list. This means:

**Option 1 — Buy blank Mifare Classic 1K cards and enrol them directly (easiest, cheapest)**
Any Mifare Classic 1K card/fob/keytag will work. The lock doesn't care about what data is on the card, only the UID. Just hold the blank card to the reader during enrolment mode and it's done. You can buy these for $1–3 each:
- eBay: "Mifare Classic 1K S50 ISO 14443A 13.56MHz key fob" — bulk packs of 10–50
- AliExpress: even cheaper, same product
- EverythingID.com.au (Australian seller) also sells pre-verified compatible ones

**Option 2 — Clone an existing enrolled card onto a new blank (if you want duplicates of the same UID)**
You need a "Magic UID" card (also called Gen1/Gen2 CUID cards — writable UID, backdoor-accessible sector 0). You can write to these with:
- ACR122U USB reader (~$30–50) + MIFARE Classic Tool app on Android
- Flipper Zero (if you have one handy — you do?)
- Any Android phone with NFC + MIFARE Classic Tool app CAN read UIDs, but standard phones cannot write to sector 0. You need the ACR122U or a magic card programmer.

Actually for your use case, Option 1 is better. Just enrol a new blank card — no cloning needed, cleaner, takes 30 seconds.

The official Lockwood key tags (RFKC10, RFKF10) are sold on Amazon and the ASSA ABLOY eshop at $28.24 for a 10-pack of the fob version. These are convenient but no different functionally than any generic Mifare 1K tag you buy in bulk.

### How secure are Mifare Classic 1K RFID cards in this context?

**Honest answer: not great cryptographically, but fine in practice for a home front door.**

Mifare Classic 1K uses the **Crypto1** algorithm, which was reverse-engineered and published in 2007. The keys have been known to be brute-forceable since then. In 2024, security researchers found new hardware backdoors in some card variants, allowing key extraction in minutes.

What this means practically:
- Someone with a **Flipper Zero** (sub-$200 device, very popular) can read your card's UID and clone it onto a magic card in under 60 seconds if they can get within 5–10cm of your keyring. This is a realistic attack vector.
- Someone with an **ACR122U** + laptop is slightly less convenient but same result.
- **A standard smartphone** cannot do this — writing to sector 0 requires specialist hardware. The average person cannot clone your tag with their phone.

The bigger question is whether someone determined enough to do this would also just pick the cylinder or kick the door. For a home, that's the right framing — Mifare Classic is weaker than the physical door hardware, but the physical door hardware is also weaker than most people assume.

**Mitigations:**
- Don't leave your key tags unattended near strangers
- RFID-blocking sleeve/wallet for the fob (if you carry it in your pocket near strangers)
- The lock also has a PIN — use PIN + tag as your primary entry, so a cloned tag alone isn't enough if you can set it to require both (check if the lock supports dual-factor — it probably doesn't, most consumer locks don't)
- The lock auto-locks, so at least it's self-securing

For a home front door in Melbourne suburbs: this is an acceptable risk. You're not protecting a server room.

### Pricing Summary (as of June 2026)

| Product | Colour | Price | Where |
|---|---|---|---|
| 001 Touch Plus Smart Deadlatch | Matte Black | **$414.96** | Bunnings (Bunnings-exclusive colour) |
| 001 Touch Plus Smart Deadlatch | Chrome | ~$429–$439 | Amazon AU |
| 001 Touch Plus Smart Deadlatch | Chrome | ~$419 | The Lock Shop |
| 001 Touch Plus Smart Deadlatch | Chrome or Black | $580 | Electronic Locks Australia |
| 001 Touch Plus (full kit with new latch body + L-Module) | Chrome | $746.71 | ASSA ABLOY trade eshop |
| RFKF10 key fob/card (10-pack) | — | $28.24 | ASSA ABLOY eshop |
| Generic Mifare 1K key fob (10-pack) | — | ~$5–15 | eBay/AliExpress |
| Yale Connect Plus WiFi Bridge | — | ~discontinued~ | Various (unavailable) |

**Historical note:** Buywisely shows the all-time low was **$350 on 1 Nov 2025** (Bunnings). Not far from current $415. If there's a Bunnings sale coming, could be worth waiting.

**Chrome vs Black price:** The black is actually cheaper at $415 (Bunnings exclusive) vs ~$429+ for chrome. You're not paying a premium for black, if anything it's a small discount.

### Is it actually a good lock? Honest take.

Mixed bag. ProductReview.com.au: **2.4/5 from 11 reviews** — 45% positive, 55% negative.

**Good things people report:**
- When it works, it works well and can last 7+ years without issues
- Phone support from Lockwood/ASSA ABLOY is apparently responsive and willing to replace defective units
- Keyless convenience is genuinely great
- Multiple user codes useful for household

**Bad things:**
- Reliability/QC concerns — several units have failed within months
- Latch mechanism refusing to retract (lock-out risk — important to know)
- Battery terminal corrosion on some units
- Touchpad becoming unresponsive over time
- Build quality rated 2.5/5 for money spent

**Broader context:** This is essentially the only real option for your door prep, the alternatives are worse (rimlock add-ons, full door modification). The failure rate isn't catastrophic — most units seem to work fine. ASSA ABLOY does support the product. The mechanical key override means you can always get in even if the electronics die. Given the constraint (001 deadlatch, no cutting), it's the right call. Just register it and keep a spare battery handy.

**25-year mechanical guarantee** on the latch mechanism itself, which is reassuring.

### Things to confirm before buying

1. **Which SKU do you actually need?** There appear to be two versions:
   - Just the keypad/smartlock unit (to swap onto your existing 001 latch body) — cheaper
   - Full kit with a new 001 latch body included — more expensive, probably not needed if your existing latch is fine
   The Bunnings $414.96 listing is the "Smart Deadlatch" which appears to include a new latch body. If your existing 001 latch mechanism is in good shape, you might save money buying just the keypad component, but the display pack from Bunnings is probably the cleanest option given you want a known-working system.

2. **Door prep confirmation:** The lock replaces the 001 directly. Measure your backset (should be 60mm — distance from door edge to centre of cylinder hole). Almost certainly already 60mm if you have a standard 001.

3. **Strike plate:** You may need to swap or adjust the strike plate in the door frame. This is the one place where minor modifications (chisel work on the frame, not the door) might be needed if the latch throw is slightly different. Low risk.

---

## Original Questions
- Is it a good lock? → Mixed reviews, 2.4/5, QC inconsistent but support is decent. For your application it's the only real option.
- Is the Yale one on Amazon and the Lockwood one at Bunnings actually the same? → **No.** Same ASSA ABLOY parent company, same app ecosystem, but the YD-01 is a nightlatch, not a deadlatch. Wrong product for your door. Don't buy it.
- Am I correct in saying it's good for replacing the 001 Deadlatch? Are there others? → **Yes, correct, and no there are no others.** This is purpose-built for the 001.
- What can you do with and without the smart module? → See detailed breakdown above. You lose nothing meaningful without the WiFi Bridge. The bridge is also discontinued/hard to get anyway.
- Will we be able to make our own key cards with standard NFC business card equipment? → **Yes, very easily.** Buy any blank Mifare Classic 1K tag, enrol it in the lock. Standard NFC *card writer* equipment for business cards may or may not work depending on whether it can handle Mifare 1K format (some can, some are NTAG213-only which is different). If your business card gear is NTAG-based, it won't work. Buy a pack of Mifare 1K key fobs off eBay for $10. Easiest path.
- If not, how hard or expensive is it to do? → ~$10 for a pack of generic Mifare 1K tags off eBay, 0 extra hardware needed. Easy.
- How secure are RFID cards in this context? → Mifare Classic 1K has known crypto weaknesses, clonable with Flipper Zero. Acceptable for home use. Not suitable for a data centre. Don't leave your fob unattended near strangers.
- Best price you can find? → **$414.96 at Bunnings, Matte Black.** All-time low was $350 in November 2025 — watch for Bunnings sales. Chrome is $429+ elsewhere. Black is actually the cheaper option here.

---

## Sources
- [Lockwood official product page](https://www.lockweb.com.au/au/en/products/keyless-smart-products/smart-products/001touch-plus-smart-deadlatch)
- [Bunnings listing](https://www.bunnings.com.au/lockwood-001touch-plus-smart-deadlatch_p0458564)
- [ProductReview.com.au - Lockwood 001 Touch](https://www.productreview.com.au/listings/lockwood-001-touch)
- [BuyWisely price tracker](https://buywisely.com.au/product/lockwood-001touch-plus-smart-deadlatch)
- [ASSA ABLOY official eshop (trade)](https://eshop.assaabloyopeningsolutions.com.au/001touch-plus-keypad-with-001-latch-and-yale-network-l-module-chrome-plate-chrome-plate-lwl-001tddl-cpdp-64899edf14a9f)
- [NCC Group - Cracking Mifare Classic 1K](https://www.nccgroup.com/research/cracking-mifare-classic-1k-rfid-charlie-cards-and-free-subway-rides/)
- [Quarkslab - Mifare Classic backdoors 2024](https://blog.quarkslab.com/mifare-classic-static-encrypted-nonce-and-backdoors.html)
- [GitHub - Cloning Mifare Classic 1K walkthrough](https://github.com/aalex954/cloning-MIFARE-classic-1k)
- [The Lock Shop - 001 Touch Plus + Yale L-Module](https://thelockshop.com.au/products/lockwood-001-touch-plus-yale-network-l-module)
- [ASSA ABLOY - discontinuation of original 001 Touch](https://www.assaabloy.com/au/en/stories/news/discontinuation-of-the-lockwood-001touch-keyless-digital-deadlatch)
