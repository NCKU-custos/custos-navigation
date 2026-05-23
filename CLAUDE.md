# custos-navigation — CLAUDE.md

SLAM/VIO state estimation and trajectory planning for the Custos drone stack.

Workspace-wide rules and state caveats live at `../CLAUDE.md` (= `/space/drone/CLAUDE.md`). If you are in a standalone clone, the one rule below is the one most likely to be violated when working in this repo.

## The one rule

**Navigation consumes perception output by topic, not by linking.** This repo depends on `custos_perception_msgs` (the message types), never on `custos-perception` (the package that emits them). The graph stays acyclic: perception publishes `custos_perception_msgs`, navigation subscribes; planning publishes `custos_navigation_msgs`, control subscribes. No package in either direction `<depend>`s on the producer.

## State of this repo

- Single package: `custos_navigation_planner` (`package.xml`, `CMakeLists.txt`, empty `src/`). No SLAM/VIO or planner implementation yet.
- This package depends on **three** `custos-interfaces` packages, the heaviest interface surface in the stack — every interface bump is more likely to ripple here than anywhere else.

## Cross-repo edges

- **Depends on:** `rclcpp`, `custos_common_msgs`, `custos_navigation_msgs`, `custos_perception_msgs`.
- **Consumes by topic:** perception outputs (detections, tracked features, fused state) from `custos-perception` or `custos-novatek-sdk-wrapper`.
- **Emits by topic:** `custos_navigation_msgs` (planned trajectories, costmaps, SLAM/VIO state) — consumed by `custos-control` (the mission manager / Pixhawk bridge).

## Repo-specific hard rules

- **No build-time dependency on `custos-perception` or `custos-control`.** Use the wire types only. The graph must stay acyclic; sharing types across these packages goes through `custos_common_msgs` or a new `custos_navigation_msgs`.
- **SLAM/VIO vs planning may eventually split into separate packages.** Today both are scoped under `custos_navigation_planner`. Splitting later is fine — but the split is a new `<package>` boundary, not a rename of the existing one (renames break consumers).
- **NOVATEK isolation (ADR 0010) applies here too.** Navigation never imports anything wrapper-private. Sensor inputs arrive as `sensor_msgs/Image` or `custos_perception_msgs`, regardless of whether they originated in the wrapper or the mock.
- **Watch the three-interface fanout.** When `custos-interfaces` bumps a major version, this package needs to verify against all three of its interfaces deps simultaneously. Expect to spend more time on interfaces-PR consumer builds than other repos do.

## Build / test cheat sheet

```bash
colcon build --packages-select custos_navigation_planner
colcon test --packages-select custos_navigation_planner

# Run planner with a recorded bag of perception inputs.
ros2 launch custos_navigation_planner planner.launch.py   # TODO: launch file unwritten
```

## Pointers specific to this repo

- Wire types: `custos-interfaces/custos_navigation_msgs/`, `custos_perception_msgs/`, `custos_common_msgs/`
- Cross-repo CI: ADR 0009 (this repo's `interfaces-consumers.yml` run will be one of the slower ones)

> TODO(post-active): write the SLAM/VIO frontend and the planner implementation.
> TODO(post-active): decide whether to split SLAM/VIO out of `custos_navigation_planner` once the planner gets large.
