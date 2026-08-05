# interactive-planning-lab

Visual demos of interactive decision-making (MCTS) and trajectory optimization (iLQR) for autonomous driving.

This repo is a **showcase**, not a third copy of the algorithms — short GIFs, short tech cards, links to source.

---

## InteractMCTS — lane-change vertical slice

**Semantic MCTS for lane change:** sticky gap selection → variable-duration actions (`WAIT` / `ALIGN` / `CommitINSERT` / `ContinueCommit` / `AbortNow`) → hard safety gate (collision / gap / TTC) → closed-loop S8.

Default setup: dual-lane road, left lane change, Frenet planning on a reference path, Cartesian display. Output: ~8 s coarse trajectory (`dt = 0.5 s`) + semantic schedule + single sticky `gap_id`.

### Casebook definitions (C1–C10)

| Case | Name | Setup / intent | Expected behavior |
|------|------|----------------|-------------------|
| **C1** | Comfortable gap | Wide Follow–Lead spacing, mild relative speed | Should reach **CommitINSERT** and merge |
| **C2** | Tight gap | Gap just above threshold; scoring / gate boundary | Borderline insert or careful align; easy to flicker |
| **C3** | Follow always keep | Open-loop constant-speed follower baseline | Legal insert when geometry allows |
| **C4** | Follow blocks from start | Follower accelerates and closes the gap | **Defer**: WAIT / ALIGN, do not force insert |
| **C5** | Gap disappears mid-run | Sticky Follow leaves the target lane / gap invalid | Must **reselect gap** or defer |
| **C6** | Two near-tied gaps | Top-2 scores almost equal | Sticky gap should **avoid chattering** |
| **C7** | Block mid-INSERT | Enter CROSSING, then Follow switches keep→block | **AbortNow**, return to ego lane if feasible |
| **C8** | Curved reference (R≈90 m) | Cubic-spline Frenet ↔ Cartesian; never assume \(x \equiv s\) | Same decision logic on a curve |
| **C9** | No same-lane lead · speed up into gap | Free ego lane; gap center ahead / faster → ALIGN accelerate then insert | Auto speed-up then CommitINSERT |
| **C10** | No same-lane lead · slow into gap | Free ego lane; gap center behind / slower → ALIGN decelerate then insert | Auto slow-down then CommitINSERT |

### Closed-loop demos

#### C1 — comfortable gap (insert)

![C1](assets/InteractMCTS_lane_change/C1_closed_loop.gif)

#### C2 — tight gap

![C2](assets/InteractMCTS_lane_change/C2_closed_loop.gif)

#### C3 — follow always keep

![C3](assets/InteractMCTS_lane_change/C3_closed_loop.gif)

#### C4 — follower blocks from the start (defer)

![C4](assets/InteractMCTS_lane_change/C4_closed_loop.gif)

#### C5 — gap disappears mid-run

![C5](assets/InteractMCTS_lane_change/C5_closed_loop.gif)

#### C6 — two near-tied gaps (sticky)

![C6](assets/InteractMCTS_lane_change/C6_closed_loop.gif)

#### C7 — block mid-INSERT (abort)

![C7](assets/InteractMCTS_lane_change/C7_closed_loop.gif)

#### C8 — curved reference

![C8](assets/InteractMCTS_lane_change/C8_closed_loop.gif)

#### C9 — no same-lane lead, accelerate into gap

![C9](assets/InteractMCTS_lane_change/C9_closed_loop.gif)

#### C10 — no same-lane lead, decelerate into gap

![C10](assets/InteractMCTS_lane_change/C10_closed_loop.gif)

### Tech card

| | |
|---|---|
| **Problem** | Lane-change gap acceptance / insert / abort under interactive followers |
| **Method** | Sticky gap + semantic variable-duration MCTS + hard gate; closed-loop S8 |
| **Demo** | Casebook C1–C10 closed-loop GIFs |
| **Output** | Semantic schedule + coarse ego trajectory + tags |
| **Limitations** | Python research demo; simplified opponent models; not production / Orin / real perception |
| **Source** | Private research repo `InteractMCTS` — demos published here |

> Unprotected left-turn plugin for InteractMCTS is **not** shown here yet. PairMCTS below covers intersection Level-1 demos separately.

---

## PairMCTS

Intersection **Level-1 game + MCTS** for unprotected left turns: ego negotiates yield / gap / continue against oncoming traffic (and sometimes pedestrians), then emits a coarse trajectory + semantic tags for downstream optimization.

Closed-loop demos below use **receding-horizon replanning** (re-run MCTS each sim step, ~20 s). In these sims, **obstacle vehicles drive at constant speed and do not play a game** — only ego runs PairMCTS; opponents are open-loop constant-velocity agents.

### Scenario 4 — unprotected left vs one oncoming straight

![Scenario 4](assets/pairmcts/scenario_4_sim_20s_replan.gif)

### Scenario 1 — unprotected left vs crossing left-turner

![Scenario 1](assets/pairmcts/scenario_1_sim_20s_replan.gif)

### Scenario 3 — unprotected left vs two oncoming straights (gap)

![Scenario 3](assets/pairmcts/scenario_3_sim_20s_replan.gif)

### Scenario 2 — three left-turners + pedestrian

![Scenario 2](assets/pairmcts/scenario_2_sim_20s_replan.gif)

### Tech card

| | |
|---|---|
| **Problem** | Unprotected left at intersection; multi-agent interaction |
| **Method** | Pairwise Level-1 games inside MCTS → Boltzmann cost → rollout → backprop; closed-loop replan |
| **Demo** | 20 s receding-horizon sims (scenarios 1–4) |
| **Output** | Semantic tags (yield / overtake / …) + coarse ego trajectory |
| **Limitations** | Python research demo; opponents are CV (no game); not production / Orin / real perception |
| **Source** | PairMCTS (formerly `ks_decision`) — public link TBD |

---

## Coming next

| Project | Status |
|---------|--------|
| **myiLQR** — DDP / iLQR trajectory optimization | Planned |
| **InteractMCTS unprotected left turn** | Later (lane-change slice is shown above) |

---

## Layout

```text
interactive-planning-lab/
├── README.md
└── assets/
    ├── InteractMCTS_lane_change/
    │   ├── C1_closed_loop.gif … C10_closed_loop.gif
    └── pairmcts/
        ├── scenario_1_sim_20s_replan.gif
        ├── scenario_2_sim_20s_replan.gif
        ├── scenario_3_sim_20s_replan.gif
        └── scenario_4_sim_20s_replan.gif
```
