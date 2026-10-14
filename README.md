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
First, the user types a request, then the agent searches listings. If something matches, it takes the first item and comes back with an outfit and a fit card. The card names the item, the price, and the platform. If nothing matches, it stops and says what to change.


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

- **What it does:** 
Search listings.json by keyword overlap with a description, then optional size and price filters.
- **Inputs:** 
Description (str), size (str/None), max_price (float/None).
- **Returns:** 
List of listing dicts: title, price, size, and platform, best match first, at most SEARCH_RESULT_LIMIT entries. Each dict keeps above fields.
- **When it has nothing:**
Return [] (empty list).

### `suggest_outfit`

- **What it does:** 
One model call that suggests one or two outfits for a listing plus the user's wardrobe.
- **Inputs:**
new_item(dict, one_listing), wardrobe(dict with an items list, maybe empty)
- **Returns:**
a non-empty str
- **When it has nothing:**
if wardrobe["items"] is empty, still returns a stringof general styling advice.

### `create_fit_card`

- **What it does:**
One model call that writes a 2-4 sentence post caption.
- **Inputs:**
outfit(str), new_item(dict).
- **Returns:**
a str caption that names each item, its price, and its platform
- **When it has nothing:**
if outfit is empty or only whitespace, return a message

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

**Branch rule:**
If search_listings returns an empty list, put a message in the session and stop. Otherwise, take the first result and go to suggest_outfit.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex. `under $N` becomes max_price, `size X` becomes size, and the remaining words are the description.

**What moves through the session:** parsed goes in first, then search_results. The first hit becomes selected_item. suggest_outfit gets that same item and the wardrobe. create_fit_card gets the outfit string and the item, and the caption goes in fit_card. If search_results is [], error gets the message and the later fields stay None.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'looking for a vintage graphic tee under $30'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   **Outfit 1:**
- Y2K Baby Tee — Butterfly Print
- Baggy straight-leg jeans, dark wash
- Vintage black denim jacket
- Chunky white sneakers
- Black crossbody bag

**Outfit 2:**
- Y2K Baby Tee — Butterfly Print
- Wide-leg khaki trousers
- Brown leather belt
- Black combat boots
- Black crossbody bag

  Fit card: Scored the ultimate piece for my Y2K rotation. Paired this butterfly print baby tee with dark denim and chunky sneakers for pure 2000s mall-rat energy, though it looks just as good with khaki trousers and combat boots. Snagged it on depop for $18.0 and I’m genuinely obsessed.

0 model calls this session, 2 served from cache
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Outfit 1:
- Vintage Levi's 501 Jeans
- White ribbed tank top
- Vintage black denim jacket
- Black combat boots
- Brown leather belt
- Black crossbody bag
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Nothing beats that perfectly broken-in denim feel. Grabbed these vintage Levi's 501 jeans for $38.0 on depop and honestly, they're the only medium wash pair you'll ever need. Just throwing them on withcrisp white sneakers for that effortlessly cool 90s off-duty look.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked for search_listings to filter on description, size, and max price. If nothing matches, return [].
- *What came back:* search_listings was defined twice. The helpers were inside the first copy, and import re was missing. Running it raised NameError.
- *What I changed:* I kept one search_listings and put the helpers above it. I added import re. No match still returns [].

**Moment 2**

- *What I asked for:* I asked to fill in run_agent. An empty search should stop. A hit should save the item on the session, then call suggest_outfit and create_fit_card from there.
- *What came back:* The empty message was one long line made from the description, the size, and the price. check_iterations also ran before every step.
- *What I changed:* The message is one sentence now. It says to try different words, a different size, or a higher price. suggest_outfit and create_fit_card read selected_item and the wardrobe from the session.

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
| 3. Selected item id matches the id passed into suggest_outfit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Five fit cards name price and platform, and no two open with the same sentence | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Under $30 keeps every result price at or below 30 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

Criterion 1, try 1.

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- selected_item id: lst_002
- search result prices: 18.0, 24.0, 19.0, 26.0, 15.0, 22.0, 27.0, 20.0, 30.0, 12.0
- search_results: 10

Outfit suggestion:

Outfit 1: Pair the Y2K baby tee with the baggy straight-leg jeans, black combat boots, black cropped zip hoodie, and black crossbody bag.

Outfit 2: Style the Y2K baby tee with the wide-leg khaki trousers, brown leather belt, chunky white sneakers, and vintage black denim jacket.

Fit card:

Found this little butterfly print Y2K baby tee rummaging through the racks and honestly, I'm obsessed. It’s giving total early-2000s mall goth energy when you pair it with baggy straight-leg jeans, a cropped zip hoodie, and beat-up combat boots. Grabbed it on depop for $18.0 and I already know it’s going to be on heavy rotation.

Trace:

