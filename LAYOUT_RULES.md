# Layout Builder Rules

## Purpose

This file is the working rulebook for creating and updating Bhome layouts.
It should be updated every time new rules, terms, actions, conditions, or exceptions are provided.

When starting work on this project, this file should be read first.

## Source Status

Current source used to initialize this document:

- `8 Strong Hives.json`
- `bhome_config.json`

This document currently contains:

- Confirmed structure observed directly in the JSON example
- Inferred logic based on the example
- `TBD` items that need user confirmation

## Update Protocol

- New rules must be added here immediately.
- This file must be read before each work session on this project.
- If a new rule conflicts with the current logic, stop and ask for clarification before changing the rulebook.
- If a rule is inferred from data and not explicitly confirmed, mark it as `Inferred`.
- If a rule is explicitly given by the user, mark it as `Confirmed`.

## Core Concepts

### Bhome

- Definition: A Bhome is divided into 2 stations: `a` and `b`. `Confirmed`.
- Role in system: A Bhome contains the frame layout across both stations. `Confirmed`.
- Confirmed structure: a Bhome contains a `frames` array.
- Confirmed example pattern: the sample file appears to describe one Bhome with 120 total frame slots.

### Stations

- A Bhome has 2 stations: `a` and `b`. `Confirmed`.
- Hive naming by station:
  - Station `a`: `A1`, `A2`, `A3`, `A4`
  - Station `b`: `B1`, `B2`, `B3`, `B4`
- Each station has positions for frames. `Confirmed`.
- The frame layout logic is determined mainly by the `x` position. `Confirmed`.
- The frame `x` position represents the center of the frame. `Confirmed`.
- `y` and `z` exist in the station definition, but for frame layout logic the important coordinate is `x`. `Confirmed`.

Confirmed station ranges:

- Station `a`
  - `start`: `x=150`, `y=20`, `z=280`
  - `end`: `x=2770`, `y=20`, `z=280`
- Station `b`
  - `start`: `x=130`, `y=540`, `z=280`
  - `end`: `x=2725`, `y=540`, `z=280`

### Hive

- Definition: In each station, the Bhome contains Hives. `Confirmed`.
- Role in system: A Hive is a repeating layout unit inside a station. `Confirmed`.
- Confirmed pattern per hive:
  - 1 `partition`
  - 1 `feeder`
  - 12 `comb_frame`
  - 1 `static_partition`
- Confirmed total per hive: 15 frame slots
- Inferred from sample: the sample appears to contain 8 hives total and 120 slots total
- Hive boundaries for evaluation should be identified from structure, not only from fixed slot counts. `Confirmed`
- Operationally, a hive should be detected from `partition` through `static_partition`. `Confirmed`
- A hive can also be evaluated at layout level as `open` or `closed` based on partition spacing. `Confirmed`

### Frame

- A frame is one slot entry inside the Bhome `frames` array.
- Observed frame fields:
  - `slot`
  - `rfid`
  - `type`
  - `place`
  - `side_a`
  - `side_b`
  - `slotId`
  - `status`
  - `id`
  - `last_updated`
  - `previous_side_a`
  - `previous_side_b`
  - `previous_update_date`
  - `bhome_id`

## Frame Types

Observed in the sample JSON:

- `partition`
- `feeder`
- `comb_frame`
- `static_partition`
- `queen_excluder`

Observed counts in `8 Strong Hives.json`:

- `partition`: 8
- `feeder`: 8
- `comb_frame`: 96
- `static_partition`: 8

### Frame Type Definitions

#### `static_partition`

- A `static_partition` is a frame in a permanent position. `Confirmed`
- It cannot be moved. `Confirmed`
- It is part of the hive boundary. `Confirmed`

#### `feeder`

- A `feeder` is a frame that can receive syrup for the bees in the hive to feed. `Confirmed`
- Historical rule: the feeder used to be placed near the dynamic partition. `Confirmed`
- Current rule: the feeder is now placed near the `static_partition`. `Confirmed`
- The feeder can be placed anywhere when needed, but the preferred position in new layouts is near the `static_partition`. `Confirmed`
- In the new layout logic, the feeder should be positioned in the last movable slot before the `static_partition`. `Confirmed`

#### `partition`

- The `partition` is the dynamic partition. `Confirmed`
- The dynamic partition is meant to close the hive. `Confirmed`
- It represents the end of the hive. `Confirmed`
- Because the dynamic partition closes the hive, it is movable logic-wise, unlike the `static_partition`. `Inferred from user wording`

