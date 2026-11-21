# Layout Builder Rules

## Purpose

This file is the working rulebook for creating and updating Bhome layouts.
It should be updated every time new rules, terms, actions, conditions, or exceptions are provided.

When starting work on this project, this file should be read first.

## Source Status

Current source used to initialize this document:

- `8 Strong Hives.json`
- `bhome_100_config_2026-07-17_14-01-38.json`

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

### Bhome Config Files

- Each Bhome has its own config. Config values may differ between Bhomes. `Confirmed`
- All Bhome configs should contain the same fields, even when the values differ. `Confirmed`
- The config can change when features are updated, so every config file must be identified by the Bhome it came from and the date it was downloaded. `Confirmed`
- Naming convention in this project: `bhome_<bhome_id>_config_<YYYY-MM-DD_HH-MM-SS>.json`. `Confirmed`
- Any config value quoted in this rulebook must name its source file, because a value is only valid for that Bhome at that download date. `Confirmed`
- Current config source in this project: [`bhome_100_config_2026-07-17_14-01-38.json`](bhome_100_config_2026-07-17_14-01-38.json) (Bhome `100`, downloaded `2026-07-17`). `Observed`
- Older config values recorded here were taken from a `bhome_config.json` that is no longer in the project. Where the Bhome 100 config disagrees, the Bhome 100 values are recorded as the current observed values and the difference is noted. `Observed`

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
  - `start`: `x=149`, `y=23`, `z=325`
  - `end`: `x=2730`, `y=23`, `z=325`
- Station `b`
  - `start`: `x=148.7`, `y=541.75`, `z=325`
  - `end`: `x=2730`, `y=551.47`, `z=325`
- Frames for a hive should remain within the station `start.x` and `end.x` limits; otherwise the backend may not recognize the group as a hive. `Observed from test layout 3605`

### Hive

- Definition: In each station, the Bhome contains Hives. `Confirmed`.
- Role in system: A Hive is a repeating layout unit inside a station. `Confirmed`.
- Common pattern per hive:
  - 1 `partition`
  - 1 `feeder`
  - 12 `comb_frame`
  - 1 `static_partition`
- Common total per hive: 15 frame slots
- Inferred from sample: the sample appears to contain 8 hives total and 120 slots total

#### Hive Count And Hive Size Are Not Fixed

- The number of hives per station is set by the beekeeper, not by the system. `Confirmed`
- The usual setup is 4 hives per station side, but 3 hives per side is also common. `Confirmed`
- The beekeeper's setup determines where the `static_partition` frames are physically placed, so `static_partition` x-coordinates vary between Bhomes and are not the usual coordinates when fewer hives are configured. `Confirmed`
- With fewer hives per station, each hive spans more space and can hold many more frames than the common 15-slot pattern. `Confirmed`
- Therefore 15 slots per hive and 120 slots per Bhome are the common case, not a system constraint. `Confirmed`
- Practical consequence: hive boundaries and hive size must always be derived from the actual `partition` / `static_partition` positions in the layout being evaluated, never assumed from a fixed slot count. `Confirmed`
- Practical consequence for every rule below that uses `number_of_frames_in_hive` or a max-frame limit: the value must be counted from the layout under evaluation. `Confirmed`
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
- The `static_partition` is narrower than normal movable frames; observed width is approximately `14 mm`. `Confirmed from user observation`
- Because the `static_partition` is narrower, the center-to-center x-distance from the last movable frame to the `static_partition` should be smaller than the normal movable-frame spacing when building spacing-sensitive Auto Compress test layouts. `Confirmed`

#### `feeder`

- A `feeder` is a frame that can receive syrup for the bees in the hive to feed. `Confirmed`
- Historical rule: the feeder used to be placed at the beginning of the hive, near the dynamic partition. `Confirmed`
- Current rule: the feeder is now placed at the end of the hive, right next to the `static_partition`. `Confirmed`
- Reason for the change: actions such as Auto Bees-Balancing and Auto Compress Hives move comb frames and then tuck them against the dynamic partition. Placing the feeder at the `static_partition` end keeps it out of the way of those moves, so those actions never have to handle the feeder. `Confirmed`
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
- `place.station`: the station where the frame is located, such as `a` or `b`
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
- Config naming: Bhome 100 (`2026-07-17`) spells the key correctly as `auto_condense_hive_policy`. The earlier `auto_condence_hive_policy` typo is not present in this config version, but may still exist in implementation or in older Bhome configs. `Observed`

### Action Identity

- Auto Condense executes with:
  - `command = condenseHiveBundle`
  - `initiator = autoCondenseHive`

### Inputs

- Required minimum amount of hives in the Bhome to start condensing. `Confirmed`
- Expected minimum amount of frames that must remain in the hive after condense. `Confirmed`
- Current related Bhome config key:
  - `activity_manager.auto_condense_hive_policy.min_populated_hive_size = 4` `Observed in Bhome 100, 2026-07-17`
