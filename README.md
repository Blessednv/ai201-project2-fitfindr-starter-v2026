# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **Update (Unit 3):** this is no longer true. All three tools are built and
> wired into the planning loop in `agent.py`, so that last command now runs the
> full agent. See Sample Run below.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr is a thrift-shopping assistant. You type what you're looking for in plain language, like "vintage graphic tee under $30, size M," and it searches a small set of secondhand listings that match your words, size and budget. It picks the best match, suggests one or two outfits that combine it with clothes you already own (or general styling advice if you haven't saved any), and writes a short social-media-style caption about the find. If nothing matches, it stops early and tells you what to change, such as a broader description, a different size or a higher price.



---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** It searches the clothing database for items that match the keywords, size, and maximum budget the user asks for.
- **Inputs:** description (str), size (str or None), and max_price (float or None). To match a size, the code first checks whether the requested size equals the whole listing size (so "US 9" matches "US 9"). Otherwise it ignores parentheses, splits the listing's size by spaces or slashes, and checks for an exact match on one piece, so "M" matches "S/M" but not "XL" or "US 9". Results are ranked by keyword overlap, with title matches counting double.
- **Returns:** A list of listing dictionaries, where each dictionary contains the fields: id, title, description, category, style_tags, size, condition, price, colors, brand, and platform.
- **When it has nothing:** It returns an empty list (`[]`).

### `suggest_outfit`

- **What it does:** It takes the thrifted item that was just found and suggests one or two outfit combinations using clothes the user already owns.
- **Inputs:** `new_item` (dict) — a listing dict for the item being considered. `wardrobe` (dict) — has an `'items'` key holding a list of wardrobe item dicts; this list may be empty.
- **Returns:** A non-empty string containing one or two AI-generated outfit suggestions.
- **When it has nothing:** If the user's wardrobe dictionary is empty, it returns general styling advice for the new piece instead of crashing or returning a blank string.

### `create_fit_card`

- **What it does:** It writes a short, two-to-four sentence social media caption that mentions the item, its price, its platform, and the outfit's vibe.
- **Inputs:** `outfit` (str) — the outfit suggestion string from `suggest_outfit()`. `new_item` (dict) — the listing dict for the item, with fields like title, price, and platform.
- **Returns:** A string (str) containing the generated caption: two to four sentences, written as the buyer who just thrifted the piece, that mention the item, its price and its platform once each and say something specific about the vibe.
- **When it has nothing:** If the outfit string is completely empty or just blank spaces, it returns a descriptive message instead of crashing.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If search_listings returns an empty list, put a message in `session["error"]` that repeats what was searched and tells the user to try a more general description, a different size, or a higher price, then stop (leaving `fit_card` as `None`). Otherwise, take the first result and go to suggest_outfit.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex (no model call), in `agent.py::_parse_query`. It looks for a price after "under", "below", "less than", "max", "up to" or a bare "$30"; for a size after the word "size" (XXS to XXXL, "US 9", or "W30"); and treats whatever is left, minus filler words like "looking for", as the description. A part that is missing comes back as `None` and is simply not filtered on.

**What moves through the session:** In order: `query` (what the user typed) → `parsed` (description, size, max_price) → `search_results` (everything `search_listings` returned) → `selected_item` (the first result) → `outfit_suggestion` (from `suggest_outfit`) → `fit_card` (from `create_fit_card`). `error` stays `None` unless the search comes back empty, in which case it holds the message and the later fields stay `None`. Each tool reads its inputs back out of the session, never straight from the previous call. The loop checks the session each time round and does the next missing step, and calls `trace.check_iterations` first.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30, size M'
  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   That Y2K Butterfly Tee is so cute! Here are two fun ways to style it using pieces already in your closet:

**Outfit 1: Effortless Casual**
*   Baggy straight-leg jeans, dark wash
*   Chunky white sneakers
*   Black crossbody bag

**Outfit 2: Edgy Contrast**
*   Wide-leg khaki trousers
*   Black cropped zip hoodie
*   Black combat boots 
*   Brown leather belt

  Fit card: Just scored the ultimate Y2K butterfly tee on depop for only $18 and I’m literally obsessed. The print is giving major 2000s mall goth energy, and I already have five different outfits planned out in my head. Can't wait to style this little beauty!

2 model calls this session, 457 prompt + 160 output tokens
```

And the query that matches nothing, which stops before the outfit and fit card (no model calls):

```
$ python agent.py     # second example
=== A query it can't ===
  stopped: Nothing matched 'designer ballgown', size XXS, under $5. Try a more general description (for example 'tee' instead of a very specific style), a different size, or a higher price limit.
  fit_card is None — it should still be None here
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Ooh, those Vintage Levi's 501 Jeans — Medium Wash are a classic find! 

