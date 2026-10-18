# Research Roadmap

This roadmap organizes the evaluation plan in [RFC-0001](rfcs/0001-private-semantic-rendezvous.md). All stages below are proposed; this repository reports no completed experiments or measured results.

| Stage | Experiment | Evidence to report |
|---|---|---|
| E0 · Semantic baseline | Equivalent descriptions, multilingual pairs, hard negatives, and declared constraints | Precision, recall, error taxonomy, and threshold-selection method |
| E1 · Protected computation | Evaluate the same inputs and matching function as E0 | Numerical error, threshold flips, runtime, bandwidth, and memory |
| E2 · Adversarial evaluation | Fabricated inputs, repeated queries, multiple identities, and modified authorizations | Inferable information, attack cost, and residual risks |
| E3 · Asynchronous discovery | Offline publication, later arrivals, expiry, and node failures | Notification correctness, metadata leakage, and availability |
| E4 · Independent review | Review the selected construction and implementation | Threat-model assessment, security analysis, and external review findings |

## Decisions required before implementation

- Define the encoding profile, interoperability vectors, quantization, and compatibility rules.
- Specify the corruption bound, non-collusion assumption, and each party's permitted leakage.
- Select a concrete MPC protocol and define input validation, private output, and abort handling.
- Define candidate scheduling and query policies, including the treatment of multiple identities.
- Specify session authentication, expiry, consent binding, replay handling, and contact release.
- Document wire formats and key configuration before claiming interoperability.

## Reporting experiments

Use synthetic or appropriately consented, non-sensitive evaluation material. Record the input definition, profile and software versions, assumptions, procedure, metrics, and observed failures. Distinguish proposed mitigations from tested behavior.

The illustrative threshold in the RFC is not a measured accuracy result. A completed benchmark alone would not establish the security of the full protocol.
