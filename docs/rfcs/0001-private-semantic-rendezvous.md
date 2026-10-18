# PPSR

### Privacy-preserving semantic rendezvous

**private intent, mutual discovery**

**Public Research Proposal · RFC-0001 · Draft v0.3 · October 2026**

> Discover semantically compatible participants without disclosing private intentions to a discovery operator. Establish contact only through mutual authorization.

**Status.** This document specifies a research objective, a reference architecture, and open questions. It does not provide a cryptographic implementation, performance results, a security proof, or an independent audit. It is not suitable for handling real high-sensitivity information. “RFC” denotes this project's request for comments, not endorsement by a standards organization.

---

## 1. Abstract

PPSR investigates an interpersonal discovery protocol combining **on-device semantic encoding, privacy-preserving computation, and mutual consent**.

Participants may disclose a coarse role, such as **Provider** or **Seeker**, while keeping detailed product descriptions, service descriptions, requirements, and contact information on their own devices. A shared, versioned encoding profile transforms free-form text into semantic representations. Clients then submit cryptographically protected matching inputs. The computation layer evaluates compatibility without requiring the discovery operator to read the source text or complete plaintext representations.

A successful comparison produces only a restricted contact opportunity. Following independent approval by both participants, clients exchange explicitly authorized contact information and transition to a separate end-to-end encrypted communication channel.

**PPSR concerns private discovery before contact. It is not a transaction-execution, payment-settlement, or messaging protocol.**

The proposal studies a composition of existing techniques. It does not claim a new cryptographic primitive, absolute anonymity, or zero information leakage.

## 2. Minimal Example

Let `e` denote a lawful product or service that neither participant wishes to discuss publicly.

| Attribute | User A | User B |
|---|---|---|
| Public role `r` | Provider | Seeker |
| Local private intention `m` | “I provide product or service e.” | “I am seeking product or service e.” |
| Local semantic representation | `e(A)` | `e(B)` |
| Protected matching input | `e'(A)` | `e'(B)` |
| Publicly disclosed product information | None | None |
| Intended outcome | Independently decide whether to connect with B | Independently decide whether to connect with A |

The object `e` and the encoded representations `e(A)` and `e(B)` are distinct. The notation does not identify a product with a vector.

Participants disclose a **role, not a real-world identity**. A role, publication time, and other observable metadata nevertheless remain part of the leakage surface.

The two descriptions need not be identical, and participants need not select a product from a fixed catalog. The objective is to evaluate semantic compatibility, not merely string equality.

## 3. System Architecture

The diagram below describes the intended information flow. **Protected** refers to an input representation produced by a selected cryptographic profile; it does not necessarily mean an ordinary public-key ciphertext. Candidate discovery, private output delivery, and authorization binding still require concrete protocol definitions.

