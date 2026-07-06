---
title: "Applying Generate Random Extensions And Sustain Extensibility (GREASE) to EDHOC Extensibility"
docname: draft-ietf-lake-edhoc-grease-latest
ipr: trust200902
stand_alone: true
cat: info
wg: LAKE
author:
- ins: C. Amsüss
  name: Christian Amsüss
  country: Austria
  email: christian@amsuess.com
normative:
  RFC9528:
informative:
  RFC8701:
  RFC9170:


--- abstract

This document applies the extensibility mechanism GREASE (Generate Random Extensions And Sustain Extensibility),
which was pioneered for TLS,
to the ecosystem of Ephemeral Diffie-Hellman Over COSE (EDHOC).
It reserves a set of External Authorization Data (EAD) labels and unusable cipher suites
that may be included in messages
to ensure peers correctly handle unknown values.

--- middle

# Introduction {#introduction}

This document applies the extensibility mechanism GREASE (Generate Random Extensions And Sustain Extensibility),
which was pioneered for TLS in {{RFC8701}},
to Ephemeral Diffie-Hellman Over COSE (EDHOC, {{RFC9528}}) ecosystem.

The introduction of {{RFC8701}} and {{Section 3.3 of RFC9170}} provide comprehensive motivation for adding such extensions;
{{I-D.iab-protocol-greasing}} provides additional background that influenced this document.

The extension points of the EDHOC protocol are
cipher suites,
methods,
EAD (External Authorization Data) items
and COSE header parameters.
This document utilizes the cipher suite and EAD extension points.

Unlike in TLS GREASE {{RFC8701}},
EDHOC is operating on tight bandwidth and message size budget,
with some messages just barely fitting within relevant networks' fragmentation limits.
Thus,
more than with TLS GREASE,
it is up to implementations to decide
whether in their particular use case
they can afford to send additional data.

## Variability in other extension points

If the selected method is unsupported by the Responder,
EDHOC does not conclude successfully.
While values could be reserved for these for use as GREASE,
these failed attempts would not be verified between the EDHOC participants
without maintaining state between attempted EDHOC sessions.
Such an addition is considered impractical for constrained devices,
and thus out of scope for this document.

Recommendations for GREASE in {{Section 4 of ?I-D.iab-protocol-greasing}}
also include varying other aspects of the protocol,
such as varying sequences of elements.
EDHOC has little known variability,
and intentionally limits choice at times
(for example, {{Section 3.3.2 of RFC9528}} allows only the numeric identifier form where that is possible).
Where variation is allowed,
e.g., in padding or in the ordering of EAD items,
applications are encouraged to exercise it.

The extension point of COSE header parameters
(identifying other ID_CRED_x types)
is beyond the scope of this document,
and might be addressed orthogonally in the "COSE Header Parameters" registry.

## Terminology

{::boilerplate bcp14-tagged}

# The GREASE EAD labels

This document registers the following EAD labels for use with GREASE EAD items:

160, 41120, 43690, 44975

These EAD labels can be used in any EDHOC message
for non-critical EAD items (see {{Section 3.8 of RFC9528}}).

The numbers cover the different lengths of encoding available in CBOR for the registry's range (except the 23 precious small values).
It is expected that future documents register additional values with the same semantics.

## Use of GREASE EAD items by message senders

A sender of an EDHOC message MAY include an arbitrary number of GREASE EAD items,
with any or no ead_value (that is, with or without a byte string of any usable length).

Senders SHOULD consider the properties of the network their messages are sent over,
and refrain from adding GREASE when its use would be detrimental to the network
(for example, they might use it less frequently when the added size causes fragmentation of the message).

On networks where the data added by the grease EAD items does not significantly impact the network,
senders SHOULD irregularly send arbitrary (possibly random) GREASE EAD items with their messages
to ensure that errors resulting from the use of GREASE are detected.

The GREASE EAD items MAY be used as an alternative form of padding.

### Pattern for limited fingerprinting {#suggested-pattern}

A method of applying GREASE is suggested as follows:

* For every message, use GREASE with a random probability of 1 in 64.
* Pick a random GREASE label out of the uniform distribution of available options.
* Pick a random length from the uniformly distributed interval 9 to 40 (inclusive).
* Add the selected GREASE label with a value of the selected length,
  filled with random bytes.

Running EDHOC already requires the presence of a cryptographically secure random number generator.
Implementers can use that same source here to avoid any privacy implications from insufficiently initialized faster sources of randomness.

## Use of GREASE EAD items by message recipients

A party receiving a GREASE EAD item MUST NOT alter its behavior
in any way that would allow random GREASE EAD items
to alter the security context that gets established.

It MAY alter its behavior in other ways;
in particular, it SHOULD randomly insert GREASE EAD items
in later messages of a session in which unprocessed EAD items (including GREASE EAD items) were present.

Implementations SHOULD NOT attempt to recognize GREASE EAD items,
and SHOULD apply the default processing rules.

# GREASE cipher suites

This document registers the following GREASE cipher suites:

160, 41120, -41121, 43690

The numbers cover the different lengths of encoding available in CBOR for the registry's range (except the 46 precious small values), and both available signs.
It is expected that future documents register additional values with the same semantics.

An Initiator may insert a GREASE cipher suite
at any position in its sequence of preferred cipher suites.

