# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. _"The agent handles errors"_ is an opinion.
_"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"_ is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; _"80% seemed reasonable"_ does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
I chose 4 out of 5 because `search_listings` uses keyword matching over short thrift listing text, so a reasonable query can still miss if its wording does not overlap well with the listing title, description, or style tags. I still expect the full three-tool path to complete most of the time when the data contains a relevant listing.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
I chose 5 out of 5 because this branch is controlled by the code, not by model wording. When `search_listings` returns an empty list, the loop should always stop before `suggest_outfit` is called and should return a clear message telling the user what they can change, such as the description, size, or price.

---

## 3. The selected listing stays consistent across tools

For at least 4 of 5 matching queries, `session["selected_item"]` has the same listing `id` as the `new_item` passed into `suggest_outfit`.

**Why this target:**
I chose 4 out of 5 because the purpose of session state is to carry the exact item found by `search_listings` into the next tool without asking the user again. Comparing the listing `id` makes that flow directly testable instead of assuming the right item moved through the loop.

---

## 4. The fit card includes the key listing details

For at least 4 of 5 matching queries, the final fit card mentions the selected item's price and platform and stays between 2 and 4 sentences.
**Why this target:**
I chose 4 out of 5 because `create_fit_card` uses a language model, so the exact wording can vary even when the input is the same. I still expect the important listing details to survive that variation, and the 2-to-4 sentence limit keeps the result short enough to read like a real post instead of a long product description.

---

## 5. Search respects the maximum price

For 5 of 5 queries that include a maximum price, every listing returned by `search_listings` has a `price` less than or equal to that maximum.
**Why this target:**
I chose 5 out of 5 because the price ceiling is a direct numeric filter in `search_listings`, not a subjective model judgment. If the user asks for something under a specific amount, returning an item over that amount would violate the request even if the item matched the description well.

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
