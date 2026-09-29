# SPEC-003: Interaction Receipt Protocol

## 1. Context and Objective

The Interaction Receipt is the central immutable fact of RepID: an NFT that can only exist if both parties of an interaction signed it jointly. It acts as an **anti-corruption layer** between the application-specific interaction protocols (RFC-002) and the RepID core, preventing external business logic from contaminating the base protocol.

## 2. User Stories

- As a pair of participants in an interaction, I want to jointly sign an immutable receipt so that it is recorded as a verifiable fact on the chain.
- As a participant, I want to receive a Rating Right at the moment the receipt is created, so I can rate the other party later without relying on any additional third-party action.

## 3. Functional Requirements (EARS Syntax)

- **RF-01** (Ubiquity): The system must mint an Interaction Receipt NFT only when both parties jointly sign the genesis transaction.
- **RF-02** (Ubiquity): The system must mint, in the same genesis transaction, two Rating Right UTXOs, one per party.
- **RF-03** (Ubiquity): The system must fix participation at exactly two parties per receipt.
- **RF-04** (Ubiquity): The system must lock the Interaction Receipt to `partyA`; this rule is enforced by the covenant (`ReceiptGenesisValidator`), not mere application convention.
- **RF-05** (Ubiquity): The system must act as an anti-corruption layer, without inheriting application-specific business logic into the Receipt.
- **RF-06** (Events): When a validating entity (platform/application) wants to corroborate that an interaction occurred, the system must record a confirmation as an independent on-chain fact (separate attestation), without modifying the Receipt's genesis transaction.
- **RF-07** (Ubiquity): The Receipt's genesis transaction must include a P2PKH change output to `partyA` (the party that funds the genesis); the change may not carry tokens minted in that same transaction. This enables spending on the real network (Chipnet) without losing the funding UTXO's value in fees.

## 4. Non-Functional Requirements

- The joint signature must be verified at the covenant level (CashScript), not delegated to off-chain validation.
- No commit-reveal scheme is required: the Receipt is signed before any rating exists, so there is no sensitive information to protect at that point.

## 5. Edge Cases and Constraints

**Design decisions:**
- RF-03: participation is fixed at exactly 2 parties per receipt. Group interactions are modeled as multiple pairwise receipts (one per pair) at the application layer. Extension to N parties is out of scope.
- RF-04: locking the Receipt to `partyA` is a rule imposed by the covenant (`ReceiptGenesisValidator`), not an application convention. The only remaining convention is that the application defines who `partyA` is (RFC-002).

- Missing signature from either of the two parties → the genesis transaction is invalid and nothing is minted (neither Receipt nor Rating Rights).
- Confirmation of an interaction by a third party (platform) → separate attestation with a P2PKH spend + `OP_RETURN` (pattern analogous to `ISSUED_RATING`), not a covenant extension. The confirming entity spends its own UTXO with a protocol tag and a reference to the already-indexed Receipt's txid.

**Design decision (platform confirmation, "pattern C"):**
- The validator does not need to mint its own Identity: its pkh suffices (P2PKH spend of the UTXO it signs).
- The Indexer recognizes `PLATFORM_CONFIRMATION` only if the referenced Receipt was already indexed (traceability). If an unknown Receipt is referenced, the fact is recognized but marked `valid: false` (it is not silently ignored).

## 6. Out of Scope

- Commit-reveal scheme for ratings (unnecessary for the MVP; see Non-Functional Requirements).
- Multi-party receipts (>2). Group interactions are decomposed into pairwise receipts at the application layer.
- Modifying the genesis transaction to include the validator: platform confirmation is always a separate fact (RF-06), never added to the Receipt's genesis.

## 7. Acceptance Criteria (Definition of Done)

- [x] The genesis transaction requires the joint signature of both parties.
- [x] Two Rating Rights are minted together with the Receipt in a single transaction.
- [x] The covenant requires the P2PKH change output to `partyA` and rejects hidden mintings (10 `ReceiptGenesisValidator` tests, RF-07).
- [x] Design decisions in section 5 confirmed by the project architect (TASK-001, TASK-002).
- [x] `PLATFORM_CONFIRMATION` (RF-06) covered by indexer tests (TASK-016).
- [ ] The code complies with `constitution.md`.