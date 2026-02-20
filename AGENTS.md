# HA OpenCode Agent Guidelines

This document provides guidelines for agents working on the HA OpenCode codebase.

## Project Structure

```
ha_opencode/
├── ha_opencode/           # Main Docker application
│   ├── rootfs/
│   │   ├── opt/
│   │   │   ├── ha-mcp-server/    # MCP server (Node.js)
│   │   │   └── ha-lsp-server/    # LSP server (Node.js)
│   │   └── etc/                  # S6 overlay init scripts
│   ├── config.yaml               # App configuration
│   └── Dockerfile
├── .github/workflows/     # CI/CD
└── README.md
```

## Build Commands

### Docker Build (Local)
```bash
docker build -t ha-opencode:local ./ha_opencode
```

### Docker Build (Multi-arch)
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t ha-opencode ./ha_opencode
```

### Run MCP Server (Development)
```bash
cd ha_opencode/rootfs/opt/ha-mcp-server
npm install
npm start
```

### Run LSP Server (Development)
```bash
cd ha_opencode/rootfs/opt/ha-lsp-server
npm install
npm start
```

## Node.js Code Style

### General Rules
- Use ES modules (`"type": "module"` in package.json)
- Use async/await over raw promises
- Always handle errors with try/catch in async functions
- Use Zod for runtime validation of external data

### Imports
```javascript
// Named imports preferred
import { readFileSync } from 'fs';
import { Server } from '@modelcontextprotocol/sdk/index.js';

// Relative imports for local modules
import { MyClass } from './my-class.js';
```

### Naming Conventions
- **Files**: kebab-case (`my-module.js`)
- **Classes**: PascalCase (`class MyClass`)
- **Functions/variables**: camelCase (`myFunction`, `myVariable`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_RETRIES`)
- **Booleans**: Prefix with `is`, `has`, `can`, `should` (`isEnabled`, `hasData`)

### Error Handling
```javascript
// Always wrap async operations
async function fetchData() {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return await response.json();
  } catch (error) {
    throw new Error(`Failed to fetch data: ${error.message}`);
  }
}

// Use error subclasses for specific error types
class ConfigurationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ConfigurationError';
  }
}
```

### Types and Validation
- Use JSDoc for complex function signatures
- Validate all external input with Zod schemas
- Example Zod schema:
```javascript
import { z } from 'zod';

const EntityQuerySchema = z.object({
  entity_id: z.string().optional(),
  domain: z.string().optional(),
  state: z.string().optional(),
});

function queryEntities(input) {
  const validated = EntityQuerySchema.parse(input);
  // use validated data
}
```

## YAML Configuration Style

All YAML configuration files (automations, scripts, etc.) MUST follow the Home Assistant YAML Style Guide.

### Key Rules
- **Indentation**: 2 spaces (no tabs)
- **Booleans**: Only `true`/`false` (lowercase)
- **Strings**: Double quotes (`"value"`) - exceptions: entity_ids, action names
- **Lists**: Block style (`- item`) - never flow style `[item1, item2]`
- **Mappings**: Block style - never flow style `{ key: val }`
- **Null**: Use implicit null (`key:`) - never `null` or `~`
- **Templates**: Double quotes outside, single quotes inside
  ```yaml
  value_template: "{{ states('sensor.temperature') }}"
  ```

### Service Actions
Always use `target:` for entity targeting:
```yaml
actions:
  - action: light.turn_on
    target:
      entity_id: light.living_room
```

Reference: https://developers.home-assistant.io/docs/documenting/yaml-style-guide/

## Prettier Configuration

A Prettier config exists at `ha_opencode/rootfs/opt/ha-mcp-server/.prettierrc.yaml` for YAML files. It enforces:
- 2-space indentation
- Double quotes
- 120-character print width
- Preserves HA custom tags (`!include`, `!secret`)

## Docker/S6 Overlay

The app uses S6 Overlay for process management. Key directories:
- `etc/s6-overlay/s6-rc.d/init-opencode/` - Init scripts
- `etc/s6-overlay/s6-rc.d/ha-opencode/` - Service scripts

## Version Management

Version is managed in `ha_opencode/config.yaml`. On release:
1. Tag repo with `v{x.y.z}`
2. CI updates version automatically
3. Builds and pushes to GHCR

## Testing

There are currently no automated tests in this repository. When adding tests:
- Use Vitest for Node.js unit tests
- Test file pattern: `*.test.js` or `*.spec.js`
- Run single test: `npm test -- --run filename.test.js`

## Security Guidelines

- Never commit secrets or API keys
- Use environment variables for sensitive configuration
- Validate all external input (Zod schemas)
- Never expose Home Assistant tokens in logs

## Common Tasks

### Adding a New MCP Tool
1. Define Zod input schema
2. Implement handler function
3. Register in tools list
4. Add to INSTRUCTIONS.md documentation

### Modifying the LSP Server
1. Edit `ha-lsp-server/server.js`
2. Test with a YAML file in Home Assistant config
3. Verify diagnostics and completions work

### Building a Release
```bash
git tag v1.0.0
git push origin main --tags
```
