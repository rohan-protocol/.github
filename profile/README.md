<h1 align="center">Rohan Protocol</h1>

<p align="center">
  <strong>The stateless zero-knowledge trust and security layer for autonomous AI agents.</strong><br>
  <em>Universal MCP gateway, WebMCP browser shield, and in-memory WASM engine on Midnight.</em>
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@rohan-protocol/sdk"><img src="https://img.shields.io/npm/v/@rohan-protocol/sdk?style=for-the-badge&color=blue" alt="NPM Version"></a>
  <a href="https://rohanprotocol.network/"><img src="https://img.shields.io/badge/Spec-IEEE_Paper_v2.0-00629B?style=for-the-badge&logo=ieee" alt="IEEE Spec v2.0"></a>
  <a href="https://midnight.network/"><img src="https://img.shields.io/badge/Settlement-Midnight_Preprod_Live-black?style=for-the-badge" alt="Midnight Preprod"></a>
  <a href="#streamable-http-ndjson-lifecycle"><img src="https://img.shields.io/badge/Transport-Streamable_HTTP_(NDJSON)-orange?style=for-the-badge" alt="Streamable HTTP"></a>
  <a href="#formal-threat-and-mitigation-register"><img src="https://img.shields.io/badge/Security-V--01_|_V--02_|_V--07-red?style=for-the-badge" alt="Security Register"></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge" alt="License: MIT"></a>
</p>

