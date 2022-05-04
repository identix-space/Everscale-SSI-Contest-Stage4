# Verifiable Credential Lifecycle, Data Model and Implementation Design

## Introduction
A Verifiable Credential is an entity, which [VC brokerage](vc-brokerage-overview.md) is circling around.
Identix team's view on this concept is coherent with [the common W3C doctrine](https://www.w3.org/TR/vc-data-model/).
The actual VC lifecycle design and implementation add certain specifics, but strive to comply to other
mature (like [JWT](https://datatracker.ietf.org/doc/html/rfc7519), [DIDs](https://www.w3.org/TR/did-core/)) or 
emerging (like [EDV](https://digitalbazaar.github.io/encrypted-data-vaults/)) 
Web standards, and to reuse approaches we see advantageous (like anchoring 
from [DIF SideTree](https://identity.foundation/sidetree/spec/) or 
anoncreds from [Hyperledger Aries](https://github.com/hyperledger/indy-hipe/tree/main/text/0109-anoncreds-protocol)).

This document doesn't explain the Identix concept in all details, but rather focuses on the design 
for the [Everscale Contest Stage 4](https://forum.freeton.org/t/freeton-self-sovereign-identity-framework-stage-4/12415) 
implementation, accompanying by a few indications of the planned future developments.

## The target trust scenarios
The target scenarios are based on the VC Brokerage Protocol's 
[agency model](vc-brokerage-overview.md#vcbp-agency-model-and-dids). From all multitude and variability of
digital trust relationships the Stage 4 implementation covers the following, as specified in contest's 
[hard criteria](https://forum.freeton.org/t/freeton-self-sovereign-identity-framework-stage-4/12415#hard-criteria-7):

>1. The Issuer issues Verifiable Credentials with assertions about the subject.
>2. The Holder possesses Verifiable Credentials and presents them to Verifiers.
>3. The Verifier ensures:
>   1. VC’s relation to the certain statement
>   2. VC’s relation with the Subject
>   3. VC is authentic (cryptographically certified by the Issuer)

According to this requirement, the Stage 4 solution realizes the following procedures:
1. An Agent signs in to Identix.PASS and can act impersonating any of normative roles: 
Holder, Issuer, Verifier. Successful sign in attempt results creates or activates a user account with associated DID.
   > An agent needs no re-login to switch a role. Role functions are separated on UX/UI level.
2. As an Issuer, the agent can `issue a VC` of a type of choice to a Holder, by specifying the Holder's DID 
and other data, according to [VC Claims Specification](#vc-claim-specifications) of the VC type.
   > - VC issuance requests from Holder to Issuer is a subject for further developments and is not implemented for the Stage 4.
   > - In future versions, Issuers will have to explicitly declare a list of VC types they serve. Current implementation is free from this restriction.
   > The "Issuer - VC type list" associations must be available at a VCBP Registry.
   > - Potentially, there can be many Issuers to sign a single VC. Support for such scenarios with even more complex trust chaining is also envisioned.  
3. Once issued, the VC instance delivered to the `Holder to possess` and can be found at his/her wallet.
4. For VCs at Holder's disposal, he/she can `request verification` by choosing a provided verifier service and sending a message.
   > Currently, ony one agent is declared to be a verifier (find "FlatQube"). For the protocol to work, verifiers have to publish their 
   > wanted VC type lists the same way as Issuers do. VCBP Registry for these purposes will be implemented in future versions.
5. The chosen verifier receives the request in their inbox list and can review the VC data sent
and `Approve` or `Reject` the verification. Cryptographic operations, required to ensure integrity of credentials 
and certificate chains are performed automatically. A secret code(s) that allows Verifier to access the information
is sent as a part of this interaction.
   > Some scenarios may require more complex interactions between Issuer and Holder for issuance 
   > and between Holder and Verifier. These may include sharing symmetric RSA keys, HMAC secrets, nonces etc, 
   > and also a few-round communication between signers, like in Schnorr' scheme. All those peculiarities must be
   > hidden from user under the hood of VC Brokerage Protocol implementation. Secrecy, required for the certain data,
   > must be provided by agent's Encrypted Data Vault and VC Broker, so no sensitive data will ever leave the agent's
   > process boundary and EDV storage area.

## VC Claim Specifications
The


### Scheme as a contract

The Verified Credential Scheme is a contract that fixes

1. Syntactic requirements for the representation of semantics (template) needed to express statements that define some specific trust protocol.
   > Example: the association of a name with an identifier is implemented through the syntax of the SPO triplet
   "did:ever:123" “has_name” “Pete Petrov”
   Scheme
   NameVC: [”has name”]
   requires all VC declaring compliance with the Namevk schema to have statements with the predicate has name.
   The credibility of this scheme further extends to the credibility of the practice of identifying the “name” of the DID agent in the relevant practices.

### Practices of LC VC schemes
| 1 | 2|
| --- | --- |
| Schema design | The VC schema is constructed as a named group of requirements for the presence of statements in a VC instance.The requirements for the presence of statements are constructed as a named ordered group of 3 elements, in accordance with the SubjectReq PredicateReq ObjectReq RDF syntax |
| Schema validation | The constructed circuit must be validated by a public circuit registrar |
| Schema publication | A valid schema is published by the registrar as an artifact available for receipt in accordance with the URN (DID). |
| Constructing a certificate request | A certificate request (to the Issuer) can be constructed as a reference to a VC scheme, according to which it is required to provide a VC instance |
| Constructing a VC instance | Constructing a VC instance consists of constructing elementary statements and grouping them according to the selected scheme, and creating all the necessary certificates for this group |
| Validation of the VC instance | Validation of the VC instance - checking the presence of all statements in the submitted group of elementary statements in accordance with the declared scheme |
| VC instance verification | VC instance verification - verification of the entire group of certificates associated with the submitted group of statements for compliance with the requirements of the trust protocol (correctness of the cryptographic signature, expiration, etc.) |
| Revocation of the scheme | Formal revocation of the scheme is not expected. The actual decommissioning of the scheme (for example, as a result of a failure of the registrar's service) only means the absence of a public contract. This does not imply the termination of trust practices between counterparties, but only limits, complicates or reduces the level of trust in practices, which remains at the discretion of the counterparties. |
## VC representation

## VC anchoring

## Notes on cryptography
Ed25519 -> schorr, ZKP