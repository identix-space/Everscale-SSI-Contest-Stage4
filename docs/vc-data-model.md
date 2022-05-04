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

## VC Claims Specification
In order to make a digital trust transaction to happen, a set of contracts between actors and between VCPB brokers 
must be established: semantical, syntactical, procedural. VC Brokerage Protocol 
[specifies](vc-brokerage-overview.md#vc-brokerage-protocol) this set of high-level contracts, 
and represents itself a *metacontract* in this respect.

The contracts must ensure all parties understand each other, meaning all trust procedures are coherent 
in an expected order of execution and in terms of their input and output artifacts.

Contracting the protocol communication around Verifiable Credentials needs to address several layers 
of representation and interpretation. 
Fortunately, most of them are already addressed by the community, but there are [debatable] reasons 
for VCBP trust architects to make additional efforts in order to introduce certain novelty. 

1. **Representation layer** contract specifies how VC's body is stored and transmitted over the wire. Since the format 
may affect data integrity, it must be regulated to a reasonable extent. We chose JSON Web Token as 
a final destination of VC composition (see [VC Representation](#vc-representation) for the schema), 
plus an Everscale smart contract for VC anchoring (see [VC anchoring](#vc-anchoring)).
2. **VC syntax layer** contract regulates how the transmitted object is interpreted to be a Verifiable Credential 
having its specific properties. We chose the Verifiable Credentials Data Model W3C Recommendation 
[v1.1](https://www.w3.org/TR/vc-data-model/) over JSON as the VC syntax standard.
3. **Claim syntax layer** contract specifies how a semantic identity of a *claim*, 
shared in a [semantic community](https://semanticcommunity.org/SemanticCommunity.aspx), is represented to have 
the same meaning for all parties of trust relationships. W3C standards tend to reuse one of the most elaborated syntaxes
to represent domain semantics: [JSON for Linked Data](https://w3c.github.io/json-ld-syntax/). It definitely makes sense
in the long run, but this approach has a [number of drawbacks]() in terms of cognitive load, performance, total cost 
and overall solution complexity. Particularly for the VC context, there are issues related to 
zero knowledge proofs and partial disclosure concerns, which e.g. motivated Hyperledger team to elaborate on 
a specific binary format for representing claims: [anoncreds](https://github.com/hyperledger/indy-hipe/tree/main/text/0109-anoncreds-protocol).
<br>The presented Stage 4 solution introduces a hybrid approach, which allows simple claims, represented as
[RDF triples](https://www.w3.org/TR/rdf11-concepts/#dfn-rdf-triple) to be compacted in a set of
binary attributes, similar to those from anoncreds. This form gives more freedom to use advanced cryptographic approaches,
elevate privacy, and significantly reduce VC anchor size, what may be advantageous for reasons of reducing storage costs
in Everscale. 

### Scheme as a contract

The Verified Credential Claims Specification is a scheme, which contracts

1. Syntax to express a `claim`.
<br>Each claim thus become an atomic **artifact of shared meaning**.
2. Syntax to express a `group of claims`.
<br>Assumed, that for the most of practical cases, a *set* of semantically bound atomic claims is required to express a
fact, valuable for a trust transaction.
<br>Each claim group thus become an atomic **artifact of shared data**. E.g. first name, middle name and last name
may be considered semantically different, but can be meaningfully shared only as group to address a subject individual
in the respect of own names. Maybe more important, this will allow certain flexibility for partial disclosure and storage optimization. 

W3C Recommendation makes the symmetric conceptual differentiation, partially for the sake of mereology:
*"A credential is a set of one or more claims made by the same entity."* [VC DM §3.2](https://www.w3.org/TR/vc-data-model/#credentials)
<br>The aggregation hierarchy, proposed here, `VC -> ClaimGroup -> Claim` does more to that. 
A verifiable credential instance thus become an atomic **artifact of trust transaction**, and provides common
trust context (e.g. certificate chain) for the members.  

VCCS requires claims to be expressed as `Subject-Predicate-Object` RDF triplet.
> Example<br>
> Let the DID `did:ever:123` was acquired by a respected individual we know as Pete Petrov. Then the triplet<br>
> `"did:ever:123" “has_first_name” “Pete”`<br>
> specifies (the claim syntax) that such association exists somewhere, and that certified by a trusted agent - issuer.<br>
> `"did:ever:123" “has_last_name” “Petrov”`<br>
> gives us another valuable claim, which combined may form a group "names" for a Government ID verifiable credential. 
> Corresponding VC Claim Specification (name it `StateID`), which includes a requirement like `["has_first_name", "has_first_name"]`
requires all VCs, which declare compliance with this specification to have claims with the indicated predicates.


### Lifecycle of VC Claim Specification
| Practice name | Description |
| --- | --- |
| Designing a specification | A VCCS is constructed as a named group of requirements for the presence of predicates in a VC instance. The requirements for the presence of statements are constructed as a named ordered group of 3 elements, in accordance with the `Subject Predicate Object` RDF syntax |
| Validating a specification | The constructed specification must be validated by a public VCCS registrar |
| Publishing a specification | A valid specification is published by the registrar as an artifact, available for retrieval by VCCS instance' URN (DID). |
| Constructing a VC request | A VC request (to the Issuer) must include a reference to a VCCS, according to which involved parties have to provide data to construct a set of claims for a VC instance |
| Constructing a VC instance | Constructing a VC instance consists of constructing unit claims and grouping them according to the selected scheme, and creating all the necessary certificates for this group |
| Validating a VC instance | Validation of the VC instance means checking presence of all claim groups and claims in a submitted VC in accordance with the specified VCCS |
| Verifying a VC instance | Checking that the entire group of certificates associated with each group of claims comply with requirements of required trust protocol (cryptographic signature, expiration date, etc.) |
| Revocation of a specification | Formal revocation of a VCCS is not expected. The actual decommissioning of a VCCS instance (for example, as a result of a failure of the registrar's service) only means the absence of a public contract. This does not imply the termination of trust relationships between counterparties, but only limits, complicates or reduces the level of trust in them, which remains at the discretion of the counterparties. |


## VC representation

## VC anchoring

## Notes on cryptography
Ed25519 -> schorr, ZKP