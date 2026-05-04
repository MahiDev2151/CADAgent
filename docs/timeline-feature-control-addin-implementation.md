# Timeline Feature Control Add-In Implementation

## What Changed

- Added suppress/unsuppress handling for feature operation messages in both macOS and Windows add-in copies.
- Added `delete_feature` handling for feature operation messages in both macOS and Windows add-in copies.
- Added `feature_tools.delete_feature` so the add-in can resolve a feature token, validate optional guardrails, delete the feature or its timeline object, recompute, and return structured result data.
- Added `feature_tools.set_feature_suppression` to resolve a feature token, validate optional identity guardrails, toggle suppression, recompute, and return updated state.
- Declared `client_capabilities.timeline_feature_delete` and `client_capabilities.timeline_feature_suppression` in execution requests so the backend can safely choose the structured payload path only for compatible add-ins.
- Added `suppression_supported` to feature snapshots so the backend and agent can see whether a feature exposes suppression state.
- Broadened through-all hole fallback detection so `NO_TARGET_BODY` / "No target body" failures retry the opposite extent direction and then use the sketch-cut fallback before failing.
- Changed timeline revert handling so designs without a timeline return chat-history rollback success with `geometry_reverted=False`.
- Added explicit Parametric Design Mode gating before claiming a geometry timeline revert.
- Avoided moving the Fusion timeline marker when `deleteAllAfterMarker` is unavailable and the revert can only be chat-history-only.
- Fixed palette rollback rendering so visible chat trims by checkpoint `message_id` before using backend provider-history length as a fallback.
- Persisted request/checkpoint correlation IDs in document UI state so delayed checkpoints survive rerenders and document switches.
- Added operation-resume UI fallback that trims stored run state when the live DOM checkpoint anchor is missing.
- Cleared persisted checkpoint IDs on disconnect/session change so stale revert buttons do not reappear after rerender.
- Changed the add-in release workflow to publish only through explicit `workflow_dispatch`, not every `main` push.

## Why

The backend can now use one consistent structured transport for both reversible timeline suppression and direct feature deletion when the connected add-in supports it, while still keeping safer fallback behavior for problematic hole creation.

## Design Notes

- Timeline operations still require Parametric Design Mode.
- Feature tokens are resolved through Fusion's `findEntityByToken`.
- Name and timeline index are safety checks only.
- Delete support uses the same `feature_operation` transport as suppression instead of inventing a separate message type.
- No-op suppression returns success without recomputing.
- Delete and suppression both recompute the design before reporting success.
- Recompute failures now attempt to restore and verify the original suppression state before failing the operation; successful suppression also verifies the final Fusion state and fails if that verification cannot be read.
- Mac and Windows add-in copies were kept identical for this behavior.
- Missing timeline support no longer logs a Fusion error during chat rollback.
- `timeline_reverted` is forwarded to the palette and chat-only rollback shows a small "model unchanged" toast.

## Setup And Migration

- No environment or manifest changes are required.
- Add-in publishing is now manual-only through the release workflow inputs.
- Fusion smoke testing is recommended because the local test environment does not provide Autodesk's `adsk` runtime.
- Backend delete payloads require an add-in that declares `timeline_feature_delete`; otherwise the backend should continue using its older codegen delete path.
