# 3. Check, log & reflection

AI session: **Claude Haiku 5.5 as Interpretation Auditor**, 2026-11-18 ~22:05 PKT, after the `attempt-before-AI` commit (`ab4af76`, 2026-11-18 21:40 +0500). Data file SHA-256: `a298f754d72600bdf0552404ebe7a37530c8ca9dd8fc7178945f00f915eb45cd`.

## The ASK, in words (no rows pasted)

I pasted the statistics table from `1_attempt.md`: four items, 24 observations each, with mean, median, first and last values, and first-to-last percentage change; the Week 2 flagged-value comparison for Staple A and Sweetener D; and the statement that the file is illustrative teaching data. I asked Claude Haiku 5.5 to interpret the table as a referee would: what should the two neighbours conclude, and which statistic is the better headline. I did not paste the draft note itself and did not ask it to write the note.

## The AI interpretation, numbered

> 1. All four item averages were higher in the last week than in the first week.
> 2. Staple A rose 12.98%, Oil B rose 15.28%, Pulse C rose 3.69% and Sweetener D rose 19.41%.
> 3. For every item, the mean and median differ by less than 1 PKR.
> 4. The mean is always the typical price because it uses every observation.
> 5. Restoring the 1,250 flagged value raises Staple A's mean from 111.26 to 158.71 but leaves the median at 112.50; restoring the 9,999 flag raises Sweetener D's mean from 167.15 to 576.81 while its median moves only from 167.50 to 167.75.
> 6. The median is more robust than the mean when the data contain an extreme value.
> 7. Pulse C's 3.69% rise proves that demand for pulses is stagnant.
> 8. City 2 is exactly 6.0 PKR above City 1 in every complete pair.
> 9. The 6.0 city gap is caused by transport costs.
> 10. The rise was caused by a supply shock that hit all four items.
> 11. The data are illustrative teaching data, not actual PBS observations.
> 12. The first-to-last change is the most reliable trend measure because it uses the whole period.
> 13. The result is consistent with the PBS Sensitive Price Indicator for the week ended 29 October 2026.
> 14. Because City 2 is 6.0 PKR higher, City 2 is 6.0% more expensive.

## Colour audit

**Legend:** **green** = I can recompute it from the statistics table or the file; **amber** = an explanation or methodological assertion, not a direct observation; **red** = a causal claim or a claim about data I was not shown.

| # | AI sentence (short) | Colour | How checked | Verdict |
|---:|---|---|---|---|
| 1 | All four item averages were higher in the last week than in the first week. | **green** | Recomputed in `2_work.ipynb` §2: item averages 104.0 → 117.5 (Staple A), 301.0 → 347.0 (Oil B), 203.0 → 210.5 (Pulse C), 152.0 → 181.5 (Sweetener D). All four differences are positive. | **Accepted.** |
| 2 | Staple A +12.98%, Oil B +15.28%, Pulse C +3.69%, Sweetener D +19.41%. | **green** | 117.5 / 104.0 − 1 = 12.98%; 347.0 / 301.0 − 1 = 15.28%; 210.5 / 203.0 − 1 = 3.69%; 181.5 / 152.0 − 1 = 19.41%. Exact match. | **Accepted.** |
| 3 | For every item, mean and median differ by less than 1 PKR. | **green** | `2_work.ipynb` §2: Oil B +0.83, Pulse C −0.08, Staple A −0.58, Sweetener D 0.00. Maximum absolute difference = 0.83. | **Accepted.** |
| 4 | The mean is always the typical price because it uses every observation. | **amber** | Not a data observation; a methodological assertion. The “uses every observation” part is true, but the conclusion is not. Week 2 §4 is the counterexample: with the 1,250 flag, Staple A's mean is 158.71 while its median is 112.50; with the 9,999 flag, Sweetener D's mean is 576.81 while its median is 167.75. | **Rejected** as a general rule. The mean uses every value, and that is exactly why one extreme value can move it. |
| 5 | 1,250 raises Staple A's mean to 158.71 but leaves the median at 112.50; 9,999 raises Sweetener D's mean to 576.81 while its median moves to 167.75. | **green** | Recomputed in `2_work.ipynb` §4: clean means 111.261 and 167.152; with flags 158.708 and 576.812; medians 112.50 → 112.50 and 167.50 → 167.75. Mean shifts +42.65% and +245.08%; median shifts 0.00% and +0.15%. | **Accepted.** |
| 6 | The median is more robust than the mean when the data contain an extreme value. | **amber** | An explanation, not an observation. It is correct about robustness: the median is positional, so the flag at the top barely moves the middle. It is not a reason to ignore the mean when the question is a total or a basket cost. | **Accepted with a condition:** use the median for a “typical price”; use the mean or a weighted total for a total-cost question. |
| 7 | Pulse C's 3.69% rise proves that demand for pulses is stagnant. | **red** | Causal claim about demand, which is not a column in the file. It also contradicts the table: Pulse C rose 3.69%, and its monthly average rose 204.00 → 206.75 → 208.00. “Proves” is too strong even if demand data existed. | **Rejected.** No demand variable; the direction is positive, not stagnant. |
| 8 | City 2 is exactly 6.0 PKR above City 1 in every complete pair. | **green** | `2_work.ipynb` §5: 48 complete item-date pairs; one distinct gap, 6.0. | **Accepted.** |
| 9 | The 6.0 city gap is caused by transport costs. | **red** | Causal claim. There is no transport, distance or cost variable in the file; the columns are `date`, `item`, `city`, `price`, `unit`, `data_label`. | **Rejected.** |
| 10 | The rise was caused by a supply shock that hit all four items. | **red** | Causal claim. There is no supply, weather, policy or demand variable in the file. | **Rejected.** |
| 11 | The data are illustrative teaching data, not actual PBS observations. | **green** | `data_label` is constant across all 96 rows: `ILLUSTRATIVE TEACHING DATA - NOT ACTUAL PBS OBSERVATIONS`; `unit` is `PKR per kg (illustrative)`. | **Accepted.** |
| 12 | The first-to-last change is the most reliable trend measure because it uses the whole period. | **amber** | Methodological assertion; checked against `2_work.ipynb` §3. First-to-last uses only the first and last weeks, not the whole path. Pulse C, Staple A and Sweetener D each have four or five down weeks. The monthly averages are a better check. | **Rejected.** It is an endpoint comparison, not a trend measure. |
| 13 | The result is consistent with the PBS Sensitive Price Indicator for the week ended 29 October 2026. | **red** | Refers to data I was not shown; no PBS series is in the statistics table. The file's own label says it is not PBS data. | **Rejected.** |
| 14 | Because City 2 is 6.0 PKR higher, City 2 is 6.0% more expensive. | **green** (the 6.0 is recomputable) | 6.0 / 111.17 = 5.40% for Staple A; 6.0 / 324.83 = 1.85% for Oil B; 6.0 / 205.92 = 2.91% for Pulse C; 6.0 / 166.75 = 3.60% for Sweetener D. The 6.0 is PKR, not percent. | **Rejected.** The number is right in PKR and wrong in percent. |