```text
                            PPSR / PRIVATE SEMANTIC RENDEZVOUS

+-----------------------------------------+     +-----------------------------------------+
| USER A                                  |     | USER B                                  |
| Public role: PROVIDER                   |     | Public role: SEEKER                     |
| Private m_A: "I provide e."             |     | Private m_B: "I seek e."                |
| Contact payload: local only             |     | Contact payload: local only             |
+-----------------------------------------+     +-----------------------------------------+
                     |                                               |
                     v                                               v
+-----------------------------------------+     +-----------------------------------------+
| 01 / LOCAL ENCODING                     |     | 01 / LOCAL ENCODING                     |
| Versioned encoder E_v                   |     | Same compatible encoder E_v             |
| Text -> semantic vector                 |     | Text -> semantic vector                 |
| Optional private constraints            |     | Optional private constraints            |
| Output: e(A)                            |     | Output: e(B)                            |
+-----------------------------------------+     +-----------------------------------------+
                     |                                               |
                     v                                               v
+-----------------------------------------+     +-----------------------------------------+
| 02 / INPUT PROTECTION                   |     | 02 / INPUT PROTECTION                   |
| Protect_profile(e(A); rho_A)            |     | Protect_profile(e(B); rho_B)            |
| Output: e'(A)                           |     | Output: e'(B)                           |
+-----------------------------------------+     +-----------------------------------------+
                     |                                               |
================================== CLIENT TRUST BOUNDARY ==================================
                     |                                               |
                     +-----------------------+-----------------------+
                                             |
                                             v
                 +-------------------------------------------------------+
                 | 03 / SIGNAL RELAY                                     |
                 | Route opaque protected inputs                         |
                 | Public: role, versions, expiry, temporary route IDs   |
                 | No source text or plaintext vectors                   |
                 +-------------------------------------------------------+
                                             |
                                             v
                 +-------------------------------------------------------+
                 | 04 / CANDIDATE SCHEDULING                             |
                 | Compatible roles and protocol versions                |
                 | Bounded pool / fixed matching rounds                  |
                 | No public semantic index                              |
                 | Scalable private discovery: OPEN                      |
                 +-------------------------------------------------------+
                                             |
                                             v
                 +-------------------------------------------------------+
                 | 05 / PRIVATE EVALUATION                               |
                 | Inputs: protected e(A), e(B)                          |
                 | Internal score: p                                     |
                 | Predicate: role_ok AND p >= tau AND constraints_ok    |
                 | Do not open full inputs or the exact score            |
                 +-------------------------------------------------------+
                                             |
                     +-----------------------+-----------------------+
                     |                                               |
+-----------------------------------------+     +-----------------------------------------+
| NO MATCH                                |     | MATCH / 06 PRIVATE OUTPUT               |
| No contact released                     |     | Restricted, pair-specific invitations   |
| Wait / expire                           |     | No public score or ranking              |
| No per-target response                  |     |                                         |
+-----------------------------------------+     +-----------------------------------------+
                                                                     |
                                             +-----------------------+
                                             |
                                             v
                 +-------------------------------------------------------+
                 | 07 / INDEPENDENT CONSENT                              |
                 | User A: accept / decline                              |
                 | User B: accept / decline                              |
                 | Both approvals bound to this session                  |
                 +-------------------------------------------------------+
                                             |
                                             v
                 +-------------------------------------------------------+
                 | BOTH CONSENT AND SESSION STILL VALID?                 |
                 +-------------------------------------------------------+
                                             |
                     +-----------------------+-----------------------+
                     |                                               |
+-----------------------------------------+     +-----------------------------------------+
| NO / EXPIRED                            |     | YES / 08 CONTACT HANDOFF                |
| No contact release                      |     | Authenticate session keys               |
|                                         |     | Encrypt approved contact payloads       |
|                                         |     | No private intention is disclosed       |
+-----------------------------------------+     +-----------------------------------------+
                                                                     |
                                             +-----------------------+
                                             |
                                             v
                 +-------------------------------------------------------+
                 | EXTERNAL END-TO-END ENCRYPTED COMMUNICATION           |
                 | Direct or relayed; outside the matching layer         |
                 +-------------------------------------------------------+
```

The no-match branch does **not** prove that a failed comparison is unobservable. Notification presence, timing, and subsequent access patterns may reveal information and must be included in the leakage analysis. The branch labels describe the intended functionality, not a branch that the operator is necessarily permitted to observe.

## 4. Local Semantic Profile

### 4.1 Free-form input rather than a fixed tag catalog

Users author natural-language intentions. An on-device model produces a semantic vector for comparison. Optional tags may support local explanation and constraint confirmation, but different participants are not required to generate identical tag strings.

Define:

$$
e(i)=E_v(m_i,r_i)=(z_i,h_i)
$$

Here, `m_i` is the private source text; `r_i` is the disclosed role; `E_v` is the versioned encoding procedure; `z_i` is a normalized semantic vector; and `h_i` contains optional private hard constraints.

Sentence-vector similarity has an established research basis.[^sbert] However, nearby vectors do not establish transaction compatibility. “Offering oranges” and “seeking orange seeds,” for example, may be topically close while referring to different goods.

### 4.2 A shared profile is more than a shared prompt

Compatible clients should fix the following within each profile:

**Model weights, tokenizer, preprocessing, role instructions, truncation rules, output dimensions, normalization, and quantization.**

Each profile should identify its version and include interoperability test vectors. A shared prompt does not make representations from different models comparable. Even with the same model, numerical differences across devices and inference backends require testing. Representations from different profile versions must not be compared without a validated compatibility procedure.

Private source text and plaintext features should not be sent to remote model APIs, telemetry services, or error logs. Existing client-side inference frameworks can support local encoding.[^transformersjs]

**Local encoding does not complete the privacy protection.** Text embeddings can expose sensitive information through inversion and related analysis; they must remain protected inputs, not public substitutes for the source text.[^inversion]

## 5. Matching Function

For normalized vectors, the reference similarity is:

