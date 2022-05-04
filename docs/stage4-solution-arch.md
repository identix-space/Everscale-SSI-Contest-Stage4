# Solution Architecture Overview

## Introduction

The requirements of Everscale SSI contest Stage 4 demand that a proposed solution must provide 
a basic implementation of SSI Triangle of Trust with Issuer, Holder, Verifier roles, and also VC issuance 
and verification procedures.

The core conception that drives the architectural decisions Identix team have made is 
[Verifiable Credential Brokerage Protocol](vc-brokerage-overview.md). The design described in this document 
and the working solution delivered as a nomination for the contest implement a part of VCBP, the basic subset of
roles, operations and components, necessary to meet the contest's criteria.

## Architecture of services and major components

The solution architecture the Identix team proposed for the contest Stage 4 should be considered as a prototype
for the perspective SSI ecosystem. The two main product parts of it are Identix.SSO and Identix.PASS. They include 
a number of internal services and components, and rely on Everscale as a backbone for DID and VC anchoring systems.

![Identix solution architecture for Stage 4](idx-ecosystem-stage4.png)

### Identix SSO and custodial wallets
Identix Single Sign-On is a service that allows both Web2 and Web3 users become parties of 
decentralized trust relationships and participants of trust transactions using [DIDs](https://www.w3.org/TR/did-core/).
Google, Facebook, Twitter users may utilize their corresponding Web2 accounts to create a *custodial wallet* via 
Identix SSO by literally one click. For [Everwallet](https://wallet.broxus.com/) users there is also a possibility to
create a DID using a connected wallet. The created DID then associated with used Web2 or Web3 account to control

> The proposed solution implements 'custodial wallets' to control DIDs and VCs for both cases, since there is no
fully functional [VcWallet](vc-brokerage-overview.md#vcwallet) currently in the market. That means that 
Identix keeps a key pair and operates in VCBP on behalf of a user. In the future, when the VcWallet implementation
will appear, control transfer of these DIDs to non-custodial wallets will be possible. 

Identix SSO also allows registered third-party applications to use the single sign-on functionality to bring their 
user audience into the world of decentralized trust. 

### Anchoring system for the DID management
The Stage 4 solution implements the two anchoring systems. The DID anchoring system is an evolution of 
DID management system we previously created at Stage 3. Everscale blockchain is used to fix DID document anchors and 
the developed `did:ever` [method]() is used to construct DIDs.

The DID anchoring subsystem is designed with the two smart contracts:
1. The DID fabric and controller: `IdxDidRegistry` [(code)]()
2. The DID document anchor: `IdxDidDocument` [(code)]()

> For the Stage 4, the main purpose of `IdxDidDocument` is to bind a user DID with a user public key. 
Further development may include extensions to that by adding e.g. capability or delegation relationships,
as per [DID Core §5.3.4](https://www.w3.org/TR/did-core/#capability-invocation). 

### Identix PASS
Identix PASS is a service that provides users access to operations with VC. Identix PASS backend implements 
VC Agent and VC broker functionality, while Identix PASS frontend application impersonates users via Identix SSO
to bind VC agents and VC brokers to a user identity, and provides necessary UI.

`Identix Wallets` service is an infrastructure service that stores custodial wallets.

For an end user, Identix PASS interface is a central point of contact and the tool for accessing 
decentralized trust capabilities Identix offers; VC management is the first to mention.