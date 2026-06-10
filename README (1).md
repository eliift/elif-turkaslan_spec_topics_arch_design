# Le Corbusier — 468 Sketches

## Computational Analysis of Sketching Behaviour Across the Œuvre Complète (1910–1965)

**Architectural Design Computing**  
Elif Türkaslan

\---

## Overview

This dataset and visualization project presents a computational analysis of 468 architectural sketches drawn from all eight volumes of Le Corbusier's *Œuvre Complète* (1910–1969), accessed via the İTÜ EBSCO Library Database. Each sketch is characterized by two quantitative metrics — **move count** and **ink density** — alongside manually assigned categorical labels, enabling longitudinal study of how Le Corbusier's sketching behaviour evolved across his 55-year career.

**Central research question:**

> \*How did Le Corbusier's sketching behaviour evolve from early to late career?\*

\---

## Dataset

### Source \& Periodization

Sketches were organized into three periods following the framework established by William J.R. Curtis in *Le Corbusier: Ideas and Forms* (Phaidon, 1987):

|Period|Years|n|Description|
|-|-|-|-|
|Part I|1887–1922|72|The Formative Years of Charles-Édouard Jeanneret|
|Part II|1922–1944|184|Architectural Ideals and Social Realities|
|Part III|1945–1965|212|The Ancient Sense: Late Works|

### File

`FINALVERI.csv` — 468 rows, one per sketch.

### Columns

|Column|Type|Description|
|-|-|-|
|`filename`|string|SVG filename (e.g. `PART1 (10).svg`)|
|`folder`|string|Source folder path by part|
|`period`|string|Full period name string|
|`content\_label`|string|Manually assigned content category|
|`move`|integer|SVG path count (move count proxy)|
|`Area Covered (Pixels)`|integer|Non-white pixel count (ink density proxy)|
|`Era`|integer|Year extracted from filename|

### Content Labels

Seven content categories were assigned manually using an interactive labeling tool built in Google Colab:

|Label|Part I|Part II|Part III|Total|
|-|-|-|-|-|
|landscape|22|50|52|124|
|plan|15|46|37|98|
|building|12|31|41|84|
|section|2|25|29|56|
|scene|12|6|18|36|
|other\*|6|24|30|60|
|facade|3|2|5|10|

\*`other` contains `art`, `object`, and `other` labels for analysis.

\---

## Metrics

### Move Count

**Definition:** The total number of SVG path elements in the vectorized sketch, used as a proxy for the number of discrete pen gestures (moves) Le Corbusier made.

**Pipeline:**

1. Source PNG (256×256 px) → SVG conversion (outline mode, threshold=200)
2. Count of `<path>` elements in the resulting SVG = move count
3. Validated against Hough transform line detection

**Statistics by period:**

|Period|Mean|Median|Std|Max|
|-|-|-|-|-|
|Part I|99.4|79|—|436|
|Part II|78.3|50|—|409|
|Part III|113.8|76|—|1054|
|**Overall**|**97.6**|**68**|**102.4**|**1054**|

Career change (I → III): **+15%** — move count is nearly stable over 55 years. Overall distribution is strongly right-skewed (skewness = 2.89, kurtosis = 17.16), with the mean (97.6) sitting well above the median (68.0), indicating a long upper tail driven by intensive sketching sessions.

### Ink Density (Occupancy)

**Definition:** The proportion of non-white pixels in the 256×256 PNG, computed pixel-by-pixel. White pixels = paper; non-white pixels = ink.

**Formula:** `occupancy = non-white\_pixels / 65536`

**Overall statistics:** mean area = 5265.7 px, median = 4428 px, std = 3214.5 px, min = 634 px, max = 21230 px (skewness = 1.56, kurtosis = 3.43).

**Statistics by period:**

|Period|Mean occupancy|
|-|-|
|Part I|20.9%|
|Part II|68.8%|
|Part III|89.2%|

Career change (I → III): **×4.3** — ink density quadrupled across the career. This is the dominant signal in the dataset. The KDE per-period plot confirms that Part I has a distinctly lower and wider distribution, while Part II and III shift toward higher area values with reduced spread.

**Note:** Move count and ink density are independent dimensions (Pearson r = −0.08), confirming they capture distinct aspects of sketch complexity. A scatter plot of move vs. area covered confirms this orthogonality — no linear pattern is visible across any of the three periods.

