# OODL: Beyond Ordinals

## The Bitcoin Intent Layer Using Embedded PSBT Execution

**Version 1.0 — Protocol White Paper**
**Author: switch-900**
**Platform: oodinals.com**

---

## Thesis Statement

OODL transforms Bitcoin transactions into publicly discoverable executable agreements enforced entirely by Bitcoin validation. No smart contracts. No trusted operators. No consensus changes required.

---

## Abstract

Bitcoin enables peer to peer value transfer through deterministic transaction validation. The underlying transaction model also permits a broader capability. Transactions can express agreements that become executable in the future.

OODL introduces a protocol that transforms partially signed Bitcoin transactions into publicly discoverable executable intents. By embedding seller offer PSBT data inside on chain transactions, OODL allows any participant to finalize trustless exchanges without centralized coordination.

OODL moves Bitcoin beyond artifact storage. It establishes a coordination model where agreements are published directly to the blockchain and executed permissionlessly by network participants.

Bitcoin becomes not only a settlement system but also a global agreement execution network. This paper describes the protocol mechanics, the security model, the full range of application possibilities, and a rigorous assessment of how each possibility fits within existing Bitcoin design principles.

---

## 1. Introduction

Ordinals demonstrated that Bitcoin can permanently host digital artifacts. This revealed that Bitcoin can act as a data layer as well as a payment system. The implications of that discovery are still unfolding.

OODL explores the next evolution. Instead of publishing completed payments, or storing static data, participants publish executable agreements. The agreement itself lives on chain. Execution happens later, by any willing participant, without permission from any central authority.

Traditional Bitcoin usage broadcasts finalized transactions. OODL publishes incomplete transactions that become valid once completed by another participant. This distinction is fundamental. Finality is deferred. Authority is open. Coordination is global.

The blockchain functions as a coordination environment. Bitcoin becomes a public notice board where economic intent is published and permissionlessly fulfilled.

> Bitcoin has always been a verification machine. OODL reveals that verification can enforce contracts without execution engines, oracles, or trusted operators.

---

## 2. Motivation

Existing inscription marketplaces depend on centralized services. These systems maintain off chain order books, coordinate buyers and sellers, and construct transactions on behalf of users. This introduces trust assumptions that directly conflict with Bitcoin design principles.

Users must trust that the marketplace will not front-run their orders, censor their listings, misappropriate funds, or simply cease to operate. These risks are not theoretical. Centralized exchange failures have repeatedly demonstrated that custodial and coordinating intermediaries represent a systemic vulnerability.

OODL removes the marketplace entirely. The protocol allows agreements themselves to exist on chain. Discovery and execution occur through Bitcoin validation rather than centralized infrastructure. The marketplace is replaced by a protocol.

### Primary Goals

1. Eliminate trusted intermediaries from inscription trading and economic coordination.
2. Preserve native Bitcoin security guarantees at every stage of execution.
3. Enable permissionless participation for both sellers and buyers.
4. Maintain full compatibility with existing Bitcoin consensus rules.
5. Require no protocol upgrades, no soft forks, and no new opcodes.
6. Create a foundation extensible beyond inscription trading to broader Bitcoin coordination mechanisms.

---

## 3. Core Concept

OODL introduces the concept of an executable transaction intent. An intent is a transaction commitment that defines conditions for execution while remaining incomplete. It is a promise published to the world, waiting for the right counterparty to fulfill it.

Participants publish a valid transaction template awaiting completion by any counterparty. Bitcoin signatures enforce the agreement. Execution authority becomes open rather than predetermined. No escrow is held. No trusted party coordinates the match. The agreement is self-enforcing through cryptography.

### The Signature Construction

The technical foundation is SIGHASH_SINGLE combined with ANYONECANPAY. This signature mode commits a signer to one specific input and one specific output while leaving all other transaction fields open for extension by future participants.

In OODL, the seller signs their inscription input and their payment output. The amounts, the destination, the conditions are locked by that signature. But the transaction is incomplete. A buyer may add their own inputs, add their own change output, and broadcast the completed transaction. Bitcoin validates the combined result as a whole.

The seller's conditions are guaranteed by Bitcoin consensus itself. The buyer cannot redirect the seller's payment. The seller cannot reclaim the inscription after a buyer has committed. Settlement is atomic and trustless.

> Contract logic emerges from transaction structure rather than scripting complexity. Bitcoin does not execute programs. Bitcoin verifies signatures. OODL converts those signatures into contract enforcement.

---

## 4. Embedded Offer Mechanism

OODL listings operate through a three transaction lifecycle. Each stage serves a distinct purpose. Together they transform an inscription into a publicly discoverable, permissionlessly executable economic offer.

### 4.1 Commit Transaction

The commit transaction prepares execution state. The inscription being listed is aligned to sat offset zero within a Taproot carrier output. Protocol fee routing is established so that indexers can identify and classify the listing. Funding inputs establish the deterministic structure required for later execution. The commit transaction creates the conditions under which the reveal can safely embed the executable agreement.

### 4.2 Reveal Transaction

The reveal transaction publishes the executable agreement. The seller offer PSBT is embedded within witness data using an envelope pattern similar to inscription creation. Compression may optionally reduce payload size, lowering transaction fees without affecting protocol validity.

Indexers detect listings through routed fee outputs. The reveal transaction does not perform a sale. It does not transfer the inscription. It publishes execution intent. The inscription remains under the seller's control until a buyer completes execution.

### 4.3 Buy Transaction

Any participant may extract the embedded PSBT from the reveal transaction and complete execution. The executor adds funding inputs to cover the sale price and network fees. Placeholder fields in the PSBT template are completed with real values. Buyer signatures are applied. The completed transaction is broadcast to the Bitcoin network.

Bitcoin validation guarantees atomic settlement. Either the entire transaction is valid and both parties receive exactly what the PSBT specified, or nothing happens. Seller receives payment. Buyer receives the inscription. No partial execution is possible.

---

## 5. Seller Offer PSBT Structure

OODL uses a deterministic PSBT template. The structure is fixed so that indexers, wallets, and execution tools can parse and validate offers without bespoke logic.

### Inputs

1. Placeholder input supplied by buyer at execution time.
2. Placeholder input supplied by buyer at execution time.
3. Seller inscription input signed by seller using SIGHASH_SINGLE | ANYONECANPAY.

