<h1 align="center">Rohan Protocol</h1>

<p align="center">
  <strong>A security and settlement gateway for AI agents.</strong><br>
  <em>MCP integrations, browser-side protections, and cryptographic intent commitments.</em>
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@rohan-protocol/sdk"><img src="https://img.shields.io/badge/NPM-v0.6.0-blue?style=for-the-badge" alt="NPM version 0.6.0"></a>
  <a href="https://zenodo.org/records/22730240"><img src="https://img.shields.io/badge/Research-Zenodo%20Preprint-1682d4?style=for-the-badge" alt="Zenodo preprint"></a>
  <a href="https://midnight.network/"><img src="https://img.shields.io/badge/Network-Midnight-black?style=for-the-badge" alt="Midnight Network"></a>
  <a href="#streamable-http"><img src="https://img.shields.io/badge/Transport-Streamable%20HTTP-orange?style=for-the-badge" alt="Streamable HTTP"></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge" alt="MIT License"></a>
</p>

Rohan is a set of client libraries and gateway components for routing agent tool
calls through policy checks and submitting cryptographic commitments to a
relayer. This README describes the intended architecture and provides
illustrative setup examples; verify package APIs, network status, and plan
details against the current releases before deploying.

## Contents

- [How it works](#how-it-works)
- [Privacy model and limitations](#privacy-model-and-limitations)
- [Packages](#packages)
- [Setup](#setup)
- [SDK quickstart](#sdk-quickstart)
- [Streamable HTTP](#streamable-http)
- [Relayer plans](#relayer-plans)
- [Security considerations](#security-considerations)
- [Network details](#network-details)
- [Research and citation](#research-and-citation)
- [License](#license)

## How it works

An agent integration can inspect or filter a tool request, derive a
cryptographic commitment from the request's intent and private inputs, and
submit the commitment to a relayer. The relayer handles the configured
settlement flow on Midnight.

The privacy boundary depends on what the client actually sends. In the
intended flow, private witness data is processed on the client and the
commitment (along with the metadata needed to route and settle the request) is
sent to the relayer. Do not include secrets in fields that are transmitted to
the relayer.

```text
Agent / application
        |
        v
MCP or WebMCP integration
  - request and policy checks
  - browser-side protections, where applicable
        |
        v
SDK
  - local commitment generation
  - best-effort clearing of sensitive buffers
        |
        | commitment + required request metadata
        v
Rohan relayer
  - request handling and configured sponsorship
        |
        v
Midnight settlement network
```

### Cryptographic commitments and memory clearing

The documented protocol uses a 32-byte intent commitment. A commitment is a
fixed-size cryptographic digest that can bind data to a request; it is not
itself a proof that the underlying request is valid, nor does its size alone
establish confidentiality. Confidentiality depends on keeping private inputs
out of transmitted payloads and logs.

The `zeroize` crate and similar mechanisms can overwrite specific buffers
after use. This is best-effort memory hygiene, not hardware-level sanitization
or a guarantee that every copy has been erased. Copies may exist in language
runtimes, allocators, logs, crash dumps, swap, or application code. Treat
inputs as sensitive throughout their lifecycle and review the complete
deployment path.

## Privacy model and limitations

- The client should keep private witness values local and submit only the
  commitment and necessary public metadata.
- A 32-byte commitment should not be described as a 32-byte zero-knowledge
  proof. The proof system and payload format must be verified for the exact
  SDK version and flow in use.
- Memory clearing reduces residual-data risk for buffers it actually clears;
  it cannot prove that all runtime or operating-system copies are gone.
- MCP policy checks and prompt-injection filters can reduce risk but cannot
  guarantee that an agent will never be manipulated or make an unsafe call.
- This README does not claim GDPR, HIPAA, SOC 2, or other regulatory
  compliance. Compliance depends on the complete product and deployment.

## Packages

| Package | Description | Example environments |
| --- | --- | --- |
| [`@rohan-protocol/sdk`](https://www.npmjs.com/package/@rohan-protocol/sdk) | Client SDK for commitment generation and relayer requests | Node.js, Deno, Bun, edge runtimes (check runtime compatibility) |
| [`@rohan-protocol/mcp`](https://www.npmjs.com/package/@rohan-protocol/mcp) | MCP server integration for agent tool calls | MCP-compatible clients |
| [`@rohan-protocol/webmcp`](https://www.npmjs.com/package/@rohan-protocol/webmcp) | Browser-oriented WebMCP integration | Browser applications |

Package availability, versions, and runtime support can change. Consult the
package registry and the package-specific documentation for current details.

## Setup

### Claude Desktop

Add the MCP server to your Claude Desktop configuration. Use an API key issued
to you; never commit a live key to source control.

```json
{
  "mcpServers": {
    "rohan": {
      "command": "npx",
      "args": ["-y", "@rohan-protocol/mcp"],
      "env": {
        "ROHAN_RELAYER_URL": "https://api.rohanprotocol.network/api/v1/handshake",
        "ROHAN_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

For Cursor or another MCP-compatible client, add the same command and
environment variables using that client's MCP server settings.

### Google ADK

The following illustrates the integration pattern. Confirm the constructor and
tool registration API against the installed package versions:

```ts
import { RohanMcpGateway } from "@rohan-protocol/mcp";

const gateway = new RohanMcpGateway({
  transport: "streamable-http",
  relayerUrl: "https://api.rohanprotocol.network/api/v1/handshake",
  apiKey: process.env.ROHAN_API_KEY
});

agent.registerTool(gateway.asAdkTool());
```

## SDK quickstart

Install the SDK:

```bash
npm install @rohan-protocol/sdk
```

Example request using the SDK API shown in the v0.6.0 documentation:

```ts
import { RohanClient } from "@rohan-protocol/sdk";

const client = new RohanClient({
  relayerUrl: "https://api.rohanprotocol.network/api/v1/handshake",
  apiKey: process.env.ROHAN_API_KEY
});

const receipt = await client.submitHandshake({
  agentId: "did:example:agent-01",
  intent: "submit_example_request",
  privateData: { example: "sensitive input; review what the SDK transmits" }
});

console.log(receipt.status, receipt.txHash);
```

Review the SDK implementation and network request payloads before passing
production secrets as `privateData`. Ensure that the installed package
processes sensitive inputs locally and does not serialize them to the relayer.

## Streamable HTTP

The documented streaming endpoint is:

```text
POST /api/v1/handshake/stream
```

The response may provide newline-delimited JSON lifecycle events. The
following is an illustrative shape, not a guarantee of current server
behavior:

```jsonl
{"stage":"received"}
{"stage":"firewall_approved"}
{"stage":"subsidizing_gas"}
{"stage":"confirmed","status":"success","txHash":"<transaction-hash>","intentHash":"<commitment>"}
```

Treat event fields and stage names as API details that may vary by release.
Handle network errors, non-success HTTP responses, and incomplete streams in
your integration.

## Relayer plans

The following indicative plans were listed in previous project materials.
Confirm current quotas and pricing on the [Rohan Protocol website](https://rohanprotocol.network/)
before relying on them.

| Tier | Use case | Listed quota | Listed price |
| --- | --- | ---: | ---: |
| Sandbox | Testing and evaluation | 10 handshakes per day | Free |
| Developer | Staging and development | 500 handshakes per month | $0 |
| Pro Agent | Production agents and startups | 25,000 handshakes per month | $49 per month |
| Scale | Higher-throughput workloads | 125,000 handshakes per month | $199 per month |

Any fee sponsorship, quota, rate limit, service level, and availability is
subject to the current service terms.

## Security considerations

Security controls are useful risk mitigations, not proofs that a system is
invulnerable. Validate the implementation and deployment configuration for
your threat model.

| Area | Risk | Practical mitigation |
| --- | --- | --- |
| Agent and MCP tools | Malicious or misleading tool input can influence an agent | Validate inputs and authorization at the application boundary; do not rely on prompt text as an access-control mechanism |
| Client memory | Sensitive values may remain in memory or be copied by a runtime | Minimize secret lifetime, avoid unnecessary copies and logging, and use memory-clearing primitives where supported |
| Relayer | Abuse, outages, or overuse can affect availability and cost | Use scoped API keys, quotas, rate limits, and operational monitoring |
| Smart contract | Contract or configuration errors can affect settlement | Verify deployed contract addresses, network, authorization, and contract source before use |
| Browser integrations | Untrusted page content may influence agent behavior | Treat DOM content as untrusted input; test sanitization and browser isolation against the application's threat model |

Do not describe heuristic filters as a complete defense against prompt
injection. Do not claim a security property is formally verified unless the
relevant code, tests, and audit evidence are available.

## Network details

The project has documented a Midnight Preprod integration. Testnet contracts
and endpoints can change; confirm their status before use.

- Network: Midnight Preprod (test network)
- Relayer endpoint: `https://api.rohanprotocol.network/api/v1/handshake`
- Contract address previously listed in project materials:
  `585ac0c4448257507d8ffa2a89e2aa00abd86ec9e94bdb6f553bc83e05f4dd0e`

Do not treat a testnet deployment as a production availability or security
guarantee.

## Research and citation

The Rohan preprint is available on Zenodo:

> Julian von Bordelius. “Rohan: A Stateless Zero-Knowledge Trust Gateway for
> Privacy-Preserving Agentic Workflows via Streamable HTTP.” Preprint,
> version 1.0.0, 2026. https://doi.org/10.5281/zenodo.22730240

The preprint describes the research design and reported results. It is not, by
itself, an independent security audit or a guarantee that every published
claim applies to every released package.

```bibtex
@misc{vonbordelius2026rohan,
  author       = {Julian von Bordelius},
  title        = {Rohan: A Stateless Zero-Knowledge Trust Gateway for
                  Privacy-Preserving Agentic Workflows via Streamable HTTP},
  year         = {2026},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.22730240},
  url          = {https://doi.org/10.5281/zenodo.22730240},
  note         = {Preprint, version 1.0.0}
}
```

## License

This project is distributed under the MIT License. See the repository's
license file for details.

