%%%
title = "More Instant Messaging Interoperability (MIMI) Identity Concepts"
abbrev = "MIMI Identity"
ipr= "trust200902"
area = "art"
workgroup = "mimi"
keyword = ["mimi","immi","identity"]
diagram = true

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mimi-identity-00"
stream = "IETF"

[[author]]
initials="R."
surname="Mahy"
fullname="Rohan Mahy"
organization = "Wire"
  [author.address]
  email = "rohan.mahy@wire.com"
%%%

.# Abstract

This document discusses concepts in instant messaging identity interoperability
when using end-to-end encryption, for example with the MLS 
(Message Layer Security) Protocol.

{mainmatter}

# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in RFC 2119 [@!RFC2219].

# Description

The MIMI (More Instant Messaging Interop) concept document [!@I-D.mahy-mimi-concept]
identifies areas where more work is needed to have interoperable IM systems.

The largest and most widely deployed Instant Messaging (IM) systems support
end-to-end message encryption using a variant of the Double 
Ratchet [] protocol popularized by Signal and the companion X3DH []
key agreement protocol. Many vendors are also keen to support the Message Layer Security
(MLS) protocol [@!I-D.ietf-mls-protocol] and architecture [@!I-D.ietf-mls-architecture].
These protocols provide confidentiality
of sessions (with Double Ratchet) and groups (with MLS) once the participants in 
a conversation have been identified. However, the current state of most systems
require the end user to manually verify key fingerprints or blindly trust their
instant messaging service not to add and remove participants from their 
conversations. This problem is exacerbated when these systems federate and try to
interoperate.

While some single vendor solutions exist, clearly an interoperable mechanism
for IM identity is needed. 
First this document attempts to articulate a clear description and semantics
of different identifiers used in IM systems. Next the document provides an
example of how to represent those identifiers in a common way. Then the document
discusses different trust approaches. Finally the document surveys various
cryptographic methods of making and verifying assertions about these
identifiers. 

Arguably, as with email, the success of XMPP [@!RFC6120] was partially due to 
the ease of communicating among XMPP users in different domains with 
different XMPP servers, and a single
standardized address format for all XMPP users.


# Types of Identifiers

IM systems have a number of types of identifiers. Few (or perhaps no) systems use
every type of identifier described here. Not every configuration of the same 
application necessarily use the same list of identifiers.

Domain identifier:
: A bare domain name is often used for discovery of a specific IM service such as
`example.com` or `im.example.com`. Many proprietary IM systems operate in a single
domain and have no concept of domains or federation.