### Outputs

1. Buyer change output defined at execution time by the executor.
2. Inscription receiver output carrying postage value to transfer the sat.
3. Seller payout output with the agreed price, committed by seller signature.

### Signature Model

The seller signs using SIGHASH_SINGLE combined with ANYONECANPAY. SIGHASH_SINGLE commits the seller to input 3 and output 3 only. Their payout is locked. ANYONECANPAY allows unknown future counterparties to extend the transaction by adding inputs 1 and 2 without invalidating the seller signature.

The executor may extend the transaction freely but cannot alter the seller's committed input-output pair without invalidating the signature. Bitcoin consensus enforces this guarantee without any additional code.

> The executor may extend the transaction but cannot violate seller conditions. No trusted party is needed to enforce this. Bitcoin does it automatically.

---

## 6. Execution Guarantees

Bitcoin does not execute programs. Bitcoin verifies signatures against spending conditions. OODL exploits this verification architecture to create contract enforcement that requires no additional runtime, no VM, and no trusted execution environment.

The seller commits to spending conditions at listing time. The buyer provides execution funding at purchase time. Bitcoin validates the combined agreement automatically at broadcast time. The contract is enforced by every full node in the network simultaneously, without any single point of coordination.

### Atomicity

Settlement is fully atomic. The inscription transfer and the payment transfer occur in a single transaction. There is no window in which one party receives their asset while the other does not. Double spend attacks against the listing are prevented by Bitcoin's UTXO model. A UTXO can only be spent once. If the seller's inscription UTXO is consumed, the listing is dead.

### Cancellation

A seller may revoke a published intent by spending their inscription UTXO to themselves in a separate transaction. Once the inscription UTXO is spent, the embedded PSBT becomes invalid. Any attempt to execute against a spent input will be rejected by Bitcoin validation. The cancellation is enforceable by the seller unilaterally, without marketplace permission.

### Stale Input Prevention

Indexers and execution tools must verify that the seller's input UTXO remains unspent before presenting a listing as available. This is a standard UTXO lookup, not an off chain trust assumption. Bitcoin's UTXO set provides the definitive answer.

---

## 7. OODL Usage Paths

### 7.1 Single Asset Listing

A single inscription is published using the three transaction lifecycle. Optional parameters include pricing, collection classification, provenance hints, and block height restrictions. Block height restrictions allow time-limited offers, creating auction dynamics without any external auction infrastructure. Execution remains atomic and permissionless.

**Bitcoin Fit: NATIVE** — Uses only existing Taproot, witness data, and PSBT mechanisms. No new primitives required.

### 7.2 Multi Asset Listing

Multiple inscriptions may be prepared simultaneously and published in a single reveal transaction. Each asset maintains independent pricing and execution capability. A single reveal transaction publishes multiple executable intents as separate PSBT payloads. Batch publication reduces per-listing transaction overhead while preserving flexibility and atomicity at the individual execution level.

**Bitcoin Fit: NATIVE** — Multiple PSBT payloads in a single witness envelope. Fully valid within existing rules.

### 7.3 On Chain Collection Registry

Collections exist as on chain registry manifests using parent-child inscription relationships. A designated parent inscription defines registry authority. Child inscriptions contain collection metadata encoded as JSON. Each collection specifies an indexing address and membership rules. Listings route protocol fees to these addresses, enabling decentralized discovery without a centralized catalog.

Any indexer that observes fee routing to a known collection address can identify and classify that listing without any off chain coordination. The collection registry is therefore permissionless and censorship-resistant by construction.

**Bitcoin Fit: NATIVE** — Parent-child inscription relationships are established Ordinals convention. JSON metadata in witness data is valid today.

### 7.4 Direct Offer Distribution

Seller offer PSBT data may be shared directly between participants without on chain publication. Distribution channels include direct messaging, QR code transfer, private community channels, and peer to peer file sharing. Execution validation is identical regardless of how the PSBT was obtained. The on chain publication step is optional for parties who already know each other.

**Bitcoin Fit: NATIVE** — PSBTs are a standard Bitcoin primitive. Sharing them privately and executing them on chain has always been valid.

---

## 8. The Bitcoin Intent Layer

OODL introduces a new conceptual layer to Bitcoin's stack. The existing layers are well understood. Payments transfer value. Contracts define spending conditions. OODL adds a third: intents publish executable agreements.

An intent is not a payment. It is not merely a spending condition. It is a published economic desire backed by a cryptographic commitment. The combination of public discoverability and cryptographic enforcement creates something genuinely new: a permissionless, trustless, globally accessible coordination mechanism running entirely on Bitcoin.

> Bitcoin becomes a public bulletin board of economic coordination. Participants publish desired outcomes rather than finalized actions. Any matching counterparty anywhere in the world may fulfill the intent permissionlessly.

This is not metaphorical. The blockchain literally stores the executable agreement. The network literally validates execution. Coordination is not layered on top of Bitcoin. Coordination is expressed in Bitcoin.

---

## 9. Expanded Possibility Space

The OODL mechanism extends well beyond inscription trading. The combination of on chain PSBT publication, SIGHASH_SINGLE and ANYONECANPAY, and permissionless execution creates a design space for trustless coordination that has not previously existed on Bitcoin. Each possibility below is assessed for its fit with existing Bitcoin capabilities.

### 9.1 Decentralised Order Books

Traditional order books require a trusted matching engine. OODL replaces the matching engine with the blockchain itself. Sellers publish offers. Buyers scan the blockchain for matching offers and execute directly. Price discovery happens through market participants reading and responding to publicly visible intents.

An OODL order book requires no server, no API, and no operator. The blockchain is the order book. Any participant can build indexing tooling to aggregate and display visible offers. Competing indexers provide redundancy. No single indexer controls access.

**Bitcoin Fit: NATIVE** — Offer publication uses existing PSBT and witness mechanisms. Order book state is derived from UTXO state. No new primitives needed.

### 9.2 Permissionless Auctions

Block height restrictions embedded in PSBT metadata allow time-bounded offers. A seller may publish an offer valid only until a specified block. Any buyer who executes before that block wins the asset at the stated price. This is a first-come-first-served auction enforced by block time.