#### `comb_frame`

- A `comb_frame` is the main bee-content frame inside the hive. `Inferred`
- A hive usually contains between 2 and 15 `comb_frame` entries. `Confirmed`
- Values outside that range are special cases. `Confirmed`

#### `queen_excluder`

- `queen_excluder` is a frame type used to keep the queen in a bounded hive area. `Confirmed`
- The queen should stay between the `queen_excluder` and the `static_partition`. `Confirmed`
- A QE hive introduces one additional frame between the dynamic `partition` and the first `comb_frame`. `Confirmed`
- In split layouts, the `queen_excluder` should be placed near the dynamic `partition`. `Confirmed`

## Frame Content

### Shared Frame Metadata

- `slot`: slot index in the full Bhome layout
- `slotId`: appears to duplicate `slot`
- `rfid`: physical or logical identifier of the frame
- `type`: frame type
- `status`: current frame state
- `id`: unique frame record ID
- `bhome_id`: owning Bhome ID
- `last_updated`: latest update timestamp
- `previous_update_date`: previous update timestamp
- `place.station`: observed value `a` in the sample; exact meaning `TBD`
- `place.station`: the station where the frame is located, such as `a` or `b`
- `place.position.x`: x-position of the frame within the layout
- `place.position.x`: the key coordinate used for frame layout logic

### Side Content

Applicable mainly to `comb_frame` entries in the sample.

Each frame may contain:

- `side_a`
- `side_b`
- `previous_side_a`
- `previous_side_b`

These objects describe the observed biological / material content of each side of a frame.

### Observed Side Content Fields

- `bee`
- `bee-queen`
- `beeCount`
- `cell`
- `cell-drawnout`
- `cell-foundation`
- `cell-worker`
- `cell-worker-caped_honey`
- `cell-worker-empty`
- `cell-worker-honey`
- `cell-worker-larva_0_to_1`
- `cell-worker-larva_2_to_3`
- `cell-worker-larva_4_to_5`
- `cell-worker-larva_6_to_7`
- `cell-worker-larva_8_to_21`
- `cell-worker-pollen`
- `fileName`
- `queen_cell-1_to_16`
- `scanTimestamp`

### Side Content Meaning

The meanings below are based on user-confirmed scan logic unless marked otherwise.

### Scan Types

- There are 2 scan types used for comb frames. `Confirmed`
- `BeeCount`: take a picture of the frame with bees on it and use vision algorithms to count bees. `Confirmed`
- `Scan`: shake the bees off the frame, then inspect the visible cells and evaluate their content. `Confirmed`

### Confirmed Side Content Meaning

- `bee`: number of bees identified on the visible frame side. `Confirmed`
- `bee-queen`: number of queen bees identified. `Confirmed`
- `beeCount`: computer vision bee count object. `Confirmed`
- `beeCount.fileName`: link to the bee-count image. `Confirmed`
- `beeCount.beesPercent`: percent of bees in the visible frame. `Confirmed`
- `beeCount.scanTimestamp`: timestamp of the bee-count scan. `Confirmed`
- `cell`: visible cells. `Confirmed`
- `cell-worker`: worker cells identified on the frame side. `Confirmed`
- `cell-drawnout`: drawn-out cells. `Confirmed`
- `cell-foundation`: foundation cells. `Confirmed`
- `cell-worker-empty`: amount of worker cells that are empty. `Confirmed`
- `cell-worker-honey`: amount of worker cells with honey. `Confirmed`
- `cell-worker-caped_honey`: amount of worker cells with capped honey. `Confirmed`
- `cell-worker-pollen`: amount of worker cells with pollen. `Confirmed`
- `queen_cell-1_to_16`: special cells for queen identification; barely used now. `Confirmed`
- `fileName`: link to the scanned-cells image. `Confirmed`
- `scanTimestamp`: timestamp of the cell scan. `Confirmed`

### Brood / Larva Meaning

- Worker brood starts as eggs. `Confirmed`
- Then it becomes larva in an open cell. `Confirmed`
- Then the cell is capped with the larva. `Confirmed`
- `cell-worker-larva_0_to_1`: eggs. `Confirmed`
- `cell-worker-larva_8_to_21`: capped brood. `Confirmed`
- Open brood is represented by the intermediate larva buckets. `Confirmed`
- The JSON stores intermediate larva buckets as:
  - `cell-worker-larva_2_to_3`
  - `cell-worker-larva_4_to_5`
  - `cell-worker-larva_6_to_7`