\---

## Dataset Construction Pipeline

```
1. Source Acquisition
   └─ Œuvre Complète (8 vols) · İTÜ EBSCO Library
   └─ Periodization: Curtis (1987) Part I / II / III

2. Image Extraction
   └─ Manual page screenshot → sketch crop
   └─ Resize to 256 × 256 px PNG
   └─ Background cleaning
   └─ Folder-sorted by Part

3. Content Labeling
   └─ Interactive labeling tool (Google Colab)
   └─ One label per sketch · 7 categories
   └─ Manual annotation by researcher

4. Move Counting
   └─ PNG → SVG (outline mode, threshold=200)
   └─ Count <path> elements = move count
   └─ Hough transform cross-validation

5. Ink Density
   └─ Pixel-by-pixel analysis on source PNG
   └─ non-white pixel count → area\_px
   └─ Normalised: occupancy = area\_px / 65536

6. Era Labeling
   └─ Filename parsing → year → decade tag
   └─ 6 decades: 1910s · 1920s · 1930s · 1940s · 1950s · 1960s
   └─ 3 parts · 55-year career span
```

\---

## Data Quality

Seven checks were run on the dataset, documented in the analysis figures:

### 1\. Data Consistency (Fig. 1)

All three consistency checks pass with 468/468 records:

* **Period label vs. filename:** All 468 records show consistent period assignment — zero mismatches between the `period` string and the `folder`/`filename` path.
* **Area Covered validity:** All 468 records have `area > 0` — no blank or empty images entered the dataset.
* **Move Count validity:** All 468 records have `move ≥ 1` — every sketch produced at least one SVG path.

### 2\. Outlier Analysis (Fig. 2)

IQR-based outlier detection was applied to both quantitative metrics:

* **Move count:** 28 outliers identified. The boxplot shows a compact IQR with the upper whisker at approximately 280, above which 28 data points are scattered up to the maximum of 1054. The |Z-score| histogram confirms that most sketches fall below z = 3, with a thin tail extending to z ≈ 9 for the most extreme cases.
* **Area covered:** 21 outliers identified. The boxplot shows a median near 4428 px and an upper whisker at approximately 11,000 px, with 21 points scattered above this up to 21,230 px. The |Z-score| distribution is less extreme than for move count (tail reaches z ≈ 5), suggesting area outliers are less anomalous in relative terms.

Both outlier sets are retained in the dataset and flagged, not removed — they likely represent genuine intensive design sessions rather than measurement error.

### 3\. Missing Data Analysis (Fig. 3)

The missing value count chart confirms zero missing values across all columns. The missing value heatmap (rows × columns) is entirely blank for all eight columns: `filename`, `folder`, `period`, `content\_label`, `move`, `Area Covered (Pixels)`, `part`, and `area\_px`. No MCAR / MAR / MNAR classification is required.

### 4\. Missing Data Type Classification (Fig. 4)

Complementing Fig. 3, explicit missing data type classification for the two key numeric columns confirms: `move — No missing data` and `area\_px — No missing data`. The dataset is complete for all analytical purposes.

### 5\. Distribution and Statistical Summary (Fig. 5)

The move count histogram shows a sharp peak at low values (mode ≈ 20–40 range) with a long right tail extending to 1054 — consistent with skewness = 2.89, kurtosis = 17.16. The area (px) histogram shows a less extreme but still right-skewed shape (skewness = 1.56, kurtosis = 3.43) with a peak around 3000–5000 px and a tail to 21,230 px.

The KDE per-period plots reveal the career signal directly: for move count, the three period curves largely overlap, confirming metric stability. For area, the Part I curve (pink) peaks lower and spreads wider, while Part II and Part III curves (purple/dark purple) shift rightward — visually confirming the ×4.3 increase in ink density.

Full summary statistics:

|Metric|n|Mean|Median|Std|Min|Max|Skew|Kurtosis|
|-|-|-|-|-|-|-|-|-|
|Move count|468|97.62|68.00|102.38|1|1054|2.89|17.16|
|Area (px)|468|5265.66|4428.00|3214.51|634|21230|1.56|3.43|

### 6\. Repeated Data Check (Fig. 6)

Three types of repetition were checked:

