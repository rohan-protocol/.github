<h1 align="center">Rohan Protocol 🛡️</h1>

<p align="center">
  <b>The Stateless Zero-Knowledge Trust & Security Layer for Autonomous AI Agents.</b><br>
  <i>Universal Model Context Protocol (MCP) Gateway, WebMCP Browser Shield & In-Memory WASM Engine on Midnight.</i>
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@rohan-protocol/sdk"><img src="https://img.shields.io/npm/v/@rohan-protocol/sdk?style=for-the-badge&color=blue" alt="NPM Version"></a>
  <a href="https://rohanprotocol.network/"><img src="https://img.shields.io/badge/Spec-IEEE_Paper_v2.0-00629B?style=for-the-badge&logo=ieee" alt="IEEE Spec v2.0"></a>
  <a href="https://midnight.network/"><img src="https://img.shields.io/badge/Settlement-Midnight_Preprod_Live-black?style=for-the-badge" alt="Midnight Preprod"></a>
  <a href="#-streamable-http-ndjson-lifecycle"><img src="https://img.shields.io/badge/Transport-Streamable_HTTP_(NDJSON)-orange?style=for-the-badge" alt="Streamable HTTP"></a>
  <a href="#-formal-threat-and-mitigation-register"><img src="https://img.shields.io/badge/Security-V--01_|_V--02_|_V--07-red?style=for-the-badge" alt="Security Register"></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge" alt="License: MIT"></a>
</p>

