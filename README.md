# Bland-Altman Agreement Analysis: Worked Examples

A step-by-step Jupyter notebook for the standalone performance analysis of AI measurement tools, as used in FDA 510(k) submissions. It runs two worked examples:

- **AI bladder volume (mL)**
- **AI liver craniocaudal length (cm)**

Each example compares the AI measurement with the mean of 3 expert readers. The notebook goes from the raw differences to the PASS/FAIL decision.

> ⚠️ The data in this repository are **synthetic** and for training only. They are not real patient data and do not represent the performance of any device.

---

## Contents

1. [Quick start](#1-quick-start)
2. [What question does Bland-Altman answer?](#2-what-question-does-bland-altman-answer)
3. [Study design](#3-study-design)
4. [The primary endpoint, step by step](#4-the-primary-endpoint-step-by-step)
5. [Why correlation is not enough](#5-why-correlation-is-not-enough)
6. [Choosing the scale: absolute or percent](#6-choosing-the-scale-absolute-or-percent)
7. [Acceptance limits and their sources](#7-acceptance-limits-and-their-sources)
8. [Supportive analyses](#8-supportive-analyses)
9. [Sample size](#9-sample-size)
10. [Results of the synthetic studies](#10-results-of-the-synthetic-studies)
11. [Data dictionary](#11-data-dictionary)
12. [Common mistakes](#12-common-mistakes)
13. [References](#13-references)

---

## 1. Quick start

| File | Content |
|---|---|
| `bland_altman_study.ipynb` | The full analysis with explanations |
| `bladder_study.csv` | 150 bladder cases |
| `liver_study.csv` | 160 liver cases |
| `requirements.txt` | Python packages |

```bash
pip install -r requirements.txt

# Open interactively, then Kernel -> Restart & Run All
jupyter notebook bland_altman_study.ipynb

# Or run headless and keep the executed copy
jupyter nbconvert --to notebook --execute bland_altman_study.ipynb --output bladder_executed.ipynb

# HTML report
jupyter nbconvert --to html --execute bland_altman_study.ipynb
```

- **Choose the study:** set `STUDY = "bladder"` or `"liver"` in the first code cell.
- **Skip the simulation:** set `RUN_POWER = False` to skip the sample size simulation (about 1 minute).
- **Use your own data:** prepare a CSV with the same column names and set `CSV_PATH`.
- **Change limits, scales or cut-offs:** edit the `CONFIG` dictionary.
- **Outputs:** figures and tables are saved to `results_bladder/` or `results_liver/`.
- **Missing CSVs:** if the CSV files are missing, the notebook regenerates them with a fixed seed.

---

## 2. What question does Bland-Altman answer?

Bland-Altman analysis compares two methods that measure the same continuous quantity (mL, mm, cm, %). It answers one question: *for an individual patient, how far apart can the two methods be?* It is the usual primary analysis in FDA submissions for AI measurement tools (product code QIH), where a new method (the AI) is compared with an established one (expert readers).

It does **not** decide on its own whether the agreement is good enough. It gives an interval, and the decision comes from comparing that interval with a clinically justified **acceptance limit** written into the protocol in advance.

It measures **agreement with the experts, not accuracy**. The reference is the experts' average, not the true value. If the experts are systematically wrong, an AI that copies them will still look "in agreement".

---

## 3. Study design

Both synthetic studies follow the pattern seen in FDA 510(k) summaries such as K230497 (Bladder AI) and K253595/K260667 (Philips Auto Measure Abdomen).

| Element | Bladder study | Liver study |
|---|---|---|
| Design | Retrospective, multi-site, standalone | Retrospective, multi-site, standalone |
| Sites and devices | 3 sites; cart-based and handheld | 3 sites; cart-based and handheld |
| Enrolled / analyzed | 150 / 137 | 160 / 147 |
| Measurement | Bladder volume (mL) | Liver craniocaudal length, midclavicular line (cm) |
| Reference standard | Mean of 3 independent, blinded experts | Mean of 3 independent, blinded experts |
| Primary endpoint | Bland-Altman 95 % LoA, absolute (mL) | Bland-Altman 95 % LoA, percent (%) |
| Acceptance limit | ±55 mL | ±15 % |
| Clinical cut-off | PVR > 200 mL | Hepatomegaly > 16 cm |

The reference averages 3 readers to reduce its noise. With a single reader, that reader's own error would widen the LoA and penalise the AI unfairly.

---

## 4. The primary endpoint, step by step

*Notebook section 2*

### Step 1: the difference

For each patient *i*:

- absolute scale: $d_i = \text{AI}_i - \text{Ref}_i$
- percent scale: $d_i = \dfrac{\text{AI}_i - \text{Ref}_i}{(\text{AI}_i + \text{Ref}_i)/2} \times 100$

A positive difference means the AI measures larger than the experts. The percent difference is divided by the **mean of AI and reference**, not by the reference alone. Dividing by the reference gives slightly different numbers.

### Step 2: bias

$\bar d = \frac{1}{n}\sum d_i$ is the average systematic shift. A bias of 0 does not mean good agreement, because +50 and −50 also average to 0.

### Step 3: SD of the differences

$s_d$ (sample SD, n − 1 denominator; `STDEV.S` in Excel) measures how scattered the individual differences are around the bias. This is the random part of the disagreement.

### Step 4: limits of agreement

$$\text{LoA} = \bar d \pm 1.96\, s_d$$

If the differences are roughly normal, about 95 % of patients fall between the two limits. This gives the clinically useful statement: *"for 95 % of patients the AI differs from the experts by between X and Y."*

| Quantity | Bladder (mL) | Liver (%) |
|---|---|---|
| n | 137 | 147 |
| Bias | +1.77 | −2.13 |
| SD | 21.59 | 5.60 |
| Lower LoA = bias − 1.96 × SD | 1.77 − 42.32 = **−40.54** | −2.13 − 10.98 = **−13.10** |
| Upper LoA = bias + 1.96 × SD | 1.77 + 42.32 = **+44.09** | −2.13 + 10.98 = **+8.84** |

### Step 5: confidence intervals

The bias and the LoA are sample estimates, so each has a confidence interval (Bland & Altman 1999):

- $SE(\bar d) = s_d/\sqrt{n}$
- $SE(\text{LoA}) \approx s_d\sqrt{\tfrac{1}{n} + \tfrac{1.96^2}{2(n-1)}}$
- each CI = estimate ± *t*(n − 1) × SE

The confidence intervals are **not** the LoA themselves; they show how precisely the LoA are estimated. If the bias CI excludes 0, the systematic shift is statistically real. In the liver study the bias is −2.13 % with a CI of −3.04 to −1.22. The strict acceptance criterion uses the LoA CIs.

### Step 6: decision

| Criterion | Rule |
|---|---|
| Point | −limit ≤ lower LoA, and upper LoA ≤ +limit |
| Strict | −limit ≤ lower end of the lower-LoA CI, and upper end of the upper-LoA CI ≤ +limit |

The strict criterion protects against passing by luck with a small sample. Philips used it in K260667.

### Step 7: assumption checks

| Check | How | If it fails |
|---|---|---|
| Proportional bias: does the *average* difference change with size? | Regress d on the mean and test the slope | Use the percent scale or a log transform |
| Heteroscedasticity: does the *spread* change with size? | Spearman ρ between \|d\| and the mean (notebook section 4.1) | Change scale, or report LoA as a function of size |
| Normality | Histogram, Q-Q plot, Shapiro-Wilk | Also report nonparametric LoA (2.5th–97.5th percentile) |
| Outliers | About 5 % outside the LoA is expected | Review each case; never remove cases after unblinding |

Shapiro-Wilk becomes significant easily with large n. In the bladder study it is significant (p < 0.001) because of a few large AI errors. The nonparametric LoA (−34.8 / +44.6 mL) is close to the parametric one, so the conclusion does not change.

### Reading the Bland-Altman plot

| Element | Meaning |
|---|---|
| x-axis | Mean of AI and reference, the best estimate of the true size |
| y-axis | Difference |
| Blue line and band | Bias and its 95 % CI |
| Orange dashed lines and bands | LoA and their 95 % CIs |
| Grey band | Acceptance limit |
| Dash-dot line | Regression trend (proportional bias) |
| Red circles | Cases outside the LoA |

Four questions to ask: Is the blue line far from 0? Is the band narrow enough clinically? Do the dots slope or fan out? Do the outliers cluster in one region?

---

## 5. Why correlation is not enough

*Notebook section 3*

1. **Correlation measures association, not agreement.** r asks whether the points lie on *any* straight line. Agreement requires them to lie on the line y = x.
2. **r is blind to systematic error.** Pearson r does not change when a constant is added or when the values are multiplied by a constant. In the notebook, the original AI, "AI + 30 mL" and "AI × 1.20" all have r = 0.9906, yet the bias and LoA of the two faulty versions fall far outside the limit.
3. **r depends on the range.** Splitting the cases into thirds by size gives a much lower r in each third, even though the disagreement per patient is similar. A study with very different patient sizes can report an impressive r without the agreement being any better.

ICC(A,1), which measures absolute agreement, does fall for the faulty versions, so it is a better supportive measure than r. It still gives no answer in clinical units.

---

## 6. Choosing the scale: absolute or percent

*Notebook section 4*

| Scale | Question it answers | Natural when |
|---|---|---|
| Absolute (mL, cm) | "How many units can the AI be off?" | The error is constant across sizes; the decision uses a fixed threshold |
| Percent (%) | "By what fraction can the AI be off?" | The error grows with size; sizes span a wide range |

The check is the Spearman correlation between |difference| and the mean. The scale with ρ closer to 0 has the more stable spread.

| Study | ρ absolute | ρ percent | Primary scale | Why |
|---|---|---|---|---|
| Bladder | +0.50 | −0.24 | Absolute (mL) | The clinical threshold (PVR 200 mL) and FDA K230497 use mL, even though the spread grows with volume. The secondary percent analysis (reference ≥ 100 mL) covers the gap. |
| Liver | +0.19 | +0.02 | Percent (%) | Percent removes the size dependence; cm is reported as the secondary endpoint. |

**General rule:** choose the scale with a stable spread. If a clinical threshold or regulatory precedent requires a specific unit, use that unit, report the other scale as a secondary endpoint, and justify the trade-off in the protocol.

---

## 7. Acceptance limits and their sources

The usual logic is that *the AI should not disagree with the experts more than the experts disagree with each other*. The limit comes from published inter-reader variability or from accepted precedents.

### Bladder volume: ±55 mL

- Inter-sonographer agreement for manual 2D ultrasound was 3.2 ± 54.4 mL (AI bladder volume study, 2024, PMC11353831). In the same study the AI reached a bias of 17.1 mL and LoA of −52.7 / +86.9 mL against a single expert.
- FDA K230497 (Bladder AI) used 122 subjects with volumes of 11–645 mL, and the reference was the mean of 3 experts. Dual-view bias was 2 mL, LoA −42 / +46 mL, ICC 0.98. The summary does not state a pre-specified acceptance limit.

### Liver craniocaudal length: ±15 %

- No published numeric LoA for liver length was found, so the limit follows FDA organ-length precedents in K260667 (Philips Auto Measure Abdomen): kidney sagittal length ±14.3 %, spleen length ±15.9 %.
- Liver measurements are less reproducible than kidney measurements (inter-rater ICC 0.71–0.94, Childs et al. 2014), so a tighter limit would not be realistic.

### Clinical cut-offs

- PVR > 200 mL: elevated post-void residual (PMC11353831).
- Hepatomegaly > 16 cm (midclavicular line). The upper limit of normal is usually given as 15–16 cm and varies by institution.

> In a real study, write the limit into the protocol before unblinding and confirm it with FDA in a Pre-Submission meeting.

---

## 8. Supportive analyses

*Notebook sections 5–7*

| Analysis | What it shows |
|---|---|
| ICC(A,1) | Absolute-agreement index between 0 and 1 |
| MAE | Mean absolute error in clinical units |
| Clinical cut-off | Sensitivity, specificity (Wilson CI) and kappa: does the AI put the patient on the same side of the decision threshold as the experts? |
| AI vs. readers | Compares the AI's LoA with reader-vs-reader LoA; also tests whether the limit is realistic |
| Subgroups | Site, device, sex, BMI, age and image quality; exploratory, so they do not change the decision |

---

## 9. Sample size

*Notebook section 8*

The notebook simulates, for each n, the probability that both LoA confidence intervals lie inside the limit (the strict criterion), given an assumed bias and SD. This is a simulation counterpart of the analytic method of Lu et al. (2016). The SD should come from pilot data or the literature, and 10–20 % should be added for non-analyzable cases.

| Study | Assumed bias | Assumed SD | Limit | n for 80 % power | n for 90 % power |
|---|---|---|---|---|---|
| Bladder | +2 mL | 22 mL | ±55 mL | 140 | 180 |
| Liver | −1.5 % | 5 % | ±15 % | 60 | 80 |

The bladder study (150 enrolled, about 9 % excluded, 137 analyzed) matches the 80 % target.

---

## 10. Results of the synthetic studies

| | Bladder | Liver |
|---|---|---|
| Analyzed | 137 / 150 | 147 / 160 |
| Bias (95 % CI) | +1.77 mL (−1.87 / +5.42) | −2.13 % (−3.04 / −1.22) |
| LoA | −40.54 / +44.09 mL | −13.10 / +8.84 % |
| Limit | ±55 mL | ±15 % |
| Point criterion | PASS | PASS |
| Strict criterion | PASS | PASS |
| Reader-vs-reader LoA | about −52 / +50 to −76 / +55 mL | about −18 / +14 to −21 / +12 % |
| ICC(A,1) | 0.991 | 0.926 |
| Kappa at the clinical cut-off | 0.851 | 0.843 |
| Subgroups exceeding the limit | Age 65+ | Site_A, BMI ≥ 30 |

**Interpretation:** both AIs pass on both criteria and agree with the reference more closely than the readers agree with each other. The liver AI has a small but statistically real negative bias, meaning it measures slightly short. The flagged subgroups are exploratory, but a real submission would need to discuss them.

---

## 11. Data dictionary

| Column | Description |
|---|---|
| `case_id` | Case number |
| `site` | Site_A, Site_B or Site_C |
| `device_type` | Cart-based or Handheld |
| `age`, `sex`, `bmi`, `bmi_category` | Demographics for the subgroup analysis |
| `image_quality` | Good, Adequate or Poor |
| `analyzable` | Yes or No |
| `exclusion_reason` | Reason for exclusion; measurements are empty for excluded cases |
| `reader1_*`, `reader2_*`, `reader3_*` | Expert measurements (`_ml` or `_cm`) |
| `ai_ml` / `ai_cm` | AI measurement |

The notebook computes the reference (the mean of the 3 readers); it is not stored in the CSV.

The readers have small built-in systematic differences (reader 3 measures slightly larger), and a few AI outliers were added on purpose so that the plots show realistic patterns.

---

## 12. Common mistakes

| Mistake | Why it is wrong |
|---|---|
| Reporting only r or R² | Blind to systematic error and inflated by the range |
| Choosing the limit after seeing the data | Makes the result meaningless; FDA expects a pre-specified limit |
| Treating the LoA as a hypothesis test | There is no p-value; the LoA is an interval judged against a clinical limit |
| Confusing the LoA with their confidence intervals | The CIs show the precision of the LoA; they are not bias ± 1.96 × SD |
| Ignoring the LoA confidence intervals | A small study can pass by chance |
| Using absolute differences when the spread grows with size, or the reverse | The LoA is wrong for part of the range |
| Removing outliers after unblinding | Biases the result; review and report them instead |
| Using one reader as the reference | That reader's error widens the LoA |
| Treating repeated images of one patient as independent | Underestimates the SD; use Bland & Altman (2007) for repeated measures |
| Claiming the AI can replace experts | The analysis shows standalone agreement with experts, not reader performance with AI; that needs an MRMC design |

---

## 13. References

**Method**
- Bland JM, Altman DG. Statistical methods for assessing agreement between two methods of clinical measurement. *Lancet* 1986;327:307–310.
- Bland JM, Altman DG. Measuring agreement in method comparison studies. *Stat Methods Med Res* 1999;8:135–160.
- Bland JM, Altman DG. Agreement between methods of measurement with multiple observations per individual. *J Biopharm Stat* 2007;17:571–582.
- Lu MJ et al. Sample size for assessing agreement between two methods of measurement by Bland-Altman method. *Int J Biostat* 2016.
- McGraw KO, Wong SP. Forming inferences about some intraclass correlation coefficients. *Psychol Methods* 1996;1:30–46.
- Wen S, Gallas BD. Three-way mixed effect ANOVA to estimate MRMC limits of agreement. arXiv:2107.08891.

**FDA precedents**
- K230497 Bladder AI: https://fda.innolitics.com/device/K230497
- K260667 Philips Auto Measure Abdomen: https://fda.innolitics.com/device/K260667
- K262548 Philips 2D Auto LA: https://www.accessdata.fda.gov/cdrh_docs/pdf26/K262548.pdf

**Clinical literature**
- AI-powered automatic bladder volume measurement with 2D ultrasound (2024): https://pmc.ncbi.nlm.nih.gov/articles/PMC11353831/
- Alfuraih et al. Handheld 3D vs 2D ultrasound for bladder volume. *Diagnostics* 2025;15:2229: https://pmc.ncbi.nlm.nih.gov/articles/PMC12428280/
- Childs JT et al. Ultrasound measurements of the liver: an intra and inter-rater reliability study. *Australas J Ultrasound Med* 2014;17:113–119.
- Liver size reference values and hepatomegaly thresholds: https://radshare.net/articles/liver-size-reference-values-and-hepatomegaly-thresholds-on-ultrasound
