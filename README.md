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

FitFindr is an agent that helps a user evaluate a thrift listing based on a plain-language request such as "vintage graphic tee under $30." The agent parses the request into a description, optional size, and optional maximum price, then searches the listings data for matching items. If a match is found, it selects the best result, suggests one or two outfits using the user's saved wardrobe, and generates a short fit-card caption for the item. If no listing matches, the agent stops before calling the later tools and tells the user what they can change in the request.

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

- **What it does:** Searches the listings data for items that match a text description and can optionally filter by size and a maximum price.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None).
- **Returns:** A list of matching listing dictionaries, ordered with the best keyword match first. Each listing contains fields such as `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list `[]`, never `None` and never an exception.

### `suggest_outfit`

- **What it does:** Takes the selected thrift listing and the user's wardrobe and generates one or two outfit suggestions that show how the new item could be worn.
- **Inputs:** `new_item` (dict), `wardrobe` (dict).
- **Returns:** A non-empty string containing outfit suggestions. When wardrobe items are available, the suggestions should name pieces the user already owns.
- **When it has nothing:** If the wardrobe contains no items, it returns general styling advice for the selected item instead of returning an empty string or raising an exception.

### `create_fit_card`

- **What it does:** Takes an outfit suggestion and the selected thrift listing and generates a short post-style caption about the find.
- **Inputs:** `outfit` (str), `new_item` (dict).
- **Returns:** A two-to-four sentence caption that mentions the item, its price, its platform, and the specific style or vibe of the outfit.
- **When it has nothing:** If `outfit` is empty or contains only whitespace, it returns a descriptive fallback message instead of returning an empty string or raising an exception.

---

## Planning Loop

The planning loop lives in `agent.py` inside `run_agent()`. The agent first parses the user query into `description`, `size`, and `max_price`, then starts with the action `"search"`. After `search_listings()` runs, the loop checks the returned results: if the list is empty, it stores an error message in the session and stops immediately; if results exist, it stores the first result in `session["selected_item"]` and changes the next action to `"outfit"`.

The `"outfit"` step calls `suggest_outfit()` using `session["selected_item"]` and `session["wardrobe"]`, then stores the result in `session["outfit_suggestion"]` and moves to `"fit_card"`. The `"fit_card"` step calls `create_fit_card()` using the stored outfit suggestion and selected item, saves the result in `session["fit_card"]`, and returns the finished session. Each pass through the loop also calls `trace.check_iterations()` so the agent stops if the loop ever exceeds the configured maximum number of iterations.

**Branch rule:** If `search_listings()` returns an empty list, store a helpful message in `session["error"]` and stop without calling `suggest_outfit()`. Otherwise, store the first result in `session["selected_item"]` and continue to `suggest_outfit()`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** The query is parsed with Python regular expressions in `run_agent()`. Regex extracts an optional `under $...` maximum price and an optional `size ...`, then removes those phrases from the query to leave the search description.

**What moves through the session:** The parsed values go into `session["parsed"]`; search results go into `session["search_results"]`; the chosen listing goes into `session["selected_item"]`; `suggest_outfit()` writes its result to `session["outfit_suggestion"]`; and `create_fit_card()` writes the final caption to `session["fit_card"]`. If search returns nothing, the stop message goes into `session["error"]`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**
$ python app.py ask 'vintage graphic tee under $30'

Found: Y2K Baby Tee — Butterfly Print — $18.0 on depop

Outfit: Here are two outfits using the Y2K butterfly baby tee and pieces from your wardrobe:

### Outfit 1: Y2K Streetwear Contrast

Pair the fitted, feminine baby tee with relaxed denim and chunky footwear for a classic early 2000s street style look.

- **Top:** Y2K Butterfly Baby Tee
- **Bottoms:** Baggy straight-leg jeans (dark wash)
- **Outerwear:** Vintage black denim jacket
- **Shoes:** Chunky white sneakers
- **Accessories:** Black crossbody bag

### Outfit 2: Casual Earth-Tone Mix

Balance the pink and purple butterfly print with neutral, relaxed trousers for an effortless, everyday look.

- **Top:** Y2K Butterfly Baby Tee
- **Bottoms:** Wide-leg khaki trousers
- **Accessories:** Brown leather belt, Black crossbody bag
- **Shoes:** Chunky white sneakers

Fit card: Obsessed with this Y2K butterfly baby tee I just scored for only $18.00 over on my Depop! It gives off the absolute best early 2000s street style when paired with baggy dark-wash denim and a distressed black jacket. Grab it before I change my mind and keep it for myself! 🦋✨

0 model calls this session, 2 served from cache
**The three tools, tested one at a time**

$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[
{
'id': 'lst_002',
'title': 'Y2K Baby Tee — Butterfly Print',
'price': 18.0,
'size': 'S/M',
'platform': 'depop',
...
},
{
'id': 'lst_006',
'title': 'Graphic Tee — 2003 Tour Bootleg Style',
'price': 24.0,
'size': 'L',
'platform': 'depop',
...
},
...
]

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Here are two outfits utilizing the vintage Levi's 501 jeans and pieces from your wardrobe:

### Outfit 1: Clean Streetwear Minimal
Leans into the classic, effortless streetwear vibe of the 501s.

* **Bottoms:** Vintage Levi's 501 Jeans (Medium Wash)
* **Top:** White ribbed tank top (tucked in)
* **Outerwear:** Vintage black denim jacket (worn overtop)
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag

### Outfit 2: Casual Grunge Contrast
Plays with proportions by pairing a fitted base with an oversized cozy layer and chunky footwear.

* **Bottoms:** Vintage Levi's 501 Jeans (Medium Wash)
* **Top:** Oversized grey crewneck sweatshirt
* **Shoes:** Black combat boots
* **Accessories:** Brown leather belt (to cinch the waist)
```

