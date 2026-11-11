# 1. Attempt (before AI)

## Snapshot

- File: `teaching_prices_messy.csv`
- SHA-256: `5818dad3807487d4e37f764e5d19cbaa152edd78fe4db182eb28b087c8709d35`
- Size: 11,256 bytes; 97 data rows + header; 6 columns.
- Git commit: `70d9283` (`attempt-before-AI`, 2026-11-11 23:02 +0500)
- The raw file is read-only in the notebook; every cleaning run uses `raw.copy()`.

## What the file looks like before any change

`head()` shows a six-column table: `date`, `item`, `city`, `price`, `unit`, `data_label`. The rows are weekly observations for four items (Staple A, Oil B, Pulse C, Sweetener D) in two cities (City 1, City 2). `price` arrives as text, not a number. `unit` is `PKR per kg (illustrative)` and `data_label` is `ILLUSTRATIVE TEACHING DATA - NOT ACTUAL PBS OBSERVATIONS`.

The first checks are structural: `shape`, `dtypes`, `value_counts()`, `isna().sum()`, `duplicated()` and a `pivot_table` of city gaps. The point is to diagnose before deleting, renaming or filling anything.

## Diagnosis

### D1 — `price` is text, and two cells are not plain numbers

**Code:**

```python
raw.dtypes
price_text = raw["price"].str.strip()
pd.to_numeric(price_text, errors="coerce").isna().sum()
(price_text == "").sum()
```

**Result:** `price` is `str`. One cell is blank (`Pulse C / City 2 / 2026-01-01`); one is `"1,250"` (`Staple A / City 2 / 2026-01-15`), which fails a strict parse because of the comma.

**Planned decision:** **correct** — strip the thousands comma, parse to float, keep blanks and non-numeric text as `NaN` (not zero).

**Reason:** `describe()`, means and plots are meaningless on text. Coercing silently would hide the two problem cells; I count them first and keep the raw text beside the parsed value.

### D2 — `1,250` is not a plausible price for Staple A

**Code:**

```python
pair = raw.pivot_table(index=["item", "date"], columns="city", values="price")
(pair["City 2"] - pair["City 1"]).dropna().value_counts()
```

**Result:** 44 of 46 complete pairs have a gap of exactly **6.0**. The two exceptions are `Staple A 2026-01-15` (gap **1147.0**) and `Sweetener D 2026-01-22` (gap **-9835.5**). Staple A / City 2 around that date: 105.5 (Jan 8) → **1,250** (Jan 15) → 112.5 (Jan 22). With City 1 at 103.0 and the usual 6.0 gap, the expected value is about 109.0.

**Planned decision:** **flag** — set the price to `NaN`, keep the row.

**Reason:** 1,250 is about 11× the local level. The 6.0 city gap implies 109.0, but that is an inference from the paired city, not a value observed in this row. I cannot tell whether the true value was 109.0, 112.5 or something else without inventing a number. Keeping the row preserves the observation and the 24-row item count; flagging the value records the doubt rather than overwriting it.

### D3 — `9999` is a sentinel, not a price

**Code:** same city-gap check; `raw["price"].describe()` (max 9999).

**Result:** Sweetener D / City 1 around 2026-01-22: 153.0 (Jan 15) → **9999** (Jan 22) → 162.0 (Jan 29). City 2 on Jan 22 is 163.5; with the usual 6.0 gap, City 1 should be about 157.5. 9999 is about 63× that.

**Planned decision:** **flag** — set to `NaN`, keep the row.

**Reason:** 9999 is a common missing-value sentinel. The 6.0 gap implies 157.5, but I flag rather than overwrite. Treated as a price, 9999 inflates the Sweetener D mean from about 167 to about 577. Dropping the row would lose a valid observation.

### D4 — one blank price

**Code:** `(raw["price"].str.strip() == "").sum()` → 1. The row is `Pulse C / City 2 / 2026-01-01`.

**Planned decision:** **retain** — keep the row, leave the price as `NaN`.

**Reason:** every other item-city has 12 weekly observations; dropping the row breaks the balanced panel for a fixable missing cell. The 6.0 gap implies 206.0 (City 1 is 200.0), but I leave it missing rather than overwrite it. Imputing the item-city mean would hide the gap and shrink the variance; with 23 of 24 prices present, leaving one missing is honest.

### D5 — `oil b` at 2026-02-05 / City 1 is a mislabeled Staple A observation

**Code:**

```python
raw["item"].value_counts()
# key grid: 4 items x 2 cities x 12 dates
set(expected_keys) - set(actual_keys)
set(actual_keys) - set(expected_keys)
```

**Result:** `Oil B` 24, `Pulse C` 24, `Staple A` 24, `Sweetener D` 24, `oil b` 1. `Staple A / City 1 / 2026-02-05` is the only missing canonical key; `oil b / City 1 / 2026-02-05` is the only unexpected key. Item ranges: Oil B 298–350; Staple A 99.5–122. The row is `oil b, City 1, 108.5`. That price is less than half of the lowest Oil B value (298), and it is exactly `114.5 - 6.0`: Staple A / City 2 on the same date is 114.5, and the normal City 2 − City 1 gap is 6.0. Staple A / City 1 runs 105.0 (Jan 29) → **108.5** (Feb 5) → 107.0 (Feb 12). Oil B / City 1 on Feb 5 is already present at 318.0.