- Other Auto Condense keys in Bhome 100: `condense_cycle_days = 3`, `min_soc_for_activity = 25`, `min_voltage_for_activity = 23`, `min_temperature = 12`, `max_temperature = 45`, window `09:00`–`17:00`, Friday and Saturday disabled. `Observed`
- Bhome 100 has no config key for the required minimum amount of hives to start condensing. `Observed`

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
- Current defaults in [`bhome_100_config_2026-07-17_14-01-38.json`](bhome_100_config_2026-07-17_14-01-38.json) for Auto Condense:
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
- `condence` vs `condense` typos should be treated carefully because they may exist in older configs, in other Bhomes, or in implementation. The Bhome 100 config (`2026-07-17`) uses the correct `condense` spelling, so the spelling must be checked per config file rather than assumed. `Confirmed risk`
- Of the overlapping names above, only `min_populated_hive_size` exists in the Bhome 100 config. `Observed`

### Bhome Config Rules

Observed in [`bhome_100_config_2026-07-17_14-01-38.json`](bhome_100_config_2026-07-17_14-01-38.json) (Bhome `100`, `2026-07-17`). These are the values for this Bhome on this date; another Bhome may differ. `Observed`

- `thresholds.broodFrameThreshold = 25`
- `thresholds.honeyFrameThreshold = 25`
- `thresholds.beePopulationThreshold = 15`
- `thresholds.openBroodFrameThreshold = 5`
- `thresholds.openBroodLiveScanThreshold = 50`
- `activity_manager.frame_layout_policy.enabled = true`
- `activity_manager.frame_layout_policy.minimum_space_between_comb_frames = 30`
- `activity_manager.auto_scan_policy.auto_scan_type = 0`
- The reason for the minimum spacing rule is that comb frames are approximately 26 to 28 mm wide, so their x-coordinate distance should always be greater than 30. `Confirmed from user explanation`

Differences from the earlier removed `bhome_config.json`, still to be resolved:

- `auto_scan_policy.auto_scan_type`: earlier `2` (edge scan), Bhome 100 has `0`. The Scan Strategy section below describes edge scan as current behavior. `TBD`
- `auto_frames_split_policy.min_amount_brood_frames_in_hive`: earlier `5`, Bhome 100 has `4`. `TBD`
- The Auto Condense config key is `activity_manager.auto_condense_hive_policy` in Bhome 100, spelled correctly. The `condence` typo noted elsewhere in this document is not present in this config version. `Observed`
- The Auto Brood Donation config key is `activity_manager.auto_brood_donation_policy`, not `activity_manager.father_config.auto_brood_donation`. `Observed`

Config fields present in Bhome 100 that this rulebook had not recorded:

- `beekeeperLock.durationDays = 18`
- `super_detection.known_super_x_threshold = 20`
- `featureFlags`: `feed`, `scan`, `countBees` are `true`; `moveFrames`, `treatPests`, `adjustEntrances` are `false`
- `activity_manager.visual_inspection_type = beecount_only`
- `activity_manager.chill_out_time_after_opening_hives = 900`
- `activity_manager.auto_frames_split_policy.scan_brood_frame_during_split = true`
- `activity_manager.auto_frames_split_policy.auto_frames_split_optimized_planner = false`
- `activity_manager.field_beehome_mapping_policy.update_frame_layout_enable = true`
- Several activities are disabled in this Bhome: `auto_compress_hive_enabled`, `auto_brood_donation_enabled`, `auto_heat_treatment_enabled`, `auto_super_detection_enabled`, `auto_varroa_detection_enabled`, `auto_thermal_scan_enabled` are all `false`. `Observed`

Implication:

- Consecutive `comb_frame` positions should maintain an x-distance greater than or equal to the configured minimum spacing rule.
- This spacing rule is intended to prevent comb frames from being placed too close to each other.

### Frame Widths and Center Spacing