```text
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Nothing beats the fit of broken-in vintage denim, and these medium wash Levi's 501s are the holy grail. Grab this classic pair over on my Depop for just $38 before I change my mind and keep them. Throw them on with a crisp white tee and retro sneakers for that effortlessly cool, off-duty streetwear look.
```

---

## How I Used AI

**Moment 1**

- _What I asked for:_ I asked for help implementing `search_listings` from the tool specification, including keyword scoring, size filtering, and the maximum-price filter.
- _What came back:_ The first pasted implementation caused an `IndentationError` because the implementation block was indented one level too far inside `search_listings`.
- _What I changed:_ I moved the entire implementation block left one indentation level, reran `python -m py_compile tools.py`, and then tested both a matching query and an impossible query to confirm the tool returned listing dictionaries or `[]` as specified.

**Moment 2**

- _What I asked for:_ I asked for help wiring the planning loop in `agent.py` so the next action depended on what `search_listings` returned.
- _What came back:_ The suggested loop used an `action` variable with `"search"`, `"outfit"`, and `"fit_card"` states, stored each result in the session, and stopped early when search returned an empty list.
- _What I changed:_ I tested the parser and branch separately before running the full loop, then verified that a matching query completed all three tools while an impossible query stopped with `session["fit_card"]` still set to `None`.
<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

| Criterion                            | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict   |
| ------------------------------------ | ------ | ----- | ----- | ----- | ----- | ----- | --------- |
| 1. matching query completes          | 4 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 2. impossible query stops early      | 5 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 3. selected item stays consistent    | 4 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 4. fit card includes listing details | 4 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 5. search respects maximum price     | 5 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```
File: results/run_2026-10-11_1943_before.md
Function: agent.py::run_agent

[1] search_listings (via MCP)
    in:  dict with keys: description, size, max_price
    out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
    in:  dict with keys: new_item, wardrobe
[3] create_fit_card
    in:  dict with keys: outfit, new_item

Found: Y2K Baby Tee — Butterfly Print — $18.0 on depop
```

---

## Verdicts and Diagnoses

| #   | Criterion                         | Target | Verdict   | How I decided                                                                                                                                          |
| --- | --------------------------------- | ------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | matching query completes          | 4 of 5 | MET — 5/5 | All five matching-query tries completed search, outfit suggestion, and fit-card creation.                                                              |
| 2   | impossible query stops early      | 5 of 5 | MET — 5/5 | All five impossible-query tries stopped after search returned `[]` and returned the helpful change-the-query message without calling `suggest_outfit`. |
| 3   | selected item stays consistent    | 4 of 5 | MET — 5/5 | In all five tries, the selected listing stayed the same item passed forward as `new_item` to `suggest_outfit`.                                         |
| 4   | fit card includes listing details | 4 of 5 | MET — 5/5 | All five fit cards included the selected item's price and platform and stayed within the 2–4 sentence target.                                          |
| 5   | search respects maximum price     | 5 of 5 | MET — 5/5 | All five price-limited searches returned only listings at or below the requested `$30` maximum.                                                        |