- Current interpreted grouping:
  - `cell-worker-larva_2_to_3` + `cell-worker-larva_4_to_5` + `cell-worker-larva_6_to_7` = open brood
- User shorthand may refer to this combined range as `3_to_7`, even though the JSON stores it as three separate buckets. `Confirmed`

### Content Interpretation Rules

- `cell-worker...` fields represent worker cells and their identified content. `Confirmed`
- The numeric value of each `cell-worker...` field is the amount of cells identified for that content type. `Confirmed`
- Example: `cell-worker-honey` is the amount of worker cells with honey. `Confirmed`

## Layout Rules

### Confirmed From Sample

- A layout is represented as a `frames` array.
- Each frame occupies a slot.
- Slot order matters.
- Each frame has a `type`.
- `comb_frame` entries contain side-level content data.
- Non-comb frames such as `partition` and `feeder` may have empty side objects.

### Inferred From Sample

- The sample layout is organized into 8 repeating hive groups.

### Confirmed Layout Rules

- A hive is bounded by partitions.
- The `static_partition` is fixed and cannot be moved.
- The dynamic partition is the `partition` frame type.
- The dynamic partition closes the hive.
- A QE hive places `queen_excluder` immediately after the dynamic `partition`. `Confirmed`
- In a QE hive, the queen should remain in the area between the `queen_excluder` and the `static_partition`. `Confirmed`
- The feeder placement rule changed over time:
  - historical placement: near the dynamic partition
  - current placement: near the `static_partition`
- A hive can have between 2 and 15 `comb_frame` frames in normal operation.
- Values outside 2 to 15 `comb_frame` frames are exceptional cases.
- Comb frame spacing is controlled by Bhome configuration.

## Auto Condense Rules

### Status Notes

- This section reflects the latest clarified Auto Condense behavior. `Confirmed unless marked otherwise`
- Current config naming still uses `auto_condence_hive_policy`. The typo is preserved in config and may exist in implementation. `Observed`

### Action Identity

- Auto Condense executes with:
  - `command = condenseHiveBundle`
  - `initiator = autoCondenseHive`

### Inputs

- Required minimum amount of hives in the Bhome to start condensing. `Confirmed`
- Expected minimum amount of frames that must remain in the hive after condense. `Confirmed`
- Current related Bhome config key:
  - `activity_manager.auto_condence_hive_policy.min_populated_hive_size` `Observed`

### Derived Post-Condense Minimum

- Auto Condense must not reduce a hive below:
  - `min_populated_hive_size + userDefinedFramesMinimumFrames` `Confirmed`
- Example:
  - hive size `12`
  - `min_populated_hive_size = 4`
  - `userDefinedFramesMinimumFrames = 4`
  - minimum frames after condense = `8`
  - maximum removable frames = `4` `Confirmed`
- If more than 4 frames are frame-eligible in that example, only 4 may be removed. `Confirmed`

### Bhome-Level Gate Conditions

- Bhome must have enough hives to allow condense:
  - `total_hives_in_bhome >= required_minimum_amount_of_hives` `Confirmed`
- Bhome must not be in:
  - `super`
  - `visit`
  - `transportation` mode `Confirmed`
- Bhome status must be receptive. `Confirmed from ticket + user wording`
- Auto Condense must be enabled for the local day of week. `Observed in config / ticket`
- Current local Bhome time must be inside the Auto Condense window. `Observed in config / ticket`
- Condense must be inside the allowed condense cycle. `Observed in config / ticket`
- Voltage, temperature, operational-state, and sequence constraints are still part of the feature rules. Exact final wording is partially pending but they remain active requirements. `Partially confirmed`

### Hive Definitions

- `Eligible` hive:
  - valid for condense
  - not locked
  - not condensed since the last condense cycle start
  - has recent valid scan + bee count data `Confirmed`
- `Potential` hive:
  - valid for condense
  - not locked
  - not condensed since the last condense cycle start
  - missing fresh scan + bee count data, but data can still be collected `Confirmed`

### Data Validity

- Valid bee count data:
  - within `beecount_data_validity_days` `Observed in config`
- Valid scan data:
  - within `scan_data_validity_days` `Observed in config`
