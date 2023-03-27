---
title: "Applying Generate Random Extensions And Sustain Extensibility (GREASE) to EDHOC Extensibility"
docname: draft-amsuess-core-edhoc-grease-latest
ipr: trust200902
stand_alone: true
cat: info
wg: CoRE
author:
- ins: C. Amsüss
  name: Christian Amsüss
  country: Austria
  email: christian@amsuess.com
normative:
  I-D.ietf-lake-edhoc:
informative:
  RFC8701:


--- abstract

This document applies the extensibility mechanism GREASE (Generate Random Extensions And Sustain Extensibility),
which was pioneered for TLS,
to the EDHOC ecosystem.
It reserves a set of non-critical EAD labels and unusable cipher suites
that may be included in messages
to ensure peers correctly handle unknown values.

--- middle

# Introduction {#introduction}

\[ See abstract \]

The introduction of {{RFC8701}} provides comprehensive motivation for adding such extensions.

The extension points of EDHOC are
cipher suites,
methods,
EADs (External Authorization Data items)
and COSE headers.
Of these,
EADs and cipher suites
can be used in such a way that even in the presence of an unknown value,
a connection can still be established.

# The GREASE EAD labels

This document registers the following EAD labels as GREASE EADs:

161, 41121, 43691, 44976

These EADs are available in all EDHOC messages.
The EADs are only used in their negative (non-critical) form.

## Use of GREASE EADs by message senders

A sender of an EDHOC message MAY send a GREASE EAD using the non-critical (negative) form at any time,
with any or no EAD value (that is, with or without a byte string of any usable length),
in any message.

Senders SHOULD consider the properties of the network their messages are sent over,
and refrain from adding GREASE when its use would be detrimental to the network
(for example, when the added size causes fragmentation of the message).

On networks where the data added by the grease EADs does not significantly impact the network,
senders SHOULD irregularly send arbitrary (possibly random) GREASE EADs with their messages
to ensure that errors resulting from the use of GREASE are detected.

The GREASE messages MAY be used as an alternative form of padding.

### Pattern for limited fingerprinting {#suggested-pattern}

A method of deciding how to apply GREASE is suggested as follows:

* For every message, use GREASE with a random probability of 1 in 64.
* Pick a random GREASE label out of the uniform distribution of available options.
* Pick a random length from the uniformly distributed interval 9 to 40 (inclusive).
* Add the selected GREASE label with a value of the selected length,
  filled with random bytes.

## Use of GREASE EADs by message recipients

A party receiving a GREASE EAD MUST NOT alter its behavior
in any way that would allow random GREASE EADs
to alter the security context that gets established.

It MAY alter its behavior in other ways;
in particular, it is SHOULD randomly insert GREASE EADs
in later messages of an exchange in which any were received.

As the EADs are only used in non-critical form,
the behavior of a recipient that is unaware of the GREASE options
is to ignore them.
This satisfies the requirements on GREASE processing.

# GREASE cipher suites

This document registers the following cipher suites:

160, 41120, -41121, 43690

An initiator may insert a GREASE cipher suite
at any position in its sequence of preferred cipher suites.

A responder MUST NOT support any of these cipher suites,
and MUST treat them like any other cipher suite it does not support.

An initiator whose choice of a GREASE cipher suite is accepted
MUST discontinue the protocol.

# Privacy considerations

The way in which GREASE is applied
can contribute to identifying which implementation of EDHOC
is being used.
Implementers of EDHOC are encouraged to use the algorithm described in {{suggested-pattern}},
both to reduce the likelihood of their implementation to be identified through the use of GREASE
and to increase the anonymity set of other users of the same algorithm.

# Security Considerations

The use of the GREASE option has no impact on security
in a correct EDHOC implementation.

# IANA considerations

## EDHOC EADs

IANA is requested to register
four new entries into the EDHOC External Authorization Data Registry
established in {{draft-ietf-lake-edhoc-19}}:

161, 41121, 43691, 44976

All share the name "GREASE",
the description "Arbitrary data to ensure extensibility",
and this document as a reference.

## EDHOC cipher suites

IANA is requested to register
four new values into the EDHOC Cipher Suites Registry
established in {{draft-ietf-lake-edhoc-19}}:

160, 41120, -41121, 43690

All share the name "GREASE",
the array N/A,
the description "Unimplementable cipher suite to ensure extensibility",
and this document as a reference.

--- back

# Open questions

Do the GREASE EADs add any value that padding does not already add?

Probably yes, because padding is "special enough" that it could be handled in a hard-coded fashion.
(Then again, there's nothing but the effort stopping anyone else from doing the same with the GREASE EADs, right?)