More sophisticated auction structures, such as declining price Dutch auctions, can be approximated by publishing multiple offers at different price points with different block height expiries. The highest price offer expires first. If unclaimed, the next lower price becomes the active offer. No smart contract logic is required.

**Bitcoin Fit: YES** — Block height as an expiry condition can be approximated using PSBT metadata and off chain validation. Fully enforceable on chain through UTXO spending.

### 9.3 Global Bounty Systems

A bounty is an offer to pay for the delivery of a specific output. In OODL terms, a bounty is a PSBT that commits payment to any party who can produce a specific transaction structure satisfying stated conditions. The conditions are defined in the PSBT and enforced by Bitcoin signature rules.

Bounty systems built on OODL require no escrow agent, no dispute resolution service, and no trusted arbiter. Bitcoin consensus determines whether the execution conditions are satisfied.

**Bitcoin Fit: PARTIAL** — Conditions verifiable by Bitcoin signature rules work natively. Conditions requiring off chain data validation require an indexer layer, but payment enforcement remains trustless on chain.

### 9.4 Sponsored Transactions

ANYONECANPAY enables fee sponsorship natively. A third party may add inputs to an existing PSBT to cover network fees without altering the seller's committed conditions. This allows protocols, wallets, or fee markets to sponsor specific transactions for their users or for economic reasons.

**Bitcoin Fit: NATIVE** — ANYONECANPAY is the native Bitcoin mechanism for fee sponsorship. This works today with no changes.

### 9.5 Execution Markets

Once intents are public and execution is permissionless, a competitive market for execution services emerges naturally. Execution specialists monitor the blockchain for profitable intents, provide required funding inputs, collect protocol fees, and broadcast completed transactions. These services compete on speed, capital efficiency, and reliability.

Miners represent the most privileged execution participants. A miner observing a profitable intent may finalize the execution transaction and include it in a block they produce. The miner captures both the block fee and any protocol fee embedded in the execution. This creates a new revenue stream for miners that does not depend on the block subsidy.

**Bitcoin Fit: NATIVE** — Miners constructing their own transactions to fulfill intents is valid today. This is miner extractable value operating in favour of protocol participants rather than against them.

### 9.6 Miner Incentive Evolution

Bitcoin's long-term security depends on miner revenue remaining sufficient to sustain adequate hash rate as block subsidies decline toward zero. Transaction fees are the intended replacement revenue source. OODL introduces a third mechanism: execution opportunity.

A miner who monitors the OODL intent layer may identify profitable execution opportunities. By acting as an execution participant, miners can capture protocol fees with the additional advantage of controlling block inclusion. This creates a new alignment between miner incentives and protocol health.

**Bitcoin Fit: NATIVE** — No changes required. Miners acting as execution participants is valid under current consensus rules. This is an emergent economic consequence, not a protocol change.

### 9.7 Trustless OTC Markets

Over-the-counter trading of large inscription lots currently requires escrow services or trusted counterparties. OODL enables direct bilateral OTC execution without any escrow. Two parties negotiate terms privately, construct a PSBT reflecting those terms, and execute. Settlement is atomic. Neither party can unilaterally abort after both have committed inputs.

**Bitcoin Fit: NATIVE** — Private PSBT construction and on chain execution is standard Bitcoin behavior. No new primitives are required.

### 9.8 Automated Settlement Agreements

OODL intents may be constructed programmatically by autonomous systems. A wallet or protocol may publish execution intents automatically in response to market conditions, portfolio triggers, or protocol events. The automating system publishes the PSBT. Any execution participant fulfills it. The protocol operator does not need to be online at execution time.

**Bitcoin Fit: YES** — Automated PSBT construction is software, not a Bitcoin protocol change. On chain execution remains standard.

### 9.9 Cross Protocol Settlement

OODL's execution model may be extended to assets beyond Ordinals inscriptions. Any UTXO-based asset that can be expressed in a PSBT, including Runes and future inscription standards, may be listed and traded using the OODL mechanism. The protocol is asset-agnostic at the Bitcoin layer.

**Bitcoin Fit: YES** — PSBT and Taproot handle any UTXO-based asset. Cross-asset support depends on indexer implementation, not Bitcoin protocol changes.

### 9.10 Subscription Markets

Intents may be published as streams. A seller may publish a series of time-limited offers at regular intervals, creating a subscription-like market for recurring asset access. Subscription markets using OODL primitives enable recurring revenue models for digital asset creators without any centralised subscription service.

**Bitcoin Fit: YES** — Time-limited PSBT offers published in series. Each is a standard OODL listing. Subscription logic lives in the client layer, not the protocol layer.

### 9.11 Reputation and Provenance Anchoring

The on chain publication of intents creates an immutable record of market participation. A seller's history of fulfilled intents, unfulfilled listings, and cancellations is permanently readable on Bitcoin. Indexers may derive reputation scores from this history without any centralised rating service.

Provenance chains for inscriptions become automatically derivable from OODL transaction history. The chain of custody from creation through every subsequent sale is readable from Bitcoin directly.

**Bitcoin Fit: NATIVE** — Transaction history is inherent to Bitcoin. OODL listing history is a subset of that. Indexers derive reputation from existing blockchain data.

### 9.12 Decentralised Launchpads

Collection launches currently require centralized launchpad services. OODL enables trustless launchpad mechanics through on chain intent publication. A collection creator publishes mint intents with specified conditions, prices, and execution windows. Revenue flows directly to the creator without passing through a launchpad operator.

**Bitcoin Fit: PARTIAL** — Price and execution window enforcement is native. Complex whitelist logic requires indexer layer validation. Revenue routing is fully trustless on chain.

### 9.13 Protocol Fee Markets

OODL protocol fees are routed to designated addresses observable by indexers. Multiple indexers may offer competing discovery services, each routing fees to their own address. Fee market competition drives indexer quality without any central authority setting terms.

**Bitcoin Fit: NATIVE** — Fee routing addresses in transactions are fully valid today. Competitive indexers are a client layer concern, not a protocol concern.

---

## 10. Relationship to Existing Bitcoin Systems

### Lightning Network

