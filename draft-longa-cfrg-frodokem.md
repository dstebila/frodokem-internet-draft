---
title: "FrodoKEM: key encapsulation from learning with errors"
abbrev: "FrodoKEM"
category: info

docname: draft-longa-cfrg-frodokem-latest
submissiontype: IRTF
date: 2024-04-23
v: 3
ipr: trust200902
area: IRTF
workgroup: CFRG
pi: [toc, sortrefs, symrefs]
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Your Name Here
    organization: Your Organization Here
    email: your.email@example.com

normative:

informative:


--- abstract

This memo specifies FrodoKEM, an IND-CCA2 secure Key Encapsulation Mechanism (KEM).

--- middle

# Introduction

FrodoKEM is a conservative yet practical post-quantum key encapsulation
mechanism whose security derives from cautious parameterizations of the
well-studied learning with errors problem, which in turn has close
connections to conjectured-hard problems on generic, "algebraically
unstructured" lattices.

As a key encapsulation mechanism, FrodoKEM is a three-tuple of
algorithms (_KeyGen_, _Encapsulate_, _Decapsulate_):

*  _KeyGen_ takes no inputs, requires randomness, and outputs a private
  key and a public key;

*  _Encapsulate_ takes as input a public key, requires randomness, and
  outputs a ciphertext and a shared secret;

*  _Decapsulate_ takes as input a ciphertext and a private key, and
  outputs a shared secret.

These algorithms are assembled as a two-pass protocol that allows two
parties to derive a shared secret.  This shared secret can then be used
to establish a secure communication channel using a symmetric-key
algorithm.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview

# FrodoKEM

## Key Generation

The key generation algorithm accepts no input, requires randomness, and
outputs the keypair (pk, sk) = (seedA || b, s || seedA || b || S^T || pkh).



# Parameter Sets

# Security Considerations

FrodoKEM-640, FrodoKEM-976 and FrodoKEM-1344 are designed to be
post-quantum IND-CCA2 secure KEMs at the security levels of AES-128,
AES-192 and AES-256, respectively.

Users are recommended to use the highest possible security level that
a given application allows.  In particular, the designers of FrodoKEM
recommend to use either FrodoKEM-976 or FrodoKEM-1344 for most
applications, and limit the use of FrodoKEM-640 to applications that
require short-term security.

Lattice-based cryptographic schemes such as FrodoKEM are still relatively
young.  Therefore, it is recommended to use FrodoKEM in combination with
a classical scheme (e.g., based on elliptic curves) while our confidence
in the security of lattice schemes increases over time.


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
