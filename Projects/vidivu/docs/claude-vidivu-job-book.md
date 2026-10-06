# The Vidivu Job Book — Worked Examples

Artifact: https://claude.ai/code/artifact/0cd8a760-ed92-4524-8c53-04230dca11df
Companion to: https://claude.ai/code/artifact/512fba09-b09d-4b9d-849c-db44f86ca041

Eight concrete jobs across eight industries, all built from the same three patterns
(capture & extract / route & approve / gather & report). Indian price first,
international price for the identical build second.

## The eight jobs

**01 · Garment export unit (Tiruppur)** — capture & extract
Merchandiser retypes ~40 PDF POs/week into Excel; mistyped size ratio shipped 400 wrong
pieces; hand-built packing lists caused a container hold and 3-week payment delay.
Build: inbox watcher → PO extraction → validate against style master → Sheets/Tally →
WhatsApp for uncertain items. Second flow generates packing list + carton labels.
**2 wks · ₹45,000 / $3,500 · ₹15,000/mo**

**02 · Chartered accountant firm (Coimbatore)** — capture & extract
GST week: hundreds of WhatsApp invoice photos, two juniors typing GSTIN/invoice no/date/
taxable value/tax. A missed invoice produced a notice last quarter.
Build: image intake → OCR+LLM extraction → GSTIN checksum validation → duplicate
detection → clean sheet in their filing software's import format; unreadable items queued
with image attached.
**3 wks · ₹1,20,000 / $7,000 · ₹20,000/mo**

**03 · Diagnostic lab (Erode, 3 branches)** — route & approve
3 machines, 3 formats; staff retype then WhatsApp ~90 reports/day by hand; one went to the
wrong number; doctor wants abnormal results before the patient sees them.
Build: machine output → match to booking by patient ID → branded PDF → auto-send normals →
**abnormal results stop at a doctor approval gate**.
**3 wks · ₹85,000 / $6,000 · ₹18,000/mo**

**04 · School / coaching centre (Erode — warmest network)** — route & approve
Enquiries across phone/WhatsApp/Instagram/walk-in live in four places and one person's
memory. Fee reminders sent when someone remembers.
Build: all channels → one list → instant acknowledgement → owner assignment → automatic
follow-up sequence → Monday digest to principal → scheduled fee reminders with stop rule.
**2 wks · ₹40,000 / $3,000 · ₹12,000/mo**

**05 · Freight forwarder / customs agent (Coimbatore–Tuticorin)** — capture & extract
BL + commercial invoice + packing list + certificate of origin, 4 formats, checked by eye.
Mismatch = customs hold = demurrage.
Build: extract key fields from all four → cross-document reconciliation (qty, value, weight,
consignee, HS code) → exception report naming which document disagrees. Clean files pass
silently.
**4 wks · ₹1,50,000 / $9,000 · ₹25,000/mo**

**06 · Auto component manufacturer (Coimbatore, OEM supplier)** — gather & report
OEM enquiry arrives with drawing + spec sheet; estimator digs through 3 years of quotes in
folders; takes 2–3 days; sometimes too late to win.
Build: index past quotes in pgvector → extract new enquiry specs → surface 5 most similar
past quotes with costings and won/lost outcome. **Estimator still sets the price** — system
only retrieves. That distinction is what earns trust.
**4 wks · ₹1,75,000 / $11,000 · ₹30,000/mo**

**07 · Wholesale distributor (Erode, FMCG/hardware)** — gather & report
Orders arrive as WhatsApp voice notes, photos of handwritten lists, phone calls. Owner
rebuilds the same sales report by hand every Monday from three sources.
Build: speech-to-text + handwriting OCR → structured order lines matched to product master
→ ambiguities auto-confirmed back to retailer → self-building Monday report showing what
moved and which retailers have gone quiet.
**3 wks · ₹95,000 / $6,500 · ₹18,000/mo**

**08 · Bookkeeping practice (UK/US/AU, remote)** — capture & extract
Same job as #02 in a different currency. Bookkeeper hand-codes receipts/invoices into Xero
or QuickBooks; practice can't grow without hiring, and hiring eats the margin.
Build: extraction → code against existing chart of accounts → match to bank transactions →
push as drafts into Xero/QuickBooks → review queue; low-confidence flagged not guessed.
**4 wks · $8,500 · $1,500/mo**

## The four questions (portable process-interviewing skill)

1. "What does someone here do every week that is exactly the same every time?"
   — repetition is the target; variable work is judgement, leave it alone
2. "Where does the same information get typed in more than once?"
   — double entry = two systems not talking = the entire job
3. "What goes wrong often enough that you have stopped being surprised by it?"
   — finds abandoned pain; they can tell you what each occurrence costs
4. "What report does someone build by hand on a regular schedule?"
   — always exists, always hated, quick win when nothing else is obvious

**Then be quiet.** The answer to Q3 arrives after the silence. Don't fill it.

## Pricing formula

```
Their annual cost of doing it by hand:
  hours/week × 52 × loaded hourly cost
  + cost of errors (rework, delays, penalties, lost orders)
  + cost of things not done (missed enquiries, late quotes)

Build price = 1/3 to 1/2 of year-one savings
  → 4–6 month payback, which is what gets a yes

Retainer = 15–25% of build price per year, billed monthly
```

Never quote before getting their numbers. "What does that person cost you, roughly, and how
many hours does this take them?" — normal question, they answer it, and the quote becomes
their arithmetic rather than your opinion.

## Case study template

HEADLINE — [what changed], for a [type of business] in [place]
BEFORE — what the person actually did, plain language; how long, how often, what went wrong
WHAT WE BUILT — 3–4 sentences, no tool names unless recognisable, never "leverage"
AFTER — hours [was → now]; errors [was → now]; anything previously impossible
CLIENT LINE — one sentence in their words, named, approved in writing
TIME AND COST — built in X weeks, paid back in Y months

**Naming permission goes in the contract, not afterwards:**
"Vidivu may describe this work publicly, including the client name and measured results,
subject to the client's written approval of the text."

## First-conversation script (warm network)

> Anna / sir — I have started doing something new at Vidivu.
>
> I take the jobs where someone in an office types the same thing again and again, and I
> make them happen automatically. Invoices, orders, reports, that kind of work.
>
> I am building my first few examples. Can I spend thirty minutes at your place looking at
> how one of your processes actually runs? If I find something worth fixing I will tell you
> what it would cost. If I do not, I will tell you that too and it costs you nothing.

Why it works: names a concrete problem not a technology (say "AI" as little as possible
with local buyers); asks for time not money; offers an honest exit.

Run it 15 times before concluding anything. 3 paid jobs from 15 conversations is a working
funnel at this stage, not a disappointment.
