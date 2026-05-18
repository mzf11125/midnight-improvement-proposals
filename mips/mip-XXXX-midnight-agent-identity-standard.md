---
MIP: X
Title: Midnight Agent Identity Standard (MAIS)
Authors:
  - Zidan (mzf11125)
Status: Draft
Category: Standards
Created: 2026-05-18
Requires: none
Replaces: none
---

<!--
 Copyright Midnight Foundation

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

## Abstract

This MIP proposes the Midnight Agent Identity Standard (MAIS)  --  a standardized identity, reputation, and validation framework for autonomous AI agents operating on Midnight Network. As agentic AI proliferates across DeFi, governance, and privacy-preserving applications, Midnight faces a new class of on-chain entities  --  autonomous agents  --  that need verifiable identity without sacrificing the privacy guarantees that define the Midnight network.

MAIS provides a three-tier disclosure identity model (public, auditor, and private view) mapped directly to Midnight's existing privacy architecture. It defines a Compact contract interface for agent registration, reputation accumulation, and third-party validation, with ZK-provable reputation claims. The standard includes a dual-mode identity system  --  contract-address-based for public agents and ZK-credential-based for private agents  --  and an optional ERC-8004 cross-chain identity bridge for interoperability with the Ethereum agent identity ecosystem.

## Motivation

### The Gap

ERC-8004 ("AI Agent Identity, Reputation, and Validation Registries") launched on Ethereum mainnet in January 2026. It provides agent identity and reputation registries on EVM chains, but operates on transparent blockchains where all agent metadata, reputation scores, and validation records are publicly readable. This creates a fundamental tension for agents operating in competitive or strategic domains  --  a trading agent's identity and reputation are visible to every competitor who can reverse-engineer its strategy from transaction patterns.

Midnight's zero-knowledge architecture solves this tension by design. Selective disclosure means an agent can prove its identity and reputation to a counterparty without revealing its full transaction history to the network. However, Midnight currently lacks a standardized interface for agents to do this  --  there is no MIP defining what an agent identity _means_ on Midnight, how reputation is accumulated and proven, or how validators can vouch for agent behavior without exposing private transaction data.

### Why This Matters Now

Three converging developments make this standard urgent:

1. **Midnight mainnet launched March 30, 2026.** The network is live with Google Cloud as a validator. Developer activity surged 1,617% post-Summit in late 2025. The window to establish a foundational identity standard  --  before a fragmented ecosystem of incompatible agent identity implementations emerges  --  is open right now.

2. **AlphaTON partnership targets Telegram's one billion users** with privacy-preserving AI agents. These agents need a native Midnight identity system  --  they cannot depend on EVM-based ERC-8004 for identity when their core value proposition is Midnight's privacy guarantees.

3. **Midnight City Simulation** uses Google Gemini-powered agents as a live public testbed. These agents have no formal identity standard  --  each simulator implements its own ad-hoc agent identity scheme. A standard would make Midnight City agents composable and interoperable with real-world Midnight applications.

### Design Principles

MAIS is designed around five principles:

- **Privacy-first**: Identity and reputation claims are verifiable without revealing the underlying private data. This is not an adaptation of a transparent-chain standard; it is designed for Midnight's ZK architecture from the start.
- **Disclosure-graded**: Different stakeholders need different levels of visibility. A regulator needs auditor-view access; a counterparty needs reputation-proof verification; the operator needs full private access. MAIS maps identity data to Midnight's three-tier disclosure model.
- **Composable**: Other Midnight standards (x402 for payments, Bastion for security enforcement, governance contracts) should be able to consume MAIS identity and reputation data as standard inputs.
- **Ecosystem-bridging**: MAIS should interoperate with the broader AI agent identity ecosystem  --  specifically ERC-8004  --  without compromising Midnight's privacy guarantees. Cross-chain identity linking should be optional, not mandatory.
- **Incremental adoption**: The simplest mode (public identity via Compact address) should work with one contract call. Advanced modes (ZK credentials, auditor-view, cross-chain bridging) should be additive without breaking the base interface.

## Specification

### 1. Core Concepts

MAIS defines four registries, each implemented as a Compact contract:

