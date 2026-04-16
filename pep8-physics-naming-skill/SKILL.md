---
name: pep8-physics-naming-skill
description: Physics variable naming for Python — combines PEP 8 formatting, ISO/IEC 80000 physical-quantity symbols, the Google Python Style rule that licenses short mathematical names, and unit-suffix conventions. Use when writing Python code involving physical quantities (mass, velocity, energy, force, temperature, wavelength, pressure, charge, etc.), implementing algorithms from physics papers or textbooks, scientific computing with numpy / scipy / astropy / pint / unyt, numerical solvers, or whenever variable names for physical quantities are being chosen.
---

# pep8-physics-naming-skill — Physics Variable Naming for Python

There is no single authoritative "Python physics naming standard". `pep8-physics-naming-skill` is the minimal composition that actually carries authority:

1. **PEP 8** (formatting: `snake_case`, `UPPER_SNAKE`, identifier hygiene)
2. **Google Python Style Guide §3.16** (explicitly *licenses* short math names when they match the source paper)
3. **ISO/IEC 80000** (the international standard for physical-quantity symbols — 13 parts)
4. **Unit suffixes or `Quantity` objects** (make units visible in the code, not just in comments)

Follow this skill whenever naming variables, parameters, attributes, or module-level constants that represent physical quantities.

## Core Rule

> **Descriptive `snake_case` by default. Single-letter or Greek-letter names only when they match an established mathematical convention, and only when the source is cited.**

Both halves matter. If you use `v` for velocity without citing why, a reader cannot tell whether it means velocity, voltage, volume, or variance. If you spell out `angular_momentum_about_z_axis` inside a tight integrator loop, the code stops resembling the paper it implements.

## Decision Flow

```
Is this variable a physical quantity?
├── NO  → plain PEP 8 (snake_case, descriptive).  Skill doesn't apply.
└── YES → Is this code implementing a specific paper/textbook algorithm?
          ├── NO  → Use a descriptive snake_case name.
          │         Annotate units (suffix or Quantity).  See §Units.
          └── YES → Use the paper's notation (often ISO 80000 symbols).
                    Cite the source in the docstring or a comment.
                    Annotate units.  See §Units.
                    If the paper's symbol CONFLICTS with ISO 80000,
                    follow the paper and note the deviation in the
                    docstring — the reader is holding the paper.
```

## The Three Layers, Concretely

### Layer 1: PEP 8 formatting (always)

- `snake_case` for variables and functions
- `UPPER_SNAKE_CASE` for module-level constants
- `PascalCase` for classes
- **Never use `l`, `O`, `I` as *bare* single-letter identifiers** (PEP 8 — visually confusable with `1`, `0`, `1`). When the physics literally uses these symbols (length `l`, moment of inertia `I`, electric current `I`), either spell them out (`length`, `current`) or add a disambiguating subscript (`I_mom`, `I_elec`, `l_path`).
- Trailing underscore for names that collide with Python keywords (`lambda_`, `from_`, `in_`)

### Layer 2: Google §3.16 license for short names

Google Python Style Guide §3.16 *recommends* short mathematical names **when** they match "well-established notation in a reference paper or algorithm". This is the only mainstream style guide that explicitly authorizes `v`, `r`, `T`, `rho`, etc. inside Python.

The rule comes with a condition: **you must cite the reference**. The citation lives in the function/class/module docstring, not scattered across inline comments.

```python
def leapfrog_step(r, v, a, dt):
    """Leapfrog (kick-drift-kick) integrator step.

    Symbols follow Hairer, Lubich & Wanner, *Geometric Numerical
    Integration*, 2nd ed., §I.1.4:
      r  position vector [m]
      v  velocity vector [m/s]
      a  acceleration vector [m/s^2]
      dt timestep [s]
    """
    v_half = v + 0.5 * dt * a
    r_new  = r + dt * v_half
    # ...
```

Without the docstring citation, `r`, `v`, `a` are opaque. With it, they are a direct translation of the source.

### Layer 3: ISO/IEC 80000 symbols

When you do use short names, use the ISO 80000 symbol (which is almost always the symbol from standard textbooks). The 13 parts are summarized below; the full tables live in [references/iso80000.md](references/iso80000.md).

