# Anonymity Lab

A single-file, dependency-free web app for **statistical disclosure control**. Load a CSV,
mark which columns are quasi-identifiers and which are sensitive, and it will anonymize the
data to a k-anonymity / l-diversity / t-closeness budget — then show you exactly what that
cost in privacy and what it cost in data utility.

**Everything runs in the browser.** No upload, no server, no build step, no dependencies.
The file is the whole application.

## Run it

Press **Load sample dataset** to see the whole pipeline run on a synthetic adult-style
table, or drop in your own CSV. No other setup exists — the file *is* the application.

```bash
# hosted:      https://<user>.github.io/<repo>/
# local check: python3 -m http.server 8000   (then open http://localhost:8000)
# it also works straight off disk: xdg-open index.html
```

## Deploy to GitHub Pages

Push this directory to a repository and turn Pages on. The app is a single static file with
no build step, no dependencies, no absolute paths and no external requests, so it works from
the repo root or from a `/docs` folder, at any URL depth.

**Option A — publish the repo root (simplest)**

1. `git init && git add index.html README.md .nojekyll && git commit -m "Anonymity Lab"` and push.
2. Repo → **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
   branch `main`, folder `/ (root)` → Save.
3. The site appears at `https://<user>.github.io/<repo>/` within a minute.

**Option B — publish from `/docs`**

1. `mkdir docs && cp index.html docs/`
2. Same as above, but pick folder `/docs`.

`.nojekyll` is already included so Pages serves the file verbatim instead of running it
through Jekyll, and the favicon is an inline SVG data URI, so there are no extra asset
requests to 404. `Settings → Pages` also lets you set a custom domain; nothing in the page
hardcodes a hostname.

**Sharing a configuration.** *Copy settings link* puts the whole privacy budget, the
algorithm, the t-closeness metric, the repair toggles and the column roles into the URL
fragment. Anyone opening that link gets the identical configuration and only has to load
their own file — the dataset itself is never part of the link. Values that are missing or
out of range in a URL are ignored rather than clamped, so a hand-edited link cannot quietly
weaken someone's k / l / t budget.

## What it does

1. **Load** — CSV/TSV/semicolon files, quoted fields, embedded commas and newlines, BOM,
   optional header row, type detection (numeric / categorical / date) with a manual override.
2. **Classify** — tag each column as quasi-identifier (QI), sensitive attribute (SA), or ignore.
3. **Budget** — set k, l, t; pick the algorithm and the repair stages.
4. **Audit** — get a pass/fail verdict per criterion, before → after metrics, charts, the
   anonymized table with every changed cell highlighted, the equivalence-class listing, an
   adversary walk-through, and CSV / JSON export.

## Algorithms

| Criterion | How it is enforced | How it is measured |
|---|---|---|
| **k-anonymity** | full generalization (global lattice search), Mondrian multi-dimensional partitioning, or hybrid; plus record suppression | min/avg/median class size, records in classes < k, singleton rows, re-identification rate, class-size entropy |
| **l-diversity** (full) | **data swapping** of sensitive values between classes — class sizes, therefore k-anonymity, are untouched by construction | min/avg/max distinct sensitive values per class, % of violating classes, per-class normalized entropy |
| **t-closeness** | greedy, utility-preserving single-value repair toward the global distribution, respecting l-diversity | worst and mean class↔global distance, % of violating classes, per-class histogram |

Generalization hierarchies are built automatically: power-of-two interval widths for numeric
attributes, frequency-chunked group hierarchies for categorical ones. t-closeness is measured
with the **t-closeness hierarchy** distance (Elfakam et al. 2008) by default, with
Earth Mover's Distance and total variation selectable.

**Utility metrics**: normalized information loss (cell precision for QIs, per-row distortion
for edited sensitive values), suppression rate, and per-attribute cardinality reduction.

The **k-sweep** tab re-runs the entire pipeline across a range of k and plots the classic
privacy/utility trade-off curve.

## Honesty notes

* If a target cannot be met — for example l is larger than the number of distinct values an
  attribute has, or a class is 100% homogeneous so no swap can help — the app says so and
  explains why instead of silently passing. Classes that cannot reach l are released unchanged
  by default; suppressing them is opt-in, because it can delete most of the table.
* The report is a measurement, not a certificate. Treat the numbers as evidence for a decision,
  and read the "What was actually done to the data" section at the bottom of the report.

## Limits

* Up to 50,000 rows (in-browser). 20k rows takes ~2 s, 50k ~4 s; the tab blocks while it works.
* The t-closeness repair is a greedy heuristic — it finds a minimal-damage repair, not a
  provably minimal one.
* Full generalization collapses badly on sparse cross-products (that is a property of global
  generalization, not a bug — the log says so and Mondrian usually wins on utility).

## References

Sweeney (2002) k-anonymity · Machanavajjhala et al. (2007) l-diversity · El Emam et al. (2007)
t-closeness · Elfakam et al. (2008) the t-closeness hierarchy · Mondrian multi-dimensional
de-identification · NIST SP 800-188.
