# 3. Check, log & reflection

AI session: Claude Haiku 5.5 as **Socratic coach**, 2026-11-04 ~22:23 (+0500), after the `attempt-before-AI` commit (`04dcc28`, 22:22).
Data dictionary (from `2_work.ipynb`): `date` (12 weekly dates, 2026-01-01 → 2026-03-19), `item` (Oil B, Pulse C, Staple A, Sweetener D), `city` (City 1, City 2), `price`, `unit` (only "PKR per kg (illustrative)"), `data_label` (ILLUSTRATIVE TEACHING DATA – NOT ACTUAL PBS OBSERVATIONS).

## Checked and rejected list — the five AI-proposed questions

| # | AI question (short) | Variables needed | In data? | Verdict | Check & evidence |
|---|---|---|---|---|---|
| 1 | Did Staple A in City 1 rise, fall or stay flat Jan→Mar? | date, item, city, price | Yes | **Accepted** | Monthly averages 103.0 → 110.0 → 114.3: it rose. |
| 2 | Which city had the higher average Oil B price in Q1, and is the gap bigger than the variation? | city, item, price, date (SD derived) | Yes | **Accepted** | City 2 at 327.8 vs City 1 at 321.8, a gap of 6.0. Within-city SD is 14.5, so the gap is smaller than the variation, though most of that SD is the upward trend. |
| 3 | By what % did Staple A in City B exceed City 1 in March, after unit conversion? | city "B", unit conversion factor | **No** | **Rejected** | There is no "City B" (cities are City 1 and City 2 only). The file has a single unit, so there is no conversion factor and none is needed. |
| 4 | Did Oil B and Staple A move in the same direction each month in each city? | item, city, price, date→month | Yes | **Accepted** | Both rose Jan→Feb and Feb→Mar in both cities (every item did). |
| 5 | What does a fixed basket cost in each city in March? | basket quantities/weights | **No** | **Rejected** | The data has no quantities or weights. A basket would mean inventing weights, which the data cannot support. |

Other coach points checked:
- "Are there several Oil B rows per city per month?" Yes: 5 in Jan, 4 in Feb and 3 in Mar (weekly dates). Averaging by month weights each week equally, and March has only 3 weeks (up to 19 Mar).
- "Is Oil B a grade or brand?" The data does not say. `item` is only a label.

## Sources the AI named (opened 2026-11-04)
- **Pakistan Bureau of Statistics, https://www.pbs.gov.pk.** The site is real and lists weekly *Sensitive Price Indicator* reports (latest: week ended 29-10-2026, posted 30/10/2026). The file is explicitly *not* PBS data, so I can't check it against PBS. PBS would only be the real source to use in place of this file.
- **FAO Food Price Index, https://www.fao.org/worldfoodsituation/foodpricesindex/.** The source is real but **not suitable**. It is a monthly *international* commodity index in points (Sept 2026 = 136.0, released 02/10/2026), not city-level PKR-per-kg retail prices. It can't test either claim.

## My two questions, tested (`2_work.ipynb`)
- Q1, Staple A / City 2 / Jan–Mar average ±10% of 350: average **114.2**, so **No**. Claim 1 as heard ("City B") is **untestable**, because the city isn't in the data.
- Q2, Oil B / City 1 / Jan–Mar average ±15% of 550: average **321.8**, so **No**.
- Predictions vs actual: shape (97, 6) vs **(96, 6)** (I counted the header line as a row); items 4 vs 4 ✓; dates 12 vs 12 ✓.

## Log row
| Date/time | Tool & role | Asked | Kept / rejected | Check |
|---|---|---|---|---|
| 2026-11-04 22:23 | Claude Haiku 5.5, Socratic coach | Challenge my 2 questions; 5 alternatives | Kept 1, 2, 4; rejected 3, 5 | Columns checked against the data dictionary; values recomputed in `2_work.ipynb`; both source links opened |

## Reflection

### What my data can say, and what it cannot (100 words)
My dataset has 96 rows of weekly illustrative prices for four items in two cities from January to March 2026. It can describe within-sample levels, the trends and observed city gaps for those four labels, and it can test whether a stated average falls inside a simple tolerance. It cannot support broad claims about real Pakistan Bureau of Statistics prices, a third city such as "City B", unit conversions, basket costs, causality, or prices outside the January–March window. Because the file is explicitly teaching data, every result is only an honest statement about the file, not about the actual market.

### Most tempting but unsupported AI suggestion (3 sentences)
The most tempting AI suggestion was the fixed-basket question, because a basket cost sounds like exactly the kind of policy number the assignment is asking for. It was unsupported: the file has prices but no quantities or weights, so any basket would require invented assumptions. I rejected it and kept only questions the columns could answer.