| Registry | Responsibility | Disclosure Default |
|----------|---------------|-------------------|
| **Identity Registry** | Agent registration, metadata, operator binding | Public (basic), Private (advanced) |
| **Reputation Registry** | Accumulated trust scores from historical behavior | Public score, private evidence |
| **Validation Registry** | Third-party attestations of agent integrity | Auditor-view |
| **Disclosure Tier Registry** | Maps each agent's data to public/auditor/private tiers | Per-agent configurable |

An agent identity is represented by one of two modes:

- **Mode A  --  Public Identity**: The agent's Compact contract address serves as its identity. Registration is public. Metadata (name, capabilities, operator address) is stored in public state. Suitable for agents operating in open marketplaces.

- **Mode B  --  Private Identity**: The agent holds a ZK credential (a commitment to a secret). The commitment is stored in private state. The operator proves knowledge of the credential via ZK proof without revealing the underlying identity on-chain. Suitable for agents requiring operational privacy.

Both modes expose the same standard interface  --  the mode choice is an implementation detail abstracted by the SDK.

### 2. Identity Registry (`identity-registry.compact`)

#### 2.1 Public Mode Registration

An agent registers by calling `registerPublic()` on the Identity Registry contract. The call provides:

- `operator`: The Compact address of the human or organization controlling the agent.
- `metadata`: A public metadata struct containing `name` (string), `description` (string), `capabilities` (array of capability enum values), and `version` (string).
- `ownershipProof`: A cryptographic proof that `operator` controls `agentAddress` (a simple signature verification).

On successful registration, the contract stores the identity in public state and emits a `RegistrationEvent(agentAddress, operator)` event.

The agent is assigned a unique `AgentId` (numeric, auto-incremented). The Compact address and `AgentId` are both valid identifiers  --  contracts can reference agents by either.

#### 2.2 Private Mode Registration

An agent registers by calling `registerPrivate()` on the Identity Registry contract. The call provides:

- `operator`: The Compact address of the controlling entity.
- `metadata`: Identical struct to public mode.
- `credentialCommitment`: A hash commitment to the agent's secret credential.
- `credentialProof`: A ZK proof that the operator knows the preimage of `credentialCommitment`.

On successful registration, the contract stores the credential commitment in **private state**  --  it is not readable by other contracts or external observers. No public event is emitted for private registrations.

The agent is assigned a unique `AgentId` but this ID is stored in private state. External observers cannot enumerate private agents.

#### 2.3 Standard Interface

All identity operations, regardless of mode, are accessed through a uniform interface:

```
// Query whether an agent exists (returns boolean, works for both modes)
function exists(agentId: AgentId): bool

// Get public metadata (returns null for private agents)
function getMetadata(agentId: AgentId): Metadata?

// Prove agent identity to a verifier
// Public mode: returns the agent address from public state
// Private mode: returns a ZK proof of credential knowledge
function proveIdentity(agentId: AgentId): IdentityProof

// Update metadata (operator only)
function updateMetadata(agentId: AgentId, newMetadata: Metadata)

// Transfer operator control (requires multi-sig or ZK proof of current operator consent)
function transferOperator(agentId: AgentId, newOperator: CompactAddress, transferProof: TransferProof)

// Deactivate agent (operator only)
function deactivate(agentId: AgentId)
```

#### 2.4 Metadata Schema

```
struct Metadata {
    name: string,           // Human-readable agent name
    description: string,    // What the agent does
    capabilities: [Capability],  // Enumerated: PAYMENT, TRADING, GOVERNANCE, COMPLIANCE, DATA
    version: string,        // Semantic version
    homepage: string?,      // Optional URL
    erc8004TokenId: u256?,  // Optional cross-chain link to ERC-8004 NFT
}
```

### 3. Reputation Registry (`reputation-registry.compact`)

#### 3.1 Model: Public Scores, Private Evidence

The reputation system adopts a **public score, private evidence** model:

- **Public score**: A numeric value (0-100) and a tier label (Basic, Proven, Institutional) stored in public state. Any contract or observer can read an agent's current reputation tier without authorization.

