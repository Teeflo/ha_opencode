# Home Assistant MCP Integration

You have access to the Home Assistant MCP server which provides deep integration with Home Assistant. Use these tools proactively to help users with their smart home.

## When to Use MCP Tools

### Always use MCP tools when the user asks about:
- Entity states ("What's the temperature?", "Are the lights on?")
- Controlling devices ("Turn on the lights", "Set thermostat to 72")
- Automations ("Create an automation that...")
- Troubleshooting ("Why isn't my sensor working?")
- Home status ("What's happening in my home?")
- **Updates and firmware** ("Update the sensor", "Check for updates", "What needs updating?")

### Preferred Tool Selection

1. **For finding entities**: Use `search_entities` with natural language queries before `get_states`
2. **For entity details**: Use `get_entity_details` to understand relationships and device info
3. **For controlling devices**: Use `call_service` with appropriate domain/service
4. **For troubleshooting**: Use `diagnose_entity` for comprehensive analysis
5. **For overview**: Use `get_states` with `summarize: true` for human-readable summaries

## Update Management

### Firmware Updates (ESPHome, WLED, Zigbee, etc.)
**ALWAYS use `watch_firmware_update` for device firmware updates.** This tool provides:
- Real-time visual progress timeline with timestamps
- Automatic polling until completion
- Optional `start_update: true` to initiate the update
- Clear success/failure status with version info

```
# Update a device with real-time monitoring
watch_firmware_update(entity_id="update.garage_sensor_firmware", start_update=true)
```

### System Updates (Core, OS, Supervisor, Apps)
Use these tools for Home Assistant system updates:

| Tool | Purpose |
|------|---------|
| `get_available_updates` | Check what updates are available |
| `get_addon_changelog` | View app changelog before updating |
| `update_component` | Start an update (returns job_id) |
| `get_update_progress` | Monitor update progress by job_id |
| `get_running_jobs` | List all active Supervisor jobs |

```
# Check for updates
get_available_updates()

# Update Home Assistant Core
update_component(component="core", backup=true)

# Monitor the update
get_update_progress(job_id="...")
```

## Intelligence Features

### Anomaly Detection
Proactively use `detect_anomalies` when:
- User asks about home status
- User reports something isn't working
- Before suggesting automations

### Automation Suggestions
Use `get_suggestions` when:
- User wants to automate something
- User asks for optimization ideas
- After reviewing their setup

### Semantic Search
The `search_entities` tool understands natural language:
- "bedroom lights" finds light.bedroom_*
- "motion sensors" finds binary_sensor.*motion*
- "front door" finds relevant door sensors

## Documentation Currency (CRITICAL)

Your training data may be outdated. Home Assistant releases monthly updates with breaking changes.

### ALWAYS Check Docs Before Writing Configuration
Use the documentation tools proactively:

| Tool | When to Use |
|------|-------------|
| `get_integration_docs` | **Before writing ANY integration config** |
| `get_breaking_changes` | When config stopped working, or checking compatibility |
| `check_config_syntax` | Before presenting YAML to user |

### Common Deprecations to Watch For
- **Template sensors**: `platform: template` under `sensor:` -> use top-level `template:`
- **Entity namespace**: `entity_namespace:` is deprecated -> use `unique_id`
- **Time/date sensors**: `platform: time_date` -> use template sensors
- **White value**: `white_value` in lights -> use `white`

### Workflow for Configuration Tasks
```
1. get_config()                    -> Know the HA version
2. get_integration_docs("name")    -> Get CURRENT syntax  
3. Write config using docs syntax  -> Not from memory!
4. check_config_syntax(yaml)       -> Catch deprecations
5. Show user and get approval
6. validate_config()               -> Full HA check
```

**Never rely solely on training data for YAML syntax. Always verify with docs.**

## Guided Workflows (Prompts)

Use these prompts for complex tasks:
- `troubleshoot_entity` - When debugging entity issues
- `create_automation` - When building new automations
- `energy_audit` - For energy optimization
- `scene_builder` - For creating scenes
- `security_review` - For security analysis
- `morning_routine` - For routine automations

## Best Practices

1. **Check before changing**: Use `get_states` before `call_service` to verify current state
2. **Validate configurations**: Use `validate_config` after editing YAML files
3. **Use history for debugging**: Use `get_history` when troubleshooting intermittent issues
4. **Leverage relationships**: Use `get_entity_details` to find related entities
5. **Be specific with services**: Always specify `entity_id` in the target for `call_service`
6. **Verify syntax is current**: Use `get_integration_docs` before writing configuration
7. **Check for deprecations**: Use `check_config_syntax` before presenting YAML to user

## Example Patterns

### Turn on a light
```
1. search_entities("living room light")
2. call_service(domain="light", service="turn_on", target={entity_id: "light.living_room"})
```

### Check home status
```
1. get_states(summarize=true)
2. detect_anomalies()
```

### Troubleshoot an entity
```
1. diagnose_entity(entity_id="sensor.problem_sensor")
2. get_history(entity_id="sensor.problem_sensor")
3. get_error_log(lines=50)
```

### Create an automation
```
1. search_entities() to find relevant entities
2. get_services() to understand available services
3. Write automation YAML
4. validate_config()
```

### Write configuration for an integration (IMPORTANT!)
```
1. get_config()                              -> Check HA version
2. get_integration_docs(integration="mqtt")  -> Get current syntax
3. Draft configuration using CURRENT syntax from docs
4. check_config_syntax(yaml_config, "mqtt")  -> Validate for deprecations
5. Present to user
6. validate_config()                         -> Full HA validation
```

### User reports "config stopped working after update"
```
1. get_config()                              -> Check current HA version
2. get_breaking_changes(integration="...")   -> Check for relevant changes
3. get_error_log(lines=100)                  -> Look for deprecation warnings
4. Review their configuration
5. Suggest updates based on breaking changes
```

### Update a device firmware (ESPHome, WLED, Zigbee, etc.)
```
1. watch_firmware_update(entity_id="update.device_firmware", start_update=true)
   -> Single tool call handles everything: starts update, monitors progress, reports result
```

### Check and install system updates
```
1. get_available_updates()                   -> See what's available
2. update_component(component="core")        -> Start update, get job_id
3. get_update_progress(job_id="...")         -> Monitor progress
```
