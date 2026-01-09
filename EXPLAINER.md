# Decentralized Identifiers (DIDs) 1.1 – Explainer

## Discussion Venues

The Decentralized Identifiers (DIDs) 1.1 specification is developed openly by
the W3C Decentralized Identifier Working Group. Anyone interested can join the
conversation. The working group maintains a public GitHub repository where
issues and pull requests are used for discussion and development. There is also
an open mailing list for broader conversation: public-did-wg@w3.org, with
archives available online. As the spec itself notes, feedback is welcomed via
the GitHub issue tracker or the mailing list. (For more details on use cases and
requirements, see the W3C DID Use Cases and Requirements document.)

## User-Facing Problem

Today’s digital identity is often controlled by large centralized authorities.
For example, email addresses, social media accounts, and even government IDs
(passports, driver’s licenses) are issued by external organizations. These
identifiers are “not under our control” and depend on a central issuer. They may
work only in certain systems (e.g. your employer’s network or a particular
website) and can fail if the issuing body changes its rules or goes offline.
Moreover, many such identifiers unnecessarily reveal personal data when they are
used, and stolen or replicated copies can lead to fraud and identity theft. From
the user’s perspective, this means relying on others to manage our online
identities. If your email provider were hacked or your login service went down,
you could suddenly lose access to many accounts. Every time you “Sign in with X”
or hand over your driver’s license to a service, you trust a third party with
your identity and data. These centralized systems create single points of
failure and privacy risks. Users have little control over how their identifiers
are issued or revoked, and they often end up sharing more personal information
than necessary. The DID specification is motivated by solving exactly these
problems – giving end users control, continuity, and privacy for their
identifiers without a central gatekeeper.

## Proposed Approach

A Decentralized Identifier (DID) is essentially a new kind of identifier that a
user or entity fully controls.

DIDs have the form `did:<method>:<specific-id>`. For example, in the DID
did:example:123456, `did:` is the scheme, `example` is the method name, and
`123456` is the method-specific identifier. This syntax is standardized by the
DID spec.

There can be many different DID methods (each with its own rules) – for
instance, `did:ethr` might indicate a DID on the Ethereum blockchain, while
`did:web` might use DNS and web hosting. Each method specification defines how
DIDs using that method are created, updated, and resolved. Each DID resolves to
a corresponding DID Document.

A DID Document is a simple machine-readable JSON document that describes how to
interact with the entity identified by the DID. In practice, a DID Document
usually contains public cryptographic keys and optional service endpoints. These
public keys are the means by which the DID controller (the person or
organization controlling the DID) can prove ownership of the identifier. For
example, if Alice controls a DID, she holds the private keys matching the public
keys in the DID Document. When she wants to authenticate or sign something,
others check against the public key in her DID Document. In the spec’s words, a
DID Document is “a set of data describing the DID subject, including mechanisms,
such as cryptographic public keys, that the DID subject … can use to
authenticate itself and prove its association with the DID”. A DID Document may
also list services (e.g. URLs for messaging or data access) associated with that
identity. Resolvers and verifiable data registries complete the picture.

A DID resolver is software (or a service) that takes a DID as input and returns
its DID Document. How it does this depends on the method: for example, a
blockchain-based method might look up a transaction on a ledger, whereas a
web-based method might fetch data from a specific URL. These underlying storage
systems are often called verifiable data registries. The spec defines a
verifiable data registry as any system (like a distributed ledger or database)
that lets DIDs and DID Documents be created, updated, and deactivated.
Importantly, the DID Core spec does not mandate any particular technology for
this. It explicitly states that implementers can use blockchains, peer-to-peer
networks, centralized registries – whatever suits the application. The goal is
flexibility: almost any identity system can adopt DIDs by defining an
appropriate method.

Here is a very simple example of a DID Document for illustration (using a
made-up did:example method):

```json
{
  "@context": "https://www.w3.org/ns/did/v1.1",
  "id": "did:example:123456789abcdefghi",
  "authentication": [{
    "id": "did:example:123456789abcdefghi#keys-1",
    "type": "Multikey",
    "controller": "did:example:123456789abcdefghi",
    "publicKeyMultibase": "z6MkmM42vxfqZQsv4ehtTjFFxQ4sQKS2w6WR7emozFAn5cxu"
  }]
}
```

This document says that the DID `did:example:123456789abcdefghi` has a public
key (in multibase format) that can be used to authenticate its controller. A
user or system can retrieve this document (via the resolver for the example
method) and trust anything signed by that key. In practice, a DID Document can
be richer – it might include multiple keys (for key rotation or different
purposes) and service endpoints – but this shows the core idea.

## Practical Use Cases

DIDs have many practical uses for everyday people, devices, and services. For
instance, a person could use a DID to log into websites or apps. Instead of
creating a username and password for each service, Alice could present proof of
control over her DID when needed (for example, signing a challenge). Because the
public key was fetched from her DID Document, the website can verify her
identity without a password. If Alice wants extra privacy, she can use different
DIDs for different contexts (work vs. personal) and reveal only the information
she chooses.

