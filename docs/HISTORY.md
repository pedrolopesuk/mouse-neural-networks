# History of findings

Each entry lists the claim, the measurement and the outcome. All rates are in biological time: the substrate steps at dt = 0.1 ms, and one engine tick advances `bio_ms` of real milliseconds.

## 1. Substrate validation (Phase 1)
Eight single-cell and synapse checks against reference behaviour, all passing:

| Check | Result |
|---|---|
| RS adaptation | last/first ISI 7.97 |
| FS adaptation | 1.50 at I = 20, inside the 1.1–1.6 range recorded for PV cells |
| CH | Bursts |
| f–I curve | Monotonic, silent below rheobase |
| Conduction delays | Exact |
| GABA | Hyperpolarises |
| STDP window | Correct sign and decay |
| Dopamine | Gates learning |

The refractory fix in §9 changed the STDP magnitudes. Correct values: +5 ms → +0.0071, −5 ms → −0.0085. The textbook value A₊·e^(−5/20) is 0.0078.

## 2. Resting rate: "10 Hz" corrected to a 1.3 Hz median
- **Claim:** the network rests at about 10 Hz, "physiological".
- **The error:** that number was measured *after* 200 ms of strong sensory drive, and the live server also injected a scripted thalamic rhythm. It was not rest. Cortical pyramidal cells have a median rate below 1 Hz, with a lognormal distribution (Buzsáki & Mizuseki 2014).
- **First true measurement:** 20 s of rest with no input:

  | Population | Median |
  |---|---|
  | Cortex | 1.75 Hz |
  | Micro-circuit | 0.35 Hz |
  | Thalamus | 23.8 Hz, every cell identical |

- **Fixes:**
  - No tonic bias on thalamic relay cells at rest.
  - A per-cell excitability offset, N(0, 1.2), which produces the skewed distribution.
- **Result:** network median 1.10–1.30 Hz, mean 3.1–3.8 Hz (mean above median), 17–22% near-silent, thalamic median 5.6–8.3 Hz.
- **Locked in as bands:** median 0.5–3 Hz, mean > median, 5–40% silent, no cell above 50 Hz at rest, thalamic median 3–20 Hz.
- **Also corrected:** "the tick rate caps firing at 18 Hz" does not apply. Each tick simulates about 55 ms at dt = 0.1 ms, so rates up to about 1 kHz are possible.
- **The brain UE loads** (rate-pretrained, then run on spiking) passes the same bands: median 1.80 Hz, mean 3.85 Hz, 7% silent, max 22 Hz, thalamic median 6.7 Hz.

## 3. Thalamic runaway fixed by adaptation (sAHP sweep)
- **Problem:** under strong input (0.9), thalamic relay cells sustained 234 / 276 Hz (median / max) for 2 s. That is a runaway, not a brief onset peak.
- **Test written first:** sustained max ≤ 100 Hz, and sustained median ≥ 35 Hz so the brain still reacts to input.
- **Fix:** a Ca²⁺-activated K⁺ afterhyperpolarisation current (τ = 100 ms, E_K = −90 mV) on thalamic relay cells only.

  | Increment | Median / max |
  |---|---|
  | 0.02 | 95 / 112 Hz (fail) |
  | 0.03 | 73 / 87 Hz (pass), onset 180 Hz |

- **Robustness:** 5 seeds × increments 0.020–0.040.
  - At 1.0× input, everything from 0.025 to 0.04 passes 5/5, with seeds varying by ±2 Hz.
  - At 1.5× input, the rate scales almost linearly (0.03 → 110 / 123 Hz) and fails 0/5. There is no gain cap.
- **Consequence:**
  - Sensory input is clamped to 0.9 (`max_sensory_input`).
  - Short-term depression at the sensory driver is a requirement. Its acceptance test is a saturating input–output curve: monotonic, sublinear, plateau at 150–200 Hz.

## 4. Endocrine decay was tied to tick size
- **Problem:** homeostasis applied once per tick, so dopamine after a reward faded 55× faster at 1 ms ticks than at 55 ms ticks.
- **Fix:** relaxation in biological time (fear τ ≈ 1 s, dopamine τ ≈ 0.75 s), with metabolic load scaled to match.
- **Result:** phasic dopamine 500 ms after a reward is 0.423 at 1 ms ticks and 0.420 at 50 ms ticks.

## 5. Modulators on the substrate

| Modulator | Effect at rest |
|---|---|
| Fear 0.9 | 3.4 → 10.7 Hz |
| Fatigue 0.9 | 3.4 → 0.4 Hz |
| Curiosity 0 → 1 (background noise) | 1.2 → 7.6 Hz |

Reward/punish in spiking mode is dopamine only. There is no backprop.

