MasumiGuard – AI Agent for On-Chain Compliance & Risk Checking

MasumiGuard is an AI-powered on-chain compliance and risk analysis agent.
It helps Web3 builders, compliance teams, and regular users quickly understand:

How risky is this wallet?

Is this transaction linked to scams, mixers, or sanctioned entities?

Does this smart contract show suspicious or malicious behavior patterns?

Instead of manually reading block explorers and scanning contracts, users can just enter a wallet address / transaction hash / contract address and let the AI agent:

Fetch on-chain data

Analyze behavior & patterns

Generate a risk score + natural language explanation

Suggest next actions (e.g., safe to interact, proceed with caution, avoid etc.)

🧠 Core Use Case
Problem

Compliance and risk checks in Web3 are:

Slow

Expensive

Hard to understand for non-experts

Developers and users don’t know if:

A wallet is tied to hacks / scams / mixers

A transaction looks linked to money laundering

A contract could be a rug-pull or exploit

This leads to:

Fraud

Hacks

Loss of funds

Low trust in dApps

Our Solution

MasumiGuard uses an AI Agent that combines:

On-chain data (transactions, balances, flows, contract metadata)

Heuristics & rules (e.g., flags for mixers, blacklisted addresses, abnormal behavior)

LLM reasoning (explains in simple language what’s going on)

to provide:

✅ Risk score (e.g., 0–100 or Low / Medium / High)

✅ Why (explanations in English)

✅ What now? (recommended action)

🧩 High-Level Architecture
User (UI: web app, dashboard)
         |
         v
   MasumiGuard Backend (API)
         |
         +--> Blockchain Node / Explorer API (Cardano or others)
         |
         +--> Risk Heuristics Engine (rules, thresholds)
         |
         +--> AI Agent (LLM API: e.g. OpenAI)
                  |
                  v
           Risk Score + Explanation

✨ Features

🔍 Wallet analysis – Risk score based on transaction history, counterparties, behavior.

🔗 Transaction analysis – Flags suspicious patterns, large movements, mixer usage, etc.

📜 Smart contract analysis – Reads contract metadata + heuristics (e.g., owner permissions).

🧾 Human-readable report – AI generates an explanation a non-technical person can understand.

📊 Compliance angle – Can be extended to check sanctions lists or KYC risk signals.

🧩 Modular design – You can plug in:

Different chains (Cardano, EVM, etc.)

Different LLM providers

Custom rule sets

🚀 Getting Started
1. Prerequisites

Node.js (for backend and/or frontend)

Package manager: npm or yarn

API access to:

A blockchain explorer / indexer (e.g., Cardano explorer API)

An LLM API (e.g., OpenAI, other provider)

Git + basic terminal knowledge

2. Clone the Repository
git clone https://github.com/your-username/masumiguard.git
cd masumiguard

3. Set Environment Variables

Create a .env file in the project root (example):

# AI provider
OPENAI_API_KEY=your_openai_key_here

# Explorer / node URLs
BLOCKCHAIN_NETWORK=cardano-mainnet
CARDANO_EXPLORER_API_URL=https://...

# (Optional) Scoring settings
RISK_SCORE_MIN=0
RISK_SCORE_MAX=100


Adjust variable names to match your actual code.

4. Install Dependencies

If you have a backend (e.g., in /server):

cd server
npm install


If you have a frontend (e.g., in /client):

cd client
npm install

5. Run the App

Backend:

cd server
npm run dev      # or: npm start


Frontend:

cd client
npm run dev
# open http://localhost:3000 or whatever port

🛠 How the AI Agent is Implemented

This is the important part for judges and devs: how the AI agent actually works.

Step 1: Input from User

User enters:

Wallet address

Transaction hash

Smart contract address

In the frontend, you send this to your backend:

// Example (frontend)
await fetch("/api/analyze", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    type: "wallet", // "transaction" | "contract"
    value: "addr1q..."
  })
});

Step 2: Fetch On-Chain Data

Backend receives this request and:

Detects type: wallet / tx / contract

