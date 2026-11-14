# APBMA — PUF- and BIKE-Based Mutual Authentication for Vehicular Networks

Reproducibility package for APBMA, a lightweight, PUF-based mutual authentication and
BIKE-based key-agreement protocol for Vehicle-to-Infrastructure (V2I) communication, designed
to resist cloning, impersonation, replay, and man-in-the-middle attacks while remaining secure
against quantum adversaries.

This repository accompanies the associated research paper and provides everything needed to
independently inspect, re-verify, and re-run the experiments reported there: the protocol's
formal models (AVISPA/HLPSL and Scyther/SPDL), the OMNeT++/Veins/SUMO network-simulation source
code, and the original experimental result exports.

## How it works

During an offline registration phase, each vehicle's physical unclonable function (PUF) is
challenged once; the challenge-response pair and a temporary identity derived from it are
stored at the infrastructure side, so no long-term secret key needs to be kept on the vehicle
afterwards. At run time, the vehicle and a roadside unit (RSU) mutually authenticate using this
temporary identity and lightweight hash/MAC operations (messages `M3`-`M5`), then establish a
shared session key through a BIKE key-encapsulation mechanism (message `M6`), whose security
rests on the hardness of decoding a random linear code rather than on the discrete-logarithm or
factorization problems — keeping the resulting key secure even against an adversary equipped
with a large-scale quantum computer.

## Reproducibility purpose and scope

This package lets a reader:
1. Formally re-verify the protocol's security claims using the included AVISPA and Scyther
   models.
2. Re-run the network simulation that produced the authentication-latency, message-delivery,
   and communication/computation-overhead results reported in the paper.
3. Inspect the original experimental result exports used to produce the paper's tables and
   figures, without any modification.


## Repository structure

```text
APBMA_Reproducibility/
│
├── README.md                              <- this file
├── LICENSE
│
├── Formal_Verification/
│   ├── AVISPA/
│   │   ├── v2x_hardened.hlpsl              <- HLPSL model of APBMA's run-time phase
│   │   ├── AVISPA_Instructions.md          <- install, run, and read SAFE/UNSAFE/INCONCLUSIVE
│   │   └── Verification_Results/
│   │       └── NOTE.md                     <- status: no AVISPA run captured yet (see below)
│   │
│   └── Scyther/
│       ├── v2x_hardened.spdl               <- lightweight model (3 claims/role)
│       ├── v2x_hardened_full.spdl          <- full model (9 claims/role)
│       ├── Scyther_Instructions.md         <- install, run, and read the claim table
│       └── Verification_Results/
│           ├── v2x_hardened_raw_output.txt       <- actual captured Scyther output (3-claim)
│           └── v2x_hardened_full_raw_output.txt  <- actual captured Scyther output (9-claim)
│
├── Simulation/
│   ├── APBMA.ini                           <- self-contained OMNeT++ configuration
│   ├── TraCIDemo11p.{h,cc,ned}             <- vehicle application (APBMA protocol)
│   ├── TraCIDemoRSU11p.{h,cc,ned}          <- RSU application (APBMA protocol)
│   ├── analyze_apbma.py                    <- computes the paper's evaluation metrics from a .sca/CSV export
│   ├── suggest_rsu_positions.py            <- finds RSU coordinates for a given SUMO scenario
│   └── README_Simulation_Notes.md          <- IMPORTANT: explains the Veins dependency (read first)
│
├── Results/
│   └── Excel/                              <- original experimental CSV exports (see table below)
│
└── Documentation/
    ├── Installation_and_Requirements.md    <- OMNeT++/Veins/SUMO versions and setup
    ├── Simulation_Instructions.md          <- compile, run, extract, compare results
    ├── AVISPA_Instructions.md              <- pointer to Formal_Verification/AVISPA/ (not duplicated)
    └── Scyther_Instructions.md             <- pointer to Formal_Verification/Scyther/ (not duplicated)
```

## Required software and versions

| Tool | Version used for the results in the paper |
|---|---|
| OMNeT++ | 6.0.3 |
| Veins | 5.3.1 (also compiles against 5.2) |
| SUMO | 1.18.0 |
| OS (simulation) | Ubuntu 24.04 LTS, x86_64 |
| Scyther | built from `cascremers/scyther` GitHub `main` branch (no tagged release number available) |
| AVISPA | not yet run for this repository — see `Formal_Verification/AVISPA/Verification_Results/NOTE.md` |

Full installation steps: `Documentation/Installation_and_Requirements.md`.

## Formal verification

### AVISPA

```bash
avispa v2x_hardened.hlpsl --ofmc
avispa v2x_hardened.hlpsl --cl-atse
```

\small The HLPSL file `v2x_hardened.hlpsl` was run with `avispa v2x_hardened.hlpsl --ofmc` and
`--cl-atse` (AVISPA version `\fillin{x.y}`, `\fillin{OS}`).\normalsize

**Status: these commands have not yet been executed for this repository.** No AVISPA
installation was available in the environment used to assemble this package, so — in keeping
with this project's scientific-integrity requirement — **no SAFE/UNSAFE/INCONCLUSIVE result is
reported, claimed, or fabricated here.** The `\fillin{}` placeholders above must be replaced
only after you run the commands yourself and observe the actual version string and verdict; see
`Formal_Verification/AVISPA/AVISPA_Instructions.md` for full install/run/interpretation steps
and `Verification_Results/NOTE.md` for exactly what to save once you do.

### Scyther

```bash
scyther-linux --plain --max-runs=3 v2x_hardened.spdl
scyther-linux --plain --max-runs=5 v2x_hardened.spdl
scyther-linux --plain v2x_hardened_full.spdl
```