| ISO 80000 part | Domain                        | Typical symbols                         |
|----------------|-------------------------------|-----------------------------------------|
| 3              | Space and time                | `l`, `h`, `r`, `t`, `f`, `omega`, `T`   |
| 4              | Mechanics                     | `m`, `rho`, `v`, `a`, `F`, `p`, `E`, `P` |
| 5              | Thermodynamics                | `T`, `Q`, `S`, `U`, `H`, `G`, `cp`      |
| 6              | Electromagnetism              | `q`, `I`, `V`, `E`, `B`, `R`, `C`, `L`  |
| 7              | Light and radiation           | `n`, `lambda_`, `nu`, `c`               |
| 9              | Physical chemistry            | `n` (amount), `c` (concentration)       |
| 10             | Atomic and nuclear physics    | `Z`, `A`, `N`, `E_k`                    |
| 11             | Characteristic (dimensionless) numbers | `Re`, `Pr`, `Nu`, `Ma`, `Kn`, `Fr`, `We` |
| 12             | Condensed matter              | `k` (wave vector), `epsilon_F`          |

Greek letters are spelled out in Latin: `alpha`, `beta`, `gamma`, `delta`, `epsilon`, `theta`, `kappa`, `mu`, `nu`, `rho`, `sigma`, `tau`, `phi`, `psi`, `omega`. Capital Greek for capital symbols: `Gamma`, `Delta`, `Lambda`, `Sigma`, `Omega`. `lambda_` gets a trailing underscore because `lambda` is a Python keyword.

### Layer 4: Units must be visible

Opaque floats are the #1 source of physics bugs in Python. Pick one of:

1. **`Quantity` objects** (preferred when units mix): `astropy.units`, `pint`, `unyt`.
    ```python
    import astropy.units as u
    wavelength = 656.28 * u.nm      # H-alpha line
    mass       = 1.0 * u.M_sun
    ```
2. **Fixed SI per module**, stated in the module docstring. Use bare names.
    ```python
    """Orbit propagator. All quantities in SI (m, kg, s, N, ...)."""
    ```
3. **Unit suffix** when neither of the above fits:
    ```python
    wavelength_nm = 656.28
    mass_kg       = 9.109e-31
    t_s           = 3600.0
    dt_ms         = 16.7
    ```
   Only attach the suffix when the unit is not obvious from context. `T_K` in a thermodynamics module is overkill; `T_K` in a module that also has `T_C` (Celsius) is required.

## Subscripts, Primes, Vectors

- **Subscripts** → underscore: `v_0` (initial), `x_cm` (center of mass), `T_crit`, `E_kin`, `rho_air`.
- **Primes / alternatives** → trailing underscore or descriptive word: `r_prime`, or just `r_new`.
- **Vectors vs scalars** → rely on types (`NDArray[np.float64]`, `Vector3`) rather than Hungarian prefixes. Use `_vec` suffix only when a scalar with the same name is also in scope.
- **Component access** → named attributes (`r.x`, `v.z`) or fixed indices with a comment, not `r[0]`, `r[1]`, `r[2]` with no explanation.

## Constants

Module-level `UPPER_SNAKE_CASE` with a docstring-level unit annotation — or better, import from `scipy.constants`:

```python
from scipy.constants import c, h, k, G, e, m_e, m_p  # all SI
```

If you must redefine:

```python
SPEED_OF_LIGHT = 299_792_458.0   # m/s, exact
BOLTZMANN      = 1.380_649e-23   # J/K, exact
```

## Examples

### Good

```python
def kepler_period(a, M_central):
    """Keplerian orbital period.

    T = 2*pi * sqrt(a^3 / (G*M)), Murray & Dermott, *Solar System
    Dynamics*, eq. 2.28.
      a         semi-major axis [m]
      M_central primary mass   [kg]
    Returns
      T         orbital period [s]
    """
    from scipy.constants import G
    return 2 * math.pi * math.sqrt(a**3 / (G * M_central))


def drag_force(rho, v, C_d, A):
    """Quadratic drag force magnitude, F = 0.5 * rho * v^2 * C_d * A."""
    return 0.5 * rho * v**2 * C_d * A


# Descriptive names when no paper is being implemented:
peak_wavelength_nm = 550.0
ambient_temperature_K = 293.15
electron_number_density = 1e19   # m^-3, document in module docstring
```

### Bad

