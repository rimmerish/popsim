USE index2.html for latest version. simply download file and open with chrome.

# Rolph Population Sandbox

A tiny, client‑side population dynamics simulator you can share as a single HTML file or embed as a React component. It models age‑structured population change year‑by‑year with adjustable fertility, mortality, and periodic catastrophes, and plots the result with a hoverable tooltip.

---

## Quick start

**Single‑file HTML build**

* Open the HTML file in any modern browser. No server required.
* Everything runs locally; you can upload or host the file like a normal web page (GitHub Pages, Netlify, WordPress HTML block, etc.).

**React component build**

* Import the component into your app and render it inside your UI framework (it uses shadcn/ui cards and Recharts for the chart).
* State is local to the component; there’s no backend or storage.

---

## What the model does (in plain language)

The simulator keeps a **distribution of people by age** (0…maxAge). Each simulated year it:

1. **Counts fertile girls** (ages within your birthing window, assuming 50% of people are female).
2. **Adds births**: births per fertile girl per year = `kidsPerGirl / (fertileEnd − fertileStart + 1)`; total births = that rate × fertile girls × 0.5.
3. **Removes natural deaths** using an age‑specific death probability `q[a]` (life table).
4. **Ages everyone** one year (age 0 is refilled by the births).
5. **Applies catastrophe** (if enabled) at the end of the year by removing a fixed fraction across all ages.

The chart displays total population over time. A sampling step keeps the chart fast for long timelines.

---

## Life table and survivorship fit

We fit a simple, tunable survivorship curve from your inputs:

* **Mean life** (years)
* **Survive to 40 (1 in N)** → target survivorship $S(40) = 1/N$
* **Survive to max age (1 in N)** → target survivorship $S(maxAge) = 1/N$

Construction:

* Ages **0–39**: deaths follow a **truncated geometric** distribution whose parameter is solved so that, combined with the point masses below, the overall **mean life** matches your input.
* Ages **40** and **maxAge**: add **point‑mass deaths** to match $S(40)$ and $S(maxAge)$.
* Convert the empirical death distribution to **annual death probabilities** `q[a]` via `q[a] = pmf[a] / S[a]` (with `S[0] = 1`).

This isn’t a demographic gold standard—it’s a compact way to respect your three constraints without micromanaging every age.

---

## Controls and what they mean

### Population & Timeline

* **Start population** – Initial total population.
* **Start mean age** – Used to spread the starting population over early ages.
* **Years to simulate** – Up to 100,000. The chart samples to ≤ \~1,200 points for responsiveness.
* **Log scale** – Y‑axis becomes logarithmic (minimum is 1).

### Mortality

* **Mean life (years)** – Target average lifespan.
* **Max age** – Oldest possible age (everyone dies by here).
* **Survive to 40 (1 in N)** – Fraction alive at 40.
* **Survive to max age (1 in N)** – Fraction alive at `maxAge`.
* **Checks** – Displays the fitted survivorship at 40 and at max age, plus the resulting mean life.

### Fertility

* **Birthing age start / stop** – Inclusive fertile window (e.g., 9–35).
* **Average kids per girl** – Lifetime births per girl. Yearly rate within the window is `kidsPerGirl / window`.

### Catastrophic event (optional)

* **Enabled** – Toggle.
* **Every N years** – Period of the event.
* **Kill fraction (%)** – Percentage removed across all ages when it hits.
* **Starts at year** – When the first event occurs. Catastrophes apply **after** births and natural deaths for that year.

### Chart

* **Y‑max selector** (HTML build) – Set the top of the linear/log scale to **100k**, **1M**, or **1B**. Linear mode keeps a −100k floor for readability.
* **Hover tooltip** – Shows: `Year`, `Population`, `B` (births this year), `ΣB` (cumulative births), `D` (deaths this year including catastrophe), `ΣD` (cumulative deaths).

### Start year estimate (optional panel)

* Maps **minimum birthing age** to a rough **“years ago”** using anchors: 8→1,000,000; 9→100,000; 12→10,000; 15→1,000; 17→0.
* Interpolates linearly between anchors; below 8 shows **PRE‑HUMAN**.

---

## Population health indicator

Projects **10,000 years** forward under current settings and labels the outcome:

* **Healthy** – final pop ≤ **100,000**
* **Medium** – ≤ **1,000,000**
* **Caution** – < **1,000,000,000,000** (1 trillion)
* **Warning** – ≥ **1 trillion**
* **Population death** – final pop < **2**

The label is a coarse guardrail for “keep growth in a reasonable band,” not a moral judgment or a policy recommendation.

---

## Math details (compact)

* **Female share**: fixed at **50%**.
* **Fertility**: `births = 0.5 × fertRate × (# people in fertile window)`, where `fertRate = kidsPerGirl / yearsInWindow`.
* **Natural deaths**: `deaths = Σ ageCounts[a] × q[a]`.
* **Aging**: survivors shift from age `a` to `a+1`; births enter age 0.
* **Catastrophe**: multiply all ages by `(1 − killFraction)` at the end of the year when scheduled.
* **Sampling**: `sampleEvery = max(1, floor(years / 1200))` → responsive charts for long runs.

---

## FAQ

**Do dead people have babies?**
No. Births are computed **before** deaths, but only living people in the fertile window count. After deaths and aging, catastrophes (if enabled) remove a fraction of the **survivors**.

**Why can the linear chart show negative values?**
The linear axis includes a small −100k floor to keep the pink line visually centered when populations are near zero. The model’s population never goes negative.

**Is the survivorship realistic?**
It’s a pragmatic fit to match three constraints (mean life, S(40), S(maxAge)). For realism you’d use full life tables by age and sex; this sandbox favors control and speed.

**What about migration, resource limits, or carrying capacity?**
Not modeled. You can emulate shocks with the catastrophe feature or by tweaking fertility/mortality over time.

---

## Performance notes

* All computation is in the browser; long horizons are kept fast via sampling and simple array math.
* The React build uses memoization; the HTML build uses a small debounce and `requestAnimationFrame` for drawing.

---

## Embedding & sharing

* **HTML**: upload the single file anywhere (static hosting, GitHub Pages) or paste into a WordPress **Custom HTML** block.
* **React**: ensure Recharts and your UI kit (e.g., shadcn/ui) are available; render the component inside your app.

---

## Known limitations & future ideas

* Single region; no cohorts by sex beyond a fixed 50% female share.
* Fixed fertility across ages within the window; no parity limits or spacing.
* No explicit infant/child mortality schedule beyond the fit.
* No density dependence (carrying capacity) or migration.

**Potential extensions**

* Age‑specific fertility schedules; male/female separation and sex ratio at birth.
* Carrying capacity / resource constraints.
* Multiple catastrophe types; stochastic events; vaccination/medicine toggles.
* CSV export of the time series (already trivial to add).

---

## Credits & license

* Charting: Recharts (React build) and a tiny Canvas renderer (HTML build).
* UI: shadcn/ui components (React build) and custom CSS (HTML build).
* Code is provided as‑is for demonstration and sandboxing. Use responsibly.
