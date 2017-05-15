__OBIP__ obip-michaelfolkson-risk-contract  
__Title__ Facilitating risk contracts on OpenBazaar protocol  
__Author__ Michael Folkson <michael@riskbazaar.org>  
__Discussions-To__: <michael@riskbazaar.org>  
__Status__ Draft  
__Type__ Standards Track  
__Created__ 05/05/2017
__Post-History__ Slack, 05/03/2017 - 05/04/2017  
__Copyright__ Public Domain  

## Abstract
Although the protocol at this stage is primarily built for decentralized e-commerce, it is broadly agreed that a longer term aim should be to build a protocol for decentralized trade which would include the ability to enter into risk contracts. We are defining a _risk contract_ as "a contract that pays out conditional on the occurrence of a future event".

We propose that there are additions made to the protocol such that applications/clients can be built that enable peer-to-peer risk contracts.

The primary additions to the protocol we propose are:

1) Allow two individuals to fund the multi-signature address rather than just one individual in the e-commerce example.
2) A switch to risk contract JSON (from e-commerce JSON) when transaction is flagged as a risk contract.
3) The ability to negotiate and/or reject a proposed risk contract before sending funds to the multi-signature address.



## Motivation

The potential for the OpenBazaar protocol to facilitate risk contracts has been discussed for a while. Discussions on the OpenBazaar Slack cumulated in this article on Oracles & Risk Contracts by @drwasho.

https://medium.com/@therealopenbazaar/oracles-risk-contracts-768c09cee46c

There have also been articles and white papers written by @RiskBazaar.

https://github.com/RiskBazaar/RiskBazaar/blob/master/white-papers/building-a-risk-market-for-the-digital-age.md

https://github.com/RiskBazaar/RiskBazaar/blob/master/white-papers/internet-for-risk-exchange.md

The primary additions to the protocol we propose are:

> 1) Allow two individuals to fund the multi-signature address rather than just one individual in the e-commerce example.

Hello

2) A switch to risk contract JSON (from e-commerce JSON) when transaction is flagged as a risk contract.

Hello

3) The ability to negotiate and/or reject a proposed risk contract before sending funds to the multi-signature address.

Hello



Add the ability to cancel and/or negotiate a contract before sending funds to the multi-signature address. A contract is only accepted (in a technical sense rather than a legal sense) when both parties have funded the multi-signature address. 

The funds still need to be removed from the wallet after each contract proposal to ensure the individual owns sufficient funds. However, they are sent to a holding address rather than the multisig address. If the proposed contract is accepted, the funds are sent to the multi-signature address. If the proposed contract is rejected, these funds are returned to the individual's wallet.

(invitation to tender, invitation to purchase)

To be deleted
Flexible moderator fees and involvement - The moderator is able to receive a fee for every transaction they are assigned as the moderator (even when there is no dispute or before a dispute has been raised). The moderator is also able to report (sign a transaction from the multi-signature address) even before a dispute has been raised. This was previously raised in a GitHub issue in October 2016.

https://github.com/OpenBazaar/openbazaar-desktop/issues/78