Calls explorer / indexer APIs, for example:

// Pseudo-code
const txHistory = await getTxHistoryFromExplorer(walletAddress);
const counterparties = extractCounterparties(txHistory);
const stats = computeStats(txHistory); // volume, frequency, age, etc.


Examples of computed stats:

Number of transactions

Age of address

Total volume in/out

Interactions with known risky tags (if you have a blacklist / flag list)

Usage of mixers / bridges / DeFi protocols

Step 3: Pre-Risk Heuristics (Rule Engine)

Before using the LLM, we can use basic logic:

let baseScore = 50;

if (stats.interactsWithFlaggedAddresses) baseScore += 30;
if (stats.veryNew && stats.highVolume) baseScore += 10;
if (stats.manySmallRandomTxs) baseScore += 10;

baseScore = Math.min(100, Math.max(0, baseScore));


This gives a numerical score that the AI can refine and explain.

Step 4: Call the AI Agent (LLM)

Now we send the structured data + base score to an LLM:

import OpenAI from "openai";

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const prompt = `
You are a blockchain risk and compliance assistant.

Given:
- Address type: wallet
- Base risk score: ${baseScore} (0 = safe, 100 = very risky)
- Statistics: ${JSON.stringify(stats, null, 2)}

Tasks:
1. Decide the final risk score between 0 and 100.
2. Classify risk level as: "Low", "Medium", or "High".
3. Give a clear explanation (3–6 bullet points).
4. Give a recommendation: "Safe to interact", "Proceed with caution", or "Avoid".

Respond in JSON only with keys: score, level, explanation, recommendation.
`;

const completion = await client.chat.completions.create({
  model: "gpt-4.1-mini",
  messages: [{ role: "user", content: prompt }],
  response_format: { type: "json_object" }
});

const aiResult = JSON.parse(completion.choices[0].message.content);


Now you have:

{
  "score": 78,
  "level": "High",
  "explanation": [
    "Multiple interactions with previously flagged addresses.",
    "Very new wallet with sudden large inflows.",
    "Transaction pattern resembles mixer usage."
  ],
  "recommendation": "Avoid"
}

Step 5: Return to Frontend

Backend returns a JSON response:

return res.json({
  address: walletAddress,
  score: aiResult.score,
  level: aiResult.level,
  explanation: aiResult.explanation,
  recommendation: aiResult.recommendation,
  rawData: stats   // optional: show more technical info
});

🎨 Frontend / UI Ideas

In the frontend, you can show:

Big risk score (speedometer or progress circle)

Color code:

Green = Low

Yellow = Medium

Red = High

Key explanation bullets

Recommendation as a clear badge:

✅ Safe to interact

⚠️ Proceed with caution

⛔ Avoid

You can also add:

History of analyses

Comparison between multiple wallets / transactions

“Why is this risky?” expandable section

🧪 Example API Contract

POST /api/analyze

Request body:

{
  "type": "wallet",
  "value": "addr1q..."
}


Response:

{
  "type": "wallet",
  "value": "addr1q...",
  "score": 27,
  "level": "Low",
  "explanation": [
    "Wallet is older than 180 days.",
    "Mostly interacts with a few known exchanges.",
    "No known links to flagged entities."
  ],
  "recommendation": "Safe to interact",
  "rawData": { "...": "..." }
}

🔌 How to Integrate in Other dApps

You can:

Expose /api/analyze as a REST API and let other dApps call it.

Show a “Check Risk on MasumiGuard” button inside:

Wallets

NFT marketplaces

DeFi dashboards

Build browser extension / widget that calls this backend API.

🧱 Tech Stack (Example – customize for your project)

Frontend: React / Next.js / Tailwind

Backend: Node.js / Express / Fastify

AI Agent: OpenAI API (or other LLM provider)

Blockchain Data: Cardano explorer / indexer API (or other chain)

📌 Future Improvements

On-chain reputation system

On-chain storage of risk attestations

Integration with multiple chains

Support for more detailed compliance rules (jurisdiction-specific)
