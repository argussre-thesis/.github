# Argus SRE

**AI agents for incident management on Kubernetes, connected to infrastructure through MCP.**

Argus SRE is the platform built for my university thesis on how AI agents can investigate production incidents. When an alert fires, an agent gathers evidence from the affected cluster and its monitoring, writes a root cause analysis, and then answers engineers' questions about it in a chat.

The thesis question: *can an agent with read-only access through the Model Context Protocol (MCP) find the root cause of real Kubernetes failures, and can its conclusions be trusted?* Argus SRE is the system built to explore that question end to end, from the alert to the report.

## How it works

```
 Grafana / Alertmanager                                       Customer's cluster
          │ alert                                           ┌───────────────────┐
          ▼                                                 │  Runner           │
      Backend ──start investigation──▶ Agent service        │  (read-only)      │
          ▲                              │        │         └─────────┬─────────┘
          │                         MCP  │        │ MCP               │ outbound WebSocket
      Frontend                           ▼        ▼                   ▼
  (incidents, RCA, chat)      Org's MCP servers   Gateway ◀───────────┘
                              (e.g. Grafana)
```

1. **An alert arrives** from Grafana or Alertmanager, and the backend groups it into an incident.
2. **The agent investigates.** Its tools come from MCP servers: the cluster (pods, logs, events, deployments) through the gateway, plus any MCP servers the team added, such as Grafana for metrics and logs.
3. **The report must cite its evidence.** Every claim has to point to a tool call that actually ran, or the report is rejected.
4. **Engineers review it** in the web app or in Slack, and can ask follow-up questions in a chat (Slack only).

## Key ideas

- **MCP as the agent's only access path.** The agent never talks to the infrastructure directly. Everything it can see is an MCP tool, so its access is explicit and easy to limit.
- **Read-only by design.** The in-cluster runner's Kubernetes role can only read. It can't change, delete or see Secrets.
- **No inbound ports.** The runner opens the connection out to the gateway, so a cluster can be connected without firewall changes.
- **Evidence over confidence.** Reports are checked against the tool calls behind them.

## Repositories

| Repository | What it does | Stack |
| --- | --- | --- |
| [argus-sre-frontend](https://github.com/argussre-thesis/argus-sre-frontend) | Web app: setup, incidents, RCA reports, chat | React, TypeScript, Vite, Clerk |
| [argus-sre-backend](https://github.com/argussre-thesis/argus-sre-backend) | API, alert and Clerk webhooks, Slack, database | Express, Prisma, PostgreSQL |
| [argus-sre-agent-service](https://github.com/argussre-thesis/argus-sre-agent-service) | The AI agent: investigations and chat | LangChain, Claude, MCP |
| [argus-sre-gateway](https://github.com/argussre-thesis/argus-sre-gateway) | Connects runners and exposes clusters as MCP servers | Socket.IO, MCP |
| [argus-sre-runner](https://github.com/argussre-thesis/argus-sre-runner) | Read-only agent inside the cluster, installed with Helm | Node.js, Kubernetes |

---

Thesis by [Tsandas](https://github.com/Tsandas) · <!-- University, department, year -->
