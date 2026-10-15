## Introduction

*This section is informative.*

> **Initial draft.** This is a first contribution to an unfilled template. It
> exists to carry one concrete profile, the [[ref: vetting card]], so that the
> working group has text to review. Its structure, scope and requirements are
> proposals. The editors, document status, abstract and the remaining template
> sections have not been settled and are left as template placeholders.

The DTG Credentials specification defines a verifiable data structure as:

> "A data structure digitally signed by the publisher so that subscribers are able to cryptographically verify the authenticity of both the original and any updates. A VC is one kind of VDS. An r-card is another type of VDS."
>
> — [DTG Credentials, Terminology](https://github.com/trustoverip/dtgwg-cred-spec/blob/main/spec/terms-definitions/verifiable_data_structure.md)

This specification defines [[ref: VDS]] types that are exchanged over DTG relationships but are not DTG credentials. The DTG Credentials specification names two such types as planned for this specification: the [[ref: r-card]] and an agent card.

This initial draft defines one profile of the r-card: the [[ref: vetting card]]. An applicant who wants to join a community signs a vetting card and presents it to an existing member, the *vetter*. The vetter checks the applicant's identity in person or on video, then issues a statement about the applicant. That statement is a verifiable endorsement credential under the DTG Credentials specification. This specification defines only the card.

The base r-card structure is not yet defined here. Where the vetting card profile depends on it, the draft says so.
