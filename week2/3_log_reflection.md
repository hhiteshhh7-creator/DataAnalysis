# 3. Check, log & reflection

AI session: **Claude Haiku 5.5 as Socratic coach**, 2026-11-11 ~22:20 PKT, after the `attempt-before-AI` commit (`70d9283`, 23:02). Raw file SHA-256: `5818dad3807487d4e37f764e5d19cbaa152edd78fe4db182eb28b087c8709d35`.

## The ASK, in words (no rows pasted)

I described the file to the AI as: one table, 97 data rows, six columns (`date`, `item`, `city`, `price`, `unit`, `data_label`); four items (Oil B, Pulse C, Staple A, Sweetener D) in two cities (City 1, City 2) over twelve weekly dates from 2026-01-01 to 2026-03-19; `price` is text with one blank, one quoted `"1,250"` and one `9999`; one item label is `oil b`; one exact duplicate row; and one missing item-city-date key. I asked for pandas cleaning code that would load, parse, de-duplicate and handle outliers.

I did not paste any rows into the prompt. The file is explicitly illustrative teaching data, not personal or confidential data.

## The AI draft, numbered

```python
def ai_clean_v1(path):
    df = pd.read_csv(path)                                              # L1
    df.columns = [c.strip().lower() for c in df.columns]                # L2
    df = df.dropna()                                                    # L3
    df["item"] = df["item"].str.strip().str.title()                     # L4
    df["price"] = df["price"].astype(str).str.replace(",", "", regex=False)  # L5
    df["price"] = pd.to_numeric(df["price"], errors="coerce")           # L6
    df = df.drop_duplicates()                                           # L7
    q1, q3 = df["price"].quantile([0.25, 0.75])                         # L8
    iqr = q3 - q1                                                       # L9
    df = df[df["price"].between(q1 - 3 * iqr, q3 + 3 * iqr)]            # L10
    df["date"] = pd.to_datetime(df["date"])                             # L11
    df["price"] = df.groupby(["item", "city"])["price"].transform(      # L12
        lambda s: s.fillna(s.mean())                                    # L13
    )
    return df.sort_values(["date", "item", "city"]).reset_index(drop=True)  # L14
```

## The trial, line by line

| Line | What it does | Verdict | Evidence (before → after) | Replacement / condition |
|---|---|---|---|---|
| **L1** | `read_csv(path)` | **keep** | Loads 97 × 6. No value change. The only trap is that `price` is inferred as text because of the comma and blank; that is L5–L6's job. | Keep. Record the shape and dtypes before trusting it. |
| **L2** | Strip/lower column names | **keep** | Columns are already lowercase and have no whitespace. Output identical. | Keep; harmless and idempotent. |
| **L3** | `dropna()` | **reject** | 97 → 96 rows immediately. The removed row is `Pulse C / City 2 / 2026-01-01`, a valid observation whose only problem is a blank price. In v1, Pulse C has 23 rows, not 24. | Retain the row, leave `price = NaN`; count missing values, do not delete observations. |
| **L4** | `.str.title()` on `item` | **reject** | Changes `oil b` → `Oil B`. v1 then has Oil B 25 and Staple A 22, and two rows at `2026-02-05 / Oil B / City 1` with prices **108.5** and **318.0**. The 108.5 price is less than half of the lowest Oil B value (298) and exactly fills the missing Staple A / City 1 cell (`114.5 − 6.0 = 108.5`). | Explicit map `oil b → Staple A`; verify the 4 × 2 × 12 key grid afterwards. |
| **L5** | `astype(str).str.replace(",", "")` | **keep** | Turns `"1,250"` into `1250`. The parse is syntactically correct; the value is still implausible. | Keep as a parsing step, but add a plausibility check immediately afterwards. |
| **L6** | `pd.to_numeric(errors="coerce")` | **keep** | `1,250` → `1250`; no exception. The remaining 96 values parse (the blank row was already dropped by L3). | Keep, but report how many were coerced. |
| **L7** | `drop_duplicates()` | **modify** | One exact duplicate exists (`row_id` 96 = `row_id` 8), so dropping one copy is right. But because L4 ran first, v1 still has a duplicate **key** at `2026-02-05 / Oil B / City 1` (108.5 and 318.0). | Run after the label fix; restrict to the six original columns; assert `duplicated(["date","item","city"]).sum() == 0`. |
| **L8** | `quantile([0.25, 0.75])` | **keep** (arithmetic) | Computes `Q1 = 151.0`, `Q3 = 304.0` on the pooled 95 rows. The arithmetic is right; the population is wrong because the four items sit at different price levels. | Keep the function, but apply it per item. |
| **L9** | `iqr = q3 - q1` | **keep** (arithmetic) | Pooled IQR = **153.0**. | Same condition: arithmetic only. |
| **L10** | Pooled 3×IQR row filter | **reject** | Pooled fences are **(−308.0, 763.0)**. It deletes the whole rows holding `1250` and `9999` (Staple A → 22 rows, Sweetener D → 23) and **leaves** the 108.5 Oil B contamination because 108.5 is inside the pooled fence. Per-item fences are Staple A (79.1, 143.9), Sweetener D (119.4, 216.5), Oil B (248.0, 403.8), Pulse C (184.5, 226.5). | Per-item 3×IQR flag: set only `price = NaN`; keep the row; report the fence and the raw value. |
| **L11** | `to_datetime` | **keep** | 12 distinct dates, 0 NaT, every gap exactly 7 days. | Keep; strict ISO parse is safe. |
| **L12** | `groupby(...).transform` | **reject** | In v1 it has nothing to fill only because L3 and L10 deleted every row with a missing/invalid price. On a retained row it replaces a missing price with the item-city mean, inventing a number and shrinking variance. | Do not impute. Retain `NaN` and report it. |
| **L13** | `lambda s: s.fillna(s.mean())` | **reject** | Same line as L12, treated as one logical step. It makes the table look complete (0 missing) by inventing the missing cells. | Keep missing values visible; the 3 missing prices are evidence, not a defect to hide. |
| **L14** | Sort / reset index | **keep** | Panel order changes; no value changes. | Keep. |