Lightning channels rely on pre-signed transactions stored privately between channel participants. OODL publishes pre-signed transactions publicly. Lightning represents the private, high-frequency, low-value payment layer. OODL represents the open, lower-frequency, higher-value economic coordination layer. They are complementary rather than competitive.

### Discreet Log Contracts

DLCs enable conditional execution based on oracle signatures. OODL enables execution based purely on participant action. No oracle is required. The executing buyer fulfills the contract condition simply by constructing and broadcasting the valid transaction.

### Ordinals

Ordinals introduced digital artifacts on Bitcoin. OODL extends the Ordinals framework by embedding not static content but executable economic agreements in that same witness data space. Ordinals made Bitcoin a data layer. OODL makes Bitcoin a coordination layer.

### Ethereum and Smart Contract Platforms

Smart contract platforms achieve coordination through programmable execution environments. This model introduces significant complexity, attack surface, and trust assumptions about the execution environment itself.

OODL achieves coordination without computation. Bitcoin verifies. It does not compute. OODL's coordination guarantees are therefore backed by Bitcoin's security model rather than the security of a separate execution environment.

> What Ethereum achieves through programmability, OODL approximates through transaction structure. The result is less expressive but dramatically more secure and simpler to reason about.

---

## 11. Security Model

OODL relies exclusively on existing Bitcoin validation. No new cryptographic assumptions are introduced. No trusted operators maintain security properties. Security derives from Bitcoin consensus.

### Security Properties

1. Seller payout commitment is enforced by SIGHASH_SINGLE. The seller's output cannot be modified without invalidating the seller's signature.
2. Buyer verification occurs prior to execution through PSBT inspection. Any wallet or tool can verify PSBT integrity before adding buyer inputs.
3. Transaction mutation after signing is detected by Bitcoin's signature verification. A modified PSBT will fail validation at the network level.
4. Stale input execution is prevented by Bitcoin's UTXO model. A spent input cannot be used in a new transaction.
5. Deterministic validation is independent of external services. Any full node can validate any OODL execution transaction without querying external APIs.

### Trust Assumptions

OODL introduces no trust assumptions beyond those already present in Bitcoin. Indexers provide discovery services but cannot alter execution guarantees. Wallets construct PSBTs but their output is verifiable by any party. The only trusted party in OODL is Bitcoin itself.

### Attack Vectors and Mitigations

Front-running of execution is possible. A miner observing a profitable execution may prioritize their own execution over a user's broadcast. The seller is unaffected regardless of who executes. The buyer who front-runs still pays the seller's price.

PSBT forgery is infeasible given Bitcoin's signature scheme. A forged PSBT that modifies the seller's committed outputs will fail signature verification when broadcast.

Replay attacks on cancelled listings are prevented by UTXO consumption. A cancelled listing spends the seller's inscription UTXO to themselves, permanently invalidating the original PSBT.

---

## 12. Decentralisation Properties

OODL eliminates several roles that currently require trusted operators in inscription marketplaces.

- Marketplace coordination is unnecessary. Bitcoin coordination replaces marketplace coordination.
- Escrow agents are unnecessary. Atomic settlement replaces escrow.
- Custodial infrastructure is unnecessary. Self-custody is preserved throughout the listing and execution lifecycle.
- Order matching servers are unnecessary. The blockchain is the order book.
- Fee distribution operators are unnecessary. Protocol fee routing is embedded in transactions.
- Listing approval gatekeepers are unnecessary. Any valid OODL transaction is a valid listing.

The residual centralisation risks in OODL are indexer concentration and wallet tooling distribution. If a small number of indexers dominate discovery, they acquire soft censorship power. This risk is mitigated by the open nature of the indexing protocol. Any party may run an indexer. No permission is required to index OODL transactions.

---

## 13. Beyond Ordinals

OODL demonstrates that Bitcoin can support decentralised coordination mechanisms that do not depend on inscription content. The protocol generalises from inscription trading to a broader set of use cases that all share a common structure: an intent published publicly and executed permissionlessly.

### The Generalised Intent Pattern

Every OODL application follows the same pattern. A party publishes a PSBT expressing desired conditions. Any counterparty satisfying those conditions may execute. Bitcoin validates settlement. The pattern is invariant. The application space is determined by what conditions can be expressed in PSBT structure and enforced by Bitcoin signature rules.

### Application Categories

- Decentralised asset trading without marketplace operators.
- Permissionless auctions without auction house infrastructure.
- Trustless bounty and reward systems without escrow agents.
- Open fee markets for protocol services without fee administrators.
- Automated economic agents without custody or operational risk.
- Immutable provenance and reputation records without centralised databases.
- Miner execution markets creating new revenue streams without subsidy dependence.

> Bitcoin evolves from a payment network into an agreement network. Payments move value. Agreements coordinate behavior. Both now run natively on Bitcoin.

---

## 14. Protocol Philosophy

OODL follows Bitcoin's core design philosophy rigorously. Every decision reflects an explicit principle.

### Minimalism Over Complexity

OODL uses the minimum necessary primitives. Taproot outputs, witness envelopes, PSBTs, and existing signature hash types. No new opcodes, no new transaction types, no new cryptographic constructions. Complexity is the enemy of security.

### Verification Over Computation

OODL achieves contract enforcement through signature verification rather than computation. Bitcoin verifies. It does not compute. OODL aligns with this design constraint rather than fighting it.

### Coordination Over Programmability

Programmability enables arbitrary computation. Coordination enables economic agreement. OODL optimises for coordination. The expressive power of OODL contracts is narrower than Ethereum smart contracts. The security model is stronger. This is an intentional tradeoff.

### Backward Compatibility Over Protocol Modification

OODL requires no changes to Bitcoin consensus rules. Every OODL transaction is valid under current rules. No soft fork. No miner signaling. No activation threshold. Deployment requires tooling, not permission.

### Permissionless Participation Over Gatekeeping

Any participant may list, execute, index, or build on OODL without seeking permission from a protocol authority. The protocol has no owner. The mechanism is open.

---

## 15. Bitcoin Fit Assessment Summary