## 6. Reward weakened pathways; fixed with reward prediction error
- **Observation:** summed |w| change on sensory pathways after pairing was −4 with no reward, **−45** with reward and **+37** with punishment.
- **Cause:** sensory cells fire at about 73 Hz and cortical cells at about 5 Hz. Each sensory synapse is too weak to cause a cortical spike, so spike timing is effectively random. Pair STDP with A₋ > A₊ then nets −0.088 against +0.073 per second, so the trace records weakening, and dopamine commits it.
- **The "−4 with no reward":** the default endocrine state started dopamine at 0.25 against a tonic level of 0.20, a small burst at startup. It was not drift from the timing rule. A later no-dopamine control measured **0.00%** weight change on 5 seeds.
- **Fix:** dopamine = reward − expected reward, with a critic per stimulus context (Schultz 1997; Frémaux & Gerstner 2016).
- **Outcome:** RPE removed the sign bias. With the E/I routing in §8, the reward test passes on `wip/rpe-learning`.

## 7. Shuffled-reward drift (fix proposed: weight-dependent STDP)
- **Discrimination task** (A → output X, B → output Y, 400 trials, 5 seeds): chance in every condition. The outputs were silent: 0.01–0.03 spikes per trial, with 37–40 of the last 40 trials tied.
- **Shuffled-reward control:** sensory weights grew +86% to +124%. This is a random walk with a floor at zero. RPE removes the bias but not the diffusion.
- **Proposed fix, not yet implemented:** weight-dependent (multiplicative) STDP (van Rossum 2000). Lowering the learning rate is the secondary knob.
- **Acceptance:** shuffled drift within ±10%; discrimination from chance to at least 80% on decided trials over 5 seeds, with ties reported separately; sensory weights must not shrink.

## 8. Dale's-law bug, and the inhibitory fraction after the fix
- **Bug:** the bridge made a cell inhibitory only if its *net* outgoing weight was negative, so 0–4% of cells were inhibitory. All negative synapses from "excitatory" cells were delivered as **excitation**: 110 of 455 synapses (24%) in the UE brain and 144 of 378 (38%) in the test network.
- **Why earlier tests missed it:** synapses were weak enough to hide the missing inhibition.
- **Fix** (on `wip/rpe-learning`):
  - Every graph cell is excitatory.
  - Each column, the thalamus (TRN) and the readout get a hidden PV/LTS interneuron pool.
  - Negative graph weights become disynaptic inhibition: the source excites an interneuron of the target's column.
  - A local E→I→E loop, plus feedforward inhibition from sensory input.
- **Result:**
  - 0 sign violations.
  - **24% inhibitory** in both the test network (24 / 102) and the UE brain (30 / 127).
  - Rest is asynchronous-irregular: ISI CV 0.81, χ 0.20.
- **But:** the network is over-inhibited. Rest is 0.3–0.35 Hz, and cortex *falls* with drive (0.52 Hz at drive 0.3, then 0.17 Hz at 0.6).
- **24-point sweep** (excitation G × I/E ratio × feedforward gain): no passing region. Cortex falls with drive in 21 of 24 settings, and the readout never exceeds 4.3 Hz.

## 9. Why single cells cannot balance
- **Theory:** balanced-network theory (van Vreeswijk & Sompolinsky 1996; Brunel 2000) needs K ≈ 1000 inputs per cell, so excitation and inhibition average out.
- **Here:** each cell has K ≈ 8. Each column's 1–2 interneurons see all 8 sensory cells at about 70 Hz, saturate, and clamp the column.
- **Inherited wiring:** about 40% of sensory → cortex weights are negative, from the rate model's mixed-sign receptive fields. Under correct Dale routing, every stimulus therefore recruits net inhibition.
- **Refractory finding:** the core had no absolute refractory period, so under large conductances cells fired on every 0.1 ms step (7–10 kHz). Adding a 2 ms refractory period fixed it, and also exposed that the old STDP test values were inflated by multiple spikes per kick.

## 10. The population plan, and the pilot measurements
- **Plan:** each graph node becomes a population of spiking cells (80% excitatory, 20% PV).
  - Projections between populations are excitatory only.
  - Inhibition is local, plus disynaptic for negative graph weights.
  - The graph weight is the average of the cell-to-cell weights.
  - UE shows one dot per population, coloured by rate.
  - Precedent: Potjans & Diesmann 2014.
- **Pilot** (`tests/pilot_population.cpp`): sensory population → cortical population, sparse p = 0.2, coupling J/√K. Best parameters: JE 0.3, g 8, Jff 0.4. Seeds 42, 7 and 1234:

  | Cells per population | Seeds passing | Notes |
  |---|---|---|
  | 30 | 0 / 3 | |
  | 60 | 1 / 3 | |
  | 120 | 2 / 3 | Minimum candidate |
  | 240 | 1 / 3 | χ 0.36–0.40 |

- Rest and CV stabilise from N ≥ 120. Synchrony χ stays at 0.29–0.40 and does not fall with N. The likely cause is shared thalamic input; the proposed fix is more independent background input per cell and wider delay spread.
- **Cost:** linear in synapses. 64k synapses take 0.45 s per biological second, almost all of it the per-step eligibility scan.
  - The full brain at 120 cells per node is about 11.6k cells and about 1.4M synapses, roughly 10 s per biological second.
  - With lazy eligibility decay the estimate is 0.15–0.3 s. So lazy decay is required; a full event-driven engine is not.
