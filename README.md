# G0W0 for the Cu complex: N⁴ code and low-scaling code

Two G0W0 calculations for the same Cu complex (57 atoms, charge +1, doublet, UKS)
with the two GW implementations of CP2K. Both give the same HOMO-LUMO gap for α and β spin.

| Folder | GW implementation | Input section |
|---|---|---|
| `01_GW_N4_WF_CORRELATION` | N⁴-scaling GW (the one used with LR-BSE) | `&XC / &WF_CORRELATION / &RI_RPA / &GW` |
| `02_GW_low_scaling_BANDSTRUCTURE` | low-scaling RI-RS GW (the one used with real-time BSE) | `&PROPERTIES / &BANDSTRUCTURE / &GW` |

## Result

G0W0@PBEh (35 % exact exchange) + ADMM2, energies in eV.

| | N⁴ code | low-scaling code | difference (meV) |
|---|---|---|---|
| α HOMO | −10.578 | −10.581 | −3 |
| α LUMO | −2.106 | −2.111 | −5 |
| **α gap** | **8.472** | **8.470** | −2 |
| β HOMO | −10.616 | −10.619 | −3 |
| β LUMO | −2.910 | −2.919 | −9 |
| **β gap** | **7.706** | **7.700** | −6 |

The ten highest occupied and ten lowest unoccupied levels of each spin agree within 16 meV.

Where to find the numbers:

- N⁴ code: `out.log`, tables "GW quasiparticle energies of alpha spins" and "... of beta spins".
- Low-scaling code: file `bandstructure_SCF_and_G0W0`, blocks `spin: 1` (α) and `spin: 2` (β),
  last column. The gap printed in `out.log` ("G0W0 direct band gap") is the gap over both spins.

## Setup shared by both calculations

The two inputs are identical except for the GW section and the project name.

- Functional: PBE with 35 % exact exchange (`SCALE_X 0.65`, `&HF FRACTION 0.35`), ADMM2 with `EXCH_CORRECTION_FUNC PBEX`
- Orbital basis: aug-DZVP-MOLOPT-GTH-tier-2 for H, C, N; aug-TZVP-MOLOPT-GTH-tier-2 for N1, S, Cu
- RI basis and ADMM basis: see the `&KIND` sections; basis set files are in the folders
- Non-periodic: `PERIODIC NONE`, `PSOLVER MULTIPOLE`, cell 30 Å
- Geometry: `qm_region_side_chains_N1.xyz`

## GW settings

N⁴ code: `QUADRATURE_POINTS 200`, `NPARAM_PADE 24`, ten occupied and ten unoccupied levels per spin.
With `QUADRATURE_POINTS 100` and the default Padé order the gaps change by at most 6 meV.

Low-scaling code:

- `CUTOFF_RADIUS_RL_AO` is not set. A value of 5 Å cuts the diffuse functions of the augmented
  basis and produces a negative gap. Without the keyword the radius is chosen from the basis set.
- `CUTOFF_RADIUS_RI` is not set (default 3 Å).
- `RS_AO_RATIO 8`. With `RS_AO_RATIO 4` the real-space RI grid is too coarse:
  the gaps are then 8.354 eV (α) and 7.534 eV (β), i.e. 0.12 to 0.18 eV too small.

## CP2K version

The low-scaling GW needs the ADMM fix of https://github.com/cp2k/cp2k/pull/6607
(without it, G0W0 with a hybrid functional and ADMM is wrong; the gap over both spins is 2.42 eV instead of 7.66 eV).
Both calculations here were run with CP2K master (commit 019eaaba9a) plus this pull request.

```
git clone https://github.com/cp2k/cp2k.git
cd cp2k
git fetch origin pull/6607/head:gw-hybrid-vxc
git checkout gw-hybrid-vxc
```

## Cost

Noctua 2, 4 nodes, 64 MPI ranks × 8 OpenMP threads: N⁴ code 447 s, low-scaling code 548 s (SCF included).
