# [RFC] MIP-XXXX: Midnight Agent Identity Standard (MAIS)

**Category**: Standards — Midnight Improvement Proposal  
**Status**: Draft — seeking community feedback before formal submission  
**Author**: Zidan (@mzf11125)  
**Date**: 18 May 2026  

---

## TL;DR

AI agents are coming to Midnight. Midnight City runs Gemini agents. AlphaTON is building privacy-preserving agents for Telegram. But there is no standard for agent identity, reputation, or validation on Midnight.

This proposal defines **MAIS** — the Midnight Agent Identity Standard. Four Compact contracts that let agents:

- **Register** with public or private identity
- **Build reputation** with public scores backed by private evidence
- **Be validated** by a hybrid system of open, trusted, and institutional validators
- **Disclose selectively** across Midnight's public/auditor/private tiers
- **Optionally bridge** to ERC-8004 on Ethereum for cross-chain interoperability

Ethereum has ERC-8004 for this, but it is fully transparent. Midnight lets us do better — verifiable trust **without** exposing agent strategies.

---

## The Problem in Detail

Ethereum launched ERC-8004 ("AI Agent Identity, Reputation, and Validation Registries") in January 2026. It provides on-chain identity for AI agents on EVM chains. But ERC-8004 is designed for transparent blockchains — every agent's metadata, reputation score, and validation record is publicly readable. This works for open ecosystems but fails for competitive or regulated domains.

**On Midnight, the stakes are different.** A trading agent's strategy is its competitive moat — exposing its identity and reputation history on-chain would let competitors reverse-engineer its approach. A compliance agent handling sensitive financial data cannot have its full validation trail visible to the network. A governance agent's voting logic is sensitive to the DAO it serves.

Midnight's zero-knowledge architecture solves this by design. Selective disclosure means an agent can prove its identity and reputation without revealing its full state. But Midnight currently has **no standard for doing this**. Every project building agents on Midnight (Midnight City, AlphaTON, independent builders) invents its own ad-hoc identity scheme.

### Why Act Now

Three developments make this standard urgent:

1. **Midnight mainnet went live 30 March 2026.** Developer activity surged 1,617% post-Summit. If we do not establish a standard now, we will have ten incompatible agent identity schemes by end of year.

2. **AlphaTON targets Telegram's one billion users** with privacy-preserving AI agents. These agents need a native Midnight identity system — their entire value proposition is Midnight's privacy guarantees. They cannot depend on an EVM-transparent standard.

3. **Midnight City Simulation** uses Gemini-powered agents as a live public testbed. These agents currently have no formal identity standard. A standard would make Midnight City agents composable and interoperable with real-world Midnight applications.

---

## The Proposal: MAIS Four-Registry Architecture

MAIS defines four registries as Compact contracts:

### 1. Identity Registry

| Feature | Public Mode | Private Mode |
|---------|-------------|--------------|
| Identity representation | Compact address | ZK credential (commitment) |
| Registration event | Public event emitted | No event (private) |
| Metadata | Public state | Private state |
| Use case | Open marketplaces, DAOs | Institutional trading, sensitive apps |

Common interface regardless of mode: `exists()`, `getMetadata()`, `proveIdentity()`, `updateMetadata()`, `deactivate()`.

### 2. Reputation Registry

**Public score, private evidence** model:

- **Score (0-100)** stored in public state — any contract reads it in O(1)
- **Evidence commitments** stored in private state — the *reasons* for the score are hashed, not readable
- **ZK-prove**: agent can generate a proof "my score ≥ 60" without revealing the exact value
- **Three tiers**: Basic (0-59), Proven (60-89), Institutional (90-100)
- **Weighted decay**: recent evidence carries more weight; scores decay without fresh evidence

### 3. Validation Registry

Three validator tiers with anti-Sybil mechanisms:

| Tier | Requirement | Stake | Slashable |
|------|-------------|-------|-----------|
| Open | Stake NIGHT | 10,000 | Yes |
| Trusted | Foundation audit | 50,000 | Yes |
| Institutional | ZK-KYC proof | 100,000 | Yes |

All validators have accuracy scores tracked on-chain. Bad validations → slashing. Low accuracy → demotion.

### 4. Disclosure Tier Registry

Maps agent data directly to Midnight's native three-tier privacy model:

| Data Category | Public | Auditor | Private |
|--------------|--------|---------|---------|
| Agent existence | ✓ | ✓ | ✓ |
| Reputation score | ✓ | ✓ | ✓ |
| Metadata | ✓ | ✓ | ✓ |
| Activity stats | — | ✓ | ✓ |
| Evidence history | — | ✓ | — |
| Transaction content | — | — | ✓ |
| Strategy/configuration | — | — | ✓ |

### Optional: ERC-8004 Bridge

Agents can optionally link their Midnight identity to an ERC-8004 NFT on Ethereum. The bridge:
- Verifies cross-chain proofs of identity control on both chains
- Enables MAIS reputation data to be written to ERC-8004's Validation Registry
- Is a separate, optional contract — core MAIS functions without it