- Current defaults in `bhome_config.json` for Auto Condense:
  - `beecount_data_validity_days = 3`
  - `scan_data_validity_days = 3` `Observed`

### Auto Condense Flow

- Validate Bhome-level condense conditions first. `Confirmed`
- Separate valid hives into:
  - `Eligible`
  - `Potential` `Confirmed`
- Scan Potential hives first. `Confirmed`
- Start condensing only after the scan step. `Confirmed`
- Therefore, all hives to be condensed must have recent scans before condensing. `Confirmed`
- Condense should process all eligible hives in the cycle, not just one hive. `Confirmed`
- Eligible hives should be processed in hive-name order:
  - `A1`, `A2`, `A3`, ... `Confirmed`

### Hive Size Interpretation For Condense

- In many contexts, hive size refers to the amount of bee frames. `Confirmed`
- For Auto Condense removal capacity, we should look at the total amount of frames that remain in the hive after removal, so the hive does not drop below the allowed post-condense minimum. `Confirmed`
- This rule exists specifically to prevent over-condensing when several empty frames are removable. `Confirmed`

### Frame Eligibility For Condense

- Frame eligibility is evaluated on the full physical frame. `Confirmed`
- Both sides count. `Confirmed`
- Brood threshold:
  - brood cells `< 100` `Confirmed from current ticket wording`
- Honey threshold:
  - includes open honey + capped honey
  - total honey cells `< 800` `Confirmed`
- Pollen threshold:
  - pollen cells `< 800` `Confirmed from current ticket wording`

### Frame Selection Order

- Eligible frames are ranked by the least total occupied content. `Confirmed`
- Occupied content for selection is:
  - `brood + honey + pollen` across both sides of the physical frame `Confirmed`
- The system should start with the frame that has the lowest total `brood + honey + pollen`. `Confirmed`
- Removal stops once the hive reaches its post-condense minimum size. `Confirmed`

### Open Questions

- If some Potential hives fail to scan, should Auto Condense continue for the hives that do have fresh valid data, or stop the whole cycle? `TBD`
- Final tie-breaker when 2 frames have the exact same emptiness score is not fully confirmed.
  - likely fallback: outermost frame first `TBD`
- Final documented cross-station ordering should be confirmed if station `b` hives are condensed in the same cycle as station `a`. `TBD`

### Naming / Documentation Risks

- The following names appear to refer to overlapping concepts and should be standardized before implementation:
  - `minAmountOfHivesToProceed`
  - `min_hives_to_proceed_with_condence`
  - `min_eligble_hives_to_proceed_with_condence`
  - `userDefinedFramesMinimumFrames`
  - `user_defined_frames`
  - `extra_non_populated_frames_default`
  - `min_populated_hive_size`
  - `min_populated_frames`
- `condence` vs `condense` typos should be treated carefully because they may already exist in real config keys. `Confirmed risk`

### Bhome Config Rules

Confirmed from [`bhome_config.json`](/Users/fabio/Documents/Layout_Builder/bhome_config.json):

- `thresholds.broodFrameThreshold = 25`
- `thresholds.honeyFrameThreshold = 25`
- `thresholds.beePopulationThreshold = 15`
- `thresholds.openBroodFrameThreshold = 5`
- `thresholds.openBroodLiveScanThreshold = 50`
- `activity_manager.frame_layout_policy.enabled = true`
- `activity_manager.frame_layout_policy.minimum_space_between_comb_frames = 30`
- `activity_manager.auto_scan_policy.auto_scan_type = 2`
- The reason for the minimum spacing rule is that comb frames are approximately 26 to 28 mm wide, so their x-coordinate distance should always be greater than 30. `Confirmed from user explanation`

Implication:

- Consecutive `comb_frame` positions should maintain an x-distance greater than or equal to the configured minimum spacing rule.
- This spacing rule is intended to prevent comb frames from being placed too close to each other.

### Confirmed Hive Pattern

1. `partition`
2. `feeder`
3. `comb_frame`
4. `comb_frame`
5. `comb_frame`
6. `comb_frame`
7. `comb_frame`
8. `comb_frame`
9. `comb_frame`
10. `comb_frame`
11. `comb_frame`
12. `comb_frame`
13. `comb_frame`
14. `comb_frame`
15. `static_partition`

### QE Hive Pattern

