# [RFC] MIP-XXXX: Midnight Agent Identity Standard (MAIS)

**Status**: Draft — seeking community feedback  
**Category**: Standards  
**Author**: Zidan (@mzf11125)  
**Date**: 2026-05-18  
**Repository**: https://github.com/midnightntwrk/midnight-improvement-proposals

---

## TL;DR

We need a standard way for AI agents to identify themselves, build reputation, and be validated on Midnight. This proposal defines **MAIS** — a privacy-preserving identity standard that lets agents operate openly or privately, build verifiable trust, and optionally link to ERC-8004 on Ethereum. Public identity + ZK-provable reputation + three disclosure tiers (public/auditor/private). Looking for feedback from agent builders, the Midnight City team, AlphaTON, and anyone deploying agents on Midnight.

---

## The Problem

AI agents are arriving on Midnight. Midnight City runs Google Gemini agents. AlphaTON is building privacy-preserving agents for Telegram's billion users. But there's no standard for:

- **Who is this agent?** — each simulator and project invents its own ad-hoc identity
- **Can I trust this agent?** — no reputation system exists to verify agent behavior
- **Did someone validate this agent?** — no way to know if an agent's code has been audited

Ethereum has ERC-8004 for this, but it's fully transparent. Every agent's metadata, reputation, and validation records are public. On Midnight, we can do better: verifiable trust **without** exposing agent strategies.

## The Proposal

MAIS defines four Compact contracts as a standard:

### 1. Identity Registry
Agents register with metadata (name, capabilities, operator). Two modes:
- **Public mode**: Contract address = identity. Good for marketplaces.
- **Private mode**: ZK credential = identity. Identity is provable without being public.

### 2. Reputation Registry
- **Public score** (0-100) + tier (Basic/Proven/Institutional) — quick to read, O(1)
- **Private evidence** — the *reasons* for the score are committed as hashes, not readable
- **ZK-prove**: "my score is ≥ 60" without revealing the exact score

### 3. Validation Registry
Three validator tiers with anti-Sybil mechanisms:
- **Open** (anyone, 10k NIGHT stake)
- **Trusted** (audited, 50k NIGHT stake)  
- **Institutional** (KYC'd, 100k NIGHT stake)

Validators add evidence. Bad validations → slashing.

### 4. Disclosure Tier Registry
Maps directly to Midnight's three-tier privacy model:

| Data | Public | Auditor | Private |
|------|--------|---------|---------|
| Agent exists? | ✓ | ✓ | ✓ |
| Reputation score | ✓ | ✓ | ✓ |
| Evidence history | — | ✓ | — |
| Transaction content | — | — | ✓ |
| Strategy/config | — | — | ✓ |

### Optional: ERC-8004 Bridge
Agents can optionally link their Midnight identity to an ERC-8004 NFT on Ethereum for cross-chain interoperability.

## Why This Matters Now

- **Midnight mainnet went live March 30, 2026.** Developer activity surged 1,617% post-Summit. If we don't establish a standard now, we'll have 10 incompatible agent identity schemes by next year.

- **AlphaTON + Telegram agents** need a native Midnight identity system. They can't use ERC-8004 — their value prop is Midnight privacy.

- **Midnight City** already has AI agents but no standard identity. MAIS would make those agents composable with real applications.

## Design Principles

1. **Privacy-first** — verifiable without revealing private data
2. **Disclosure-graded** — different stakeholders see different levels
3. **Composable** — other contracts (x402, Bastion, governance) can consume MAIS data
4. **Bridge-compatible** — optional ERC-8004 linking without compromising privacy
5. **Incremental** — public mode works with one call; private mode is additive

## Where I Need Feedback

### Critical Questions

1. **Dual-mode identity**: Does the community want both public and private agents, or should we start with one mode and add the other later? Private mode adds ZK commitment complexity.

2. **Private score vs public score**: The proposal says public score + private evidence. Should reputation scores themselves be private (only ZK-provable)? Public scores are faster for quick trust decisions but expose more information.

3. **TEE attestation**: Should TEE attestation be a first-class evidence type, or should it be a separate extension MIP? TEE doesn't fit Midnight's ZK-native philosophy perfectly.

4. **Validator staking amounts**: 10k/50k/100k NIGHT — are these reasonable? What should the slashing parameters be?

5. **Regulator disclosure flow**: How should regulator access work? Current proposal: ZK-KYC proof of authority + operator notification + auto-disclosure after timeout. Is this the right model?

### Ecosystem Questions

6. **Midnight City team**: Would MAIS identity be adoptable for Midnight City's Gemini agents? What's missing?

7. **AlphaTON team**: Do Telegram agents need private mode identity? What metadata fields would you need?

8. **Agent builders**: What's your minimum viable identity feature? Can you work with just public mode + reputation to start?

9. **x402 integration**: Should the standard include a payment method field in agent metadata (what the agent charges and how)?

10. **Validator ecosystem**: Who would run validators? Should the foundation run reference validators initially?

## Timeline (If Accepted)

| Phase | What | When |
|-------|------|------|
| 1 | Identity Registry (Compact contracts) | 4 weeks |
| 2 | Reputation Registry + Disclosure Tiers | 8 weeks |
| 3 | Validation Registry | 12 weeks |
| 4 | TypeScript SDK + CLI tools | 16 weeks |
| 5 | ERC-8004 Bridge | 18 weeks |
| 6 | Testnet deployment | 22 weeks |
| 7 | Security audit | 26 weeks |
| 8 | Mainnet launch | 30 weeks |

## Call to Action

Read the full MIP: https://github.com/midnightntwrk/midnight-improvement-proposals/blob/main/mips/mip-XXXX-midnight-agent-identity-standard.md

**I'm looking for:**
- Technical review of the Compact contract design
- Feedback on the reputation model (public scores? private evidence?)
- Validator tier structure and stake amounts
- Interest from agent builders who would adopt this
- Anyone who wants to contribute to the reference implementation

**Discussion format**: Reply in this thread. I'll consolidate feedback in weekly summaries and schedule a community workshop in ~3 weeks if there's enough interest.

---

## FAQ (Preemptive)

**Q: Why not just use ERC-8004 for everything?**  
A: ERC-8004 is on transparent EVM chains. Midnight agents need privacy — a trading agent's reputation shouldn't reveal its strategy. MAIS is designed for Midnight's ZK architecture.

**Q: Can existing ERC-8004 agents work on Midnight with MAIS?**  
A: Yes — the optional ERC-8004 bridge allows cross-chain identity linking. An agent can have both an ERC-8004 identity and a MAIS identity.

**Q: What if I just want a simple agent identity?**  
A: Public mode registration is one function call. Skip reputation, validation, and bridge — they're all additive.

**Q: When can I build with this?**  
A: The proposal is in Draft. If accepted, the reference implementation starts immediately. You can experiment with the design concepts now.

---

*Cross-posted to: Midnight Discord (#developers, #governance), Midnight Forum (TBD), GitHub MIPs repository*
