# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing (for example "vintage graphic
tee under $30"), the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**

My search scores listings by keyword overlap with the user's words, not by
meaning. A listing can genuinely suit what someone's asking for and still be
missed if they phrase it differently than the listing is worded (e.g. "retro
band shirt" vs. a listing tagged "vintage graphic tee, band tee"). So some
truly matching queries will still come back empty — not because the agent is
broken, but because plain keyword matching doesn't understand synonyms.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit`, reports 0 model calls for that session, and returns a
message naming what to change — 5 of 5 tries.

**Why this target:**

Criterion 1 depends on whether the user's words happen to overlap with a
listing's words — that's genuinely uncertain. This one doesn't depend on
wording at all: it's a single `if` check on whether the list is empty, with no
model and no fuzzy matching involved. If that check is written correctly, it
either always stops on an empty list or it has a bug — there's no reason it
would work 4 times and randomly fail a 5th, so 5 of 5 is the honest target.

---

## 3. Something about state

Given a completed run where search_listings returns at least one result, the
id of session["selected_item"] matches the id of the new_item actually
received by suggest_outfit — 5 of 5 tries.

**Why this target:**

This is plain code moving a value from one variable to another, not a model
doing approximate work — so a single mismatch out of 5 would mean a real bug
in the loop (the wrong item, or a stale one, getting passed along), not
natural variation. Unlike criterion 1, there's no reason to allow slack here.



---

## 4. Something about the fit card

Given the same item run through create_fit_card 5 separate times, the item's
price appears in the caption text, written as a dollar amount (for example
"$38" for a listing priced 38.0), in 5 of 5 tries.

**Why this target:**

The caption's wording is expected to differ every run — that's the model
doing its job, not a bug. But the price is a fact I put directly into the
prompt myself, not something the model has to invent, so it should come
through reliably every time rather than only most of the time.



---

## 5. Your choice

Running suggest_outfit on the same item twice — once with an empty wardrobe
({"items": []}) and once with a non-empty wardrobe — produces, in 5 of 5
pairs, two results that are each at least 50 characters long, raise no
exception, and are not identical to each other.

**Why this target:**

The tools.py docstring flags the empty wardrobe as a deliberate edge case
tested on purpose in unit 4, so it needs a real, checkable target now. I
can't check the empty-wardrobe response against a list of item names, since
there are none — so length and no-crash is the concrete fact I can verify
for that response alone. But length and no-crash alone wouldn't catch a
subtler bug: code that returns text either way but never actually checked
whether the wardrobe was empty. Comparing it against the non-empty-wardrobe
result for the same item catches that — if the two were identical, the
empty-wardrobe branch isn't doing anything different. 5 of 5 because this is
just one `if` statement being wired up correctly, which should be reliable
every time.



---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