Handle identifier:
: A handle is an identifier which represents a user or service. A handle is usually
intended for external sharing (for example it could appear on or in a paper or
electronic business card). 
IM systems could have handles which are unscoped (don't contain a domain)
or scoped (contain a domain).
Unscoped handles are often prefixed with a commercial at-sign ("@").
Handles in some services are mutable. For example, `@alice_smith` could
become `@alice_jones` or `@alex_smith` after change of marital status or
gender transition.

| Protocol        | Identifier Address      | Example                  |
| --------------- | ----------------------- | ------------------------ |
| Jabber/XMPP     | Bare JID                | `juliet@example.com`     |
| SIP             | Address of Record (AOR) | `sip:juliet@example.com` |
| IRC             | nick                    | `@juliet`                |
| Generic example | "unscoped handle"       | `@juliet`                |
| Generic example | "scoped handle"         | `@juliet@example.com`    |
| Email style     | Mailbox address         | `juliet@example.com`     |
Table: some Handle identifier styles

User or account identifier:
: Many systems have an internal representation of a user, service, or account separate
from the handle. This is especially useful when the handle is allowed to change.
Unlike the handle, this identifier typically cannot change.  For example the user 
identifier could be a UUID or a similar construction. In IRC, a user identifier is
prefixed with a "!" character (example: `!jcapulet1583@example.com`).

Client or Device identifier:
: Most commercial instant messaging systems allow a single user to have multiple
devices at the same time, for example a desktop computer and a phone. Usually, each
client instance of the user is represented with a separate identifier with separate
keys. Typically these identifiers are internal and not visible to the end-user (XMPP
fully qualified JIDs are a rare exception). The client or device identifier is often
based on a UUID, a persistent long-term unique identifier like an IMEI or MAC address,
a sequence number assigned by the IM service domain, or a combination. In some cases
the identifier may contain the user identifier.

| Protocol    | Identifier Address  | Example                                      |
| ----------- | ------------------- | -------------------------------------------- |
| Jabber/XMPP | Fully-qualified JID | `juliet/balcony@example.com`                 |
| SIP         | Contact Address     | `sip:juliet@[2001:db8::0225:96ff:fe12:3456]` |
| Wire        | Qualified client ID | `BFuVxW5BfqaMEfJDc8R7Qw:072b@example.com`    |
Table: some Client/Device identifier styles.

Group Chat or Channel identifier (external):
: All or nearly all instant messaging systems have the concept of named groups
or channels which support more than 2 members and whose membership can change over time.
Many IM systems support an external identifier for these groups and allows them to
be addressed. In IRC and many other systems, they are identified with a "#"
(hash-mark) prefix. The proliferation of hashtags on social media makes this
convention less common on newer systems.
 
Group, Conversation, or Session identifiers (internal):
: Most IM protocols use an internal representation for a group or 1:1 chat. 
In MLS this is called the `group_id`. The Wire protocol uses the term
`qualified conversation ID` to refer to a group internally across domains. 
Among implementations of the Double Ratchet family of protocols a unidirectional
sequence of messages from one client to another is referred to as a session, and
often has an associated session identifier. 

Team or Workspace identifier:
: A less common type of identifier among IM systems is used to describe a set of
users or accounts. This is described variously as a team, workspace, or tenant.



One user may have multiple clients (for example a mobile and a desktop client).
Rarely, a handle may refer to a single user or it may redirect to multiple users.
In some systems, the user identifier is a handle. In other systems the user
identifier is an internal representation, for example a UUID. Handles may be
changed/renamed, but hopefully internal user identifiers do not.
Unscoped handles are often prefixed with a commercial at-sign ("@").

Likewise, group conversation identifiers could be internal or external
representations, whereas group names or channel names are often external
friendly representations.  Unscoped channel names are often prefixed
with a hash character ("#"). Some systems have an additional level of hierarchy
with a team identifier under which groups/channels can be organized and
authorized.

This proposal relies on URIs for naming and identifiers. All the example use
the `im:` URI scheme (defined in [@!RFC3862]), but any instant messaging scheme
is acceptable.

It is easy to imagine a loose hierarchy between these identifiers
domain to user to device, but hard to agree on a rigid structure. 
In some systems, the group chat or session itself has
a position in the hierarchy underneath the domain, the user, or the device.


# Representation of identifiers using URIs

Most if not all of the identifiers described in the previous section could be
represented as URIs. While individual instant messaging protocol-specific URI
schemes may not have been specified with this use of URIs in mind, the `im:`
URI scheme should be flexible enough to represent all of or any needed subset of the
previously discussed identifiers.

XMPP can represent a domain, a handle (bare JID), or a device (fully qualified JID). 
Unfortunately its xmpp: URI scheme was only designed to represent handles and domains.
But the `im:` URI scheme can represent all XMPP identifiers:

* im:xmpp=example.com  (domain only)
* im:xmpp=juliet@example.com  (bare JID - handle)
* im:xmpp=juliet/balcony@example.com  (fully qualified JID - client/device)

Likewise the IRC protocol can represent domain, handle (nick), user (account),
and channel.

* im:irc=
* im:irc=
* im:irc=
* im:irc=

Imagine a hypothetical WXYZ IM protocol with support for all our identifiers.
These could be represented unambiguously using the conventions below:


| id type  | unscoped form               | domain scoped form                      |
| -------- | --------------------------- | --------------------------------------- |
| domain   |              -              | example.com                             |
| handle   | @alice                      | @alice@example.com                      |
| user     | BFuVxW5BfqaMEfJDc8R7Qw      | BFuVxW5BfqaMEfJDc8R7Qw@example.com      |
| device   | BFuVxW5BfqaMEfJDc8R7Qw/072b | BFuVxW5BfqaMEfJDc8R7Qw/072b@example.com |
| channel  | #projectX                   | #projectX@example.com                   |
| team     | ##engineering               | ##engineering@example.com               |
| channel  | ##engineering/projectX      | ##engineering/projectX@example.com      |
| group id | $TII9t5viBrXiXc             | $TII9t5viBrXiXc@example.com             |
Table: examples of all identifier types in the WXYZ IM protocol


Now imagine that WXYZ reserved the wxyz: URI scheme. The example below shows how
most reasonable protocol-specific identifier schemes can be represented as an `im:`
URI.

```
wxyz:example.com
wxyz:%40alice@example.com
wxyz:BFuVxW5BfqaMEfJDc8R7Qw/072b@example.com
wxyz:#projectX@example.com
wxyz:##engineering@example.com
wxyz:$TII9t5viBrXiX@example.com

im:wxyz=example.com
im:wxyz=%40alice@example.com
im:wxyz=BFuVxW5BfqaMEfJDc8R7Qw/072b@example.com
im:wxyz=#projectX@example.com
im:wxyz=##engineering@example.com
im:wxyz=$TII9t5viBrXiX@example.com
```

Give example IRC URIs here.

If there is no domain, an `im:` URI can use `local.invalid` in place of a resolvable
domain name.

```
im:wxyz=%40alice@local.invalid
```

# Different Root of Trust Approaches

Different IM applications and different users of these applications may have
different trust needs. The following subsections describe three specific trust
models for example purposes. Note that the descriptions in this section use certificates
in their examples, but nothing in this section should preclude using a different
technology which provides similar assertions.

## Centralized credential hierarchy

In this environment, end-user devices trust a centralized authority operating on
behalf of their domain (for example, a Certificate Authority), that is trusted by
all the other clients in that domain (and can be trusted by federated domains). The
centralized authority could easily be associated with a traditional Identity
Provider (IdP). This is a popular trust model for companies running services for
their own employees and contractors. This is also popular with governments providing
services to their employees and contractors or to residents or citizens for whom
they provide services.

For example XYZ Corporation could make an assertion that "I represent XYZ
Corporation and this user demonstrated she is Alice Smith of the Engineering
department of XYZ Corporation."

In this model, a Certificate Authority (CA) run by or on behalf of the domain generates
certificates for one or more of the identifier types described previously. The
specifics of the assertions are very important for interoperability. Even within
this centralized credential hierarchy model, there are at least three ways to make
assertions about different types of IM identifiers with certificates:


Example 1 (Separate Certs):
: The CA generates one certificate for a user Alice which is used to sign Alice's profile.
The CA also generates a separate certificate for Alice's desktop client and a third
for her phone client. The client certificates are used to sign MLS KeyPackages or
Double Ratchet-style prekeys. 

Example 2 (Single Combined Cert):
: The CA generates a single certificate per client which covers both Alice's handle and
her client identifier in the same certificate. Each of these certificates is used to
sign MLS KeyPackages or Double Ratchet-style prekeys. 

Example 3 (Cascading Certs):
: The CA generates a single user certificate for Alice's handle and indicates that the
user certificate can issue its own certificates.
The user certificate then generates one certificate for Alice's desktop client and
another certificate for Alice's phone client.
The client certificates are used to sign MLS KeyPackages or
Double Ratchet-style prekeys. 

validatring these certificates wuold require...  ****

The subjectAltName is a URI type 


Regardless of the specific implementation, this model features a strong hierarchy.



What is important in all these examples is that other clients involved in a session or
group chat can validate the relevant credentials of the other participants in the
session of group chat.
 
 
The disadvantage of this approach is that if the CA colludes with an attacker or
is compromised, the attacker can easily insert a rogue client which would be as
trusted as a legitimate client.


## Web of Trust 

In some communities, it may be appropriate to make assertions about IM
identity by relying on a web of trust. The following specific example of this general
method is used by the OMEMO community and proposed in []. 

It uses a web of trust with cross signing as follows:

Each user has a master key.
Alice's master key signs exactly two keys: Alice's device-signing key (which then signs
her own device keys), and Alice's user-signing key (which can sign the master key of
other users).