Here are two easy ways to style them using your wardrobe:

**Outfit 1: Casual Streetwear**
*   White ribbed tank top
*   Black cropped zip hoodie
*   Chunky white sneakers
*   Black crossbody bag

**Outfit 2: Edgy & Layered**
*   Oversized grey crewneck sweatshirt
*   Vintage black denim jacket
*   Black combat boots
*   Brown leather belt
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Just thrifted the ultimate pair of vintage Levi's 501 jeans in the dreamiest medium wash. I can’t believe I scored these for only $38 on depop—they’ve got that exact slouchy, lived-in 90s vibe I've been hunting for. Can't wait to style them with crisp white sneakers and an oversized tee for the easiest everyday fit.
```

Note: I ran `create_fit_card` three times with the cache off (`AI201_CACHE=0`) and got three different captions, so the output varies. With the cache on, the same prompt returns the same saved caption.

The three runs, pasted from my terminal:

```
$ AI201_CACHE=0 python -c "from tools import create_fit_card; from utils.data_loader import load_listings; item = load_listings()[0]; [print(i+1, create_fit_card('jeans and white sneakers', item), '\n') for i in range(3)]"

1 Finally scored the holy grail vintage Levi's 501 jeans in the absolute best medium wash on Depop for just $38. They’ve got that perfectly broken-in 90s feel that you literally cannot fake. Can’t wait to live in these with my beat-up white sneakers all fall. 

2 Just scored these vintage Levi's 501 jeans on Depop for $38 and I am never taking them off. The medium wash has that 90s slouchy vibethat is impossible to fake. Can't wait to wear these with crisp white sneakers and a simple tee for the ultimate effortless look. 

3 Just thrifted the dreamiest pair of vintage Levi's 501 jeans in the absolute best medium wash. Scored them on Depop for just $38, andhonestly, they have that perfectly worn-in, effortless 90s vibe. Can't wait to live in these with crisp white sneakers all season. 
```

The missing spaces in "vibethat" and "andhonestly" are in the text the model returned. `create_fit_card` only trims the ends of the reply and doesn't change the words.

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1 — size matching in `search_listings`**

- *What I asked for:* I asked the AI to help me build the `search_listings` filter, especially how to handle sizes like "S/M" and "US 9". My Tool Inventory spec already said to split the listing size on spaces and slashes and match an exact piece, because the starter warned that a plain substring test like `"s" in "us 9"` would return shoes for a small top.
- *What came back:* It implemented that rule. Before coding, it read the real sizes in `listings.json` and found two cases my rule missed: sizes with brackets, like "XL (fits oversized)", and sizes like "US 9" that split into two pieces and could never match a search for "US 9".
- *What I changed:* I agreed to two additions. The code now ignores brackets, and a size can also match the whole listing size, so "US 9" finds "US 9". I tested "S" (no shoes, no XL), "US 9" (only the sneakers) and an impossible search (returns `[]`), then updated my Tool Inventory line to match the code.

**Moment 2 — the fit card sounded like a seller**

- *What I asked for:* I asked the AI to write the caption in `create_fit_card` from the thrifted item's details, mentioning the item, price and platform.
- *What came back:* Captions that read like a seller's listing, for example "Now live on my depop!". The prompt had a price and a platform but didn't say who was speaking, and the app is for someone who just bought the item.
- *What I changed:* I added a line to the prompt in `tools.py`: write it as the buyer who just thrifted the piece (using words like "found", "scored", "thrifted"), not as a seller listing it for sale. The new caption said "Just thrifted..." and "I can't believe I scored these for only $38 on depop", and I replaced the old caption in my Sample Run.

**Moment 3 — four identical captions**

- *What I asked for:* I asked the AI to build `create_fit_card`, then ran the same command four times to check that the captions varied, as the brief says to.
- *What came back:* All four captions were word-for-word identical. The AI checked `config.py` and explained that `TEMPERATURE` was already 0.9, so the temperature wasn't the cause. The cache was: the first answer was saved, and every later run with the same prompt got that saved answer back without calling the model.
- *What I changed:* I changed no code. I ran the same command with the cache turned off (`AI201_CACHE=0`) and got three different captions, which confirmed the tool works and the repeats came from the cache. I also decided to paste only one caption into the README, since four identical ones would have looked like a bug.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. A matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. An impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item matches the item passed to suggest_outfit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card mentions the item's price | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Empty vs non-empty wardrobe, 5 pairs | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Run with `python run_eval.py --label before` (cache off, 5 tries per scenario). The full output, every try, is in `results/run_2026-10-10_2001_before.md`. Criterion 5 ran as two scenarios (empty wardrobe and full wardrobe, same query), so its row compares them pair by pair. Criterion 3 is checked by item title, because the trace prints titles, not ids.

**Real output from one try**, pasted as text, naming the file and function
that produced it:

All of it is **Try 1** of each scenario, copied from `results/run_2026-10-10_2001_before.md` (written by `run_eval.py::main`).

**Criterion 1** — query `vintage graphic tee under $30`. Fit card written by `tools.py::create_fit_card`, run by `agent.py::run_agent`:

```
- stopped early: no
- selected_item: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
- search_results: 10

