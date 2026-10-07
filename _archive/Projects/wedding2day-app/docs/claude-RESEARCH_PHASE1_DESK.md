# W2D — Research Phase 1: Desk Research

Status: Dev PAUSED. This document does not change DECISIONS.md. Nothing here is locked — it's input to discovery, not a new plan.

---

## 1. Market size (context only, not a green light)

| Source | Figure |
|---|---|
| Grand View Research | India wedding services market ~$228.69B, growing |
| Custom Market Insights | India wedding services ~$502.56B by 2035 |

Both are broad "wedding services" (venues, catering, photography, decor, travel, etc.), not decor-manufacturing specifically. Wide spread between sources is normal for this kind of report — treat as "large and growing," not a precise number. This does NOT tell us whether a B2B trade layer between manufacturers and decorators is a real gap — it just says the overall pie is big.

---

## 2. Direct incumbents already exist — this is the critical finding

**IndiaMART already runs a live "Wedding Decoration" directory** (dir.indiamart.com/impcat/wedding-decoration.html) with real usage signals, not a dead category:

| Signal | What's there |
|---|---|
| Supplier count | 25+ manufacturers/suppliers visible on one category page alone (mandap decoration, party decoration, wedding planners, etc.) |
| Geographic coverage | City-level pages exist (Chennai, Madurai already indexed) |
| Pricing | Prices shown openly, ₹50/piece to ₹5L/event |
| Contact | WhatsApp button, Call Now, "Post Your Requirement" (RFQ), "Get Best Price" |
| Trust signals | Star ratings (4.0–4.6), review counts (6–545), response rates (63–96%), Verified Exporter badges |

**TradeIndia has the same thing in parallel** — separate "Wedding Decorative Items" manufacturer directory, plus city pages for Chennai and Madurai specifically.

What this means for W2D as currently scoped: the core mechanic locked in DECISIONS.md — post a listing, reveal phone, go to WhatsApp — **is what IndiaMART and TradeIndia already do, at national scale, with years of SEO, trust signals, and existing supplier bases.** A new app doing the same reveal-phone mechanic with zero existing supply or demand on it is not differentiated on the mechanic. It would need to win on something IndiaMART doesn't have: TN-specific curation, wedding-specific workflow, community/trust, or something else not yet defined.

This is the single most important thing to resolve before writing more code.

---

## 3. Why B2B marketplaces structurally fail (and how each applies to W2D as locked)

