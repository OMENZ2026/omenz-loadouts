---
name: omenz-loadouts
description: Equip an agent or robot with a job loadout (HOST MODE, PERSONAL TRAINER, RECEPTIONIST, ACCOUNTANT skin...). Check it against the robot platform (ROS 2 via ros-mcp, Unitree G1/Go2, MCP agent) before running, simulate it, and gate every physical action (ALLOW / CONFIRM_FIRST / DENY) with a signed receipt. Use when a user asks a robot or agent to "be" something for a job, or before running a third-party robot skill.
---

# OMENZ Loadouts

Base URL: `https://www.omenz.org/api/tollbooth`. Use `/ayne/v1/...` to try for free; `/v1/...` is paid per call over x402 (USDC on Base, no account).

## Steps

1. **Pick or write a loadout.** Call `GET /v1/loadouts` (free) for presets and capability words. Match the user's request to a preset (`host-mode`, `personal-trainer`, `receptionist`, `security-patrol-observe`, `skin-accountant`, `skin-daycare-starter`), or write an `omenz.loadout.v0` object (see SPEC.md).
2. **Check before running.** `POST /v1/loadout/check` with `{"loadout": "...", "platform": "unitree-g1"}`. Tell the user the verdict, the blocking capabilities, the add-ons to install, and the `safety_gaps` in plain words. Do not run anything that comes back NOT_COMPATIBLE.
3. **Simulate.** `POST /v1/loadout/simulate` with `{"loadout": "...", "workflow": "..."}` to show the user the step list and where a human yes is needed.
4. **Compile.** `POST /v1/loadout/compile` gets the platform plan and an `install_token`. Keep the token for this job.
5. **Gate every physical step.** Before each `move.*`, `arm.*` or `gesture.*` step, call `POST /v1/robot/action-gate` with `{"install_token": "...", "action": {"cap": "...", "args": {...}}, "context": {"human_confirmed": false, "children_present": false}}`.
   - ALLOW: run the step.
   - CONFIRM_FIRST: ask the human and run the step only after a clear yes, then call again with `human_confirmed: true`.
   - DENY: don't run the step. Tell the human why.

## Rules

- Never skip the gate for physical steps, and never send `human_confirmed: true` unless a human actually said yes.
- The robot's own e-stop and safety system always win over any ALLOW.
- Context such as house rules or menus goes in an OMENZ Job Jacket (`POST /v1/jacket/create`). Only store guest names if the guests agree.