The advantage of this approach is that if Alice's and Bob's keys, implementations,
and devices are not compromised
there is no way the infrastructure can forge a key for Alice or Bob and insert
an eavesdropper or active attacker. 
The disadvantages of this approach are that this requires Alice's 
device-singing key to be available any time
Alice wants to add a new device, and Alice's user-signing key to be available
anytime she wants to add a new user to her web of trust. This could either make
those operations inconvenient and/or unnecessarily expose either or both of those
keys.

~~~ ascii-art
          Alice          :          Bob
        +--------+       :       +--------+
        | master |<---\  /------>| master |
        +--------+     \/:       +--------+
         /    \       / \___      /     \
        /      \     /   :  \    /       \
+---------+  +---------+  +---------+  +---------+
| device  |  |  user   | :|  user   |  | device  |
| signing |  | signing | :| signing |  | signing |
+---------+  +---------+ :+---------+  +---------+
   /     \               :              /     \
+----+  +----+           :          +----+  +----+
| A1 |  | A2 |           :          | B1 |  | B2 |
+----+  +----+           :          +----+  +----+
~~~
Figure: Alice and Bob cross sign each other's master keys


## Well-known service cross signing

In this trust model, a user with several services places a cross signature for all
their services at a well known location on each of those services (for example a
personal web site home page, an IM profile, the profile page on an open source code
repository, a social media About page, a picture sharing service profile page, 
a professional interpersonal networking site, and a dating application profile).
This concept was perhaps first implemented for non-technical users by Keybase.
The user of this scheme likely expects that at any given moment
there is a risk that one of these services is compromised or controlled by a
malicious entity, but expects the likelihood of all or most of their services being
compromised simultaneously is very low.

