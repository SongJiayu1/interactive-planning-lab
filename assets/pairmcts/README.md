# PairMCTS showcase assets

These are presentation exports, not a solver copy. Run regeneration commands in the PairMCTS source checkout.

Reproducible commands:

```bash
cd PairMCTS/src
MPLBACKEND=Agg python3 tests/run_tests.py
MPLBACKEND=Agg python3 demos/export_showcase.py --clean --seed 0 --iterations 20 --duration 20
```

Each scenario has a closed-loop + current-tree GIF and a separate single-cycle search GIF.
Green tree edges are the raw search choice; safety repair may replace that trajectory.
N = visits; Q = mean cumulative cost (lower is better); node Δt is relative to the current plan start.

## Measured scene assessment

### S1 — Crossing left-turner
Shows a real yield/overtake branch and UCB visit allocation; synthetic equal-priority encounter.

Initial tree: 3 nodes, 1 branching nodes, depth 1.
Executed ego overlaps on 0.1s interpolated samples: 0; exit observed: True.
Safety statuses: {'fallback_brake': 3, 'validated': 38}.
Other scripted actors overlap count: 0; interior=0, boundary-band=0.

![S1 closed loop](scenario_1_sim_20s_replan.gif)
![S1 search](scenario_1_search.gif)

### S2 — Multi-actor stress test
Shows sequential pair expansion and full-plan safety overrides; diagonal point-mass pedestrian is not a zebra-crossing model.

Initial tree: 5 nodes, 1 branching nodes, depth 3.
Executed ego overlaps on 0.1s interpolated samples: 0; exit observed: True.
Safety statuses: {'fallback_brake': 7, 'validated': 34}.
Other scripted actors overlap count: 26; interior=26, boundary-band=0.

![S2 closed loop](scenario_2_sim_20s_replan.gif)
![S2 search](scenario_2_search.gif)

### S3 — Two oncoming vehicles
Tests a gap opportunity; threading must be measured, not inferred from the scene name or raw tags.

Initial tree: 3 nodes, 0 branching nodes, depth 2.
Executed ego overlaps on 0.1s interpolated samples: 0; exit observed: True.
Safety statuses: {'validated': 28, 'fallback_brake': 13}.
Other scripted actors overlap count: 57; interior=0, boundary-band=57.
Gap threading observed under the declared 3m disk criterion: False.
Late overlap of the two CV actors occurs at the cropped reference endpoint, not as a negotiated interaction.

![S3 closed loop](scenario_3_sim_20s_replan.gif)
![S3 search](scenario_3_search.gif)

### S4 — Single oncoming vehicle
Cleanest wait/restart and independent-CV validation example; initial tree is heavily pruned.

Initial tree: 2 nodes, 0 branching nodes, depth 1.
Executed ego overlaps on 0.1s interpolated samples: 0; exit observed: True.
Safety statuses: {'validated': 41}.
Other scripted actors overlap count: 0; interior=0, boundary-band=0.

![S4 closed loop](scenario_4_sim_20s_replan.gif)
![S4 search](scenario_4_search.gif)

## Limits that must remain visible
- No rule/Europa baseline, feedback-driving opponents, noise sweep or runtime benchmark is included; these GIFs do not establish comparative superiority.
- Raw quadrant-based yield/overtake tags may disagree with geometric passage order. They are shown as raw labels, not silently corrected.
- Opponents execute independent CV paths. Hypothesized game responses are not simulated ground truth.
- The diagonal pedestrian is a simplified point-mass actor; no crosswalk or human reaction model is provided.
- Reference-end zero-speed clamps remain a known dynamics artifact. Collision checks do not certify comfort.
- All overlap checks are sampled and use the demo geometry, not a continuous safety certificate.

Full per-cycle point CSVs remain in the source PairMCTS/results/audit folder. This folder contains presentation media, posters and manifest.json; no solver copy.

Fresh regression suite: 110 tests passed. All eight GIFs were fully decoded and checked against the recorded hashes.