- **Private evidence**: The transactions and validations that produced the score are stored as hash commitments in private state. The score is verifiable (it's on-chain), but the _reasons_ for the score are private  --  preventing competitors from reverse-engineering an agent's strategy from its reputation history.

- **ZK prove**: An agent can generate a ZK proof that asserts "my reputation score is ≥ X" without revealing the exact score. This enables nuanced trust negotiations  --  a counterparty can set a minimum reputation threshold without knowing the agent's precise standing.

#### 3.2 Reputation Tiers

| Tier | Score Range | Transaction Limits | Audit Access | Validator Requirement |
|------|-------------|-------------------|--------------|----------------------|
| **Basic** | 0-59 | Low volume, low value | Public only | 1+ Open validators |
| **Proven** | 60-89 | Standard commerce access | Auditor-view available | 3+ validators, ≥1 Trusted |
| **Institutional** | 90-100 | Full access, reduced friction | Full auditor-view | 5+ validators, ≥1 Institutional |

#### 3.3 Score Calculation

Reputation scores are calculated as a weighted moving average over a 90-day evidence window:

```
newScore = 0.7 * currentScore + 0.3 * weightedAverage(evidenceWindow)

weightedAverage = sum(evidence.weight * validatorReputation) / sum(validatorReputation)
```

- **Recency bias**: Evidence within the last 7 days carries 2x weight; 7-30 days carries 1x; 30-90 days carries 0.5x.
- **Validator quality**: Evidence from Institutional validators carries 3x weight; Trusted carries 2x; Open carries 1x.
- **Negative penalty**: One negative validation requires three positive validations to offset the score impact.
- **Decay**: If no new evidence is added for 30 days, the score decays by 1 point per week.

#### 3.4 Evidence Types

Evidence is added by validators (see Validation Registry) after observing agent behavior:

- **Transaction success**: The agent completed a transaction within policy limits.
- **Counterparty dispute**: A counterparty reported a policy violation.
- **Audit pass**: An auditor reviewed the agent's activity log and found compliance.
- **TEE attestation**: The agent produced a valid TEE attestation proving code integrity (optional extension).
- **Manual review**: A human reviewer approved a HITL-gated transaction.

Each evidence struct contains:
```
struct Evidence {
    evidenceType: EvidenceType,
    commitment: Hash,          // Hash of the underlying transaction data
    validatorId: ValidatorId,
    timestamp: u64,
    positive: bool,
}
```

#### 3.5 ZK Reputation Proof Interface

```
// Standard interface for reputation-constrained transactions
function proveReputation(agentId: AgentId, minScore: u8): ReputationProof

// ReputationProof contains:
//   - agentId (public input)
//   - proof that agent's score >= minScore
//   - timestamp (proof freshness)
//   - does NOT reveal exact score or evidence history
```

### 4. Validation Registry (`validation-registry.compact`)

#### 4.1 Hybrid Validator Model

The validation system uses three validator tiers with escalating requirements:

| Tier | Registration Requirement | Stake Required | Accuracy Tracking | Privileges |
|------|-------------------------|---------------|-------------------|------------|
| **Open** | Any address, stake NIGHT | 10,000 NIGHT | Yes | Can validate Basic tier agents |
| **Trusted** | Foundation audited | 50,000 NIGHT | Yes | Can validate Proven tier agents |
| **Institutional** | KYC-compliant entity | 100,000 NIGHT | Yes | Can validate all agents, produce compliance reports |

#### 4.2 Validator Registration

Validators register by calling `registerValidator()`:

```
function registerValidator(
    operator: CompactAddress,
    tier: ValidatorTier,
    stake: u64,
    proof: ValidatorProof  // ZK-proof of qualification
)
```

- **Open validators**: Must stake 10,000 NIGHT. No additional proof required. Stake is slashable for bad behavior.
- **Trusted validators**: Must prove Foundation audit completion. Stake is 50,000 NIGHT.
- **Institutional validators**: Must prove KYC completion via a ZK-KYC proof from an accredited provider. Stake is 100,000 NIGHT.

#### 4.3 Validation Operation

A validator adds evidence by calling `addEvidence()`:

```
function addEvidence(
    agentId: AgentId,
    evidence: Evidence,
    observationProof: ObservationProof  // ZK proof that validator observed the transaction
)
```

The validator's stake is at risk: if a validation is later challenged and found to be inaccurate (e.g., the agent committed a policy violation that the validator missed), the validator's `accuracyScore` is reduced and stake may be slashed.

#### 4.4 Validator Accuracy Tracking

```
function reportOutcome(agentId: AgentId, validatorId: ValidatorId, outcome: Outcome)

// SUCCESS: validator accuracyScore += 1
// VIOLATION_MISSED: validator accuracyScore -= 5, stake slashed by SLASH_AMOUNT
// FALSE_POSITIVE: validator accuracyScore -= 3, stake slashed by half SLASH_AMOUNT
```

Validators whose accuracy score falls below a threshold (configurable per tier) are demoted or deactivated.

### 5. Disclosure Tier Registry (`disclosure-tier-registry.compact`)

#### 5.1 Mapping to Midnight's Three-Tier Model

MAIS maps agent data to Midnight's three disclosure tiers as follows:

| Data Category | Public Tier | Auditor Tier | Private Tier |
|--------------|-------------|--------------|--------------|
| Agent existence | ✓ | ✓ | ✓ |
| Agent metadata | ✓ | ✓ | ✓ |
| Reputation score | ✓ | ✓ | ✓ |
| Current validator set | ✓ | ✓ | ✓ |
| Aggregated activity stats |  --  | ✓ | ✓ |
| Validator evidence history |  --  | ✓ |  --  |
| Transaction contents |  --  |  --  | ✓ |
| Agent strategy/configuration |  --  |  --  | ✓ |
| Operator identity (private agents) |  --  | ✓* | ✓ |

*Auditors registered by the agent operator can access operator identity at auditor tier. Unregistered auditors cannot.

#### 5.2 Auditor Registration

An agent operator can designate auditor keys:

```
function registerAuditor(
    agentId: AgentId,
    auditorAddress: CompactAddress,
    permissions: AuditorPermissions
)
```

`AuditorPermissions` specifies which data categories the auditor can access:
- `ACTIVITY_STATS`: Aggregate activity data (transaction counts, volume summaries)
- `EVIDENCE_HISTORY`: Individual validation evidence records
- `OPERATOR_IDENTITY`: The operator's Compact address (for private agents)

#### 5.3 Regulator Disclosure

For regulated use cases, MAIS defines a standard regulator disclosure flow:

1. A regulator submits a disclosure request with a ZK-KYC proof of regulatory authority.
2. The agent operator is notified (off-chain) of the disclosure request.
3. If the operator approves (or if a timeout passes and the agent's policy permits auto-disclosure), the firewall generates a structured compliance report at auditor-view level.
4. The report is a ZK proof that proves "this agent's activities for period [T1, T2] were within its declared policy set" without revealing individual transaction contents.

### 6. ERC-8004 Cross-Chain Identity Bridge (`erc8004-bridge.compact`)

#### 6.1 Optional Integration

ERC-8004 integration is **optional**  --  agents that don't need cross-chain identity operability can register without it. The bridge is a separate Compact contract that agents opt into.

#### 6.2 Bridge Registration

An agent links its Midnight identity to an ERC-8004 identity by calling `linkIdentity()`:

```
function linkIdentity(
    agentId: AgentId,
    erc8004TokenId: u256,
    crossChainProof: CrossChainProof
)
```

`crossChainProof` is a ZK proof that:
1. The submitter controls the Midnight agent identity (proven via the Identity Registry).
2. The submitter controls the ERC-8004 NFT token (proven by a signature from the EVM address holding the NFT, verified against a state proof of the ERC-8004 Identity Registry contract on Ethereum).
3. Both proofs are tied to the same root of trust (the operator's key).

#### 6.3 Cross-Chain Reputation

Once linked, an agent's Midnight reputation score can be contributed back to the ERC-8004 ecosystem:

- **Read from ERC-8004**: MAIS contracts can consume ERC-8004 reputation data as an input to Midnight reputation calculation (weighted alongside Midnight-native evidence).
- **Write to ERC-8004**: MAIS audit results can be written to ERC-8004's Validation Registry on Ethereum via a cross-chain message. This makes Midnight-sourced trust data visible in the broader ERC-8004 ecosystem  --  a first-of-its-kind contribution.

#### 6.4 Security Boundary

The bridge contract enforces:
- Cross-chain proofs must be freshness-constrained (within 1 hour of bridge call).
- A threshold of confirmations on the source chain before the bridge accepts a proof.
- Rate limiting on cross-chain state updates to prevent bridge flooding.

### 7. Standard Interface Summary

All MAIS-compliant contracts must implement the following query interface:

```
interface MAISAgent {
    // Identity
    function getAgentId(): AgentId
    function getOperator(): CompactAddress
    function getMetadata(): Metadata
    function getIdentityMode(): IdentityMode  // PUBLIC or PRIVATE
    
    // Reputation
    function getReputationScore(): u8
    function getReputationTier(): ReputationTier
    function proveReputation(minScore: u8): ReputationProof
    
    // Validation
    function getValidators(): [ValidatorId]
    function getValidatorCount(): u16
    
    // Disclosure
    function getDisclosureTier(): DisclosureTier
    function isAuditor(auditorAddress: CompactAddress): bool
    
    // Cross-chain (optional)
    function getErc8004TokenId(): u256?
    function isErc8004Linked(): bool
}
```

## Rationale

### Why a Three-Tier Privacy Model?

Midnight's core architectural differentiator from transparent blockchains is its three-tier disclosure model (public, auditor, private). MAIS maps directly to this model rather than abstracting it away because:

- **Public tier** enables open agent marketplaces  --  discoverability without compromising anything sensitive.
- **Auditor tier** is the compliance enabler. Regulated AI agents in finance, healthcare, and legal domains cannot operate without an auditable trail. Midnight's auditor tier provides this without exposing strategic data to competitors.
- **Private tier** protects competitive advantage. A trading agent's strategy is its moat; a governance agent's voting logic is sensitive. Private tier keeps these behind ZK proofs.

A purely public identity model (like ERC-8004) would discard Midnight's privacy advantage. A purely private model would prevent verifiable trust. The three-tier balance is the correct design.

### Why Dual-Mode Identity (Public + Private)?

Monolithic identity models fail for one side. A public-only model forces all agents to be linkable to their operators  --  unacceptable for agents handling sensitive data. A private-only model prevents open agent commerce  --  you cannot build a marketplace if you cannot discover agents.

Dual-mode identity lets the ecosystem choose per use case:
- **Public mode** is the default for open ecosystems (marketplaces, DAOs, public services).
- **Private mode** is the option for sensitive applications (institutional trading, healthcare agents, legal document agents).

Both modes expose the same `MAISAgent` interface, so consuming contracts and SDKs work identically regardless of the underlying mode.

### Why Public Scores + Private Evidence?

Alternative models considered:

| Model | Pros | Cons | Verdict |
|-------|------|------|---------|
| **Fully public scores + evidence** | Maximum verifiability | Competitors can reverse-engineer agent strategy from evidence history | Rejected: violates Midnight privacy thesis |
| **Fully private scores + evidence** | Maximum privacy | No verifiable trust; every transaction requires bilateral trust negotiation | Rejected: defeats the purpose of a reputation system |
| **Public scores + private evidence (chosen)** | Verifiable trust without strategy leak | Evidence privacy depends on hash commitment security | **Accepted** |
| **ZK-provable scores only (no public score)** | Maximum flexibility | Constant ZK proof generation adds latency; no public signal for quick decisions | Deferred to extension |

The chosen model  --  public score with private evidence  --  enables:
- **Quick trust decisions**: A contract can read the score from public state in O(1) with no ZK proof.
- **Deep trust verification**: When needed, a ZK proof can verify the score without revealing evidence.
- **Competitive privacy**: Evidence commitments prevent strategy reverse-engineering.
- **Compliance**: Auditors can access evidence history at auditor-view tier.

### Why Hybrid Validator Tiers?

A pure permissionless validation model (anyone can validate) is vulnerable to Sybil attacks where an agent operator creates fake validators to inflate their own score. A pure permissioned model requires governance overhead and creates a bottleneck for ecosystem growth.

The hybrid model:
- **Open tier** provides permissionless entry with anti-Sybil mechanisms (stake + accuracy tracking + slashing).
- **Trusted tier** provides a credibility upgrade path for serious validators.
- **Institutional tier** provides the regulatory credibility that enterprise customers require.

This is analogous to how certificate authorities operate in TLS  --  multiple tiers with escalating requirements and corresponding trust.

### Why Optional ERC-8004 Bridge?

ERC-8004 is the dominant agent identity standard on EVM chains. Midnight agents that also operate on Ethereum (or interact with Ethereum-based agents) benefit from cross-chain identity linking. However:

- Not all Midnight agents need EVM interoperability.
- Cross-chain bridges introduce additional attack surface.
- Making the bridge optional keeps the core standard simpler and safer.

The bridge is a separate contract that agents opt into. The core identity, reputation, and validation registries function independently of it.

## Path to Active

### Acceptance Criteria

MIP-X shall be considered for **Accepted** status when:

1. **Reference implementation**: A working implementation of the Identity Registry, Reputation Registry, and Disclosure Tier Registry Compact contracts exists and passes a test suite on Midnight testnet.
2. **Community review**: The proposal has been presented at a Midnight community workshop and feedback has been incorporated.
3. **Ecosystem endorsement**: At least two Midnight ecosystem projects (e.g., Midnight City, AlphaTON, or an Aliit Fellowship project) have expressed intent to adopt the standard.
4. **Security review**: An initial security review of the Compact contract design has been completed by a Midnight Foundation reviewer or an independent auditor.

MIP-X shall be considered for **Active** status when:

1. **Testnet deployment**: All core contracts are deployed on Midnight testnet and operating for at least 4 weeks without critical issues.
2. **SDK availability**: The `@midnight-agent/mais-sdk` TypeScript package is published and documented.
3. **Ecosystem adoption**: At least 5 unique agent identities registered on testnet, with at least 20 reputation evidence operations performed.
4. **Documentation**: Complete developer documentation including ARCHITECTURE.md, CONTRIBUTING.md, and a developer quickstart that gets a test agent registered in under 10 minutes.
5. **Mainnet readiness**: A mainnet deployment plan has been prepared and reviewed.

### Implementation Plan

| Phase | Component | Timeline | Owner |
|-------|-----------|----------|-------|
| 1 | Identity Registry (`identity-registry.compact`) | Weeks 1-4 | TBD |
| 2 | Reputation Registry (`reputation-registry.compact`) | Weeks 4-8 | TBD |
| 3 | Disclosure Tier Registry (`disclosure-tier-registry.compact`) | Weeks 8-10 | TBD |
| 4 | Validation Registry (`validation-registry.compact`) | Weeks 10-14 | TBD |
| 5 | ERC-8004 Bridge (`erc8004-bridge.compact`) | Weeks 14-16 | TBD |
| 6 | TypeScript SDK (`@midnight-agent/mais-sdk`) | Weeks 16-20 | TBD |
| 7 | Testnet deployment + ecosystem onboarding | Weeks 20-24 | TBD |
| 8 | Security audit + mainnet preparation | Weeks 24-28 | TBD |

## Backwards Compatibility Assessment

### Impact on Existing Systems

MAIS is a new standard and does not modify any existing Midnight protocol components. No hard fork is required. The standard is implemented entirely at the application layer via Compact contracts.

### Compatibility with Existing Midnight Standards

- **MPS-0003 (CAIP Support)**: No conflict. MAIS uses native Midnight identifiers. CAIP chain identifiers can be used for cross-chain bridge operations but are not required for core functionality.
- **MPS-0004 (Trusted Proof Serving)**: Complementary. MAIS agents that require delegated proof generation can use MPS-0004's TEE attestation flow as a validator evidence type.
- **MPS-0005 (Events)**: No conflict. MAIS emits standard Midnight events for public operations. Private operations do not emit events by design.
- **Midnight ZSwap**: No conflict. MAIS agents can hold and transact ZSwap shielded tokens normally.

### Compatibility with ERC-8004

MAIS is designed as a complementary standard to ERC-8004, not a competing one. The optional ERC-8004 bridge enables interoperability. ERC-8004 agents that link to MAIS maintain their ERC-8004 identity. MAIS agents that don't link to ERC-8004 are unaffected.

### Migration Path

No migration is required as this is a new standard. Existing ad-hoc agent identity implementations on Midnight can adopt MAIS by:
1. Registering their agent with the Identity Registry.
2. Implementing the `MAISAgent` interface on their existing contract (a thin wrapper).
3. Optionally migrating any existing reputation data into the Reputation Registry.

## Security Considerations

### Threat Model

**Threat Actor 1  --  Sybil validator**: An agent operator creates multiple fake validator identities to inflate their own reputation score.

*Mitigation*: Open validators must stake 10,000 NIGHT. Accuracy tracking with slashing makes Sybil validation economically irrational. The stake cost multiplied by the number of validators needed to meaningfully inflate a score exceeds the expected benefit of a higher reputation tier.

**Threat Actor 2  --  Reputation gaming**: An agent executes a series of trivial, low-risk transactions to accumulate positive evidence, then pivots to high-value malicious transactions once reputation is high.

*Mitigation*: The 90-day sliding window with recency bias makes reputation a "perishable" asset. Score decay (1 point/week without new evidence) prevents long-term hoarding. Evidence type weighting gives lower weight to low-value transactions  --  the system can be configured to require high-value successful transactions for tier upgrades.

**Threat Actor 3  --  Private agent deanonymization**: An observer correlates private agent activity with public metadata patterns (timing, transaction sizes, counterparty patterns) to identify the agent's operator.

*Mitigation*: This is a fundamental metadata analysis problem, not unique to MAIS. Midnight's privacy layer already provides baseline protections. MAIS adds: (a) private agent IDs are not emitted in events; (b) credential commitments use salted hashes; (c) the standard recommends randomized transaction timing and amount bucketing for privacy-sensitive agents (implementation guidance in SDK documentation).

**Threat Actor 4  --  Bridge oracle manipulation**: An attacker feeds false ERC-8004 data through the cross-chain bridge to manipulate Midnight-based reputation scores.

*Mitigation*: The bridge requires confirmed source-chain state proofs (not oracle reports). Freshness constraints prevent stale data injection. Rate limiting prevents flooding attacks. ERC-8004 data is weighted alongside Midnight-native evidence  --  even a completely compromised bridge cannot override Midnight-sourced reputation.

**Threat Actor 5  --  Compromised operator key**: If an agent operator's key is stolen, the attacker can deactivate the agent, transfer control, or modify metadata.

*Mitigation*: Operator transfer requires multi-factor proof (either multi-sig or a ZK proof of current operator consent). Deactivation is irreversible  --  an attacker cannot profit from deactivating an agent. Metadata modifications are non-critical (they don't affect transaction authorization). Key rotation is recommended via Midnight's wallet SDK.

### What MAIS Does Not Protect Against

Explicit scope boundaries:

- MAIS does not guarantee agent _behavior_  --  only that agent identity, reputation, and validation are verifiable. An agent with a high reputation score can still execute a bad transaction; the reputation downgrade that follows is the mechanism, not prevention.
- MAIS does not protect against smart contract vulnerabilities in the Compact contracts implementing the standard itself. These contracts require independent security audit.
- MAIS does not protect against quantum adversaries breaking the underlying ZK proof system. Midnight's Plonk implementation inherits the cryptographic assumptions of the BLS12-381 curve.

## Implementation

### Compact Contract Structure

The reference implementation consists of four Compact contracts:

```
midnight-agent-identity/
├── contracts/
│   ├── identity-registry.compact       # Agent registration + metadata
│   ├── reputation-registry.compact     # Score calculation + evidence storage
│   ├── validation-registry.compact     # Validator management + evidence submission
│   ├── disclosure-tier-registry.compact # Three-tier access control
│   └── erc8004-bridge.compact          # Cross-chain identity linking
├── tests/
│   ├── identity-registry.test.ts
│   ├── reputation-registry.test.ts
│   ├── validation-registry.test.ts
│   ├── disclosure-tier-registry.test.ts
│   └── erc8004-bridge.test.ts
├── sdk/
│   └── mais-sdk/
│       ├── src/
│       │   ├── agent.ts
│       │   ├── identity.ts
│       │   ├── reputation.ts
│       │   ├── validation.ts
│       │   └── disclosure.ts
│       └── package.json
├── ARCHITECTURE.md
├── CONTRIBUTING.md
└── README.md
```

### Dependencies

- **Midnight Compact Runtime**: For ZK proof generation and verification.
- **Midnight.js SDK**: For wallet integration and transaction submission.
- **Midnight ZSwap**: For NIGHT staking (validators).
- **zk-kit**: For credential commitment schemes (optional, for private identity mode).
- **External (optional)**: ERC-8004 contract ABIs for cross-chain bridge.

### SDK Interface Design

```typescript
// Initialize agent identity
const agent = new MAISAgent({
  identityMode: 'public' | 'private',
  operator: midnightWallet,
  metadata: {
    name: 'TradingBot Alpha',
    description: 'Automated DEX arbitrage agent',
    capabilities: ['TRADING', 'PAYMENT'],
    version: '1.0.0',
  },
  disclosureTier: 'public',
  // Optional
  erc8004TokenId: 12345,
});

// Register on Midnight
await agent.register();

// Get reputation
const reputation = await agent.getReputation();
// => { score: 85, tier: 'PROVEN', evidenceCount: 42, lastUpdated: '2026-05-15' }

// Prove reputation (ZK proof that score >= threshold)
const proof = await agent.proveReputation({ minScore: 60 });
// Submit proof to counterparty contract for verification

// Add evidence as validator
await validator.addEvidence({
  agentId: agent.id,
  evidence: { type: 'TRANSACTION_SUCCESS', positive: true },
  proof: observationProof,
});

// Read audit trail (auditor-view)
const auditTrail = await agent.getAuditTrail({
  from: '2026-05-01',
  to: '2026-05-18',
  disclosureTier: 'auditor',
});
```

## Testing

### Unit Tests

Each Compact contract must have ≥90% branch coverage:

- **Identity Registry**: Test public registration, private registration, metadata update, operator transfer (authorized and unauthorized), deactivation, reactivation prevention.
- **Reputation Registry**: Test score calculation edge cases (no evidence, single evidence, many evidence), negative penalty, decay over time, tier transitions, ZK proof generation and verification.
- **Validation Registry**: Test validator registration (all tiers), evidence submission, accuracy tracking, stake slashing, demotion.
- **Disclosure Tier Registry**: Test tier transitions, auditor registration, unauthorized access attempts, regulator disclosure flow.
- **ERC-8004 Bridge**: Test identity linking, cross-chain proof verification, replay prevention, rate limiting.

### Integration Tests

- Full agent lifecycle: register → accumulate reputation → upgrade tier → cross-chain link → deactivate.
- Multi-agent scenario: 10 agents with interleaved reputation operations.
- Validator collusion scenario: multiple validators from same operator attempting Sybil attack.

### Stress Tests

- 10,000 agent registrations (concurrent).
- 1,000 evidence submissions per minute (reputation throughput).
- 100 concurrent reputation proof generations.

### Test Environment

Tests run against a local Midnight testnet (devnet) configured via the Midnight.js SDK. CI pipeline via GitHub Actions runs the full test suite on every PR.

## References

- ERC-8004: AI Agent Identity, Reputation, and Validation Registries  --  [ERC-8004 Specification](https://eips.ethereum.org/EIPS/eip-8004)
- Midnight Network Documentation  --  [docs.midnight.network](https://docs.midnight.network/)
- Midnight Compact Language Reference  --  [docs.midnight.network/develop/reference/compact](https://docs.midnight.network/develop/reference/compact/)
- AlphaTON + Midnight Partnership  --  Midnight Foundation Announcement (2026)
- Midnight City Simulation  --  Midnight Network Developer Portal
- Bastion Agentic Defense  --  On-chain security middleware for AI agents (complementary project)
- ZK-KYC: Zero-Knowledge Proofs for KYC Compliance  --  Reference implementations by various providers

## Acknowledgements

This MIP draws inspiration from ERC-8004's agent identity model (Ethereum Foundation dAI team), the Bastion Agentic Defense project's security middleware architecture, and Midnight's three-tier disclosure privacy model (IOG/Midnight Foundation).

## Copyright Waiver

All contributions (code and text) submitted in this MIP must be licensed under the Apache License, Version 2.0. Submission requires agreement to the Midnight Foundation Contributor License Agreement [Link to CLA], which includes the assignment of copyright for your contributions to the Foundation.