| Application | Consensus Change | New Trust | Available Today |
|---|---|---|---|
| Inscription Trading | No | None | Yes, natively |
| Decentralised Order Books | No | None | Yes |
| Permissionless Auctions | No | None | Yes (approximated) |
| Global Bounty Systems | No | Indexer layer | Partially |
| Sponsored Transactions | No | None | Yes, natively |
| Execution Markets | No | None | Yes, natively |
| Miner Revenue Expansion | No | None | Yes, emergent |
| Trustless OTC Markets | No | None | Yes |
| Automated Settlement | No | None | Yes |
| Cross Protocol Settlement | No | None | Yes |
| Subscription Markets | No | None | Yes |
| Reputation and Provenance | No | Indexer layer | Yes |
| Decentralised Launchpads | No | Indexer layer | Partially |
| Protocol Fee Markets | No | None | Yes |

---

## 16. Future Directions

### Standardised Intent Discovery

A common indexer protocol would allow wallets and applications to discover OODL intents across competing indexers without bespoke integrations. Standardisation increases the liquidity of the intent layer by aggregating offers from multiple sources into a unified view.

### Wallet Native Execution Tools

Current execution requires custom tooling to extract, complete, and broadcast PSBT offers. Wallet integration would make execution a native wallet feature, reducing technical barriers to participation and expanding the buyer market.

### Optimised Miner Participation

Mining pool software may be extended to monitor the OODL intent layer and automatically identify profitable execution opportunities. Formalised miner participation tooling would accelerate the development of execution markets.

### Cross Protocol Settlement Frameworks

A generalised settlement framework extending OODL mechanics to Runes, BRC-20 tokens, and future Bitcoin asset standards would create a unified trustless trading protocol across all Bitcoin-native assets.

### Decentralised Automation Markets

Intent publication by autonomous wallet software creates a foundation for decentralised automation markets. Execution specialists compete to fulfil published intents. The market coordinates without any central operator.

### Intent Aggregation and Composability

Multiple OODL intents may be combined into atomic multi-party transactions. A single broadcast could fulfil multiple published intents simultaneously, enabling complex multi-step economic agreements that settle atomically.

---

## 17. Conclusion

OODL reveals that Bitcoin transactions can represent agreements awaiting execution. This is not a metaphor. The executable agreement is embedded in the transaction itself. Execution is validated by Bitcoin consensus. Settlement is atomic and irreversible.

By embedding executable PSBT data on chain, OODL enables permissionless coordination of economic activity directly through Bitcoin validation rules. Buyers, sellers, miners, and automated systems participate without seeking permission from any central authority.

The range of application is broader than inscription trading. Every application assessed in this paper fits within existing Bitcoin protocol capabilities. No consensus changes are required. No new cryptographic primitives are needed. No trusted operators are introduced.

Bitcoin becomes capable of hosting decentralised agreements without requiring smart contract environments, without execution VMs, and without oracle networks. The agreement is published on Bitcoin. It is enforced by Bitcoin. It settles on Bitcoin.

OODL moves Bitcoin beyond Ordinals and introduces the Bitcoin Intent Layer. The intent layer is not built on top of Bitcoin. It is expressed through Bitcoin. That distinction matters.

> OODL transforms Bitcoin transactions into publicly discoverable executable agreements enforced entirely by Bitcoin validation. No smart contracts. No trusted operators. No consensus changes required. Bitcoin is already the agreement machine. OODL makes that visible.

---

## 18. Investor and Ecosystem Narrative

### 18.1 Understanding the Opportunity

Most Bitcoin innovation historically expands a single capability of the network. Bitcoin introduced decentralised money. Lightning introduced scalable payments. Ordinals introduced digital ownership. OODL introduces decentralised coordination.

The protocol does not compete with marketplaces or applications. It replaces an entire category of infrastructure. Instead of building platforms that coordinate users, OODL allows coordination itself to occur on Bitcoin. This represents infrastructure level innovation rather than application level innovation.

Infrastructure adoption compounds over time because every application built later depends on it.

### 18.2 The Missing Bitcoin Layer

Bitcoin currently operates with well understood layers. The settlement layer provides irreversible finality. The payment scaling layer enables high frequency low-value transfers. The ownership layer enables digital artifact permanence. A coordination layer has not previously existed.

Markets, escrow systems, and trading environments therefore developed outside Bitcoin, dependent on centralised operators. OODL fills this structural gap. It allows agreements to be published, discovered, and executed without introducing trusted intermediaries.

### 18.3 Why This Matters Economically

Large markets form around coordination rather than storage. Exchanges coordinate liquidity. Marketplaces coordinate buyers and sellers. Financial systems coordinate agreements between participants. Coordination generates economic gravity. When coordination becomes decentralised, entire industries restructure around the new primitive.

OODL positions Bitcoin as the place where agreements originate rather than where payments merely finalise. This is a fundamental repositioning of Bitcoin's economic role.

### 18.4 Infrastructure Versus Application Value

Applications compete with one another. Infrastructure compounds. Protocols such as TCP IP, HTTP, and Bitcoin itself created value not by owning users but by enabling entire ecosystems. OODL follows this model.

Any wallet can integrate execution. Any indexer can discover intents. Any participant can execute agreements. The protocol expands without permission or ownership concentration.

### 18.5 Market Expansion Potential

The immediate addressable market begins with Ordinals inscription trading. However the underlying coordination primitive extends far beyond collectibles. Each of the following represents an existing market currently dependent on trusted intermediaries that OODL can displace: peer to peer commerce, digital licensing, escrow agreements, auction systems, over the counter settlement, bounty markets, sponsored transactions, and Bitcoin native financial coordination.

The transition from application-level market capture to infrastructure-level market enabling is the critical distinction. OODL does not compete for a slice of any of these markets. It becomes the layer underneath all of them.

### 18.6 Ecosystem Participants and Role Diversification

OODL introduces new economic roles that did not previously exist in the Bitcoin ecosystem. Intent publishers express desired outcomes and set conditions. Executors discover and complete profitable opportunities. Wallets become execution interfaces rather than passive signing tools. Indexers become discovery engines competing for users. Miners become economic participants rather than passive transaction validators.

This diversification of economic roles strengthens the ecosystem by distributing incentives across multiple actor classes. No single participant captures all value. Each role depends on the others, creating structural alignment.

### 18.7 Miner Incentive Alignment

Long term Bitcoin security discussions frequently focus on declining block subsidies and the sustainability of fee-only miner revenue. OODL introduces an additional economic dynamic. Miners may execute profitable intents they observe on chain, capturing protocol fees in addition to standard transaction fees. Bitcoin security evolves from pure fee dependence toward execution participation.