In the Internet of Things, each device can have a DID and corresponding keys.
For example, a smart home thermostat might have `did:home:thermostat123`. Its
DID Document could list the thermostat’s public key for authentication, and even
a service endpoint where the device accepts configuration commands. When the
homeowner’s app wants to talk to the thermostat, it fetches
`did:home:thermostat123`, verifies the device’s signature, and then communicates
securely with the trusted endpoint. Because the device’s identity is not tied to
any central manufacturer’s login service, the homeowner retains control over it.
Websites and online services themselves can also use DIDs.

A company could publish a DID to represent a service or organization, embedding
in its DID Document a public key for API signing or a URL for service discovery.
Clients trusting that DID can then interact with the service without relying on
traditional SSL certificate authorities. In supply chains, products can be
assigned DIDs so that each item’s origin and ownership history are verifiable.

In each case, the DID acts as a portable identifier that can move across
systems: for example, you could take your identity DID from one social media
platform to another, or control your device’s DID even if you replace your home
router.

## Alternatives Considered

Before DIDs, there were other identity solutions, each with trade-offs.
Federated login systems (OAuth 2.0/OpenID Connect) let you sign in using a
Google, Facebook, or corporate account. While convenient, these still rely on a
central provider who can track or revoke your identity at will. They also
typically require revealing personal data (like your email or profile info) to
multiple sites. By contrast, DID does not assume any fixed authority: you can
choose which service (or none) to trust. As the DID spec puts it, DIDs are
“designed so that they may be decoupled from centralized registries, identity
providers, and certificate authorities”, whereas OAuth/OpenID services are
exactly centralized identity providers.

Another approach is using blockchain addresses or public key certificates
directly as identities. For example, your cryptocurrency wallet address is a
kind of identifier on one blockchain, but it is not easily portable or
human-readable across systems. Classic PKI (certificate authority) systems (like
SSL certificates) also provide verifiable identities, but they require trusting
CAs that issue and manage certificates. DIDs generalize these ideas: many DID
methods do use blockchains or other ledgers under the hood, but the DID
abstraction allows multiple methods.

In other words, a DID can be backed by Ethereum (did:ethr), by a private ledger,
by DNS (did:web), or even by your own custom database – all following the same
DID model. This flexibility is why the W3C spec emphasizes that DIDs should be
portable and “system- and network-independent”. In summary, unlike single-vendor
or single-technology schemes, DIDs offer a unified, extensible framework where
many different underlying systems can interoperate securely.

## Accessibility, Security, and Privacy Considerations

The DID specification has been developed with attention to inclusivity,
security, and privacy. Because DIDs and DID Documents are simply text-based
standards (JSON), they can be used with any technology that handles web
identifiers or JSON data. The spec enables DID Documents to be interpreted as
JSON-LD, which can help with internationalization (e.g. including language tags)
and use in different environments.

The W3C working group follows open processes, and the draft spec is publicly
available and will be translated and reviewed to ensure broad accessibility.
Security is built in by design. A key DID design goal is to “enable sufficient
security for requesting parties to depend on DID documents for their required
level of assurance”. In practice, this means the DID Controller must prove
possession of private keys to authenticate. For example, to prove control of a
DID, an entity typically produces a digital signature with the private key,
which others verify against the public key in the DID Document. This
cryptographic proof is stronger and more flexible than passwords or centralized
tokens. Because DIDs are decentralized, there is no single server whose
compromise would break the scheme; even if one registry is attacked, other
methods or registries can still operate independently. The spec also requires
DID methods to document their security requirements, such as how to resist
common attacks and how to handle key loss or recovery.

Privacy is another core concern. The DID architecture explicitly supports user
privacy. For instance, the spec’s goals include “minimal, selective, and
progressive disclosure” of information. A DID by itself does not reveal your
name or personal details – it’s just an opaque identifier tied to a public key.
You can also create multiple DIDs for different contexts, so your activities in
each context are unlinkable unless you choose otherwise. As the spec explains,
each entity can have “as many DIDs as necessary to maintain their desired
separation of identities, personas, and interactions”. Only the data you (or
your DID controller) choose to publish goes into a DID Document. Any additional
profile or attribute information can be kept separate or shared via specialized
credentials (e.g. verifiable claims) only with the parties you trust. In short,
DIDs give users control to share the minimum data needed: the public keys for
verification are public, but other personal data can remain private or managed
outside of the DID system.

Further Reading: See the official W3C DID Core spec (v1.1) for full details, and
the W3C Use Cases and Requirements document for more examples and motivations.

This document was generated using ChatGPT's "Research Mode" to read the W3C TAG's Explainer authoring guidelines, read the W3C DID v1.1 specification,
and then write an explainer that conforms to the TAG's guidelines. The entire
content was then lightly edited by an Editor of the DID v1.1 specification.
This is an experiment to see if we can replace hours of Editor labor with
a few minutes of LLM time.