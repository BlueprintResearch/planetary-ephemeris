# Blueprint Research Planetary Ephemeris

```
   ════════════════════════════════════════════════════════════
      BLUEPRINT RESEARCH • OPEN SOURCE PLANETARY EPHEMERIS
   ════════════════════════════════════════════════════════════

                  ★       ★       ★
                      ╱   │   ╲
                     ─    ☉    ─
                      ╲   │   ╱
                  ★       ★       ★

   ════════════════════════════════════════════════════════════
```

**Open-source TradingView Pine Script library for calculating planetary positions. Single import, validated against NASA JPL DE440, all 10 solar system bodies.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Pine Script v6](https://img.shields.io/badge/Pine%20Script-v6-blue)](https://www.tradingview.com/pine-script-docs/)
[![TradingView](https://img.shields.io/badge/TradingView-Published-green)](https://www.tradingview.com/script/f7cvFJjG-blueprint-ephemeris-lib/)

---

## Overview

**[blueprint_ephemeris_lib](https://www.tradingview.com/script/f7cvFJjG-blueprint-ephemeris-lib/)** is a consolidated planetary ephemeris library for TradingView. One import gives you geocentric and heliocentric positions for all 10 solar system bodies: Sun, Moon, Mercury, Venus, Mars, Jupiter, Saturn, Uranus, Neptune, and Pluto.

This v2 library **supersedes the original 11-library chain** (lib_vsop_core, lib_vsop87_mercury, etc.) with a single file that is 28% smaller and requires no dependency chain. A full validation audit against NASA's JPL DE440 ephemeris uncovered critical coefficient errors that have been corrected in this release.

**Theories:**
- **VSOP87D** (Bretagnon & Francou, 1988) — Mercury through Neptune
- **ELP2000-82** (Chapront-Touze & Chapront, 1983) — Moon
- **Meeus Series** (Meeus, 1998) — Pluto

## Open Source Philosophy

**Astronomical knowledge belongs to everyone.**

This project provides a free, open-source alternative to proprietary astronomical libraries. We believe that the mathematics of planetary motion should be accessible to all traders, researchers, and enthusiasts.

- **Study the code** - Learn how planetary calculations work
- **Modify freely** - Adapt to your specific needs
- **Share openly** - Help others understand the cosmos
- **Attribute properly** - Give credit where it's due

This is not just code — it's a commitment to open access in financial astrology.

## What's New in V2

- **L1[0] Precession Fix** — All 8 VSOP87 planets had incorrect longitude rate coefficients. Correcting them reduced error from ~0.75 degrees to < 0.1 degrees across the board. See [Validation](#validation-against-jpl-de440) below.
- **Single File** — 4,300 lines across 11 libraries consolidated into ~3,100 lines in one file (28% reduction).
- **Single Import** — No dependency chain. Faster execution.
- **Moon Improvements** — Functions accept raw `time` directly (no manual Julian century conversion). Node functions renamed with explicit north/south designation.

---

## Validation Against JPL DE440

Every body was validated against NASA's DE440 ephemeris (via the [Skyfield](https://rhodesmill.org/skyfield/) Python library) across **250 years (1850-2100)** with 2,314 sample points. Using only **1.6% of the full VSOP87D theory** (511 of 31,577 terms), this library achieves sub-arcminute accuracy for all planets.

### Understanding the Terminology

The following terms appear throughout the validation results:

- **RMS Error (Root-Mean-Square Error)** — A standard measure of average prediction error. It's the square root of the mean of squared differences between the library's computed position and the reference position, sampled across the full 250-year window. Lower is better.

- **L1[0] Coefficient** — In VSOP87, each planet's longitude is computed as a polynomial series in time: `L = L0(t) + L1(t)*t + L2(t)*t^2 + ...`. Each `Ln(t)` is itself a sum of cosine terms. `L1[0]` is the dominant term of `L1` — it represents the planet's mean orbital rate combined with the precession of the coordinate frame. Think of it as the planet's fundamental "speed" through the sky.

- **VSOP87B vs VSOP87D** — The VSOP87 theory comes in several variants. VSOP87B gives positions referenced to the fixed J2000.0 equinox. VSOP87D gives positions referenced to the ecliptic of date (a frame that rotates with Earth's precession). The L1[0] values differ between variants by exactly the general precession rate. The original libraries mistakenly used VSOP87B L1[0] values in a VSOP87D context.

- **General Precession (+0.24382 rad/millennium)** — Earth's rotational axis precesses like a wobbling top with a ~26,000 year period. This causes the VSOP87D coordinate grid to rotate at approximately 50.3 arcseconds per year. When this rotation rate is missing from the L1[0] coefficient, computed positions drift by approximately 0.5 degrees per century.

- **JPL DE440** — A numerical integration ephemeris from NASA's Jet Propulsion Laboratory, computed using the most precise observational data available (radar ranging, spacecraft tracking, lunar laser ranging). DE440 is the gold standard for solar system positions.

- **0.1-degree target** — One-tenth of a degree equals 6 arcminutes, roughly one-fifth the apparent diameter of the full Moon. This accuracy is more than sufficient for astrological aspect calculations (where orbs are typically 1-10 degrees), planetary ingress timing, and planetary line charting.

### Before vs After: RMS Error

![Ephemeris Error: Before vs After L1[0] Precession Fix](docs/images/before_after_rms.png)

This chart compares RMS longitude error for each body before and after the L1[0] precession fix. The Y-axis uses a logarithmic scale in degrees. The orange dashed line marks the 0.1-degree target.

**Before (red bars):** All eight VSOP87 planets show approximately 0.75 degrees of RMS error, caused by the incorrect L1[0] precession coefficient. The error is nearly identical across planets because it originates from Earth's longitude computation, which is shared by every geocentric calculation.

**After (green bars):** Every planet falls well below the 0.1-degree target. The Sun achieves 0.004 degrees (14 arcseconds), while the outer planets (Uranus, Neptune) show the largest residual at 0.013-0.017 degrees due to fewer terms in the truncated series.

Moon and Pluto are unaffected because they use ELP2000-82 and Meeus series respectively, not VSOP87. The left panel shows geocentric error (positions as seen from Earth), the right panel shows heliocentric error (positions relative to the Sun).

### Improvement Factor

![Improvement Factor: Error Reduction from L1[0] Fix](docs/images/improvement_factor.png)

This chart shows the improvement factor for each body (old RMS / new RMS). The Sun shows the largest improvement at 212x. Neptune improved 130x, Venus and Mars 95x each, Mercury 81x, Jupiter 78x, Saturn 59x, and Uranus 45x.

Moon and Pluto show 1x (no change), confirming that the fix correctly targets only VSOP87-based calculations.

### Error Over Time

![Geocentric Error vs Time: Before and After L1[0] Fix](docs/images/before_after_timeseries.png)

These six panels plot geocentric longitude error (degrees) against time from 1850 to 2100 for Sun, Mercury, Venus, Mars, Jupiter, and Saturn.

**Red line (before):** A clear linear drift centered on the year 2000 (the J2000.0 reference epoch). The error is zero at J2000.0 and grows proportionally with time distance — this is the signature of a missing linear rate term. The slope corresponds to the missing 0.24382 rad/millennium of general precession.

**Green line (after):** Flat near zero across the entire 250-year span, with only the small residual from series truncation remaining. Each panel shows the old and new RMS values in the upper-left corner.

### Accuracy Results

| Body | Heliocentric RMS | Geocentric RMS | Method |
|------|-----------------|----------------|--------|
| Sun | — | 0.006° (22") | VSOP87D |
| Mercury | 0.005° (18") | 0.011° (40") | VSOP87D |
| Venus | 0.004° (14") | 0.010° (36") | VSOP87D |
| Mars | 0.006° (22") | 0.010° (36") | VSOP87D |
| Jupiter | 0.009° (32") | 0.010° (36") | VSOP87D |
| Saturn | 0.012° (43") | 0.013° (47") | VSOP87D |
| Uranus | 0.016° (58") | 0.017° (61") | VSOP87D |
| Neptune | 0.006° (22") | 0.007° (25") | VSOP87D |
| Moon | — | 0.062° (3.7') | ELP2000-82 |
| Pluto | 0.059° (3.5') | 0.060° (3.6') | Meeus |

*Validated across 250 years (1850-2100) against JPL DE440. Uses only 511 of 31,577 VSOP87D terms (1.6%), optimized for Pine Script's token limits.*

---

## The L1[0] Precession Fix

The original libraries used VSOP87B L1[0] coefficients (heliocentric spherical, equinox J2000) where VSOP87D values (ecliptic of date) were required. The difference is exactly the general precession in longitude: **+0.24382 rad/millennium** (approximately 13.97 degrees/millennium or 50.3 arcseconds/year).

This affected all 8 VSOP87 planets because every geocentric position computation uses Earth's longitude, which carries the same L1[0] error. The error was invisible near J2000.0 (the year 2000 reference epoch) but grew linearly with time, reaching approximately 0.75 degrees RMS across the 250-year validation window.

**Corrected L1[0] values (VSOP87D, rad/millennium):**

| Planet | Corrected L1[0] |
|--------|-----------------|
| Mercury | 26088.14706222746 |
| Venus | 10213.52943052898 |
| Earth | 6283.31966747491 |
| Mars | 3340.85627474342 |
| Jupiter | 529.93480757497 |
| Saturn | 213.54295595986 |
| Uranus | 75.02543121646 |
| Neptune | 38.37687716731 |

The error was identified by comparing our truncated coefficients against the complete VSOP87D tables from [Greg Miller's vsop87-multilang project](https://github.com/gmiller123456/vsop87-multilang). Each planet's L1[0] appears in two places per library (the longitude computation and the geocentric speed computation), and all instances have been corrected.

---

## Features

- **Single-import architecture** — One library, one import, all 10 bodies
- **Validated accuracy** — Every body < 0.1° RMS against JPL DE440
- **Heliocentric & geocentric coordinates** — Both reference frames
- **Ecliptic & equatorial systems** — Longitude, latitude, declination
- **Speed calculations** — Detect retrograde motion
- **Planetary averages** — 6-planet and 8-planet composite indices
- **Lunar nodes** — Mean and true north/south node positions
- **Simplified API** — Helper functions with `time` variable directly
- **Future projection** — 250-bar look-ahead with polyline visualization
- **Truncated series** — Optimized for TradingView's execution limits

## Quick Start

### blueprint_ephemeris (Recommended)

```pinescript
//@version=6
indicator("My Planetary Indicator", overlay=true)

import BlueprintResearch/blueprint_ephemeris_lib/1 as eph

// Select a planet
planet = eph.string_to_planet("Jupiter")

// Get planetary data
longitude = eph.get_longitude(planet, time, true)   // geocentric
declination = eph.get_declination(planet, time)
speed = eph.get_speed(planet, time)

// Check if retrograde
isRetro = eph.is_retrograde(planet, time)

// Plot
plot(longitude, "Jupiter Longitude", color = isRetro ? color.red : color.green)
```

**Retrograde detection:**
```pinescript
bool mercury_retro = eph.is_retrograde(eph.Planet.Mercury, time)
bgcolor(mercury_retro ? color.new(color.red, 90) : na)
```

**Moon nodes and declination:**
```pinescript
float north_node = eph.get_mean_north_node_lon(time)
float south_node = eph.get_mean_south_node_lon(time)
float moon_decl  = eph.get_declination(time)
```

**Dynamic planet selection from input:**
```pinescript
string planet_str = input.string("Sun", "Planet",
     options=["Sun","Moon","Mercury","Venus","Mars","Jupiter","Saturn","Uranus","Neptune","Pluto"])
eph.Planet p = eph.string_to_planet(planet_str)

float geo   = eph.get_longitude(p, time, true)
float helio = eph.get_longitude(p, time, false)
float speed = eph.get_speed(p, time)
```

### Chain Libraries (Legacy)

The original 11-library architecture is still available and has received the same L1[0] fix. Use if you need individual planet imports:

```pinescript
//@version=6
indicator("My Planetary Indicator", overlay=true)

import BlueprintResearch/lib_ephemeris/1 as eph

planet = eph.string_to_planet("♂ Mars")
longitude = eph.get_longitude(planet, time, true)
```

### Future Projections with Polylines

Each library includes showcase code that demonstrates 250-bar future projections using Pine Script polylines:

```pinescript
//@version=6
indicator("Planetary Future Projection", overlay=false)

import BlueprintResearch/blueprint_ephemeris_lib/1 as eph

var int LOOK_AHEAD = 250
var eph.Planet planet = eph.Planet.Mars
var polyline lon_line = na

lon = eph.get_longitude(planet, time, true)
plot(lon, "Mars Longitude", color=color.red)

if barstate.islast
    polyline.delete(lon_line)

    array<chart.point> pts = array.new<chart.point>()
    array.push(pts, chart.point.from_time(time, lon))

    for i = 1 to LOOK_AHEAD
        future_time = time + i * 86400000  // +1 day per bar (daily chart)
        future_lon = eph.get_longitude(planet, future_time, true)
        array.push(pts, chart.point.from_time(future_time, future_lon))

    lon_line := polyline.new(pts, xloc=xloc.bar_time,
                line_color=color.new(color.red, 50),
                line_style=line.style_dotted)
```

Adjust the time increment (`86400000` ms = 1 day) to match your chart timeframe.

## Functions

**Unified API** (all planets):
- `get_longitude(Planet, time, preferGeo)` — geocentric or heliocentric longitude
- `get_declination(Planet, time)` — equatorial declination
- `get_speed(Planet, time)` — longitude speed (deg/day)
- `is_retrograde(Planet, time)` — true when retrograde
- `string_to_planet(string)` — name to enum

**Averages:**
- `get_avg6_geo_lon` / `get_avg6_helio_lon` — Mercury through Saturn
- `get_avg8_geo_lon` / `get_avg8_helio_lon` — Mercury through Neptune

**Moon** (direct access):
- `get_geo_ecl_lon(time)` / `get_geo_ecl_lat(time)` / `get_declination(time)`
- `get_mean_north_node_lon(time)` / `get_mean_south_node_lon(time)`
- `get_true_north_node_lon(time)` / `get_true_south_node_lon(time)`
- `get_north_node_declination(time)` / `get_south_node_declination(time)`

## Libraries

### Consolidated (V2 — Recommended)
- **[blueprint_ephemeris_lib](https://www.tradingview.com/script/f7cvFJjG-blueprint-ephemeris-lib/)** — All 10 bodies in a single library

### Chain Architecture (V1 — Legacy)

#### Core Utilities
- **lib_vsop_core** — Foundation library with VSOP87 evaluators and utilities

#### Planetary Libraries (VSOP87)
- **lib_vsop87_mercury** / **lib_vsop87_venus** / **lib_vsop87_mars**
- **lib_vsop87_jupiter** / **lib_vsop87_saturn** / **lib_vsop87_uranus** / **lib_vsop87_neptune**

#### Special Bodies
- **lib_elp2000_moon** — Moon positions (ELP2000-82 theory)
- **lib_meeus_pluto** — Pluto positions (Meeus algorithms)

#### Master Library
- **lib_ephemeris** — Unified API importing all planetary libraries

**Publication order** (if publishing chain libraries to your own account):
1. lib_vsop_core (no dependencies)
2. All planetary libraries (depend on lib_vsop_core)
3. lib_ephemeris (depends on all 10 above)

## Repository Structure

```
planetary-ephemeris/
├── src/
│   ├── blueprint_ephemeris.pine  # V2 consolidated library (recommended)
│   ├── DESCRIPTION.txt           # TradingView publication description
│   ├── Core_Util                 # lib_vsop_core
│   ├── Mercury                   # lib_vsop87_mercury
│   ├── Venus                     # lib_vsop87_venus
│   ├── Mars                      # lib_vsop87_mars
│   ├── Jupiter                   # lib_vsop87_jupiter
│   ├── Saturn                    # lib_vsop87_saturn
│   ├── Uranus                    # lib_vsop87_uranus
│   ├── Neptune                   # lib_vsop87_neptune
│   ├── Moon                      # lib_elp2000_moon
│   ├── Pluto                     # lib_meeus_pluto
│   └── main_ephemeris_lib        # lib_ephemeris (chain master)
├── docs/
│   └── images/                   # Validation plots
├── LICENSE
└── README.md
```

## Technical Details

### Coordinate Systems

- **Heliocentric**: Positions relative to the Sun (VSOP87 native)
- **Geocentric**: Positions relative to Earth (computed by vector subtraction)
- **Ecliptic**: Longitude/latitude in the plane of Earth's orbit
- **Equatorial**: Right Ascension/Declination in the celestial equator system

### Time Scales

- **VSOP87 Planets** (Mercury-Neptune): Julian millennia from J2000.0
  - `t = (JD - 2451545.0) / 365250.0`
- **Moon & Pluto**: Julian centuries from J2000.0
  - `T = (JD - 2451545.0) / 36525.0`

The blueprint_ephemeris library handles time conversion automatically — pass TradingView's `time` variable directly.

### Performance

All libraries use truncated series optimized for TradingView's execution limits:

- 511 VSOP87D terms total (1.6% of the full 31,577-term theory)
- 91 ELP2000-82 terms for the Moon (48 longitude + 43 latitude)
- Pluto uses Meeus analytical series (valid ~1900-2100)

The entire library fits within Pine Script's 100K token limit per library.

## Example Use Cases

- **Planetary speed oscillators** - Track acceleration/deceleration cycles
- **Retrograde indicators** - Highlight retrograde periods on charts
- **Planetary aspects** - Calculate angular relationships between planets
- **Heliocentric overlays** - Compare helio vs. geo perspectives
- **Planetary indices** - Create composite indicators from multiple planets
- **Node studies** - Analyze lunar node positions
- **Declination analysis** - Study out-of-plane motion
- **Planetary line charting** - Gann-style price/longitude overlays

## TradingView Links

All libraries are published and available on TradingView:

### V2 (Recommended)
- **blueprint_ephemeris**: https://www.tradingview.com/script/f7cvFJjG-blueprint-ephemeris-lib/

### V1 (Chain Architecture)
- **Core**: https://www.tradingview.com/script/z1Zy5i9B-lib-vsop-core/
- **Mercury**: https://www.tradingview.com/script/LtS4cY82-lib-vsop87-mercury/
- **Venus**: https://www.tradingview.com/script/4jqxHSck-lib-vsop87-venus/
- **Mars**: https://www.tradingview.com/script/xpCxFyqU-lib-vsop87-mars/
- **Jupiter**: https://www.tradingview.com/script/R9kp0lbC-lib-vsop87-jupiter/
- **Saturn**: https://www.tradingview.com/script/jC9ATVEA-lib-vsop87-saturn/
- **Uranus**: https://www.tradingview.com/script/0CTzo5sg-lib-vsop87-uranus/
- **Neptune**: https://www.tradingview.com/script/z8yHMJNT-lib-vsop87-neptune/
- **Moon**: https://www.tradingview.com/script/XsQ6sAXD-lib-elp2000-moon/
- **Pluto**: https://www.tradingview.com/script/E1N99jzO-lib-meeus-pluto/
- **Master**: https://www.tradingview.com/script/RcA3QSO7-lib-ephemeris/

## Citation & Attribution

If you use these libraries in published indicators, strategies, or research, please provide attribution:

```
Blueprint Research Ephemeris Libraries
https://github.com/BlueprintResearch/planetary-ephemeris
Licensed under MIT License
```

Example in TradingView description:
```
This indicator uses the Blueprint Research Open Source Ephemeris Libraries.
See: https://github.com/BlueprintResearch/planetary-ephemeris
```

## References

- **Meeus, Jean.** *Astronomical Algorithms* (2nd Edition, 1998)
  - Primary reference for implementation details
  - Chapters on VSOP87, ELP2000-82, Pluto, coordinate transformations

- **Bretagnon, P. & Francou, G.** "Planetary theories in rectangular and spherical variables. VSOP87 solutions." *Astronomy and Astrophysics* 202 (1988): 309-315
  - Original VSOP87 theory publication

- **Chapront-Touze, M. & Chapront, J.** "The lunar ephemeris ELP2000." *Astronomy and Astrophysics* 124 (1983): 50-62
  - Original ELP2000-82 lunar theory

- **Miller, Greg.** *vsop87-multilang* — Complete VSOP87 coefficient tables converted from original Fortran data files into accessible multi-language formats (CSV, JSON, etc.). Public domain.
  - https://github.com/gmiller123456/vsop87-multilang

- **Park, R.S. et al.** "The JPL Planetary and Lunar Ephemerides DE440 and DE441." *The Astronomical Journal* 161 (2021): 105
  - Validation ground truth ephemeris

## Acknowledgments

- **Greg Miller** — For his [vsop87-multilang](https://github.com/gmiller123456/vsop87-multilang) project, which provides the complete VSOP87D coefficient tables in accessible formats. His public-domain conversion of the original Fortran data files into CSV and multi-language code was essential for identifying the L1[0] precession coefficient errors in the original libraries.
- **Jean Meeus** - For making complex astronomical algorithms accessible
- **Bretagnon & Francou** - For VSOP87 planetary theory
- **Chapront-Touze & Chapront** - For ELP2000-82 lunar theory
- **TradingView community** - For inspiration and feedback
- **Open-source astronomy community** - For advancing accessible astronomical computation

## Contributing

Contributions are welcome! Areas for improvement:

- **Add more VSOP87 terms** - Increase precision for outer planets
- **Extend date ranges** - Improve Pluto accuracy outside 1900-2100
- **Add perturbations** - Planetary perturbations, nutation, aberration corrections
- **Performance optimization** - Reduce calculation overhead
- **Documentation** - Examples, tutorials, technical guides
- **Additional bodies** - Asteroids, dwarf planets

Please open an issue or pull request to contribute.

## License

MIT License - see [LICENSE](LICENSE) file for full text.

**In brief**: Free to use, modify, and distribute. Attribution required.

## Support

- **Issues**: Open a GitHub issue for bugs or questions
- **Discussions**: Use GitHub Discussions for general questions
- **TradingView**: Comment on published libraries

---

守護之獅 (Guardian Lions) — Protecting and sharing astronomical knowledge for all.

&copy; 2025-2026 BlueprintResearch &bull; Licensed under MIT License
