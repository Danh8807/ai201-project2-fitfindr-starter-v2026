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

- **What it does:** Returns listings that match the description, size, and price ceiling.
- **Inputs:** <!-- name and type each: `max_price` (float), not "a price" --> `description` (str), `size` (str or None), `max_price` (float or None)
- **Returns:**A list of full listing dicts (id, title, description, category, style_tags, size, condition, price, colors, brand, platform).
- **When it has nothing:** Returns `[]`. Never None, never an exception.

### `suggest_outfit`

-**What it does:** Takes one listing and the user's wardrobe and returns outfit ideas that pair the listing with wardrobe pieces.
- **Inputs:** `new_item` (dict, one listing), `wardrobe` (dict with an `"items"` key holding a list of dicts: id, name, category, colors, style_tags, notes which may be null)
- **Returns:** A list of outfit idea strings. Each names the new item and at least one wardrobe piece by its `name`.
- **When it has nothing:**Returns a list of general styling suggestions that mention no wardrobe pieces. Never an empty list.

### `create_fit_card`

- **What it does:** Writes a short caption someone would actually post.
- **Inputs:** `outfit` (str, one outfit idea), `new_item` (dict, one listing)
- **Returns:** One caption string, one to three sentences, mentioning the item.
- **When it has nothing:** If the model call fails or returns empty text, returns a fallback string built from the item's title and price. Never None. 

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

**Branch rule:**If `search_listings` returns an empty list, put a message in `session["message"]` naming what the user could change (raise the price limit, drop or change the size, use fewer description words), leave `session["fit_card"]` as None, and stop. Otherwise, store the first result in `session["selected_item"]` and call `suggest_outfit`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** With regex, in `agent.py::parse_query`. It pulls out a price ("under $30", "below 30", "up to 30"), then a size ("size M"), drops filler words, and treats what's left as the description. <!-- regex, string splitting, or asking the model — say which -->

**What moves through the session:** `query` → `parsed` (description, size, max_price) → `search_results` → `selected_item` → `outfit_suggestion` → `fit_card`. `error` is set only when the run stops early. Each tool reads its inputs back out of the session instead of taking the previous call's return value directly. <!-- which fields, in what order -->

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**
```
$ python app.py ask 'vintage graphic tee under $30, size M'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   **Outfit 1: Casual Y2K Streetwear**
Pair the Y2K Baby Tee — Butterfly Print with the **Baggy straight-leg jeans, dark wash** and the **Brown leather belt**.Layer the **Vintage black denim jacket** over top. Finish the look with the **Chunky white sneakers** and the **Black crossbody bag** for an effortless, throwback vibe.

**Outfit 2: Soft Contrast**
Style the Y2K Baby Tee — Butterfly Print tucked into the **Wide-leg khaki trousers**. Throw the **Black cropped zip hoodie** over your shoulders or wear it unzipped, and step into the **Black combat boots** to balance the cute butterfly graphic with an edgy, grounded footwear choice.

  Fit card: Obsessed with this Y2K baby tee with the cutest butterfly print. Found it for just $18 and knew I had to list it on Depop before I hoard all the early 2000s streetwear pieces. It’s giving total 90s-meets-aughts mall goth vibes depending on how you style it.

2 model calls this session, 476 prompt + 220 output tokens
```


**The three tools, tested one at a time**

`search_listings`

```
$ python -c "from tools import search_listings as s; print([x['id'] for x in s('graphic tee', 'M', 30)])"
['lst_002', 'lst_017']

$ python -c "from tools import search_listings as s; print([x['id'] for x in s('graphic tee', 'L', 30)])"
['lst_006', 'lst_033', 'lst_015']

$ python -c "from tools import search_listings as s; print(s('zzzzz', None, None))"
[]
```

`suggest_outfit` (example wardrobe, then empty wardrobe)

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
**Outfit 1: Casual Streetwear**
Pair the Vintage Levi's 501 Jeans with the White ribbed tank top tucked in. Layer the Oversized grey crewneck sweatshirton top, and finish with the Chunky white sneakers and Black crossbody bag for an effortless, classic look.

