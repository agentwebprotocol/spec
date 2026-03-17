# Agent Web Protocol (AWP)

The open standard for declaring any web surface as agent-ready.

agent.json is to AI agents what robots.txt is to web crawlers — a simple, 
machine-readable file any domain can publish to tell agents exactly what 
they can do, what inputs they need, and how to recover from errors.

---

## The Problem

The web was built for human eyes and hands. AI agents fail on it constantly — 
not because the AI isn't capable, but because:

- Websites require JavaScript rendering before content appears
- Auth flows expire with no agent recovery path
- CAPTCHAs and MFA are designed to block non-humans
- Error messages are written for humans, not machines
- No standard exists to declare what actions are available on any surface

There is no equivalent of robots.txt for agent capabilities. Agents are left 
to infer, guess, and fail. Agent Web Protocol fixes this.

---

## The Pattern
```
robots.txt     →  what crawlers cannot access
sitemap.xml    →  what pages exist  
llms.txt       →  what content means
agent.json     →  what agents can DO
```

---

## How It Works

Any domain publishes a file at `/agent.json` — the same convention as 
robots.txt. Agents discover it automatically. No intermediary required.

The file declares:
- **Actions** — what the agent can do, with typed inputs and outputs
- **Auth** — what requires authentication and how to refresh tokens
- **Errors** — every failure state and its recovery instruction
- **Dependencies** — which actions must precede others
- **Hints** — semantic guidance for agent planning

→ [Full specification](./SPEC.md)  
→ [Schema reference and examples](https://agent-json.org)  
→ [Validator](https://agent-json.org/validate)

---

## Status

Current version: **v0.1 (draft)**  
Status: Open RFC — feedback welcome

This spec is community-governed. Changes follow the RFC process in `/RFC`.

---

## Contributing

Read [CONTRIBUTING.md](./CONTRIBUTING.md) for the RFC process.  
Open an issue to propose a change or flag a gap in the spec.  
Contact: spec@agentwebprotocol.org
