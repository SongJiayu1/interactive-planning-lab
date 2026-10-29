# InteractMCTS fresh simulation recordings

Run date: 2026-10-29 (Asia/Shanghai). All 18 existing cases were rerun using the decision evidence dashboard. 20s simulation; 0.5s replanning; ULT 24 iterations/cycle, lane change 20. All 50 function regression tests passed. All 18 GIFs were fully decoded (41 frames each, 1600×900); hashes are in manifest.json.

The figures show current-cycle search snapshots, actual action masks and rejection reasons, root visits/mean total cost, reconstructed costs, current plan and executed history. These are not iteration replays. Offline full-run warnings are distinct from current-cycle planner evidence.

## Read the results correctly

- Final WAIT after a lane change may mean target-lane HOLD; it is not a failure to merge.
- C3/C10 reached the target lane but subsequently overlap with scripted followers (13/9 samples at 0.1s). They are diagnostics, not safe successes.
- L8 passes a full-occupancy mid-gap test with 0.4s margins under the prototype interval model. L5 selects the mid-gap but does not thread it.
- L4/L5 have late scripted-actor overlaps at finite reference endpoints; C4 has non-ego scripted overlaps.
- C7 uses an injected blocking hook and is an abort demonstration.
- Perfect tracking, simplified prediction/VRU and sampled OBB checks do not establish continuous safety, comfort or superiority over Europa.

## Suggested viewing order

C9 (ALIGN/commit), L8 (time-window reasoning), C7 (abort). C3/C10 and L5 are useful failure/limit explanations.

## All cases

- **L1**: cleared=True; whole_gap_threaded=False; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L1_closed_loop.gif) · [audit](audit/L1.json)
- **L2**: cleared=True; whole_gap_threaded=False; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L2_closed_loop.gif) · [audit](audit/L2.json)
- **L3**: cleared=True; whole_gap_threaded=False; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L3_closed_loop.gif) · [audit](audit/L3.json)
- **L4**: cleared=True; whole_gap_threaded=False; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L4_closed_loop.gif) · [audit](audit/L4.json)
- **L5**: cleared=True; whole_gap_threaded=False; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L5_closed_loop.gif) · [audit](audit/L5.json)
- **L6**: cleared=True; whole_gap_threaded=False; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L6_closed_loop.gif) · [audit](audit/L6.json)
- **L7**: cleared=True; whole_gap_threaded=False; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L7_closed_loop.gif) · [audit](audit/L7.json)
- **L8**: cleared=True; whole_gap_threaded=True; ego-overlap samples=0. [GIF](../InteractMCTS_ult/L8_closed_loop.gif) · [audit](audit/L8.json)
- **C1**: target_reached=True; abort=False; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C1_closed_loop.gif) · [audit](audit/C1.json)
- **C2**: target_reached=True; abort=False; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C2_closed_loop.gif) · [audit](audit/C2.json)
- **C3**: target_reached=True; abort=False; ego-overlap samples=13. [GIF](../InteractMCTS_lane_change/C3_closed_loop.gif) · [audit](audit/C3.json)
- **C4**: target_reached=True; abort=False; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C4_closed_loop.gif) · [audit](audit/C4.json)
- **C5**: target_reached=True; abort=False; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C5_closed_loop.gif) · [audit](audit/C5.json)
- **C6**: target_reached=True; abort=False; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C6_closed_loop.gif) · [audit](audit/C6.json)
- **C7**: target_reached=False; abort=True; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C7_closed_loop.gif) · [audit](audit/C7.json)
- **C8**: target_reached=True; abort=False; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C8_closed_loop.gif) · [audit](audit/C8.json)
- **C9**: target_reached=True; abort=False; ego-overlap samples=0. [GIF](../InteractMCTS_lane_change/C9_closed_loop.gif) · [audit](audit/C9.json)
- **C10**: target_reached=True; abort=False; ego-overlap samples=9. [GIF](../InteractMCTS_lane_change/C10_closed_loop.gif) · [audit](audit/C10.json)

Full per-cycle data remains in the source checkout at python/results/decision_review/20261029_publish. This repository contains presentation media and measured metadata, not a solver copy.