**Status: actually executed** (captured raw output in
`Formal_Verification/Scyther/Verification_Results/`). All claims — `Reachable`, `Secret(Ks)`,
`Nisynch` in the lightweight model, plus `Secret(R)`, `Secret(C)`, `Secret(theta)`, `Alive`,
`Weakagree`, `Niagree` in the full model — report `Ok` with no attack found within the explored
bounds, for both roles (`V` and `RSU`). See `Formal_Verification/Scyther/Scyther_Instructions.md`
for exactly which security property each claim certifies, and the raw `.txt` files for the
unedited tool output.

## OMNeT++ / Veins / SUMO simulation

See `Documentation/Installation_and_Requirements.md` (install) and
`Documentation/Simulation_Instructions.md` (configure, compile, run, extract, compare) for the
full step-by-step procedure. **Read `Simulation/README_Simulation_Notes.md` first** — it
explains that `APBMA.ini` reuses Veins' own stock `RSUExampleScenario` network rather than
redefining it, so a working Veins installation (which already provides that network and its
supporting files) is required before `APBMA.ini` can be run.

## Supplied experimental datasets (`Results/Excel/`)

Every CSV attachment supplied for this project is included, unmodified, with its original
filename. These are OMNeT++ scalar-result exports (one row per recorded scalar); they can be
opened directly in Excel or with `pandas.read_csv`.

**Important, factual note on dataset provenance:** the `configname` actually recorded inside
each file was extracted directly from the file itself (column `attrname=configname`) and is
listed below. Only **`N-27-2RSU-100V-04.csv`** records `config = APBMA`, matching the
`[Config APBMA]` section of the `APBMA.ini` included in this repository, and it is the dataset
analyzed in the paper's main evaluation section. They are included in full, unmodified, for transparency
and traceability, but a reader should not assume every file in this folder represents a
finished, current-configuration APBMA run; the `config`/vehicle/RSU columns below are the
objective, file-derived facts needed to tell them apart.

| File | Recorded `config` | Vehicles | RSUs | Notes |
|---|---|---:|---:|---|
| `N-27-2RSU-100V-04.csv` | **APBMA** | 100 | 2 | **Primary dataset** — analyzed in the paper's evaluation section (authentication, delivery, communication/computational overhead). |
| `New-Data-2RSU-100V-Becone-00.csv` | WithBeaconing | 100 | 2 | Beaconing-variant comparison run; not the `APBMA` config. |
| `New-Data-channels.csv` | WithChannelSwitching | 100 | 2 | Channel-switching-variant comparison run; not the `APBMA` config. |
| `Data-100v-2Rsu.csv` | Default | 100 | 2 | Baseline/default-config run, 100 vehicles, 2 RSUs. |
| `Data-100v-2Rsu1.csv` | Default | 100 | 2 | Second baseline/default-config run, 100 vehicles, 2 RSUs. |
| `N-DATA-01.csv` | Default | 100 | 2 | Default-config run, 100 vehicles, 2 RSUs. |
| `N-Data-25-100v-2RSU.csv` | Default | 100 | 2 | Default-config run, 100 vehicles, 2 RSUs. |
| `new-Data-2RSU-100V.csv` | Default | 100 | 2 | Default-config run, 100 vehicles, 2 RSUs. |
| `Data-15-50v-2RSU.csv` | Default | 50 | 2 | Default-config run, 50 vehicles, 2 RSUs. |
| `Data-17-50v-2RSU.csv` | Default | 25 | 2 | Default-config run (despite the filename, 25 vehicle scalar blocks are recorded), 2 RSUs. |
| `Data-100v-4rsu.csv` | Default | 25 | 2 | Default-config run (despite the filename, 25 vehicles / 2 RSUs are recorded, not 100/4). |
| `Data-03.csv` | Default | 5 | 2 | Small default-config diagnostic run, 5 vehicles, 2 RSUs. |
| `N-DATA.csv` | Default | 1 | 0 | Degenerate/diagnostic run: 1 vehicle, no RSU scalars recorded. |
| `new-Data_2RSU_100v-21.csv` | Default | 1 | 0 | Degenerate/diagnostic run: 1 vehicle, no RSU scalars recorded. |

## Reproducing and comparing results

1. Install OMNeT++ 6.0.3, Veins 5.3.1, and SUMO 1.18.0 (`Documentation/Installation_and_Requirements.md`).
2. Run the simulation with `APBMA.ini`'s `[Config APBMA]` (`Documentation/Simulation_Instructions.md`).
3. Export your run's scalar results to CSV and run `Simulation/analyze_apbma.py` on it.
4. Compare the printed metrics against `Results/Excel/N-27-2RSU-100V-04.csv` (the matching,
   confirmed-`APBMA`-config dataset) using the same script.

Protocol-level counters (handshake counts, message counts, byte/operation totals) should match
exactly for identical `.ini` settings and SUMO scenario; timing figures will only match if the
same assumed cryptographic costs are used, and SUMO traffic is regenerated identically (fix a
SUMO random seed if you need bit-for-bit repeatable vehicle trajectories).

## Expected output files

- Simulation: an OMNeT++ `.sca` scalar file (and, if enabled, a `.vec` vector file for
  `APBMA_authLatency_vec`) per run, exportable to CSV in the same format as `Results/Excel/`.
- AVISPA: console `SUMMARY`/`DETAILS`/`STATISTICS` output per backend, to be saved as
  `ofmc_output.txt` / `cl-atse_output.txt`.
- Scyther: a per-claim `Ok`/`Fail` line per role, as already captured in
  `Formal_Verification/Scyther/Verification_Results/`.

## Citation

If you use this repository, please cite the associated paper:




