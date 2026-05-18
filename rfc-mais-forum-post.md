**Category**: Standards, Midnight Improvement Proposal
**Status**: Draft, looking for community feedback before formal submission

---

## TL;DR

AI agents are arriving on Midnight. Midnight City already runs Gemini agents. AlphaTON is building privacy agents for Telegram's billion users. But there is no standard way for these agents to say who they are, prove they can be trusted, or get validated by others.

This proposal defines **MAIS**, the Midnight Agent Identity Standard. It is made of four Compact contracts that let agents do the following.

Register with either a public or private identity. Build up a reputation with scores that anyone can read but whose underlying evidence stays private. Get validated by a system that mixes open validators, trusted validators, and institutional validators. Choose what data to show at each of Midnight's three disclosure tiers (public, auditor, private). And optionally link to ERC-8004 on Ethereum if cross chain identity matters to you.

Ethereum has ERC-8004 for agent identity already. But ERC-8004 lives on a transparent chain. Midnight lets us do something better. We can have trust that is verifiable without giving away how the agent operates.

---

## The Problem in More Detail

Ethereum launched ERC-8004 in January 2026. It gives AI agents an on chain identity, reputation score, and validation record on EVM chains. But because ERC-8004 sits on transparent blockchains, all of that data is public. Anyone can read any agent's metadata, score, and full validation history. That works fine for open ecosystems. It fails hard when agents operate in competitive or regulated domains.

Midnight raises the stakes. A trading agent's strategy is what makes it money. If its identity and reputation trail sit on chain for anyone to read, competitors can reverse engineer its approach. A compliance agent handling sensitive financial data cannot have its validation history visible to the entire network. A governance agent serving a DAO has voting logic that needs to stay confidential.

Midnight's zero knowledge architecture was built for exactly this. Selective disclosure means an agent can prove who it is and how trusted it is without dumping its full state onto the chain. But Midnight does not yet have a standard for doing that. Every team building agents on Midnight right now invents its own identity scheme from scratch.

### Why We Should Act Now

Three things make this standard urgent.

First, Midnight mainnet went live on 30 March 2026. Developer activity surged over 1,600 percent after the Summit in late 2025. If we do not put a standard in place right now, we could end up with ten different identity schemes by the end of the year.

Second, AlphaTON is targeting Telegram's one billion users with privacy preserving AI agents. These agents need an identity system that is native to Midnight. Their entire pitch depends on Midnight's privacy model. They cannot rely on an EVM standard that puts everything in public view.

Third, Midnight City Simulation uses Gemini agents as a live testbed. Those agents have no formal identity standard today. If they did, they would become composable with real Midnight applications instead of staying locked inside the simulation.

---

## The Proposal

MAIS defines four registries as Compact contracts.

### 1. Identity Registry

Agents can register in one of two modes.

In public mode the agent's Compact address serves as its identity. Metadata like name, capabilities, and operator address lives in public state. This suits agents that operate in open marketplaces.

In private mode the agent holds a ZK credential instead. The contract stores only a commitment to that credential in private state. The operator proves they know the credential via a ZK proof without ever revealing the identity on chain. No event gets emitted when a private agent registers. This suits agents that handle sensitive data.

Both modes expose the same interface. Contracts and SDKs work the same way regardless of which mode sits underneath.

### 2. Reputation Registry

The reputation system uses a model where the score is public but the evidence is private.

The score, a number from 0 to 100, sits in public state. Any contract can read it in a single lookup. Three tiers sit on top of that score. Basic covers 0 to 59. Proven covers 60 to 89. Institutional covers 90 to 100.

The evidence that produced the score lives as hash commitments in private state. Nobody can read the reasons behind a score unless the agent chooses to reveal them. This stops competitors from studying an agent's transaction history to reverse engineer its strategy.

An agent can also generate a ZK proof that says "my score is at least X" without giving away the exact number. That lets two parties negotiate trust at whatever threshold matters to them.

Scores use a weighted sliding window across 90 days. Recent evidence carries more weight. Old evidence decays. Scores also decay slowly if an agent stops accumulating fresh evidence.

### 3. Validation Registry

Three tiers of validators, each with anti Sybil protections.

Open validators can be anyone. They must stake 10,000 NIGHT. Their stake gets slashed if they submit bad validations.

Trusted validators must pass a Foundation audit. They stake 50,000 NIGHT.

Institutional validators must complete a ZK KYC check from an accredited provider. They stake 100,000 NIGHT.

Every validator has an accuracy score tracked on chain. When someone submits bad evidence and gets caught, their accuracy drops and their stake gets cut. If accuracy falls below a threshold, the validator loses their tier.

### 4. Disclosure Tier Registry

This maps agent data directly onto Midnight's three built in privacy tiers.

The public tier shows that the agent exists, its metadata, and its reputation score. Anyone can read these.

