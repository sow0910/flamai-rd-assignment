# FlamAI R&D Assignment — Parametric Curve Parameter Recovery

## Problem

Given 1500 unordered (x, y) points sampled from the parametric curve

x(t) = t·cos(θ) − e^(M|t|)·sin(0.3t)·sin(θ) + X
y(t) = 42 + t·sin(θ) + e^(M|t|)·sin(0.3t)·cos(θ)

for t in (6, 60), recover the unknown parameters θ, M, and X, given:
- 0° < θ < 50°
- −0.05 < M < 0.05
- 0 < X < 100

## Approach

1. **Visualize the raw data.** The 1500 (x, y) points, plotted directly, form a smooth
   wavy line rising from lower-left to upper-right — no noise. The wobble amplitude
   grows toward the higher end, suggesting M > 0.

2. **Implement the forward model.** A `curve(t, theta, M, X)` function was written
   directly from the given equation to generate (x, y) for any parameter guess.

3. **Manual parameter sensitivity.** θ, M, and X were varied one at a time (holding
   the others fixed) and overlaid against the real data to build intuition:
   - θ controls the overall rotation/slope of the curve.
   - M controls whether the wobble grows (M > 0) or shrinks (M < 0) along the curve.
   - X shifts the whole curve horizontally without changing its shape.
   This narrowed plausible regions but did not, by itself, give a precise fit,
   since the three parameters interact (e.g. θ also rotates how the wobble term
   projects onto x and y).

4. **Rotated-coordinate loss function.** Rather than searching for each point's
   unknown t via nearest-neighbor matching against a dense candidate curve, the
   equation's structure was exploited directly. Since (x−X, y−42) is just a
   rotation of (t, wobble) by angle θ, the rotation can be inverted algebraically:

   u = (x − X)·cosθ + (y − 42)·sinθ   → estimate of t
   v = −(x − X)·sinθ + (y − 42)·cosθ  → estimate of the wobble value

   The model's predicted wobble at that estimated t is:
   w_model = e^(M|u|)·sin(0.3u)

   The loss is the mean absolute difference between v and w_model, averaged
   over all 1500 points. This is exact (no discretization from sampling t)
   and vectorized (no nearest-neighbor search needed).

5. **Global optimization.** `scipy.optimize.differential_evolution` was run over
   the full given parameter bounds (not the narrower manually-explored ranges,
   to avoid excluding the true answer) to minimize this loss.

6. **Validation.** The optimized parameters were plugged back into the forward
   model and visually overlaid against the real data, confirming a near-perfect
   match across the entire curve.

## Result

| Parameter | Value |
|---|---|
| θ | 30° (0.5236 rad) |
| M | 0.03 |
| X | 55 |

Final mean absolute error: **2.56 × 10⁻⁶** (effectively zero).

## Final answer (Desmos / LaTeX format)

Domain: 6 ≤ t ≤ 60

Desmos graph: https://www.desmos.com/calculator/iywunw4fbu

## Files

- `Untitled2.ipynb` — full notebook (data loading, exploration, optimization, validation)
- `xy_data.csv` — provided data
- `requirements.txt` — dependencies
