# RI-RS grid-optimization benchmarks for cp2k/cp2k#6434

End-to-end runs behind the timings and band-edge comparison in the description of
[cp2k/cp2k#6434](https://github.com/cp2k/cp2k/pull/6434). Each system was run with
current master and with the PR, from identical inputs, in the same SLURM job and on the same node.

| directory | system |
|---|---|
| `L2/` | armchair graphene nanoribbon segment (7-AGNR), the benchmark input attached to [cp2k/cp2k#6354](https://github.com/cp2k/cp2k/pull/6354) |
| `Si45H56/` | hydrogen-terminated silicon cluster |
| `H24P64/` | hydrogen-terminated phosphorus cluster |
| `Si293H172/` | hydrogen-terminated silicon cluster |

Each directory contains:
- `input.inp` and every file it reads: geometry (`geometry.xyz` / `struc_l2.xyz`), basis and
  potential files, and the starting wavefunction (`start.wfn`). L2 starts from `SCF_GUESS ATOMIC`.
  L2 takes `GTH_POTENTIALS` from the CP2K data directory.
- `out_master.out`: CP2K master `27d6b73051`.
- `out_pr.out`: the PR (`fc0e4fa73a`; its two changed source files are byte-identical to PR commit `bdd2e31639`;
  the later head `58a39a231d` changes only a code comment).

`SHA256SUMS` covers every file (`sha256sum -c SHA256SUMS`).

## How the runs were made
- One node, 2 × AMD EPYC 9655 (Zen 5, 192 cores), 16 MPI ranks × 12 OpenMP threads.
- Toolchain: foss-2025b (GCC 14.3, OpenMPI 5.0.8), CMake Release build with MPI, LIBINT2, LIBXC, COSMA and FFTW3.
- `OMP_PLACES=cores OMP_PROC_BIND=close`,
  `srun --ntasks=16 --cpus-per-task=12 --cpu-bind=cores cp2k.psmp -i input.inp -o out.out`.
- Default `&GRID_OPTIMIZATION` settings (`MAX_ITER 300`). The timer tables at the end of each output give the
  grid-optimization and total times.

## Notes
- **Si45H56 and Si293H172:** their `BASIS_AO` (`AUG-SZV-MOLOPT-GTH-TIER-2_1`) has the same exponents and shells as
  `aug-SZV-MOLOPT-GTH-tier-2`, but some contraction coefficients differ (Si 2nd s, 2nd p, d; and H).
- One run per build and system. Repeats of shorter runs (20 optimizer steps) differed by less than 10 % in time per step.