1. `partition`
2. `queen_excluder`
3. `comb_frame`
4. `comb_frame`
5. `comb_frame`
6. `comb_frame`
7. `comb_frame`
8. `comb_frame`
9. `comb_frame`
10. `comb_frame`
11. `comb_frame`
12. `comb_frame`
13. `comb_frame`
14. `comb_frame`
15. `feeder`
16. `static_partition`

## Max Frames Per Auto Activity

### Validation Timing

- After frame layout confirmation, every auto activity that targets a hive must validate the attempted hive against the activity's max-frame rule before sending the activity to the planner. `Confirmed`
- This validation is per attempted hive, not only once per activity cycle. `Confirmed`
- If the selected hive exceeds the activity maximum, skip that hive and evaluate the next candidate hive for the same activity. `Confirmed`
- If no eligible hive remains after applying this validation, no activity is defined for that cycle. `Confirmed`

### Frame Count Metric

- Count only frames located between the dynamic `partition` and the `static_partition`. `Confirmed`
- Do not include the dynamic `partition` in the count. `Confirmed`
- Do not include the `static_partition` in the count. `Confirmed`
- Frames between the partitions include frame types such as `comb_frame`, `feeder`, and `queen_excluder` unless an activity-specific rule says otherwise. `Confirmed`

### Activity Limits

- `Auto Scan`: maximum `14` counted frames. `Confirmed`
- `Auto Bee Count`: maximum `14` counted frames. `Confirmed`
- `Auto Feeding`: feeder must exist and the hive can have a maximum of `13` `comb_frame` frames. `Confirmed`
- `Auto Thermal Scan`: no max-frame limit. `Confirmed`
- `Auto Frame Layout`: no max-frame limit. `Confirmed`
- `Auto Heat Treatment`: maximum `14` counted frames. `Confirmed`
- `Auto Compress Hives`: no max-frame limit. `Confirmed`
- `Auto Balancing (Bees)`: maximum `14` counted frames. `Confirmed`
- `Auto Super Detection`: no max-frame limit. `Confirmed`
- `Auto Split`: before split planning, the current confirmed layout must have `queen_excluder` next to the dynamic `partition`, and the hive can have a maximum of `12` counted frames including `feeder` and `queen_excluder`. `Confirmed`
- `Auto Condense`: maximum `14` counted frames. `Confirmed`
- `Auto Brood Donation`: maximum `14` counted frames. `Confirmed`
- `Auto Varroa Sentinel`: no max-frame limit. `Confirmed`
- `Auto Honey Balancing`: maximum `14` counted frames. `Confirmed`

## Action Catalog

No explicit Bhome action list has been provided yet.

`TBD`:

- What actions can be executed in a Bhome
- What actions can be executed per hive
- What actions can be executed per frame
- Whether actions depend on frame type
- Whether actions depend on frame content thresholds

Known action under definition:

- `Split` / `autoSplitHive`

## Action Rules

### Split Action

- The split action is also referred to by initiator message `autoSplitHive`. `Confirmed`
- Split decisions use values from [`bhome_config.json`](/Users/fabio/Documents/Layout_Builder/bhome_config.json), especially `activity_manager.auto_frames_split_policy`. `Confirmed`
- Split evaluation uses all frames or edge-scan-derived data according to `activity_manager.auto_scan_policy.auto_scan_type`. `Confirmed`

#### Current Split Config Values

From [`bhome_config.json`](/Users/fabio/Documents/Layout_Builder/bhome_config.json):

- `activity_manager.auto_frames_split_policy.in_hive_populated_frames = 7`
- `activity_manager.auto_frames_split_policy.scan_validity_days = 2`
- `activity_manager.auto_frames_split_policy.beecount_validity_days = 2`
- `activity_manager.auto_frames_split_policy.min_amount_brood_frames_in_hive = 5`
- `activity_manager.auto_frames_split_policy.min_amount_honey_frames_in_hive = 2`
- `activity_manager.auto_frames_split_policy.auto_frames_split_start_time = 09:00`
- `activity_manager.auto_frames_split_policy.auto_frames_split_cycle_days = 4`

#### Split Hive Selection Rules

