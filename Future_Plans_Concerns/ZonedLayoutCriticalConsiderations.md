# Zoned Layout Critical Considerations

> [!IMPORTANT]
> **Current Status Note:** The proposed resolution-based percentage scaling constraints described below are not the perfect final solution, but this design-time enforcement is what we want for now.

---

## 🔍 The Caveats of the Current Percentage-Based Implementation

Currently, Plaisoram stores screen layout zone coordinates as percentages (`xPercent`, `yPercent`, `widthPercent`, `heightPercent`). While this makes the database resolution-agnostic and maintains screen-independent scaling, it introduces a major semantic mismatch.

### 🔴 The Critical Flaw: Design-Time vs. Physical Screen Minimums
Enforcing a size minimum in the editor (e.g., preventing a zone from being smaller than `200px` width) is a **design-intent contract**, not a **physical hardware guarantee**. Because sizes are saved as percentages, the actual physical pixels rendered on a TV screen depend entirely on that TV's resolution.

| Design Reference Resolution | Target Player Resolution | 10.42% Width | Actual Physical Pixels | Meets 200px Minimum? |
| :--- | :--- | :--- | :--- | :--- |
| 1920 × 1080 | 1920 × 1080 (1080p) | 10.42% | 200px | ✅ Yes |
| 1920 × 1080 | 1280 × 720 (720p) | 10.42% | 133px | ❌ No (Readability Breaks) |
| 1920 × 1080 | 3840 × 2160 (4K) | 10.42% | 400px | ✅ Yes (Overkill) |

If a layout designed on a 1080p reference is fetched by a player running on a 720p screen, raw percentage calculations make the Weather and News widgets 33% smaller, breaking readability of text like city names or news headlines.

---

## 🛠️ The 4 Recommendations for a Robust Signage System

To turn a basic percentage layout into a professional, production-ready digital signage engine, we should consider the following changes:

### 💡 Recommendation 1: Code Semantics & Naming
Rename variables in the codebase to make this distinction clear (e.g., `designMinWidthPx` or `referenceMinWidthPercent`), preventing future developers from assuming the editor constraints guarantee physical screen sizes.

### 💡 Recommendation 2: Two-Layer Validation (Editor UX + Player Safety)
Professional digital signage systems split layout constraints into two boundaries:
1. **Layer 1 (Frontend Editor):** Prevents the user from making obviously poor design choices on the canvas.
2. **Layer 2 (Android Player Runtime):** Acts as the final safety guard. The player is the only component that knows the physical hardware resolution. It should verify:
   ```kotlin
   val actualWidthPx = (zone.widthPercent / 100.0) * screenWidth
   if (actualWidthPx < widget.hardMinWidthPx) {
       // Trigger fallback (e.g. simplified layout, log warning to server)
   }
   ```

### 💡 Recommendation 3: Storing Reference Resolution in Schema
If the database stores percentages without the reference resolution (e.g., 1920x1080), they are dimensionless ratios. If the default resolution of `TVCanvas.tsx` is updated in a future release, all older saved layout positions will map to incorrect values. Storing `designResolutionWidth` and `designResolutionHeight` alongside the layout makes the percentages interpretable forever.

### 💡 Recommendation 4: Aspect Ratio & Compositing
Players might render on horizontal, vertical (9:16), or ultra-wide (21:9) screens. Naively stretching percentages distorts layout elements. A compositing engine in the player should decide how to handle aspect ratio mismatches:
* **Letterbox:** Map percentages to a centered box matching the design aspect ratio (adds black bars).
* **Scale-to-fit:** Scale percentages proportionally until one dimension fills the screen, clipping the other.
* **Stretch:** Distort the layout to fill the screen (current default).

---

## 📐 The "For Now" Solution: Reference Resolution Conversion

To enforce widget limits inside `editLayout/page.tsx` for now, the editor can dynamically translate absolute pixel bounds into minimum percentages relative to the current reference resolution:

$$\text{minWidthPercent} = \left(\frac{\text{minWidthPixels}}{\text{activeResolution.width}}\right) \times 100$$
$$\text{minHeightPercent} = \left(\frac{\text{minHeightPixels}}{\text{activeResolution.height}}\right) \times 100$$

### Example (Weather Widget Constraints):
* Weather needs at least **`200px` width** and **`150px` height**.
* On a **1920 × 1080** reference resolution:
  * $\text{minWidthPercent} = (200 / 1920) \times 100 = 10.42\%$
  * $\text{minHeightPercent} = (150 / 1080) \times 100 = 13.89\%$
* The editor compares the zone's `widthPercent` and `heightPercent` with these values during resizing or configuration drops and enforces the limits.