just thrifted the ultimate 2003 tour bootleg style graphic tee for $24 and I'm obsessed with this faded wash. honestly can't wait to pair it with baggy denim and beat-up combat boots for that perfect messy indie sleaze look. posting it on depop soon if I can somehow bring myself to part with it 🥲
```

**Criterion 2** — query `designer ballgown size XXS under $5`. Message from `agent.py::_no_results_message`, trace from `agent.py::run_agent`:

```
- stopped early: yes — Nothing matched 'designer ballgown', size XXS, under $5. Try a more general description (for example 'tee' instead of a very specific style), a different size, or a higher price limit.
- selected_item: (none)
- search_results: 0

[1] parse_query
      in:  'designer ballgown size XXS under $5'
      out: description='designer ballgown', size='XXS', max_price=5.0
[2] search_listings (via MCP)
      in:  description='designer ballgown', size='XXS', max_price=5.0
      out: [] (empty)
      →    branch: empty, stopping
```

**Criterion 3** — query `90s track jacket in size M`. Trace steps 3 and 4 from `agent.py::run_agent` (`trace.step` calls):

```
[3] select_item
      in:  4 results
      out: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
      →    took the first result
[4] suggest_outfit
      in:  item='90s Track Jacket — Navy/White Stripe', wardrobe items=10
      out: That 90s Track Jacket is a great find! Here are two sporty, vintage-inspired outfits using pieces you already …
```

**Criterion 4** — query `vintage graphic tee under $30`. Fit card from `tools.py::create_fit_card`:

```
Just scored the absolute sickest 2003 tour bootleg tee for $24 on depop and the fade on it is unreal. It’s got that perfect lived-in, crusty vintage grunge vibe that usually takes years to find naturally. Can't wait to style this with baggy denim and combat boots.
```

**Criterion 5** — query `denim jacket under $50`, same item, two wardrobes. Outfits from `tools.py::suggest_outfit`.

Empty wardrobe (`wardrobe items=0`):

```
Hey there! That light wash cropped denim jacket is a timeless streetwear staple. 

Try throwing it over a basic white tee, high-waisted black straight-leg jeans, and white sneakers for an effortless, classic look. Alternatively, pair it with a cozy ribbed knit midi dress and chunky boots to mix feminine and edgy vibes. You're going to get so much wear out of this piece!
```

Full wardrobe (`wardrobe items=10`):

```
Do it! That light wash denim jacket is super versatile. 

Here is a great outfit using your new piece:
* **White ribbed tank top**
* **Baggy straight-leg jeans, dark wash**
* **Chunky white sneakers**

