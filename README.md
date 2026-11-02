# A Maintainer Trust Ledger for Decentralized Projects

**Concept whitepaper · 2 November 2026**

## Executive summary

Decentralized projects often depend on people who review code, operate infrastructure, publish releases, moderate communities, and steward shared resources. Participants need a way to understand who has endorsed whom, what work was reviewed, and whether an endorsement or authorization has since been withdrawn—without relying on one platform's mutable profile or a single central authority.

This paper proposes a **signed, append-only trust ledger**: a portable record of maintainer proposals, reviews, approvals, rejections, endorsements, and revocations. Maintainers can endorse other maintainers, but endorsements are explicit, scoped, attributable, and challengeable. Participants interpret the record according to local policies and evidence; the ledger does not issue a universal trust score.

The ledger is an auditable social coordination tool. It is **not equivalent to Bitcoin consensus**, does not establish a globally agreed truth, and cannot make people honest. Cryptographic signatures help establish which keys signed which statements; governance and human judgment remain essential.

## 1. Goals and non-goals

The system aims to:

- Make project stewardship and review history inspectable across platforms.
- Let maintainers vouch for others with stated scope, rationale, and duration.
- Preserve the history of decisions, including rejection, correction, and revocation.
- Help communities evaluate authority and evidence without depending on one administrator.
- Support multiple projects, federated groups, and independent verification tools.

It does not aim to prove a person's legal identity, guarantee competence or good intent, decide disputes automatically, or replace code review, release security, due process, or project-specific governance. A signature is evidence of a key's action, not proof that the statement is true.

## 2. Trust model

### 2.1 Identities, keys, and roles

A participant is represented by one or more public keys and optional human-readable identifiers. Projects publish how keys are associated with roles, such as reviewer, release manager, or infrastructure operator. Key rotation and recovery are explicit events; a new key does not silently inherit another key's history. Communities may use multiple keys, hardware-backed keys, or threshold signing for sensitive roles.

An endorsement is a signed statement from one maintainer about another. It should identify:

- The issuer and subject keys, and the project or community context.
- The role or capability endorsed (for example, reviewing a particular subsystem).
- The basis for the endorsement, preferably with references to reviewable evidence.
- Any limits, conditions, or expiration date.

Endorsements are directional: A endorsing B does not imply B endorses A. They are not votes in a universal popularity contest. Participants may consider the endorser's demonstrated experience, independence, and history, rather than simply counting endorsements.

### 2.2 Records and interpretation

Each event is a signed, structured record with a unique identifier, event type, subject, context, timestamp claim, and links to relevant evidence. The ledger may contain proposals, reviews, approvals, rejections, endorsements, role assignments, corrections, and revocations. A review or decision should state its scope and rationale; an approval of one change is not blanket approval of its author or all future work.

Records are append-only in the logical sense: later events do not erase earlier signed statements. A correction references the record it corrects; a revocation states what it withdraws and when it takes effect. Views may hide spam or private material, but clients should preserve the distinction between omission from a view and removal from the underlying history.

Trust is a **contextual graph**, not a scalar. A participant can inspect paths of endorsement and evidence, but should account for common control, reciprocal endorsements, stale statements, conflicts of interest, and differing scopes. Local policy determines what evidence is sufficient for a particular action.

### 2.3 What cryptography can and cannot establish

Digital signatures can detect unauthorized alteration of signed records and verify that a particular key produced a signature. Hash-linked logs, Merkle trees, and independent witnesses can make undetected rewriting or equivocation more difficult. They do not prove that a key belongs to the person claimed, that a timestamp is accurate, that all events were published, or that reviewers exercised sound judgment. Those properties require key-binding procedures, distribution, witnesses, audits, and social accountability.

### 2.4 Admission as a provenance graph, not an unchecked family tree

A project may model maintainer admission as a directed graph: an active maintainer signs an approval that contributes to a candidate's admission. This makes the proposed “maintainers approve new maintainers” rule inspectable, but an ordinary tree is too restrictive when a candidate needs several independent approvers or is supported by more than one branch. The canonical structure should therefore be a **directed acyclic graph (DAG)** of scoped admission grants and their dependencies; a UI may display a tree-like view, but must not hide additional approvers or paths.

A safe admission procedure is:

1. The project publishes an admission policy for each role: which currently authorized maintainers may approve, the required threshold, any independence constraints, and the scope and duration of the resulting authority.
2. A candidate accepts a signed, scoped nomination. Required approvers sign distinct approval records that refer to the same candidate, role, policy version, proposal, and evidence. Approval records are not themselves active authority; a separate signed admission decision, or a verifier's deterministic threshold evaluation, records when the published rule is met.
3. The verifier checks that each approval was made by a currently eligible approver under the applicable policy, counts only distinct authorized keys/people as policy defines, and binds the resulting role to the exact approvals and policy version. Thresholds should be chosen for the project's risk and size; this paper does not prescribe a universal number.
4. Each grant identifies the authority lineage on which it depends. A candidate cannot approve their own admission, approve an ancestor in the same dependency chain, or create circular authority. Reject cycles at validation time, including cycles introduced by key aliases, policy changes, or later references. A cycle is invalid, not a means to make every member self-authorizing.