> **Foundational research paper**  
> [*Rohan: A Stateless Zero-Knowledge Trust Gateway for Privacy-Preserving Agentic Workflows via Streamable HTTP*](https://github.com/rohan-protocol/sdk/blob/main/WHITEPAPER.md)  
> By Julian von Bordelius ([ORCID 0009-0005-2436-0988](https://orcid.org/0009-0005-2436-0988)) — Model Context Protocol Working Group & Rohan Protocol Lab.

## Table of Contents

- [The problem](#the-problem-context-leaks-and-blind-latency-in-agentic-ai)
- [Architecture](#architecture-the-rohan-trinity)
- [Ecosystem packages](#ecosystem-packages)
- [Universal MCP setup](#universal-mcp-client-setup)
- [SDK quickstart](#developer-quickstart-nodejstypescript-sdk)
- [Streamable HTTP lifecycle](#streamable-http-ndjson-lifecycle)
- [Threat register](#formal-threat-and-mitigation-register)
- [Live settlement infrastructure](#live-settlement-infrastructure)
- [Academic citation](#academic-citation)
- [License](#license)

## The problem: Context leaks and blind latency in agentic AI

When autonomous AI agents negotiate or execute tools across heterogeneous boundaries using the **Model Context Protocol (MCP)** or **WebMCP**, modern enterprise infrastructure encounters three structural vulnerabilities:

1. **Context and credential exfiltration** — Sensitive parameters, prompt instructions, and PII are exposed in plaintext to centralized model proxies and external endpoints.
2. **Adversarial prompt injection (V-01 / V-07)** — Compromised agent counterparties or malicious web pages inject hidden override instructions (for example, zero-width Unicode characters `[\u200B-\u200D\uFEFF]`) to coerce agents into executing unauthorized on-chain transactions.
3. **Stateful socket overhead and blind latency** — Legacy Server-Sent Events (SSE) and persistent WebSockets hold TCP connections open, making scale-to-zero serverless runtimes harder to operate and leaving callers without progressive state visibility.

## Architecture: The Rohan Trinity

Rohan shifts the security perimeter from network firewalls to **stateless, client-side cryptographic verification**. Proving circuits execute inside isolated client memory and produce an ultra-compact **approximately 580-byte ZK-SNARK**, streamed through **Streamable HTTP (NDJSON)**.

```text
┌───────────────────────────────────────────────────────────────────────────────┐
│                          AUTONOMOUS AGENT RUNTIME                             │
│     Claude Desktop / Cursor IDE         │        Web Copilots / React Apps    │
│                  │                      │                    │                │
│                  ▼                      │                    ▼                │
│     [@rohan-protocol/mcp]               │       [@rohan-protocol/webmcp]      │
│     • Pre-Prover Firewall (V-01)        │       • DOM Sanitizer (V-07)        │
│     • Stdio MCP Protocol Server         │       • WebWorker Isolation         │
└──────────────────┬──────────────────────┴─────────────────┬───────────────────┘
                   │                                        │
                   └──────────────────┬─────────────────────┘
                                      │ Private local witness (w)
                                      ▼
                   ┌───────────────────────────────────────┐
                   │       [@rohan-protocol/sdk]           │
                   │  • In-memory WASM engine (WASI P2)    │
                   │  • Deterministic zeroize (V-02)       │
                   │  • Compact proof π (≈ 580 bytes)      │
                   └──────────────────┬────────────────────┘
                                      │ Streamable HTTP
                                      │ POST /api/v1/handshake/stream
                                      ▼
                   ┌───────────────────────────────────────┐
                   │          Rohan Relayer Station        │
                   │  received → firewall_approved         │
                   │  subsidizing_gas → confirmed          │
                   └──────────────────┬────────────────────┘
                                      │ On-chain anchoring ($tDUST sponsored)
                                      ▼
                   ┌───────────────────────────────────────┐
                   │        Midnight Blockchain Ledger     │
                   │  Contract: rohan_handshake            │
                   │  Preprod: 6d2d603235f996424d76c...    │
                   └───────────────────────────────────────┘
```

## Ecosystem packages

| Package | Version | Architectural role | Target runtime |
| --- | --- | --- | --- |
| [`@rohan-protocol/sdk`](https://www.npmjs.com/package/@rohan-protocol/sdk) | `v0.5.0` | Core cryptographic proving engine, Poseidon/SHA-256 commitment hashing, and Streamable Relayer Client | Node.js, Deno, Bun |
| [`@rohan-protocol/mcp`](https://www.npmjs.com/package/@rohan-protocol/mcp) | `v0.5.0` | MCP server with Pre-Prover Semantic Firewall (V-01) | Claude Desktop, Cursor, Google ADK, CLI bots |
| [`@rohan-protocol/webmcp`](https://www.npmjs.com/package/@rohan-protocol/webmcp) | `v0.5.0` | Browser-native WebMCP shield with DOM defense (V-07) and WebWorker proving | React 18/19, Next.js, browser extensions |

## Universal MCP client setup

Rohan is a standard-compliant MCP server. Any AI agent, IDE, or reasoning engine that supports MCP can execute confidential zero-knowledge handshakes.

### Claude Desktop

Add the following to `claude_desktop_config.json`. See the [official configuration guide](https://modelcontextprotocol.io/quickstart/user) for file locations.

```json
{
  "mcpServers": {
    "rohan-zk-gateway": {
      "command": "npx",
      "args": ["-y", "@rohan-protocol/mcp"],
      "env": {
        "ROHAN_CONTRACT_ADDRESS": "6d2d603235f996424d76c85186a79cc403245ea8ee1ba9087e40967fe71bdc4d",
        "ROHAN_RELAYER_URL": "https://api.rohanprotocol.network/api/v1/handshake/stream"
      }
    }
  }
}
```

### Cursor IDE and Windsurf

In Cursor, open **Settings → Features → MCP Servers → Add New MCP Server**:

- **Name:** `rohan-zk`
- **Type:** `command`
- **Command:** `npx -y @rohan-protocol/mcp`

Alternatively, configure the server in `~/.cursor/mcp.json` or the corresponding Windsurf settings.

### Google ADK and Gemini

Mount Rohan as a native tool using a local MCP stdio transport or a remote Streamable HTTP gateway:

```typescript
import { GoogleGenAI } from "@google/genai";

const agent = new GoogleGenAI({
  model: "gemini-1.5-pro",
  tools: [{
    mcpServer: {
      command: "npx",
      args: ["-y", "@rohan-protocol/mcp"]
    }
  }]
});
```

### Headless bots and custom MCP hosts

```bash
ROHAN_CONTRACT_ADDRESS="6d2d603235f996424d76c85186a79cc403245ea8ee1ba9087e40967fe71bdc4d" \
ROHAN_RELAYER_URL="https://api.rohanprotocol.network/api/v1/handshake/stream" \
npx -y @rohan-protocol/mcp
```

Once connected, agents can negotiate and anchor handshakes using natural language:

> “Seal our confidential data-sharing agreement with `did:midnight:agent-partner-9x4a...` using Rohan ZK Handshake.”

`did:midnight:...` represents the decentralized identity (W3C DID) of the counterparty agent. Rohan Protocol acts as the neutral zero-knowledge verification layer; agents retain cryptographic ownership of their identities.

## Developer quickstart: Node.js / TypeScript SDK

Install the core package:

```bash
npm install @rohan-protocol/sdk
```

Generate a proof in memory and submit it to the relayer with live Streamable HTTP stage telemetry:

```typescript
import { RohanProver, RohanRelayerClient } from "@rohan-protocol/sdk";

const relayer = new RohanRelayerClient({
  relayerUrl: "https://api.rohanprotocol.network", 
  contractAddress: "6d2d603235f996424d76c85186a79cc403245ea8ee1ba9087e40967fe71bdc4d"
});

const prover = new RohanProver();

const proofData = await prover.generateHandshakeProof({
  agentId: "did:midnight:agent-01",
  intent: "execute_confidential_settlement",
  privateData: { maxTransfer: 5000, authCode: "ALPHA_VERIFIED" }
});

console.log("ZK proof computed locally (~580 bytes). Witness memory scrubbed.");

const receipt = await relayer.submitProofStream(proofData, (event) => {
  switch (event.stage) {
    case "received":
      console.log("[1/4] Ingress acknowledged by relayer.");
      break;
    case "firewall_approved":
      console.log("[2/4] Semantic Firewall (V-01) validation passed.");
      break;
    case "subsidizing_gas":
      console.log(`[3/4] Relayer subsidizing gas ($tDUST) via ${event.gasPayer}.`);
      break;
    case "confirmed":
      console.log(`[4/4] Finalized on Midnight. TxHash: ${event.txHash}`);
      break;
  }
});
```

## Streamable HTTP (NDJSON) lifecycle

The SDK processes chunked transfers from `POST /api/v1/handshake/stream`:

| Stage | Example NDJSON payload | Description |
| --- | --- | --- |
| `received` | `{"stage":"received","timestamp":1788903808207}` | Relayer gateway confirms proof ingress |
| `firewall_approved` | `{"stage":"firewall_approved","v01":"passed"}` | Pre-Prover Firewall verifies policy constraints |
| `subsidizing_gas` | `{"stage":"subsidizing_gas","gasPayer":"rohan-relayer-node-01"}` | Paymaster sponsors transaction fees ($tDUST) |
| `confirmed` | `{"stage":"confirmed","status":"success","txHash":"0x7c92...","intentHash":"0916..."}` | Transaction is verified and anchored to Midnight Preprod |





## Formal threat and mitigation register

| ID | Target layer | Threat vector | Technical impact | Protocol mitigation |
| --- | --- | --- | --- | --- |
| V-01 | Semantic / MCP layer | Indirect prompt injection (M2M) | Unauthorized proof generation | Pre-Prover Semantic Firewall with TF-IDF intent sharding and declarative JSON guardrails in `@rohan-protocol/mcp` |
| V-02 | WASM runtime heap | Serverless / client heap scraping | Plaintext witness (`w`) extraction | Deterministic zeroization: buffers are overwritten with null bytes (`0x00`) after proof synthesis |
| V-03 | Relayer / Paymaster | Gas station exhaustion / DDoS | Depletion of relayer gas | Rate-limited API gateway with ephemeral API-key verification before sponsorship |
| V-07 | Browser DOM / WebMCP | Client-side prompt injection | Tool spoofing and session theft | DOM sanitizer strips zero-width Unicode (`[\u200B-\u200D\uFEFF]`) and proving runs inside thread-isolated WebWorkers |

## Live settlement infrastructure

- **Network:** Midnight Preprod Testnet
- **Smart contract:** `rohan_handshake.compact` (compiled with `compactc v0.30.0`)
- **Contract address:** `6d2d603235f996424d76c85186a79cc403245ea8ee1ba9087e40967fe71bdc4d`
- **Streamable Relayer endpoint:** `https://api.rohanprotocol.network/api/v1/handshake/stream`

## Academic citation

If you integrate Rohan Protocol, its WASI Preview 2 prover components, or the Streamable HTTP transport in research or production systems, cite the specification:

```bibtex
@inproceedings{vonbordelius2026rohan,
  author    = {Julian von Bordelius},
  title     = {Rohan: A Stateless Zero-Knowledge Trust Gateway for Privacy-Preserving Agentic Workflows via Streamable HTTP},
  booktitle = {Model Context Protocol Working Group & Rohan Protocol Lab},
  year      = {2026},
  url       = {https://rohanprotocol.network/}
}
```

## License

MIT © Rohan Protocol. Built for the autonomous agentic economy on Midnight.


