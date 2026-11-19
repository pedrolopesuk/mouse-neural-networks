# Fidelity per aspect: current state and target

Fidelity scale: **0** none or decorative · **1** qualitative analogy · **2** right mechanism, hand-tuned · **3** mechanism plus parameters validated against measurements · **4** data-driven from atlases or recordings.

| Aspect | Current (`bio-paused-v0`) | Level | Target | Level |
|---|---|---|---|---|
| Scale | About 100 nodes, one spiking cell each (`main`); the pilot reaches 240 cells per population | 1 | Full mouse brain regions as populations, about 10⁵–10⁷ cells | 4 |
| Neuron model | Izhikevich, 7 classes (RS/IB/CH/FS/LTS/TC/RZ), sAHP on thalamic cells, 2 ms refractory (`wip`) | 3 | Per-type parameters fitted to Allen Cell Types electrophysiology | 4 |
| Firing statistics | Rest: median 1.1–1.8 Hz, skewed, 7–22% near-silent. Thalamic cells: 73 Hz sustained when driven. Asynchronous-irregular only in the pilot at N ≥ 120 (2 of 3 seeds) | 2–3 | Region- and type-specific rate distributions matching in-vivo recordings | 4 |
| Wiring | Hand-built graph; Dale correct only on `wip` (24% inhibitory); mixed-sign legacy input | 1 | Allen Mouse Connectivity projection strengths, layer-specific local circuits | 4 |
| Synapses | AMPA, NMDA (Mg²⁺ block), GABA_A, GABA_B conductances; axonal delays from 3D distance | 3 | Plus short-term plasticity (Tsodyks–Markram) and per-pathway kinetics | 3–4 |
| Learning | STDP into a 1 s eligibility trace, committed by dopamine; RPE critic (`wip`); no weight dependence yet; discrimination at chance | 2 | Weight-dependent three-factor STDP, homeostatic scaling, consolidation during sleep | 3 |
| Dopamine | A global scalar; phasic part = RPE; decay in biological time | 2 | VTA/SNc populations computing RPE, with spatially targeted release | 3 |
| Emotions (fear, fatigue, curiosity) | Global scalars that shift membrane bias or noise; tested for direction only | 1 | Amygdala, LC-NE and serotonergic circuits with emergent state changes | 2–3 |
| Consciousness | Not modelled; no claims made | 0 | Out of scope. At most, measurable correlates such as thalamocortical integration and global ignition | 0–1 |
| Time | Real time: 1 s wall = 1 s biology, dt = 0.1 ms | 3 | Same, at full scale (needs lazy eligibility, then possibly event-driven simulation) | 3 |