| Failure mode | What it means | How it applies to W2D's locked design |
|---|---|---|
| **Disintermediation trap** | Once buyer and seller connect once, they go direct next time (call/WhatsApp), skipping the platform entirely — especially in B2B where relationships are long-term and repeat | W2D's mechanic **is** "reveal phone, go to WhatsApp" — by design, the platform actively pushes every user OFF the app on transaction #1. There's no reason for a repeat visit once both sides have each other's number. This isn't a bug to fix later — it's baked into the locked design. |
| **Chicken-and-egg liquidity** | No buyers without sellers, no sellers without buyers; B2B is slower to bootstrap than consumer apps because procurement relationships are sticky | TN decorators and manufacturers likely already have known suppliers they call directly (see section 4). Getting either side to open a new app before the other side is there is the classic cold-start problem, worse in a state-only launch. |
| **Thin unit economics beyond directory listing** | Pure matchmaking = a catalog. Catalogs get disintermediated fast and have no recurring revenue lever | Current W2D has no take rate, no payments, no logistics, no financing — it's explicitly a catalog + phone reveal. There's currently no answer to "how does this make money" or "why would someone keep coming back" beyond the listing itself. |
| **Vertical niche advantage (the one point in W2D's favor)** | Winners get 60%+ of transactions in ONE specific category, not spread thin | This is the strongest argument for staying narrow (TN, wedding stage decor) rather than expanding — IF the mechanic itself can create stickiness, which right now it structurally can't (see disintermediation above). |
| **Full-stack operations win** | Marketplace leaders that survive don't just match — they handle assortment, quality assurance, logistics, workflow, or payments so leaving the platform costs the user something | W2D is explicitly NOT doing chat, payments, booking, or workflow (per DECISIONS.md section 10). This is the opposite of what makes marketplaces sticky post-connection. |

**Bottom line from desk research:** the current locked mechanic (reveal-phone-only, no chat, no payments, no workflow) matches almost every documented failure pattern for B2B marketplaces, and duplicates a mechanic two national incumbents already run in the same category. This isn't "the branding is off" — it's a structural problem with the core loop.

---

## 4. What's still unknown (desk research can't answer these — needs real people)

- Do TN decorators/manufacturers actually use IndiaMART/TradeIndia today for this, or do they ignore it and rely on personal networks/associations? (No evidence found of an active TN-specific wedding-decor trade association online — worth asking directly rather than assuming absence means it doesn't exist.)
- What does the ACTUAL current sourcing workflow look like — phone calls to known contacts, WhatsApp groups, trade fairs, walk-ins? If it's already informal and relationship-based, a directory app competes with "who I already know," which is a much harder problem than competing with IndiaMART.
- Where does the real pain sit: finding NEW suppliers (discovery problem — directory could help) vs. speed/price/production-capacity checking with suppliers ALREADY known (coordination problem — directory does nothing for this)?
- Would either side pay for something beyond a directory (e.g., production-capacity visibility, group buying, financing, logistics) — i.e., is there a "full-stack" wedge available that isn't already occupied?
- Is there a reason for repeat use post-connection that isn't chat/payments/booking (which are explicitly out of scope)?

---

## 5. Recommendation for next step

Do not resume dev. Do not revise DECISIONS.md yet either — revising it now would just be guessing again with better vocabulary.

Next: structured customer discovery — real conversations with TN decorators AND manufacturers — targeted at the 5 unknowns in section 4, especially the discovery-vs-coordination question, since that determines whether a directory-style product (what's currently built) is solving the right problem at all.

---

## 6. Approach assessment — is this the correct approach as currently locked?

**Short answer: the strategy is right, the product mechanic is very likely wrong.**

**What's correct about the approach:**

- Starting narrow (Tamil Nadu, one category — stage decor) instead of the full B2B2C vision is the right sequencing. Standard marketplace playbook: niche and local strength before expanding — you're following it.
- Real domain authority — 10 years inside manufacturing gives you supply-side knowledge a generic outside founder doesn't have. Genuine edge.
- The underlying pain in your own founder story is specific and plausible: decorators/service providers manually re-checking price, production ability, and delivery feasibility across multiple manufacturers, for every job. That's a believable, well-articulated problem — not vague fragmentation hand-waving.

**What's wrong about the approach as currently locked:**

1. **The core mechanic doesn't solve the pain you described.** Your own story is about *repeated coordination* — comparing price/capacity/delivery across manufacturers for the same requirement. "Post a listing → reveal phone → go to WhatsApp" is a directory. It doesn't let anyone compare price/capacity/delivery in one place. It just gets you a phone number — which is the part people already know how to do; they already call. The product as scoped answers "who has this?" not "who's the best option for this, right now?" — and the second question is the one from your own story.

2. **You're rebuilding something that already exists at national scale.** IndiaMART and TradeIndia already run live wedding-decoration manufacturer directories — pricing, RFQ, WhatsApp/Call buttons, ratings, response rates, Chennai/Madurai city pages already indexed (see section 2). If the target pain is "I don't know who makes this," that's already partially addressed by tools whose actual usage among TN decorators is still unverified — but the point stands: a new app copying the same discovery mechanic isn't differentiated on the mechanic itself.

3. **The mechanic is structurally self-defeating.** No chat, no payments, no booking, no workflow (DECISIONS.md §10) means all the app's value is delivered in a single tap — the phone reveal — after which the user leaves for that relationship permanently. Nothing brings them back to the app for that same manufacturer next time. This is the textbook B2B marketplace disintermediation failure (section 3): you build the bridge, people cross it once, then stop paying any toll to use it again.

**Net assessment:** the *strategic instinct* (niche, TN, supply-side-informed, sequenced before the bigger vision) is sound and should probably survive discovery unchanged. The *product mechanic* (directory + phone reveal, no comparison/coordination features) is very likely solving a smaller, different problem than the one described in your own founder story, and duplicates existing national tools rather than beating them at the thing you're actually positioned to fix — which looks more like **comparison and coordination across manufacturers**, not **discovery of a manufacturer's existence**.

This reframes the open questions in section 4: the sharpest discovery question isn't just "discovery vs. coordination pain" in the abstract — it's specifically whether decorators would value a tool that lets them check price/capacity/delivery across several manufacturers for one requirement in a single flow, versus what they do today (calling each one separately).