Multiple approvals are represented explicitly, not flattened into a single mysterious “parent.” A grant can require, for example, a threshold from independent eligible approvers, and record all approvals considered, the threshold rule, and conflicts or recusals. Independence is a governance judgment, not something graph topology or distinct keys alone proves. If policy permits two separate sufficient approval sets, preserve both as alternative provenance paths; losing one path need not invalidate a grant while another still satisfies policy.

**Revocation has a defined scope.** A signed revocation identifies the grant or key/role being withdrawn, the issuer's authority to revoke, the effective time or event sequence, reason category where safe, and whether it is an emergency suspension or a reviewed final decision. Clients should distinguish (a) revoking one person's current role, (b) invalidating a particular admission grant because its approval was unauthorized or compromised, and (c) retiring a key. These are not interchangeable. Revoking an endorsement as an opinion does not automatically revoke a role; the project's policy must say when an endorsement is a required authority dependency.

If a project chooses cascade semantics, the verifier recomputes the graph from the signed history. Grants that depend on a revoked grant lose that specific support path; a descendant loses active authority only if, after recomputation, it no longer satisfies the published admission rule through any valid independent path. This avoids treating a single “root” as an unquestionable global authority and avoids automatically ejecting legitimate descendants with independent approvals. If policy deliberately defines a single-rooted chain, revoking that root can invalidate all authority derived solely from it, but the potentially broad consequence must be stated in advance. A cascade changes the current authorization view; it does not erase historical approvals, revoke people's unrelated work or endorsements, or establish misconduct by descendants. Any direct suspension broader than lineage invalidation needs its own signed, scoped decision and review/appeal route.

Recovery must not silently rewrite lineage. A compromised or lost root key should be suspended according to the published emergency rule, with independent witnesses or a documented quorum where practical. The project records key retirement/recovery and whether prior signatures remain valid, are disputed from a stated cutoff, or require case-by-case review; cryptography cannot determine when an attacker obtained a key. Affected people can be nominated or re-admitted through the current policy, using fresh approvals from currently valid maintainers. Re-admission is a new grant with new provenance, not resurrection or deletion of a revoked grant. If no valid maintainers remain, the charter needs an explicit recovery mechanism (for example, a predesignated independent recovery quorum or a time-limited community process); otherwise recovery may be impossible without a policy fork.

All proposals, approvals, decisions, suspensions, revocations, recovery actions, and policy versions are signed, uniquely identified, and append-only. A revocation adds an event that references earlier events; it never edits or deletes them. Clients compute current status from the log and publish their policy and algorithm version. Replicated logs, signed checkpoints, and independent witnesses help expose equivocation or selective omission, but no log proves completeness without independent observation. Where an event must be redacted for safety or privacy, retain a signed tombstone/commitment and explain the limits rather than claiming the underlying bytes have been erased everywhere.

This structure limits neither Sybil creation nor collusion by itself. Attackers may invent identities, compromise enough approvers, or gradually capture a branch; distinct keys are not necessarily independent people. Use reviewable work, conflict disclosures, reviewer diversity, cautious thresholds, periodic revalidation, and non-graph evidence. A powerful cascade rule can also be abused to disable many maintainers: require narrowly defined revocation authority, recorded rationale, notice and appeal where feasible, independent confirmation for broad cascades, and a temporary, time-bounded emergency path. Avoid unbounded graph traversal or per-descendant event storms: use a bounded, deterministic graph evaluation, cache results with verifiable inputs, rate-limit proposals, and reject malformed or oversized graphs. These controls reduce operational denial-of-service risk but cannot remove governance capture or availability risks.

## 3. Workflow

1. **Publish project policy.** The project defines roles, eligibility, review requirements, decision thresholds, appeal routes, key recovery, and how the record is hosted and mirrored.
2. **Propose a maintainer.** A proposer submits a signed nomination with scope, evidence, and any known conflicts. The candidate may accept, clarify, or decline.
3. **Review independently.** Reviewers sign scoped assessments. They disclose conflicts and distinguish factual evidence from opinion. Private security details may be referenced by a controlled disclosure process rather than exposed publicly.
4. **Decide and record.** Authorized maintainers apply the published policy. The result—approval, rejection, or deferral—is signed with rationale and references to the proposal and reviews. Rejections remain visible as decisions, not as proof of misconduct.
5. **Endorse and activate.** Maintainers may issue individual endorsements. A role becomes active only when the project's stated decision rule is met. The signed role assignment names its scope and, where useful, its expiry or review date.
6. **Monitor and update.** New evidence can lead to a new review, a narrower scope, an expiration, or a revocation. The new event links to the earlier record; the historical record remains available.
7. **Verify locally.** Clients validate signatures and log proofs, retrieve mirrors, and apply a clearly described policy. Users can inspect the evidence and understand why a role is treated as valid or disputed.