**Diagnoses**
All five criteria were met in the before run, so there was no failure mechanism to diagnose. The clearest target that was set too low was Criterion 1: it required 4 of 5 matching queries to complete all three tools, but the agent completed 5 of 5. A tighter target would be 5 of 5 matching queries completing all three tools and returning a fit card.

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
[1] search_listings (via MCP)
    in:  dict with keys: description, size, max_price
    out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
    in:  dict with keys: new_item, wardrobe
    out: Here are two outfits using the Y2K butterfly baby tee and pieces from your wardrobe: ### Outfit 1: Y2K Street…
[3] create_fit_card
    in:  dict with keys: outfit, new_item
    out: Obsessed with this Y2K butterfly baby tee I just scored for only $18.00 over on my Depop! It gives off the abs…

Found: Y2K Baby Tee — Butterfly Print — $18.0 on depop

Outfit: Here are two outfits using the Y2K butterfly baby tee and pieces from your wardrobe:

### Outfit 1: Y2K Streetwear Contrast
Pair the fitted, feminine baby tee with relaxed denim and chunky footwear for a classic early 2000s street style look.

* **Top:** Y2K Butterfly Baby Tee
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Outerwear:** Vintage black denim jacket
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag

### Outfit 2: Casual Earth-Tone Mix
Balance the pink and purple butterfly print with neutral, relaxed trousers for an effortless, everyday look.

* **Top:** Y2K Butterfly Baby Tee
* **Bottoms:** Wide-leg khaki trousers
* **Accessories:** Brown leather belt, Black crossbody bag
* **Shoes:** Chunky white sneakers

Fit card: Obsessed with this Y2K butterfly baby tee I just scored for only $18.00 over on my Depop! It gives off the absolute best early 2000s street style when paired with baggy dark-wash denim and a distressed black jacket. Grab it before I change my mind and keep it for myself! 🦋✨

0 model calls this session, 2 served from cache
```

**Empty search**

```
[1] search_listings (via MCP)
    in:  dict with keys: description, size, max_price
    out: [] (empty)

I couldn't find a matching listing. Try changing the description, size, or maximum price.

0 model calls this session
```

**On the MCP move:**
I moved `search_listings` behind the MCP boundary by registering it in `mcp_server.py` and changing `agent.py` to call it through `mcp_client.call_tool()` instead of importing and calling `search_listings()` directly. The returned search results behaved the same after the rewire: matching queries still returned listing dictionaries, impossible queries still returned `[]`, and the later outfit and fit-card steps continued to work. The trace now makes the boundary visible as `search_listings (via MCP)`.

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**
I added `ModelUnavailable` handling around `create_fit_card` in `agent.py`, matching the existing protection around `suggest_outfit`. If the final model call fails, the agent now records a useful error in the session, traces the failure, and returns cleanly instead of crashing.

**Which failure it was meant to fix:**
The before evaluation had no graded acceptance-criterion miss, so this was a robustness improvement rather than a score repair. It addresses the remaining failure mode where `suggest_outfit` was protected from model outages but `create_fit_card` was not.

### Run Log — After

| Criterion                            | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict   |
| ------------------------------------ | ------ | ----- | ----- | ----- | ----- | ----- | --------- |
| 1. matching query completes          | 4 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 2. impossible query stops early      | 5 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 3. selected item stays consistent    | 4 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 4. fit card includes listing details | 4 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |
| 5. search respects maximum price     | 5 of 5 | MET   | MET   | MET   | MET   | MET   | MET — 5/5 |

**Did it help, and how do I know:**
The before and after runs both met all five acceptance criteria at 5/5, so this change did not improve the measured acceptance scores. It did improve robustness by adding `ModelUnavailable` handling around `create_fit_card`, so a model failure at the final generation step can return a useful session error instead of crashing. The after evaluation confirmed that the normal path still passed all five criteria with no regression.

---

## What's Still Broken

All five graded acceptance criteria were met in the after run, so there are no remaining graded misses to diagnose. One limitation still exists outside those criteria: model availability can still affect generated outfit and fit-card content. I added `ModelUnavailable` handling around `create_fit_card`, but external API outages or invalid credentials can still prevent model-backed steps from completing. I stopped here because the measured criteria were all passing and the remaining risk depends on the external model service rather than the agent's control flow.
═════════════════════════════════════════════════════════════════════

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

```

```

```

```

```

```
