# SPEC-004: RepID Rating Protocol

## 1. Context and Objective

Once an Interaction Receipt has been issued, each party holds a Rating Right: the right to rate the other party exactly once. The protocol leverages the single-spend semantics of a UTXO to guarantee "one rating per participant" without additional covenant logic.

## 2. User Stories

- As a participant in an interaction, I want to rate the other party using my Rating Right, to leave a verifiable reputation fact on the chain.
- As a protocol observer, I want to be able to trust that nobody can rate twice for the same interaction.

## 3. Functional Requirements (EARS Syntax)

- **RF-01** (Ubiquity): The system must represent each Rating Right as a single-use UTXO.
- **RF-02** (Events): When the owner of a Rating Right spends the UTXO, the system must include an `OP_RETURN` payload with the rating (integer score between 1 and 5).
- **RF-03** (Ubiquity): The system must implicitly burn the Rating Right NFT upon spend, guaranteeing that only one rating can be issued per participant.
- **RF-04** (Undesired Behavior): If the score included in the `OP_RETURN` is outside the 1–5 range, then the system must consider the transaction invalid.
- **RF-05** (Ubiquity): The system must record the Rating Right commitment using the owner's pkh, instead of the full Identity NFT category.

## 4. Non-Functional Requirements

- The Rating Right spend is implemented as a standard P2PKH spend — it requires no dedicated covenant, which simplifies the implementation (see `constitution.md`, Article 2).
- The Indexer must be able to detect the `OP_RETURN` and decode the score unambiguously.

## 5. Edge Cases and Constraints

**Design decision:**
- RF-05: the Rating Right commitment uses the owner's pkh (not the full Identity NFT category). The pkh → Identity link (RFC-001) is resolved in a later, off-chain layer (per Article 1 of the Constitution).

- `addOpReturnOutput` treats strings as UTF-8 unless they are prefixed with `"0x"` — a risk of encoding the score incorrectly if the prefix is not used.
- A score outside the 1–5 range is recognized but marked `valid: false` by the recognizer; it is not silently ignored (RF-04, and enforced by the conditional in `protocol/schemas/repid-fact.schema.json`).

## 6. Out of Scope

- On-chain reputation aggregation algorithms (composite score computation, averages, weights).
- Disputes or challenges to an already-issued rating.
- Ratings with free text or comments (only the numeric score 1–5 is admitted in this phase).

## 7. Acceptance Criteria (Definition of Done)

- [x] `ISSUED_RATING` implemented: P2PKH spend with a score `OP_RETURN`, implicit NFT burn.
- [x] Explicit score-range validation (RF-04) covered by a test (scores 0, 6 and 200 marked invalid; TASK-008).
- [x] Section 5 assumption confirmed by the project architect: commitment with the owner's pkh (TASK-003).
- [ ] The code complies with `constitution.md`.