## 4. Governance

The ledger records governance decisions; it does not itself choose who governs. Each project should publish a concise charter defining who may nominate, review, approve, appeal, and revoke; what constitutes quorum; how conflicts are handled; and how policy changes take effect. High-impact roles may require multiple independent approvals, a waiting period, or periodic renewal. Emergency suspension can be available, but should be time-limited and followed by review.

A rejected nomination or disputed revocation should have a documented appeal path. The community should distinguish temporary access control from a final finding about a person's conduct. Policy changes must themselves be signed, versioned, and announced; otherwise clients may apply incompatible rules to the same history. Participation should not be conditioned on accepting an opaque global reputation score.

## 5. Threats and mitigations

| Threat | Mitigations and residual risk |
|---|---|
| Stolen or compromised key | Hardware-backed keys, least-privilege scopes, short-lived roles, rapid revocation, recovery rules, and multiple signers for sensitive actions. A valid signature may still predate the compromise and require investigation. |
| Sybil identities or collusion | Require project-specific evidence, evaluate independent work and reviewer diversity, disclose relationships, and avoid simple endorsement counts. Real-world identity checks may help some projects but exclude pseudonymous participants and do not guarantee integrity. |
| Clique endorsements or reciprocal boosting | Track endorsement provenance, conflicts, reciprocity, and shared control; require independent review or evidence for consequential roles. Graph patterns are signals, not automatic proof of abuse. |
| Censorship, withholding, or equivocation | Replicate signed events across independent mirrors, publish periodic checkpoints, and use witnesses or transparency proofs. A ledger cannot prove completeness if no independent observer saw an omitted event. |
| Malicious or careless reviewers | Scope reviews, retain rationale and evidence, enable appeals, rotate reviewers, and audit outcomes. Signatures make accountability possible but do not ensure expertise. |
| Harassment, doxxing, or defamatory claims | Minimize personal data, separate evidence from accusation, define moderation and appeal rules, and use controlled disclosure for sensitive reports. Append-only records complicate correction and takedown; publication requires care. |
| Stale or misleading endorsements | Include issue/expiry dates, display age and scope, support superseding events, and require periodic renewal for sensitive roles. Expiry is not a guarantee that the underlying person changed or remains safe. |
| Parser, policy, or software bugs | Use simple canonical formats, independent implementations, test vectors, versioned policy, and reproducible verification. A flawed client can still misrepresent valid records. |

## 6. Limitations and tradeoffs

A public, durable history can improve accountability but can also preserve mistakes, expose relationships, invite retaliation, and make rehabilitation difficult. Not every review or security incident should be public immediately. Projects need a carefully designed disclosure and retention policy, while recognizing that cryptographic records cannot reliably be recalled after broad replication.

Trust judgments remain contextual and contested. Communities differ in values, expertise, and acceptable risk; two clients can honestly derive different conclusions from the same records. A signed history can create an appearance of rigor while embedding bias or concentrating influence among established maintainers. The system also adds operational burden: key management, mirrors, policy maintenance, accessibility, and dispute resolution all require resources.

The ledger therefore complements—not replaces—relationships, direct inspection of work, independent security review, and accountable governance. It should show uncertainty and provenance rather than claim to eliminate them.

## 7. Implementation outline

A practical first version can use a versioned JSON event schema and widely supported digital signatures. Each event should include a schema version, event ID, project ID, issuer key ID, event type, subject, scope, referenced event IDs, evidence links or content hashes, creation time, optional expiry, and signature. Canonical serialization is essential so that different implementations sign and verify identical bytes. Avoid putting secrets or unnecessary personal data in public records.

Store events in an append-only, hash-linked sequence or Merkle log. Publish signed checkpoints and inclusion proofs; mirror the log through independent operators and allow offline verification. Keep policy documents and role assignments versioned alongside the events. Build a verifier that checks signature validity, key status at relevant times, event references, log proofs, and policy version, then reports findings with explanations—not just a badge or score.

Start with a pilot for one bounded role, such as code review in a single repository. Test key loss, conflicting records, revocation propagation, migration, privacy requests, and appeal handling before assigning high-impact permissions. Integrate with existing review and release tools only after participants can independently export and verify the record. Document governance changes and preserve a path to leave or fork the system.

## Conclusion

A maintainer trust ledger can make endorsements and decisions more legible, durable, and contestable. Its value lies in preserving who made which scoped claim, on what stated basis, and how the community later responded. Cryptography can protect record integrity and help expose tampering; it cannot manufacture legitimacy or consensus. Decentralized projects should treat the ledger as shared evidence for human governance—useful when transparent, plural, and carefully limited, dangerous when mistaken for an oracle.
