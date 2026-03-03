# ⚠️ Security Analysis Report — "Fake Job Demo" Manicro Demo Repository

**This is an active malware. Do not run this code.**

## CRITICAL SECURITY ALERT - EMBEDDED BACKDOOR DETECTED

**Source**: Linkedin job offer

**Sender** https://www.linkedin.com/in/andrii-shan-0335b2368/

**Sender's notion** https://www.notion.so/Manicro-V-Demo-Review-2ee561957fc680798cdaf8479bf3bccb

**Original repository**: https://bitbucket.org/tre555/manicro-demo-version/src/main/

**Analysis Date**: 2026-03-01

**Status**: ⚠️ HIGH-SEVERITY EXPLOIT — Do not run npm install or any project commands



Hidden at the end of `tailwind.config.js`, pushed roughly 2,000 characters off-screen with whitespace - is a heavily obfuscated, multi-layer JavaScript backdoor. The moment the victim runs `npm start`, the malicious code silently executes inside Node.js: it contacts a hardcoded C2 server to fingerprint the victim's machine (hostname, OS, network info, external IP), then downloads and installs a Stage 2 payload into `~/.vscode/` and executes it with full Node.js privileges, all with zero console output. The implant then calls home every 10 minutes and self-updates, making it persistent and stealthy. Based on known campaigns of this type, the final payload is typically a **remote access trojan (RAT) and credential harvester** targeting browser data, crypto wallets, SSH keys, and developer credentials.

<details>
<summary>Original description</summary>


![](/public/favicon.ico)

[![Styled With Prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://prettier.io/)

# Manicro Project

## 🌐 Overview

This repository represents the **initial foundation** of the Manicro decentralized trading and prediction platform.  
It includes the **core logic, components, and architecture** that mark the starting point of our next major development phase.

The current version was built to demonstrate the essential mechanics — such as trading, staking, and basic gamification — but it also exposes several architectural and performance limitations.  
Our goal now is to **analyze, optimize, and evolve** this base system into a fully scalable, modular, and production-ready multi-chain platform.

To strengthen this transformation, we’re introducing **European engineers and developers** to lead key areas of infrastructure, system design, and scalability — addressing weak points and preparing the platform for global expansion.

## ⚙️ Getting Started

### Prerequisites
Before running the project, ensure you have the following installed:
- **Visual Studio Code**
- **Node.js** (v20 or higher)
- **npm**
- **Git**

### Install Dependencies

```bash
1. Front-end    `npm install`
2. Back-end    `cd backend & npm install`
```

### Run

```bash
cd ..
npm start
```

## 🚀 Project Goals

- Refine the core foundation for cross-chain trading and prediction systems
- Redesign architecture for modularity, performance, and scalability
- Implement enhanced user engagement features (XP, leaderboards, staking)
- Transition from hybrid Web2/Web3 infrastructure toward full decentralization
- Prepare for the next design and development phase led by the EU engineering team

## 🧩 Notes

- This project is our internal starting build, not a client reference.
- Some modules, services, and integrations may still be incomplete or outdated.
- The main objective is to understand current feature flows and pain points before implementing structural improvements.
- Please document any issues, bugs, or improvement ideas — these will guide the technical call and refactoring roadmap.

## 📅 Next Steps

After setting up and exploring the application:
1. Review the main features and workflows
2. Identify bottlenecks, UX issues, and improvement opportunities
3. Share your feedback focusing on:
    - System design and data flow
    - Performance and modularity
    - User experience clarity
4. A follow-up CTO session will be arranged to discuss:
    - Technical direction and architecture improvements
    - New design implementation
    - Role-specific contributions and priorities

## 🧠 Summary

This repository forms the base layer of Manicro’s new decentralized ecosystem — merging DeFi trading, staking, and prediction gaming under one unified experience.

Your review, insights, and technical feedback will directly influence the next evolution of the platform — guiding our transition from this foundational build to a robust, scalable, and globally distributed system.

</details>