- Choose the hive with the most brood frames from the set of hives that satisfy all split conditions. `Confirmed`
- Split-condition evaluation uses full-frame data or edge-scan inference according to `auto_scan_type`. `Confirmed`
- Split-condition evaluation is based on hive evaluation. `Confirmed`
- For splitting, use frames scanned in the last 2 days. `Confirmed`
- A candidate hive must have at least `in_hive_populated_frames`. `Confirmed`
- General split rule provided earlier: a candidate hive must have more than `min_amount_brood_frames_in_hive` OR more than `min_amount_honey_frames_in_hive`. `Confirmed`
- For the current split-test scenario, the selected hive should satisfy both brood and honey conditions. `Confirmed`
- A candidate hive must not have been split since the last `FRAMES_SPLIT_START`. `Confirmed wording from user`
- If a hive fails any of the above conditions, exclude it and evaluate the next hive. `Confirmed`

#### Split Workflow Order

- The previous workflow caused significant disorder among the bees because it performed immediate consecutive operations on the same hive. `Confirmed`
- Updated workflow:
  - First scan all hives in station `A`, in order: `A1`, `A2`, `A3`, `A4`. `Confirmed`
  - After scanning all hives in station `A`, perform split only on the hives in station `A` that meet the split requirements. `Confirmed`
  - Only after all operations in station `A` are completed may the system move to station `B`. `Confirmed`
- The same station-based workflow should then apply to station `B`. `Confirmed`
- This workflow is intended to reduce disorder among the bees by avoiding immediate repeated interventions on a single hive while scanning is still ongoing in the same station. `Confirmed`

#### Choosing Frames To Split In The Selected Hive

- Choose all frames between the `queen_excluder` and the closest brood frame on the positive direction. `Confirmed`
- Here, `closest` means nearest to the dynamic partition. `Confirmed`

#### In-Hive Reordering Rules During Split

- If the number of honey frames in the hive is greater than `min_amount_honey_frames_in_hive`, first move honey frames that are close to the dynamic partition to the negative side of the `queen_excluder` inside the same hive. `Confirmed`
- Then move honey frames that are close to the `static_partition` to the negative side of the `queen_excluder` until reaching `min_amount_honey_frames_in_hive`. `Confirmed`
- Then, if brood frames in the hive are greater than `min_amount_brood_frames_in_hive`, move brood frames to the negative side of the `queen_excluder` inside the same hive. `Confirmed`
- If `min_amount_honey_frames_in_hive` has already been reached and there is still a honey frame between the `queen_excluder` and the closest brood frame on the positive direction, move it to the negative side of the feeder / comb frame. `Confirmed`
- Else, if there is an `Empty` frame, move it to the positive side of the brood frame that is closest to the `static_partition`. `Confirmed`
- Frames between 2 brood frames must not be considered empty. `Confirmed`
- Else, if there is no scan for relevant frames above, consider a frame as `Empty` and apply the empty-frame rule. `Confirmed`

#### Split Action Open Questions

- A `populated frame` means a bee-populated frame. `Confirmed`
- Population is always about bees. `Confirmed`
- Hive-level brood / honey eligibility is obtained through hive evaluation. `Confirmed`
- What field stores whether a hive was split since the last split start?
- What exactly is meant by `negative side` and `positive side` in terms of x-direction for each station?
- What is the exact destination intended by `negative side of the feeder/comb frame`?

## Conditions

### Hive Strength Conditions

- Hive strength depends on evaluation of the frames content in the last scan. `Confirmed`

#### Bee Frames

- A bee frame is evaluated at the frame level using both sides of the frame. `Confirmed`
- The frame bee percentage is the average of `side_a.beeCount.beesPercent` and `side_b.beeCount.beesPercent`. `Confirmed`
- A bee frame is a frame where that averaged bee percentage is greater than or equal to `beePopulationThreshold`. `Confirmed`
- The threshold source is [`bhome_config.json`](/Users/fabio/Documents/Layout_Builder/bhome_config.json) -> `thresholds.beePopulationThreshold`. `Confirmed`
- Current configured value: `15`. `Confirmed from config`

#### Hive Strength By Number Of Bee Frames

- `1-4` bee frames -> `Weak hive`. `Confirmed`
- `5-7` bee frames -> `Medium hive`. `Confirmed`
- `8+` bee frames -> `Strong hive`. `Confirmed`

#### Empty Hive vs Dead Hive

- An `Empty hive` is a hive structure with no `comb_frame` entries to evaluate. `Confirmed`
- Temporary working definition: a `Dead hive` is a hive with no bees. `Confirmed`
- Historical definition: a `Dead hive` is a hive that still contains `comb_frame` entries, but evaluates to `0` bee frames and `0` brood frames. `Confirmed`
- Operationally:
  - empty = no comb frames
  - dead = no bees under the current temporary rule

