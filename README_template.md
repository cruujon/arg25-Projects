# Heritage Inheritance Protocol  
*A tool for securely passing cultural assets or secret knowledge across generations.*

Welcome to Invisible Garden – ARG25.  
This README documents our project and weekly progress.

---

# Project Title  
**Heritage Inheritance Protocol**

Short concept: A tool that allows people who wish to preserve cultural assets or secret knowledge to securely and permanently pass them down to others across generations.

---

# Team  
**Team/Individual Name:**  
**GitHub Handles:**  
**Devfolio Handles:**  
Roles (optional): Smart Contract / Product / Design / Infra

---

# Project Description

## Problem / Motivation

Currently, those who want to pass down their knowledge or skills have no choice but to share secret information directly — either orally or on paper.  
There is no verifiable way to record *who passed it to whom*, which makes extinction of such knowledge a real risk.

Traditional craftsmanship is disappearing globally not only because knowledge is lost, but because the act of transmission is invisible to institutions and future generations.  
Anchoring these successions on-chain — with auditable records and public funding mechanisms — makes cultural inheritance *visible* and *preservable*.

Additional risks:

- Even though such cultural assets are meaningful and important to preserve, it is often unclear who contributed to their preservation.
- Historically sensitive knowledge is vulnerable to censorship by institutions or governments. Oral traditions can be altered, sanitized, or erased.

## Solution

- Record *who (wallet)* passed knowledge to *whom (wallet)* on-chain, preserving lineage and provenance.  
- Encrypt content **client-side** so the knowledge itself remains private.  
- Store encrypted data on IPFS; only its hash (CID) is referenced on-chain.  
- Only the designated successor's wallet can derive the correct key to decrypt the content.  
- This enables preservation of private cultural assets without forcing public disclosure.

## Target Users

- Individuals wanting to pass down secret or valuable knowledge *privately*.
- Examples:
  - A restaurant owner with a secret recipe but no successor.
  - Craftsmen with unique techniques that cannot be publicized.
  - Oral storytelling traditions and local cultural narratives.

---

# Key Features

- **Client-side encrypted inheritance**
  - The owner selects a PDF and a successor wallet address.
  - File is encrypted entirely in the browser using **AES-256-GCM**.
  - The AES key is derived from the successor’s Ethereum address via **PBKDF2 (100,000 iterations)**.

- **Successor-only decryption**
  - Only the wallet that matches the successor address can regenerate the AES key.

- **IPFS-based decentralized storage**
  - Only encrypted blobs are uploaded.
  - Blob format: `[IV (12 bytes)][Encrypted Data]`.

- **On-chain lineage**
  - Immutable record of: owner → successor, `ipfsHash`, `fileName`, `fileSize`, `timestamp`, and status flags.
  - Creates verifiable historical context for each inheritance.

- **End-to-end flow**
  - Originator: encrypt → upload → register on-chain  
  - Successor: verify → fetch → decrypt → download

---

# Architecture Overview

## System Components

- **Frontend (Next.js + Wagmi + viem)**  
  Handles:
  - file encryption/decryption (Web Crypto API)
  - IPFS upload/download (via API route or direct)
  - contract calls
  - lineage display

- **Blockchain (Arbitrum Sepolia / Solidity)**  
  Responsible for:
  - storing inheritance metadata
  - verifying successor identity (`msg.sender`)
  - preserving lineage

- **Storage: IPFS**
  - Stores encrypted blobs only.
  - Contract stores the `ipfsHash` as reference.

---

## Inheritance Flow Diagram  
(*ASCII diagram — GitHub compatible*)


## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        INHERITANCE FLOW                          │
└─────────────────────────────────────────────────────────────────┘

1️⃣  OWNER CREATES INHERITANCE
   ┌──────────────┐
   │ Select PDF   │
   │ + Successor  │
   └──────┬───────┘
          │
          ▼
   ┌─────────────────────────┐
   │ ENCRYPT CLIENT-SIDE     │
   │ • Use successor address │
   │ • AES-256-GCM           │
   │ • Generate random IV    │
   └──────┬──────────────────┘
          │
          ▼
   ┌─────────────────────────┐
   │ UPLOAD TO IPFS          │
   │ • Encrypted file only   │
   │ • Returns IPFS hash     │
   └──────┬──────────────────┘
          │
          ▼
   ┌─────────────────────────┐
   │ STORE ON BLOCKCHAIN     │
   │ • IPFS hash             │
   │ • Successor address     │
   │ • Metadata              │
   └─────────────────────────┘