```python
# Opaque short names, no citation:
def step(r, v, a, dt):                     # which paper?
    ...

# Unit-less float that will eventually be mixed up:
wavelength = 656.28                        # nm? m? Angstrom?

# Style-guide violations:
l = 1.0                                    # banned identifier (PEP 8)
Velocity = 3e8                             # should be snake_case
speedOfLight = 3e8                         # camelCase, not Pythonic
lambda = 532e-9                            # reserved keyword

# Hungarian-style vector prefix where the type already says so:
vec_r: NDArray = ...                       # just `r: NDArray`
```

## When Short Names Are NOT Appropriate

- **Public APIs** exposed to users who aren't reading the paper you're implementing. `def orbit(a, e, i, Omega, omega, nu)` is fine inside a Keplerian module; on a public function, prefer keyword arguments with descriptive aliases.
- **Glue code / data pipelines** that shuffle quantities without operating on them mathematically.
- **Any variable whose meaning depends on local context** that the reader can't recover — if `T` could plausibly be temperature, period, or tension in this file, spell it out.

## Anti-patterns

- Matching *visual* notation (`ρ`, `λ`, `π`) by using non-ASCII identifiers. Python 3 permits them; your collaborators and diff tools often don't. Always transliterate: `rho`, `lambda_`, `pi`.
- Citing the source in a free-floating comment 200 lines away. Citations go on the function/class/module that owns the symbols.
- Using a unit suffix *and* a `Quantity` object for the same value — pick one.
- Inventing your own single-letter symbol not found in ISO 80000 or the source paper. If the textbook uses `J` for moment of inertia and ISO uses `I`, follow the textbook (and note it).

## Common Pitfalls Beyond Pure Quantities

Three situations that bite physics-in-Python code and the rules for each:

### Coordinate-system conventions

Physics spherical coordinates use `(r, theta, phi)` with `theta` as the polar angle from +z. Mathematics (ISO 31-11, often in plotting libraries) swaps them: `(r, phi, theta)` with `theta` as the azimuth. **Declare your convention in the module docstring.** When interfacing with `scipy`, `matplotlib`, or `numpy` trig, also note that they take radians — annotate `_rad` or `_deg` if both appear.

Cylindrical: `(rho, phi, z)` (physics) vs `(r, theta, z)` (math). Same rule — declare it.

### Complex numbers

Python spells the imaginary unit as `j`. Electrical engineering also uses `j` (to avoid `i` = current). Physics papers use `i`. Inside Python code the literal is always `1j`, but the *variable* holding a unit imaginary should be named `i_unit` or simply not aliased. Phasors: use a `_phasor` suffix when the same quantity also exists as a real-valued time signal in scope.

### Statistics and uncertainty

- `mu` for mean **collides** with magnetic permeability and chemical potential. Disambiguate: `mean_` prefix or `mu_x` / `mu_y`.
- `sigma` for standard deviation **collides** with stress and conductivity. Use `std_` prefix or `sigma_x` / `sigma_y` when stress / conductivity are also present.
- Uncertainties: pair the value and its 1-σ uncertainty explicitly — `wavelength_nm, wavelength_nm_err`, or use `uncertainties.ufloat`.

### Tensor / index notation

When implementing something like `T_ij`, `g_munu`, `epsilon_ijk`: name the array after the tensor (`T`, `g`, `eps_levi_civita`) and document the index meaning in the docstring. Do not try to encode indices in the variable name (`T_ij` as a Python identifier loses the free-index semantics).

## Author Workflow Checklist

When writing or reviewing physics code, run through this:

- [ ] Every short name (1–3 chars) has a citation in the owning docstring.
- [ ] Every physical quantity either carries a `Quantity` type, a `_unit` suffix, or is covered by a module-level unit declaration.
- [ ] No `l`, `O`, `I` as single-letter identifiers.
- [ ] No non-ASCII identifiers.
- [ ] Greek letters spelled out; `lambda_` (keyword) gets trailing underscore.
- [ ] Subscripts use underscore (`v_0`, not `v0` unless matching the source exactly).
- [ ] Module-level constants are `UPPER_SNAKE_CASE` with units.
- [ ] Public API names are descriptive; only *internal* helpers use terse paper notation.

## Further Reference

- Full ISO 80000 symbol tables by part: [references/iso80000.md](references/iso80000.md)
- Unit suffix catalog (SI + common non-SI): [references/unit-suffixes.md](references/unit-suffixes.md)
- Worked examples (mechanics, EM, thermo, optics): [references/examples.md](references/examples.md)