**Outfit 2: Edgy Contrast**
Style the Vintage Levi's 501 Jeans with the Black cropped zip hoodie and the Vintage black denim jacket for a cool double-denim moment. Accessorize with the Brown leather belt and anchor the outfit with the Black combat boots.

$ python -c "from tools import suggest_outfit; from utils.data_loader import load_listings; print(suggest_outfit(load_listings()[0], {'items': []}))"
**Outfit 1: Casual Streetwear**
Pair the 501s with an oversized graphic tee or a vintage band t-shirt in white, grey, or black. Layer with a distressed leather jacket or a boxy flannel. Finish with classic retro sneakers like Nike Dunks or Adidas Sambas.

**Outfit 2: Effortless Chic**
Tuck a fitted black ribbed tank top or a crisp, oversized white button-down shirt into the jeans. Add a brown leather belt to cinch the waist. Complete the look with black leather loafers, heeled ankle boots, and minimalist silver jewelry for a timeless, balanced vibe.
```

`create_fit_card` (cache off, three runs, then the empty-outfit case)

```
$ $env:AI201_CACHE = "0"
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Found these vintage Levi's 501 jeans on Depop for $38 and I am never taking them off. That perfect broken-in medium washgives off the ultimate effortless 90s indie sleaze vibe. Just need to style them with my beat-up white sneakers and a simple tee for the easiest everyday uniform.

(same command, run 2)
Found these vintage Levi's 501 jeans in the absolute best medium wash and couldn't pass them up. For $38, they have thatperfectly broken-in, effortless 90s vibe I've been hunting for. Just dropped them on my depop so someone else can live out their dream outfit of beat-up denim and crisp white sneakers.

(same command, run 3)
scored these vintage Levi's 501 jeans on depop for just $38 and they fit like an absolute dream. obsessed with the medium wash and that perfect 90s slouch. can't wait to style them with a basic tee and fresh white sneakers for that ultimate effortless look.

$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('   ', load_listings()[0]))"
No outfit was provided, so there's nothing to write a fit card about yet.
```

**State check** (the id in `selected_item` matches the first search result)

```
$ python -c "from agent import run_agent; from utils.data_loader import get_example_wardrobe as w; s = run_agent('graphic tee under `$30', w()); print(s['parsed']); print(s['selected_item']['id'], s['search_results'][0]['id'])"
{'description': 'graphic tee', 'size': None, 'max_price': 30.0}   <- paste what YOUR run printed with the backtick
lst_002 lst_002
```

**Known issues noticed during the build** (for next unit's testing):
- The model sometimes glues words together ("washgives", "thatperfectly", "sweatshirton", "hoodieon").
- Some fit cards read as if the writer is *selling* the item ("list it on Depop", "just dropped them on my depop"). They still mention item, price and platform, so a property check would pass them.


## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked Claude to write my Milestone 2 tool specs and branch rule, then pasted my data fields and wardrobe schema so it could tighten them.
- *What came back:* A spec where `suggest_outfit` returned a list of strings, `search_listings` required every description word to match, `create_fit_card` was 1 to 3 sentences, and the branch wrote to `session["message"]`. When I pasted `tools.py` and `agent.py`, the starter said otherwise: a single string, keyword-overlap scoring with zero scores dropped, 2 to 4 sentences, and `session["error"]`.
- *What I changed:* I rewrote the Tool Inventory and branch rule to match the starter, and I made size matching token-based so "M" matches "S/M" but "S" can't match "US 9". I also corrected my fit card criterion to 2 to 4 sentences.

**Moment 2**

- *What I asked for:* I ran `create_fit_card` three times on the same item to check that the captions varied.
- *What came back:* Three word-for-word identical captions.
- *What I changed:* I found `CACHE_ENABLED = os.getenv("AI201_CACHE", "1") != "0"` in `config.py`, so the starter was returning a saved answer for the identical prompt. I set `$env:AI201_CACHE = "0"` for the test and got three different captions. I turned caching back on while building the loop so repeat runs were free.


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
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

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
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



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

```

**Empty search**

```

```

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
