# Kidney-Stone-Risk-Calculator
# KNO₃ Crystallization Risk Calculator

## What this is

An interactive tool that models kidney stone crystallization risk using our real KNO₃ solubility data. You set three patient conditions via sliders — body temperature, hydration level, and solute concentration — and the calculator tells you how close a patient is to the point where dissolved salts start forming crystals (i.e. kidney stones).

---

## The experiment

We dissolved KNO₃ (potassium nitrate) in water at 19 different temperatures from 10°C to 100°C, recording how many grams dissolved per 100 mL before the solution became saturated. The relationship is exponential — solubility accelerates with temperature rather than increasing at a steady rate.

---

## The math

### Step 1 — Exponential curve fit

We fit our data to the equation:

```
y = a · e^(bx)
```

Where `y` = grams of KNO₃ per 100 mL, `x` = temperature in °C, and `a` and `b` are constants we solve for.

To find `a` and `b`, we use **log-linear least squares regression**:
- Take `ln(y)` of all data points — this turns the exponential into a straight line
- Run standard linear regression on `(x, ln(y))` to get slope `b` and intercept `ln(a)`
- Convert back: `a = e^(intercept)`

The code does this in `fitExp()`.

### Step 2 — Effective concentration

Raw solute concentration isn't the whole picture. Dehydration makes urine more concentrated even if the total solute amount stays the same. We model this as:

```
effectiveConc = C × (100 / H)
```

Where `C` = solute concentration (g/L) and `H` = hydration level (%).

Example: at 80% hydration, `100/80 = 1.25`, so concentration is 25% higher than it would be in a fully hydrated patient.

### Step 3 — Saturation ratio

```
ratio = effectiveConc / maxSolubility(T)
```

This is the key number. It compares how much is dissolved to how much *can* dissolve at the current temperature:
- `< 1.0` = under-saturated → safe, nothing precipitates
- `= 1.0` = exactly at the limit
- `> 1.0` = supersaturated → crystallization occurs

### Step 4 — Risk % and thresholds

```
riskPercent = min(100, (ratio / 1.2) × 100)
```

| Ratio | Risk level |
|-------|------------|
| < 0.6 | Low |
| 0.6 – 0.9 | Moderate |
| 0.9 – 1.2 | High |
| > 1.2 | Critical |

---

## Why risk spikes so fast

The hydration factor (`100 / H`) is multiplicative, not additive. Going from 100% → 60% hydration multiplies concentration by `100/60 = 1.67×`. Combined with a solute concentration already near the saturation limit, small slider changes push the ratio past 1.0 quickly. This is intentional — it reflects how real kidney stone risk is disproportionately sensitive to dehydration.

**Slider ranges are set to clinically realistic values:**
- Temperature: 35–42°C (hypothermia to hyperthermia)
- Hydration: 60–100% (severe dehydration to fully hydrated)
- Concentration: 0–30 g/L (none to critically elevated)

---

## How to use it

1. Open `kidney_stone_calculator.html` in any browser — no install needed
2. Drag the sliders to set patient conditions
3. Watch the saturation ratio, risk bar, and % risk update in real time
4. The chart shows your experimental data (dots), the fitted exponential curve (orange), and a dashed line at the current temperature (green)

---

## Tech

- Plain HTML, CSS, JavaScript — no frameworks
- [Chart.js](https://www.chartjs.org/) for the graph
- Fonts: DM Sans + DM Mono via Google Fonts
- Self-contained single file, works offline once loaded
