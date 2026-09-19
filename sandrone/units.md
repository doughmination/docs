# Unit reference

Every unit `/convert` knows, by category. Around 100 of them across 12 categories.

## How the options behave

Both unit options suggest as you type, and they narrow each other — once one side is filled in, only units in the same category are offered on the other. That's why you rarely need this page in practice; it's here for when you want to know whether something is supported before you go looking.

A few details worth knowing:

- **Symbols match exactly, case included, before names do.** `b` gets you bits, `B` gets you bytes. That distinction matters enough to be worth the sharp edge.
- **Aliases work.** The shorthand you'd actually type — `kph`, `lbs`, `mph`, `floz`, `kcal`, `ft2` — is recognised alongside the formal name.
- **Cross-category conversions are refused with a reason.** Metres into kilograms tells you one is length and the other is mass, rather than returning a nonsense number.
- **Results are formatted for reading**, with thousands separators and trailing zeros trimmed. Very large and very small numbers switch to scientific notation.

## Temperature

Offsets are handled properly, so 0 °C is 32 °F and not 0 °F.

| Unit | Symbol | Also answers to |
|---|---|---|
| Kelvin | `K` | `k` |
| Celsius | `°C` | `c`, `degc`, `centigrade` |
| Fahrenheit | `°F` | `f`, `degf` |
| Rankine | `°R` | `r`, `degr` |

## Length

| Unit | Symbol | Also answers to |
|---|---|---|
| Metre | `m` | `meter`, `metres`, `meters` |
| Nanometre | `nm` | `nanometer` |
| Micrometre | `µm` | `um`, `micron`, `micrometer` |
| Millimetre | `mm` | `millimeter` |
| Centimetre | `cm` | `centimeter` |
| Kilometre | `km` | `kilometer` |
| Inch | `in` | `inches` |
| Foot | `ft` | `feet` |
| Yard | `yd` | `yards` |
| Mile | `mi` | `miles` |
| Nautical mile | `nmi` | |
| Light-year | `ly` | `lightyear` |
| Astronomical unit | `AU` | |

## Mass

| Unit | Symbol | Also answers to |
|---|---|---|
| Kilogram | `kg` | `kilo`, `kilos` |
| Microgram | `µg` | `ug` |
| Milligram | `mg` | |
| Gram | `g` | `grams` |
| Tonne | `t` | `tonnes`, `metricton` |
| Ounce | `oz` | `ounces` |
| Pound | `lb` | `lbs`, `pounds` |
| Stone | `st` | |
| US ton | `ton` | `shortton` |
| Carat | `ct` | |

## Digital storage

Decimal and binary units are both here and are kept distinct — a gigabyte is 10⁹ bytes, a gibibyte is 2³⁰.

| Unit | Symbol | Also answers to |
|---|---|---|
| Byte | `B` | `bytes` |
| Bit | `b` | `bits` |
| Kilobyte | `kB` | `kb` |
| Kibibyte | `KiB` | |
| Megabyte | `MB` | |
| Mebibyte | `MiB` | |
| Gigabyte | `GB` | |
| Gibibyte | `GiB` | |
| Terabyte | `TB` | |
| Tebibyte | `TiB` | |
| Petabyte | `PB` | |
| Pebibyte | `PiB` | |

## Data rate

| Unit | Symbol | Also answers to |
|---|---|---|
| Bit/second | `bit/s` | `bps` |
| Kilobit/second | `kbit/s` | `kbps` |
| Megabit/second | `Mbit/s` | `mbps` |
| Gigabit/second | `Gbit/s` | `gbps` |
| Byte/second | `B/s` | |
| Kilobyte/second | `kB/s` | |
| Megabyte/second | `MB/s` | |
| Gigabyte/second | `GB/s` | |

!!!secondary
This is the category for settling the "but my 100 Mbps connection only downloads at 12 MB/s" argument. It isn't broken; those are the same number.
!!!