$$
s(A,B)=z_A^{\top}z_B\in[-1,1]
$$

A rescaled score may be defined as:

$$
p(A,B)=\frac{1+s(A,B)}{2}\in[0,1]
$$

The candidate compatibility predicate is:

$$
b=\mathrm{RoleCompatible}(r_A,r_B)
\land[p(A,B)\geq\tau_v]
\land C(h_A,h_B)
$$

`C` evaluates the compatibility of declared hard constraints. When no constraints are configured, this term is true; in that case, the result concerns semantic proximity and does not establish that a transaction is feasible.

**`p` is a score, not a calibrated probability that the participants refer to the same item.** A threshold of `0.95` is illustrative only. Under the rescaling above, it corresponds to cosine similarity of at least `0.90`. Operational thresholds require evaluation against the intended use case.

The computation layer should evaluate both the score and the threshold predicate over protected inputs. By default, neither the operator nor a participant receives the exact score. Only requests satisfying the publication and query policies may produce a restricted invitation.

Hiding a plaintext score in the user interface is not cryptographic output control. Numerical encodings, input validation, comparison precision, and boundary behavior must be specified by the eventual computation profile.

These equations define a desired functionality, **not an implemented or proven protocol**.

## 6. Cryptographic Profile

### 6.1 Protected inputs are not ordinary asymmetric encryption

Write:

$$
e'(i)=\mathrm{Protect}_{\mathrm{profile}}(e(i);\rho_i)
$$

`rho_i` denotes fresh randomness. `Protect` must be defined by a concrete cryptographic profile; a conventional `Encrypt()` operation cannot substitute for the entire private matching mechanism.

Encryption is not “irreversible to everyone.” A party with the relevant decryption capability can recover the information allowed by that capability. Identical inputs may produce different ciphertexts, but randomization does not eliminate leakage from matching outputs, message lengths, or network metadata.

| Mechanism | Intended role in PPSR | What it does not independently provide |
|---|---|---|
| Conventional public-key or hybrid encryption | Protect transport, key material, and contact payloads | Semantic comparison of arbitrary ciphertexts |
| MPC / 2PC | Evaluate scores and predicates over private inputs | Elimination of inference from authorized outputs |
| Homomorphic encryption (HE) | Evaluate supported operations on ciphertexts in a compatible key system | Automatic resolution of independent-user keys, private thresholding, or output authorization |
| Zero-knowledge proofs (ZKPs) | Prove a precisely defined input-validity or computation relation | Semantic understanding or proof of genuine transaction intent |

Homomorphic evaluation generally produces an encrypted result. CKKS supports approximate numerical operations, but comparison and multi-user key management require additional design.[^seal]

**ZKPs may verify a defined relation; they are not interchangeable with MPC or HE as a complete “encrypted similarity engine.”**

### 6.2 Candidate reference profile: two-operator MPC

A client first quantizes its features and splits them into secret shares. It then independently protects and submits the shares to two separately operated computation nodes.

```text
                         CLIENT i
                  Local vector / constraints
                              |
                     Secret-share input
                              |
                 +------------+------------+
                 |                         |
             Share i,1                 Share i,2
                 |                         |
       Encrypt to Node 1         Encrypt to Node 2
                 |                         |
                 v                         v
       +------------------+      +------------------+
       | COMPUTE NODE 1   |<---->| COMPUTE NODE 2   |
       | Operator 1       | MPC  | Operator 2       |
       | Its own shares   |      | Its own shares   |
       +------------------+      +------------------+
                 \                         /
                  +-----------------------+
                  | Protected score       |
                  | Protected threshold   |
                  | Private output to     |
                  | authorized clients    |
                  +-----------------------+
```

Each node may decrypt the transport envelope addressed to it, obtaining only its own share. The full vector or intention should not be reconstructed during evaluation.

The accurate description of this profile is therefore:

> **Private evaluation without reconstructing complete private inputs.**

It is not:

> No ciphertext is ever decrypted at any stage.

The candidate implementation must select a protocol appropriate to its malicious-participant model and specify input validation, authentication, private output delivery, and abort behavior. MP-SPDZ offers implementations under different security models, but using that framework does not automatically establish PPSR's required guarantees.[^mpspdz]

In this two-operator configuration, non-collusion is a central assumption. If one operator can obtain both complete shares, the input-confidentiality objective fails. A malicious node may also abort execution: confidentiality does not imply availability.

