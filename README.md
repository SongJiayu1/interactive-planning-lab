# interactive-planning-lab

Visual demos of interactive decision-making (MCTS), geometric search (Hybrid A*), and trajectory optimization (iLQR) for autonomous driving.

This repo is a **showcase**, not a third copy of the algorithms — short GIFs, short tech cards, links to source.

## Contents

- [InteractMCTS — lane-change vertical slice](#interactmcts--lane-change-vertical-slice)
  - [Casebook (C1–C10)](#casebook-definitions-c1c10)
  - [Closed-loop demos](#closed-loop-demos)
- [InteractMCTS — unprotected left turn (ULT)](#interactmcts--unprotected-left-turn-ult)
  - [Casebook (L1–L8)](#casebook-definitions-l1l8)
  - [Closed-loop demos](#closed-loop-demos-u6--u7)
- [PairMCTS](#pairmcts)
- [Hybrid A* — perpendicular parking](#hybrid-a--perpendicular-parking)
- [Hybrid A* — ROS / RViz parking (ROSParking)](#hybrid-a--ros--rviz-parking-rosparking)
- [Coming next](#coming-next)
- [Layout](#layout)

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

---

## InteractMCTS — unprotected left turn (ULT)

**Semantic MCTS for unprotected left:** conflict-zone geometry → opposing **time** gaps + sticky gap* → `WAIT` / `CREEP` / `GO` → GO hard gate (VRU stub + keep∧block) → closed-loop U7 (replan every 0.5 s).

Default setup: PairMCTS-style 4-way cross; ego left-turn ref × opposing southbound; fixed intersection camera. Output: rolling 0.5 s decisions under perfect control over ~20 s.

### Casebook definitions (L1–L8)

| Case | Name | Setup / intent | Expected behavior |
|------|------|----------------|-------------------|
| **L1** | Comfortable time gap | Sparse / slow oncoming | **GO** and clear the conflict zone |
| **L2** | Opposing arrives first | Oncoming already occupies CZ | **Yield** (WAIT/CREEP) until clear |
| **L3** | Opposing already cleared | Oncoming past \(s_{\mathrm{exit}}\) | **GO** with little wait |
| **L4** | Narrow time gap | Mid-gap just above margin | Boundary / observational |
| **L5** | Sticky mid-gap | Two oncoming cars; mid seam scores best | Locks sticky `opp_1__opp_2` at t=0 (select gap; may not thread) |
| **L6** | Creep to observe | Same blocking opp as L2; ego farther from STOP | Prefer **CREEP**, stay out of CZ while occupied |
| **L7** | Opposing rushes gap | Comfortable gap at t=0; opp accelerates once ego commits | Delayed commit, then finish left turn |
| **L8** | Thread mid gap | Wide legal mid-gap between opp_1 and opp_2 | **GO through** the seam (opp_1 cleared, opp_2 not yet in) |

### Closed-loop demos (U6 / U7)

#### L1 — comfortable time gap (GO)

![L1](assets/InteractMCTS_ult/L1_closed_loop.gif)

#### L2 — opposing arrives first (yield)

![L2](assets/InteractMCTS_ult/L2_closed_loop.gif)

#### L3 — opposing already cleared (GO)

![L3](assets/InteractMCTS_ult/L3_closed_loop.gif)

#### L4 — narrow time gap

![L4](assets/InteractMCTS_ult/L4_closed_loop.gif)

#### L5 — sticky mid-gap selection

![L5](assets/InteractMCTS_ult/L5_closed_loop.gif)

#### L6 — creep to observe

![L6](assets/InteractMCTS_ult/L6_closed_loop.gif)

#### L7 — opposing rushes the gap

![L7](assets/InteractMCTS_ult/L7_closed_loop.gif)

#### L8 — thread between opp_1 and opp_2

![L8](assets/InteractMCTS_ult/L8_closed_loop.gif)

### Tech card

| | |
|---|---|
| **Problem** | Unprotected left vs oncoming traffic (time-gap acceptance / creep / go) |
| **Method** | Ref-line conflict disk + sticky time gap* + WAIT/CREEP/GO MCTS + keep∧block hard gate; closed-loop U7 |
| **Demo** | Casebook L1–L8 closed-loop GIFs (~20 s, `dt = 0.5 s`) |
| **Output** | Per-cycle macro + sticky gap* + executed coarse trajectory |
| **Limitations** | Python research demo; scripted / hooked opponents; VRU is a stub; not production / Orin / real perception |
| **Source** | Private research repo `InteractMCTS` — demos published here |

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

## Hybrid A* — perpendicular parking

**Kinematic search for perpendicular parking:** state is rear-axle `(x, y, θ)`; motion primitives go forward and reverse; Reeds-Shepp analytic expansion when close; SAT collision on the vehicle polygon; gradient-descent smoother with re-check.

Same family as Nav2 **SmacPlannerHybrid** and Autoware open-space parking — not a lane-level planner.

![Hybrid A* parking](assets/hybrid_a_star/parking_demo.gif)

Final path:

![Hybrid A* result](assets/hybrid_a_star/parking_result.png)

### Tech card

| | |
|---|---|
| **Problem** | Perpendicular bay parking with parked cars on both sides; goal is a pose, not a point |
| **Method** | Hybrid A* on a 0.5 m / 15° grid + Reeds-Shepp CSC + SAT OBB + curvature smoother |
| **Demo** | Search tree → raw path → drive-in GIF; static result PNG |
| **Output** | Collision-free `(x, y, θ)` path the bicycle model can follow |
| **Limitations** | Python demo; Ackermann (max steer 35°), not a ±90° forklift steer; no live lidar costmap |
| **Source** | Local repo `hybrid_a_star_avp_demo` (not copied here) |

---

## Hybrid A* — ROS / RViz parking (ROSParking)

**ROS Noetic** Hybrid A* parking stack in RViz: search a collision-free pose, then smooth. Same kinematic family as the Python bay-parking demo above; this is the Gazebo/RViz visualization recorded from [`ROSParking`](https://github.com/SongJiayu1/ROSParking).

[Watch the demo (mp4)](assets/ros_parking/hybrid_a_star_ros_noetic.mp4)

<video src="assets/ros_parking/hybrid_a_star_ros_noetic.mp4" controls width="800"></video>

### Tech card

| | |
|---|---|
| **Problem** | Parking / open-space pose planning with Hybrid A* in ROS |
| **Method** | `my_hybrid_astar`: 3D node search + Reeds-Shepp coarse path + spiral / FEM smoothers |
| **Demo** | ~36 s RViz screen recording (also in this folder as mp4) |
| **Stack** | ROS Noetic · `my_hybrid_astar` · `my_load_map` · `my_show_markers` |
| **Limitations** | Research / pre-study; not a production AVP stack |
| **Source** | [SongJiayu1/ROSParking](https://github.com/SongJiayu1/ROSParking) |

---

## Coming next

| Project | Status |
|---------|--------|
| **myiLQR** — DDP / iLQR trajectory optimization | Planned |

---

## Layout

```text
interactive-planning-lab/
├── README.md
└── assets/
    ├── InteractMCTS_lane_change/
    │   ├── C1_closed_loop.gif … C10_closed_loop.gif
    ├── InteractMCTS_ult/
    │   ├── L1_closed_loop.gif … L8_closed_loop.gif
    ├── pairmcts/
    │   ├── scenario_1_sim_20s_replan.gif
    │   ├── scenario_2_sim_20s_replan.gif
    │   ├── scenario_3_sim_20s_replan.gif
    │   └── scenario_4_sim_20s_replan.gif
    ├── hybrid_a_star/
    │   ├── parking_demo.gif
    │   └── parking_result.png
    └── ros_parking/
        └── hybrid_a_star_ros_noetic.mp4
```