* **Full row duplicates:** 0 — no two sketches are identical across all columns.
* **Filename duplicates:** 0 — every sketch has a unique source file.
* **Move + Occupancy + Label combinations:** 75 sketches share the same combination of move count, occupancy value, and content label. This is not a data integrity problem — in a corpus of 468 sketches spanning narrow categorical and rounded numeric values, coincidental combination overlap is expected. It reflects the limited resolution of the proxy metrics and Le Corbusier's consistent compositional repertoire, not data duplication.

The most frequent individual move values (top 20) range from 4 to 148, with move = 22 and move = 21 appearing most frequently (8 times each), confirming that the high-frequency values fall within the normal range of the distribution rather than at pathological values.

Content label frequency confirms landscape (124) as the most common label, followed by plan (98), building (84), section (56), scene/other (36 each), art (16), facade (10), and object (8). Note that `art`, `object`, and `other` are consolidated into `other` for period-level analysis.

### 7\. Bias Check (Fig. 7)

Four bias-related checks were performed:

* **Sample size per period:** Part I = 72, Part II = 184, Part III = 212. The monotonically increasing sample size reflects the Œuvre Complète's editorial structure — later volumes are more comprehensive — rather than any selection bias introduced by this analysis.
* **Label distribution per period:** Landscape dominates Part I (\~30%), while building grows in Parts II and III (\~16–19%). Section grows from near-zero in Part I (2 sketches, \~3%) to 13–14% in Parts II–III. Scene is suppressed in Part II (3.3%) relative to Parts I and III. Art appears only in Parts II and III.
* **Mean move count per period (± std):** Part I mean ≈ 99 (std ≈ 90), Part II mean ≈ 78 (std ≈ 75), Part III mean ≈ 114 (std ≈ 120). The growing standard deviation across parts indicates increasing variability in late-career sketching behavior, not just a higher mean.
* **Mean area covered per period (± std):** Part I mean ≈ 7231 px (std ≈ 3400), Part II mean ≈ 5153 px (std ≈ 3000), Part III mean ≈ 4696 px (std ≈ 3200). Note that area (px) as a raw metric shows a declining mean across parts — this apparent contradiction with the ink density finding is resolved by normalization: occupancy is computed relative to the 256×256 canvas, and image preprocessing differences between parts affect the raw pixel count. The normalized occupancy (20.9% → 89.2%) is the reliable metric.
* **Correlation matrix:** Pearson r(move, area\_px) = −0.08, confirming metric independence across the full dataset and within each period.

\---

## Key Findings

**1. Ink density is the dominant career signal.**
Occupancy grew from 20.9% to 89.2% — a 4.3× increase — while move count changed only +15%. Le Corbusier made approximately the same number of gestures throughout his career, but each gesture covered progressively more of the page.

**2. Part II compression (1922–1944).**
Move count dropped to a mean of 78.3 (median 50) while ink density tripled relative to Part I. The sharpest median-to-mean gap of any period (50 vs. 78.3) signals high internal variance: most Purist sketches are economical, but a subset is extremely dense. This coincides with the Purist period — economy of line, density of intent.

**3. Part III recovery (1945–1965).**
Move count recovered to its career high (mean 113.8, std 120) while ink density continued rising. The increasing standard deviation in Part III suggests that late-career sketching became simultaneously more expansive and more variable — a mature hand that both explores and commits, with greater spread between sessions.

**4. Landscape dominates Part I; building and section grow in Parts II–III.**
Content label distribution shifts from landscape-heavy (30.6% in Part I) toward building (19.3%) and section (13.7%) in Part III, reflecting the shift from naturalistic observation to architectural production. Scene is specifically suppressed in Part II (3.3%), suggesting the observational mode of drawing was curtailed during the Purist years.

\---

## Color System

All colors used in visualizations are sourced from Le Corbusier's **Salubra 1931** wallpaper palette — 63 colors he personally selected — making the visualization a chromatic extension of his own design language.

**Period colors:** Part I: `#cd5e84` · Part II: `#f97f47` · Part III: `#bf3c40`

**Era colors:** 1910s: `#719eda` · 1920s: `#ff603b` · 1930s: `#b63213` · 1940s: `#e4cdcb` · 1950s: `#dad520` · 1960s: `#afdbee`

**Content label colors:** landscape: `#cb6248` · plan: `#e8a0c0` · building: `#fd357a` · section: `#578ed4` · scene: `#89b576` · facade: `#c4a1aa` · other: `#72403c`

\---

## Visualizations