### 18.8 Developer Adoption

Developers do not need new programming environments, new toolchains, or new cryptographic libraries. OODL uses existing Bitcoin primitives. The PSBT standard, Taproot witness flexibility, signature validation rules, and UTXO ownership guarantees are all already deployed across the Bitcoin ecosystem.

Integration complexity remains low while expressive capability increases dramatically.

### 18.9 Strategic Position Within Bitcoin

The most successful Bitcoin innovations historically share several properties: they introduce new capability without modifying consensus, they preserve self custody, they align with miner incentives, and they extend Bitcoin rather than compete with it. OODL satisfies all four conditions.

### 18.10 Competitive Landscape

Programmable chains pursue coordination through virtual machines and complex smart contract execution environments. OODL demonstrates an alternative model. Coordination emerges from transaction commitments rather than programmable computation. Bitcoin gains new expressive power without sacrificing the security assumptions that make it uniquely trustworthy.

### 18.11 Long Term Vision

The long term outcome is not a single marketplace or application. The long term outcome is a Bitcoin environment where agreements exist natively alongside payments. Users publish intentions. The network discovers opportunities. Execution becomes permissionless. Bitcoin evolves into a global coordination layer for economic activity.

### 18.12 Investment Thesis

OODL represents a protocol level opportunity rather than a product opportunity. Protocols that redefine how coordination occurs historically generate the largest ecosystems. The value accrues not through platform ownership but through ecosystem expansion. Every wallet that integrates execution, every indexer that discovers intents, every application that settles on chain, contributes to a network whose value compounds with each participant.

The protocol owns no users. It enables all of them.

---

## 19. Protocol Vision Statement

Every major Bitcoin innovation began with a question about what Bitcoin already was.

Satoshi asked whether value could be transferred without a trusted third party. The answer was Bitcoin.

The Lightning Network asked whether Bitcoin could settle at the speed of commerce. The answer was payment channels.

Ordinals asked whether Bitcoin could permanently host digital artifacts. The answer was inscriptions.

OODL asks a different question.

> Can Bitcoin host the agreement before the payment? Can the intent live on chain before the exchange occurs?

The answer is yes. It has always been yes. The mechanism existed inside Bitcoin from the beginning, encoded in the signature rules, waiting to be understood as something more than a payment tool.

OODL does not add anything to Bitcoin. It reveals something already present.

A partially signed transaction is not an incomplete payment. It is a published intention. It is an open offer, cryptographically committed, broadcast to every node on the network, available to any participant in the world who is willing to fulfil it. No permission required. No intermediary required. No trust required.

That is not a feature. That is a layer.

The world runs on coordination. Economies exist because people can make agreements, trust that those agreements will be honoured, and exchange value across time and distance. Every intermediary in the financial system exists to solve one problem: how do two parties who do not know each other make an agreement that both will keep?

Bitcoin solved that problem for payments. OODL extends the solution to agreements.

When the intent layer matures, a seller anywhere in the world can publish an economic offer directly to Bitcoin. It will be discoverable by any participant on earth without requiring a platform, a marketplace, a custody service, or an account. Any buyer who finds it and wants to fulfil it can do so directly, immediately, atomically. Bitcoin settles the exchange in one transaction. No recourse needed. No arbitration needed. No trust needed.

> Coordination without intermediaries. Agreements without operators. Settlement without escrow. This is what Bitcoin becomes when the intent layer exists.

The Bitcoin stack now has four layers. Settlement. Payments. Ownership. Coordination.

Each layer made the one above it possible. Settlement made payments trustworthy. Payments made ownership transferable. Ownership made coordination necessary. Coordination makes Bitcoin complete.

OODL is not the final chapter of Bitcoin. It is the opening of a new one.

The agreement is on chain. Execution is permissionless. The network is the marketplace. Bitcoin is the coordination layer.

**Bitcoin was always capable of this. OODL makes it visible.**

---

## 20. Protocol Specification

This section defines the canonical OODL protocol in sufficient detail for independent implementation by wallet developers, indexer operators, and execution tooling authors. Conforming implementations must satisfy all requirements described here.

### 20.1 Transaction Envelope Format

OODL data is embedded using the Ordinals inscription envelope pattern within Taproot script-path spend witness data. The script begins with OP_FALSE OP_IF followed by a protocol tag identifying the envelope as an OODL payload. The tag field contains the ASCII bytes for the string `oodl` followed by a version byte of value `0x01` for version 1 of this specification. Subsequent push data fields carry the PSBT payload. The script closes with OP_ENDIF.

Compression of the PSBT payload using the deflate algorithm is permitted. A compression flag byte immediately preceding the PSBT data indicates whether compression has been applied. A value of `0x01` indicates compressed data. A value of `0x00` indicates uncompressed data.

### 20.2 PSBT Version Requirements

OODL seller offer PSBTs must conform to BIP 174 version 0 or BIP 370 version 2. Version 2 is recommended for new implementations due to its support for explicit input sequence and output amount fields, which reduce the verification burden on execution tooling.

The seller offer PSBT must be complete and valid as a partial transaction. It must contain all seller-signed inputs with finalised input scripts. Placeholder inputs defined for buyer use must be present with empty signing data.

### 20.3 Signature Requirements

The seller must sign all inscription inputs using SIGHASH_SINGLE combined with SIGHASH_ANYONECANPAY. The sighash flag byte value for this combination under Taproot key-path spending is `0x83`. Under script-path spending the same flag applies.

The seller signature must commit to the seller payout output at the corresponding output index matching the seller input index under SIGHASH_SINGLE semantics.

### 20.4 Input and Output Ordering

The canonical OODL PSBT input ordering places buyer placeholder inputs first followed by seller inscription inputs. The canonical output ordering places buyer change output first followed by inscription receiver output followed by seller payout output.

The inscription receiver output must carry a value sufficient to satisfy the current Bitcoin dust threshold for a Taproot output. The recommended postage value is 546 satoshis.

### 20.5 Fee Routing for Indexer Detection

OODL listings are discoverable by indexers through protocol fee outputs included in the reveal transaction. The reveal transaction must include at least one output paying a non-zero amount to a designated indexing address associated with a known OODL collection registry or the OODL protocol fee address.