2️⃣  SUCCESSOR CLAIMS INHERITANCE
   ┌──────────────────────────┐
   │ Connect Wallet           │
   │ (Must be successor)      │
   └──────┬───────────────────┘
          │
          ▼
   ┌─────────────────────────┐
   │ VERIFY ON BLOCKCHAIN    │
   │ • Check successor match │
   │ • Verify not claimed    │
   └──────┬──────────────────┘
          │
          ▼
   ┌─────────────────────────┐
   │ FETCH FROM IPFS         │
   │ • Download encrypted    │
   │ • Extract IV + data     │
   └──────┬──────────────────┘
          │
          ▼
   ┌─────────────────────────┐
   │ DECRYPT CLIENT-SIDE     │
   │ • Use their address     │
   │ • Decrypt with key      │
   │ • Download PDF          │
   └─────────────────────────┘
```


```
---

# Encryption & Decryption Flow (MVP)

## 1. Owner (Originator)

1. Select PDF + successor wallet address.  
2. Derive AES key with PBKDF2(successorAddress, 100k iterations).  
3. Encrypt file using AES-256-GCM (with random 12-byte IV).  
4. Create blob: `[IV][ciphertext]`.  
5. Upload encrypted blob to IPFS via API route or client-side upload.  
6. Call `createInheritance(successor, ipfsHash, tag, fileName, fileSize)`.

## 2. Successor (Receiver)

1. Connect wallet.  
2. Contract verifies:
   - caller == successor  
   - inheritance is active & unclaimed  
3. Fetch encrypted blob from IPFS.  
4. Derive AES key from successor’s address (PBKDF2).  
5. Decrypt and download PDF.  
6. Optionally call `claimInheritance(id)` to mark as received.

---

# Security Properties (MVP)

- Files are encrypted **before upload** (E2E).  
- Only successor wallet can derive the correct key.  
- No keys stored on-chain, off-chain, or in IPFS.  
- IPFS blobs are public but unreadable.  
- On-chain lineage is tamper-proof.

Security limitations:

- If successor wallet is compromised, the encrypted file can be decrypted.  
- No key rotation mechanism yet.  
- Browser-based crypto requires trustworthy hosting.

---

# Tech Stack

- **Blockchain:** Arbitrum Sepolia  
- **Smart Contracts:** Solidity  
- **Frontend:** Next.js 14, TypeScript, Wagmi, viem, shadcn/ui  
- **Storage:** IPFS  
- **Crypto:** Web Crypto API (AES-256-GCM, PBKDF2)  
- **Tooling:** pnpm, dotenv, eslint/prettier  

---

# Contract Addresses

```txt
Contract: Inheritance
Address: (fill after deployment)
Network: Arbitrum Sepolia


Demo

Repository:
https://github.com/Heirloom-Inheritance-Protocol

Live Demo:
https://heirloom-inheritance-protocol.vercel.app/dashboard

Slides:
https://www.figma.com/make/rSGqrMpI7cr1QmmQGiirqD/Create-Presentation-Material

Objectives

By the end of ARG25, the following were achieved:

On-chain inheritance event (owner → successor).

Successful encrypted file upload to IPFS.

Successor-only decryption via connected wallet.

UI for creation, verification, claim, and lineage display.

Weekly Progress
Week 1

Goals: Team forming & idea exploration

Summary: Teamed up with @DaroMacs, @masaun

Week 2

Goals: Finalize architecture & tech stack

Summary:

Scaffolded frontend

Wallet integration + lineage prototype

Designed encryption/IPFS/contract flow

Added architecture documentation

Week 3

Goals: Complete MVP

Summary:

Contract deployed

IPFS integration live

End-to-end inheritance (encrypt → upload → register → decrypt)

Final Wrap-Up

MVP fully functional

Contract + frontend deployed

Full E2E “inheritance → claim → decrypt” demo works

Limitations identified for future work (key rotation, stronger crypto)

Learnings

Verified feasibility of a minimal inheritance protocol with client-side crypto + IPFS + EVM.

Identified security limitations of address-based key derivation.

Learned to translate abstract cultural preservation concepts into concrete dApp UX and cryptographic flows.

Next Steps
Short Term

Mainnet / multi-L2 deployment

Improve encryption model (e.g., move from PBKDF2 to ECDH in next revision)

Integrate EAS for composable on-chain lineage

Medium Term

AI features for cultural importance scoring & tag generation

Matching successors and inheritances

Funding mechanisms via Gitcoin stack & partnerships with local governments