**Planned decision:** **investigate, then correct** — relabel `oil b` to `Staple A`; do **not** title-case it to `Oil B`.

**Reason:** `.str.title()` would turn a label error into a price error: Oil B would have 25 rows (including 108.5), Staple A 22, and there would be two Oil B / City 1 / 2026-02-05 rows. The price level, the missing key and the constant 6.0 city gap all point to Staple A.

### D6 — exact duplicate row

**Code:** `raw.duplicated().sum()` → 1; `raw[raw.duplicated(keep=False)]`.

**Result:** `row_id` 8 and `row_id` 96 are both `2026-01-08, Staple A, City 1, 99.5` (`row_id` 96 is at the end of the file, not next to its twin).

**Planned decision:** **exclude** — drop one copy, keep the first.

**Reason:** it is an exact duplicate on all six columns. Keeping it would double-count 99.5 and, because Jan 8 is already present, it would mask the missing Feb 5 Staple A / City 1 cell.

### D7 — 5 item labels for 4 items; one missing key and one extra key

**Code:** key grid of 4 items × 2 cities × 12 dates = 96; `set(expected) - set(actual)` and the reverse.

**Result:** expected 96, actual 96 unique keys, but 95 canonical keys + 1 `oil b` key; `Staple A / City 1 / 2026-02-05` is missing and `oil b / City 1 / 2026-02-05` is extra.

**Planned decision:** **correct** — canonicalise to four labels; the extra key is the D5 row.

**Reason:** the panel is meant to be balanced. Until the label is fixed, the item counts cannot be checked (one item has 25 rows and one has 22 after naive title-casing).

### D8 — `date` is text

**Code:** `raw.dtypes`; `pd.to_datetime(raw["date"], format="%Y-%m-%d", errors="coerce").isna().sum()`.

**Result:** 0 unparseable; 12 distinct dates, 2026-01-01 → 2026-03-19; every gap is 7 days.

**Planned decision:** **correct** — parse to datetime with the ISO format.

**Reason:** dates must be sortable and comparable; the format is unambiguous, so a strict parse is safe.

### D9 — `unit` and `data_label` are constant

**Code:** `raw[["unit", "data_label"]].nunique()` → 1 each.

**Result:** unit `PKR per kg (illustrative)`; label `ILLUSTRATIVE TEACHING DATA - NOT ACTUAL PBS OBSERVATIONS`.

**Planned decision:** **retain** both.

**Reason:** they are not redundant data: the unit is needed to interpret `price`, and the label is the disclaimer that this is teaching data, not PBS data. Dropping them would make the file easier to misread.

### D10 — row order is not panel order

**Code:** `raw.head()` / `raw.tail()`; the duplicate is at the end.

**Result:** rows are roughly date-then-item order, but the duplicate is appended at `row_id` 96.

**Planned decision:** **retain** — sort by date, item, city; no value changes.

**Reason:** sorting is cosmetic but makes the panel and the duplicate visible.

## Checks that passed

- 97 data rows (header excluded), 6 columns.
- 2 cities only: City 1, City 2; no whitespace variants.
- 12 dates, all ISO, exactly 7 days apart; no duplicate dates.
- No negative or zero prices; no duplicate columns.
- `unit` and `data_label` are constant, as expected for a teaching extract.
- No other missing values: only `price` has a blank.

## Planned decision summary

| # | Problem | Decision | Rows affected | Value changed? |
|---|---|---|---:|---|
| D1 | price is text | correct | 97 | parse only |
| D2 | 1,250 outlier | flag | 1 | 1250 → NaN |
| D3 | 9999 sentinel | flag | 1 | 9999 → NaN |
| D4 | blank price | retain | 1 | blank → NaN |
| D5 | `oil b` mislabel | correct | 1 | item → Staple A |
| D6 | exact duplicate | exclude | 1 | drop copy |
| D7 | 5 labels / key gap | correct | 1 | via D5 |
| D8 | date text | correct | 97 | parse only |
| D9 | constant columns | retain | 0 | none |
| D10 | row order | retain | 0 | sort only |

## What I expect after cleaning

- 96 rows = 4 items × 2 cities × 12 dates.
- 24 rows per item; 12 per item-city; 8 per date.
- 0 duplicate keys; 0 missing keys.
- 3 missing prices (D2, D3, D4); all other 93 prices numeric.
- 1 label corrected; no other label or price changed without a flag.
- City 2 − City 1 = 6.0 for every complete pair.

## Snapshot hash

- SHA-256 of `teaching_prices_messy.csv`: `5818dad3807487d4e37f764e5d19cbaa152edd78fe4db182eb28b087c8709d35`
- Commit `attempt-before-AI`: `70d9283` (2026-11-11 23:02 +0500)
