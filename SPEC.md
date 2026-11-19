# omenz.loadout.v0

```jsonc
{
  "schema": "omenz.loadout.v0",
  "id": "host-mode",                 // stable id
  "name": "HOST MODE",
  "kind": "embodied | digital | both",
  "for": "one sentence a human understands",
  "layers": {
    "skin":     {"name": "...", "look": {"accent_color": "#0aa0ff", "outfit": "..."}},
    "persona":  {"tone": "...", "boundaries": ["..."]},
    "voice":    {"style": "...", "greeting": "..."},
    "job":      {"title": "...", "goals": ["..."]},
    "tools":    ["agent-side tools or OMENZ routes"],
    "context":  {"jacket_fields": ["menu", "house_rules"]},
    "permissions": {
      "max_speed_mps": 0.6, "max_payload_kg": 1.0,
      "no_go_zones": ["stairs", "pool edge", "grill"],
      "require_human_confirm": ["arm.handover"],
      "forbidden": ["sense.recognize_person"],
      "contact_with_people": "handover only, never touch",
      "spend_cap_usd_per_day": 1.0
    },
    "behaviors": ["greet_at_gate", "drink_run"]
  },
  "requires": ["move.navigate_to", "speech.say", "safety.estop"],
  "optional": ["arm.pick", "home.music"],
  "workflows": [{"name": "drink_run", "steps": [{"cap": "move.navigate_to", "args": {"place": "drink_station"}}]}]
}
```

## Rules the checker applies

- Capability words come from `capabilities.json`. Unknown words are reported back, never guessed.
- On a body, any physical capability (`move.*`, `arm.*`, `gesture.*`) makes `safety.estop` required automatically.
- On a platform with no body (`digital-agent-mcp`), `safety.*` and physical steps don't apply. Workflows that need them come back not runnable.
- Agent-side capabilities (`home.*`, `notify.owner`, `timer.count`) work on any platform through the agent's own tools.
- Safety gaps flagged on bodies: no e-stop on the platform, no `max_speed_mps`, no `contact_with_people`, empty `no_go_zones`, contact capabilities missing from `require_human_confirm`, and identity recognition that isn't forbidden.

## Platform profile

```jsonc
{"id": "my-robot", "name": "...", "bridge": "how an agent reaches it",
 "native": {"move.navigate_to": "exact action/topic/client"},
 "addon":  {"arm.pick": "what must be installed"},
 "verified_on_hardware": false, "body": true}
```

## Signed records

`loadout/compile` returns `omenz.loadout-install.v0`: install_id, loadout digest, platform, allowed capabilities, limits, expiry. `robot/action-gate` returns `omenz.action-gate.v0` for each step. Both are Ed25519-signed. Verify them offline with the key at `GET https://www.omenz.org/api/tollbooth/v1/passport/key`.
