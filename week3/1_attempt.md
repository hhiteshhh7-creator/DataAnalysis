# 1. Attempt (before AI)

## Snapshot

- File: `teaching_prices_clean.csv`
- SHA-256: `a298f754d72600bdf0552404ebe7a37530c8ca9dd8fc7178945f00f915eb45cd`
- Size: 11,148 bytes; 96 data rows + header; 6 columns.
- Git commit: `attempt-before-AI` · `ab4af76` (2026-11-18 21:40 +0500)
- The file is read-only; the notebook only reads it and writes no changes back to it.

## What the file contains

The file is a balanced weekly panel: four items (Oil B, Pulse C, Staple A, Sweetener D), two cities (City 1, City 2) and 12 Thursdays from 2026-01-01 to 2026-03-19. Every item has 24 rows (12 per city), every date has 8 rows, and all 96 prices are numeric. `unit` is constant (`PKR per kg (illustrative)`) and `data_label` is constant (`ILLUSTRATIVE TEACHING DATA - NOT ACTUAL PBS OBSERVATIONS`). There are no duplicate item-city-date keys and no missing prices.

## Statistics table (computed before AI)

Mean and median use all 24 observations for each item (12 weeks × 2 cities). First-to-last is computed on the item average across the two cities at each date, so the comparison is not a City 1 first week against a City 2 last week. The per-city check is in `2_work.ipynb`.

| Item | n | Mean | Median | Mean − median | First (avg of cities) | Last (avg of cities) | First-to-last % change |
|---|---:|---:|---:|---:|---:|---:|---:|
| Oil B | 24 | 324.83 | 324.00 | +0.83 | 301.0 | 347.0 | +15.28% |
| Pulse C | 24 | 205.92 | 206.00 | −0.08 | 203.0 | 210.5 | +3.69% |
| Staple A | 24 | 111.17 | 111.75 | −0.58 | 104.0 | 117.5 | +12.98% |
| Sweetener D | 24 | 166.75 | 166.75 | 0.00 | 152.0 | 181.5 | +19.41% |

Supplementary checks:

- Monthly averages rise from January to March for every item: Oil B 311.00 → 328.75 → 342.67; Pulse C 204.00 → 206.75 → 208.00; Staple A 106.00 → 113.00 → 117.33; Sweetener D 158.00 → 168.75 → 178.67.
- Oil B rises in all 11 weekly steps in both cities. Pulse C, Staple A and Sweetener D each have four or five down weeks, so their first-to-last numbers are endpoint comparisons, not smooth trends.
- City 2 − City 1 is exactly 6.0 PKR in all 48 complete item-date pairs. That is 5.40% of the Staple A mean and 1.85% of the Oil B mean, so the gap is not a 6% gap.
- A pooled mean across all 96 prices is 202.17 and the pooled median is 192.25. Neither is the price of a single item: Oil B averages 324.83 and Staple A 111.17.

## Which statistic I would report, and why

| Item | Statistic to report | Why |
|---|---|---|
| **Staple A** | **Median 111.75** for the level; **+12.98%** first-to-last for the change | The mean is 111.17, only 0.58 lower, so the level is not sensitive to the choice. The median is the safer "typical price" label because it is not moved by one extreme value. The change is confirmed by the monthly averages 106.00 → 113.00 → 117.33. |
| **Oil B** | **Mean 324.83** for the level; **+15.28%** for the change | The median is 324.00, a difference of 0.83. The series rises in every one of the 11 weekly steps in both cities, so the mean is not distorted by a few points and the first-to-last change is representative. |
| **Pulse C** | **Median 206.00** for the level; **+3.69%**, described as a small rise | The mean is 205.92, a difference of 0.08. The series has four down weeks in each city and only a 3.69% endpoint change; the median and the monthly averages are safer than a headline percentage. |
| **Sweetener D** | **Either; mean = median = 166.75**; **+19.41%** for the change | The two summaries are identical. The rise is confirmed by the monthly averages 158.00 → 168.75 → 178.67. |

The general rule I would apply: report the **median** when the question is "what is a typical price?" and the data may contain an extreme or flagged value; report the **mean** when the question is an average over the period and the data are clean and roughly symmetric. In this clean file the two agree within 0.83 PKR per item, so the choice does not decide the direction. The choice matters when a flagged value is present, which is the Week 2 check below.

## Week 2 flagged-value check

The Week 2 clean file has 23 numeric Staple A values, 23 Sweetener D values and 23 Pulse C values. Each item is missing one price: Pulse C is blank in the source, Staple A carries the flagged `1,250`, and Sweetener D carries the flagged `9,999`. Putting a flagged value back into its row gives the "with flag" column.