`e'(i)` in this configuration denotes a protected, distributed input bundle, not one publicly readable set of shares. Publishing all shares together would defeat secret sharing. A relay may forward separately encrypted envelopes, but must not obtain the full reconstructed input.

HE remains an alternative research direction. Ordinary ciphertexts under unrelated user keys cannot simply be mixed as if they shared a compatible evaluation key. Nor should users receive a common private key capable of decrypting other users' inputs. Key establishment, evaluation, and authorized output release require an explicit construction.

## 7. Consent and Contact Handoff

**A match must not automatically decrypt or disclose an intention.** The intended application rule is:

$$
\mathrm{ReleaseContact}(A,B)
\iff (b=1)\land\mathrm{consent}_A\land\mathrm{consent}_B\land\mathrm{valid}
$$

`valid` means the session, expiration conditions, and authorizations remain valid. This is a proposed release policy, not a claim that a fair-exchange protocol has been implemented.

Each invitation should use a distinct session identifier. Confirmation messages should bind that session to the relevant published signals, recipient keys, and expiration conditions to prevent replay, cross-session reuse, and key substitution. The exact authentication and binding mechanisms remain implementation requirements.

After both approvals, each client encrypts and transmits only the contact information explicitly approved by its user. It does not disclose source text, semantic vectors, or hidden constraints.

HPKE is one candidate for encryption encapsulation, but it does not by itself supply a complete session-authentication, general replay-protection, or mutual-consent protocol.[^hpke]

**End-to-end encryption is not network anonymity, and a direct peer-to-peer connection does not inherently hide IP addresses.** Subsequent communication may be direct or relayed. The absence of a relay is not a necessary condition for confidentiality.

The proposed handoff does not guarantee simultaneous receipt by both participants. It cannot retract information that a recipient has already read, copied, or disclosed.

## 8. Privacy and Decentralization Model

### 8.1 Protected information and permitted leakage

The objective is to protect source text, complete semantic representations, and unauthorized contact payloads under the selected cryptographic assumptions and corruption bound.

Public or observable information may include coarse roles, protocol versions, message sizes, publication times, node connections, and access behavior. An authorized invitation also reveals that a potential counterpart satisfies some matching conditions.

A future formal security definition must explicitly specify each party's leakage function. “No one learns anything” is not an adequate security definition for a useful discovery protocol.

### 8.2 Decentralization as a design objective

Local AI, encrypted storage, and multiple servers do not independently establish decentralization.

PPSR aims to support replaceable clients and relays, with independently operated computation parties. The project operator should not hold universal decryption capability. Software updates, operator admission, and key configuration must also have explicit control and trust models.

This RFC does not solve permissionless node admission, organizational resistance to collusion, or global traffic-analysis resistance. It does not require a blockchain or propose permanently publishing private intentions or relationship graphs on a public ledger.

### 8.3 Meaning of an open protocol

An open protocol is a public specification of semantic encoding, cryptographic parameters, message formats, authorization states, and leakage boundaries. **It is not merely a published prompt.**

Independent developers should be able to implement compatible clients and verify interoperability using shared test vectors. This document does not yet define a stable wire format or production cryptographic suite.

## 9. Threat Model and Known Limitations

### 9.1 Adaptive probing

An adversary can pretend to seek product `e` and use matching feedback to infer whether another participant offers it. Randomized encryption, MPC, and ZKPs do not remove the information contained in legitimate outputs. Existing private-computation projects explicitly discuss additional inference from repeated, related queries.[^pjc]

PPSR proposes to evaluate restrictions on named-target queries, exact scores, and public result lists, together with bounded matching rounds, independent consent, anonymous eligibility, and query budgets.

These are candidate mitigations. **They neither establish genuine purchase intent nor eliminate impersonation, phishing, or probing.** Their effectiveness must be measured, including resistance to multiple identities.

### 9.2 Semantic errors and fabricated inputs

Shared subject matter does not establish compatibility. Negation, product subtypes, price, time, and role-specific conditions require evaluation.

A malicious client may submit a constructed vector without running the specified encoder. A version declaration does not prove correct inference. Even a proof of correct encoding would not establish that the source statement is true, that a product exists, or that its author is honest.

### 9.3 Candidate discovery and scale

Privately comparing an already selected A and B is different from privately finding B in a large network. Naive cross-role comparison has approximate computational cost:

$$
O(N_{\mathrm{provider}}\times N_{\mathrm{seeker}}\times d)
$$

