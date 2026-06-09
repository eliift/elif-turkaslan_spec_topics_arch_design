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

Sketches were drawn from the *Œuvre Complète* and organized into three periods following the framework established by William J.R. Curtis in *Le Corbusier: Ideas and Forms* (Phaidon, 1987):

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

\*`other` consolidates `art`, `object`, and `other` labels for analysis.

\---

## Metrics

### Move Count

**Definition:** The total number of SVG path elements in the vectorized sketch, used as a proxy for the number of discrete pen gestures (moves) Le Corbusier made.

**Pipeline:**

1. Source PNG (256×256 px) → SVG conversion using Potrace (outline mode, threshold=200)
2. Count of `<path>` elements in the resulting SVG = move count
3. Validated against Hough transform line detection

**Statistics by period:**

|Period|Mean|Median|Max|
|-|-|-|-|
|Part I|99.4|79|436|
|Part II|78.3|50|409|
|Part III|113.8|76|1054|

Career change (I → III): **+15%** — move count is nearly stable over 55 years.

Distribution is right-skewed (skewness = 2.89, kurtosis = 17.16). 28 sketches exceed the IQR upper fence and are classified as outliers, likely representing intensive design sessions.

### Ink Density (Occupancy)

**Definition:** The proportion of non-white pixels in the 256×256 PNG, computed pixel-by-pixel. White pixels = paper; non-white pixels = ink.

**Formula:** `occupancy = non-white\_pixels / 65536`

**Statistics by period (from processed values):**

|Period|Mean occupancy|
|-|-|
|Part I|20.9%|
|Part II|68.8%|
|Part III|89.2%|

Career change (I → III): **×4.3** — ink density quadrupled across the career. This is the dominant signal in the dataset.

**Note:** Move count and ink density are independent dimensions (Pearson r = −0.08), confirming they capture distinct aspects of sketch complexity.

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
   └─ PNG → SVG (Potrace, outline mode, threshold=200)
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

|Check|Result|
|-|-|
|Missing values|None — all 468 records complete across all columns|
|Duplicate rows|0 full duplicates|
|Repeated value combinations|75 sketches share same (move + occupancy + label) — expected in a large corpus; reflects design repertoire, not data error|
|Move outliers (IQR)|28 sketches (max = 1054) — intensive design sessions|
|Area outliers (IQR)|21 sketches|
|Sampling bias|Sample sizes grow I→III (72, 184, 212), reflecting Œuvre Complète editorial choices, not analysis design|
|Move–area correlation|r = −0.08 — metrics are independent|

\---

## Key Findings

**1. Ink density is the dominant career signal.**
Occupancy grew from 20.9% to 89.2% — a 4.3× increase — while move count changed only +15%. Le Corbusier made approximately the same number of gestures throughout his career, but each gesture covered progressively more of the page.

**2. Part II compression (1922–1944).**
Move count dropped −21% relative to Part I (99.4 → 78.3) while ink density tripled. This coincides with the Purist period — economy of line, density of intent.

**3. Part III recovery (1945–1965).**
Move count recovered to its career high (113.8) while ink density continued rising. Late-career drawings are simultaneously more exploratory (more gestures) and denser — suggesting a mature hand that both explores and commits.

**4. Landscape dominates Part I; building and section grow in Parts II–III.**
Content label distribution shifts from landscape-heavy (30.6% in Part I) toward a more balanced distribution across building types, reflecting the shift from naturalistic observation to architectural production.

\---

## Color System

All colors used in visualizations are sourced from Le Corbusier's **Salubra 1931** wallpaper palette — 63 colors he personally selected — making the visualization a chromatic extension of his own design language.

**Period colors:**

* Part I: `#cd5e84` (pink)
* Part II: `#f97f47` (orange)
* Part III: `#bf3c40` (red–crimson)

**Era colors (decade):**

* 1910s: `#719eda` · 1920s: `#ff603b` · 1930s: `#b63213`
* 1940s: `#e4cdcb` · 1950s: `#dad520` · 1960s: `#afdbee`

**Content label colors:**