**Tally:** 14 sentences: 7 green, 3 amber, 4 red. I accepted 6 green claims and 1 amber claim (conditionally); I rejected 1 green claim (wrong units), 2 amber claims and all 4 red claims.

## Every number recomputed

| Number in the AI text | My recomputation | Match? |
|---|---|---|
| 12.98% | 117.5 / 104.0 − 1 = 12.98% | yes |
| 15.28% | 347.0 / 301.0 − 1 = 15.28% | yes |
| 3.69% | 210.5 / 203.0 − 1 = 3.69% | yes |
| 19.41% | 181.5 / 152.0 − 1 = 19.41% | yes |
| less than 1 PKR | max absolute mean − median = 0.83 (Oil B) | yes |
| 111.26 → 158.71 | 23-value mean 111.261; with 1,250: 158.708 | yes |
| 112.50 unchanged | middle pair of 24: 112.5 and 112.5 | yes |
| 167.15 → 576.81 | 23-value mean 167.152; with 9,999: 576.812 | yes |
| 167.50 → 167.75 | middle pair of 24: 167.5 and 168.0 | yes |
| 6.0 PKR | 48 / 48 complete pairs; distinct gap 6.0 | yes |
| 6.0% | 5.40% Staple A, 1.85% Oil B, 2.91% Pulse C, 3.60% Sweetener D | **no** |

The last row is the important one: a correct PKR gap became a false percentage claim. The audit kept the 6.0 PKR gap (green, accepted) and rejected the 6.0% sentence (green number, wrong unit).

## The confused classmate (excerpt)

I asked Claude Haiku 5.5 to play a classmate who believes “the mean is always the typical price” and keeps asking “but why?”. This is the excerpt after the classmate's first two objections started to repeat.

> **Classmate:** The mean is the average, so it is always the typical price. Why would anyone use the median?
>
> **Me:** The mean is the total divided by the count. It is the typical price only when the values are balanced around it. In the clean Week 3 file, Staple A's mean is 111.17 and its median is 111.75, so both are typical. In the Week 2 file, the flagged 1,250 raises the mean to 158.71 while the median stays 112.50.
>
> **Classmate:** But why does one value move the mean so much?
>
> **Me:** Because the mean adds every value to the total. The flag is 1,138.74 above the clean mean; spread over 24 observations, it adds 47.45 to the mean. The median only looks at the middle position. With 24 values it is the average of the 12th and 13th; the flag lands at the top, so the middle pair stays 112.5 and 112.5.
>
> **Classmate:** So the mean is wrong?
>
> **Me:** The arithmetic is right; the label “typical price” is wrong. The mean answers “what is the total divided by the count?” The median answers “what is the middle value?” If you want a typical price and the data may contain an extreme value, the median is safer. If you want the total cost of a basket, the mean or a weighted total is the right building block.
>
> **Classmate:** But why not just always use the mean?
>
> **Me:** Because a single flag can dominate it. The 9,999 in Sweetener D moves the mean from 167.15 to 576.81, a 245% increase, while the median moves from 167.50 to 167.75. A statistic that can be moved that far by one value is not a safe headline for raw data.
>
> **Classmate:** So which one should the neighbour quote?
>
> **Me:** For the clean Week 3 file, either: the mean and median differ by at most 0.83 PKR per item. For the dirty Week 2 file, quote the median until the 1,250 is handled. The neighbours are not arguing about the data; they are arguing about what “typical” means.
>
> **Classmate:** That makes sense: the mean is an average, not automatically the typical price.