The minimum detectable fee output value is 546 satoshis.

### 20.6 Cancellation Procedure

A seller cancels a published OODL listing by spending the inscription UTXO referenced in the seller offer PSBT to any output the seller controls. Cancellation is implicit in UTXO consumption. Any attempt to execute against a spent UTXO will be rejected by Bitcoin validation regardless of indexer state.

### 20.7 Multi-Asset Listing Encoding

A single reveal transaction may embed multiple OODL offer PSBTs for multiple distinct inscription UTXOs. Each PSBT is encoded as a separate envelope within the same witness data block, separated by envelope boundary markers. Each listed UTXO must be independently owned by the seller and independently spendable.

### 20.8 Block Height Expiry

Optional block height expiry is encoded as a metadata field within the OODL envelope following the PSBT payload. The field key is the ASCII string `expiry` and the value is a four-byte little-endian encoded block height integer. Block height expiry is advisory metadata enforced at the indexer layer. It is not enforced at the Bitcoin consensus layer.

### 20.9 Indexer Conformance Requirements

A conforming OODL indexer must:

- Monitor the Bitcoin blockchain for reveal transactions containing valid OODL envelopes
- Validate PSBT structure and signature conformance for each detected listing
- Verify that all listed inscription UTXOs are unspent at the time of indexing
- Detect and mark listings as cancelled upon UTXO spend
- Respect block height expiry fields in all offer presentations
- Expose discovered listings through a queryable interface accessible to execution tooling and wallet applications
- Not modify PSBT data
- Not require account registration as a condition of listing discovery
- Not censor conforming listings based on content criteria not defined in this specification

---

## 21. Implementation Roadmap

### Phase 1: Reference Implementation

A minimal but complete implementation demonstrating the OODL mechanism on Bitcoin mainnet. Deliverables include a PSBT construction library implementing the seller offer template, a commit and reveal transaction builder, a UTXO monitoring tool capable of detecting cancellations, and a documented example execution on mainnet demonstrating full atomic settlement.

### Phase 2: Indexer and Discovery Layer

A conforming OODL indexer implementing the full specification. Deliverables include a blockchain scanner for OODL reveal transactions, a REST API exposing discovered listings, UTXO spend monitoring for automatic cancellation detection, and block height expiry enforcement.

### Phase 3: Wallet Integration

Integration of OODL execution capability into existing Bitcoin wallets. Deliverables include an execution SDK compatible with major Bitcoin wallet architectures, integration guides for wallet development teams, PSBT validation and safety checking tooling, and a web-based reference execution interface.

### Phase 4: Collection Registry and Ecosystem

Formalisation of the on-chain collection registry system and broader OODL ecosystem. Deliverables include a collection registry inscription standard with validation tooling, a multi-indexer aggregation framework, developer tooling for automated listing agents, and documentation for the full OODL ecosystem stack.

### Phase 5: Advanced Primitives

Research and development into advanced coordination primitives buildable on the OODL intent layer: multi-party atomic settlement combining multiple independent intents, Dutch auction mechanics, execution market tooling for mining pool integration, and cross-protocol settlement frameworks.

---

## 22. Ecosystem Adoption Strategy

### 22.1 Technical Community Engagement

The Bitcoin technical community evaluates protocols on mechanism soundness rather than marketing. The appropriate venues are the Bitcoin Development mailing list and Delving Bitcoin, where protocol-level discussion among Core contributors and serious researchers occurs.

### 22.2 Wallet Developer Outreach

Wallet integration is the critical adoption unlock. Priority wallet targets for early integration are the major Ordinals-compatible wallets. Outreach should be framed as a technical conversation. Arriving with working code rather than a specification document changes the nature of the conversation.

### 22.3 Indexer and Infrastructure Ecosystem

Multiple competing indexers are desirable as a decentralisation property. Early ecosystem strategy should actively encourage independent indexer development by making the indexer conformance specification freely available, publishing test vectors for validation, and recognising conforming indexers publicly.

### 22.4 Public Communication Strategy

The narrative frame for all public communication is not a new marketplace. The frame is a new Bitcoin layer. This is a more significant claim, it is also the accurate one, and it reaches a different and more valuable audience than application-level product announcements.

### 22.5 Capital Formation

The investor conversation is more productive once ecosystem evidence exists alongside the white paper. Bitcoin-native investors with deep technical backgrounds will engage seriously with OODL given a live implementation, wallet team interest, and a functioning indexer API.

### 22.6 Protocol Ownership and Governance

OODL is defined by this specification. The specification is versioned. No central authority owns the protocol. No permission is required to implement a conforming indexer, wallet integration, or execution tool. Protocol improvement proposals may be submitted by any participant.

---

## Appendix A: Glossary

**Atomic Settlement** — A transaction outcome in which all intended transfers either complete together in a single confirmed transaction or none complete.

**ANYONECANPAY** — A Bitcoin sighash flag modifier that restricts a signature's commitment to a single input, allowing unknown additional inputs to be added by future participants without invalidating the original signature.

**Buy Transaction** — The third transaction in the OODL lifecycle, constructed by an executor who extracts the seller offer PSBT, adds buyer funding inputs, and broadcasts the completed transaction.

**Commit Transaction** — The first transaction in the OODL lifecycle, preparing execution state by aligning the inscription to sat offset zero and establishing protocol fee routing.

**Execution** — The act of completing and broadcasting a buy transaction that fulfils a published OODL intent.

**Executor** — A participant who discovers a published OODL intent and constructs and broadcasts the corresponding buy transaction.

**Indexer** — A software service that monitors the Bitcoin blockchain for OODL reveal transactions, validates and stores discovered listings, monitors listed UTXOs for spend events, and exposes listings through a queryable API.

**Intent** — An executable transaction commitment that defines conditions for completion while remaining incomplete. In OODL, a seller offer PSBT embedded on chain in a reveal transaction.

**Intent Layer** — The conceptual Bitcoin layer introduced by OODL in which executable agreements exist on chain alongside payments and ownership records.

**PSBT** — Partially Signed Bitcoin Transaction. A standardised format defined in BIP 174 and extended in BIP 370 for representing Bitcoin transactions that are not yet fully signed.

