# CAMB with Second-Order Dark Energy Pressure Parametrization

This modified version of [CAMB](https://github.com/cmbant/CAMB) implements
the second-order dark energy pressure expansion studied in
[Cheng et al. (2025), arXiv:2505.02932](https://arxiv.org/pdf/2505.02932).

The notation follows Section 2 of the paper. The scale factor is normalized to
$a=1$ today, $\rho$ and $p$ denote the dark energy density and pressure, and

```math
\Omega_{\mathrm{DE},0}\equiv\frac{\rho_{\mathrm{DE},0}}{\rho_{\mathrm{crit}}},
\qquad
\rho_{\mathrm{crit}}=\frac{3H_0^2}{8\pi G}.
```

The present dark energy abundance is calculated from the matter, radiation and
curvature abundances: $\Omega_{\mathrm{DE},0}=1-\Omega_m-\Omega_k-\Omega_r$.

At second order (Eqs. 2.8–2.12),

```math
\rho=\rho_{\mathrm{DE},0}
-\frac34(p_1+p_2)(1-a)+\frac3{10}p_2(1-a^2),
```

```math
p=-\rho_{\mathrm{DE},0}
+\left(\frac34-a\right)p_1
+\left(\frac9{20}-a+\frac12a^2\right)p_2,
```

```math
\Omega_{1,2}\equiv\frac34\frac{p_{1,2}}{\rho_{\mathrm{crit}}},
```

```math
w_{\mathrm{DE}}=-1+\frac13
\frac{\left[\Omega_1+\left(1-\frac45a\right)\Omega_2\right]a}
{\left[\Omega_1+\frac35\left(1-\frac23a\right)\Omega_2\right](1-a)-\Omega_{\mathrm{DE},0}}.
```

Setting $\Omega_2=0$ gives the first-order analytic model; setting both
$\Omega_1=\Omega_2=0$ recovers a cosmological constant. This expands the
pressure $p(a)$ about $a=1$, not the CPL equation of state.

## Key Modifications

Relative to the upstream CAMB base commit
[`c525b16`](https://github.com/cmbant/CAMB/commit/c525b16), the model changes are:

- [`fortran/DarkEnergyInterface.f90`](fortran/DarkEnergyInterface.f90) — analytic dark energy density and equation of state, pressure-expansion parameters, parameter input, and the numerical branches described below. Internally `grho_de` stores $a^4\rho/\rho_{\mathrm{DE},0}$, rather than $\rho$ itself.
- [`fortran/results.f90`](fortran/results.f90) — present-day massive-neutrino density bookkeeping and propagation of the matter, radiation and curvature densities to the dark energy routines.
- [`fortran/equations.f90`](fortran/equations.f90) — adapted calls to the background density/pressure interface during background and perturbation calculations.
- [`camb/dark_energy.py`](camb/dark_energy.py) — Python/Fortran field bindings and the dark energy class's `set_params` signature for the pressure-expansion coefficients.
- [`fortran/DarkEnergyFluid.f90`](fortran/DarkEnergyFluid.f90) — interface adaptation and removal of the old CPL-specific checks and analytic `wa` derivative term.
- [`fortran/DarkEnergyQuintessence.f90`](fortran/DarkEnergyQuintessence.f90) — interface-signature adaptation; no new quintessence potential is introduced.


## New Input Parameters

| Input / Python field | Paper symbol | Meaning | Default |
| --- | --- | --- | --- |
| `Omega1_chy` | $\Omega_1$ | Dimensionless first-order pressure coefficient | `0.0` |
| `Omega2_chy` | $\Omega_2$ | Dimensionless second-order pressure coefficient | `0.0` |

The existing sound-speed parameter is retained: `cs2_lam` in Fortran input and
`cs2` in the Python dark energy class, with default `1.0`. It is not a new
pressure-expansion coefficient. The analytic formulas apply when
`use_tabulated_w` is false (the default).

## Compilation

Build this repository from source with Python and a supported Fortran compiler
(`gfortran` 6 or newer for this CAMB snapshot). Initialize the `forutils`
submodule before compiling:

```bash
git submodule update --init --recursive
python -m pip install -e .
```

To build the Fortran command-line executable:

```bash
cd fortran
make camb
```

Install the two pressure-expansion forks in separate Python environments:
both provide the module named `camb`. Installing the standard PyPI `camb`
package does not install this modified model.

## Citation

If you use this modified code, please cite **both** the pressure-parametrization
paper and the original CAMB paper:

- **Hanyu Cheng, Eleonora Di Valentino, Luis A. Escamilla, Anjan A. Sen and Luca Visinelli (2025)**, *Pressure Parametrization of Dark Energy: First and Second-Order Constraints with Latest Cosmological Data*, [arXiv:2505.02932](https://arxiv.org/pdf/2505.02932).
- A. Lewis, A. Challinor and A. Lasenby, *Efficient Computation of CMB anisotropies in closed FRW universes*, Astrophys. J. 538 (2000) 473–476, [arXiv:astro-ph/9911177](https://arxiv.org/abs/astro-ph/9911177).

Please identify this repository and the commit used for reproducibility.
The upstream copyright, licence and publication conditions in
[`LICENCE.txt`](LICENCE.txt) are retained.
