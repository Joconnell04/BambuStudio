# Continuous Support Base

## Summary

This change adds an optional experimental FFF setting called `Continuous support base (experimental)`.

When enabled, it modifies only the first normal-support layer that touches the build plate. The goal is to make that first support layer less fragmented and more continuous, so the printer spends less time making very short segments and repeated tight turns.

The setting is off by default.

## What Changed

### New setting

- Added a new print/object config key: `support_continuous_base`
- Added UI exposure in Support > Advanced
- Hid the setting when tree support is selected
- Added support-step invalidation so toggling the setting recomputes support

Relevant files:

- `src/libslic3r/PrintConfig.hpp`
- `src/libslic3r/PrintConfig.cpp`
- `src/slic3r/GUI/Tab.cpp`
- `src/slic3r/GUI/ConfigManipulation.cpp`
- `src/slic3r/GUI/GUI_Factories.cpp`
- `src/libslic3r/PrintObject.cpp`

### Support generation behavior

The implementation hooks into the existing classic support first-layer base shaping path in:

- `src/libslic3r/Support/SupportCommon.cpp`

It reuses the existing no-raft first-layer support-base branch and adds a guarded regularization pass:

1. Start from the already-generated first build-plate support base polygons.
2. Apply a conservative morphological closing step to merge very near fragments.
3. Apply outward smoothing to reduce sharp micro-features.
4. Re-trim against the first object layer keepout.
5. Union/simplify the result and remove tiny islands.
6. Fall back to the original geometry if the result is empty or changes area too aggressively.

## Scope

This implementation is intentionally narrow.

It affects:

- normal/classic support only
- the first support layer on the build plate
- no-raft first-layer support base shaping

It does not affect:

- upper support layers
- support interfaces directly under the model
- tree support
- support type selection
- preview/G-code architecture

## Important Guardrails

The feature falls back to existing behavior when:

- the setting is disabled
- tree support is selected
- there is no eligible first-layer base geometry
- the total first-layer support area is too small
- the merged result becomes empty
- the merged result shrinks or expands too far relative to the original area

## Review Notes and Bug Fix

During review, one real bug was found and fixed:

- The original implementation only hid the setting in the UI for tree support.
- A preset or project could still set `support_continuous_base=true` while using tree support.
- The core support hook now explicitly checks `is_tree(object.config().support_type)` and refuses to run in that case.

This keeps the behavior aligned with the intended scope.

## Test Coverage

Added a focused regression test in:

- `tests/fff_print/test_support_material.cpp`

The test compares support output with the setting off vs on and checks:

- the first support layer changes
- later support layers keep the same total toolpath length

## Verification Status

Static verification completed:

- `git diff --check` passed

Build/test execution was not completed in this environment because local build dependencies are not fully available. A temporary configure attempt reached dependency resolution and stopped on missing Boost setup for the project.

## Current Limitations

- The feature is intentionally disabled for tree support.
- The current implementation is geometry-regularization only; it does not introduce a new support fill type.
- The effect is limited to the existing no-raft first-layer support base path.

## Suggested Follow-up Validation

When build dependencies are available, validate:

1. Support preview with the setting off vs on on a support-heavy overhang model.
2. No change in upper support layers.
3. No change in support interfaces.
4. No crashes on models with no support, tree support, or raft-enabled jobs.