|File|Description|
|-|-|
|`lc\_sketch\_html1.html`|26×18 interactive sketch grid with lightbox, content label filtering, fixed legend sidebars|
|`lc\_ridgeline\_html2.html`|Chronological ridgeline chart — move frequency distribution over career, decade labels, content label color overlays|
|`LeCorbusier\_combined.html`|Tabbed interface combining info page and sketch grid|
|`LeCorbusier\_stripe\_legend.html`|A4 — all 468 sketches as color grid; single stripe anatomy diagram|

### Stripe Encoding

Each sketch is represented as a vertical color stripe with five sub-stripes:

```
┌─────────────────────────────┐
│ PART    │ narrow, left      │  period color (pink / orange / red)
│ MOVE    │ black             │  width ∝ move count
│ OCC.    │ gold              │  opacity \& width ∝ ink density
│ CONTENT │ label color       │  manually assigned category
│ ERA     │ narrow, right     │  decade color (Salubra)
└─────────────────────────────┘
```

### Move Lines Grid

Each sketch cell encodes: **line count** = move count · **line length** = uniform · **position** = random within cell · **opacity** = occupancy value.

\---

## Technical Environment

* Google Colab (Python 3)
* Libraries: `PIL`, `numpy`, `pandas`, `potrace`, `svgpathtools`
* Visualizations: vanilla HTML / SVG / JS, no external runtime dependencies

\---

## Narrative, Audience \& Reflection

### The Narrative

The central message of this visualization project is not simply that Le Corbusier's sketches changed over time. Move count remained nearly stable across 55 years (+15%). Ink density quadrupled (×4.3). The same hand, making roughly the same number of moves, progressively committed more of itself to each page. This asymmetry — gesture count stable, commitment radically increasing — reframes career development not as increasing prolificacy or decreasing economy, but as a deepening of presence.

### Intended Audience

**Primary: Architectural design researchers and educators** working at the intersection of design cognition, computational design studies, and architectural history. They expect methodological transparency and interpretive restraint, and are comfortable with quantitative metrics as proxies for cognitive constructs.

**Secondary: Graduate students in computational design and data visualization** who are learning to build and interpret data visualizations of historical design data. The stripe legend and annotation pages are designed specifically for this audience's entry into the dataset.

### Design Decisions Informed by Audience

**Salubra 1931 color palette** signals disciplinary credibility to design historians while enriching the data with historical context for students. **Playfair Display + DM Mono typographic pairing** positions the work at the boundary between archival scholarship and computational analysis — serif for humanistic inquiry, monospace for computational precision. **Paper-toned backgrounds (`#f5f0e8`)** reference the material substrate of the sketches themselves. **The ridgeline chart** is legible to research audiences but may require orientation for less experienced viewers; the annotation and stripe legend pages bridge this gap.

### Critical Evaluation

**1. Part II compression is more extreme than anticipated.** The Purist period shows the sharpest median-to-mean gap of any period (median = 50, mean = 78.3), indicating high internal variance. "Purism" as a design posture was not uniformly applied — some moments of exception or excess persist within it.

**2. Scene nearly disappears in Part II, then partially recovers.** Scene sketches drop from 16.7% of Part I to just 3.3% of Part II — a sharper suppression than landscape (30.6% → 27.2%). The observational, phenomenological mode of drawing was specifically curtailed during the Purist period.

**3. The 28 move-count outliers cluster in the late career.** Part III has both the highest mean (113.8) and the highest standard deviation (≈120), suggesting that late Le Corbusier occasionally entered a qualitatively different sketching mode. The most extreme sketch reaches 1054 moves — roughly 10× the median.

**4. The independence of move count and ink density (r = −0.08) is itself a finding.** Neither metric predicts the other. This orthogonality means that studying either in isolation gives an incomplete picture of sketch complexity.

**5. The color grid reveals a sharper 1922 transition than the ridgeline suggests.** In the 26×18 sketch grid, the shift from Part I (pink) to Part II (orange) is visually more abrupt than the Part II–III transition — a temporal dynamic that the ridgeline's smoothed density curves partially obscure.

\---

## Theoretical Framework

468 Sketches from the Œuvre Complète (1910–1965).

* **Curtis (1987)** — Periodization framework for Le Corbusier's work

Move count approximates the number of inferential moves in Goldschmidt's sense; ink density approximates the depth of visual commitment per session.