The classmate stopped repeating “but why” once the distinction was tied to a concrete number: 47.45 is the amount the 1,250 adds to the mean, and 112.5 is the middle that does not move. That was the clearest way to show that the mean is a total-based summary, not a definition of “typical”.

## Reflection

### Where did my first draft say more than the data allowed?

The first draft's headline, “did the staple price really go up?”, was broader than the file. The file is explicitly illustrative teaching data, not PBS data, and the 12.98% is a comparison of two weekly averages inside this file. The first draft did say “teaching file” in O1 and “not actual PBS prices” in O11, but the headline and the phrase “the price rose” could still be read as a claim about the market. I kept the number and tightened the surrounding language: O10 now says “all four illustrative item prices” and O11 limits the note to this teaching file.

The second overclaim was treating first-to-last as a trend. The first draft used the endpoint percentages as the main change measure. For Pulse C the path has four down weeks and the total is only +3.69%; calling that “the increase” suggests a smooth movement that the weekly path does not show. I added O5, the monthly averages, and I now describe the number as an endpoint comparison. The monthly check supports the direction for all four items, but it also shows that Pulse C's rise is much weaker than the headline number suggests.

The third overclaim was saying the neighbours' summaries “agree” without limiting that to the clean file. In the Week 2 file they do not agree: the 1,250 moves Staple A's mean by 42.65% and the 9,999 moves Sweetener D's mean by 245.08%, while the medians barely move. I added “in this clean file” to O3, and I limited the Week 2 and clean-file claims in O6 and O7. The general statement “the choice between mean and median does not matter” would have been false; the correct statement is that it does not matter here, after cleaning.

The fourth overclaim was scope. “The mean and median differ by less than one PKR” is item-level. If I pooled all 96 prices, the mean is 202.17 and the median is 192.25, a difference of 9.92. That pooled number is not a typical staple price because Oil B is roughly three times Staple A. I made the item-level scope explicit and added O8. This was the most tempting overclaim: a single “average staple price” sounds like exactly the headline the neighbours want.

The draft did not claim causation, and I kept that discipline. The AI's causal sentences (transport costs, a supply shock, stagnant demand) were rejected. The only E sentence remains labelled as an untested explanation. The C label is a refusal, not a causal claim; I considered writing a causal sentence as an example and rejected it because a referee's note should not assert what the file cannot test.

### What I would keep from the AI's approach

The AI was useful as an auditor. It spotted the direction, the robustness point and the 6.0 PKR gap, and its arithmetic matched mine on the green claims. The classmate role was more useful than the referee role: it forced me to explain *why* the mean moves, not just *that* it moves. The number 47.45, the flag's contribution to the mean, did more explanatory work than a definition of “average”. I kept the green numbers, the conditional median point and the 6.0 PKR gap. I did not keep the red causal claims, the “always typical” rule or the 6.0% error.

### What I would do differently

Next time I would put the O/E/C label beside each number in the statistics table, not only in the note. That would make the colour audit faster and would show at a glance which numbers are observations and which are interpretations. I would also put the Week 2 flagged-value rows in the same table as the clean Week 3 rows, so the contrast between 111.26/112.50 and 158.71/112.50 is visible at the point of drafting. The 1,250 and 9,999 sensitivity is the clearest evidence for the note's argument, and it should sit next to the claim it supports.

## Log row

| Date/time | Tool & role | Asked | Kept / modified / rejected | Check |
|---|---|---|---|---|
| 2026-11-18 22:05 PKT | Claude Haiku 5.5, Interpretation Auditor | Interpret the statistics table; say what the two neighbours should conclude and which statistic is the better headline | Kept 1, 2, 3, 5, 8, 11; accepted 6 conditionally; rejected 4, 7, 9, 10, 12, 13, 14 | Recomputed every number in `2_work.ipynb`; coloured 14 sentences; checked the O/E/C labels in `1_attempt.md` |
| 2026-11-18 22:35 PKT | Claude Haiku 5.5, confused classmate | Play a classmate who believes the mean is always the typical price; keep asking “but why?” | Kept the explanation structure and the worked numbers; no data changes | Transcript excerpt above; every number checked against `2_work.ipynb` §4 |

## Bottom line

The clean Week 3 file has 96 prices and no missing values. Mean and median agree within 0.83 PKR per item, so the two neighbours are not disagreeing about the data: they are using different definitions of “typical”. The disagreement becomes real only when a flagged value is present. The Week 2 1,250 raises Staple A's mean by 42.65% but leaves its median unchanged; the 9,999 raises Sweetener D's mean by 245.08% while its median moves 0.15%. The defensible note reports item-level statistics, uses the median for a typical price in dirty data, uses the mean for a clean average, and makes no causal claim.
