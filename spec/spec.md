The DID DHT Method Specification 1.0
==================

**Specification Status**: Implementer's Draft

**Latest Draft**: [https://did-dht.com](https://did-dht.com)

**Registry**: [https://did-dht.com/registry](https://did-dht.com/registry)

**Draft Created**: October 20, 2023

**Last Updated**: July 25, 2024

**Editors**:
~ [Gabe Cohen](https://github.com/decentralgabe)
~ [Daniel Buchner](https://github.com/csuwildcat)

**Contributors**:
~ [Moe Jangda](https://github.com/mistermoe)
~ [Frank Hinek](https://github.com/frankhinek)
~ [Henry Tsai](https://github.com/thehenrytsai)
~ [Kendall Weihe](https://github.com/KendallWeihe)

**Participate**:
~ [GitHub repo](https://github.com/TBD54566975/did-dht)
~ [File a bug](https://github.com/TBD54566975/did-dht/issues)
~ [Commit history](https://github.com/TBD54566975/did-dht/commits/main)

## Abstract

A DID Method [[spec:DID-CORE]] based on [[ref:DNS Resource Records]] and [[ref:Mainline DHT]], identified by the prefix
`did:dht`.

<style id="protocol-stack-styles">
  #protocol-stack-styles + table {
    display: table;
    width: 400px;
    border-radius: 4px;
    box-shadow: 0 1px 3px -1px rgb(0 0 0 / 80%);
    overflow: hidden;
  }
  #protocol-stack-styles + table tr, #protocol-stack-styles + table td {
    border: none;
  }
  #protocol-stack-styles + table tr {
    text-shadow: 0 1px 2px rgb(255 255 255 / 75%);
  }
  #protocol-stack-styles + table tr:nth-child(1) {
    background: hsl(149deg 100% 86%);
  }
  #protocol-stack-styles + table tr:nth-child(2) {
    background: hsl(0deg 100% 82%);
  }
  #protocol-stack-styles + table tr:nth-child(3) {
    background: hsl(198deg 100% 87%);
  }
</style>

:--------------------------------------------------------: |
DID DHT                                                    |
[DNS RRs](https://datatracker.ietf.org/doc/html/rfc1035)   |
[Mainline DHT](https://en.wikipedia.org/wiki/Mainline_DHT) |

DID DHT makes use of [[ref:Mainline DHT]], specifically [[ref:BEP44]] to store signed mutable records.
This DID method uses [[ref:DNS Resource Records]] to efficiently represent _[[ref:DID Documents]]_.

[[def:Mainline]] is in use for the following reasons:

1. It has a proven track record of 15 years.
2. It is the biggest DHT in existence with an estimated 10 million servers.
3. It retains data for multiple hours at no cost.
4. It has been implemented in most languages and is stable.

The syntax of the identifier and accompanying data model used by the protocol is conformant with the [[spec:DID-Core]]
specification and shall be registered with the [[spec:DID-Spec-Registries]].

## Conformance

The keywords MAY, MUST, MUST NOT, RECOMMENDED, SHOULD, and SHOULD NOT in this document are to be interpreted as
described in [BCP 14](https://www.rfc-editor.org/info/bcp14) [[spec:RFC2119]] [[spec:RFC8174]] when, and only when,
they appear in all capitals, as shown here.

## Terminology

[[def:Decentralized Identifier, Decentralized Identifier, DID, DIDs, DID Document, DID Documents]]
~ A [W3C specification](https://www.w3.org/TR/did-core/) describing an _identifier that enables verifiable, 
decentralized digital identity_. A DID identifier is associated with a JSON document containing cryptographic keys,
services, and other properties outlined in the specification.

[[def:DID Suffix, Suffix]]
~ The unique identifier string within a DID URI (e.g., the part after `did:dht:`). For DID DHT the suffix is 
the [[ref:z-base-32]] encoded public key portion of the [[ref:Identity Key]].

[[def:Identity Key Pair, Identity Key]]
~ An [Identity Key Pair](#identity-key-pair) is an [[ref:Ed25519]] key pair required to authenticate all records in
[[ref:Mainline DHT]]. The public key portion is encoded using [[ref:z-base-32]] and represented in the [[ref:DID Suffix]].
The public key, also known as the [[ref:Identity Key]] is guaranteed to be present in each `did:dht` document.

[[def:DNS Resource Records, DNS Resource Record, DNS Packet]]
~ An efficient format for representing [[ref:DID Documents]] and providing semantics pertinent to DID DHT,
such as TTLs, caching, and different record types (e.g., `NS`, `TXT`). Follows [[spec:RFC1035]].

[[def:Mainline DHT, DHT, Mainline, Mainline Server]]
~ [Mainline DHT](https://en.wikipedia.org/wiki/Mainline_DHT) is the name given to the 
[Distributed Hash Table](https://en.wikipedia.org/wiki/Distributed_hash_table) used by the 
[BitTorrent protocol](https://github.com/bittorrent/bittorrent.org). It is a distributed system for storing and
finding data on a peer-to-peer network. It is based on [Kademlia](https://en.wikipedia.org/wiki/Kademlia) and is
primarily used to store and retrieve peer data. It is estimated to have tens of millions of concurrent
active users consistently.

[[def:Gateway, Gateways, DID DHT, Bitcoin-anchored Gateway, DID DHT Service]]
~ A server that that facilitates [[ref:Mainline]] and DID DHT operations. The gateway may offer a set of 
APIs to interact with the DID DHT method features such as providing [guaranteed retention](#retained-did-set),
[historical resolution](#historical-resolution), and other features. Gateways can make themselves
discoverable via a [[ref:Gateway Registry]].

[[def:Gateway Registry, Gateway Registries]]
~ A system used to aid in the discovery of [[ref:Gateways]]. One such system is the 
[registry provided by this specification](registry/index.html#gateways).

[[def:Republish, Republishing, Republishes]]
~ The action that keeps a record alive in [[ref:Mainline]], which offers a limited duration (approximately 2 hours)
for retaining records in the DHT. See [Republishing Data](#republishing-data).

[[def:Client, Clients]]
~ A client is a piece of software that is responsible for generating a `did:dht` and submitting it to a 
[[ref:Mainline Server]] or [[ref:Gateway]]. Notably, a client has the ability to sign messages with the
private-key portion of an [[ref:Identity Key Pair]].

[[def:Retained DID Set, Retained Set, Retention Set, Retention Sets]]
~ The set of DIDs that a [[ref:Gateway]] is retaining and thus is responsible for [[ref:republishing]].
See [Retained DID Set](#retained-did-set).

[[def:Retention Challenge, Retention Challenges, Retention Solution, Retention Solutions]]
~ A _Retention Challenge_ refers to a computational cost associated with a set of guarantees to be provided to a
DID controller by a [[ref:Gateway]], who submits a corresponding _Retention Solution_. A solution provided by the [[ref:DID]]
controller to a [[ref:Gateway]] which attests both that (1) they control the DID and (2) have done sufficient work to
have a [[ref:Gateway]] retain their DID for a set period of time as a part of its [[ref:Retention Set]].

[[def:Sequence Number, Sequence Numbers, Sequence]]
~ A sequence number, or `seq`, is a property of a mutable item as defined in [[ref:BEP44]]. It is a 64-bit integer that
increases in a consistent, unidirectional manner, ensuring that items are ordered sequentially. This specification
requires that sequence numbers are [[ref:Unix Timestamps]] represented in seconds.

[[def:Unix Timestamp, Unix Timestamps]]
~ A value that measures time by the number of non-leap seconds that have elapsed since 00:00:00 UTC on 1 January 1970,
the Unix epoch.

## DID DHT Method Specification

### Format

The format for `did:dht` conforms to the [[spec:DID Core]] specification. It consists of the `did:dht` prefix followed by 
the [[ref:Mainline]] identifier. The [[ref:Mainline]] identifier is a [[ref:z-base-32]]-encoded [[ref:Ed25519]] public
key, which we refer to as an [[ref:Identity Key]].

```text
did-dht-format := did:dht:<z-base-32-value>
z-base-32-value := [a-km-uw-z13-9]+
```

Alternatively, one can interpret the encoding rules as a series of transformation functions on the raw public key bytes:

```text
did-dht-format := did:dht:Z-BASE-32(raw-public-key-bytes)
```

### Identity Key Pair

A unique property of DID DHT is its dependence on a single non-rotatable key, which we refer to as an _Identity Key Pair_,
whose public key is referred to as the _Identity Key_. This requirement stems from [[ref:BEP44]], particularly within the
section on _Mutable Items_:

> Mutable items can be updated, without changing their DHT keys. To authenticate that only the original publisher can
update an item, it is signed by a private key generated by the original publisher. The target ID mutable items are
stored under the SHA-1 hash of the public key (as it appears in the put message).

This mechanism, as detailed in [[ref:BEP44]], ensures that all entries in the DHT are authenticated through a private
key unique to the initial publisher. Consequently, DHT records, including DID DHT Documents, are
_independently verifiable_. This independence implies that trust in a specific [[ref:Mainline]] or [[ref:Gateway]] server
for providing unaltered messages is unnecessary. Instead, all clients have the ability to verify messages themselves. This
approach significantly mitigates risks associated with other DID methods, where a compromised server or
[DID resolver](https://www.w3.org/TR/did-core/#choosing-did-resolvers) has the ability to tamper with a [[ref:DID Document]],
in a manner that could be undetectable by a client.

At present, [[ref:Mainline]] exclusively supports the [[ref:Ed25519]] signature system. In turn, [[ref:Ed25519]]-based
keys are required by DID DHT and used to uniquely identify DID DHT Documents. DID DHT identifiers are formed by 
concatenating the `did:dht:` prefix with a [[ref:z-base-32]] encoded public key (the [[ref:Identity Key]], which acts as 
its [[ref:suffix]]. Identity Keys ****MUST**** have the identifier `0` as both its Verification Method `id` and JWK `kid` 
[[spec:RFC7517]]. Identity Keys ****MUST**** have the [Verification Relationships](#verification-relationships) 
_Authentication_, _Assertion_, _Capability Invocation_, and _Capability Delegation_.

While DID DHT requires at least one [[ref:Ed25519]] key pair, a DID DHT Document can include any number of additional keys.
Additional key types ****MUST**** be registered in the [Key Type Index](registry/index.html##key-type-index).

As a unique consequence of the requirement of the Identity Key, DID DHT Documents are able to be partially-resolved
without contacting [[ref:Mainline]] or [[ref:Gateway]] servers, though it is ****RECOMMENDED**** that deterministic
resolution is only used as a fallback mechanism. Similarly, the requirement of an Identity Key enables
[interoperability with other DID methods](#interoperability-with-other-did-methods).

### DIDs as DNS Records

In this scheme, we encode the [[ref:DID Document]] as
[DNS Resource Records](https://en.wikipedia.org/wiki/Domain_Name_System#Resource_records). These resource records make
up a DNS packet [[spec:RFC1034]] [[spec:RFC1035]], which is then stored in the [[ref:DHT]] encoded as a [[ref:BEP44]] payload.

| Name         | Type | TTL    | Rdata                                                        |
| ------------ | ---- | ------ | ------------------------------------------------------------ |
| _did.`<ID>`. | TXT  |  7200  | v=0;vm=k0,k1,k2;auth=k0;asm=k1;inv=k2;del=k2;svc=s0,s1,s2    |
| _k0._did.    | TXT  |  7200  | id=0;t=0;k=`<unpadded-b64url>`                               |
| _k1._did.    | TXT  |  7200  | id=abcd;t=1;k=`<unpadded-b64url>`                            |
| _k2._did.    | TXT  |  7200  | t=1;k=`<unpadded-b64url>`                                    |
| _s0._did.    | TXT  |  7200  | id=domain;t=LinkedDomains;se=https://foo.com                 |
| _s1._did.    | TXT  |  7200  | id=dwn;t=DecentralizedWebNode;se=https://dwn.tbddev.org/dwn5 |

::: note
The ****RECOMMENDED**** TTL value is 7200 seconds (2 hours), the default TTL for [[ref:Mainline]] records.
:::

- The [Root Record](#root-record) serves as a "map" to reconstruct a [[ref:DID Document]] from a DNS packet.
This record contains a _version_ number indicating the version of this specification against which it is created.

- Additional records like `svc` (services), `vm` ([Verification Methods](#verification-methods)), and
[Verification Relationships](#verification-relationships) (e.g., authentication, assertion, etc.) are 
represented as additional records in the format `<ID>._did.`. These records contain the zero-indexed value of 
each `key` or `service` as attributes.

- All resource record names, aside from the [Root Record](#root-record) and optional 
[Authoritative Gateway Records](#designating-authoritative-gateways), ****MUST**** end in `_did.`.

- The DNS packet ****MUST**** set the _Authoritative Answer_ flag since this is always an _Authoritative_ packet.

- `TXT` records ****MAY**** exceed 255 characters as per [[spec:RFC1035]]. Records exceeding 255 characters are
represented as multiple strings, which can be concatenated to a single value upon DID Document reconstruction.

- DNS packets ****MUST**** be compressed as per [[spec:RFC1035]] 
[section 4.1.4](https://datatracker.ietf.org/doc/html/rfc1035#section-4.1.4) before transmission.

#### Root Record

The root record is a special record which serves as instructions on how to reconstruct a [[ref:DID Document]],
by providing a [property mapping](#property-mapping) for a [[ref:DID Document]], along with containing pertinent
metadata such as a version identifier.

- The root record's **name** ****MUST**** be of the form, `_did.<ID>.`, where `ID` is the [[ref:Mainline]]
identifier associated with the DID (i.e. `did:dht:<ID>` becomes `_did.<ID>.`).

- The root record's **type** is `TXT` indicating a Text record.

- The root record's **rdata** is represented by the form `v=M;vm=N;auth=O;asm=P;inv=Q;del=R;svc=S` where
`M` is the version of the DNS packet representation defined by this specification. `N` is the set 
[Verification Method](#verification-methods) identifiers (e.g., `k0,k1`) present in the document, always
containing at least `k0`. `O`, `P`, `Q`, and `R` contain the set of Verification Method resource 
aliases (e.g., `k0`) for each [Verification Relationship](#verification-relationships). `S` contains
the set of Service resource aliases (e.g., `s0`) for each [Service](#services).

Additionally:

- A version number ****MUST**** be present. The version number for this specification version 
  is **0** (e.g., `v=0`). The version number is not present in the corresponding DID Document.

- The `vm` property ****MUST**** always contain _at least_ the [[ref:Identity Key]] represented by `k0`.

- Verification Relationships (`auth`, `asm`, `agm`, `inv`, `del`) without any members ****MUST**** be omitted.

- If there are no [Services](#services), the `svc` property ****MUST**** be omitted.

**Example Root Record**:

| Name                                                       | Type | TTL    | Rdata                                                     |
| ---------------------------------------------------------- | ---- | ------ | --------------------------------------------------------- | 
| _did.o4dksfbqk85ogzdb5osziw6befigbuxmuxkuxq8434q89uj56uyy. | TXT  |  7200  | v=0;vm=k0,k1,k2;auth=k0;asm=k1;inv=k2;del=k2;svc=s0,s1,s2 |

### Property Mapping

The following section describes mapping a [[ref:DID Document]] to a DNS packet's **rdata**.

- Resource names are aliased with zero-indexed values (e.g., `k0`, `k1`, `s0`, `s1`).

- Verification Methods, Verification Relationships, and Services are separated by a semicolon (`;`), while
values within each property are separated by a comma (`,`).

- Across all properties, distinct elements are separated by semicolons (`;`) while array elements are separated by
commas (`,`).

- Additional properties not defined by this specification ****MAY**** be represented in a [[ref:DID Document]] 
and its corresponding DNS packet if the properties are registered in the
[additional properties registry](registry/index.html#additional-properties).

The subsequent instructions serve as a reference for mapping DID Document properties to
[DNS Resource Records](https://en.wikipedia.org/wiki/Domain_Name_System#Resource_records):

#### Identifiers

##### Controller

A [DID controller](https://www.w3.org/TR/did-core/#did-controller) ****MAY**** be present in a `did:dht` document.

- The [Controller](https://www.w3.org/TR/did-core/#did-controller) record's **name** is represented as a `_cnt._did.`.

- The [Controller](https://www.w3.org/TR/did-core/#did-controller) record's **type** is `TXT` indicating a Text record.

- The [Controller](https://www.w3.org/TR/did-core/#did-controller) record's **data** is represented as a comma-separated
list of controller DID identifiers.

To ensure that the DID controller is authorized to make changes to the DID Document, the controller for the
[[ref:Identity Key]] Verification Method ****MUST**** be contained within the controller property.

**Example Controller Record**:

| Name       | Type | TTL  | Rdata            |
| ---------- | ---- | ---- | ---------------- |
| _cnt._did. | TXT  | 7200 | did:example:abcd |

##### Also Known As

A `did:dht` document ****MAY**** have multiple identifiers using the [alsoKnownAs](https://www.w3.org/TR/did-core/#also-known-as) property.

- The [Also Known As](https://www.w3.org/TR/did-core/#also-known-as) record's **name** is represented as a `_aka._did.`.

- The [Also Known As](https://www.w3.org/TR/did-core/#also-known-as) record's **type** is `TXT` indicating a Text record.

- The [Also Known As](https://www.w3.org/TR/did-core/#also-known-as) record's **data** is represented as a
comma-separated list of DID identifiers.

**Example AKA Record**:

| Name       | Type | TTL  | Rdata                              |
| ---------- | ---- | ---- | ---------------------------------- |
| _aka._did. | TXT  | 7200 | did:example:efgh,did:example:ijkl  |

#### Verification Methods

- Each [Verification Method](https://www.w3.org/TR/did-core/#verification-methods) record's **name** is represented 
as a `_kN._did.` record where `N` is the zero-indexed positional index of a given Verification Method (e.g., `_k0`, `_k1`).

- Each [Verification Method](https://www.w3.org/TR/did-core/#verification-methods) record's **type** is `TXT`,
indicating a Text record.

- Each [Verification Method](https://www.w3.org/TR/did-core/#verification-methods) record's **rdata** is represented by the form
`id=M;t=N;k=O;a=P` where `M` is an **optional** Verification Method Identifier, `N` is the index of the key's type from the 
[key type index](registry/index.html#key-type-index), `O` is the unpadded base64URL [[spec:RFC4648]] representation of the public
key, and `P` is the `JWK` `alg` identifier of the key. It is important to note that the value of `O` represents the byte
representation of the public key itself rather than an encoded JWK.

  - The [[ref:Identity Key]] ****MUST**** always be at index `_k0`.

  - It is ****RECOMMENDED**** to omit Verification Method ID values from the DNS packet representation, as they can be 
  deterministically computed according to the rules specified in the [representing keys section](#representing-keys).
  Additionally:

    - For the [[ref:Identity Key]], the `id` value ****MUST**** be set to `0`.

    - For all other keys, if an `id` value is present in the record, it is used as the `id` value for the Verification
    Method. If there is no `id` value set, the JWK Thumbprint [[spec:RFC7638]] is used as the Verification Method ID.

  - The algorithm identifier (`a` property) ****MUST**** be omitted from the record if it is assigned to the _default value_
  specified in the [key type index](registry/index.html#key-type-index). If it differs from the default value, it
  ****MUST**** be present.

- [Verification Methods](https://www.w3.org/TR/did-core/#verification-methods) ****MAY**** have an _optional_ **controller** 
property represented by `c=C` where `C` is the identifier of the verification method's controller (e.g., `t=N;k=O;c=C`). If 
omitted, it is assumed that the controller of the Verification Method is the [[ref:Identity Key]].

::: note
Controllers are not cryptographically verified by [[ref:Gateways]] or this DID method. This means any DID may choose to list
a controller, even if there is no relationship between the identifiers. As such, DID controllers should be interrogated to 
assert the integrity of their relations.
:::

#### Verification Relationships

- Each [Verification Relationship](https://www.w3.org/TR/did-core/#verification-relationships) is represented as a part
of the root `_did.<ID>.` record (see: [Root Record](#root-record)).

The following table acts as a map between Verification Relationship types and their record name:

##### Verification Relationship Index

| Verification Relationship  | Record Name  |
| -------------------------- | ------------ |
| Authentication             | `auth`       |
| Assertion                  | `asm`        |
| Key Agreement              | `agm`        |
| Capability Invocation      | `inv`        |
| Capability Delegation      | `del`        |

The record data is uniform across [Verification Relationships](https://www.w3.org/TR/did-core/#verification-relationships),
represented as a comma-separated list of key references.

**Example Verification Relationship Records**:

| Verification Relationship                          | Rdata in the Root Record                     |
|----------------------------------------------------|----------------------------------------------|
| "authentication": ["did:dht:example#0", "did:dht:example#HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ"] | auth=0,HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ |
| "assertionMethod": ["did:dht:example#0", "did:dht:example#HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ"]| asm=0,HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ  |
| "keyAgreement": ["did:dht:example#1"]              | agm=1                                        |
| "capabilityInvocation": ["did:dht:example#0"]      | inv=0                                        |
| "capabilityDelegation": ["did:dht:example#0"]      | del=0                                        |

#### Services

- Each [Service](https://www.w3.org/TR/did-core/#services) record's **name** is represented as a `_sN._did.` record where `N` is
the zero-indexed positional index of the Service (e.g., `_s0`, `_s1`).

- Each [Service](https://www.w3.org/TR/did-core/#services) record's **type** is `TXT` indicating a Text record.

- Each [Service](https://www.w3.org/TR/did-core/#services) record's **data** is represented with the form `id=M;t=N;se=O`
where `M` is the Service's ID, `N` is the Service's `type` and `O` is the Service's URI.

  - Multiple service endpoints ****MAY**** be present. If present, they ****MUST**** be represented as an 
  array (e.g., `id=dwn;t=DecentralizedWebNode;se=https://dwn.org/dwn1,https://dwn.org/dwn2`).

  - Additional properties ****MAY**** be present (e.g., `id=dwn;t=DecentralizedWebNode;se=https://dwn.org/dwn1;sig=1;enc=2`) if the
  properties are registered in the [additional properties registry](registry/index.html#additional-properties).

**Example Service Record**:

| Name      | Type | TTL  | Rdata                                                    |
| --------- | ---- | ---- | -------------------------------------------------------- |
| _s0._did. | TXT  | 7200 | id=dwn;t=DecentralizedWebNode;se=https://example.com/dwn |

Each Service is represented as part of the root record (`_did.<ID>.`) as a list under the key `svc=<ids>` where `ids`
is a comma-separated list of all IDs for each Service.

#### Example

A sample transformation of a fully-featured DID Document to a DNS packet is exemplified as follows:

**DID Document**:

```json
{
  "id": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y",
  "controller": "did:example:abcd",
  "alsoKnownAs": ["did:example:efgh", "did:example:ijkl"],
  "verificationMethod": [
    {
      "id": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0",
      "type": "JsonWebKey",
      "controller": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y",
      "publicKeyJwk": {
        "kid": "0",
        "alg": "EdDSA",
        "crv": "Ed25519",
        "kty": "OKP",
        "x": "r96mnGNgWGOmjt6g_3_0nd4Kls5-kknrd4DdPW8qtfw"
      }
    },
    {
      "id": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ",
      "type": "JsonWebKey",
      "controller": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y",
      "publicKeyJwk": {
        "kid": "HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ",
        "alg": "ES256K",
        "crv": "secp256k1",
        "kty": "EC",
        "x": "KI0DPvL5cGvznc8EDOAA5T9zQfLDQZvr0ev2NMLcxDw",
        "y": "0iSbXxZo0jIFLtW8vVnoWd8tEzUV2o22BVc_IjVTIt8"
      }
    }
  ],
  "authentication": [
    "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0",
    "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ"
  ],
  "assertionMethod": [
    "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0",
    "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#HTsY9aMkoDomPBhGcUxSOGP40F-W4Q9XCJV1ab8anTQ"
  ],
  "capabilityInvocation": [
    "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0"
  ],
  "capabilityDelegation": [
    "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0"
  ],
  "service": [
    {
      "id": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#dwn",
      "type": "DecentralizedWebNode",
      "serviceEndpoint": ["https://example.com/dwn1", "https://example/dwn2"]
    }
  ]
}
```

**DNS Resource Records**:

| Name         | Type | TTL   | Rdata                                                                              |
| ------------ | ---- | ----- | ---------------------------------------------------------------------------------- |
| _did.`<ID>`. | TXT  | 7200  | v=0;vm=k0,k1;auth=k0,k1;asm=k0,k1;inv=k0;del=k0;svc=s0                             |
| _cnt._did.   | TXT  | 7200  | did:example:abcd                                                                   |
| _aka._did.   | TXT  | 7200  | did:example:efgh,did:example:ijkl                                                  |
| _k0._did.    | TXT  | 7200  | t=0;k=afdea69c63605863a68edea0ff7ff49dde0a96ce7e9249eb7780dd3d6f2ab5fc             |
| _k1._did.    | TXT  | 7200  | t=1;k=AyiNAz7y-XBr853PBAzgAOU_c0Hyw0Gb69Hr9jTC3MQ8                                 |
| _s0._did.    | TXT  | 7200  | id=dwn;t=DecentralizedWebNode;se=https://example.com/dwn1,https://example.com/dwn2 |

### Operations

Entries to the [[ref:DHT]] require a signed record as per [[ref:BEP44]]. As such, the key pair used for the [[ref:Mainline]]
identifier is also used to sign the [[ref:DHT]] record.

#### Create

To create a `did:dht` document, the process is as follows:

1. Generate an [[ref:Ed25519]] key pair and encode the public key using the format provided in the [format section](#format).

2. Construct a conformant JSON representation of a [[ref:DID Document]].

    a. The document ****MUST**** include a [Verification Method](https://www.w3.org/TR/did-core/#verification-methods) with
    the [[ref:Identity Key]]. The `id` property of this Verification Method ****MUST**** be `0` and the type of `JsonWebKey`
    as defined by [[ref:VC-JOSE-COSE]]. The key ****MUST**** be represented as a `publicKeyJwk` as per [[spec:RFC7517]]
    with a `kid` of `0`.

    b. The [[ref:Identity Key]] ****MUST**** have the [Verification Relationships](#verification-relationships) 
    _Authentication_, _Assertion_, _Capability Invocation_, and _Capability Delegation_.

    c. The document can include any number of other [core properties](https://www.w3.org/TR/did-core/#core-properties);
    always representing key material as a `JWK` as per [[spec:RFC7517]]. In addition to the properties required by
    the `JWK` specification, the `alg` property ****MUST**** always be present in the DID Document representation.
    The [indexed types registry](registry/index.html#indexed-types) defines default algorithm values per key type.

3. Map the output [[ref:DID Document]] to a DNS packet as outlined in [property mapping](#property-mapping).

4. Compress the DNS packet as per [[spec:RFC1035]] [section 4.1.4](https://datatracker.ietf.org/doc/html/rfc1035#section-4.1.4).

5. Construct a [[ref:BEP44]] conformant mutable put message.
  
   a. `v` ****MUST**** be set to a [[ref:bencoded]] compressed DNS packet from the prior step.

   b. `seq` ****MUST**** be set to the current [[ref:Unix Timestamp]] in **seconds**.

6. Submit the result of to the [[ref:DHT]] via a [[ref:Mainline]] node, or a [[ref:Gateway]], with the identifier created
in step 1.

::: note
This specification **does not** make use of [JSON-LD](https://json-ld.org/). As such, DID DHT Documents ****MUST NOT****
include the `@context` property.
:::

#### Read

To read a `did:dht` document, the process is as follows:

1. Take the [[ref:suffix]] of the DID, that is, the _[[ref:z-base-32]] encoded identifier key_, and submit it to a
[[ref:Mainline]] node or a [[ref:Gateway]].

2. Decode the resulting [[ref:BEP44]] response's `v` value using [[ref:bencode]].

3. Uncompress the DNS packet according to [[spec:RFC1035]] [section 4.1.4](https://datatracker.ietf.org/doc/html/rfc1035#section-4.1.4).

4. Reverse the DNS [property mapping](#property-mapping) process and re-construct a conformant [[ref:DID Document]].

    a. Identify the [[ref:Identity Key]] using the [[ref:suffix]] of the `did:dht` identifier, with record name `_k0._did`,
    and set it's Verification Method ID and JWK `kid` to `0`.

    b. For all other keys, assign the Verification Method ID as follows:

      i. If an `id` value is provided, use that value as the Verification Method ID and JWK `kid`.

      ii. If no `id` value is set, the JWK Thumbprint [[spec:RFC7638]] is used as the Verification Method ID
      and JWK `kid`.

    c. Expand all identifiers (i.e. Verification Methods, Services, etc. `id`s ) to their fully-qualified 
    form (e.g., `did:dht:uodqi99wuzxsz6yx445zxkp8ddwj9q54ocbcg8yifsqru45x63kj#0` 
    as opposed to `0` or `#0`, `did:dht:uodqi99wuzxsz6yx445zxkp8ddwj9q54ocbcg8yifsqru45x63kj#service-1` as opposed
    to `#service-1`).


5. If [NS records](#designating-authoritative-gateways) are present this process should be repeated, making the initial
request against the given [[ref:Gateway]]. The document with the highest [[ref:sequence number]] is to be treated
as authoritative.

::: note
As a fallback, if a `did:dht` value cannot be resolved via the network, it can be expanded to a conformant
[[ref:DID Document]] containing just the [[ref:Identity Key]].
:::

#### Update

Any valid [[ref:BEP44]] record written to the DHT is an update. As long as control of the
[[ref:Identity Key]] is retained any update is made possible by signing and writing records with a unique incremental
[[ref:sequence number]] with [mutable items](https://www.bittorrent.org/beps/bep_0044.html).

It is ****RECOMMENDED**** that updates be infrequent — at least every 2 hours — as DHT caching is highly encouraged.

#### Deactivate

To deactivate a `did:dht` document controllers have multiple options:

1. Let the DHT record expire and cease to publish it.

2. Publish a new DHT record where the `rdata` of the root DNS record is the string `deactivated`.

| Name         | Type | TTL  | Rdata       |
| ------------ | ---- | ---- | ----------- |
| _did.`<ID>`. | TXT  | 7200 | deactivated |

::: note
If you have published your DID through a [[ref:Gateway]], you can contact the operator to have them remove the
record from their [[ref:Retained DID Set]] via the [DID Deactivation API](#deactivating-a-did).
:::

### Rotation

[Verification Method Rotation](https://www.w3.org/TR/did-core/#verification-method-rotation) is a recommended practice
for avoiding the risks associated with [key compromise](#key-compromise). Rotating Verification Methods is straightforward
using the [update](#update) functionality enabled by this specification; however, rotation of the [[ref:Identity Key]] is
not possible given the constraints imposed by [[ref:Mainline]]. To mitigate this limitation, DID DHT controllers have the
option to rotate to a new DID, and thus a new [[ref:Identity Key]], while maintaining a cryptographic linkage between the 
two documents. This linkage can be useful in providing an auditable history of a controller's activity as they move between
root Verification Methods and identifiers.

To establish a cryptographic linkage between the old and new [[ref:DID Documents]], adhere to the following steps:

1. Using the old [[ref:Identity Key]], sign over the new [[ref:Identity Key]] using the [[ref:Ed25519]] variant of the 
EdDSA algorithm [[spec:RFC8032]].

2. Encode the resulting signature data using the unpadded base64URL [[spec:RFC4648]] scheme.

3. Set the resulting string as the data for a new [[ref:DNS Resource Record]], called a _Previous_ record, in
the new DID's record set. 

A `did:dht` Document ****MUST NOT**** have more than **one** _Previous_ record. The _Previous_ record is defined as follows:

- The Previous record's **name** is represented as a `_prv._did.` record.

- The Previous record's **type** is `TXT` indicating a Text record.

- The Previous record's **data** is represented with the form `id=M;s=N` where `M` is the identifier of the previous DID,
and `N` is the unpadded base64URL signature from step (3) above.

| Name       | Type | TTL   | Rdata                                                                                  |
| ---------- | ---- | ----- | -------------------------------------------------------------------------------------- |
| _prv._did. | TXT  | 7200  | id=did:dht:pxoem5sfzxxxrnrwfgiu5i5wc7epouy1jk9zb7ad159dsxbxy8io;s=ol5LbUydL3_PdChE8tVYH-z_NhyFDQlop0agYtjyYbKz_-CYrj_3JGLiFne1e7PruOwf-b91uEFq9R_PgBn-Bg |

The DID controller ****MAY**** include a statement in the old [[ref:DID Document]] indicating the rotation to the new identifier 
by setting the [controller property](#controller) to the new DID. Without the previous record present in the new DID's
record set, the linkage ****MUST NOT**** be considered legitimate.

### Designating Authoritative Gateways

[Gateways](#gateways) provide additional benefits to `did:dht`, such as the ability to
[resolve historical DID Documents](#historical-resolution) or support [type indexing](#type-indexing). To enable
these additional features, `did:dht` documents need to be published to Gateway(s) that with the necessary
capabilities. Whether it's accessing historical states, engaging with type indexes, or utilizing other specialized
features, the [resolution process](#resolving-a-did) must be directed towards a [[ref:Gateway]] that maintains this
supplementary data.

To facilitate the discovery of authoritative [[ref:Gateways]] for a `did:dht` and to ensure consistent access to a DID's
state across different [[ref:Gateways]], DID controllers may designate one or more [[ref:Gateways]] as authoritative
sources for their DID. This designation is accomplished by incorporating
[DNS NS records](https://en.wikipedia.org/wiki/List_of_DNS_record_types#NS) within the DNS packet destined for storage
in the DHT. A DID ****MAY**** have multiple NS records, enhancing redundancy and reliability. The format for these
records is outlined as follows:

- Each Gateway record's **name** is represented as `_did.<ID>.` record, where `ID` represents the suffix of the `did:dht` identifier.

- Each Gateway record's **type** is `NS` indicating a Name Server record.

- Each Gateway record's **data** is represented as a [Fully Qualified Domain Name (FQDN)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name).

| Name         | Type | TTL   | Rdata                                 |
| ------------ | ---- | ----- | ------------------------------------- |
| _did.`<ID>`. | NS   | 7200  | gateway1.example-did-dht-gateway.com. |
| _did.`<ID>`. | NS   | 7200  | gateway2.example-did-dht-gateway.com. |

### Type Indexing

Type indexing is an **optional** feature that enables DIDs to become **discoverable**, by flagging themselves as being
of a particular type. Types are not included as part of the DID Document, but rather as part of the DNS packet. This
allows for DIDs to be indexed by type by [[ref:Gateways]], and for DIDs to be resolved by type.

DIDs can be indexed by type by adding a `_typ._did.` record to the DNS packet. A DID ****MAY**** have **at most** one
type index record. This record is of the following format:

- The Type Index record's **name** is represented as a `_typ._did.` record.

- The Type Index record's **type** is `TXT` indicating a Text record.

- The Type Index record's **data** is represented with the form `id=H,I,J,...N` where the value is a comma-separated
list of integer types from the [indexed types registry](registry/index.html#indexed-types).

**Example Type Index Record**:

| Name       | Type | TTL  | Rdata     |
| ---------- | ---- | ---- | --------- |
| _typ._did. | TXT  | 7200 | id=0,1,2  |

Types can be discovered and registered in the [indexed types registry](registry/index.html#indexed-types).

::: note
Identifying entities through type-based indexing is a relatively unreliable practice. It serves
as an initial step in recognizing the identity linked to a [[ref:DID]]. To validate identity assertions in a more robust
manner, it is essential to delve deeper, employing tools like verifiable credentials and the interrogation of related data.
:::

## Gateways

[[ref:Gateways]] serve as specialized servers, providing a range of DID-centric functionalities that extend
beyond the capabilities of a standard [[ref:Mainline DHT]] servers. This section elaborates on these unique features,
outlines the operational prerequisites for managing a gateway, and discusses various other facets, including the
optional integration of these gateways into a registry system.

::: note
[[ref:Gateways]] may choose to support interoperable methods in addition to `did:dht` as outlined in the
[section on interoperability](#interoperability-with-other-did-methods).
:::

### Discovering Gateways

As an **optional** feature of the DID DHT Method, operators of a [[ref:Gateway]] ****MAY**** choose to make their server
discoverable through a [[ref:Gateway Registry]]. This feature allows for easy location through various internet-based 
discovery mechanisms. [[ref:Gateway Registries]] can vary in nature, encompassing a spectrum from centrally managed 
directories to diverse decentralized systems including databases, ledgers, or other structures.

As a convenience, one such registry is [provided by this specification](registry/index.html#gateways).

### Retained DID Set

As a feature of the DID DHT Method, operators of a [[ref:Gateway]] ****MUST**** support retaining DIDs for extended periods
of time to reduce the burden on DID controllers and [[ref:Clients]] in needing  to [[ref:republish]] their records to
[[ref:Mainline]].

A [[ref:Retained DID Set]] refers to the set of DIDs a [[ref:Gateway]] retains and [[ref:republishes]] to the DHT. 
This feature aims to safeguard equitable access to the resources of [[ref:Gateways]], which are publicly accessible 
and potentially subject to [a high volume of requests](#rate-limiting). The [[ref:Retained DID Set]] is facilitated
by a [[ref:Retention Challenge]], which requires clients to generate a solution for the challenge — a [[ref:Retention Solution]]
— in order to have their write requests accepted. [[ref:Retention Solutions]] act as proof of an amount of work
completed in exchange for a retention guarantee provided by a [[ref:Gateway]] (via the `expiry` property).

The [[ref:Retained DID Set]] aims to provide a fair mechanism that provides numerous benefits for clients while giving
[[ref:Gateway]] operators anti-spam prevention, and control over the rate at which they accept new DIDs, thus enhancing
the overall reliability and effectiveness of [[ref:Gateways]] in managing DIDs.

#### Generating a Retention Solution

A [[ref:Retention Solution]] is a form of [proof of work](https://en.bitcoin.it/wiki/Proof_of_work) bound to a specific
DID identifier, using input values supplied by a given [gateway](registry/index.html#gateways). The proof of work is
performed using the [SHA-256 hashing algorithm](https://en.wikipedia.org/wiki/SHA-2) over the concatenation of the
`did` identifier and random [`nonce`](https://en.wikipedia.org/wiki/Cryptographic_nonce) supplied by the user, and a
`hash` value [supplied by the gateway](#get-the-current-challenge). The source of a given `hash` referred to as a
`hash_source` ****MUST**** be one of the values specified in the [Hash Source Registry](registry/index.html#hash-source).
The result of a given proof of work attempt is referred to as the `retention` value.

The resulting `retention` value is determined to be a valid [[ref:Retention Solution]] based on whether it has the
requisite number of leading zeros defined by the `difficulty`. Difficulty values are
[supplied by the gateway](#get-the-current-challenge), and ****MUST**** be no less than 26 bits of the 256-bit hash value.

The algorithm for generating a [[ref:Retention Solution]] is as follows:

1. Obtain a DID identifier and set it to `DID`.

2. Get the `difficulty` and `hash` value from the server set to `DIFFICULTY` and `HASH` respectively.

3. Generate a random 32-bit integer nonce value set to `NONCE`.

4. Set `RETENTION` equal to the result of (`DID` + `HASH` + `NONCE`).

5. Compute the [SHA-256](https://en.wikipedia.org/wiki/SHA-2) hash over `RETENTION` where `ATTEMPT` = SHA256(`RETENTION`).

6. Inspect the result of `ATTEMPT`, and ensure it has >= `DIFFICULTY` bits of leading zeroes.

    a. If it does, set `RETENTION_SOLUTION` = `ATTEMPT`.

    b. Otherwise, return to step 3.

7. Set `RETENTION_SOLUTION` equal to the concatenation of `ATTEMPT` and `NONCE` separated by a colon (e.g., `ATTEMPT`:`NONCE`).

8. Submit the `RETENTION_SOLUTION` to the [Gateway API](#register-or-update-a-did) for write operations.

:::note
When a [[ref:Client]] submits a valid [[ref:Retention Solution]], conformant [[ref:Gateways]] respond with an `expiry`
timestamp. This timestamp indicates when DID will be evicted from the [[ref:Gateway]]'s
[Retained DID Set](#retained-did-set). [[ref:Clients]] are advised to take note of this `expiry` timestamp and ensure
they solve a new [[ref:Retention Challenge]] before the expiration is reached. By doing so, [[ref:Clients]] can
maintain the continuity of their DID's retention and prevent unintended eviction of their identifier.
:::

#### Validating a Retention Solution

When a [[ref:Gateway]] receives a [[ref:Retention Solution]] as part of a write operation, it ****MUST**** validate
the solution to ensure it meets the required criteria before accepting the write request and providing a retention guarantee.

The algorithm for validating a [[ref:Retention Solution]] is as follows:

1. Extract the `RETENTION_SOLUTION` value from the write request.

2. Retrieve the `DID` identifier associated with the write request.

3. Obtain the current `HASH` and `DIFFICULTY` values used by the [[ref:Gateway]].

4. Construct the `RETENTION_VALUE` by concatenating the `DID`, `HASH`, and the `NONCE` (extracted from the `RETENTION_SOLUTION`).

5. Compute the [SHA-256](https://en.wikipedia.org/wiki/SHA-2) hash of the `RETENTION_VALUE` and set it to `COMPUTED_HASH`.

6. Compare the `COMPUTED_HASH` with the `RETENTION_SOLUTION`:

  a. If the `COMPUTED_HASH` matches the `RETENTION_SOLUTION`, proceed to step 7.

  b. If the `COMPUTED_HASH` does not match the `RETENTION_SOLUTION`, the validation fails, and the write request is rejected.

7. Check if the `COMPUTED_HASH` has the required number of leading zeros specified by the difficulty value:

  a. If the `COMPUTED_HASH` has the required number of leading zeros, the [[ref:Retention Solution]] is considered valid.

  b. If the `COMPUTED_HASH` does not have the required number of leading zeros, the validation fails, and the write
  request is rejected.

If the [[ref:Retention Solution]] is deemed valid, the [[ref:Gateway]] accepts the write request and proceeds to store
the DID document in its [[ref:Retained DID Set]]. The [[ref:Gateway]] generates an `expiry` timestamp based on its
retention policy and includes it in the response to the [[ref:Client]].

By validating the [[ref:Retention Solution]], the [[ref:Gateway]] ensures that the [[ref:Client]] has performed the
necessary proof of work and is eligible for the retention guarantee. This validation process helps maintain the
integrity and fairness of the [[ref:Retained DID Set]] system. It is important for [[ref:Gateways]] to consistently
apply the validation logic and reject any write requests that do not include a valid [[ref:Retention Solution]]. Retention
solutions help prevent abuse and ensures that only [[ref:Clients]] who have invested the required computational effort can benefit
from the retention feature. [[ref:Gateways]] ****SHOULD**** regularly update their hash value to prevent pre-computation
attacks and ensure the uniqueness and freshness of the [[ref:Retention Solutions]]. The recommended hash refresh window
is **10 minutes**.

#### Managing the Retained DID Set

[[ref:Gateways]] supporting [[ref:Retention Set]] feature ****MUST**** provide an `expiry` value represented as a
[[ref:Unix Timestamp]] as a part of the [Gateway API](#gateway-api). This timestamp precisely indicates when the 
[[ref:Gateway]] will cease republishing a particular DID, thus evicting the DID from the [[ref:Gateway]]'s Retained
DID Set. This timestamp establishes a binding agreement between the [[ref:Client]] and the [[ref:Gateway]], and it
****MUST NOT**** be modified once set.

Further, it is ****RECOMMENDED**** that [[ref:Gateway]]'s adhere to the following guidance:

* To guarantee a sensible minimum retention period, it is ****RECOMMENDED**** that [[ref:Gateways]] retain DIDs for at 
least **1 week** starting from the acceptance of [[ref:Retention Solution]].

* [[ref:Gateways]] ****MAY**** choose to offer an extended grace period before evicting DIDs, particularly for those
with a substantial history with the [[ref:Gateway]], such as DIDs exposed through the [Historical Resolution API](#historical-resolution).

* [[ref:Gateways]] ****MAY**** choose to include the `expiry` value as a part of the
[DID Resolution Metadata](https://www.w3.org/TR/did-core/#did-resolution-metadata), during [DID Resolution](#did-resolution),
to aid [[ref:Clients]] in being able to assess whether further proof of work is required.

* [[ref:Gateways]] ****SHOULD**** treat each new valid [[ref:Retention Solution]] as extending a DID's retention period.
When a [[ref:Client]] submits a fresh [[ref:Retention Solution]] for a DID already in the [[ref:Retained DID Set]], the
[[ref:Gateway]] ****SHOULD**** update the DID's expiry timestamp, effectively resetting the expiry window and granting a
renewed retention period.

* When a [[ref:Gateway]] reaches its storage capacity or experiences high load, it is ****RECOMMENDED**** that it
significantly increases the `difficulty` parameter for [[ref:Retention Challenges]]. By raising the `difficulty` the
[[ref:Gateway]] can effectively limit the acceptance of new DIDs into its [[ref:Retained DID Set]], ensuring the
stability and performance of the server under resource constraints.

### Gateway API

A conformant [[ref:Gateway]] ****MUST**** support the API defined in the following sections. 

As a convenience, this API is made available via an [OpenAPI document](#open-api-definition).

#### DHT

The DHT API, drawing inspiration from [[ref:Pkarr]], is built as an abstraction layer over [[ref:Mainline DHT]]
which enables the storage of [[ref:DNS Resource Records]] within [[ref:BEP44]] payloads. It is important to exercise
caution when utilizing the DHT API, as it **does not offer any assurances regarding data retention**.

For this API, [[ref:Gateways]] will need to set up [Cross-origin resource sharing (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
headers as follows:

- `Access-Control-Allow-Origin`: `*`
- `Access-Control-Allow-Methods`: `GET`, `PUT`, `OPTIONS`

##### Put

On receiving a `PUT` request the server verifiers the `sig` and submits a mutable put to [[ref:Mainline]] as per
[[ref:BEP44]].

- **Method**: `PUT`
- **Path**: `/:id`
  - `id` - **string** - **REQUIRED** - The [[ref:z-base-32]] encoded [[ref:Identity Key]], equivalent to the suffix of
  the DID DHT identifier.
- **Request Body**: `application/octet-stream`
  - The binary representation of `<sig><seq>[<v>]` where:
    - `sig` - represents the 64-byte [[ref:BEP44]] payload signature.
    - `seq` - represents the 8-byte unsigned 64-bit integer big-endian representation of a [[ref:BEP44]] sequence number.
    - `v` - represents 0-1000 bytes of a [[ref:bencoded]] compressed DNS packet.
- **Returns**: `application/json`
  - `200` - Success.
  - `400` - Bad request if the `sig` is not valid.
  - `500` - Internal server error.

##### Get

When it receives a `GET` request the server submits a mutable get query to [[ref:Mainline]] as per [[ref:BEP44]].

- **Method**: `GET`
- **Path**: `/:id`
  - `id` - **string** - **REQUIRED** - The [[ref:z-base-32]] encoded [[ref:Identity Key]], equivalent to the suffix of
   the DID DHT identifier.
- **Returns**: `application/octet-stream`
  - `200` - Success. The binary representation of `<sig><seq>[<v>]` where:
    - `sig` - represents the 64-byte [[ref:BEP44]] payload signature.
    - `seq` - represents the 8-byte unsigned 64-bit integer big-endian representation of a [[ref:BEP44]] sequence number.
    - `v` - represents 0-1000 bytes of a [[ref:bencoded]] compressed DNS packet.
  - `404` - Record not found.

#### Get the Current Challenge

Challenge is exposed as an endpoint to facilitate functionality pertaining to the [Retained DID Set](#retained-did-set),
surfacing a [[ref:Retention Challenge]].

- **Method**: `GET`
- **Path**: `/challenge`
- **Returns**: `application/json`
  - `200` - Success.
    - `hash` - **string** - **REQUIRED** - The current hash which is to be used as input for computing a [[ref:Retention Solution]].
    - `hash_source` - **string** - **REQUIRED** - The source of the hash as defined by the [Hash Source Registry](registry/index.html#hash-source).
    - `difficulty` - **integer** - **REQUIRED** - The challenge's current difficulty represents the number of
    bits of leading zeros the resulting hash must contain.
    - `expiry` - **integer** - An _approximate_ expiry date-time value, if a valid [[ref:Retention Solution]] is
    submitted against this challenge, which ****MUST**** be a [[ref:Unix Timestamp]] in seconds. The _precise_ expiry
    date-time value is returned as a part of a [PUT operation](#register-or-update-a-did).
  - `501` - [[ref:Retention Sets]] are not supported by this gateway.
  - `503` - [[ref:Retention Sets]] have been temporarily disabled.

**Example Challenge Response**:

```json
{
  "hash": "000000000000000000022be0c55caae4152d023dd57e8d63dc1a55c1f6de46e7",
  "hash_source": "bitcoin",
  "difficulty": 26,
  "expiry": 1715578600
}
```

#### Register or Update a DID

- **Method**: `PUT`
- **Path**: `/did/:id`
  - `id` - **string** - **REQUIRED** - ID of the DID to publish.
- **Request Body**: – application/json
  - `did` - **string** - **REQUIRED** - The DID to register or update.
  - `sig` - **string** - **REQUIRED** - An unpadded base64URL-encoded signature of the [[ref:BEP44]] payload.
  - `seq` - **integer** - **REQUIRED** - A [[ref:sequence number]] for the request. This number ****MUST**** be unique
   for each DID operation, which ****MUST**** be a [[ref:Unix Timestamp]] in seconds.
  - `v` - **string** - **REQUIRED** - An unpadded base64URL-encoded [[ref:bencoded]] compressed DNS packet containing
   the DID Document.
  - `retention_solution` - **string** - **OPTIONAL** - A retention solution calculated according to the
  [retention solution algorithm](#generating-a-retention-solution).
- **Returns**: `application/json`
  - `202` - Accepted. The server has accepted the request as valid and will publish to the DHT.
    - `expiry` - **string** - **OPTIONAL** – The [[ref:Unix Timestamp]] in seconds indicating when the DID will be evicted
    from the [[ref:Gateway]]'s [[ref:Retained DID Set]].
  - `400` - Invalid request, including an invalid retention solution.
  - `401` - Invalid signature.
  - `409` - DID already exists with a higher [[ref:sequence number]]. DID may be accepted if the [[ref:Gateway]]
  supports [historical resolution](#historical-resolution).
  - `503` - [[ref:Retention Sets]] have been temporarily disabled.

**Example DID Registration Request**:

```json
{
  "did": "did:dht:example",
  "sig": "<unpadded-base64URL-encoded-signature>",
  "seq": 1234,
  "v": "<unpadded-base64URL-encoded bencoded DNS packet>",
  "retention_solution": "000000270b7c547aff552f302aad08200b3db815b69bc11ec3f263b7ef755a52"
}
```

Upon receiving a request to register a DID the [[ref:Gateway]] ****MUST**** perform the following steps:

* Verify the signature of the request and, if valid, publish the [[ref:BEP44]] payload to the DHT.

* If one is provided, [validate the `retention_solution`](#validating-a-retention-solution) and, if valid, 
add the DID to the [[ref:Gateway]]'s [[ref:Retained DID Set]].

* If the DNS packet contain a `_typ._did.` record, update the specified indexes with the DID.

:::note
Requests without a `retention_solution` have **no retention guarantees**.
:::

#### Resolving a DID

- **Method**: `GET`
- **Path**: `/did/:id`
  - `id` - **string** - **REQUIRED** - ID of the DID to resolve.
- **Returns**: `application/json`
  - `200` - Success.
    - `did` - **object** - **REQUIRED** - A JSON object representing the DID Document.
    - `dht` - **string** - **REQUIRED** - An unpadded base64URL-encoded representation of the full [[ref:BEP44]]
    payload, represented as 64 bytes sig,
    8 bytes u64 big-endian seq, and 0-1000 bytes of v concatenated, enabling independent verification.
    - `types` - **array** - **OPTIONAL** - An array of [type integers](#type-indexing) for the DID.
    - `sequence_numbers` - **array** - **OPTIONAL** - An sorted array of integers representing seen
    [[ref:sequence numbers]], used with [historical resolution](#historical-resolution).
    - `expiry` - **integer** - **OPTIONAL** - The [[ref:Unix Timestamp]] in seconds indicating when the DID will be evicted
    from the [[ref:Gateway]]'s [[ref:Retained DID Set]].
    - `400` - Invalid request.
  - `404` - DID not found.

**Example DID Resolution Response**:

```json
{
  "did": {
    "id": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y",
    "verificationMethod": [
      {
        "id": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0",
        "type": "JsonWebKey",
        "controller": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y",
        "publicKeyJwk": {
          "kid": "0",
          "alg": "EdDSA",
          "crv": "Ed25519",
          "kty": "OKP",
          "x": "r96mnGNgWGOmjt6g_3_0nd4Kls5-kknrd4DdPW8qtfw"
        }
      }
    ],
    "authentication": [
      "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0"
    ],
    "assertionMethod": [
      "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y#0"
    ]
  },
  "dht": "<unpadded-base64URL-encoded BEP44 payload as [sig][seq][v]>",
  "types": [1, 4],
  "sequence_numbers": [1700356854, 1700461736],
  "expiry": 1715579444
}
```

Upon receiving a request to resolve a DID, the [[ref:Gateway]] ****MUST**** perform the following steps:

* Query local storage for the [[ref:DID]] record set, and if the [[ref:Gateway]] [is authoritative](#designating-authoritative-gateways)
for the DID, return the stored DIDs.

* If the [[ref:Gateway]] is not authoritative for the DID, the [[ref:Gateway]] ****MUST**** query the DHT for the 
[[ref:DID Document]], and if found, return the record set. If the records are not found in the DHT, the [[ref:Gateway]] 
****MAY**** fall back to its local storage.

* If the DNS Packet contains a `_typ._did.` record, the [[ref:Gateway]] ****MUST**** return the
associated `types` value.

* If the [[ref:Gateway]] supports [historical resolution](#historical-resolution) and has multiple stored
[[ref:Sequence Numbers]] for the DID, the [[ref:Gateway]] ****MUST**** return the associated `sequence_number` value(s).

* If the DID is in the [[ref:Gateway]]'s [[ref:Retained DID Set]], the [[ref:Gateway]] ****MUST**** return the
associated `expiry` value.

:::note
This API returns a `dht` property which matches the payload of a [DHT GET Request](#DHT),
when encoded as an unpadded base64URL string. Implementers are ****RECOMMENDED**** to verify the integrity of the
response using the `dht` data and reconstruct the DID Document themselves. The `did` property is provided as a utility
which, without independent verification, ****MUST NOT**** be trusted.
:::

##### Historical Resolution

[[ref:Gateways]] ****MAY**** choose to support _historical resolution_, which is to surface different versions of the 
same [[ref:DID Document]], sorted by [[ref:Sequence Number]], according to the rules set out in the section on 
[conflict resolution](#conflict-resolution).

Upon [resolving a DID](#resolving-a-did), the [[ref:Gateway]] will return the value `sequence_numbers` if there exists
historical state for a given [[ref:DID]]. The following API can be used with specific [[ref:Sequence Numbers]] to fetch
historical state:

- **Method**: `GET`
- **Path**: `/did/:id?seq=:sequence_number`
  - `id` - **string** - **REQUIRED** - ID of the DID to resolve
  - `seq` - **integer** - **OPTIONAL** - [[ref:Sequence number]] of the DID to resolve
- **Returns**: `application/json`
  - `200` - Success.
    - `did` - **object** - **REQUIRED** - A JSON object representing the DID Document.
    - `dht` - **string** - **REQUIRED** - An unpadded base64URL-encoded representation of the full [[ref:BEP44]]
    payload, represented as 64 bytes sig, 8 bytes u64 big-endian seq, and 0-1000 bytes of v concatenated, enabling
    independent verification.
    - `types` - **array** - **OPTIONAL** - An array of [type integers](#type-indexing) for the DID.
    - `expiry` - **integer** - **OPTIONAL** - The [[ref:Unix Timestamp]] in seconds indicating when the DID will be
    evicted from the [[ref:Gateway]]'s [[ref:Retained DID Set]].
  - `400` - Invalid request.
  - `404` - DID not found for the given [[ref:sequence number]].
  - `501` - Historical resolution not supported by this gateway.

#### Deactivating a DID

To intentionally deactivate a DID, as opposed to letting the record cease being published to the DHT, a DID controller
follows the same process as [updating a DID](#register-or-update-a-did), but with a record format outlined in the
[section on deactivation](#deactivate).

Upon receiving a request to deactivate a DID, the Gateway ****MUST**** verify the signature of the request, and if valid,
stop [[ref:republishing]] the DHT. If the DNS Packets contain a `_typ._did.` record, the Gateway ****MUST**** stop
indexing the type(s) for the DID.

#### Type Indexing

**Get Info**

- **Method**: `GET`
- **Path**: `/did/types`
- **Returns**: `application/json`
  - `200` - Success.
    - **array** - An array of objects describing the known types of the following form:
      - `type` - **integer** - **REQUIRED** - An integer representing the [type](#type-indexing).
      - `description` - **string** - **REQUIRED** - A string describing the [type](#type-indexing).
  - `404` - Type indexing not supported.

**Example Type Index Response**:

```json
[
  {
    "type": 1,
    "description": "Organization"
  },
  {
    "type": 7,
    "description": "Financial Institution"
  }
]
```

**Get a Specific Type**

- **Method**: `GET`
- **Path**: `/did/types/:id`
  - `id` - **integer** - **REQUIRED** - The type to query from the index.
  - `offset` - **integer** - **OPTIONAL** - Specifies the starting position from where the type records should be
  retrieved (Default: `0`).
  - `limit` - **integer** - **OPTIONAL** - Specifies the maximum number of type records to retrieve (Default: `100`).
- **Returns**: `application/json`
  - `200` - Success.
    - **array** - **REQUIRED** - An array of DID Identifiers matching the associated type.
  - `400` - Invalid request.
  - `404` - Type not found.
  - `501` - Types not supported by this gateway.

**Example Type Response**:

```json
[
  "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y",
  "did:dht:uodqi99wuzxsz6yx445zxkp8ddwj9q54ocbcg8yifsqru45x63kj"
]
```

A query to the type index returns an array of DIDs matching the associated type. If the type is not found, a `404` is
returned. If no DIDs match the type, an empty array is returned.

## Interoperability With Other DID Methods

As an **optional** extension, some existing DID methods can leverage `did:dht` to expand their feature set. This
enhancement is most useful for DID methods that operate based on a single key and are compatible with the [[ref:Ed25519]]
key format. Users can maintain their current DIDs without any changes by adopting this optional extension. Additionally,
they gain the ability to add extra information to their DIDs by either publishing or retrieving
data from [[ref:Mainline]].

Interoperable DID methods ****MUST**** be registered in
[this specification's registry](registry/index.html#interoperable-did-methods).

## Implementation Considerations

### Conflict Resolution

Per [[ref:BEP44]], [[ref:Gateway]] servers can leverage the `seq` [[ref:Sequence Number]] to handle conflicts:

> [[ref:Gateways]] receiving a put request where `seq` is lower than or equal to what's already stored on the
[[ref:Gateway]], ****MUST**** reject the request. If the [[ref:Sequence Number]] is equal, and the value is also the
same, the [[ref:Gateway]] ****SHOULD**** reset its timeout counter.

When the [[ref:Sequence Number]] is equal, but the value is different, [[ref:Gateways]] need to decide which value
to accept and which to reject. To make this determination [[ref:Gateways]] ****MUST**** compare the payloads
lexicographically to determine a [lexicographical order](https://en.wikipedia.org/wiki/Lexicographic_order), and reject
the payload with a **lower** lexicographical order.

### Size Constraints

[[ref:BEP44]] payload sizes are limited to 1000 bytes. Accordingly, we specify [an efficient representation of a
DID Document](#dids-as-dns-records) and leveraged DNS packet encoding to optimize our payload sizes. With this
encoding format, we recommend additional considerations to minimize payload sizes:

#### Representing Keys

The following guidance on representations of keys and their identifiers using the `JsonWebKey` type defined by
[[ref:VC-JOSE-COSE]] are ****REQUIRED****:

- For the [[ref:Identity Key]], the Verification Method `id` and JWK `id` properties ****MUST**** be set to `0`.

- For all keys besides the [[ref:Identity Key]], the Verification Method `id` and the JWK `kid` are set to the 
provided `id` value, if present. If no `id` value is specified, the Verification Method's `id` value and the JWK
`kid` values are set to the key's JWK Thumbprint calculated according to [[spec:RFC7638]]. 

- [[ref:DID Document]] representations of elliptic curve (EC) keys ****MUST**** include the x- and y-coordinate pair.
To conserve space in the DNS packet representation, compressed point encoding ****MUST**** be used to transmit the
x-coordinate and a sign bit for the y-coordinate. This practice reduces each public key's size from 65 to 33 bytes.

- [[ref:DID Document]] representations ****MUST**** always use fully-qualified identifiers when referring 
to Verification Methods (e.g., `did:dht:uodqi99wuzxsz6yx445zxkp8ddwj9q54ocbcg8yifsqru45x63kj#0` as opposed to `0` or `#0`).

#### Historical Key State

Rotating keys is a widely recommended security practice. However, if you frequently rotate keys in a
[[ref:DID Document]], this increases the document's size due to the accumulation of old keys.
Storage of old keys, in turn, increases the size of the corresponding DNS packet. To manage this overhead while still
distinguishing between currently active keys and those that are no longer in use (but were valid in the past), users ****MAY****
utilize the [service property](https://www.w3.org/TR/did-core/#services). This property allows for the specification 
of services that are dedicated to storing signed records of the historical key states. Following this practice helps 
to keep the [[ref:DID Document]] more concise.

### Republishing Data

[[ref:Mainline]] offers a limited duration (approximately 2 hours) for retaining records in the DHT. To ensure the
verifiability of data signed by a [[ref:DID]], consistent republishing of [[ref:DID Document]] records is crucial. To
address this, it is ****RECOMMENDED**** for [[ref:Clients]] to use [[ref:Retention Challenges]] when interfacing
with [[ref:Gateways]].

### Rate Limiting

To reduce the risk of [Denial of Service Attacks](https://www.cisa.gov/news-events/news/understanding-denial-service-attacks),
spam, and other unwanted traffic, it is ****RECOMMENDED**** that [[ref:Gateways]] require [[ref:Retention Challenges]]
for all requests. The use of [[ref:Retention Challenges]] can act as an attack prevention measure, as it would be
costly to scale retention challenge calculations. [[ref:Gateways]] ****MAY**** choose to explore other rate-limiting
techniques, such as IP limiting, or access-token-based approaches.

### DID Resolution

The process for resolving a DID DHT Document via a [[ref:Gateway]] is outlined in the [read section above](#read).
However, we provide additional guidance for [DID Resolvers](https://www.w3.org/TR/did-core/#dfn-did-resolvers), supplying
[DID Document Metadata](https://www.w3.org/TR/did-core/#did-document-metadata) and 
[DID Resolution Metadata](https://www.w3.org/TR/did-core/#did-resolution-metadata) as follows:

::: todo
[](https://github.com/TBD54566975/did-dht/issues/136)
Register `types`, `gateway`, and `expiry` types in the [DID Specification Registry](https://www.w3.org/TR/did-spec-registries).
::: 

#### DID Document Metadata

* The metadata [`versionId` property](https://www.w3.org/TR/did-core/#dfn-versionid) ****MUST**** be set to the
[[ref:DID Document]] packet's current [[ref:sequence number]].

* The metadata [`created` property](https://www.w3.org/TR/did-core/#dfn-created) ****MUST**** be set to
[XML Datetime](https://www.w3.org/TR/xmlschema11-2/#dateTime) representation of the earliest known [[ref:Sequence Number]]
for the DID.

* The metadata [`updated` property](https://www.w3.org/TR/did-core/#dfn-updated) ****MUST**** be set to the
[XML Datetime](https://www.w3.org/TR/xmlschema11-2/#dateTime) representation of the last known [[ref:Sequence Number]]
for the DID.

* If the [[ref:DID Document]] has [been deactivated](#deactivate) the 
[`deactivated` property](https://www.w3.org/TR/did-core/#dfn-deactivated) ****MUST**** be set to `true`.

* The metadata `types` property ****MUST**** be set to the array of strings representing type values if
[type data](#type-indexing) is present in the [[ref:DID Document]]'s packet.

* The metadata `expiry` property ****MUST**** be set to the integer representing the expiry date-time value
for the DID in the [[ref:Gateway]]'s [Retained DID Set](#retained-did-set).

#### DID Resolution Metadata

* The metadata `gateway` property ****MUST**** be set to the string representing the [[ref:Gateway]]'s URI
from which the DID was resolved. This property is useful in cases where a [DID Resolvers](https://www.w3.org/TR/did-core/#dfn-did-resolvers)
performs resolution against an [Authoritative Gateway](#designating-authoritative-gateways).

## Security and Privacy Considerations

When implementing and using the `did:dht` method, there are several security and privacy considerations to be aware of
to ensure expected and legitimate behavior.

### Data Conflicts

Malicious actors may try to force [[ref:Gateways]] into uncertain states by manipulating the [[ref:Sequence Number]]
associated with a record set. There are three such cases to be aware of:

- **Low Sequence Number** - If a [[ref:Gateway]] has yet to see [[ref:Sequence Numbers]] for a given record it
****MUST**** query its peers to see if they have encountered the record. If a peer is found who has encountered the
record, the record with the latest sequence number must be selected. If the server has encountered greater
[[ref:sequence numbers]] before, the server ****MAY**** reject the record set. If the server supports
[historical resolution](#historical-resolution) it ****MAY**** choose to accept the request and insert the record into
its historically ordered state.

- **Conflicting Sequence Number** - When a malicious actor publishes _valid but conflicting_ records to two different
[[ref:Mainline Servers]] or [[ref:Gateways]]. Implementers are encouraged to follow the guidance outlined in [conflict
resolution](#conflict-resolution).

- **High Sequence Number** - Since [[ref:Sequence Numbers]] ****MUST**** be second representations of a
[[ref:Unix Timestamp]], it is ****RECOMMENDED**** that [[ref:Gateways]] reject [[ref:Sequence Numbers]] that represent
timestamps greater than **2 hours** into the future to mitigate [timing attack](#data-conflicts) risks.

### Data Availability

Given the nature of decentralized distributed systems, there are no firm guarantees that all [[ref:Gateways]] have access
to the same state. To reduce such risks, it is ****RECOMMENDED**** to publish and read from multiple [[ref:Gateways]].
As an **optional** enhancement [[ref:Gateways]] ****MAY**** choose to share state amongst themselves via mechanisms
such as a [gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol).

### Data Authenticity

To enter into the DHT using [[ref:BEP44]] records ****MUST**** be signed by an [[ref:Ed25519]] private key, known as the
[[ref:Identity Key]]. When retrieving records either through a [[ref:Mainline Server]] or a [[ref:Gateway]] is it 
****RECOMMENDED**** that one verifies the cryptographic integrity of the record themselves instead of trusting a server
to have done the validation. Implementers ****MUST NOT**** trust servers that do not return a signature value.

### Key Compromise

Since the `did:dht` uses a single, un-rotatable root key, there is are significant consequences associated with root key
compromise. Such a compromise may be tough to detect without external assurances of identity. Implementers are encouraged
to be aware of this possibility and devise strategies that support entities transitioning to new [[ref:DIDs]] regularly,
such as the mechanism for [rotation](#rotation) noted in this specification.

### Public Data

[[ref:Mainline]] is a public network. As such, there is a risk in storing private, sensitive, or personally identifying
information (PII) on such a network. Storing such sensitive information on the network or in the contents of a `did:dht`
document is strongly discouraged.

### Data Retention

It is ****RECOMMENDED**** that [[ref:Gateways]] implement measures supporting the "[Right to be
Forgotten](https://en.wikipedia.org/wiki/Right_to_be_forgotten)," enabling precise control over the data retention duration.

### Cryptographic Risk

The security of data within the [[ref:Mainline DHT]] which relies on mutable records using [[ref:Ed25519]] keys—is
intrinsically tied to the strength of these keys and their underlying algorithms, as outlined in [[spec:RFC8032]].
Should vulnerabilities be discovered in [[ref:Ed25519]] or if advancements in quantum computing compromise its
cryptographic foundations, the [[ref:Mainline]] method could become obsolete.

## Appendix

### Test Vectors

#### Vector 1

A minimal DID Document.

**Identity Public Key JWK**:

```json
{
  "kid": "0",
  "alg": "EdDSA",
  "crv": "Ed25519",
  "kty": "OKP",
  "x": "YCcHYL2sYNPDlKaALcEmll2HHyT968M4UWbr-9CFGWE"
}
```

**DID Document**:

```json
{
  "id": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo",
  "verificationMethod": [
    {
      "id": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0",
      "type": "JsonWebKey",
      "controller": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo",
      "publicKeyJwk": {
        "kid": "0",
        "alg": "EdDSA",
        "crv": "Ed25519",
        "kty": "OKP",
        "x": "YCcHYL2sYNPDlKaALcEmll2HHyT968M4UWbr-9CFGWE"
      }
    }
  ],
  "authentication": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0"
  ],
  "assertionMethod": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0"
  ],
  "capabilityInvocation": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0"
  ],
  "capabilityDelegation": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0"
  ]
}
```

**DNS Resource Records**:

| Name      | Type | TTL  | Rdata                                                                                   |
| --------- | ---- | ---- | --------------------------------------------------------------------------------------- |
| _did.cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo. | TXT  | 7200 | v=0;vm=k0;auth=k0;asm=k0;inv=k0;del=k0 |
| _k0._did. | TXT  | 7200 | t=0;k=YCcHYL2sYNPDlKaALcEmll2HHyT968M4UWbr-9CFGWE                                       |

#### Vector 2

A DID Document with two keys ([[ref:Identity Key]] and an uncompressed secp256k1 key with a custom `id` value), a service 
with multiple endpoints, a gateway, two types to index, an aka, and controller properties.

**Identity Public Key JWK**:

```json
{
  "kid": "0",
  "alg": "EdDSA",
  "crv": "Ed25519",
  "kty": "OKP",
  "x": "YCcHYL2sYNPDlKaALcEmll2HHyT968M4UWbr-9CFGWE"
}
```

**secp256k1 Public Key JWK**:

With controller: `did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y` and `id` value `sig`.

```json
{
  "kid": "sig",
  "alg": "ES256K",
  "crv": "secp256k1",
  "kty": "EC",
  "x": "1_o0IKHGNamet8-3VYNUTiKlhVK-LilcKrhJSPHSNP0",
  "y": "qzU8qqh0wKB6JC_9HCu8pHE-ZPkDpw4AdJ-MsV2InVY"
}
```

**Key Purposes**: `Assertion Method`, `Capability Invocation`.

**Service**:

```json
{
  "id": "service-1",
  "type": "TestService",
  "serviceEndpoint": ["https://test-service.com/1", "https://test-service.com/2"]
}
```

**Gateway**: `gateway1.example-did-dht-gateway.com`.

**Types**: `1`, `2`, `3`.

**DID Document**:

```json
{
  "id": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo",
  "controller": "did:example:abcd",
  "alsoKnownAs": ["did:example:efgh", "did:example:ijkl"],
  "verificationMethod": [
    {
      "id": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0",
      "type": "JsonWebKey",
      "controller": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo",
      "publicKeyJwk": {
        "kid": "0",
        "alg": "EdDSA",
        "crv": "Ed25519",
        "kty": "OKP",
        "x": "YCcHYL2sYNPDlKaALcEmll2HHyT968M4UWbr-9CFGWE"
      }
    },
    {
      "id": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#sig",
      "type": "JsonWebKey",
      "controller": "did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y",
      "publicKeyJwk": {
        "kid": "sig",
        "alg": "ES256K",
        "crv": "secp256k1",
        "kty": "EC",
        "x": "1_o0IKHGNamet8-3VYNUTiKlhVK-LilcKrhJSPHSNP0",
        "y": "qzU8qqh0wKB6JC_9HCu8pHE-ZPkDpw4AdJ-MsV2InVY"
      }
    }
  ],
  "authentication": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0"
  ],
  "assertionMethod": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0",
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0GkvkdCGu3DL7Mkv0W1DhTMCBT9-z0CkFqZoJQtw7vw"
  ],
  "capabilityInvocation": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0",
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0GkvkdCGu3DL7Mkv0W1DhTMCBT9-z0CkFqZoJQtw7vw"
  ],
  "capabilityDelegation": [
    "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#0"
  ],
  "service": [
    {
      "id": "did:dht:cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo#service-1",
      "type": "TestService",
      "serviceEndpoint": ["https://test-service.com/1", "https://test-service.com/2"]
    }
  ]
}
```

**DNS Resource Records**:

| Name       | Type | TTL  | Rdata                                                                                                                    |
| ---------- | ---- | ---- | ------------------------------------------------------------------------------------------------------------------------ |
| _did.cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo. | NS  | 7200 | gateway1.example-did-dht-gateway.com.                                     |
| _did.cyuoqaf7itop8ohww4yn5ojg13qaq83r9zihgqntc5i9zwrfdfoo. | TXT | 7200 | v=0;vm=k0,k1;auth=k0;asm=k0,k1;inv=k0,k1;del=k0;svc=s0                    |
| _cnt._did. | TXT  | 7200 | did:example:abcd                                                                                                         |
| _aka._did. | TXT  | 7200 | did:example:efgh,did:example:ijkl                                                                                        |
| _k0._did.  | TXT  | 7200 | t=0;k=YCcHYL2sYNPDlKaALcEmll2HHyT968M4UWbr-9CFGWE                                                                        |
| _k1._did.  | TXT  | 7200 | id=sig;t=1;k=Atf6NCChxjWpnrfPt1WDVE4ipYVSvi4pXCq4SUjx0jT9;c=did:dht:i9xkp8ddcbcg8jwq54ox699wuzxyifsqx4jru45zodqu453ksz6y |
| _s0._did.  | TXT  | 7200 | id=service-1;t=TestService;se=https://test-service.com/1,https://test-service.com/2                                      |
| _typ._did. | TXT  | 7200 | id=1,2,3                                                                                                                 |

#### Vector 3

A DID Document with two keys — the [[ref:Identity Key]] and an X25519 key used with a different `alg` value than
what is specified in the registry. The DID also has two gateway records and a service with an endpoint greater than
255 characters, and a previous record.

**Identity Public Key JWK**:

```json
{
  "kid": "0",
  "alg": "EdDSA",
  "crv": "Ed25519",
  "kty": "OKP",
  "x": "sTyTLYw-n1NI9X-84NaCuis1wZjAA8lku6f6Et5201g"
}
```

**X25519 Public Key JWK**:

```json
{
  "kid": "WVy5IWMa36AoyAXZDvPd5j9zxt2t-GjifDEV-DwgIdQ",
  "alg": "ECDH-ES+A128KW",
  "crv": "X25519",
  "kty": "OKP",
  "x": "3POE0_i2mGeZ2qiQCA3KcLfi1fZo0311CXFSIwt1nB4"
}
```

**Key Purposes**: `Key Agreement`.

**Service**:

```json
{
  "id": "service-1",
  "type": "TestLongService",
  "serviceEndpoint": ["https://test-lllllllllllllllllllllllllllllllllllooooooooooooooooooooonnnnnnnnnnnnnnnnnnngggggggggggggggggggggggggggggggggggggsssssssssssssssssssssssssseeeeeeeeeeeeeeeeeeerrrrrrrrrrrrrrrvvvvvvvvvvvvvvvvvvvviiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiccccccccccccccccccccccccccccccceeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee.com/1"]
}
```

**Gateways**: `gateway1.example-did-dht-gateway.com`, `gateway2.example-did-dht-gateway.com`.

**Previous DID**:
  - ID: `did:dht:x3heus3ke8fhgb5pbecday9wtbfynd6m19q4pm6gcf5j356qhjzo`.
  - Signature: `Tt9DRT6J32v7O2lzbfasW63_FfagiMHTHxtaEOD7p85zHE0r_EfiNleyL6BZGyB1P-oQ5p6_7KONaHAjr2K6Bw`.

**DID Document**:

```json
{
  "id": "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy",
  "verificationMethod": [
    {
      "id": "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#0",
      "type": "JsonWebKey",
      "controller": "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy",
      "publicKeyJwk": {
        "kid": "0",
        "alg": "EdDSA",
        "crv": "Ed25519",
        "kty": "OKP",
        "x": "sTyTLYw-n1NI9X-84NaCuis1wZjAA8lku6f6Et5201g"
      }
    },
    {
      "id": "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#WVy5IWMa36AoyAXZDvPd5j9zxt2t-GjifDEV-DwgIdQ",
      "type": "JsonWebKey",
      "controller": "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy",
      "publicKeyJwk": {
        "kid": "WVy5IWMa36AoyAXZDvPd5j9zxt2t-GjifDEV-DwgIdQ",
        "alg": "ECDH-ES+A128KW",
        "crv": "X25519",
        "kty": "OKP",
        "x": "3POE0_i2mGeZ2qiQCA3KcLfi1fZo0311CXFSIwt1nB4"
      }
    }
  ],
  "authentication": [
    "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#0"
  ],
  "assertionMethod": [
    "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#0"
  ],
  "keyAgreement": [
    "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#WVy5IWMa36AoyAXZDvPd5j9zxt2t-GjifDEV-DwgIdQ"
  ],
  "capabilityInvocation": [
    "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#0"
  ],
  "capabilityDelegation": [
    "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#0"
  ],
  "service": [
    {
      "id": "did:dht:sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy#service-1",
      "type": "TestLongService",
      "serviceEndpoint": ["https://test-lllllllllllllllllllllllllllllllllllooooooooooooooooooooonnnnnnnnnnnnnnnnnnngggggggggggggggggggggggggggggggggggggsssssssssssssssssssssssssseeeeeeeeeeeeeeeeeeerrrrrrrrrrrrrrrvvvvvvvvvvvvvvvvvvvviiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiccccccccccccccccccccccccccccccceeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee.com/1"]
    }
  ]
}
```

**DNS Resource Records**:

| Name       | Type | TTL  | Rdata       |
| ---------- | ---- | ---- | ----------- |
| _prv._did. | TXT  | 7200 | id=did:dht:x3heus3ke8fhgb5pbecday9wtbfynd6m19q4pm6gcf5j356qhjzo;s=Tt9DRT6J32v7O2lzbfasW63_FfagiMHTHxtaEOD7p85zHE0r_EfiNleyL6BZGyB1P-oQ5p6_7KONaHAjr2K6Bw |
| _did.sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy. | NS  | 7200 | gateway1.example-did-dht-gateway.com.                                        |
| _did.sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy. | NS  | 7200 | gateway2.example-did-dht-gateway.com.                                        |
| _did.sr6jgmcc84xig18ix66qbiwnzeiumocaaybh13f5w97bfzus4pcy. | TXT | 7200 | v=0;vm=k0,k1;auth=k0;asm=k0;agm=k1;inv=k0;del=k0;svc=s0                      |
| _k0._did.  | TXT  | 7200 | t=0;k=sTyTLYw-n1NI9X-84NaCuis1wZjAA8lku6f6Et5201g                                                                           |
| _k1._did.  | TXT  | 7200 | t=3;k=3POE0_i2mGeZ2qiQCA3KcLfi1fZo0311CXFSIwt1nB4;a=ECDH-ES+A128KW                                                          |
| _s0._did.  | TXT  | 7200 | id=service-1;t=TestLongService;se=https://test-lllllllllllllllllllllllllllllllllllooooooooooooooooooooonnnnnnnnnnnnnnnnnnngggggggggggggggggggggggggggggggggggggsssssssssssssssssssssssssseeeeeeeeeeeeeeeeeeerrrrrrrrrrrrrrrvvvvvvvvvvvvvvvvvvvviiiiiiiiiiiiiiii iiiiiiiiiiiiiiiccccccccccccccccccccccccccccccceeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee.com/1 |

### Open API Definition

```yaml
[[insert: api.yaml]]
```

## References

[[def:Ed25519]]
~ [Ed25519](https://ed25519.cr.yp.to/). D. J. Bernstein, N. Duif, T. Lange, P. Schwabe, B.-Y. Yang; 26 September 2011.
[ed25519.cr.yp.to](https://ed25519.cr.yp.to/).

[[def:BEP44]]
~ [BEP44](https://www.bittorrent.org/beps/bep_0044.html). Storing arbitrary data in the DHT. A. Norberg, S. Siloti;
19 December 2014. [Bittorrent.org](https://www.bittorrent.org/).

[[def:Bencode, Bencoded]]
~ [Bencode](https://wiki.theory.org/BitTorrentSpecification#Bencoding). A way to specify and organize data in a terse
format. [Bittorrent.org](https://www.bittorrent.org/).

[[def:z-base-32]]
~ [z-base-32](https://philzimmermann.com/docs/human-oriented-base-32-encoding.txt). Human-oriented base-32 encoding.
Z. O'Whielacronx; November 2002.

[[def:Pkarr]]
~ [Pkarr](https://pkarr.org). Public-Key Addressable Resource Records. Nuhvi.

[[def:VC-JOSE-COSE]]
~ [Securing Verifiable Credentials using JOSE and COSE](https://www.w3.org/TR/vc-jose-cose/). O. Steele, M. Jones,
M. Prorock, G. Cohen; 25 April 2024. [W3C](https://www.w3.org/).

[[spec]]
