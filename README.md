# Blueprint Research Planetary Ephemeris Libraries

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

**Open-source TradingView Pine Script libraries for calculating planetary positions using VSOP87, ELP2000-82, and Meeus algorithms.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Pine Script v6](https://img.shields.io/badge/Pine%20Script-v6-blue)](https://www.tradingview.com/pine-script-docs/)
[![TradingView](https://img.shields.io/badge/TradingView-Published-green)](https://www.tradingview.com/)

---

## Overview

This repository contains a complete suite of **11 TradingView Pine Script libraries** for calculating planetary ephemeris (astronomical positions). The libraries implement:

- **VSOP87 Theory** - High-precision planetary positions for Mercury through Neptune
- **ELP2000-82 Theory** - Lunar position calculations (Meeus truncated version)
- **Meeus Algorithms** - Pluto calculations valid 1900-2100

All libraries are published to TradingView and ready to import into your indicators and strategies.

## Open Source Philosophy

**Astronomical knowledge belongs to everyone.**

This project provides a free, open-source alternative to proprietary astronomical libraries. We believe that the mathematics of planetary motion should be accessible to all traders, researchers, and enthusiasts.

- **Study the code** - Learn how planetary calculations work
- **Modify freely** - Adapt to your specific needs
- **Share openly** - Help others understand the cosmos
- **Attribute properly** - Give credit where it's due

This is not just code—it's a commitment to open access in financial astrology.

## Features

- **11 modular libraries** - Use individually or together
- **Heliocentric & geocentric coordinates** - Both reference frames supported
- **Ecliptic & equatorial systems** - Longitude, latitude, declination
- **Speed calculations** - Detect retrograde motion
- **Planetary averages** - 6-planet and 8-planet composite indices
- **Simplified API** - Helper functions reduce boilerplate by 73%
- **Future projection** - 250-bar look-ahead with polyline visualization
- **Showcase code** - Ready-to-use demo code in each library
- **Truncated series** - Optimized for TradingView's execution limits
- **Well-documented** - Extensive comments and technical references

## Libraries

### Core Utilities
- **lib_vsop_core** - Foundation library with VSOP87 evaluators and utilities

### Planetary Libraries (VSOP87)
- **lib_vsop87_mercury** - Mercury positions
- **lib_vsop87_venus** - Venus positions
- **lib_vsop87_mars** - Mars positions
- **lib_vsop87_jupiter** - Jupiter positions
- **lib_vsop87_saturn** - Saturn positions
- **lib_vsop87_uranus** - Uranus positions
- **lib_vsop87_neptune** - Neptune positions

### Special Bodies
- **lib_elp2000_moon** - Moon positions (ELP2000-82 theory)
- **lib_meeus_pluto** - Pluto positions (Meeus algorithms)

### Master Library
- **lib_ephemeris** - Unified API importing all planetary libraries

## Quick Start

### Installation

Import the master library into your TradingView indicator:

```pinescript
//@version=6
indicator("My Planetary Indicator", overlay=true)

import BlueprintResearch/lib_ephemeris/1 as eph

// Select a planet
planet = eph.string_to_planet("♂ Mars")

// Get planetary data
longitude = eph.get_longitude(planet, time, true)  // geocentric
declination = eph.get_declination(planet, time)
speed = eph.get_speed(planet, time)

// Check if retrograde
isRetro = eph.is_retrograde(planet, time)

// Plot
plot(longitude, "Mars Longitude", color = isRetro ? color.red : color.green)
```

### Future Projections with Polylines

Each library includes commented showcase code that demonstrates 250-bar future projections using Pine Script polylines. This allows you to visualize where planetary positions will be in the future.

**How it works:**
1. Historical data plots normally using `plot()`
2. On the last bar, the code calculates future positions
3. Polylines extend the visualization into the future

**Example - Future Projection Setup:**

```pinescript
//@version=6
indicator("Planetary Future Projection", overlay=false)

import BlueprintResearch/lib_ephemeris/1 as eph

// Configuration
var int LOOK_AHEAD = 250  // bars into the future
var eph.Planet planet = eph.Planet.Mars

// Polyline reference (persists across bars)
var polyline lon_line = na

// Historical plot
lon = eph.get_longitude(planet, time, true)
plot(lon, "Mars Longitude", color=color.red)

// Future projection on last bar
if barstate.islast
    polyline.delete(lon_line)

    array<chart.point> pts = array.new<chart.point>()
    array.push(pts, chart.point.from_time(time, lon))  // connect to current

    for i = 1 to LOOK_AHEAD
        future_time = time + i * 86400000  // +1 day per bar (daily chart)
        future_lon = eph.get_longitude(planet, future_time, true)
        array.push(pts, chart.point.from_time(future_time, future_lon))

    lon_line := polyline.new(pts, xloc=xloc.bar_time,
                line_color=color.new(color.red, 50),
                line_style=line.style_dotted)
```

**Notes:**
- Adjust the time increment (`86400000` ms = 1 day) to match your chart timeframe
- The `force_overlay=false` parameter keeps projections in the indicator pane
- Delete and recreate polylines each bar to update the projection
- See the showcase code in each library for complete examples

### Publication Order on TradingView

When publishing these libraries to your own TradingView account, follow this order:

1. **lib_vsop_core** (no dependencies) ← Start here
2. **Planetary libraries** (depend on lib_vsop_core):
   - lib_vsop87_mercury
   - lib_vsop87_venus
   - lib_vsop87_mars
   - lib_vsop87_jupiter
   - lib_vsop87_saturn
   - lib_vsop87_uranus
   - lib_vsop87_neptune
   - lib_elp2000_moon
   - lib_meeus_pluto
3. **lib_ephemeris** (depends on all 10 above) ← Finish here

## Repository Structure

```
Planetary_Ephemeris/
├── src/                      # Source directory for all library files
│   ├── Core_Util             # lib_vsop_core - Foundation library
│   ├── Mercury               # lib_vsop87_mercury
│   ├── Venus                 # lib_vsop87_venus
│   ├── Mars                  # lib_vsop87_mars
│   ├── Jupiter               # lib_vsop87_jupiter
│   ├── Saturn                # lib_vsop87_saturn
│   ├── Uranus                # lib_vsop87_uranus
│   ├── Neptune               # lib_vsop87_neptune
│   ├── Moon                  # lib_elp2000_moon
│   ├── Pluto                 # lib_meeus_pluto
│   └── main_ephemeris_lib    # lib_ephemeris - Master library
├── tradingview_descriptions/ # TradingView publication descriptions
├── LICENSE                   # MIT License
└── README.md                 # This file
```

## Technical Details

### Coordinate Systems

- **Heliocentric**: Positions relative to the Sun (VSOP87 native)
- **Geocentric**: Positions relative to Earth (computed by vector subtraction)
- **Ecliptic**: Longitude/latitude in the plane of Earth's orbit
- **Equatorial**: Right Ascension/Declination in celestial equator system

### Time Scales

Different astronomical theories use different time scales:

- **VSOP87 Planets** (Mercury-Neptune): Julian millennia from J2000.0
  - `t = (JD - 2451545.0) / 365250.0`
  - Use `core.get_julian_millennia(time)`

- **Moon & Pluto**: Julian centuries from J2000.0
  - `T = (JD - 2451545.0) / 36525.0`
  - Use `core.get_julian_centuries(time)`

The master library (`lib_ephemeris`) handles time conversion automatically.

### Accuracy

- **VSOP87 Planets**: ~1-10 arcseconds (truncated series)
- **Moon**: ~10 arcseconds (48 main perturbation terms)
- **Pluto**: ~1 arcminute within 1900-2100

Accuracy is suitable for financial astrology and educational purposes. For professional astronomical research, use full-precision ephemeris (JPL DE440/441).

### Performance

All libraries use truncated series optimized for TradingView's execution limits:

- Core_Util: ~15-20 Earth terms
- Planetary libraries: ~10-15 terms per VSOP87 series
- Moon: 48 perturbation terms (vs. 200+ in full ELP2000-82)

To increase precision, add more terms from source references (see CLAUDE.md).

## Example Use Cases

- **Planetary speed oscillators** - Track acceleration/deceleration cycles
- **Retrograde indicators** - Highlight retrograde periods on charts
- **Planetary aspects** - Calculate angular relationships between planets
- **Heliocentric overlays** - Compare helio vs. geo perspectives
- **Planetary indices** - Create composite indicators from multiple planets
- **Node studies** - Analyze lunar node positions
- **Declination analysis** - Study out-of-plane motion

## Citation & Attribution

If you use these libraries in published indicators, strategies, or research, please provide attribution:

```
Blueprint Research Ephemeris Libraries
https://github.com/javonnii/planetary-ephemeris
Licensed under MIT License
```

Example in TradingView description:
```
This indicator uses the Blueprint Research Open Source Ephemeris Libraries.
See: https://github.com/javonnii/planetary-ephemeris
```

## References

- **Meeus, Jean.** *Astronomical Algorithms* (2nd Edition, 1998)
  - Primary reference for implementation details
  - Chapters on VSOP87, ELP2000-82, Pluto, coordinate transformations

- **Bretagnon, P. & Francou, G.** "Planetary theories in rectangular and spherical variables. VSOP87 solutions." *Astronomy and Astrophysics* 202 (1988): 309-315
  - Original VSOP87 theory publication

- **Chapront-Touzé, M. & Chapront, J.** "The lunar ephemeris ELP2000." *Astronomy and Astrophysics* 124 (1983): 50-62
  - Original ELP2000-82 lunar theory

## TradingView Links

All libraries are published and available on TradingView:

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

## Contributing

Contributions are welcome! Areas for improvement:

- **Add more VSOP87 terms** - Increase precision
- **Extend date ranges** - Improve Pluto accuracy outside 1900-2100
- **Add perturbations** - Planetary perturbations, nutation, aberration
- **Performance optimization** - Reduce calculation overhead
- **Documentation** - Examples, tutorials, technical guides
- **Validation** - Compare against JPL Horizons or Swiss Ephemeris

Please open an issue or pull request to contribute.

## License

MIT License - see [LICENSE](LICENSE) file for full text.

**In brief**: Free to use, modify, and distribute. Attribution required.

## Support

- **Issues**: Open a GitHub issue for bugs or questions
- **Discussions**: Use GitHub Discussions for general questions
- **TradingView**: Comment on published libraries

## Acknowledgments

- **Jean Meeus** - For making complex astronomical algorithms accessible
- **Bretagnon & Francou** - For VSOP87 planetary theory
- **Chapront-Touzé & Chapront** - For ELP2000-82 lunar theory
- **TradingView community** - For inspiration and feedback
- **Open-source astronomy community** - For advancing accessible astronomical computation

---

**守護之獅 (Guardian Lions)** - Protecting and sharing astronomical knowledge for all.

© 2025 BlueprintResearch • Licensed under MIT License
