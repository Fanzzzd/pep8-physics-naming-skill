# Unit Suffix Catalog

When a value is a plain float (no `Quantity` object) and the module docstring does not fix units, encode the unit in the name. This file lists the suffixes `python-physics-naming` endorses.

## Rules for suffixes

1. **Lowercase** for SI base units and most derived units: `_m`, `_s`, `_kg`, `_A`, `_K`, `_mol`, `_cd`.
   Exceptions where the SI symbol is a capital: `_Pa`, `_N`, `_J`, `_W`, `_Hz`, `_C`, `_V`, `_T`, `_H`, `_F`, `_Wb`. Preserve the case.
2. **Prefixes attach directly** to the unit: `_nm`, `_um`, `_mm`, `_cm`, `_km`, `_MHz`, `_GHz`, `_mT`, `_kJ`, `_MW`. No extra underscore between prefix and unit.
3. **Angle units**: `_rad`, `_deg`. Do not mix in a single module without calling it out in the docstring.
4. **Fractions / per** use an explicit word or power: `_per_s` (preferred) or `_Hz` when it's really a frequency. Avoid `_1_s` — unreadable.
5. **Compound units**: for unambiguous products use single-token names like `_Pa_s` (dynamic viscosity — Pa·s). For ratios, always spell the divider with `_per_` to avoid W·m·K vs W/(m·K) ambiguity: `_W_per_m_K` (thermal conductivity), `_m_per_s` (speed), `_J_per_kg_K` (specific heat). Never collapse a ratio into `_W_mK`.
6. **Logarithmic / decibel**: `_dB`, `_dBm`, `_dBW`.
7. **Non-SI but common**: `_eV`, `_erg`, `_cal`, `_atm`, `_bar`, `_mbar`, `_psi`, `_ly`, `_pc`, `_AU`, `_M_sun`, `_R_sun`. These stay because astrophysics, plasma, chemistry, and engineering literature use them directly.

## SI base and derived — suffix quick reference

| Suffix     | Unit              | Quantity                         |
|------------|-------------------|----------------------------------|
| `_m`       | metre             | length                           |
| `_s`       | second            | time                             |
| `_kg`      | kilogram          | mass                             |
| `_A`       | ampere            | current                          |
| `_K`       | kelvin            | thermodynamic temperature        |
| `_mol`     | mole              | amount of substance              |
| `_cd`      | candela           | luminous intensity               |
| `_Hz`      | hertz             | frequency                        |
| `_rad`     | radian            | plane angle                      |
| `_sr`      | steradian         | solid angle                      |
| `_N`       | newton            | force                            |
| `_Pa`      | pascal            | pressure, stress                 |
| `_J`       | joule             | energy, work, heat               |
| `_W`       | watt              | power, radiant flux              |
| `_C`       | coulomb           | charge                           |
| `_V`       | volt              | voltage                          |
| `_F`       | farad             | capacitance                      |
| `_ohm`     | ohm               | resistance (Ω as text)           |
| `_S`       | siemens           | conductance                      |
| `_Wb`      | weber             | magnetic flux                    |
| `_T`       | tesla             | magnetic flux density            |
| `_H`       | henry             | inductance                       |
| `_lm`      | lumen             | luminous flux                    |
| `_lx`      | lux               | illuminance                      |
| `_Bq`      | becquerel         | activity                         |
| `_Gy`      | gray              | absorbed dose                    |
| `_Sv`      | sievert           | dose equivalent                  |
| `_degC`    | degree Celsius    | Celsius temperature              |

## Common non-SI suffixes

| Suffix     | Unit              | When used                         |
|------------|-------------------|-----------------------------------|
| `_eV`      | electronvolt      | particle / condensed-matter energy|
| `_keV`     | kiloelectronvolt  | X-ray, nuclear                    |
| `_MeV`     | megaelectronvolt  | nuclear, particle                 |
| `_erg`     | erg               | CGS astrophysics                  |
| `_cal`     | calorie           | thermochemistry                   |
| `_kcal`    | kilocalorie       | thermochemistry                   |
| `_bar`     | bar               | engineering, atmospheric          |
| `_mbar`    | millibar          | atmospheric                       |
| `_atm`     | atmosphere        | chemistry                         |
| `_psi`     | pound per sq. inch| engineering                       |
| `_mmHg`    | millimetre Hg     | clinical                          |
| `_torr`    | torr              | vacuum                            |
| `_ly`      | light-year        | astronomy                         |
| `_pc`      | parsec            | astronomy                         |
| `_kpc`     | kiloparsec        | galactic                          |
| `_Mpc`     | megaparsec        | cosmology                         |
| `_AU`      | astronomical unit | solar system                      |
| `_M_sun`   | solar mass        | astrophysics (subscript kept)     |
| `_R_sun`   | solar radius      | astrophysics                      |
| `_L_sun`   | solar luminosity  | astrophysics                      |
| `_M_earth` | Earth mass        | planetary                         |
| `_R_earth` | Earth radius      | planetary                         |
| `_angstrom`| ångström (1e-10 m)| atomic, X-ray                     |
| `_barn`    | barn (1e-28 m²)   | nuclear cross section             |
| `_Jy`      | jansky            | radio astronomy flux              |

## Prefix conventions

Attach SI prefixes directly: `_nm` (nanometre), `_um` (micrometre — `u` for µ), `_mm`, `_cm`, `_km`, `_ms` (millisecond), `_us`, `_ns`, `_ps`, `_fs`, `_MeV`, `_GeV`, `_TeV`, `_kHz`, `_MHz`, `_GHz`, `_THz`, `_kJ`, `_MJ`, `_GJ`, `_mW`, `_kW`, `_MW`, `_GW`.

Never write `_micro_m`, `_milli_s`, etc. Always collapse prefix + unit into one token.

## Edge cases

- **`ohm`** — spell out: `R_ohm`, not `R_Ω`. Greek is banned from identifiers.
- **`degC`** — spell out: `T_degC`, not `T_°C`. Better: switch to kelvin and name it `T_K`.
- **Per-unit dimensionless combinations** (e.g. mass fraction) — no suffix; document as dimensionless in the docstring.
- **Rates**: `v_m_per_s` is verbose; `v` with a module-level SI declaration is cleaner. Reserve explicit suffixes for values that actually travel in mixed-unit contexts.
- **Log-scale values**: `S_dB`, `SNR_dB`. These are the value in decibels, not the linear ratio.

## Interaction with `Quantity` objects

If the module uses `astropy.units` / `pint` / `unyt`, drop unit suffixes entirely. Two representations of the same value (`wavelength_nm = 500` *and* `wavelength = 500 * u.nm`) invite bugs. Pick one per module and stick with it.
