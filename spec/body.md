## Vetting Card

This section is normative.

> **Initial draft.** This is proposed as the first profile in this
> specification. The member names, the binding rules and the commitment
> construction are all open for review.

An applicant presents a [[ref: vetting card]] to one vetter during an identity vetting session. The vetter's software verifies the card, and the vetter compares its claims with the person in front of them and with any identity document the vetter chooses to rely on. The vetter then issues, or declines to issue, a statement about the applicant.

The card shows the vetter three things:
- the claims come from whoever controls the identifier the applicant will join with;
- they were produced for this vetter;
- they were produced for this session.

It carries nothing about any identity document.

### Relationship to the r-card

A vetting card is an [[ref: r-card]] with three additions:
- it is always signed by its publisher;
- it is bound to one audience and one session;
- it carries a salted commitment to the identity claims it discloses.

A vetting card is not updated after issuance. Each session calls for a new card.

> **Editor's note:** This specification does not yet define the base r-card
> structure. The `claims` element shape below follows an existing r-card
> rendering and is illustrative. Once a base structure is defined, this profile
> will reference it rather than define `claims` itself.

### Members

- `type` (array of strings, REQUIRED): MUST include `"VerifiableDataStructure"`, `"RelationshipCard"` and `"VettingCard"`.
- `id` (string, REQUIRED): a unique identifier for the card, such as a `urn:uuid:` URI. A publisher MUST NOT reuse an `id` across cards.
- `publisher` (string, REQUIRED): the DID the applicant will join the community with. Any statement issued on the strength of the card names this DID as its subject.
- `cardVersion` (integer, REQUIRED): the revision of the card, as for any r-card. A vetting card is not updated after issuance, so the value MUST be `1`.
- `audience` (string, REQUIRED): the DID of the one vetter the card is presented to.
- `community` (string, REQUIRED): the DID of the community for which the applicant is being vetted.
- `challenge` (string, REQUIRED): the challenge the vetter supplied when opening the session, copied unchanged.
- `domain` (string, REQUIRED): the domain the vetter supplied when opening the session, copied unchanged. It is ordinarily the community's DID.
- `issuedAt` (string, REQUIRED): an ISO 8601 datetime at which the card was signed.
- `expiresAt` (string, REQUIRED): an ISO 8601 datetime after which the card is no longer valid. See [Binding](#binding).
- `claims` (array of objects, REQUIRED): the claims disclosed to the vetter. Each element carries:
  - `type`: a claim type identifier, such as `name.legal`;
  - `value`: the claim's value;
  - `provenance`: where the claim comes from, which is `selfAsserted` in this draft.

  The array MUST include a claim for every claim type the session requires. It SHOULD NOT include claims the session neither requires nor lists as optional.
- `identityCommitment` (string, REQUIRED): see [Identity Commitment](#identity-commitment).
- `commitmentSalt` (string, REQUIRED): see [Identity Commitment](#identity-commitment).
- `proof` (object, REQUIRED): see [Signing](#signing).

**Example:**

```json
{
  "type": ["VerifiableDataStructure", "RelationshipCard", "VettingCard"],
  "id": "urn:uuid:0f8e5c1a-6b7d-4e2f-9a3c-1d2e4f6a8b90",
  "publisher": "did:webvh:QmPq7TvZ...:people.example:alice",
  "cardVersion": 1,
  "audience": "did:webvh:QmR4sKdX...:people.example:carol",
  "community": "did:webvh:QmSbCcXWDDJmqE8m1nZ...:community.example",
  "challenge": "q4yN1Ztm0cVZB0mQv3m9rS6oPqk2Jk7hX8l1wYd5eFo",
  "domain": "did:webvh:QmSbCcXWDDJmqE8m1nZ...:community.example",
  "issuedAt": "2026-09-20T10:12:00Z",
  "expiresAt": "2026-09-20T10:27:00Z",
  "claims": [
    { "type": "name.legal", "value": "Alice Example", "provenance": "selfAsserted" },
    { "type": "account.handle", "value": "alice@example.org", "provenance": "selfAsserted" }
  ],
  "identityCommitment": "zQm...",
  "commitmentSalt": "Yb3m6c0JgqS1nF2v9wX4kL7pQ8rT5uH0aD3eG6iJ9kM",
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "created": "2026-09-20T10:12:00Z",
    "verificationMethod": "did:webvh:QmPq7TvZ...:people.example:alice#key-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "z..."
  }
}
```

### Signing

The publisher MUST sign a vetting card with a [Data Integrity](https://www.w3.org/TR/vc-data-integrity/) proof, as follows:

- `proof.type` MUST be `"DataIntegrityProof"`, and `proof.cryptosuite` MUST be `"eddsa-jcs-2022"`, as defined in [Data Integrity EdDSA Cryptosuites v1.0](https://www.w3.org/TR/vc-di-eddsa/).
- `proof.proofPurpose` MUST be `"assertionMethod"`.
- `proof.verificationMethod` MUST identify a verification method that the publisher's DID document authorizes for `assertionMethod`.

The proof covers every other member of the card, including `audience`, `challenge`, `domain`, `expiresAt`, `identityCommitment` and `commitmentSalt`. A card without a proof, or whose proof does not verify, is not a vetting card.

### Binding

A vetting card is usable by one vetter, in one session, for a short time. The publisher MUST set:
- `audience` to the vetter's DID;
- `challenge` and `domain` to the values the vetter supplied for the session in which the card is presented;
- `expiresAt` to a time after `issuedAt`, no later than the longest window the verifier accepts for the session.

How long that window should be is the publisher's choice within the verifier's limit, not a constant of this specification. What limits replay is that the window is bounded and that the verifier enforces its own bound, rather than any particular duration. Fifteen minutes suits a card presented in a single sitting, and is a reasonable default for software to offer; a session arranged across time zones, or a vetter working through a queue, may warrant longer. A community that wants a ceiling publishes it with the rest of its vetting requirements, and a verifier applies it in check 8 below.

A verifier MUST reject a card unless all of the following hold:

1. `proof` verifies against the publisher's DID document, as specified in [Signing](#signing).
2. `publisher` is the identifier the applicant asked to be vetted for.
3. `audience` is the verifier's own DID.
4. `challenge` and `domain` equal the values the verifier supplied for this session.
5. The current time is no earlier than `issuedAt` and no later than `expiresAt`.
6. `identityCommitment` recomputes from `commitmentSalt` and `claims`, as specified in [Identity Commitment](#identity-commitment).
7. The card carries none of the content listed in [What a Vetting Card Never Carries](#what-a-vetting-card-never-carries).
8. The interval between `issuedAt` and `expiresAt` is no longer than the maximum the verifier accepts, under its own policy or that of the community it vets for. A verifier that publishes no maximum applies none.

Together, these checks stop a card from being replayed:
- to a different vetter, because of `audience`;
- in a different session, because of `challenge`;
- for a different community, because of `domain`;
- later, because of `expiresAt`.

This specification does not define how the challenge and domain reach the applicant. It also does not define how the vetter confirms that the person in front of them is operating the software that signed the card. Both belong to the vetting session trust task.

### Identity Commitment

`identityCommitment` lets every vetter of one application sign the same value. The community that later receives the vetters' statements can then confirm that they all concern the same claimed identity, without ever receiving the claims.

- `commitmentSalt` MUST encode 32 bytes from a cryptographically secure random number generator, using base64url without padding ([RFC 4648 §5](https://datatracker.ietf.org/doc/html/rfc4648#section-5)).
- A publisher MUST generate one salt for each application to join a community. It MUST use that salt on every card of that application, and MUST NOT use it for any other application.
- A publisher MUST NOT disclose `commitmentSalt` to anyone other than the vetters it presents cards to.
- A card MUST NOT carry more than one claim of any committed claim type.

`identityCommitment` MUST be computed as follows:

1. Form a JSON object with two members:
   - `salt`, whose value is the `commitmentSalt` string;
   - `claims`, an array holding one object for each claim in the card whose `type` is an identity claim that the community's vetting requirements mark as required. Each object has only that claim's `type` and `value` members. The array is ordered by `type`, compared by UTF-16 code units as JCS compares property names.
2. Canonicalize the object with the JSON Canonicalization Scheme ([JCS, RFC 8785](https://datatracker.ietf.org/doc/html/rfc8785)), and compute the SHA-256 hash of the resulting UTF-8 bytes.
3. Form a Multihash by prefixing the hash with the `sha2-256` header (`0x12`) and the length (`0x20`). Encode the result with base-58-btc and prefix the Multibase header `z`, per [CID v1.0](https://www.w3.org/TR/cid-1.0/#multihash).

This is the `digestMultibase` encoding of [VC Data Integrity §2.6](https://www.w3.org/TR/vc-data-integrity/#resource-integrity), which the DTG Credentials specification also uses for its digest-valued members. A verifier comparing two commitments MUST decode both and compare the Multihash algorithm and digest bytes, rather than comparing the strings.

> **Editor's note:** Two implementations will agree on a commitment only if they
> agree on which claims are committed and in what order. This draft commits
> `type` and `value` but not `provenance`, so that a later card whose claim is
> backed by a credential still yields the same commitment. It orders claims by
> `type`. Both choices are proposals. The construction also assumes that every
> vetter of an application works from the same list of required claims, and the
> community's published vetting requirements are the natural source of that
> list.

### Card Digest

A vetter records which card it relied on by keeping a digest of the card. That digest is the `cardDigestMultibase` of the identity-vetting endorsement proposed for the DTG Credentials specification. It MUST be computed over the card exactly as it was received, not over a re-serialization of a parsed representation, as follows:
1. remove the card's top-level `proof` member;
2. apply steps 2 and 3 of [Identity Commitment](#identity-commitment).

Software that parses a card into a model and serializes it again can drop members it does not recognize. The digest would then differ from the one the applicant computes over the card it sent. The card contains `commitmentSalt` and `challenge`, so the digest cannot be reversed by trying candidate claim values.

### What a Vetting Card Never Carries

A vetter examines any identity document directly, and nothing about the document is transmitted. A vetting card MUST NOT carry:

- an identity document number, or any other identifier issued with a document;
- an image or scan of any document;
- a portrait of the publisher, or any other biometric data.

A verifier that finds such content in a card MUST reject the card, and SHOULD NOT retain it.

## Security Considerations

*This section is informative.*

1. **Replay.** A card that verified for one vetter would, without binding, be accepted by any other. `audience`, `challenge`, `domain` and a bounded validity window confine each card to one vetter, one session and one community. What matters is that the window is short enough for the risk the verifier is carrying and that the verifier enforces its own maximum, not the particular duration a publisher chose. Verifiers need to apply every check in [Binding](#binding).
2. **A signature is not liveness.** A valid proof shows that the controller of `publisher` produced the card for this session. It does not show that the person the vetter is looking at is that controller. The vetting session trust task has to supply that check, for example with a code derived from the session that both parties read aloud, and a vetter should not attest without it.
3. **Self-asserted claims.** Claims in this draft are `selfAsserted`. The proof establishes who made them, not that they are true. Any assurance comes from the vetter's own check against the person and, where used, a document.
4. **Low-entropy digests.** An unsalted digest of a legal name could be reversed by hashing candidate names until one matched. `identityCommitment` is salted with 32 random bytes per application, and the card digest covers both the salt and the session challenge. Compare the discussion of unsalted digest-valued members in [dtgwg-cred-spec#38](https://github.com/trustoverip/dtgwg-cred-spec/issues/38).
5. **Salt reuse.** A salt reused across applications gives those applications the same commitment and links them. Each application needs a fresh salt.

## Privacy Considerations

*This section is informative.*

1. **Who sees a card.** Only the vetters the applicant chooses to present it to. A community that relies on vetting receives the vetters' statements, which carry the commitment and the card digest. It never receives the card or the salt.
2. **No document data.** A card never carries document numbers, document images or portraits. Vetting software should not offer to capture a photo of a document, and should not keep any document detail the vetter sees.
3. **Retention.** Once a vetter's statement has been issued or declined, vetting software should delete the card after a short, stated period and keep only its digest.
4. **Commitment linkage.** Every vetter of one application sees the same commitment, by design. A vetter who keeps a card could recognize that commitment later. A fresh salt per application confines that to one application.
5. **The join identifier.** A card discloses the applicant's join identifier to every vetter it is presented to. Applicants choosing a [correlation scope](https://github.com/trustoverip/dtgwg-cred-spec/blob/main/spec/terms-definitions/correlation_scope.md) for that identifier should take this into account.

## Governance Considerations

{{RECOMMENDED for most ToIP specifications. This section SHOULD be formatted as either a numbered list or numbered subsections. This section SHOULD include any relevant references to the [ToIP governance metamodel](https://trustoverip.org/wp-content/uploads/ToIP-Governance-Metamodel-Specification-V1.0-2021-12-21.pdf). [See also this IETF guidance](https://www.ietf.org/archive/id/draft-attoumani-ietf-inclusion-00.html).}}

## Internationalization Considerations

{{RECOMMENDED for most ToIP specifications. This section SHOULD be formatted as either a numbered list or numbered subsections. [See this IETF guidance](https://www.ietf.org/archive/id/draft-flanagan-7322bis-07.html#name-internationalization-consid).}}

## Accessibility Considerations

{{RECOMMENDED for most ToIP specifications. This section SHOULD be formatted as either a numbered list or numbered subsections. [See this IETF guidance](https://datatracker.ietf.org/doc/html/rfc6973) and also the Accessibility Considerations section in [W3C VC](https://www.w3.org/TR/vc-data-model/#accessibility-considerations).}}

## Conformance

This section is normative.

This specification defines normative requirements, using the keywords defined in [Requirements Language](#requirements-language), for the conformance targets below.

### Conformance Targets

1. **Publishers**: parties that produce vetting cards. A conforming publisher MUST produce cards that satisfy [Members](#members), [Signing](#signing), [Binding](#binding), [Identity Commitment](#identity-commitment) and [What a Vetting Card Never Carries](#what-a-vetting-card-never-carries).
2. **Verifiers**: parties that verify vetting cards. A conforming verifier MUST apply every check in [Binding](#binding). Where it records a card digest, it MUST compute that digest as specified in [Card Digest](#card-digest).

### Conformance Tests

## References

### Normative References

- [IETF RFC 2119: Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119)
- [IETF RFC 4648: The Base16, Base32, and Base64 Data Encodings](https://datatracker.ietf.org/doc/html/rfc4648)
- [IETF RFC 8785: JSON Canonicalization Scheme (JCS)](https://datatracker.ietf.org/doc/html/rfc8785)
- [W3C Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/did-1.0/)
- [W3C Verifiable Credential Data Integrity v1.0](https://www.w3.org/TR/vc-data-integrity/)
- [W3C Data Integrity EdDSA Cryptosuites v1.0](https://www.w3.org/TR/vc-di-eddsa/)
- [W3C Controlled Identifiers (CIDs) v1.0](https://www.w3.org/TR/cid-1.0/)
- [ISO 8601: Date and time format](https://www.iso.org/iso-8601-date-and-time-format.html)

### Informative References

- [Decentralized Trust Graph Credentials - Core Specification](https://github.com/trustoverip/dtgwg-cred-spec)