| Item | Clean n | Clean mean | Clean median | Flagged value | n with flag | Mean with flag | Median with flag | Mean change | Median change |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Staple A | 23 | 111.26 | 112.50 | 1,250 | 24 | 158.71 | 112.50 | +42.65% | 0.00% |
| Sweetener D | 23 | 167.15 | 167.50 | 9,999 | 24 | 576.81 | 167.75 | +245.08% | +0.15% |
| Pulse C | 23 | 205.91 | 206.00 | blank (no value) | 23 | 205.91 | 206.00 | n/a | n/a |

**Why the difference:** the mean uses the total, so one flagged value is divided across every observation. For Staple A the extra total is `1,250 − 111.26 = 1,138.74`; divided by 24 observations that adds `47.45` to the mean, giving `158.71`. The median uses position. With 24 values it is the average of the 12th and 13th values; the flag is the largest value, so the middle pair stays `112.5` and `112.5`, and the median does not move. The Sweetener D case is the same arithmetic: `(9,999 − 167.15) / 24 = 409.66`, so the mean becomes `576.81`, while the 24-value middle pair is `167.5` and `168.0`, giving `167.75`.

The lesson is not that the mean is wrong arithmetic. It is that a mean computed on data that still contain a flagged value is not a typical price: one value can move it by 42.65% or 245.08% while the median barely moves. That is exactly why the Week 2 file flagged the two values before computing averages.

## Draft referee's note (O / E / C labels)

**Label key:** **O** = observation that can be recomputed from the file; **E** = a possible explanation, explicitly not tested; **C** = a causal claim. Every sentence below carries one label.

> **Referee's note: did the staple price really go up?**
>
> **O1**: The teaching file contains 96 weekly observations: four items (Oil B, Pulse C, Staple A, Sweetener D), two cities, and 12 Thursdays from 1 January to 19 March 2026.
>
> **O2**: For the staple, Staple A, the mean is 111.17 PKR/kg and the median is 111.75; for Oil B, 324.83 and 324.00; for Pulse C, 205.92 and 206.00; for Sweetener D, 166.75 and 166.75.
>
> **O3**: For every item the mean and median differ by less than one PKR per kg, so in this clean file the two summaries agree on the level.
>
> **O4**: The item average rose from the first to the last week by 12.98% for Staple A, 15.28% for Oil B, 3.69% for Pulse C and 19.41% for Sweetener D.
>
> **O5**: The January, February and March averages also rise for all four items, so the first-to-last comparison is not created by one unusual pair of weeks.
>
> **E1**: A possible explanation is a common upward pressure on all four items, but the file contains no supply, policy, weather or demand variable, so this remains an untested explanation.
>
> **O6**: The mean and median separate sharply only when the Week 2 flagged values are restored: 1,250 in Staple A raises its mean from 111.26 to 158.71 while the median stays at 112.50, and 9,999 in Sweetener D raises its mean from 167.15 to 576.81 while the median moves only from 167.50 to 167.75.
>
> **O7**: The Week 3 clean file has 96 numeric prices, no missing values and no flagged values.
>
> **O8**: A pooled mean across all four items is 202.17, but that is not the price of any one item: Oil B averages 324.83 and Staple A 111.17.
>
> **O9**: The neighbours are therefore both describing the same clean series with summaries that agree; their disagreement becomes material only when an extreme value is left in the raw data.
>
> **O10**: On this evidence, the defensible statement is that all four illustrative item prices were higher in the week of 19 March than in the week of 1 January, with increases between 3.7% and 19.4%.
>
> **C1**: No causal claim is made: the file cannot show why prices moved.
>
> **O11**: These figures describe this teaching file only, not actual PBS prices.

The note deliberately contains no supported causal sentence. The only `E` sentence is labelled as an untested explanation, and the only `C` sentence is the explicit refusal to make a causal claim.

## What I will check when the AI interprets this

- Recompute every number in the AI's interpretation; a fluent sentence is not evidence.
- Mark each sentence green (recomputable), amber (explanation) or red (causal claim or data I was not shown).
- Check whether the AI treats the pooled mean as a typical item price; the pooled mean is a composition artefact.
- Check whether it turns the constant 6.0 PKR city gap into a 6% claim; those are different units.
- Keep my note. The AI may criticise it, but it does not replace it.

## Snapshot hash

- SHA-256 of `teaching_prices_clean.csv`: `a298f754d72600bdf0552404ebe7a37530c8ca9dd8fc7178945f00f915eb45cd`
- Commit `attempt-before-AI`: `ab4af76` (2026-11-18 21:40 +0500)