[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Outfit 1: Pair the Y2K baby tee with the baggy straight-leg jeans, black combat boots, black cropped zip hoodi…
      →    passed id lst_002
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Found this little butterfly print Y2K baby tee rummaging through the racks and honestly, I'm obsessed. It’s gi…


Criterion 2, try 1.

- stopped early: yes — Nothing matched. Try different words, a different size, or a higher price.
- selected_item: (none)
- selected_item id: (none)
- search result prices: 
- search_results: 0

Trace:

[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    branch: empty, stopping


Criterion 3, try 1. From results/run_2026-10-14_0046_before.md, "selected id matches passed id", lines 533–576. Produced by run_eval.py::write_report.

- selected_item: Knit Cardigan — Chunky Brown ($35.0, depop)
- selected_item id: lst_008

[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      →    passed id lst_008


Criterion 4. One fit card from each of the five items. Produced by tools.py::create_fit_card, logged by run_eval.py::write_report.

Flannel, line 778 and lines 801–803:
- selected_item: Oversized Flannel Shirt — Plaid Red/Black ($22.0, thredUp)
Scored this absolute unit of a red and black Oversized Flannel Shirt on thredUp for $22.0, and it’s basically my new uniform. Giving major 90s grunge coffee shop poetry vibes when worn open over a ribbed tank, but equally ready for a moody autumn campus stroll layered under a grey crewneck. It’s got that heavy, lived-in softness you usually have to dig through bins for hours to find.

Mary Janes, line 990 and lines 1014–1016:
- selected_item: Platform Mary Janes — Black Patent ($55.0, depop)
I’ve been hunting for the ultimate chunky ’90s shoe for months and these Platform Mary Janes finally ended the search. They add the exact right amount of glossy grunge to a baggy denim and white tank combo, but look just as good dialed back with khakis and a zip hoodie. Snagged them for $55.0 on depop and my fall wardrobe is officially complete.

Bucket hat, line 1197 and lines 1223–1225:
- selected_item: Bucket Hat — Reversible, Brown Plaid ($14.0, thredUp)
Found my new emotional support headwear. This reversible brown plaid bucket hat from thredUp was only $14.0 and literally ties together every 90s indie-sleaze and normcore look I've been trying to pull off. It goes from low-key solid tan to full grunge plaid depending on whether I'm leaning into crisp khaki minimalism or throwing on an oversized grey crewneck and combat boots.

Chelsea boots, line 1445 and lines 1471–1473:
- selected_item: Suede Chelsea Boots — Tan ($44.0, poshmark)
Scored these tan suede Chelsea boots and I’m already obsessed with how buttery the leather looks. Threw them on with baggy dark denim and an oversized grey crewneck for that effortless, slouchy silhouette, but they’re just as good with wide-leg khakis. Snagged them for $44.0 on poshmark and my fall wardrobe has officially peaked.

Silk dress, line 1677 and lines 1692–1694:
- selected_item: 90s Silk Slip Dress — Floral, Midi Length ($30.0, depop)
Stumbled upon this absolute gem and couldn't pass it up. I’m picturing this floral 90s silk slip dress layered right over baggy dark denim with an oversized crewneck for that ultimate grungy, effortless look. Grabbed it on depop for $30.0 and I'm honestly obsessed.


Criterion 5, try 1. From "under 30 prices", line 1869. Produced by run_eval.py::write_report.

- search result prices: 18.0, 22.0, 24.0, 20.0, 30.0, 12.0, 26.0, 24.0, 16.0, 29.0


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
| 1 | A matching query completes all three tools and returns a fit card | 4 of 5 | MET | All 5 tries called search_listings, suggest_outfit, and create_fit_card and returned a fit card. 5 of 5 meets 4 of 5. |
| 2 | An impossible query stops before suggest_outfit and names what to change | 5 of 5 | MET | All 5 tries stopped after search_listings and returned "Nothing matched. Try different words, a different size, or a higher price." |
| 3 | The id in session["selected_item"] is the id passed into suggest_outfit | 5 of 5 | MET | All 5 tries saved lst_008 and passed lst_008 into suggest_outfit. |
| 4 | For 5 different items, each fit card names that item's price and platform, and no two cards open with the same sentence | 5 of 5 | MET | On each of the 5 tries, all five cards name that item's price and platform, and the five opening sentences are different. |
| 5 | For a query that includes "under $30", every price in session["search_results"] is less than or equal to 30 | 5 of 5 | MET | All 5 tries kept every result price at or below 30. The highest price in each try is 30.0. |


**Diagnoses**

No criterion was missed. Criteria 2, 3, and 5 ask for 5 of 5, and those tries came out the same every time. Criterion 1 is the low target. I set 4 of 5 because search_listings is a plain keyword match, and a query can name a real listing and still miss the title, description, or tags. These five tries used "vintage graphic tee under $30", and that wording hit every time. I would tighten criterion 1 to 5 of 5 on this query.



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

```
python app.py ask 'vintage graphic tee under $30' --trace
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: **Outfit 1:** - Y2K Baby Tee — Butterfly Print - Baggy straight-leg jeans, dark wash - Vintage black denim jac…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Scored the ultimate piece for my Y2K rotation. Paired this butterfly print baby tee with dark denim and chunky…
```

**Empty search**

```
python app.py ask 'designer jeans size XXS under $5' --trace
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    branch: empty, stopping

  Nothing matched. Try different words, a different size, or a higher price.

0 model calls this session
```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->
```
run_agent now calls call_tool and the return value stayed a list.
```

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

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