**Reveal Transaction** — The second transaction in the OODL lifecycle, embedding the seller offer PSBT in witness data and publishing the executable intent to the Bitcoin blockchain.

**Seller Offer PSBT** — A partially signed Bitcoin transaction constructed by the seller. The seller signs their inscription input using SIGHASH_SINGLE and ANYONECANPAY, committing to their payout output while leaving buyer input positions open.

**SIGHASH_SINGLE** — A Bitcoin sighash type that restricts a signature's commitment to the input being signed and the output at the corresponding index.

**Taproot** — A Bitcoin protocol upgrade activated in November 2021 implementing BIP 340, BIP 341, and BIP 342, enabling Schnorr signatures and Tapscript.

**UTXO** — Unspent Transaction Output. The fundamental unit of value in Bitcoin. OODL listings reference specific inscription UTXOs. A spent UTXO invalidates any OODL listing referencing it.

**Witness Data** — Segregated witness data attached to Bitcoin transaction inputs, introduced by SegWit in 2017 and extended by Taproot in 2021. OODL offer payloads are embedded in witness data.

---

## Appendix B: Technical Reference

### B.1 SIGHASH Flag Values

- `0x01` — SIGHASH_ALL. Commits to all inputs and all outputs.
- `0x02` — SIGHASH_NONE. Commits to all inputs and no outputs.
- `0x03` — SIGHASH_SINGLE. Commits to all inputs and the output at the matching index.
- `0x81` — SIGHASH_ALL combined with SIGHASH_ANYONECANPAY.
- `0x83` — SIGHASH_SINGLE combined with SIGHASH_ANYONECANPAY. Used by OODL seller signatures.

### B.2 OODL Envelope Script Structure

```
OP_FALSE
OP_IF
  PUSH("oodl")
  PUSH(version_byte)
  PUSH("Content-Type")
  PUSH("application/psbt")
  PUSH(compression_flag)
  PUSH(psbt_payload_chunk_1)
  [PUSH(psbt_payload_chunk_n) ...]
  [PUSH("expiry") PUSH(block_height_bytes)]
OP_ENDIF
```

Version byte for this specification: `0x01`. Compression flag: `0x01` for deflate-compressed, `0x00` for uncompressed. PSBT payload may be split across multiple push data operations if it exceeds the maximum push data size of 520 bytes per push.

### B.3 Transaction Structure

**Commit Transaction**
- Funding inputs from seller wallet
- Taproot output carrying inscription at sat offset zero (546 sats)
- Protocol fee output to designated indexing address (546 sats minimum)
- Change output to seller if required

**Reveal Transaction**
- Input spending the commit Taproot output via script-path spend with OODL envelope in witness
- Output returning inscription Taproot UTXO to seller control
- Witness data contains serialised seller offer PSBT in OODL envelope format

**Buy Transaction**
- Input 1: buyer funding input
- Input 2: optional additional buyer funding input
- Input 3: seller inscription UTXO from reveal transaction
- Output 1: buyer change
- Output 2: inscription receiver at buyer destination (546 sats postage)
- Output 3: seller payout at seller address with agreed sale price

### B.4 Executor Validation Checklist

Before adding buyer inputs and broadcasting, conforming execution tooling must verify:

1. The seller offer PSBT parses without error under BIP 174 or BIP 370.
2. The seller input UTXO is present and unspent in the current UTXO set.
3. The seller signature validates against the seller input using SIGHASH_SINGLE and SIGHASH_ANYONECANPAY.
4. The seller payout output is present at the index corresponding to the seller input and carries the expected sale price.
5. The inscription receiver output is present and carries sufficient postage value above the dust threshold.
6. The listing has not exceeded its block height expiry if an expiry field is present.
7. The completed transaction with buyer inputs added does not exceed the Bitcoin maximum transaction size of 400,000 weight units.
8. The total fee rate of the completed transaction is within acceptable parameters for timely confirmation.

### B.5 Indexer API Recommended Fields

A conforming OODL indexer API response for a single listing should include:

- `listing_id` — Unique identifier derived from reveal transaction identifier and envelope index.
- `inscription_id` — Ordinals inscription identifier in standard txid:vout format.
- `seller_address` — Bitcoin address to which the seller payout output is committed.
- `sale_price_sats` — Sale price in satoshis as committed in the seller payout output.
- `psbt_hex` — Serialised seller offer PSBT as a hex-encoded string.
- `reveal_txid` — Transaction identifier of the reveal transaction containing this listing.
- `status` — One of `active`, `cancelled`, or `executed`.
- `expiry_block` — Block height expiry if present, or null if absent.
- `detected_at_block` — Block height at which the indexer detected this listing.
- `collection_id` — Parent collection inscription identifier if applicable, or null.

---

## Appendix C: References

### Bitcoin Improvement Proposals

1. BIP 11: M-of-N Standard Transactions. Gavin Andresen. 2011.
2. BIP 65: OP_CHECKLOCKTIMEVERIFY. Peter Todd. 2014.
3. BIP 68: Relative lock-time using consensus-enforced sequence numbers. Mark Friedenbach, BtcDrak, Nicolas Dorier, kinoshitajona. 2015.
4. BIP 141: Segregated Witness. Eric Lombrozo, Johnson Lau, Pieter Wuille. 2015.
5. BIP 174: Partially Signed Bitcoin Transaction Format. Andrew Chow. 2017.
6. BIP 340: Schnorr Signatures for secp256k1. Pieter Wuille, Jonas Nick, Tim Ruffing. 2020.
7. BIP 341: Taproot: SegWit version 1 spending rules. Pieter Wuille, Jonas Nick, Anthony Towns. 2020.
8. BIP 342: Validation of Taproot Scripts. Pieter Wuille, Jonas Nick, Anthony Towns. 2020.
9. BIP 370: PSBT Version 2. Andrew Chow. 2020.

### Related Work

1. Casey Rodarmor. Ordinals. 2022.
2. Joseph Poon and Thaddeus Dryja. The Bitcoin Lightning Network: Scalable Off-Chain Instant Payments. 2016.
3. Nicolas Dorier. Discreet Log Contracts. 2018.

---

*OODL Protocol · Version 1.0 · Author: switch-900 · oodinals.com*