### Brood Frame Conditions

- A brood frame is evaluated at the frame level using both sides of the frame. `Confirmed`
- The frame brood value is evaluated as a brood percentage for the full frame. `Confirmed`
- The brood percentage uses both sides of the frame together. `Confirmed`
- The brood percentage denominator is the total `cell-drawnout` from both sides of the frame. `Confirmed`
- A brood frame is a frame whose brood percentage is greater than or equal to `broodFrameThreshold`. `Confirmed`
- The threshold source is [`bhome_config.json`](/Users/fabio/Documents/Layout_Builder/bhome_config.json) -> `thresholds.broodFrameThreshold`. `Confirmed`
- Current configured value: `25`. `Confirmed from config`
- Brood status is determined from scan data. `Confirmed`

### Scan Strategy Rules

#### Historical Strategy

- Previously, the system scanned all frames in a hive. `Confirmed`
- From the full-hive scan, the system determined whether each frame was a:
  - bee frame
  - brood frame
  - honey frame
- Full-hive data was then used to evaluate hive strength based on a factor. `Confirmed`

#### Current Strategy

- The system no longer scans the full hive by default because it is too invasive and disrupts the bees too much. `Confirmed`
- The new strategy is `Edge scan`. `Confirmed`
- The current config value for this is `activity_manager.auto_scan_policy.auto_scan_type = 2`. `Confirmed`

#### Edge Scan Logic

- Start scanning from the partition side of the hive. `Confirmed`
- There are 2 scan types in edge scan: `Bee Count` and `Frame Scan`. `Confirmed`
- `Bee Count` observes the bee percentage on each scanned frame. `Confirmed`
- `Frame Scan` observes the content of the frame cells. `Confirmed`
- On each edge, the system always scans the first 2 frames before making any continue / stop decision. `Confirmed`
- After those first 2 frames on an edge, the system decides whether to continue scanning deeper based on the content of the scanned frames and the scan type. `Confirmed`
- Under `Frame Scan`, scan frames from that side until a brood frame is found. `Confirmed`
- Under `Bee Count`, scan frames from that side until the bee-frame condition for that scanned region is found. `Confirmed`
- The reasoning is that the weakest brood frames are on the edges of the hive. `Confirmed`
- If a frame on that edge has enough brood to qualify as a brood frame, then the next frame inward is also assumed to be a brood frame. `Confirmed`
- After finding a brood frame from the partition side, start scanning from the other side, the `static_partition` side. `Confirmed`
- On the second edge, again scan the first 2 frames before making any continue / stop decision. `Confirmed`
- Then scan one by one from the `static_partition` side until a brood frame is found there too. `Confirmed`
- Once brood frames are found from both sides, all frames in the middle are considered brood frames as well. `Confirmed`
- Under edge scan, once the scanned bee-frame region is identified, all frames in that scanned region are treated as bee frames. `Confirmed`
- Example: if the scanned bee-frame region runs from slot `23` to slot `28`, then all frames from `23` through `28` are bee frames. `Confirmed`
- For `Frame Scan`, once the second edge brood frame has been detected, the system may continue scanning further if needed for queen evaluation open-brood aggregation. `Confirmed`

### Inference Rules

- Under edge scan, not every brood frame must be directly scanned. `Confirmed`
- Middle frames between the first qualifying brood frame on the partition side and the first qualifying brood frame on the static-partition side are inferred to be brood frames. `Confirmed`
- This means brood-frame evaluation can be partly measured and partly inferred. `Confirmed`
- The same edge-based inference logic also applies to bee frames and honey frames. `Confirmed`
- For bee frames, the operational result under edge scan follows the scanned region, not only the individually measured frame averages. `Confirmed`
- For brood frames, the operational result under edge scan follows the contiguous brood region between the first qualifying brood frame from each side, not only the individually measured frame percentages. `Confirmed`

### Open Hive Condition

- Layout-level meaning of `open hive`: the comb area plus partitions are not compressed into a tight, sealed block. `Confirmed`
- A hive is `open` if:
  `X[static_partition] - X[dynamic_partition] > 37.5 * number_of_frames_in_hive + 0.5 * dynamic_partition_width` `Confirmed`