---

## Why These Design Choices?

This forum post is the right place for deeper discussion of the rationale. I will summarise the key tradeoffs here; the full rationale is in the MIP document.

**Why dual-mode identity (public + private)?** Monolithic models fail for one side. Public-only forces every agent to be linkable to its operator — unacceptable for sensitive applications. Private-only prevents open agent commerce — you cannot build a marketplace without discoverability. Dual-mode lets the ecosystem choose per use case.

**Why public scores + private evidence?** Fully public (like ERC-8004) would let competitors reverse-engineer agent strategies from reputation history. Fully private would prevent quick trust decisions — every transaction would require a ZK proof round-trip. Public scores with ZK-provable private evidence is the balanced middle ground.

**Why hybrid validators?** Pure permissionless validation is vulnerable to Sybil attacks. Pure permissioned requires centralised governance. The tiered approach (open/trusted/institutional) provides permissionless entry with anti-Sybil staking and an upgrade path.

**Why optional ERC-8004 bridge?** Not all Midnight agents need EVM interoperability. Cross-chain bridges add attack surface. Making it optional keeps the core standard simpler and safer.

**No TEE attestation in core standard?** Midnight is ZK-native. TEE is a separate trust model that depends on trust in hardware vendors. It can be an optional extension MIP, but does not belong in the core identity standard.

---

## Where I Need Your Feedback

I am looking for community input before formalising this as a MIP submission. Here are the questions I would most like answered:

### Design Questions

1. **Dual-mode identity**: Should we ship both modes in v1, or start with public-only and add private mode later? Private mode adds ZK commitment complexity but is the entire point of building on Midnight for sensitive use cases.

2. **Reputation privacy**: Is public score + private evidence the right balance? Should reputation scores themselves be fully private (only ZK-provable)?

3. **Validator stake amounts**: 10,000 / 50,000 / 100,000 NIGHT — are these reasonable thresholds? Should they be governance-adjustable?

4. **Validator slashing parameters**: What is the right penalty curve for bad validations? Current proposal: missed violation = -5 accuracy, slashed stake. False positive = -3 accuracy, half slashed stake.

5. **Regulator disclosure flow**: Should regulated agents auto-disclose auditor-view data to regulators after a timeout, or always require operator approval?

### Ecosystem Questions

6. **Midnight City team**: Can MAIS be adopted for Midnight City's Gemini agents? What is missing from the metadata schema?

7. **AlphaTON / Telegram**: Do Telegram agents need private mode identity? What additional metadata fields would you need?

8. **Agent builders**: What is your minimum viable identity feature? Can you work with just public mode + reputation to start?

9. **x402 integration**: Should agent metadata include a payment method field (what the agent charges, how to pay)?

10. **Early validators**: Who would run validators in the early ecosystem? Should the Midnight Foundation run reference validators initially?

### Process Questions

11. **MIP numbering**: I am submitting this as a draft PR to the MIPs repository. Is this the right venue, or should it go through an MPS problem statement first?

12. **Workshop interest**: Would you attend a 2-hour community workshop to review the MAIS specification in detail? If there is sufficient interest I will schedule one in ~3 weeks.

---

## What Success Looks Like

If adopted, the standard would deliver:

- **Identity Registry + SDK**: Any developer can register an agent on Midnight with one function call
- **Reputation system**: On-chain, verifiable trust scores that agents can prove without revealing their full history
- **Validator ecosystem**: A credible path from open validators to institutional-grade validation
- **ERC-8004 bridge**: Midnight agents visible in the broader AI agent ecosystem, and vice versa

---

## Links and References

- **Full MIP document**: [mip-XXXX-midnight-agent-identity-standard.md](https://github.com/mzf11125/midnight-improvement-proposals/blob/main/mips/mip-XXXX-midnight-agent-identity-standard.md)
- **ERC-8004 Specification**: [eips.ethereum.org/EIPS/eip-8004](https://eips.ethereum.org/EIPS/eip-8004)
- **Midnight Documentation**: [docs.midnight.network](https://docs.midnight.network/)
- **Midnight Compact Language Reference**: [docs.midnight.network/develop/reference/compact](https://docs.midnight.network/develop/reference/compact/)
- **Bastion Agentic Defense**: Complementary on-chain security middleware for AI agents

---

## How to Participate

- **Reply below** with feedback on any of the questions above
- **Open issues** on the GitHub repository for specific technical concerns
- **Submit PRs** if you want to contribute to the specification or reference implementation
- **DM me** if you want to co-author or contribute to the reference implementation

I will consolidate feedback into weekly summaries and schedule a community workshop if there is sufficient interest. Thank you for reading — looking forward to the discussion.

---

*This post is also being discussed on [Midnight Discord](https://discord.com/invite/midnightnetwork) (#developers channel).*