- Frame width and center-to-center spacing are two different measurements and must not be mixed. `Confirmed`
- Frame width: a `comb_frame` is approximately `26` to `28 mm` wide. `Confirmed`
- Center-to-center spacing: `36.6 mm` is the x-distance from one frame center to the next frame center, not a frame width. `Confirmed`
- The `partition` (dynamic) and `feeder` have the same width as a `comb_frame`. `Confirmed from user observation`
- Therefore the center-to-center x-distance between a `partition`, `feeder`, and an adjacent frame must be more than the frame width; they cannot be placed nearly on top of each other. `Confirmed`
- Practical consequence for building layouts: do not place a `partition` and `feeder` only a few mm apart (e.g. `5 mm`); use the normal center-to-center spacing (`~36.6 mm`) so frames do not overlap. `Confirmed`
- The `static_partition` is the exception: it is narrower (`~14 mm`), so its center-to-center distance to the last movable frame is smaller than normal slot spacing. `Confirmed` (see [static_partition](#static_partition))

### Overlapping Frame Coordinates

- If a layout has frame coordinates that place frames over each other, the system flags the hive as an invalid layout. `Confirmed`
- An invalid layout triggers an error for the technician. `Confirmed`
- Practical consequence for building test layouts: overlapping x-coordinates do not produce a usable test case for any activity rule; the layout fails validation before the activity logic is reached. `Confirmed`
- When a test layout needs frames close together, keep the center-to-center x-distance above the frame width so the frames do not physically overlap. `Confirmed`

### Confirmed Hive Pattern

Current pattern, with the feeder at the `static_partition` end:

1. `partition`
2. `comb_frame`
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
14. `feeder`
15. `static_partition`

Historical pattern, feeder at the dynamic partition end. Older layout files in this project still use it and should be read with this pattern in mind:

1. `partition`
2. `feeder`
3. `comb_frame` … 14. `comb_frame`
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

## Auto Brood Donation Rules

### Definitions

- `QL` / `Queen-less` hive: a hive whose most recent valid deep or edge scan indicates no open brood. `Confirmed`
- Open brood for Auto Brood Donation means 1-3 day old larvae from the youngest category. The exact implementation mapping between "youngest category" and JSON fields is under validation; test layouts should satisfy both `cell-worker-larva_0_to_1` and `cell-worker-larva_2_to_3` at `> 10%` on the intended donor frame until the backend field mapping is confirmed. `Observed from layout 3676 testing`
- Donor hive: a hive with sufficient brood that can donate a frame with open brood. `Confirmed`
- Recipient hive: a QL hive receiving the donated brood frame. `Confirmed`

### Config Gates

- The activity is controlled by `activity_manager.auto_brood_donation_policy`, with `auto_brood_donation_enabled` as the on/off flag. `Observed in Bhome 100, 2026-07-17`
- Earlier wording in this rulebook referred to `activity_manager.father_config.auto_brood_donation`; that path does not exist in the Bhome 100 config. `Observed`
- Auto Brood Donation can run only on enabled days of week and inside `auto_brood_donation_start_time` / `auto_brood_donation_end_time`. `Confirmed`
- Current example enabled days: Sunday through Thursday enabled, Friday and Saturday disabled. `Confirmed`
- Current example time window: `09:00` to `17:00`. `Confirmed`
- The QL result must be newer than `scan_validity_days`. Bhome 100 (`2026-07-17`) has `auto_brood_donation_policy.scan_validity_days = 2`. `Observed`
- Temperature must be between `min_temperature = 12` and `max_temperature = 45`. `Confirmed`
- `min_soc_for_activity = 15`. `Confirmed`

### Activation Preconditions

- Auto Brood Donation is considered only when a QL hive was found within `scan_validity_days`. `Confirmed`
- In the beekeeper app, the user must select `Inspect Scan and Correct`; `Inspect Scan only` must not activate correction. `Confirmed`
- After a DAN message is sent, cloud must verify:
  - QL hive was found during the last edge scan cycle within `scan_validity_days`. `Confirmed`
  - `num of sequences left` is `0` or not reported. `Confirmed`
  - Operational state is not `non-operational`. `Confirmed`
  - Transportation, visit, and super modes are not active. `Confirmed`
  - Bhome status is `Receptive`. `Confirmed`
  - Bhome is not during an auto activity cycle. `Confirmed`
  - Activity-manager priority allows the activity. `Confirmed`

### Recipient Hive Selection

- Choose a hive that was found QL from hive-level information newer than `scan_validity_days`. `Confirmed`
- The recipient hive must not be locked. `Confirmed`
- Apply the Auto Brood Donation max-frame rule per attempted QL hive: counted frames between dynamic `partition` and `static_partition` must be `<= 14`; otherwise skip that hive. `Confirmed`

### Donor Hive Selection

- Choose the unlocked hive with more brood frames than all other hives in the Bhome. `Confirmed`
- Apply the Auto Brood Donation max-frame rule per attempted donor hive: counted frames between dynamic `partition` and `static_partition` must be `<= 14`; otherwise skip that hive. `Confirmed`
- If no donor hive exists, no activity is defined. `Confirmed`
- Test-layout implication: donor hive selection happens before donor-frame open-brood selection. A hive with many capped-brood frames but no suitable young open-brood frame can be selected first and block the intended donor hive. Keep capped-only brood-frame counts below the intended donor hive in Auto Brood Donation layouts. `Observed from layout 3676`

### Donor Frame Selection

- The donated frame must have 1-3 day open brood and open brood must be `> 10%`. `Confirmed`
- If multiple frames qualify, select the one with the highest open brood cell count, or another deterministic tie-breaker. `Confirmed`
- If no suitable frame is found, the donation is not executed. `Confirmed`

### Recipient Frame Selection

- From the QL hive, select the frame with the lowest content score. `Confirmed`
- If no minimal-content frame exists, select the honey frame with the lowest honey amount. `Confirmed`

### Placement Rules

- Donor frame placement in the QL hive: station is the same as the QL hive; position is the position of the frame closest to the center of the QL hive `beesFrames` area plus `1/2 * comb width`. `Confirmed`
- Recipient frame placement in the donor hive: station is the same as the donor hive; position is between the brood and honey areas. `Confirmed`
- Pass the new frame positions to the `balance hive` planner and send the command to the Bhome. `Confirmed`
- Test-layout implication from log `No valid recipient frame destination found`: the required physical gap is in the donor hive for the frame moved from the QL hive into the donor hive, not necessarily in the QL hive after the selected recipient frame. Create an explicit empty destination between the donor hive's brood and honey areas. `Confirmed from layout 3676 testing`
- In layout `3676`, the donor hive can provide this destination by placing an extra honey frame on the lower-x side of the brood area and leaving one comb-frame-width empty destination between that honey frame and the first brood frame. `Confirmed from user clarification`
- Test-layout implication for donor hives: make the brood-to-honey boundary unambiguous. Avoid a frame at the boundary that qualifies as both brood and honey, because the planner may fail to find a valid destination for the QL recipient frame in the donor hive. `Observed from layout 3676`

### Locking And Notification

- After successful donation, lock the QL hive until the user unlocks it. `Confirmed`
- Notification must include QL hive number and frame exchange details. `Confirmed`
- The QL notification icon remains in the beekeeper app until the next hive-info evaluation. `Confirmed`

## Auto Bees-Balancing Rules

Source: Confluence page `Auto Bees-Balancing` (SE space, page 841449473). `Confirmed from spec`

### Potential Hive

- Not balanced since the last `BEE_BALANCING_START` event. `Confirmed`
- Not locked. `Confirmed`

### Gate Conditions

- Sequences left `0` or not reported; operational state not `non-operational`; not `no Scan` when `bees_balancing_initiate_scans = true`; no transportation/visit/super mode; BH status `Receptive`; `BEE_BALANCING_START` within `auto_scan_cycle_days` without a STOP since; day-of-week enabled; inside the balancing time window; inner temperature above minimum; SOC above minimum + 5% energy cost. `Confirmed`

### Data Freshness Requirements

- Hive-level: hives must be beeCounted or scan/count-attempted within `beecount_validity_days`; at least 2 potential hives beeCounted, else no activity. `Confirmed`
- Bhome 100 (`2026-07-17`) `activity_manager.auto_bees_balancing_policy`: `beecount_validity_days = 3`, `scan_validity_days_for_weak_hive = 5`, `bees_balancing_cycle_days = 3`, `min_brood_frames_in_weak_hive = 3`, `bees_balancing_initiate_scans = true`, window `09:00`–`17:00`, Saturday disabled. `Observed`
- If some potential hives lack fresh bee count data and `bees_balancing_initiate_scans = true`, balancing commands a beeCount on the stalest hive (`countBees (beeBalancing)`) instead of balancing this cycle. `Confirmed`
- Frame-level (VIF): the donor frame must have been scanned in the last `1` day and have capped brood `> 800` cells. Test layouts must have scan timestamps < 1 day old at run time. `Confirmed`

### Weak / Strong Hive Selection

- Weak hive: least `beesFrames` among potential hives with `beesFrames >= 3` and counted frames `<= 14`. If none, no activity. `Confirmed`
- A hive with fewer than 3 bee frames is NEVER selected as weak — it is dropped by the bee-frame gate before any queen or brood check is reached. Test-layout implication: to test a QL or brood-frame exclusion, the hive must have `beesFrames >= 3` and be the strict minimum among eligible hives. `Confirmed from spec + BH 193 runs (layouts 3713/3713-b were inconclusive for this reason)`
- Strong hive: most `beesFrames` among potential hives with at least `min_bees_frames_delta_for_balancing` more beesFrames than the weak hive, counted frames `<= 14`. If none, no activity. `Confirmed`
- The delta was a hardcoded `3` until CORE-3914 replaced it with `activity_manager.auto_bees_balancing_policy.min_bees_frames_delta_for_balancing`, default `3`. Present in the Bhome 100 config downloaded `2026-09-14`. `Confirmed from CORE-3914`
- The comparison is inclusive: a candidate whose delta equals the configured value stays eligible. `Confirmed from CORE-3914 acceptance criteria`
- Structural note for test design: the strong hive is always the candidate with the most beesFrames, and the delta grows with beesFrames, so raising the configured delta never swaps one strong hive for another. It can only remove the winner and end the activity. A layout that proves the value is read from config therefore needs a pair whose delta sits below the raised threshold. `Confirmed from layout 3914`
- Observed exception to the strong-hive rule: in the BH 191 run of layout `3884`, hive `A1` with `5` bee frames was evaluated as a strong candidate against a weak hive of `3` bee frames — a difference of `2`, not `3`. Either the threshold is `>= 2 more`, or the candidate list is not filtered by this rule before the VIF search. `Observed, needs confirmation`

### Weak-Hive Candidacy Gates (CORE-3884)

Gate order applied to `bhome_hives.hives`, from the current code logic. `Confirmed from CORE-3884 flow + BH 191 run of layout 3884-b, 2026-09-08`

Eligibility gates, applied to every hive:

1. Not locked for balancing — `getIsHiveBlockedForActivity(BALANCING)`. If none survive: `NO_HIVES_PRESENT`.
2. Counted recently — bees/brood timestamp `< beecount_validity_days` OR `>= 2` frames scanned. Every hive must pass; if any is stale, balancing issues `COUNT_BEES` on the oldest hive and skips the cycle.
3. Off cooldown — no AUTO balance in the last `bees_balancing_cycle_days`. Also drops hives with no `beesFrames.amount`.
4. Fits the frame gate — `<= 14` slots between partitions, and a dynamic `partition` must exist. `B4B` is exempt from this gate. If fewer than 2 hives survive: `NO_ELIGIBLE_HIVES_FOUND`.

Weak-hive candidacy gates, applied to the surviving eligible hives:

5. `beesFrames.amount >= MIN_BEES_FRAMES_FOR_WEAK_HIVE` (`3`). A hive below this is not a balancing target.
6. Queenless check on the beecount window — `queen timestamp < beecount_validity_days AND queen.exist === false` excludes the hive.
7. Brood-scan freshness — `broodFrames.timestamp < scan_validity_days_for_weak_hive` (`5` days). **Gates 8 and 9 apply only on this branch.** A hive whose brood scan is older than the window bypasses both and remains a candidate.
8. Queenless check on that scan — `queen.exist === false` excludes the hive, timestamp ignored. This catches what gate 6 misses when the queen timestamp is stale.
9. `broodFrames.amount >= min_brood_frames_in_weak_hive` (`3`). If no candidate survives: `NO_WEAK_HIVE_CANDIDATES_FOUND`.

- The operator on gate 9 is `>=`, not `>`. The Confluence spec section 5.4 reads "must have more than [min_brood_frames_in_weak_hive] brood frames"; that wording is wrong and should read "equal or more than". The code flow is correct. `Confirmed by user, 2026-09-14`
- Consequence: with the default of `3`, a hive needs `3` brood frames to pass, not `4`. `Confirmed`

Selection:

- Weak hive = the surviving candidate with the fewest `beesFrames.amount` (sort ascending, take first). `Confirmed`
- The weak hive is chosen from the whole candidate set first; the donor is then measured against it, never the reverse. `Confirmed`
- Strong-hive search runs afterwards over the other eligible hives. `Confirmed`

Consequence of the gate 7 branch: a hive with a stale brood scan is *easier* to become the weak hive than one with a fresh scan, because it skips both the scan-based queen check and the brood-frame minimum. `Confirmed`

#### Weak-Hive Gate Test Layouts

- Validated in `Layouts/3884-b.json` on BH 191, `2026-09-08`. Result: `A1` excluded as QL, `A2` excluded for brood frames `< 3`, `A3` selected as weak hive, `A4` donated. `Confirmed from run`
- `Layouts/3914.json` covers the CORE-3914 configurable delta over two successive balances in one cycle: `A1 = 3`, `A2 = 4`, `A3 = 5`, `A4 = 6`, `B2 = 7`, `B1 = 8` bee frames, with `B3` and `B4` below the bee-frame gate. With the delta at `3` the cycle balances `B1 -> A1` and then `B2 -> A2`; with the delta at `5` it balances `B1 -> A1` and then stops, because the second round's best delta is `3`. The difference between one balance and two is the observable proof that the value comes from config. `Confirmed from layout design, pending BH run`
- `Layouts/3914-b.json` is the simpler negative case: the largest candidate sits at delta `4`, so the whole activity disappears at a delta of `5` while still running at `3`. `Confirmed from layout design, pending BH run`
- Give every candidate hive in a delta test `4` brood frames. That clears the gate 9 minimum under either operator reading and keeps a delta test from being contaminated by an unrelated brood-frame question. `Confirmed`
- To make an exclusion attributable, the excluded hive must violate exactly one gate and must be the strict minimum in `beesFrames` among candidates — otherwise it would not have been selected anyway and the exclusion is invisible in the logs. `Confirmed`
- Both exclusions can be isolated in a single layout by stacking them below the control hive in bee frames: QL hive lowest, brood-poor hive next, control hive above both. Since the QL hive satisfies the brood minimum and the brood-poor hive has a queen, neither exclusion can be attributed to the other gate. `Confirmed from layout 3884-b`
- A QL hive can still hold `>= 3` brood frames if its brood is entirely capped (`cell-worker-larva_8_to_21`) with open brood at or below the QL threshold. This is what allows the queen gate to be tested independently of the brood gate. `Confirmed from layout 3884-b`
- Keep each hive's brood frames contiguous. Under edge scan, brood-region inference can fill gaps between brood frames and inflate `broodFrames.amount`, which breaks a hive built to have an exact brood-frame count. `Observed`
- Gate 6 fires before gate 8 whenever all timestamps are fresh, so a layout with uniformly fresh timestamps tests the queen exclusion at gate 6, not gate 8. Isolating gate 8 requires a stale queen timestamp together with a fresh brood scan. Which field feeds `queen timestamp` is not yet known, and pushing the beecount timestamp stale risks failing gate 2 and skipping the cycle. `TBD`

### Frame Selection

- VIF (from strong hive): brood frame with most capped brood, scanned in last 1 day, capped brood > 800. If fewer than 2 qualifying brood frames and initiate_scans is true, scan up to 2 frames near the bee-area center; else re-choose the strong hive excluding this one. `Confirmed`
- The 1-day VIF window is frame-level and is much shorter than the hive-level windows. A layout can pass weak-hive selection on `beecount_validity_days` (`3`) and gate 7 on `scan_validity_days_for_weak_hive` (`5`) and still fail every VIF check. `Confirmed from BH 191 run of layout 3884, 2026-09-08`
- Failure signature when the VIF window is missed: `VIF_SEARCH (Strong Hive X): No primary VIF found. Reason: SCANNING_DISABLED_AND_NO_PRIMARY_VIF. Brood frames found: 0`, repeated per strong candidate, ending in `No viable strong hive found for VIF after checking all candidates`. `Brood frames found: 0` here means no *recently scanned* brood frame, not an absence of brood content. `Observed`
- The log line `Last <n> days: 0 hives with scan attempts (); 0 hives with >= 2 executed frames ()` reports the VIF search window actually used; a value below `1` day confirms the frame-level window is in force. `Observed`
- When `bees_balancing_initiate_scans = false` there is no recovery from a missed VIF window: the fallback scan is disabled, so balancing walks the strong-hive candidate list and aborts. BH 191 has it `false`; Bhome 100 has it `true`, so this flag must be read per Bhome. `Confirmed`

#### Balancing Test Layout Implications

- Every `scanTimestamp` in a balancing test layout must be under 1 day old at run time, including the copies inside `beeCount` and inside `previous_side_a` / `previous_side_b`. In layout `3884` there were `800` such fields across `120` frames. `Confirmed`
- A balancing test layout therefore expires within 24 hours of being built and must have its timestamps refreshed before each upload. `Confirmed`
- Refresh all `scanTimestamp` occurrences, not just the `side_a` / `side_b` ones. In layout `3884` a stale value survived in 14 places — nested `beeCount` and `previous_side_*` copies — after the direct fields looked correct. `Observed`
- Frame `last_updated` and `previous_update_date` are ISO strings and were over a year stale in layout `3884` without blocking bee-count or weak-hive evaluation. They do not appear to gate balancing. `Observed`
- LIF (from weak hive): comb frame closest to the dynamic partition. `Confirmed`
- VIF new place: weak hive station, frame closest to center of weak hive bee area + 1/2 comb width. LIF new place: strong hive station, bee-area frame closest to dynamic partition - 1/2 comb width. `Confirmed`

## Auto Compress Hives Rules

### Open Hive Detection

- Auto Compress Hives determines whether a hive is open from frame x-position spacing. `Confirmed`
- Auto Compress Hives must define a hive as open if either the accumulated hive gap condition or the single consecutive-frame gap condition is true. `Confirmed`
- Cloud config must include `x_distance_tolerance_between_consecutive_frames_in_a_hive`. `Confirmed`
- Initial value for `x_distance_tolerance_between_consecutive_frames_in_a_hive`: `10 mm`. `Confirmed`
- Both tolerances live under `activity_manager.auto_compress_hive_policy`. Bhome 100 (`2026-07-17`) has `x_distance_tolerance_between_frames_in_a_hive = 5` and `x_distance_tolerance_between_consecutive_frames_in_a_hive = 10`. `Observed`
- Auto Compress is disabled in Bhome 100 (`auto_compress_hive_enabled = false`), window `19:00`–`21:00`. `Observed`
- Existing accumulated hive gap condition:
  `X[static_partition] - X[dynamic_partition] > 37.5 * number_of_frames_in_hive + 0.5 * dynamic_partition_width + x_distance_tolerance_between_frames_in_a_hive` `Confirmed`
- For the Auto Compress accumulated hive gap condition, observed backend behavior indicates `number_of_frames_in_hive` counts frames between the dynamic `partition` and the `static_partition`, excluding both partitions. A standard 15-slot hive therefore uses `13` for this formula. `Observed`
- New single consecutive-frame gap condition:
  `X[frame i+1] - X[frame i] > 37.5 + x_distance_tolerance_between_consecutive_frames_in_a_hive` `Confirmed`
- With the initial consecutive-frame tolerance of `10 mm`, any single consecutive-frame x-distance greater than `47.5 mm` should trigger Auto Compress. `Confirmed`
- The single consecutive-frame gap condition exists to catch a local gap between two neighboring frames even when the accumulated dynamic-partition-to-static-partition distance is still within tolerance. `Confirmed`
- Auto Compress open-hive logs can report the reason as:
  - `Accumulated`
  - `Consecutive`
  - `Both` `Observed from upload logs`
- Upload-log observation from test layout `3605`: a hive with repeated `47 mm` consecutive x-distances was marked as `Both`, so test cases that intend to isolate `Accumulated` should keep individual consecutive x-distances comfortably below `47 mm`. `Observed`

### Auto Compress Test Layout

- Test layout `3605` should be copied from an existing layout as the initial model. `Confirmed`
- In test layout `3605`, hive `A1` should have all consecutive-frame gaps below the single-gap threshold and should also remain below the accumulated hive gap threshold, so Auto Compress should not trigger. `Confirmed`
- In test layout `3605`, hive `A2` should trigger the original accumulated hive gap condition only. `Confirmed`
- In test layout `3605`, hive `A2` may be made smaller by removing comb frames so the accumulated threshold is lower while local consecutive-frame gaps stay below the consecutive trigger. `Confirmed`
- Current test-layout implementation for `A2`: remove 2 comb frames, leaving 13 total entries from `partition` through `static_partition`; use `37.45 mm` center-to-center x-distances between movable frames and a smaller `30 mm` final center-to-center x-distance into the narrow `static_partition`, so the hive triggers accumulated spacing without a large local gap. `Confirmed`
- In test layout `3605`, hive `A3` should trigger the new single consecutive-frame gap condition with one consecutive-frame x-distance greater than `47.5 mm`. `Confirmed`
- In test layout `3605`, hive `A4` should trigger both the accumulated hive gap condition and the single consecutive-frame gap condition. `Confirmed`
- In test layout `3605`, hive `A4` must still end inside station `a.end.x = 2730`; an earlier version ended beyond that limit and was not recognized as a hive. `Observed`
- The rest of the hives in test layout `3605` should not trigger Auto Compress. `Confirmed`
- For these test notes, "gap below/above `10 mm`" means the extra distance beyond the normal `37.5 mm` frame spacing, not the absolute x-distance between frame centers. `Confirmed`

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
- Split decisions use values from [`bhome_100_config_2026-07-17_14-01-38.json`](bhome_100_config_2026-07-17_14-01-38.json), especially `activity_manager.auto_frames_split_policy`. `Confirmed`
- Split evaluation uses all frames or edge-scan-derived data according to `activity_manager.auto_scan_policy.auto_scan_type`. `Confirmed`

#### Current Split Config Values

From [`bhome_100_config_2026-07-17_14-01-38.json`](bhome_100_config_2026-07-17_14-01-38.json):

- `activity_manager.auto_frames_split_policy.in_hive_populated_frames = 7`
- `activity_manager.auto_frames_split_policy.scan_validity_days = 2`
- `activity_manager.auto_frames_split_policy.beecount_validity_days = 2`
- `activity_manager.auto_frames_split_policy.min_amount_brood_frames_in_hive = 4` (earlier config had `5`) `Observed`
- `activity_manager.auto_frames_split_policy.min_amount_honey_frames_in_hive = 2`
- `activity_manager.auto_frames_split_policy.auto_frames_split_start_time = 09:00`
- `activity_manager.auto_frames_split_policy.auto_frames_split_end_time = 17:00`
- `activity_manager.auto_frames_split_policy.auto_frames_split_cycle_days = 4`
- `activity_manager.auto_frames_split_policy.scan_brood_frame_during_split = true`
- `activity_manager.auto_frames_split_policy.auto_frames_split_optimized_planner = false`

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
- The threshold source is [`bhome_100_config_2026-07-17_14-01-38.json`](bhome_100_config_2026-07-17_14-01-38.json) -> `thresholds.beePopulationThreshold`. `Confirmed`
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
- Brood cells for this percentage are the open-brood buckets plus capped brood: `cell-worker-larva_0_to_1`, `_2_to_3`, `_4_to_5`, `_6_to_7` and `cell-worker-larva_8_to_21`, summed over both sides. `Confirmed from layout 3884-b matching backend brood-frame counts`
- Practical content bar for building test layouts: at the usual `cell-drawnout = 3280` per side, the denominator is `6560`, so a frame needs `>= 1640` brood cells across both sides to count as a brood frame at `broodFrameThreshold = 25`. `Confirmed`
- Capped brood alone can satisfy this bar, which is how a `QL` hive can still hold several brood frames. `Confirmed from layout 3884-b`
- The threshold source is [`bhome_100_config_2026-07-17_14-01-38.json`](bhome_100_config_2026-07-17_14-01-38.json) -> `thresholds.broodFrameThreshold`. `Confirmed`
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
- The config key for this is `activity_manager.auto_scan_policy.auto_scan_type`. Earlier config had `2`; Bhome 100 (`2026-07-17`) has `0`. Since `auto_scan_type` is per-Bhome, the scan strategy in effect must be read from the config of the Bhome under test, not assumed. `Observed`
- The meaning of each `auto_scan_type` value, and which value selects edge scan, is not yet documented here. `TBD`

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
- General layout-level rule before Auto Compress tolerance handling: a hive is `open` if:
  `X[static_partition] - X[dynamic_partition] > 37.5 * number_of_frames_in_hive + 0.5 * dynamic_partition_width` `Confirmed`
- For Auto Compress Hives, use the Auto Compress Hives open hive detection rules, including the accumulated tolerance and the single consecutive-frame gap condition. `Confirmed`
- Otherwise, the hive is `closed`. `Confirmed`
- For this rule, `number_of_frames_in_hive` should be interpreted as the frames from the dynamic `partition` through the last frame before the `static_partition`, excluding the `static_partition` itself. `Inferred from layout match`
- Conflict to resolve: this counting definition includes the dynamic `partition` (a common 15-slot hive gives `14`), while the Auto Compress accumulated condition above counts frames strictly between the two partitions (a common 15-slot hive gives `13`). The two definitions differ by one frame, which is `37.5 mm` of threshold and can flip an open/closed result. `TBD`
- Whichever definition is confirmed, the count must be taken from the actual layout, since hive size varies with the beekeeper's hive-per-station setup. `Confirmed`
- The value `36.6` extracted from project layout data is the frame center-to-center x-distance, not the partition width. `Confirmed` (see [Frame Widths and Center Spacing](#frame-widths-and-center-spacing))
- Layout matching to date has used `0.5 * 36.6 = 18.3` for the `0.5 * dynamic_partition_width` term. Since `36.6` is now confirmed to be center-to-center spacing rather than a width, the true `dynamic_partition_width` for this formula is not yet confirmed. `TBD`

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
- A hive with `0` brood frames and `0` open brood is still evaluated: if its scanned frames have high recognized-cell percentage (`LBB_AGG_RAW > 70%`), the hive evaluates as `QL`, not `No Data`. `Confirmed from BH 192 testing, layout 3713 (A4)`
- Therefore `LBB_AGG_RAW` is computed across scanned frames, not only scanned brood frames; zero brood frames does not produce `No Data` when recognition is high. `Confirmed from BH 192 testing, layout 3713`
- Test-layout implication: to force `No Data`, the scanned frames' `cell` recognition must be low enough that `LBB_AGG_RAW <= 70%` (e.g. `cell` ≈ 2200 of 3280 ≈ 67%). High recognition + no open brood = `QL`. `Confirmed from BH 192 testing, layout 3713`
- If `LBB_AGG_RAW > 70%`, then `QR = False`. `Confirmed`
- Else, `QR = No Data`. `Confirmed`
- The comparison operators above are exact and must remain `>` and `<=` as written. `Confirmed`
- Validation from `Test - Queen Evaluation.json`: after the latest fix, the current observed results are `A1`, `A2`, `B1`, `B4` => `QR`; `A3`, `B2` => `QL`; `A4`, `B3` => `No Data`. `Confirmed`
- This validation indicates that queen evaluation now correctly finds open brood on inward second-side scanned frames, including the `B1` and `B4` edge-scan regression cases. `Confirmed`
- Current project threshold references:
  - `thresholds.openBroodFrameThreshold = 5`
  - `thresholds.openBroodLiveScanThreshold = 50`
- The queen-evaluation open-brood threshold is `openBroodLiveScanThreshold = 50`, not `openBroodFrameThreshold = 5`. `Confirmed by back-calculation from Test - Queen Evaluation.json`
- Derivation: aggregated open brood per hive in that validated layout is `A1 = 4088`, `A2 = 292`, `B1 = 245`, `B4 = 702` (all `QR`) against `A3 = 50`, `B2 = 20` (both `QL`). The boundary therefore lies at `50`, with `> 50` giving `QR` and `<= 50` giving `QL`, matching the documented operators. A threshold of `5` would have made `A3` and `B2` evaluate as `QR`. `Confirmed`
- Test-layout recipe for a deliberate `QL` hive: aggregated open brood `<= 50` across the hive, with `cell = 3280` on scanned sides so `LBB_AGG_RAW = 100% > 70%`. Using `20` keeps a safe margin below the boundary. `Confirmed from layout 3884-b`
- Test-layout recipe for a deliberate `QR` hive: aggregated open brood comfortably above `50`; `400` was used in layout `3884-b`. `Confirmed`

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
2. What are the exact meanings of the comb-frame side content fields?
3. What exact metric defines a honey frame?
4. Does edge-scan inference apply only to brood frames, or also to bee frames / honey frames?
5. What actions can be executed in the Bhome?
6. What are the conditions and thresholds used to decide actions?
7. Which `auto_scan_type` value selects edge scan, and what do the other values mean?
8. Does `number_of_frames_in_hive` include the dynamic `partition` or not? The two formulas in this document currently disagree by one frame.
9. What is the real `dynamic_partition_width` for the open-hive formula, now that `36.6` is confirmed to be center-to-center spacing and not a width?
10. What is `B4B`, and why is it exempt from the balancing `<= 14` frame gate?
11. Which field feeds the `queen timestamp` used by balancing gate 6, so that gate 8 can be tested in isolation?
12. Is the strong-hive margin `>= 3` more bee frames or `>= 2`? The BH 191 run of layout `3884` evaluated a candidate at `+2`.

Still unresolved from the CORE-3884 comparison, because the run did not exercise them:

- Cooldown mechanism: this document records "not balanced since the last `BEE_BALANCING_START` event", while the current code logic uses a rolling "no AUTO balance in the last `bees_balancing_cycle_days`". If the rolling window is correct, the `BEE_BALANCING_START` / STOP event logic is gone and a manual balance no longer blocks a hive. `TBD`

Answered and moved into the body:

- How many hives exist per station, and is that fixed or variable? See [Hive Count And Hive Size Are Not Fixed](#hive-count-and-hive-size-are-not-fixed).
- Why did feeder placement change, and where does the feeder go now? See [`feeder`](#feeder).
- What exact brood metric is compared against `broodFrameThreshold`? See [Brood Frame Conditions](#brood-frame-conditions).
- Which config threshold does queen evaluation compare open brood against? See [Queen Evaluation](#queen-evaluation).
