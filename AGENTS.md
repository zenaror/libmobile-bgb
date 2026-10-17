# Agent guidance

This project connects libmobile to the BGB emulator through the `mobile` program. The active branch is `feature/full_server`.

## Before starting

1. Consult internal memory, then OMM in scope `libmobile-bgb`. Read `project-base` with `get_role` when resuming; consult global context when useful. Memory does not replace facts verified in Git, code and evidence.
2. For REON, libmobile or the Mobile Adapter GB protocol, use the OMM skill `reon-libmobile-expert`.
3. Check the branch, HEAD, local changes and revision of `subprojects/libmobile`, which is maintained separately. Preserve its gitlink and the other checkout. This project's only remote is `https://github.com/zenaror/libmobile-bgb`.
4. Use the [README map](README.md#project-map) for code, documentation, tools and continuity. `source/` and the root-level `test.py` are this project's valid locations; do not reorganize upstream files for appearance.

## During the work

- Build in `build/` on Linux and in `build-windows/` with `../configure --host=x86_64-w64-mingw32 --without-system-libmobile` for Windows.
- Run tests with `ln -sf build/mobile mobile && python3 test.py`. Device-auth tests require effective UID 0 in the test environment; see the limitations in the [README](README.md#integration-and-validation).
- Do not run `git commit` or `git push` without the Operator's authorization for the work. History rewrites and force-push require specific authorization in the current session.
- Do not automatically add Claude/Anthropic `Co-authored-by` trailers to commits. Preserve legitimate human authors, coauthors and credits.
- Do not add an option to override the device-auth server.
- Do not store passwords, keys (such as `device_auth_key`) or personal data in files, commits or OMM.

## When finishing

Record new durable knowledge in OMM with its source (file, commit or conversation), and leave a handoff with the state, blockers and next steps. Keep internal memory and OMM up to date.

## Cleanup and retention

The canonical local policy is in [README.md — Cleanup and retention](README.md#cleanup-and-retention). Apply it at completion, checkpoints and pauses: clean only outputs proven to belong to the task, after preserving evidence and checking processes and paths; preserve real data, releases and Git recovery artifacts. Do not use broad cleanup or treat ignored/untracked files as disposable.