> **📄 Foundational Research Paper:**  
> *["Rohan: A Stateless Zero-Knowledge Trust Gateway for Privacy-Preserving Agentic Workflows via Streamable HTTP"](https://github.com/rohan-protocol/sdk/blob/main/WHITEPAPER.md)*  
> By Julian von Bordelius (ORCID: [0009-0005-2436-0988](https://orcid.org/0009-0005-2436-0988)) — Model Context Protocol Working Group & Rohan Protocol Lab.

---

## ⚡ The Threat: Context Leaks & Blind Latency in Agentic AI

When autonomous AI agents negotiate or execute tools across heterogeneous boundaries using the **Model Context Protocol (MCP)** or **WebMCP**, modern enterprise infrastructure encounters three structural vulnerabilities:

1. **Context & Credential Exfiltration**: Sensitive parameters, prompt instructions, and PII are exposed in plaintext to centralized model proxies and external endpoints.
2. **Adversarial Prompt Injection (V-01 / V-07)**: Compromised agent counterparties or malicious web pages inject hidden override instructions (e.g., zero-width Unicode characters `[\u200B-\u200D\uFEFF]`) to coerce agents into executing unauthorized on-chain transactions.
3. **Stateful Socket Overhead & Blind Latency**: Legacy Server-Sent Events (SSE) and persistent WebSockets hold persistent TCP connections, breaking scale-to-zero serverless runtimes (Cloud Run, GKE) and forcing callers to endure 5–20 second timeouts without progressive state visibility.

---

## 🛡️ The Architecture: The Rohan Trinity

Rohan shifts the security perimeter from network firewalls to **stateless, client-side cryptographic verifications**. Proving circuits execute strictly inside isolated client memory, producing an ultra-compact **$\approx 580$-Byte ZK-SNARK** streamed via **Streamable HTTP (NDJSON)**:

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                 AUTONOMOUS AGENT RUNTIME                               │
│       Claude Desktop / Cursor IDE        │          Web Copilots / React Apps          │
│                  │                       │                      │                      │
│                  ▼                       │                      ▼                      │
│     [@rohan-protocol/mcp]                │           [@rohan-protocol/webmcp]          │
│     • Pre-Prover Firewall (V-01)         │           • DOM Sanitizer & Firewall (V-07) │
│     • Stdio MCP Protocol Server          │           • WebWorker Thread Isolation      │
└──────────────────┬───────────────────────┴──────────────────────┬──────────────────────┘
                   │                                              │
                   └──────────────────────┬───────────────────────┘
                                          │ Private Local Witness (w)
                                          ▼
                      ┌───────────────────────────────────────┐
                      │        [@rohan-protocol/sdk]          │
                      │  • In-Memory WASM Engine (WASI P2)    │
                      │  • zeroize Deterministic Wipe (V-02)  │
                      │  • Ultra-Compact Proof π (≈ 580 B)    │
                      └──────────────────┬────────────────────┘
                                         │ Streamable HTTP (POST /api/v1/handshake/stream)
                                         ▼
                      ┌───────────────────────────────────────┐
                      │          Rohan Relayer Station        │
                      │  1. received ➔ 2. firewall_approved  │
                      │  3. subsidizing_gas ➔ 4. confirmed   │
                      └──────────────────┬────────────────────┘
                                         │ On-Chain Anchoring ($tDUST sponsored)
                                         ▼
                      ┌───────────────────────────────────────┐
                      │       Midnight Blockchain Ledger      │
                      │  Contract: rohan_handshake            │
                      │  Preprod: 6d2d603235f996424d76c...    │
                      └───────────────────────────────────────┘

📦 The Ecosystem Packages
Package	Version	Architectural Role	Target Runtime
@rohan-protocol/sdk	v0.5.0	Core cryptographic proving engine, Poseidon hashing, and Streamable Relayer Client	Universal (Node.js, Deno, Bun)
@rohan-protocol/mcp	v0.5.0	Model Context Protocol Server with Pre-Prover Semantic Firewall (V-01)	Claude Desktop, Cursor, CLI Bots
@rohan-protocol/webmcp	v0.5.0	Browser-native WebMCP Shield with DOM Defense (V-07) & WebWorker Prover	React 18/19, Next.js, Extensions
## 🚀 Universal MCP Client Setup (Plug & Play)

Rohan is a universal, standard-compliant Model Context Protocol (MCP) server. Any AI agent, IDE, or reasoning engine supporting MCP can execute confidential Zero-Knowledge handshakes out-of-the-box.

### 1. Anthropic Claude Desktop
Add to your `claude_desktop_config.json` ([Config file locations](https://modelcontextprotocol.io/quickstart/user)):
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

2. Cursor IDE & Windsurf

In Cursor Settings > Features > MCP Servers > Add New MCP Server:

    Name: rohan-zk

    Type: command

    Command: npx -y @rohan-protocol/mcp
    (Alternatively, configure in ~/.cursor/mcp.json or Windsurf settings using the JSON schema above).

3. Google ADK 2.0 & Gemini Enterprise

Mount Rohan as a native tool execution gateway inside your Agent manifest or Python/TypeScript controller:
code TypeScript

import { GoogleGenAI } from "@google/genai";

// Connect to the local Rohan MCP stdio transport or remote Streamable HTTP gateway
const agent = new GoogleGenAI({
  model: "gemini-1.5-pro",
  tools: [{
    mcpServer: {
      command: "npx",
      args: ["-y", "@rohan-protocol/mcp"]
    }
  }]
});

4. Headless Bots & Universal MCP Hosts

For automated multi-agent mesh environments or custom orchestrators, launch Rohan directly over stdio:
code Bash

ROHAN_CONTRACT_ADDRESS="6d2d603235f996424d76c85186a79cc403245ea8ee1ba9087e40967fe71bdc4d" \
ROHAN_RELAYER_URL="https://api.rohanprotocol.network/api/v1/handshake/stream" \
npx -y @rohan-protocol/mcp

💬 Universal Prompt Execution

Once connected to your agent runtime (Cursor, Claude Desktop, Google or custom MCP host), models can autonomously negotiate and anchor handshakes in plain natural language:

 *"Seal our confidential data-sharing agreement with `did:midnight:agent-partner-9x4a...` using Rohan ZK Handshake."*

 **Note on DIDs:** `did:midnight:...` represents the decentralized identity (W3C DID) of your counterparty agent. Rohan Protocol acts as the neutral, zero-knowledge verification layer — agents retain full cryptographic ownership of their identities.


💻 Developer Quickstart: Node.js / TypeScript SDK

Install the core package:
code Bash

npm install @rohan-protocol/sdk

Execute an in-memory ZK-Handshake with live Streamable HTTP stage telemetry:
code TypeScript

import { RohanProver, RohanRelayerClient } from '@rohan-protocol/sdk';

// 1. Initialize the Relayer Client (Gasless — No Web3 Wallet required)
const relayer = new RohanRelayerClient({
  relayerUrl: 'https://api.rohanprotocol.network',
  contractAddress: '6d2d603235f996424d76c85186a79cc403245ea8ee1ba9087e40967fe71bdc4d',
});

// 2. Initialize the client-side WASM Prover
const prover = new RohanProver();

// 3. Compute the proof locally in RAM (Sensitive parameters remain confidential)
const proofData = await prover.generateHandshakeProof({
  agentId: 'did:midnight:agent-01',
  intent: 'execute_confidential_settlement',
  privateData: { maxTransfer: 5000, authCode: 'ALPHA_VERIFIED' },
});

console.log('✅ ZK-Proof computed locally (~580 Bytes). Witness memory scrubbed.');

// 4. Stream transaction via Streamable HTTP (NDJSON) with live progress tracking
const receipt = await relayer.submitProofStream(proofData, (event) => {
  switch (event.stage) {
    case 'received':
      console.log('📡 [1/4] Ingress acknowledged by relayer.');
      break;
    case 'firewall_approved':
      console.log('🛡️ [2/4] Semantic Firewall (V-01) validation passed.');
      break;
    case 'subsidizing_gas':
      console.log(`⛽ [3/4] Relayer subsidizing gas ($tDUST) via ${event.gasPayer}.`);
      break;
    case 'confirmed':
      console.log(`🎉 [4/4] Finalized on Midnight! TxHash: ${event.txHash}`);
      break;
  }
});

⚡ Streamable HTTP (NDJSON) Lifecycle

During execution, @rohan-protocol/sdk processes real-time chunked transfers from POST /api/v1/handshake/stream:
Stage	Streamed NDJSON Payload	Semantic Description
received	{"stage":"received","timestamp":1788903808207}	Relayer gateway confirms proof ingress
firewall_approved	{"stage":"firewall_approved","v01":"passed"}	Pre-Prover Firewall verifies policy constraints
subsidizing_gas	{"stage":"subsidizing_gas","gasPayer":"rohan-relayer-node-01"}	Paymaster wallet sponsors transaction fees ($tDUST)
confirmed	{"stage":"confirmed","status":"success","txHash":"0x7c92...","intentHash":"0916..."}	Transaction verified & anchored to Midnight Preprod
📊 Empirical Benchmarks (WASM / WASI)

Benchmarked across consumer, edge, and cloud hardware executing the verify_batched_handshakes circuit:
Hardware Tier	Proof Time (

        
Tprove
Tprove​

      

)	Memory Peak (

        
Mfootprint
Mfootprint​

      

)	Proof Payload Size
Intel Core i9-13900K (Desktop)	315 ± 12 ms	41.8 MB	

        
≈580
≈580

      

Bytes
Apple M3 Pro (Laptop / Worker)	465 ± 18 ms	43.2 MB	

        
≈580
≈580

      

Bytes
ARM Cortex-A76 (Raspberry Pi 5)	1180 ± 65 ms	46.5 MB	

        
≈580
≈580

      

Bytes

Serverless Scalability: In comparative 24-hour benchmarks, Streamable HTTP allowed Google Cloud Run containers to scale to zero within 120 seconds of idle time, reducing cold-start compute costs by over 94% compared to persistent Server-Sent Events (SSE).
🔒 Formal Threat and Mitigation Register
ID	Target Layer	Threat Vector	Technical Impact	Protocol Mitigation
V-01	Semantic / MCP Layer	Indirect Prompt Injection (M2M)	Unauthorized proof generation	Pre-Prover Semantic Firewall (TF-IDF intent sharding & declarative JSON guardrails) in @rohan-protocol/mcp.
V-02	WASM Runtime Heap	Serverless / Client Heap Scraping	Plaintext witness (

        
w
w

      

) extraction	Deterministic zeroization (zeroize): Buffer overwritten with null bytes (0x00) immediately after proof synthesis.
V-03	Relayer / Paymaster	Gas Station Exhaustion / DDoS	Depletion of relayer

        
tDUST
tDUST

      

gas	Rate-Limited API Gateway: Ephemeral API-key verification before transaction sponsorship.
V-07	Browser DOM / WebMCP	Client-Side Prompt Injection	Tool spoofing & session theft	DOM Sanitizer: Strips zero-width Unicode ([\u200B-\u200D\uFEFF]) and executes proving inside thread-isolated WebWorkers.
🏛️ Live Settlement Infrastructure

    Network: Midnight Preprod Testnet

    Smart Contract: rohan_handshake.compact (compiled via compactc v0.30.0)

    Contract Address: 6d2d603235f996424d76c85186a79cc403245ea8ee1ba9087e40967fe71bdc4d

    Streamable Relayer Endpoint: https://api.rohanprotocol.network/api/v1/handshake/stream

📚 Academic Citation

If you integrate the Rohan Protocol, its WASI Preview 2 prover components, or the Streamable HTTP transport standard in your research or production systems, please cite our specification:
code Bibtex

@inproceedings{vonbordelius2026rohan,
  author    = {Julian von Bordelius},
  title     = {Rohan: A Stateless Zero-Knowledge Trust Gateway for Privacy-Preserving Agentic Workflows via Streamable HTTP},
  booktitle = {Model Context Protocol Working Group & Rohan Protocol Lab},
  year      = {2026},
  url       = {https://rohanprotocol.network/}
}

<p align="center">
<i>License: MIT © Rohan Protocol. Built for the Autonomous Agentic Economy on Midnight.</i>
</p>
```

