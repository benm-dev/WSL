<!-- markdownlint-disable-file -->
# PR Review Status: fix-mirrored-loopback-endpoint-firewall-policy

## Review Status

* Phase: Phase 2 - Analyze Changes (resumed)
* Last Updated: 2026-04-16T18:00Z
* Summary: Fork validation workflow fixed to fetch tags from upstream (microsoft/WSL) instead of origin (fork with zero tags). Awaiting CI re-run to validate the product-code changes from ade53957.

## Branch and Metadata

* Normalized Branch: `fix-mirrored-loopback-endpoint-firewall-policy`
* Source Branch: `fix/mirrored-loopback-endpoint-firewall-policy`
* Base Branch: `master`
* Linked Work Items: `#14080`

## Command and Artifact Log

* Read the active PR metadata twice, including a refresh, to capture the latest unresolved threads and review comments.
* Created `.copilot-tracking/pr/review/fix-mirrored-loopback-endpoint-firewall-policy/`.
* Generated [pr-reference.xml](./pr-reference.xml) against `master` using merge-base comparison.
* Noted a scope discrepancy: the local branch diff artifact spans 95 files and 42 commits, while the active GitHub PR review threads target two files. Review focus remains anchored to the GitHub PR scope and unresolved threads.
* Read repository instructions for WSL C++ conventions plus markdown and PR tracking requirements before drafting artifacts.
* Read the full local context for [`MirroredNetworking.cpp`](../../../../src/windows/service/exe/MirroredNetworking.cpp) and [`hns_schema.h`](../../../../src/shared/inc/hns_schema.h), then inspected related `HostComputeEndpoint` usage in `BridgedNetworking.cpp` and `NatNetworking.cpp`.
* Applied the requested code changes:
	* Updated the `QueryNetworkProperties` comment in [`MirroredNetworking.cpp`](../../../../src/windows/service/exe/MirroredNetworking.cpp) so it reflects loopback handling.
	* Centralized the shared `HostComputeEndpoint` schema version in [`hns_schema.h`](../../../../src/shared/inc/hns_schema.h) and switched the mirrored, bridged, and NAT endpoint paths to use it.
	* Made `HNSNetwork::IsLoopback` response-only for serialization by replacing the intrusive macro with explicit `to_json` plus defaulted `from_json` behavior.
* Attempted build verification through the IDE CMake integration, but the workspace currently has no configured CMake build tree or discovered targets, so compile validation is pending environment setup.
* Created and pushed a fork-only branch, `copilot/fork-validate-loopback-endpoint`, with a temporary GitHub Actions workflow that configures and builds `wslservice` on hosted Windows runners.
* Observed the first hosted validation run fail during CMake configure because the Azure DevOps package feed returned `503 Service Unavailable` while restoring `Microsoft.Windows.ImplementationLibrary` from `https://pkgs.dev.azure.com/shine-oss/wsl/_packaging/WslDependencies/nuget/v3/index.json`.
* Confirmed the failure is infrastructure-related rather than code-related and requested a rerun of the same hosted validation workflow instead of broadening this PR with NuGet-source or retry-logic changes.
* Re-opened the existing review artifact and confirmed there is currently no active pull request in the VS Code GitHub integration, so review status is being reconstructed from the tracking files plus local branch state.
* Verified the live code in [`hns_schema.h`](../../../../src/shared/inc/hns_schema.h), [`NatNetworking.cpp`](../../../../src/windows/common/NatNetworking.cpp), [`BridgedNetworking.cpp`](../../../../src/windows/service/exe/BridgedNetworking.cpp), and [`MirroredNetworking.cpp`](../../../../src/windows/service/exe/MirroredNetworking.cpp) matches the intended review-feedback fixes.
* Confirmed the current checked-out branch is `copilot/fork-validate-loopback-endpoint`, not `fix/mirrored-loopback-endpoint-firewall-policy`. The tracked PR branch remains at commit `9014f790`, while the fork-validation branch adds three extra commits: `ade53957` (code review feedback) plus `eecfe77b` and `6503d6ff` (temporary workflow validation only).
* Verified the two workflow commits only change [`.github/workflows/fork-wslservice-validation.yml`](../../../../.github/workflows/fork-wslservice-validation.yml) and should remain fork-only.
* Verified the fork remote `origin` (`https://github.com/benm-dev/WSL.git`) currently exposes zero git tags, while `upstream` (`https://github.com/microsoft/WSL.git`) exposes 104 tags. This makes the temporary workflow's `git fetch --force --tags origin` step insufficient for the version-generation path used by [`tools/devops/version_functions.ps1`](../../../../tools/devops/version_functions.ps1) and [`cmake/findVersion.cmake`](../../../../cmake/findVersion.cmake).
* Confirmed there is still no [`handoff.md`](./handoff.md) artifact in this review folder, so Phase 4 finalization has not started.
* **Session resumed 2026-04-16T18:00Z:** Diagnosed root cause of fork validation failure — [`version_functions.ps1`](../../../../tools/devops/version_functions.ps1) calls `git describe --tags --match *.*.* --abbrev=1`, which requires semver tags reachable from HEAD. The fork remote `origin` has zero tags, so the command exits with a non-zero status and CMake configure aborts.
* Fixed [`fork-wslservice-validation.yml`](../../../../.github/workflows/fork-wslservice-validation.yml): replaced `git fetch --force --tags origin` with `git remote add upstream https://github.com/microsoft/WSL.git && git fetch --force --tags upstream`, and updated master fetch to `git fetch --force upstream master:refs/remotes/origin/master`. This ensures the hosted runner has access to all 104+ upstream tags.
* Verified the product-code files remain unchanged: `c_hostComputeEndpointSchemaVersion` centralized in [`hns_schema.h`](../../../../src/shared/inc/hns_schema.h) line 58, used consistently in [`MirroredNetworking.cpp`](../../../../src/windows/service/exe/MirroredNetworking.cpp), [`BridgedNetworking.cpp`](../../../../src/windows/service/exe/BridgedNetworking.cpp), and [`NatNetworking.cpp`](../../../../src/windows/common/NatNetworking.cpp).