- Otherwise, the hive is `closed`. `Confirmed`
- For this rule, `number_of_frames_in_hive` should be interpreted as the frames from the dynamic `partition` through the last frame before the `static_partition`, excluding the `static_partition` itself. `Inferred from layout match`
- Observed reference width from project data: `partition_width = 36.6`, so `0.5 * dynamic_partition_width = 18.3`. `Inferred from extracted layout data`

### Queen Evaluation

- `QL` means `QueenLess`. `Confirmed`
- `QR` means the hive is `queen-right`. `Confirmed`
- Normal case: hive evaluation exists and there are no skipped frames. `Confirmed`
- Aggregate the number of open brood cells, including eggs, from all current scanned frames in the hive. `Confirmed`
- Historical issue: queen evaluation previously counted open brood only inside the brood-frame area. `Confirmed`
- Current correct behavior: queen evaluation must count open brood anywhere in the hive's current scanned frames, not only inside the brood-frame area. `Confirmed`
- Under `Frame Scan`, once the second edge brood frame has been detected, continue scanning until the total open brood + eggs in the whole hive is greater than the open brood threshold. `Confirmed`
- Practical consequence for test layouts: under `Frame Scan`, a hive can evaluate with aggregated open brood `<=` the threshold only if all relevant frames in the hive were scanned and the hive total still did not exceed the threshold. `Confirmed`
- If aggregated open brood `>` the open brood threshold, then `QR = True`. `Confirmed`
- If aggregated open brood `<=` the open brood threshold, evaluate `LBB_AGG_RAW`. `Confirmed`
- `LBB_AGG_RAW` is the average recognized-cell percentage across all scanned brood frames. `Confirmed`
- For this fallback, use `LBB_AGG_RAW.cell / total frame cells` per brood frame, then average those percentages. `Confirmed`
- In current project data, `LBB_AGG_RAW.cell` is usually the frame-side `cell` count, commonly `3280`. `Confirmed`
- New implementation detail under validation: `LBB_AGG_RAW` may be computed only from recently scanned frames rather than from all inferred brood-region frames. `Confirmed from current developer behavior`
- If `LBB_AGG_RAW > 70%`, then `QR = False`. `Confirmed`
- Else, `QR = No Data`. `Confirmed`
- The comparison operators above are exact and must remain `>` and `<=` as written. `Confirmed`
- Validation from `Test - Queen Evaluation.json`: after the latest fix, the current observed results are `A1`, `A2`, `B1`, `B4` => `QR`; `A3`, `B2` => `QL`; `A4`, `B3` => `No Data`. `Confirmed`
- This validation indicates that queen evaluation now correctly finds open brood on inward second-side scanned frames, including the `B1` and `B4` edge-scan regression cases. `Confirmed`
- Current project threshold references:
  - `thresholds.openBroodFrameThreshold = 5`
  - `thresholds.openBroodLiveScanThreshold = 50`
- The exact mapping from config threshold name to the queen-evaluation open-brood threshold is not yet fully confirmed. `Inferred`

### Pending Conditions

- Honey frame condition definition
- Open brood frame condition definition
- Queen-related conditions
- Honey / pollen conditions
- Empty / foundation conditions
- Split / merge conditions
- Feed conditions

## Validation / Conflict Handling

- Any new rule that changes the meaning of Bhome, Hive, Frame, Frame Type, Content, Actions, Layout Rules, Action Rules, or Conditions must be recorded here.
- Any new rule that conflicts with the current inferred hive pattern must be confirmed before replacing it.
- Any new rule that conflicts with the confirmed station structure or confirmed hive pattern must be confirmed before replacing it.
- Any new rule that conflicts with the current feeder-placement rule must be confirmed before replacing it.
- Any new rule that places normal comb-frame counts outside the 2 to 15 range must be treated as a special case unless explicitly confirmed otherwise.
- Explicit user rules override inferred rules.

## Open Questions

These are the next missing pieces needed to turn this into a full rulebook:

1. What is the exact definition of a Bhome?
2. How many hives exist per station, and is that fixed or variable?
3. How should hive boundaries be computed in practice when feeder placement can vary historically?
4. What are the exact meanings of the comb-frame side content fields?
5. What exact brood metric is compared against `broodFrameThreshold`?
6. What exact metric defines a honey frame?
7. Does edge-scan inference apply only to brood frames, or also to bee frames / honey frames?
8. What actions can be executed in the Bhome?
9. What are the conditions and thresholds used to decide actions?
