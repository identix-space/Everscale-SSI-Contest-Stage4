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

## Identix SSO, DID management and custodial wallets
Identix Single Sign-On is a service that allows both Web2 and Web3 users become parties of 
decentralized trust relationships and participants of trust transactions using [DIDs](https://www.w3.org/TR/did-core/).
Google, Facebook, Twitter users may utilize their corresponding Web2 accounts to create a *custodial wallet* via 
Identix SSO by literally one click. For [Everwallet](https://wallet.broxus.com/) users there is also a possibility to
create a DID using a connected wallet.

The proposed solution implements 'custodial wallets' to control DIDs and VCs for both cases, since there is no
fully functional [VcWallet](vc-brokerage-overview.md#vcwallet) currently in the market.


## Anchoring system