It will add functionality to OpenBazaar transactions but it is especially important for risk contracts because the moderator is reporting on a broader range of events (e.g. sports results, elections, insurance claims) rather than just whether the buyer received the item he/she was promised. Therefore rather than just reviewing evidence provided by the buyer/vendor, the moderator is potentially going to need to research whether the specified event occurred. Also with a risk contract, both parties are putting funds at risk rather than just the buyer in an e-commerce transaction so there are likely to be longer reporting delays to release funds.

 +Currently the seed is passed through a `DeterministicReader` type. This turns the seed into a stream from which downstream functions can 'pull' bytes. Any number of bytes 'pulled' from this reader is in fact an scrypt-generated key, generating the required number of bytes from the entire seed, at a difficulty level of 512; a computationally intensive task. The original reason for this was to stretch seeds going into RSA key generation functions, which require long seeds and therefore a function like scrypt. OpenBazaar has since moved to elliptic keys, where a 32 byte seed is sufficient. It is still good practice to hash the seed, as seeds are partially leaked into ed25519 private keys, but there is no need to use an expensive key stretching function like scrypt.
 +
 +The further problem with the current protocol is that it is quite Go-specific; producing a similar 'reader' which stretches every time bytes are requested is not trivial in other languages, and it would be far simpler to initially run the seed through a hash function, obtaining 32 bytes, and passing that to the generation function.
 +
 ## Specification
 +I propose a simple change; given the seed, in ipfs/identity.go, simply run the mnemonic-derived seed through hmac-sha-256 once to obtain a 32 byte hashed seed, and then produce a `bytes.Reader` from this. This reader can then be passed to the ed25519 library, which will then pull the first (and only) 32 bytes for use in key generation.
 +
 +In other implementations, where an ed25519 library might instead take a simple byte input instead of a go-style 'reader', one can simply hash the seed, and provide the result to the ed25519 library.
 +
 +For documentation, the full identity generation process is now:
 +
 +`seed = bip39Derive(mnemonic)`  
 +`obSeed = HMAC-SHA256(seed, "OpenBazaar seed")`  
 +`identityPrivKey, identityPubkey = GenerateEd25519(obSeed)`  
 +`serializedPubkey = proto.Marshall(identityPubkey)`  
 +`hashedPubkey = sha256(serializedPubkey)`  
 +`identity = multihashEncode(hashedPubkey)`  
 +
 +Where previously the second step used scrypt to produce an obSeed from a seed.
 +
 +See the bip39 proposal [here](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) for a explanation of how a seed is generated from a mnemonic; put simply it is a pbkdf2 derivation from the mnemonic string with 2048 iterations and the salt 'mnemonic'.
 +##Rationale
 +I have chosen hmac-sha-256 as it is the current industry standard for cryptographic hash functions, and given that the input seed is already random and over 32 bytes there is no need for stretching or artificial computational difficulty. 
 +
 +Using hmac-sha-256 was discussed with lead developers Chris Pacia and Tyler Smith, and sha-256 was briefly considered instead, but we decided on using a hmac to maintain symmetry with Bitcoin BIP32 which derives seeds in a similar context.
 +
 +The salt choice, ie the second input to hmac-sha-256, is mostly irrelevant, it simply needs to be application specific: "OpenBazaar seed" was suggested by Chris Pacia.
 +
 ## Test Vectors
 +I have created a replacement IdentityKeyFromSeed function for the Go implementation: https://github.com/duomarket/identity-obip . This contains the following test vector (using terminology from pseudocode above):  
 +Mnemonic: `mule track design catch stairs remain produce evidence cannon opera hamster burst`  
 +Seed: `5WH3Ia3gtIkXIbiA8K4nquzYDI96dx4oLXo6nVcuZJCAGIgePygVy/XxgOvKzt1Uonx82wJbGftwI4VRxGON0A==`  
 +obSeed: `SZIoZF0SDRW1AIsdoLnbqJjfMoAB6gPAvoSmTEHSBf8=`  
 +serializedIdentityPrivKey (sk.Bytes()): `CAESYEmSKGRdEg0VtQCLHaC526iY3zKAAeoDwL6EpkxB0gX/G4M5owPNjPKUW2bIms` `KfqQ55cx1nAAaUKEeRr0BO6x8bgzmjA82M8pRbZsiawp+pDnlzHWcABpQoR5GvQE7rHw==`  
 +serializedIdentityPubKey (sk.GetPublic().Bytes()): `CAESIBuDOaMDzYzylFtmyJrCn6kOeXMdZwAGlChHka9ATusf`  
 +PeerID (base58): `Qmci4gUBa3YQf9Nss3gqPKpyB1jPtojViju7adpfkUnfor`  
 +