* landscape: `#cb6248` · plan: `#e8a0c0` · building: `#fd357a`
* section: `#578ed4` · scene: `#89b576` · facade: `#c4a1aa` · other: `#72403c`

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
┌──────────────────────┐
│ PART    │ narrow, left │ period color (pink / orange / red)
│ MOVE    │ black        │ width ∝ move count
│ OCC.    │ gold         │ width ∝ ink density
│ CONTENT │ label color  │ manually assigned category
│ ERA     │ narrow, right│ era color
└──────────────────────┘
```

### Move Lines Grid

Each sketch cell in the move lines grid encodes:

* **Line count** = move count of that sketch
* **Line length** = uniform (all equal)
* **Position** = randomly distributed within cell
* **Opacity** = occupancy value of that sketch

\---

## Technical Environment

* Google Colab (Python 3)
* Libraries: `PIL`, `numpy`, `pandas`, `potrace`, `svgpathtools`
* Storage: Google Drive (`/content/drive/MyDrive/spe. topic./VİZ/final process/FINAL/`)
* Visualizations: vanilla HTML/SVG/JS, no external runtime dependencies

\---

## Narrative, Audience \& Reflection

### The Narrative

The central message of this visualization project is not simply that Le Corbusier's sketches changed over time. Move count, a proxy for the number of distinct pen gestures per sketch, remained nearly stable across 55 years (+15%). Ink density, a proxy for how much of the page those gestures covered, quadrupled (×4.3). The same hand, making roughly the same number of moves, progressively committed more of itself to each page.

This asymmetry , gesture count stable, commitment radically increasing, is the key insight the visualization is designed to foreground. It reframes career development not as increasing prolificacy or decreasing economy, but as a deepening of presence: Le Corbusier did not draw more lines, he drew lines that meant more of the paper.



### Intended Audience



**Primary: Architectural design researchers and educators**
Specifically, scholars working at the intersection of design cognition, computational design studies, and architectural history. They are comfortable with quantitative metrics as proxies for cognitive constructs, and they expect both methodological transparency and interpretive restraint.

**Secondary: Graduate students in computational design and data visualization**
Students who are learning to build and interpret data visualizations of historical or behavioral design data. 





### Design Decisions Informed by Audience

**Color palette from the Salubra 1931 collection** is not merely aesthetic — for design historians, it signals that the researcher understands Le Corbusier's own chromatic thinking, establishing disciplinary credibility. For students, it introduces a layer of historical context that enriches the data without overwhelming it.

**Playfair Display + DM Mono typographic pairing** positions the work at the boundary between archival scholarship and computational analysis. Serif for titles and research questions signals humanistic inquiry; monospace for labels and data signals computational precision. This dual register matches the dual nature of the audience.

**The ridgeline chart** is legible to research audiences familiar with data visualization conventions, but may require orientation for less experienced viewers. The annotation page and stripe legend are designed to bridge this gap without condescending to either group.

**Paper-toned backgrounds (`#f5f0e8`)** reference the material substrate of the sketches themselves (paper) grounding a digital visualization in the physical practice it describes.



### Evaluation

### Several unexpected patterns emerged during analysis that complicate and enrich the central narrative:

**1. Part II compression is more extreme than anticipated.**
The Purist period (1922–1944) shows not just reduced move counts but the sharpest median-to-mean gap of any period (median = 50, mean = 78.3), indicating high variance. While most Purist sketches are economical, a subset is extremely dense. This suggests that "Purism" as a design posture was not uniformly applied even within Le Corbusier's own sketchbooks — some moments of exception or excess persist.

**2. Scene nearly disappears in Part II, then partially recovers.**
Scene sketches drop from 12 (16.7% of Part I) to just 6 (3.3% of Part II) before recovering to 18 (8.5% in Part III). This is a sharper suppression than landscape, which declines more gradually. It suggests that the observational, phenomenological mode of drawing — capturing ambient scenes — was specifically suppressed during the Purist period, not merely diluted by the growing importance of plan and section.

**3. The 28 outlier sketches form a pattern worth studying independently.**
Sketches with move counts above the IQR ceiling (max = 1054, concentrated in Part III) are not randomly distributed. They cluster in the late career, which has both the highest mean and highest variance. This suggests that late Le Corbusier occasionally entered a qualitatively different sketching mode — not just more productive, but categorically more intensive. These sketches may correspond to specific projects (Chandigarh, the Venice hospital, La Tourette) and would reward archival cross-reference.

**4. The independence of move count and ink density is itself a finding.**
The near-zero correlation (r = −0.08) means neither metric predicts the other. A sketch with many moves is not necessarily ink-dense, and vice versa. This orthogonality implies that the two dimensions capture genuinely distinct aspects of the design act — one quantifying exploratory range, the other quantifying surface commitment — and that studying either in isolation gives an incomplete picture.

**5. The color grid reveals period clustering that the ridgeline smooths.**
In the 26×18 sketch grid, the transition from Part I (pink) to Part II (orange) is visually abrupt at approximately column 6–7, while the Part II–III transition is more gradual. This suggests that Le Corbusier's style change circa 1922 was more sudden than the 1945 shift — a hypothesis that the ridgeline, which averages across sketches, partially obscures.



## Theoretical Framework

Œuvre Complète (1910–1965),

* **Curtis (1987)** — Periodization framework for Le Corbusier's work

The dataset operationalizes Goldschmidt's notion of the sketch as a cognitive artifact: move count approximates the number of inferential moves; ink density approximates the depth of visual commitment per session.