A Responder MUST NOT support any of these GREASE cipher suites,
and MUST treat them like any other cipher suite it does not support.

Thus, a GREASE cipher suite never occurs as the selected cipher suite,
i.e., it is never specified as the last cipher suite in EDHOC message_1.
An Initiator whose offer of a GREASE cipher suite is accepted through cipher suite negotiation ({{Section 6.3 of RFC9528}})
needs to discontinue the protocol.

# Processing of GREASE-related failures

It is RECOMMENDED that any counters or statistics about successful and failed sessions
distinguish between sessions in which GREASE was applied and those in which it was not applied.
Any operator feedback channel, be it immediately to the user or through network monitoring,
SHOULD warn the operator if there are errors that were determined to originate from the use of GREASE
or that are significantly likely to originate from there.
This provides a feedback path as described in {{Section 4.4 of RFC9170}}.

On constrained devices, one suitable operator feedback channel is CORECONF {{?I-D.ietf-core-comi}};
no general YANG model is available for that purpose.

Whether logging of GREASE-related failed session details is appropriate
depends on the privacy policies of the application.

# Privacy considerations

The way in which GREASE is applied
can contribute to identifying which implementation of EDHOC
is being used.
Implementers of EDHOC are encouraged to use the algorithm described in {{suggested-pattern}},
both to reduce the likelihood of their implementation to be identified through the use of GREASE
and to increase the anonymity set of other users of the same algorithm.

# Security Considerations

The use of GREASE has no impact on security
in a correct EDHOC implementation.

As the application of GREASE contributes to an ecosystem that can have security updates deployed in the future,
implementers of EDHOC are strongly encouraged to apply GREASE regularly whenever their operational constraints permit it.

# IANA considerations

## EDHOC EAD labels

IANA is requested to register
four new entries into the EDHOC External Authorization Data Registry
established in {{RFC9528}}:

160, 41120, 43690, 44975

All share the name "GREASE",
the description "Arbitrary data to ensure extensibility",
and this document as a reference.

## EDHOC cipher suites

IANA is requested to register
four new values into the EDHOC Cipher Suites Registry
established in {{RFC9528}}:

160, 41120, -41121, 43690

All share
the array N/A,
the description "Unimplementable GREASE cipher suite to ensure extensibility",
and this document as a reference.

--- back

# Using extension points beyond successful EDHOC runs

Some ways of using the extension points, in particular
the use of critical GREASE EAD items
and
placing a GREASE cipher suite in the selected position
do not result in the successful continuation of the EDHOC session.

They can be useful during testing
(e.g., to verify that a peer does indeed implement the correct behavior of not silently tolerating critical EAD items it can not process),
particularly when they allow a testing system to provoke an error response from the implementation under test.
However,
this document is concerned with test performed during successful operation,
therefore that application is out of scope.

# Change log

Since draft-ietf-lake-edhoc-grease-01: Address WGLC comments.

* seccons: Stronly encourage use of GREASE
* Explicit SHOULD on applying default processing rules (not just by exclusion of SHOULD NOT attempt)
* Point to EDHOC CS-RNG for fingerprint resistance
* Point to CORECONF as example of how to report
* Add remark on why numeric values were chosen
* Elaboration on cipher suite selection
* Added BCP14 boilerplate Terminology section
* Updated reference from edm-protocol-greasing-02 to iab-protocol-greasing (currently at -01)
* Editorial fixes; highlights:
  - Consistency around "EAD items" and "EAD labels"
  - EAD items can be critical/non-critical, not labels

Since draft-ietf-lake-edhoc-grease-00: Resolve all open issues.

* Question on "is this better than padding" removed. (There are currently implementations of EDHOC that can't use all EAD values but can do padding).
* Question of COSE header extension deferred to COSE maintenance.
* Use of GREASE values in critical form is out of scope, but appendix illustrates that it can make sense to do, and emphasizes that indeed those options do cause errors when used with negative sign.

Since draft-amsuess-lake-edhoc-grease-01:

* Document was adopted in LAKE.
* Instead of discouraging GREASE around fragmentation limits wholesale, suggest reduced frequency.
* Editorial fix to fingerprinting section.

Since draft-amsuess-lake-edhoc-grease-00:

* Expanded introduction section to just point to the abstract any more.

Since draft-amsuess-core-edhoc-grease-01:

* Update references to RFC9528 🎉
* Change target WG to LAKE, renaming to draft-amsuess-lake-edhoc-grease
* Process RFC9170
  - Add a section on failure processing
  - Reference where appropriate
* Process draft-edm-protocol-greasing-02
  - Variability outside of extension points
  - Be firmer against recognizing GREASE values
  - Point out that future options may be registered (instead of the suggested algorithmic registrations)

Since -00:

* Fixed a mix-up between positivity and criticality of options.
* Adjusted numbers accordingly to once more fit in the `0xa.` pattern
  (actually they're using `0x.a`, but that doesn't work the same way with CBOR).
* Text improvements around recipient side processing.

# Acknowledgements
{:numbered="false"}

Marco Tiloca pointed out a critical error in the numeric constructions,
and provided many general improvements.
Göran Selander provided input to reduce mistakable text.
Meiling Chen and Shujaatali Badami found places where the text did not provide the right guidance to readers.