The advantage of this approach is that it does not rely on anyone but the user
herself. This disadvantage is that if an attacker is able to delete or forge cross
signatures on a substantial number of the services, the forged assertions would looks as
legitimate as the authentic assertions (or more convincing).

## Combining approaches

These different trust approaches could be combined, however the verification rules
become more complicated. Among other problems, implementers need to decide what happens
if two different trust methods come to incompatible conclusions. For example, what
should the application do if web of trust certificates indicate that a client or
user should be trusted, but a centralized hierarchy indicates a client should not be,
or vice versa.

# Cryptographic mechanisms to make assertions about IM identifiers

## X.509 Certificates

X.509 certificates are a mature technology for making assertions about identifiers.
The supported assertions and identifier formats used in certificates are 
somewhat archaic, inflexible, and pedantic, but well understood. The semantics
are always that an Issuer asserts that a Subject has control of a specific 
public key key pair. 
The Subject field
contains a Distinguished Name, whose Common Name (CN) field can contain free form text.
The subjectAltName can contain multiple other identifiers for the Subject
with types such as a URI, email address, DNS domain names, or
Distinguished Name. The rules about which combinations of extensions are valid
are defined in the Internet certificate profile described in [@!RFC5280]. As noted
in a previous section of this document, URIs are a natural container for holding
instant messaging identifiers. Implementations need to be careful to insure that the
correct semantics are applied to a URI, as they may be referring to different
objects (ex: a handle versus a client identifier). There is a corresponding 
issuerAltName field as well.
 
Certificates are already supported in MLS as a standard credential type which can
be included in MLS LeafNodes and KeyPackages.
[In the X3DH key agreement protocol (used with Double Ratchet), the first message
in a session between a pair of clients can contain an optional
certificate, but this is not standardized.]
Arguably the biggest drawback to using X.509 certificates is that administratively
it can be difficult to obtain certificates for entities that can also generate
certificates---specifically to issue a certificate with the standard extension
`basicContraints=CA:TRUE`.]

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            04:dc:7a:4b:89:22:98:32:35:1f:91:84:f7:e9:4e:5d:24:c4
        Signature Algorithm: ED25519
        Issuer: O = example.com, CN = acme.example.com
        Validity
            Not Before: Jul  6 06:41:50 2022 GMT
            Not After : Oct  4 06:41:49 2022 GMT
        Subject: O = example.com, CN = Alice M. Smith
        Subject Public Key Info:
            Public Key Algorithm: ED25519
                ED25519 Public-Key:
                pub:
                    a0:6b:14:1e:a8:04:2a:09:6b:62:89:48:7c:da:5c:
                    68:73:b9:2a:8e:65:50:f9:15:70:bd:91:d7:86:52:
                    1e:4f
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Agreement
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier: 
                4C:EA:12:32:79:03:F6:4F:47:29:37:5F:96:BB:E1:91:5E:FC
            X509v3 Authority Key Identifier: 
                14:2E:B3:17:B7:58:56:CB:AE:50:09:40:E6:1F:AF:9D:8B:14
            Authority Information Access: 
                OCSP - URI:http://oscp.acme.example.com
                CA Issuers - URI:http://acme.example.com/
            X509v3 Subject Alternative Name: critical
                URI:im:SvPfLlwBQi-6oddVRrkqpw/04c7@example.com, 
                URI:im:%40alice.smith@example.com
            X509v3 Certificate Policies: 
                [etc....]
                
    Signature Algorithm: ED25519
    Signature Value:
        da:21:49:cc:7a:ac:ed:7b:27:59:30:81:d9:94:c0:d7:86:e7:
        db:b2:c9:ed:72:47:19:01:aa:2a:7f:24:d6:ce:2f:4f:9d:fe:
        ab:8b:e2:0e:43:1b:62:b1:1d:12:3f:78:a2:bf:cc:7b:52:ef:
        df:c1:94:5a:3f:ca:a1:f6:88:02