**Verdict count:** 8 keep, 1 modify, 4 reject (L12–L13 counted as one logical line).

## Before/after table

| Check | Raw | AI v1 | My clean |
|---|---:|---:|---:|
| Rows | 97 | **93** | **96** |
| Item labels | 5 | 4 | 4 |
| Item counts | Oil B 24, Pulse C 24, Staple A 24, Sweetener D 24, `oil b` 1 | **Oil B 25, Pulse C 23, Staple A 22, Sweetener D 23** | **24 each** |
| Unique keys | 96 (95 canonical + 1 spurious) | **92** | **96 canonical** |
| Duplicate key rows | 1 | **1** | **0** |
| Missing prices | 1 blank | **0** (rows deleted) | **3** (1 blank + 2 flagged) |
| Max price | 9999 | 350 | 350 |
| City 2 − City 1 | 44 × 6.0, plus 1147.0 and −9835.5 | not constant (Oil B contaminated) | **6.0 for all 45 complete pairs** |
| Staple A mean | 158.33 | 111.39 (22 rows) | **111.26** (23 prices) |
| Sweetener D mean | 576.81 | 167.15 | **167.15** |
| Oil B mean | 324.83 | **316.18** (25 rows) | **324.83** |

The raw means use the raw rows with `oil b` still separate. The v1 Staple A mean is close to the clean mean only because the 1,250 row was deleted; the observation is gone, which is the problem.

`describe()` on the pooled price column is identical for v1 and clean (93 numeric values, same mean and standard deviation), which is a useful warning: a summary statistic can hide a label swap. Only the item-level counts expose it.

## Exact change log (my clean)

| `row_id` | Date | Item (raw → clean) | Price (raw → clean) | Action |
|---:|---|---|---|---|
| 5 | 2026-01-01 | Pulse C → Pulse C | blank → `NaN` | Retained missing; do not impute |
| 17 | 2026-01-15 | Staple A → Staple A | `"1,250"` → `NaN` | Flagged invalid; row kept |
| 30 | 2026-01-22 | Sweetener D → Sweetener D | `9999` → `NaN` | Flagged sentinel; row kept |
| 40 | 2026-02-05 | `oil b` → **Staple A** | 108.5 → 108.5 | Label corrected with D5 evidence |
| 96 | 2026-01-08 | Staple A → dropped | 99.5 → — | Exact duplicate of `row_id` 8; dropped |

The notebook asserts that these are the **only** rows whose item or price changed. Every other cell is identical to the raw file.

