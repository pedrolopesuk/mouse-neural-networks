# Bio roadmap

Resume from branch `wip/rpe-learning`, at tag `bio-paused-v0` on `main`. Every step has a measured pass criterion. Do not move on until it passes on 5 seeds.

## Step 1: population pilot, then full-brain populations
Partly done: see HISTORY §10.

1. **Measure.** Run 30, 60 and 120 cells per population (240 optional) and report the minimum size that passes all four checks on 5 of 5 seeds:
   - resting excitatory median 0.5–3 Hz
   - ISI CV 0.8–1.2
   - synchrony χ < 0.35
   - smooth, monotonic drive response, with every excitatory cell below 150 Hz

   Current result: the minimum candidate is 120, passing 2 of 3 seeds, with χ borderline.
2. **Fix synchrony before scaling up:** more independent background input per cell, and a wider delay spread.
3. **Wiring rules:**
   - Projections between populations are **excitatory only**.
   - Inhibition comes from each population's own PV cells, about 20%.
   - A negative graph weight is delivered disynaptically, onto the target population's interneurons.
   - Drop the inherited mixed-sign sensory receptive fields.
4. **Measure before building an event-driven engine.** Implement lazy eligibility decay first, and require identical learning-test results and at least 10× speed-up. Only build event-driven simulation if the measured full-brain cost still exceeds real time.
5. **Convert the full brain.** The graph weight is the mean of the cell-to-cell weights, and UE shows one dot per population. Re-run every test, including the resting bands on the brain UE loads.
6. **Then finish learning:**
   - Weight-dependent STDP. Shuffled-reward drift must stay within ±10%.
   - Discrimination must reach at least 80% on decided trials.
   - Port pretraining to spiking plus RPE.
   - Short-term depression at the sensory driver, accepted when the input–output curve saturates.
   - Retire the rate engine.

## Step 2: Phase 3, the data-driven mouse model
Full specification: `docs/PHASE3_MOUSE_SPEC.md`.

## Step 3: human extension
- **Species packages:** the same engine, with per-species parameter sets (cell types, densities, connectivity, morphologies).
- **Provenance tags on every parameter:** source dataset or paper, species, measured or inferred, confidence.
- **Homology table** mapping mouse regions to human regions, used to carry mouse-derived parameters over only where human data is missing. Every carried-over parameter is tagged as inferred.