Layer the new **Denim Jacket — Light Wash, Cropped** over the white tank and dark jeans with your chunky white sneakers for a cool, double-denim look!
```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | A matching query completes all three tools and returns a fit card | 4 of 5 | MET | 5 of 5 tries ran the search, the outfit and the fit card (fit cards were 232 to 297 characters). 5 is above the target of 4. |
| 2 | An impossible query stops before suggest_outfit, uses 0 model calls, and names what to change | 5 of 5 | MET | 5 of 5 tries stopped after step 2 with the "Nothing matched..." message. I re-measured it separately: 0 model calls and 2 trace steps in each of 5 runs. |
| 3 | The id of the selected item matches the id of the item passed to suggest_outfit | 5 of 5 | MET | The run log's trace compared titles and printed both from the same variable, so on its own it was weak evidence. A separate check intercepted the real argument of suggest_outfit: the ids matched in 15 of 15 runs (3 queries, 5 tries each). |
| 4 | The fit card mentions the item's price as a dollar amount | 5 of 5 | MET | "$24" appeared in all 5 fit cards, and a correct price appeared in all 25 fit cards in the run, across 3 different items. |
| 5 | Empty and non-empty wardrobe give two results of at least 50 characters, with no exception, not identical | 5 of 5 pairs | MET | In all 5 pairs, both outfits were over 300 characters, nothing crashed, and none were identical. Because "not identical" cannot fail, I also checked that the empty-wardrobe outfits name 0 to 1 of the user's 10 wardrobe items and the full ones name 3 to 7. I revised this criterion for that reason (see below); the original line is kept in `criteria.md`. |

**Diagnoses**

**No criterion was missed,** so there is no miss to trace to a tool, the loop's branch, the session or the model's output. What follows is which of my targets were too low, and what I would tighten.

- **Criterion 1 (target 4 of 5).** It passed 5 of 5, so the allowed slack was never used. It also used one query written in the listing's own words, so the weakness named in its own "why" (keyword matching misses synonyms) was never exercised. **Tighter target:** 5 of 5 on each of 5 different matching queries, including different phrasings of the same item.
- **Criterion 4 (target 5 of 5).** Too easy: `create_fit_card` writes "Price: $24" into the prompt, so the model only copies a fact it was given. **Tighter target:** also 0 of 5 fit cards using selling language (such as "listed", "now live", "my closet").
- **Criterion 5.** The "not identical" clause could not fail, because the model's wording varies at temperature 0.9, so it never tested whether the empty-wardrobe branch does anything different. I revised it. The original line is still in `criteria.md`, with the new line and my reason underneath. The revised version counts how many of the user's wardrobe items each outfit names (at most 1 for the empty wardrobe, at least 3 for the full one). I chose those numbers after seeing the before-run results, which I say in the note there. The target is still 5 of 5, and the verdict is MET under both the original and the revised wording.

**A problem my criteria did not measure.** In 5 of the 25 fit cards, the caption sounds like a seller, not a buyer, for example "Now live on my Poshmark closet if anyone wants to grab it". **Place:** the model's output, from `tools.py::create_fit_card`. **Likely mechanism:** the prompt asks for the buyer's voice, but it also tells the model to mention the platform, and the model sometimes turns that into listing language. This is one problem in one tool, not five separate ones. It is not a miss on any of my five criteria, which is why it appears here and not in the verdict table.

**A weak spot in my own evidence.** For criterion 3, the trace prints the selected item and the item passed to `suggest_outfit` from the same variable, so it could never have shown a mismatch. I added a separate check that intercepts what `suggest_outfit` really receives (15 of 15 ids matched). I also re-measured criterion 2's "0 model calls" per try, because the run log did not record it.

**How I checked my verdicts.** For each criterion I argued the opposite verdict as strongly as I could against my own results. None of the arguments changed a verdict. Two of them (criteria 3 and 5) showed that my evidence or my wording was weaker than it looked, which is what the extra checks and the criterion 5 revision above came from.



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

Command: `python app.py ask 'vintage graphic tee under $30' --trace`

```
[1] parse_query
      in:  'vintage graphic tee under $30'
      out: description='vintage graphic tee', size=None, max_price=30.0
[2] search_listings (via MCP)
      in:  description='vintage graphic tee', size=None, max_price=30.0
      out: 10 items: Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey, Y2K Baby Tee — Butterfly Print … +7 more
      →    branch: found, continuing
[3] select_item
      in:  10 results
      out: Graphic Tee — 2003 Tour Bootleg Style ($24.0, depop)
      →    took the first result