## Time

Months and years use the average Gregorian lengths — 30.44 days and 365.2425 days — so a "year" here is not exactly 365 days.

| Unit | Symbol | Also answers to |
|---|---|---|
| Second | `s` | `sec`, `secs`, `seconds` |
| Nanosecond | `ns` | |
| Microsecond | `µs` | `us` |
| Millisecond | `ms` | |
| Minute | `min` | `mins`, `minutes` |
| Hour | `h` | `hr`, `hrs`, `hours` |
| Day | `d` | `days` |
| Week | `wk` | `weeks` |
| Month | `mo` | `months` |
| Year | `yr` | `years` |

## Speed

| Unit | Symbol | Also answers to |
|---|---|---|
| Metre/second | `m/s` | `ms` |
| Kilometre/hour | `km/h` | `kph`, `kmh` |
| Mile/hour | `mph` | |
| Foot/second | `ft/s` | `fps` |
| Knot | `kn` | `kt`, `knots` |
| Mach | `Ma` | |

## Area

| Unit | Symbol | Also answers to |
|---|---|---|
| Square metre | `m²` | `m2`, `sqm` |
| Square millimetre | `mm²` | `mm2` |
| Square centimetre | `cm²` | `cm2` |
| Square kilometre | `km²` | `km2` |
| Square inch | `in²` | `in2` |
| Square foot | `ft²` | `ft2`, `sqft` |
| Square yard | `yd²` | `yd2` |
| Acre | `ac` | `acres` |
| Hectare | `ha` | |
| Square mile | `mi²` | `mi2` |

## Volume

US and imperial measures are separate units, because they are different sizes and conflating them is how recipes go wrong.

| Unit | Symbol | Also answers to |
|---|---|---|
| Litre | `L` | `liter`, `litres`, `liters` |
| Millilitre | `ml` | `milliliter` |
| Centilitre | `cl` | `centiliter` |
| Cubic centimetre | `cm³` | `cc`, `cm3` |
| Cubic metre | `m³` | `m3` |
| US teaspoon | `tsp` | |
| US tablespoon | `tbsp` | |
| US fluid ounce | `fl oz` | `floz` |
| US cup | `cup` | `cups` |
| US pint | `pt` | |
| US quart | `qt` | |
| US gallon | `gal` | |
| Imperial pint | `imp pt` | |
| Imperial gallon | `imp gal` | |

## Pressure

| Unit | Symbol | Also answers to |
|---|---|---|
| Pascal | `Pa` | |
| Hectopascal | `hPa` | |
| Kilopascal | `kPa` | |
| Millibar | `mbar` | |
| Bar | `bar` | |
| Atmosphere | `atm` | |
| Pound/square inch | `psi` | |
| Torr | `Torr` | `mmhg` |

## Energy

| Unit | Symbol | Also answers to |
|---|---|---|
| Joule | `J` | `joules` |
| Kilojoule | `kJ` | |
| Calorie | `cal` | |
| Kilocalorie | `kcal` | `cals` |
| Watt-hour | `Wh` | |
| Kilowatt-hour | `kWh` | |
| Electronvolt | `eV` | |
| British thermal unit | `BTU` | |
| Foot-pound | `ft⋅lb` | `ftlb` |

!!!secondary
The "calories" on food packaging are kilocalories. One food Calorie is 1000 of the `cal` unit here.
!!!

## Angle

| Unit | Symbol | Also answers to |
|---|---|---|
| Degree | `°` | `deg`, `degrees` |
| Radian | `rad` | `radians` |
| Gradian | `gon` | `grad`, `gradians` |
| Arcminute | `′` | `arcmin` |
| Arcsecond | `″` | `arcsec` |
| Turn | `turn` | `rev`, `revolution` |

---

Missing a unit you'd use? Open an issue on the [repository](https://github.com/doughmination/sandrone/issues).
