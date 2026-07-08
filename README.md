# mumax3

**MuMax3** micromagnetic simulations of magnetic nanotubes with Dzyaloshinskii–Moriya interaction (DMI), developed alongside the atomistic Monte Carlo code in [`Nanotubes-main`](https://github.com/itxasoma/Nanotubes-main) for a Master's thesis (TFM) at the Universitat de Barcelona. The goal is to reproduce, on a continuum micromagnetic model, the chiral spin textures (helical, cycloidal, skyrmion/antiskyrmion-like) studied atomistically in that companion repository, using material parameter sets grounded in the DMI literature.

---

## Physical model

MuMax3 solves the Landau–Lifshitz–Gilbert (LLG) equation for the reduced magnetization $\mathbf{m}(\mathbf{r},t)$ on a finite-difference grid:

$$\frac{\partial\mathbf{m}}{\partial t} = -\gamma\,\mathbf{m}\times\mathbf{H}_{\text{eff}} + \alpha\,\mathbf{m}\times\frac{\partial\mathbf{m}}{\partial t}$$

with the effective field $\mathbf{H}_{\text{eff}}$ built from the standard micromagnetic energy terms plus DMI:

$$E = \int_V \Big[A(\nabla\mathbf{m})^2 - K_u(\mathbf{m}\cdot\hat{\mathbf u})^2 - \mu_0 M_s\,\mathbf{m}\cdot\mathbf{H}_{ext} - \tfrac12\mu_0 M_s\,\mathbf{m}\cdot\mathbf{H}_{d} + E_{\text{DMI}}\Big]\,dV$$

where $E_{\text{DMI}}$ is either the **interfacial/Cnv** form (MuMax3's `Dind`) or the **bulk/T-symmetry** form (MuMax3's `Dbulk`), depending on the material being modeled. The geometry of interest is a hollow cylindrical shell (a nanotube), defined by an outer radius, inner radius, and length, discretized on a regular grid with a cell size kept below the material's exchange length $\ell_{ex}=\sqrt{2A/\mu_0 M_s^2}$.

---

## Repository structure

```
mumax3/
├── literature_parameters.txt      # Material parameters extracted from 3 DMI papers,
│                                     with three recommended parameter sets (A/B/C) and
│                                     matching nanotube geometry recommendations
├── setA-Cortes-Ortuno/              # .mx3 scripts/results for Set A (benchmark DMI material,
│                                       Cortés-Ortuño et al. 2018 standard problem)
├── setB-Galvez/                       # .mx3 scripts/results for Set B (Pt/Co interfacial DMI,
│                                         Mulkers et al. 2016)
├── setC-Yang/                           # .mx3 scripts/results for Set C (FeGe bulk DMI,
│                                           Niitsu et al. 2022)
├── test/                                  # Test/scratch MuMax3 scripts
├── mplstyle/                                # Matplotlib style sheet(s) for consistent figures
├── nanotube_analysis.ipynb                    # Main analysis notebook: loads MuMax3 .ovf/table
│                                                 output, computes magnetization/energy vs.
│                                                 parameters, plots spin textures
├── mayavi_mumax3.ipynb                          # 3D visualization of the simulated spin
│                                                   textures on the nanotube shell (Mayavi)
├── .gitignore
└── README.md
```

> Note: the three set folders are named after the source of each parameter set (`Cortes-Ortuno`, `Galvez`, `Yang`) rather than literally after the paper authors in every case — see `literature_parameters.txt` for the exact provenance of each set.

---

## Parameter sets

`literature_parameters.txt` distills three papers on DMI-stabilized textures into physically grounded, ready-to-use MuMax3 parameter sets:

| Set | System | $M_s$ (A/m) | $A_{ex}$ (J/m) | DMI | $K_u$ (J/m³) | Helical length $L_D$ |
|---|---|---|---|---|---|---|
| **A** | Benchmark material (Cortés-Ortuño et al., *NJP* 20, 113015, 2018) | $8.6\times10^5$ | $13\times10^{-12}$ | $D_{ind}$: 0–3.0 mJ/m² (interfacial) | $0.4\times10^6$ | ≈ 54.5 nm |
| **B** | Pt/Co ultrathin film (Mulkers et al., *PRB* 93, 214405, 2016) | $5.8\times10^5$ | $15\times10^{-12}$ | $D_{ind}$: 0–5.0 mJ/m² ($D_c\approx4.0$) | $0.8\times10^6$ | ≈ 24 nm (at D = 3.8 mJ/m²) |
| **C** | FeGe nanoparticle (Niitsu et al., *Nat. Mater.* 21, 305, 2022) | $3.84\times10^5$ | $4.75\times10^{-12}$ | $D_{bulk}$ = 0.8527 mJ/m² (bulk) | $K_{c1}=-6\times10^3$ | ≈ 70 nm |

Recommended nanotube geometries (from the same file), chosen so the tube radius is comparable to the material's helical length:

| Set | $R_{out}$ | $R_{in}$ | Length | Cell size | Grid |
|---|---|---|---|---|---|
| A | 30 nm | 25 nm | 160 nm | 2 nm | ~30×30×80 |
| B | 20 nm | 16 nm | 100 nm | 2 nm | ~22×22×50 |
| C | 50 nm | 44 nm | 200 nm | 3 nm | ~36×36×68 |

Set A and B use **interfacial (Cnv)** DMI (`Dind` in MuMax3); Set C uses **bulk (T-symmetry)** DMI (`Dbulk`) since FeGe is a B20 cubic helimagnet.

---

## Requirements

### MuMax3
- [MuMax3](https://mumax.github.io/) (GPU-accelerated micromagnetic simulator; requires an NVIDIA GPU + CUDA, or the CPU fallback build)
- Scripts are plain `.mx3` text files, run with:
  ```bash
  mumax3 script.mx3
  ```
  which produces a `script.out/` directory with `table.txt` (time series of averaged quantities) and `.ovf` files (magnetization snapshots).

### Python (analysis & visualization)
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install numpy matplotlib pandas jupyter mayavi
```
`nanotube_analysis.ipynb` reads the `table.txt`/`.ovf` output and applies the shared style in `mplstyle/`; `mayavi_mumax3.ipynb` renders the 3D spin texture on the nanotube shell.

---

## Typical workflow

1. Pick a parameter set (A, B, or C) from `literature_parameters.txt`.
2. Write/edit the corresponding `.mx3` script in `setA-Cortes-Ortuno/`, `setB-Galvez/`, or `setC-Yang/`, setting `Msat`, `Aex`, `Dind`/`Dbulk`, `Ku1`/`Kc1`, geometry (`Cylinder`/shell via `.sub`), cell size, and the relaxation/field-scan protocol.
3. Run with `mumax3 <script>.mx3` (locally or on a GPU node).
4. Open `nanotube_analysis.ipynb` to load the output table/`.ovf` files and plot magnetization, energy terms, or texture maps as a function of the scanned parameter (e.g. DMI strength, field).
5. Optionally use `mayavi_mumax3.ipynb` for a 3D rendering of the resulting spin texture on the tube surface.

---

## Background references

- A. Cortés-Ortuño et al., *New J. Phys.* **20**, 113015 (2018) — DMI micromagnetic standard problem
- I. Mulkers, M. V. Milošević, B. Van Waeyenberge, *Phys. Rev. B* **93**, 214405 (2016) — cycloidal vs. skyrmionic states in chiral magnets
- H. Niitsu et al., *Nat. Mater.* **21**, 305–310 (2022) — skyrmionic vortices in FeGe tetrahedral nanoparticles
- A. Vansteenkiste et al., *AIP Advances* **4**, 107133 (2014) — the MuMax3 code

---

## Author

**Itxaso Muñoz-Aldalur** — Universitat de Barcelona, TFM, 2025–2026. Companion repository: [`Nanotubes-main`](https://github.com/itxasoma/Nanotubes-main) (atomistic Monte Carlo code for the same nanotube systems).