[4] suggest_outfit
      in:  item='Graphic Tee — 2003 Tour Bootleg Style', wardrobe items=10
      out: That graphic tee is a total find! Here are two ways to style it using your current wardrobe:  **Outfit 1 (Grun…
[5] create_fit_card
      in:  item='Graphic Tee — 2003 Tour Bootleg Style', outfit=598 chars
      out: Just scored this insanely good 2003 tour bootleg tee on depop for $24 and I'm obsessed. The fade on it is comp…
```

**Empty search**

Command: `python app.py ask 'designer ballgown size XXS under $5' --trace`

```
[1] parse_query
      in:  'designer ballgown size XXS under $5'
      out: description='designer ballgown', size='XXS', max_price=5.0
[2] search_listings (via MCP)
      in:  description='designer ballgown', size='XXS', max_price=5.0
      out: [] (empty)
      →    branch: empty, stopping
```

The run stops after two steps. `suggest_outfit` and `create_fit_card` never run, so the trace is visibly shorter than the happy path's five steps.

**Failure modes triggered on purpose**

I triggered each of the three failures myself and wrote down what the agent said.

1. **Empty search** — `python app.py ask 'designer ballgown size XXS under $5'`. The loop stopped before `suggest_outfit` (0 model calls) and said:
   > Nothing matched 'designer ballgown', size XXS, under $5. Try a more general description (for example 'tee' instead of a very specific style), a different size, or a higher price limit.

   No handler needed: the message names what was searched and three things to change.

2. **Empty wardrobe** — `python app.py ask 'vintage graphic tee under $30' --empty-wardrobe`. It did not crash and did not return an empty string; `suggest_outfit` returned general styling advice. The problem was that nothing told the user the advice wasn't built from their clothes, so I added a handler: `run_agent()` sets `session["notice"]` when the wardrobe has no items, and `app.py` prints it as a `Note:` line after the fit card:
   > Your wardrobe is empty, so these are general ideas rather than outfits built from your own clothes.

3. **Model unavailable** — I changed one character of the key in `.env`, ran a query with the cache off (`AI201_CACHE=0`), then restored the key. Before my handler, `app.py`'s catch-all printed a single line starting with `ModelUnavailable:`, which only a developer could act on. I added a `try/except ModelUnavailable` in `run_agent()`. It now says:
   > FitFindr couldn't reach its AI model right now, so it can't write outfit ideas. Please try again in a minute. If it keeps happening, tell whoever runs this app.
   > (For the developer: The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com.)

   It ended cleanly: no hang, no stack trace.

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

I rewrote one prompt: the instruction at the end of the prompt in `tools.py::create_fit_card`. The old version said to write as the buyer and "not as a seller listing it for sale". The new version keeps the buyer-voice instruction and adds three things: mention the platform only as where I bought it (for example "found it on Depop"); never say the item is listed, posted, for sale, or live on my own page or closet; never say I plan to resell it. The facts sent to the model (item, price, platform, outfit idea) did not change. This is the only code change in this milestone, and the old prompt text is kept as a comment above the new one.

**Which failure it was meant to fix:**

The seller-voice problem from my diagnosis. In the before run, 5 of 25 fit cards sounded like a seller instead of a buyer, for example "Now live on my Poshmark closet if anyone wants to grab it" and "Listed it over on my Poshmark if anyone else wants to steal my look". My likely mechanism was that the old prompt told the model to mention the platform, and the model sometimes turned that into listing language. None of my five criteria measured this, so I counted it with a fixed list of phrases (listed, listing, now live, my closet, my poshmark, my depop, posting it, post it, list this, for sale, resell, grab it, steal my look), fixed before the after run so it could not be tuned to the results. I also read each flagged caption: 2 of the 7 keyword hits ("my closet" meaning the buyer's own wardrobe) were not seller voice.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. A matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. An impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item matches the item passed to suggest_outfit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card mentions the item's price | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Empty vs non-empty wardrobe, 5 pairs | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Run with `python run_eval.py --label after` (cache off, 5 tries per scenario). The full output is in `results/run_2026-10-10_2225_after.md`.

**Before and after, side by side:**

| Measure | Before | After |
|---|---|---|
| Seller-sounding fit cards, keyword list | 7 of 25 | 0 of 25 |
| Seller-sounding fit cards, my reading | 5 of 25 | 0 of 25 |
| Fit cards with the correct price | 25 of 25 | 25 of 25 |
| Captions with "found it on [platform]" style wording | 1 of 25 | 6 of 25 |
| Average fit card length | 252 characters | 239 characters |
| Criteria met | 5 of 5 | 5 of 5 |

**Did it help, and how do I know:**

**Yes, on the problem it targeted, and nothing broke.** Seller-sounding fit cards went from 5 of 25 (7 by keyword) to 0 of 25, measured with the same fixed phrase list before and after, and I read all 25 after captions myself. All five criteria still passed 5 of 5, and the correct price is still in all 25 captions. If the old rate of 5 in 25 had stayed the same, getting 0 of 25 by chance would be about a 0.4% event, assuming the captions are independent.

**Limits:** this is one after-run of 25 captions at temperature 0.9, so some of the drop could be luck. I would call it "helped", not "fixed".

**Side effects:** the captions became more formulaic. The phrase "found it on [platform]" appeared in 6 captions instead of 1, so the model is copying the example I put in the prompt. One caption also misspelled the platform ("Pshmark"). With a single instance at temperature 0.9, I can't say my change caused it. A next step would be to remove the example from the prompt and re-measure.

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
