# How to Post the RFC on Discord

Discord has a ~2,000 character per-message limit. Post this as **3 messages in sequence** in `#developers` (or wherever Midnight governance discussions happen). Tag `@everyone` only if you have permission.

---

## Message 1: TL;DR + Hook (post first)

```
**[RFC] MIP-XXXX: Midnight Agent Identity Standard (MAIS)**

AI agents are arriving on Midnight — Midnight City runs Gemini agents, AlphaTON is building for Telegram's 1B users. But there's no standard for agent identity, reputation, or validation.

I'm proposing **MAIS** — a privacy-preserving agent identity standard using Midnight's three-tier disclosure model:

• **Identity Registry** — public or private agent registration
• **Reputation Registry** — public scores with ZK-provable private evidence  
• **Validation Registry** — open/trusted/institutional validators with anti-Sybil staking
• **Disclosure Tier Registry** — maps to Midnight's public/auditor/private tiers
• **Optional ERC-8004 bridge** — cross-chain identity for Ethereum agents

**Why this matters now:** Midnight mainnet is live, AlphaTON agents need native identity, and we should standardize before the ecosystem fragments.

Full MIP: https://github.com/mzf11125/midnight-improvement-proposals/blob/main/mips/mip-XXXX-midnight-agent-identity-standard.md
Forum post: https://forum.midnight.network/ (link TBD)
```

---

## Message 2: Key Design Decisions (post second, immediately after)

```
**Why MAIS is different from ERC-8004:**

Ethereum's ERC-8004 is fully transparent — every agent's metadata, score, and validation records are public. On Midnight, we can do better.

**Design decisions:**
• **Dual-mode identity:** public agents for marketplaces, private agents (ZK credentials) for sensitive use cases — both expose the same interface
• **Public scores + private evidence:** anyone reads a score in O(1), but the *reasons* for the score are hashed — competitors can't reverse-engineer agent strategy
• **Hybrid validators:** 3 tiers (Open/Trusted/Institutional) with NIGHT staking + slashing — anti-Sybil without central gatekeeping
• **No TEE in core:** Midnight is ZK-native; TEE is a different trust model (can be an extension MIP)
• **ERC-8004 bridge is optional:** not every Midnight agent needs EVM interoperability

**Timeline if accepted:** contracts in 12 weeks, SDK in 16, testnet in 22, mainnet ~30 weeks.
```

---

## Message 3: Feedback Questions (post third, create thread here)

```
**I need your feedback on these questions (reply in thread):**

1. Should we ship both public + private identity modes in v1, or start public-only?
2. Is public score + private evidence the right reputation balance?
3. Are 10k/50k/100k NIGHT validator stakes reasonable?
4. What should validator slashing parameters be?
5. How should regulator disclosure work — auto-disclose after timeout, or always require operator approval?
6. Midnight City team: would this work for Gemini agents? What metadata is missing?
7. AlphaTON: do Telegram agents need private mode?
8. Agent builders: what's your minimum viable identity feature?
9. Should agent metadata include an x402 payment method field?
10. Would you attend a 2-hour community workshop in ~3 weeks to review this in detail?

Also posted on: https://forum.midnight.network/ (link TBD)
GitHub PR coming soon to the MIPs repo after feedback.
```