## Verification after cleaning

- 96 rows = 4 items × 2 cities × 12 dates.
- 24 rows per item; 12 per item-city; 8 per date.
- 96 / 96 canonical keys; 0 duplicate keys; 0 unexpected labels.
- 3 missing prices: `row_id` 5 blank, `row_id` 17 flagged, `row_id` 30 flagged. The other 93 prices are numeric.
- Price range 99.5–350.
- City 2 − City 1 = 6.0 for all 45 complete pairs.
- Only `row_id` 5, 17, 30, 40 and 96 changed; the rest are identical.

## Log row

| Date/time | Tool & role | Asked | Kept / modified / rejected | Check |
|---|---|---|---|---|
| 2026-11-11 22:20 PKT | Claude Haiku 5.5, Socratic coach | Challenged the diagnosis in `1_attempt.md`; asked for a pandas cleaning draft to argue with | Kept L1, L2, L5, L6, L8, L9, L11, L14; modified L7; rejected L3, L4, L10, L12–L13 | Ran on a copy; before/after table; row-level diff; 96-key grid; per-item counts; constant 6.0 city gap |

## Reflection

### Which decisions differed from the AI's, and which was better supported?

The biggest difference was the `oil b` row. The AI read it as a capitalisation inconsistency and `.str.title()`-cased it to `Oil B`. I read it as a mislabeled **Staple A** observation. The evidence supports my reading: 108.5 is less than half of the lowest Oil B price (298); it exactly fills the only missing Staple A / City 1 cell; Staple A / City 1 moves 105.0 → 108.5 → 107.0 across those weeks; and the otherwise constant City 2 − City 1 gap of 6.0 gives `114.5 − 6.0 = 108.5`. The AI's version leaves Oil B with 25 rows, Staple A with 22 and a duplicate key at the same date. My version leaves 24 rows for each item and no duplicate keys. On that point my decision is better supported.

The second difference was **delete versus flag**. The AI's `dropna()` and pooled IQR filter removed three whole observations to produce a table with zero missing values. I kept the rows and set only the invalid prices to `NaN`. The assignment asks whether any valid observation was removed; the AI's answer is yes, three. Mine is no: the only row removed is the exact duplicate. That is the stronger audit position because the observation (the item-city-date) is still valid even when one cell is not.

The third difference was **imputation**. The AI filled missing prices with the item-city mean. I left them missing. The 6.0 city gap implies 109.0, 157.5 and 206.0 for the three gaps, but those are inferences from the paired city, not values observed in the row. With 23 of 24 prices present, a mean is not absurd either, but it would hide exactly the data-quality problem the audit is supposed to expose. The clean file should show three missing prices, not a smooth table that pretends they never existed.

The AI's code was better on brevity and on the mechanics of parsing: L5–L6 correctly handle the comma and the blank, L11 correctly parses the dates, and L14 sorts cleanly. I kept those lines. Its mistake was not syntax; it was treating a data-quality problem as a formatting problem.

### What would I have missed alone?

Probably the `oil b` row. A quick scan shows a lower-case label and the obvious fix is `.str.title()`. I would have caught the `9999` and the blank because they are visually loud, and I might have caught `1,250` because the comma stands out. The 108.5 row is quiet: it is a perfectly plausible **price**, just for the wrong **item**. It only becomes visible when you check the panel grid (one missing key, one extra key), the item-level price range (108.5 is not Oil B), and the city gap (114.5 − 6.0 = 108.5). The AI's draft was useful precisely because it made the wrong choice confidently; arguing with it forced me to find the evidence.

I would also have missed the duplicate at the end of the file. It is not next to its twin, so `head()` does not show it; only `duplicated()` or a key count does. And I would have missed the usefulness of the 6.0 city gap as a checking tool. Once I saw that 44 of 46 pairs were exactly 6.0, the two impossible gaps pointed straight at the two bad values. That check is now part of my cleaning routine.

### What I would keep from the AI's approach

The AI's draft was a useful first pass because it was complete and runnable. It gave me something concrete to argue with, instead of a blank page. The lesson is not "never use the draft"; it is "never keep a line you cannot explain." The lines I kept are the ones where I could state the before-and-after effect in one sentence. The lines I rejected are the ones where the code made a decision — delete, rename, trim, fill — that the data did not support.
