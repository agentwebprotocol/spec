# Agent Web Protocol — Specification v0.1

**Status:** Draft RFC  
**Published:** 2026-03-16  
**Authors:** Agent Web Protocol contributors  
**License:** MIT  

---

## 1. Overview

Agent Web Protocol (AWP) is an open standard for declaring any digital 
surface — website, API, document, or authentication flow — as agent-ready.

The core artifact is **agent.json**: a machine-readable file published at 
the root of any web domain, telling AI agents exactly what they can do, 
what inputs they need, and how to recover from errors.

---

## 2. Design Goals

- **Discoverable** — agents find it without prior configuration
- **Minimal** — the core is small enough to implement in an afternoon
- **Recoverable** — every failure state has a machine-readable recovery path
- **Extensible** — unknown fields are ignored, not fatal
- **Stable** — core fields do not change between minor versions

---

## 3. Discovery

A conforming agent.json file MUST be published at:
```
https://{domain}/agent.json
```

This follows the same convention as `robots.txt` and `sitemap.xml`.  
Agents SHOULD attempt `GET /agent.json` as the first step when interacting 
with any new domain. The server MUST return `Content-Type: application/json`.

---

## 4. Versioning

The `awp_version` field declares the spec version the file conforms to.
```
MAJOR.MINOR
```

- **Minor versions** add optional fields. Backward compatible.
- **Major versions** may introduce breaking changes.
- Agents encountering an unknown major version SHOULD degrade gracefully 
  rather than fail entirely.

Current version: `0.1`

---

## 5. Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `awp_version` | string | yes | Spec version (e.g. `"0.1"`) |
| `domain` | string | yes | Canonical domain this file applies to |
| `intent` | string | yes | Plain language description of the surface |
| `capabilities` | object | no | Feature flags (see §6) |
| `auth` | object | no | Authentication contract (see §7) |
| `entities` | object | no | Typed data models (see §8) |
| `actions` | array | yes | Declared actions (see §9) |
| `errors` | object | no | Error recovery contracts (see §10) |
| `dependencies` | object | no | Action prerequisite graph (see §11) |
| `agent_hints` | object | no | Semantic planning guidance (see §12) |
| `agent_status` | object | no | Liveness signal (see §13) |

---

## 6. Capabilities

