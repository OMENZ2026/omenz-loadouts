# OMENZ Loadouts

**"What do you want your agent or robot to be able to do right now?"**

You pick the job (HOST MODE for a Saturday barbecue, PERSONAL TRAINER, RECEPTIONIST, an ACCOUNTANT skin for the agent you already have). The loadout describes it once, in provider-neutral capability words. A platform adapter maps it onto whatever body or agent runs it.

```
HUMAN REQUEST  ->  OMENZ LOADOUT  ->  PLATFORM ADAPTER  ->  ROBOT / AGENT CAPABILITIES
```

This repo is the open part: the `omenz.loadout.v0` spec, presets, platform profiles and the capability vocabulary. The hosted checks run at `https://www.omenz.org/api/tollbooth` (x402, USDC on Base, no account). They're free to try in the AYNE playzone at `/api/tollbooth/ayne/v1/...`.

## Why

- Robot "skills" and agent skills are already sold in stores, and audits show some run purchased tasks on hardware **without any confirmation**.
- There's no common manifest that says what a skill needs (arms? speech? e-stop?) or what limits it promises (speed, no-go zones, contact with people).
- Every platform names things differently: nav2 actions, `unitree_sdk2` clients, MCP tools. A job written for one body doesn't port to another.

A loadout states requirements and safety limits up front. A check tells you whether it fits a platform before anything moves, and a gate decides each physical step.

## Layers in a loadout

| layer | what it holds |
|---|---|
| skin | name, look, outfit where the hardware allows |
| persona / voice | tone, boundaries, greeting style |
| job | title + goals |
| tools | agent-side tools (smart-home, notify, OMENZ routes) |
| context | fields kept in a Job Jacket (menu, house rules), checked back in after the job |
| permissions | max speed, max payload, no-go zones, forbidden capabilities, human-confirm list, contact rules |
| behaviors / workflows | named step lists, each step = one capability word |
| requires / optional | capability words from `capabilities.json` |

## Hosted tools

| route | price | what it does |
|---|---|---|
| `GET /v1/loadouts` | free | catalog: presets, platforms, capability words |
| `POST /v1/loadout/check` | $0.01 | loadout x platform: READY / DEGRADED / NOT_COMPATIBLE per capability and workflow, add-ons, safety gaps, confirm list |
| `POST /v1/loadout/compile` | $0.02 | platform plan (nav2 goals for [ros-mcp](https://github.com/robotmcp/ros-mcp-server), `unitree_sdk2` calls, or MCP tools + passport request) + Ed25519-signed install receipt |
| `POST /v1/loadout/simulate` | $0.005 | run one workflow in a 2D sim home: step trace + gate decision on every step |
| `POST /v1/robot/action-gate` | $0.002 | before ONE physical action: ALLOW / CONFIRM_FIRST / DENY + signed receipt |

Try it free:

```bash
curl -s -X POST https://www.omenz.org/api/tollbooth/ayne/v1/loadout/check \
  -H 'content-type: application/json' -d '{"loadout":"host-mode","platform":"unitree-go2"}'
```

## Platform profiles (v0.1)

`ros2-mobile-manipulator` (via ros-mcp / rosbridge), `unitree-g1`, `unitree-go2`, `digital-agent-mcp`, `omenz-sim-home`. You can also send your own `{id, native, addon}` manifest.

**Honesty note:** the adapter mappings are dry-run plans written from public SDK docs. **None is verified on hardware yet.** OMENZ never sends commands to a robot. The robot's own safety system always has the last word. If you run one of these on real hardware, please open an issue with what worked and what didn't.

See [COMPATIBILITY.md](COMPATIBILITY.md) for the current matrix and [SPEC.md](SPEC.md) for the schema.

## Contributing

Platform profiles are the most useful contribution: what your robot exposes natively, what needs an add-on, and the exact action/topic/client names. Corrections to the capability mappings are welcome.

License: Apache-2.0 (spec, presets, profiles). By [OMENZ](https://omenz.org), hello@omenz.org.
