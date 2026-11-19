# MetaBrain Bio

**Goal:** the most faithful interactive mouse brain model, extensible to human.

**Status:** paused at tag `bio-paused-v0`. Resume from `docs/ROADMAP_BIO.md`. The full findings log is in `docs/HISTORY.md`, and fidelity per aspect is in `docs/FIDELITY.md`.

## What exists

| Part | Where | State |
|---|---|---|
| Spiking substrate | `include/metabrain/SpikingCore.hpp`, `src/SpikingCore.cpp` | Izhikevich neurons (7 classes); AMPA/NMDA/GABA_A/GABA_B conductances; axonal delays; STDP into eligibility, committed by dopamine; sAHP adaptation. On `wip/rpe-learning` only: 2 ms refractory period |
| Graph → substrate bridge | `src/BrainEngine.cpp` (`RebuildSpikingNetwork`, `StepSpiking`) | One spiking cell per graph node; real-time stepping (1 s wall = 1 s biology) |
| Legacy rate engine | `StepRecursiveOOP`, `StepFastParallel` | Still used for pretraining and for benchmarks above 10k neurons |
| Live server | `src/LiveServer.cpp`, run with `start_metabrain.ps1` | Spiking is the default engine; `POST /api/control {"engine":"rate"}` switches back |
| UE 5.8 viewer | `Source/MetaBrainViz`, `Source/MetaBrainCore` | Runs the engine in-process; impulses are real spikes |

## Branches
- `main` is the stable, all-green state: Phase 1, one spiking cell per node.
- `wip/rpe-learning` holds the work in progress, and its tests are **not** all green:
  - reward prediction error critic
  - E/I columns with hidden interneuron pools
  - Dale's-law tests
  - refractory period
  - the population pilot (`tests/pilot_population.cpp`)
  - sweep tools

## Build and run (Windows, VS 2026 Build Tools)
```powershell
$vs = "C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\VC\Auxiliary\Build\vcvars64.bat"
cmd /c "`"$vs`" && cmake -S . -B build2 -G Ninja -DCMAKE_BUILD_TYPE=Release && ninja -C build2"
build2\spiking_tests.exe      # biological validation suite
build2\metabrain_tests.exe    # legacy engine suite
.\start_metabrain.ps1         # REST server on :8085 (spiking engine)
```

The UE viewer needs the editor closed while building:
```powershell
Build.bat MetaBrainUEEditor Win64 Development -Project=<path>\MetaBrainUE.uproject -WaitMutex
```

Measurement tools (all in `build2/`):

| Tool | What it measures |
|---|---|
| `measure_rest.exe [ue]` | Resting-rate distribution, on the test network or the brain UE loads |
| `sweep_ahp.exe` | Thalamic adaptation robustness |
| `learn_discrim.exe N mode [G]` | Reward-learning discrimination. `wip` branch only |
| `sweep_ei.exe`, `pilot_population.exe` | E/I balance. `wip` branch only |

## Test status at `bio-paused-v0`

| Branch | `spiking_tests` | `metabrain_tests` |
|---|---|---|
| `main` | 14/14 pass, 1 expected failure (raw-reward test: reward weakens pathways without RPE) | 5/5 |
| `wip/rpe-learning` | 15/17, with "resting-rate bands" and "smooth drive response" failing (single-cell E/I over-inhibition; see HISTORY §7–8) | 5/5 |