The reference experiment uses bounded pools. Any future index or prefilter must account for leakage of categories, candidate sets, and access patterns. Compass provides relevant research on encrypted semantic search, but it is not a complete decentralized discovery system for previously unknown participants.[^compass]

### 9.4 Endpoints, updates, and revocation

Malicious client updates, logs, and compromised endpoints can expose plaintext. Revocation can restrict future protocol actions but cannot guarantee deletion of data already copied by another party.

Replay, denial of service, node collusion, notification behavior, and key substitution require separate analysis. Encryption of message content alone does not address these threats.

## 10. Evaluation Plan

| Stage | Experiment | Evidence to report |
|---|---|---|
| **E0 — Semantic baseline** | Equivalent descriptions, cross-language pairs, and hard negatives | Precision, recall, error taxonomy, and threshold-selection method |
| **E1 — Protected computation** | The same inputs and function as the plaintext baseline | Numerical error, threshold flips, runtime, bandwidth, and memory |
| **E2 — Adversarial evaluation** | Fabricated inputs, repeated queries, multiple identities, and modified authorizations | Inferable information, attack cost, and residual risks |
| **E3 — Asynchronous discovery** | Offline publication, later arrivals, expiry, and node failures | Notification correctness, metadata leakage, and availability |
| **E4 — Independent review** | The selected construction and implementation | Threat-model assessment, security analysis, and external review findings |

No matching-accuracy, throughput, or security-level results are reported in this draft. **“95%” is an illustrative score threshold, not an experimental result.**

Early prototypes should use synthetic or appropriately consented, non-sensitive evaluation material rather than real high-sensitivity intentions.

## 11. Related Work and Research Position

**Matchmaking Encryption** studies disclosure governed by matching private policies.[^matchmaking] **Compass** investigates encrypted semantic retrieval and access-pattern protection.[^compass] **Index Network** provides an adjacent, intent-driven social discovery direction.[^index]

PPSR's proposed scope is the composition of:

> **Free-form local encoding → protected cross-user comparison → asynchronous candidate discovery → minimal output disclosure → mutual contact authorization.**

The existence of these individual techniques does not establish that the composition has been implemented or proven secure. PPSR makes no first-of-its-kind claim.

## 12. Request for Comments

Technical review is invited on the target functionality, leakage model, private output construction, semantic reliability, adaptive-query resistance, and asynchronous discovery. Reproducible counterexamples and relevant prior work are particularly valuable.

The central research question is:

> **Under explicit security assumptions and restricted disclosure rules, can two strangers discover semantic compatibility without requiring a discovery operator to read their private intentions?**

This project does not promise absolute anonymity, untraceability, or downstream transaction safety. It does not issue a token, provide settlement, or establish the truthfulness of participant statements. Its intended scope is lawful, privacy-sensitive discovery.

---

## References

[^sbert]: Nils Reimers and Iryna Gurevych. *Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks.* EMNLP-IJCNLP 2019. <https://arxiv.org/abs/1908.10084>

[^transformersjs]: Hugging Face. *Transformers.js documentation.* <https://huggingface.co/docs/transformers.js/index>

[^inversion]: Morris et al. *Text Embeddings Reveal (Almost) As Much As Text.* EMNLP 2023. <https://aclanthology.org/2023.emnlp-main.765/>

[^seal]: Microsoft. *SEAL: Homomorphic Encryption Library and Documentation.* <https://github.com/microsoft/SEAL>

[^mpspdz]: *MP-SPDZ documentation.* <https://mp-spdz.readthedocs.io/en/latest/readme.html>

[^hpke]: IETF. *RFC 9180: Hybrid Public Key Encryption.* 2022. <https://www.rfc-editor.org/rfc/rfc9180.html>

[^pjc]: Google. *Private Join and Compute — Security and Privacy Considerations.* <https://github.com/google/private-join-and-compute>

[^compass]: Zhu et al. *Compass: Encrypted Semantic Search with High Accuracy.* OSDI 2025. <https://www.usenix.org/conference/osdi25/presentation/zhu-jinhao>

[^matchmaking]: Ateniese et al. *Match Me if You Can: Matchmaking Encryption and Its Applications.* CRYPTO 2019; extended version available through IACR ePrint. <https://eprint.iacr.org/2018/1094>

[^index]: *Index Network — Social Discovery Protocol.* <https://github.com/indexnetwork/index>