The auditor tier adds activity statistics and evidence history. Only addresses the agent operator has registered as auditors can see this.

The private tier holds transaction contents and the agent's strategy. Only the operator can access it.

### Optional ERC-8004 Bridge

An agent can link its Midnight identity to an ERC-8004 NFT on Ethereum if it wants cross chain interoperability. The bridge verifies that the same operator controls both identities using cross chain proofs. It also lets Midnight reputation data flow back into ERC-8004's Validation Registry on Ethereum.

The bridge is a separate contract. Core MAIS works perfectly without it.

---

## Why These Design Choices

The full rationale lives in the MIP document. Here is a short version of the key tradeoffs.

**Why dual mode identity?** A single mode fails one side. Public only forces every agent to be linked to its operator, which is unacceptable for sensitive use cases. Private only prevents open agent commerce, because you cannot build a marketplace if you cannot discover agents. Letting the ecosystem pick per use case is the right call.

**Why public scores with private evidence?** Fully public would let competitors reverse engineer agent strategies. Fully private would force a ZK proof round trip for every trust decision, which is slow. Public scores give instant reads. ZK provable private evidence gives depth when needed.

**Why hybrid validators?** Purely permissionless validation invites Sybil attacks where agents create fake validators to pump their own scores. Purely permissioned validation creates a central bottleneck. The tiered approach gives everyone an entry point through staking while offering a path to higher trust.

**Why optional ERC-8004 bridge?** Not every Midnight agent touches Ethereum. Bridges add attack surface. Making it optional keeps the core standard simpler and the attack surface smaller.

**No TEE attestation in the core standard?** Midnight is built on ZK proofs. TEE means trusting a hardware vendor, which is a different security model entirely. TEE can arrive later as an extension.

---

## Where I Need Your Feedback

I am looking for community input before submitting this as a formal MIP. Here are the questions I most want answered.

### Design Questions

1. Should we ship both identity modes in v1, or start with public only and add private mode later? Private mode adds complexity but it is also the whole reason to build on Midnight for sensitive use cases.

2. Is public score plus private evidence the right balance? Or should reputation scores be fully private and only available through ZK proofs?

3. Are 10k, 50k, and 100k NIGHT the right amounts for validator stakes? Should these numbers be adjustable through governance?

4. What should the slashing curve look like for bad validations? The current draft penalises a missed violation with minus 5 accuracy and slashed stake. A false positive costs minus 3 accuracy and half the stake.

5. When a regulator requests data from a regulated agent, should the agent auto disclose auditor tier data after a timeout, or always require the operator to approve first?

### Ecosystem Questions

6. Midnight City team, could MAIS work for your Gemini agents? What is missing from the metadata schema?

7. AlphaTON team, do Telegram agents need the private identity mode? What extra metadata fields would you want?

8. Agent builders, what is the smallest set of features you need to get started? Can you work with just public mode and reputation at launch?

9. Should agent metadata include a field for x402 payment info, like what the agent charges and how to pay it?

10. Who would run validators in the early ecosystem? Should the Midnight Foundation run reference validators at the start?

### Process Questions

11. Should I submit this as a draft PR to the MIPs repository directly, or should it go through an MPS problem statement first?

12. Would you attend a workshop to review the MAIS spec together? If enough people say yes I will set one up in about three weeks.

---

## What Success Looks Like

If adopted, the standard would deliver a few concrete things.

An Identity Registry and SDK that lets any developer register an agent on Midnight with one function call. A reputation system with on chain trust scores that agents can prove without exposing their full history. A validator ecosystem that goes from open entry to institutional grade trust. And an ERC-8004 bridge that makes Midnight agents visible in the broader AI agent world and brings Ethereum agent data back the other way.

---

## Links and References

- Full MIP document: [mip-XXXX-midnight-agent-identity-standard.md](https://github.com/mzf11125/midnight-improvement-proposals/blob/main/mips/mip-XXXX-midnight-agent-identity-standard.md)
- ERC-8004 Specification: [eips.ethereum.org/EIPS/eip-8004](https://eips.ethereum.org/EIPS/eip-8004)
- Midnight Documentation: [docs.midnight.network](https://docs.midnight.network/)
- Midnight Compact Language Reference: [docs.midnight.network/develop/reference/compact](https://docs.midnight.network/develop/reference/compact/)
- Bastion Agentic Defense: Complementary on chain security middleware for AI agents

---

## How to Participate

Reply below with feedback on any of the questions above. Open issues on the GitHub repository for specific technical concerns. Submit PRs if you want to contribute to the spec or the reference implementation. DM me if you want to co author or help build the reference implementation.

I will consolidate feedback into weekly summaries. A community workshop follows if enough people show interest. Thanks for reading. Looking forward to the discussion.

---

*Also being discussed on Midnight Discord in the developers channel.*
