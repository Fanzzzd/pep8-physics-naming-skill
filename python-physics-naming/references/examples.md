# Worked Examples

Drop-in examples across four subfields. Each shows the naming choice plus the citation that licenses it.

---

## 1. Classical Mechanics — Leapfrog Integrator

Single-letter vector names (`r`, `v`, `a`) are licensed by the textbook reference.

```python
import numpy as np
from numpy.typing import NDArray


def leapfrog_step(
    r: NDArray[np.float64],
    v: NDArray[np.float64],
    a: NDArray[np.float64],
    dt: float,
) -> tuple[NDArray[np.float64], NDArray[np.float64]]:
    """One kick-drift-kick step of the leapfrog integrator.

    Symbols follow Hairer, Lubich & Wanner, *Geometric Numerical
    Integration*, 2nd ed. (Springer, 2006), §I.1.4. SI units throughout.

        r  [m]       position
        v  [m/s]     velocity
        a  [m/s^2]   acceleration evaluated at r
        dt [s]       timestep

    Returns the pair (r_new, v_new).
    """
    v_half = v + 0.5 * dt * a
    r_new  = r + dt * v_half
    v_new  = v_half + 0.5 * dt * a        # caller re-evaluates a at r_new
    return r_new, v_new
```

---

## 2. Electromagnetism — Planar Wave Impedance

`Z`, `epsilon`, `mu` come straight from Jackson §7. No `Quantity`; SI is declared at the module level.

```python
"""Transmission-line helpers. SI throughout (Ω, F/m, H/m)."""

import cmath


def wave_impedance(epsilon_r: complex, mu_r: complex = 1.0) -> complex:
    """Complex wave impedance of a linear isotropic medium.

    Z = sqrt(mu / epsilon) = Z_0 * sqrt(mu_r / epsilon_r),
    with Z_0 the free-space impedance.  Reference: Jackson, *Classical
    Electrodynamics*, 3rd ed. (Wiley, 1998), eq. (7.40).
    """
    Z_0 = 376.730_313_668       # Ω, exact (vacuum wave impedance)
    return Z_0 * cmath.sqrt(mu_r / epsilon_r)
```

---

## 3. Thermodynamics — Ideal-Gas Entropy Change

`T`, `V`, `n`, `S`, `c_v` all come from the textbook. Unit suffixes are not needed because the docstring fixes SI.

```python
"""Ideal-gas thermodynamics. All quantities in SI."""

import math
from scipy.constants import R


def entropy_change_ideal_gas(
    n: float, c_v: float, T_1: float, T_2: float, V_1: float, V_2: float
) -> float:
    """Entropy change of n moles of ideal gas between two states.

    dS = n * c_v * ln(T_2/T_1) + n * R * ln(V_2/V_1)

    Reference: Callen, *Thermodynamics and an Introduction to
    Thermostatistics*, 2nd ed. (Wiley, 1985), §1.9.

        n   [mol]          amount of substance
        c_v [J/(mol*K)]    molar heat capacity at constant volume
        T_1, T_2 [K]       initial / final temperature
        V_1, V_2 [m^3]     initial / final volume
    """
    return n * c_v * math.log(T_2 / T_1) + n * R * math.log(V_2 / V_1)
```

---

## 4. Astrophysics — Kepler's Third Law with `astropy.units`

When `Quantity` objects are in use, the variable names stay bare and unit suffixes disappear.

```python
import numpy as np
import astropy.units as u
from astropy.constants import G


def keplerian_period(a: u.Quantity, M: u.Quantity) -> u.Quantity:
    """Orbital period for a Keplerian two-body orbit.

        T = 2 * pi * sqrt(a^3 / (G * M))

    Reference: Murray & Dermott, *Solar System Dynamics* (CUP, 1999),
    eq. (2.28).

        a  semi-major axis  (any length unit)
        M  central mass     (any mass unit)
    Returns the period in whatever time unit astropy picks.
    """
    return (2 * np.pi * np.sqrt(a**3 / (G * M))).to(u.s)


# Usage:
#   T = keplerian_period(1 * u.au, 1 * u.M_sun)
#   T.to(u.year) -> ~1.0 yr
```

---

## 5. Fluid Dynamics — Reynolds Number (ISO 80000-11 dimensionless)

Characteristic numbers stay as two-letter identifiers.

```python
def reynolds_number(rho: float, v: float, L: float, mu: float) -> float:
    """Reynolds number of a flow.

        Re = rho * v * L / mu

    Reference: ISO/IEC 80000-11:2019, §2-11.  SI throughout.
    """
    return rho * v * L / mu
```

---

## 6. A Public API — Descriptive Names, Keyword Arguments

The *implementation* may use terse notation with citations. The *public API* should be self-documenting.

```python
def orbital_period(
    *,
    semi_major_axis: u.Quantity,
    central_mass:    u.Quantity,
) -> u.Quantity:
    """User-facing wrapper around :func:`keplerian_period`."""
    return keplerian_period(semi_major_axis, central_mass)
```

This pattern — terse symbols inside the computation, descriptive keywords at the boundary — gives you both the one-to-one mapping to the reference text *and* an API that a user can call without reading that reference.

---

## Anti-example: what not to do

```python
# No citation, opaque symbols, unit-less floats, banned identifier `l`.
def f(x, l, v):
    return 0.5 * x * v**2 * l
```

You cannot tell what this computes, what units `x`, `l`, `v` are in, or whether `l` is an `l` at all (it's visually `1`). Fix:

```python
def kinetic_energy_per_length(rho_line: float, v: float) -> float:
    """Kinetic energy per unit length for a line of mass density rho, moving at v.

        e = 0.5 * rho * v^2   [J/m]

    SI throughout: rho_line [kg/m], v [m/s].
    """
    return 0.5 * rho_line * v**2
```

Or, with `pint`:

```python
from pint import Quantity

def kinetic_energy_per_length(rho_line: Quantity, v: Quantity) -> Quantity:
    """Kinetic energy per unit length. Units inferred from inputs."""
    return 0.5 * rho_line * v**2
```