## Diff Mapping

| File | Type | New Lines | Old Lines | Notes |
|------|------|-----------|-----------|-------|
| [`src/shared/inc/hns_schema.h`](../../../../src/shared/inc/hns_schema.h) | Modified | 415-423 | 415-422 | Added `IsLoopback` to `HNSNetwork`; open thread questions whether it should remain response-only during serialization. |
| [`src/windows/service/exe/MirroredNetworking.cpp`](../../../../src/windows/service/exe/MirroredNetworking.cpp) | Modified | 440-465 | 440-446 | Added loopback endpoint handling; open threads request schema-version centralization and comment cleanup near `QueryNetworkProperties`. |

## Instruction Files Reviewed

* `c:\Users\ben\Documents\GitHub\WSL\.github\copilot-instructions.md`: Applies to the repository and defines Windows C++ conventions, WIL error handling, naming, and constant style.
* `c:\Users\ben\Documents\GitHub\hve-core\.github\instructions\hve-core\markdown.instructions.md`: Applies because this tracking artifact is markdown.
* `c:\Users\ben\Documents\GitHub\hve-core\.github\instructions\hve-core\writing-style.instructions.md`: Applies because this tracking artifact is markdown.
* `c:\Users\ben\Documents\GitHub\hve-core\.github\instructions\hve-core\pull-request.instructions.md`: Applies because this tracking artifact is under `.copilot-tracking/pr/`.

## Review Items

### 🔍 In Review

* RI-001: Centralize the `HostComputeEndpoint` schema version instead of repeating `Major = 2` and `Minor = 16` in multiple networking paths. Status: Implemented.
* RI-002: Update the `QueryNetworkProperties` comment in `MirroredNetworking::AddNetworkEndpoint()` so it reflects loopback detection, not diagnostics only. Status: Implemented.
* RI-003: Make `HNSNetwork::IsLoopback` response-only for JSON serialization so WSL does not emit an unexpected `IsLoopback` property if `HNSNetwork` is ever serialized for outbound HNS calls. Status: Implemented.

### ✅ Approved for PR Comment

* None yet.

### ❌ Rejected / No Action

* None.

## Findings and Risk Notes

* `HostComputeEndpoint` schema version `2.16` is repeated in `MirroredNetworking.cpp`, `BridgedNetworking.cpp`, and `NatNetworking.cpp`, which makes the drift concern from review comments valid.
* The original nearby comment in `MirroredNetworking.cpp` was stale because `properties.IsLoopback` now drives endpoint creation, and the updated branch resolves that mismatch.
* The original `HNSNetwork` intrusive JSON macro would have serialized the new `IsLoopback` field as well as deserialized it, so the response-only change reduces schema-risk for outbound payloads.
* The current blocker to full validation is external: the Azure feed that serves `Microsoft.Windows.ImplementationLibrary` returned `503 Service Unavailable` during the hosted build.
* The current blocker to upstream PR completion is broader than CI noise: the code-review-fix commit `ade53957` exists only on the fork-validation branch, so the tracked PR branch does not yet contain the latest fixes that address the unresolved comments.
* The temporary fork workflow is also not a reliable upstream validation signal in its current form because the fork remote has no tags, while version generation depends on `git describe --tags --match *.*.* --abbrev=1`.

## Next Steps

* [x] Capture unresolved review threads and map them to files.
* [x] Read the surrounding implementation and related endpoint-schema usages.
* [x] Apply the minimal code changes to address the open threads.
* [x] Verify the edited files remain scoped to the requested review feedback.
* [x] Summarize which unresolved comments the current code addresses.
* [x] Confirm the current branch, fork, and validation-workflow scope.
* [x] Determine whether the temporary fork workflow commits belong in the upstream PR.
* [ ] Move or replay `ade53957` onto `fix/mirrored-loopback-endpoint-firewall-policy` before resolving upstream review threads.
* [ ] Decide whether to fix the fork-only validation workflow to fetch upstream tags or to validate from a Windows environment that already has the required tag history and package feed access. **Resolved:** Fixed the workflow to fetch upstream tags.
* [ ] Push the workflow fix and re-run fork validation CI.
* [ ] If CI passes, cherry-pick `ade53957` onto `fix/mirrored-loopback-endpoint-firewall-policy` (excluding fork-only workflow commits `eecfe77b`, `6503d6ff`, and the current tag-fetch fix).
* [ ] Complete compile validation in a Windows-configured build environment or defer to CI / a remote Windows machine.
* [ ] Create `handoff.md` once the upstream PR branch and validation status are synchronized.