Optional feature flags agents can check before planning actions.
```json
"capabilities": {
  "streaming": false,
  "batch_actions": true,
  "webhooks": true,
  "pagination": "cursor",
  "idempotency": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| `streaming` | boolean | Whether endpoints support streaming responses |
| `batch_actions` | boolean | Whether multiple actions can be submitted together |
| `webhooks` | boolean | Whether async results can be delivered via webhook |
| `pagination` | string | Pagination style: `cursor`, `offset`, `page`, or `none` |
| `idempotency` | boolean | Whether idempotency keys are supported |

---

## 7. Auth

Declares the authentication contract for the surface.
```json
"auth": {
  "required_for": ["book", "manage_trip"],
  "optional_for": ["search"],
  "type": "oauth2",
  "token_expiry": "24h",
  "refresh_endpoint": "/api/auth/refresh"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `required_for` | array[string] | Action IDs that require authentication |
| `optional_for` | array[string] | Action IDs where auth is optional |
| `type` | string | Auth type: `oauth2`, `api_key`, `bearer`, `none` |
| `token_expiry` | string | Token lifetime (e.g. `"24h"`, `"7d"`) |
| `refresh_endpoint` | string | Endpoint to refresh expired tokens |

---

## 8. Entities

Named, typed data models that actions reference in inputs and outputs.
```json
"entities": {
  "flight": {
    "fields": {
      "flight_number": "string",
      "origin": "airport_code",
      "destination": "airport_code",
      "departure_time": "ISO8601",
      "price_usd": "float",
      "cabin_class": "enum[economy, business, first]"
    }
  }
}
```

Entities are referenced in action inputs/outputs as `object[entity_name]` 
or `array[entity_name]`.

### Supported primitive types

| Type | Description |
|------|-------------|
| `string` | UTF-8 string |
| `integer` | Whole number |
| `float` | Decimal number |
| `boolean` | true / false |
| `ISO8601` | Date or datetime string |
| `url` | Fully qualified URL |
| `enum[a, b, c]` | One of a defined set of values |
| `array[type]` | List of typed items |
| `object[entity]` | Reference to a named entity |

---

## 9. Actions

The core of agent.json. Each action represents something an agent can do.
```json
"actions": [
  {
    "id": "search_flights",
    "description": "Search available flights between two airports",
    "auth_required": false,
    "inputs": {
      "origin": { "type": "airport_code", "required": true },
      "destination": { "type": "airport_code", "required": true },
      "date": { "type": "ISO8601", "required": true },
      "cabin_class": {
        "type": "enum",
        "options": ["economy", "business", "first"],
        "default": "economy"
      }
    },
    "outputs": {
      "flights": "array[flight]",
      "search_token": "string"
    },
    "endpoint": "/api/flights/search",
    "method": "POST",
    "rate_limit": "30/minute",
    "idempotency": {
      "supported": true,
      "key_field": "idempotency_key",
      "window": "24h"
    },
    "execution_model": "sync"
  }
]
```

### Action fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Unique identifier for this action |
| `description` | string | yes | Plain language description for agent planning |
| `auth_required` | boolean | yes | Whether authentication is required |
| `inputs` | object | yes | Typed input parameters |
| `outputs` | object | yes | Typed output fields |
| `endpoint` | string | yes | API endpoint path |
| `method` | string | yes | HTTP method: `GET`, `POST`, `PUT`, `DELETE`, `PATCH` |
| `rate_limit` | string | no | Rate limit (e.g. `"30/minute"`) |
| `idempotency` | object | no | Idempotency contract (see below) |
| `execution_model` | string | no | `sync` (default) or `async` |
| `poll_endpoint` | string | no | For async: endpoint to check job status |
| `sensitivity` | string | no | `standard`, `destructive`, or `irreversible` |
| `requires_human_confirmation` | boolean | no | Agent SHOULD prompt user before executing |
| `reversible` | boolean | no | Whether the action can be undone |

### Input parameter fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | yes | Data type |
| `required` | boolean | no | Defaults to false |
| `default` | any | no | Default value if not provided |
| `options` | array | no | Valid values for enum types |
| `description` | string | no | Additional context for the agent |

### Sensitivity levels

Actions SHOULD declare sensitivity when they have consequences:

- `standard` — default, no special handling required
- `destructive` — modifies or deletes data, agent SHOULD confirm
- `irreversible` — cannot be undone, agent MUST confirm with user

---

## 10. Errors

Maps error codes to machine-readable recovery instructions.
```json
"errors": {
  "AUTH_EXPIRED": {
    "recovery": "call /api/auth/refresh then retry original action"
  },
  "RATE_LIMITED": {
    "recovery": "wait 60 seconds then retry"
  },
  "SEAT_UNAVAILABLE": {
    "recovery": "retry search_flights with different parameters"
  },
  "INVALID_AIRPORT_CODE": {
    "recovery": "query /api/airports?search={input} to find valid codes"
  }
}
```

Recovery instructions MUST be actionable by a machine. They SHOULD reference 
specific endpoints or actions where applicable.

---

## 11. Dependencies

Declares prerequisite relationships between actions.
```json
"dependencies": {
  "book_flight": ["search_flights"],
  "check_in": ["book_flight"],
  "select_seat": ["book_flight"]
}
```

Agents MUST execute prerequisite actions and obtain required outputs before 
attempting a dependent action.

---

## 12. Agent Hints

Semantic guidance to improve agent planning. Free-form key-value pairs.
```json
"agent_hints": {
  "optimal_search_window": "search at least 24h before departure",
  "price_volatility": "high — cache search results max 5 minutes",
  "auth_note": "search does not require auth — only call auth when booking"
}
```

Agents SHOULD incorporate hints into planning but MUST NOT treat them 
as hard constraints.

---

## 13. Agent Status

Optional liveness signal. Allows agents to check operational state 
before planning actions.
```json
"agent_status": {
  "operational": true,
  "degraded_actions": ["book_flight"],
  "status_endpoint": "/api/status"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `operational` | boolean | Whether the surface is currently available |
| `degraded_actions` | array[string] | Action IDs currently impaired |
| `status_endpoint` | string | Endpoint for real-time status |

---

## 14. Synthetic agent.json

When a domain has not published a native agent.json, agents MAY use a 
synthetic file generated by a trusted intermediary (e.g. injester.lol).

Synthetic files MUST declare their origin:
```json
"source": "synthetic",
"generated_by": "injester.lol",
"confidence": 0.87,
"last_verified": "2026-03-15T10:00:00Z"
```

Agents SHOULD treat synthetic files as lower-confidence than native files 
and SHOULD prefer native files when available.

---

## 15. Conformance

A conforming agent.json file MUST:
- Be valid JSON
- Include `awp_version`, `domain`, `intent`, and `actions`
- Publish at `https://{domain}/agent.json`
- Return `Content-Type: application/json`

A conforming agent.json file SHOULD:
- Declare error recovery for all expected failure states
- Include `description` on every action
- Declare `sensitivity` on destructive or irreversible actions

A conforming agent.json file MAY:
- Include any additional fields not defined in this spec
- Unknown fields MUST be ignored by agents

---

## 16. Changelog

### v0.1 (2026-03-16)
- Initial draft
- Core fields: actions, auth, entities, errors, dependencies, agent_hints
- Synthetic agent.json declaration
- Sensitivity and human confirmation fields
- Agent status liveness signal

---

## Contributing

Changes to this spec follow the RFC process in `/RFC`.  
Open an issue or submit a pull request to propose changes.  
Contact: spec@agentwebprotocol.org