```
Figure: mocked up IM client certificate with both client id and handle

If implementing cascading certificates, the Issuer might be a expressed as a URI in the
issuerAltName extension.

```
TBC
```
Figure: mocked up IM client certificate issued by the domain for the handle URI as
Subject. Then another certificate issued by the handle URI for the device URI as its
Subject.


## JSON Web Tokens (JWT) with Distributed Proof of Presence (DPoP)

JSON Web Signing (JWS) [@!RFC7515] and JSON Web Tokens (JWT) [@!RFC7519] are toolkits for 
making a variety of cryptographic claims. (CBOR Web tokens [@!RFC8392] are semantically
equivalent.)
JWT is an appealing option for carrying IM identifiers and assertions, as the
container type if flexible and the format is easy to implement. Unfortunately the
semantics are poorly specified at the time of this writing.  

The JWT Distributed Proof of Possession draft [@!I-D.ietf-oauth-dpop] adds the ability
to share claims about 


Note that this could be combined with nested JWT claims ******


Tokens need online verification with issuing IdP

```
{
    "typ": "dpop+jwt",
    "alg": "EdDSA",
    "jwk": {
         "typ": "OKP",
         "crv": "Ed25519",
         "x": "9kaYCj...3lnwW"
    }
}
.
{
    "jti": "7535d380-673e-4219-8410-b8df679c306e",
    "iat": 1653455836315,
    "htm": "POST",
    "htu": "https://example.com/client/token",
    "nonce": "WE88EvOBzbqGerznM-2P_AadVf7374y0cH19sDSZA2A",
    "sub": "im:SvPfLlwBQi-6oddVRrkqpw/04c7@example.com",
    "exp": 1661231836315
}
```
Figure: JOSE header and claims sections of a JWT DPoP proof referring to an IM URI


## Verifiable Credentials

Verifiable Credentials (VC) is a framework for exchanging machine-readable 
credentials [@!W3C.REC-vc-data-model-20191119]. The framework is well 
specified and has a very flexbile assertion structure, which 
in addition to or in place of basic names and identifiers, can
optionally include arbitrary attributes (ex: security clearance, age, nationality)
up to an including Zero Knowledge Proofs depending on the profile being used. 
For example, a verifiable credential could be used to assert that an IM client
belongs to a Customer Support agent of Sirius Cybernetic Corp, who speaks
English and Vogon, and is qualified to give support for their Ident-I-Eeze product,
without revealing the name of the agent.

The VC specification describes both Verifiable Credentials and Verifiable Presentations.
A Verifiable Credential contains assertions made by an issuer. Holders assemble 
credentials into a Verifiable Presentation. Verifiers can validate the Verifiable
Credentials in the Verifiable Presentation. Specific credential types are defined by
referencing ontologies. The example at the end of this section uses the
VCard ontology [@!W3C.WD-vcard-rdf-20130924].

Most of the examples for Verifiable Credentials use Decentralized Identifiers (DIDs),
but there is no requirement to use DID or the associated esoteric cryptography 
with VC.  The most significant problems are that
there is no off-the-shelf mechanism for proof of possession of a private key, and no
consensus to use for straightforward identity assertions.


```
{
  "sub": "im:SvPfLlwBQi-6oddVRrkqpw/04c7@example.com",
  "jti": "http://im.example.com/@alice_smith/devices/04c7",
  "iss": "https://im.example.com/keys/issuer.jwk",
  "nbf": 1653455836315,
  "iat": 1653455836315,
  "exp": 1661231836315,
  "nonce": "WE88EvOBzbqGerznM-2P_AadVf7374y0cH19sDSZA2A",
  "vc": {
    "@context": [
      "https://www.w3.org/2018/credentials/v1",
      "http://www.w3.org/2006/vcard/ns"
    ],
    "type": ["VerifiableCredential", "ImDeviceCredential"],
    "credentialSubject": {
      "fn": "Alice M. Smith",
      "hasOrganizationName": "Example Corp",
      "hasOrganizationalUnit": "Engineering",
      "hasInstantMessage": "im:%40alice_smith@example.com",
      "hasInstantMessage": "im:SvPfLlwBQi-6oddVRrkqpw/04c7@example.com"
    }
  }
}
```
Figure: fragment of an example claims payload of JWT-based Verifiable Credential
proof referencing the VCard ontology


## Other possible mechanisms 
Below are other mechanisms which were not investigated due to a lack of time.

- Anonymous credential schemes which can present attributes without the 
long-term identity (ex: travel agent for specific team)
- Zero-knowledge proofs
- Deniable credentials

# Discovery of Keying Material

- well-known URL with query format on each domain
	- search string
		- could be handle, internal user ID, internal device ID; search by anonymous credential?
		- what type of key are you searching for?
			- keypackages
			- user keys
			- domain keys
	- searcher identity and proof of identity (optional)
	- rate limiting


* General issues of interoperable end-to-end security
Signal introduced what is now know as the Double Ratchet protocol in 20xx. Today there
are over a dozen implementations of variations of Double Ratchet. While the differences
among these variations tend to be small, the is little emphasis on interoperability.


## Profiles of security protocols to enable interoperable end-to-end encryption

Enabling strong user privacy has been a core concern of the IETF for decades, and was
the main motivation for the CPIM message format. S/MIME and PGP where proposed for
use with instant messaging systems, but never widely adopted. The first broad adoption
of end-to-end encryption in messaging was with Off The Record (OTR) [OTR] introduced
in 2004, which also included perfect forward secrecy (which protects past
communications from future compromises). As OTR was available in XMPP clients, it
was possible to use across domains.

### Protocols based on Double Ratchet

Tens of instant messaging applications implement some form of end-to-end encryption
using a protocol based on the Double Ratchet protocol. Double Ratchet was originally
referred to as Axolotl Ratchet when it was introduced in 2013 and popularized in the
Signal application.  Most applications using Double Ratchet also use [X3DH] for initial
key agreement.  However the initial setup of encryption sessions among these applications
are often incompatible. 

Double Ratchet uses a fixed ciphersuite and has no content negotiation mechanism

### Instant Messaging using Messaging Layer Security

Messaging Layer Security (MLS) is a 

 Use of MLS in the Instant Messaging Context: (ex: long-lived persistent groups)



* Content negotiation

* Content format interoperability


* Administrative setup of federation: (ex: agreement on certificates, contact information, abuse policies)

# IANA Considerations

This document requires no action by IANA.

# Security Considerations

The threat model for interoperable IM systems depends on many subtle details



TBC

[OTR] https://otr.cypherpunks.ca/otr-wpes.pdf
[DoubleRatchet] https://signal.org/docs/specifications/doubleratchet/
[X3DH] https://signal.org/docs/specifications/x3dh/

Cryptographic Identity: Conquering the Fingerprint Chaos
Apr 6, 2021  Paul Schaub (vanitasvitae) gives a talk for the XMPP Office Hours about OMEMO keys and a better way to manage lots of devices and identities.
https://www.youtube.com/watch?v=oc5844dyrsc

Cross-signing devices with device signing keys
Dec 13, 2018
https://github.com/matrix-org/matrix-doc/blob/master/proposals/1756-cross-signing.md


{backmatter}

<reference anchor="I-D.mahy-dispatch-immi-mls-mime">
   <front>
      <title>Inside MLS Message Interop (IMMI) MIME type extensions</title>
      <author fullname="Rohan Mahy">
	 <organization>Wire</organization>
      </author>
      <date month="March" day="7" year="2022" />
   </front>
   <seriesInfo name="Internet-Draft" value="draft-mahy-dispatch-immi-mls-mime-00" />
   <format type="TXT" target="https://www.ietf.org/archive/id/draft-mahy-dispatch-immi-mls-mime-00.txt" />
</reference>


