<h1 align="center">Project Rohan 🛡️ (@rohan-zk/sdk)</h1>

<p align="center">
  <b>The Universal Trust Layer for AI Agents.</b><br>
  <i>Stop leaking agent data. Replace insecure REST APIs with local Zero-Knowledge Proofs.</i>
</p>

<p align="center">
  <a href="#-langchain-integration-god-mode"><img src="https://img.shields.io/badge/LangChain-Native-blue?style=for-the-badge&logo=langchain" alt="LangChain"></a>
  <a href="#architecture"><img src="https://img.shields.io/badge/Powered_by-Midnight_Blockchain-black?style=for-the-badge" alt="Midnight"></a>
  <a href="#quickstart"><img src="https://img.shields.io/badge/DX-Stripe_for_ZK-008CDD?style=for-the-badge" alt="Stripe DX"></a>
  <a href="https://github.com/rohan-zk/sdk/stargazers"><img src="https://img.shields.io/badge/GitHub-Trending_🔥-orange?style=for-the-badge" alt="Trending"></a>
</p>

> **🚨 v0.3.0 "The Gasless Update" is live:** We achieved **Stripe-DX for ZK-Proofs**. No wallets, no DUST-tokens, no cryptography PhD required. Just an API key.

---

### 🛑 The Problem: AI Agents leak data via REST
When your agent talks to another agent via API, the payload is exposed. Centralized servers see your agent's confidential logic, system prompts, and negotiation limits. 

### ⚡ The Solution: Rohan ZK-Handshakes
Rohan enables agents to negotiate confidential deals (NDAs, payments, secrets) using **Zero-Knowledge Proofs (zk-SNARKs)** on the Midnight Blockchain. 

| Before Rohan (Dangerous) ❌ | With Project Rohan (Trustless) ✅ |
| :--- | :--- |
| `Agent A` sends raw data + API Keys over the wire to execute a trade. | `Agent A` computes a **ZK-Proof locally** and settles trustlessly. |
| Server sees the agent's logic. | Server only sees cryptographic proof. |
| **Result:** Massive attack surface. | **Result:** Zero data leaks. |

---

## 📦 Install

```bash
npm install @rohan-zk/sdk

🚀 Quickstart (The Stripe-DX)

You no longer need to manage DUST-Token balances or complex cryptography. You simply provide an API Key, and our Relayer pays the gas fees and manages the chain interactions for your agent.
code TypeScript

import { RohanNode } from '@rohan-zk/sdk';

// 1. Create an agent connected to the Relayer (No Wallets required!)
const agent = new RohanNode({
  apiKey: process.env.ROHAN_API_KEY || 'rohan_sk_test',
  relayerUrl: 'http://relayer.rohan.network:4005'
});

// 2. Listen for incoming deals (Trustless Verification via Network Ledger)
agent.onHandshake((deal) => {
  console.log(`Deal confirmed on-chain! From: ${deal.from}`);
  console.log(`Commitment Hash: ${deal.commitment}`);
});

// 3. Connect to the Blockchain Tracker
await agent.start();

🔐 Send a Confidential Handshake

Prompt your agent to execute a confidential data transfer utilizing our local ZK Proof flow:
code TypeScript

const result = await agent.handshake(
  '02d7fe8b22a0142b6a958e945feae6759c8d504533a69aa3fe6a8f6f5cc761b0', // Target Contract Address
  'NDA for Project Alpha – Confidential Blueprints', // Secret data (remains offline)
  150  // Maut/Toll fee
);

console.log(result.status);            // "success"
console.log(result.txId);              // "tx_mid_892f7dbf..."

🤖 LangChain Integration (God Mode)

Turn your standard LLM into an autonomous Web3 protocol user. We provide a drop-in LangChain Tool wrapper. Your agent can decide by itself when it is appropriate to use a ZK-Handshake to protect confidential data.
code TypeScript

import { RohanSecureHandshakeTool } from '@rohan-zk/sdk/langchain';

// Inject the Tool into LangChain/LangGraph
const handshakeTool = new RohanSecureHandshakeTool(agent);
const tools = [handshakeTool, /* other system tools */];

// The LLM will now autonomously construct the ZK-Handshake payload 
// when its system prompt instructs it to securely transmit money or data.

    🎯 Want official integration into ElizaOS and CrewAI? <br>
    Star this repository ⭐️ to help us push the proposals through their governance!

🏗 Architecture
code Text

┌──────────────────────────────────────────────────────┐
│                    Your AI Agent                      │
│                                                      │
│   const agent = new RohanNode({ apiKey, relayerUrl })│
│   agent.handshake(contractRef, secret)               │
└──────────────────┬───────────────────────────────────┘
                   │
        ┌──────────▼──────────┐
        │  @rohan-zk/sdk      │
        │  (Prover Module)    │ ← Generates ZK-Proof locally
        └──────────┬──────────┘
                   │ HTTP POST (Proof Blob + API Key)
        ┌──────────▼──────────┐
        │  Rohan Relayer Node │ ← Verifies API Key & Pays DUST Fee
        └──────────┬──────────┘
                   │ Submit transaction to Midnight
        ┌──────────▼──────────┐
        │  Midnight Network   │ ← The ZK-Settlement Layer
        └─────────────────────┘

🛡 Security Pillars

    Gasless Architecture: API Keys replace vulnerable Wallet Seeds on the client side.

    Zero-Knowledge: The deal content is hashed prior to proof generation. Only mathematical assurances are broadcasted.

    Trustless Execution: Webhooks are deprecated. Incoming deals trigger strictly based on cryptographic changes on the Ledger.

👑 Become a Founding Contributor

Project Rohan is currently maintained by the core creator (@jvonb) and automated CI/CD systems (@rohan-ops). We are now opening the protocol for community governance.

The first 5 developers to merge significant PRs (especially for ElizaOS/CrewAI Python sidecars) will be invited to the @rohan-zk GitHub organization as core team members.

Check our open issues to get started!
<p align="center">
<i>License: MIT © Rohan Protocol</i>
</p>
```
