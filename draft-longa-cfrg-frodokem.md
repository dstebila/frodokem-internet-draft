---
title: "FrodoKEM: key encapsulation from learning with errors"
abbrev: "FrodoKEM"
category: info

docname: draft-longa-cfrg-frodokem-latest
submissiontype: IRTF
number: 00
date: 2024-04-23
consensus: true
v: 3
area: IRTF
workgroup: CFRG
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

About This Document

   This note is to be removed before publishing as an RFC.

   The latest revision of this draft can be found at ...
   Status information for this document may be found at ...

   Source for this draft and an issue tracker can be found at ...

Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.

   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF).  Note that other groups may also distribute
   working documents as Internet-Drafts.  The list of current Internet-
   Drafts is at https://datatracker.ietf.org/drafts/current/.

   Internet-Drafts are draft documents valid for a maximum of six months
   and may be updated, replaced, or obsoleted by other documents at any
   time.  It is inappropriate to use Internet-Drafts as reference
   material or to cite them other than as "work in progress."

   This Internet-Draft will expire on ...

Copyright Notice

   Copyright (c) 2024 IETF Trust and the persons identified as the
   document authors.  All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal
   Provisions Relating to IETF Documents (https://trustee.ietf.org/
   license-info) in effect on the date of publication of this document.
   Please review these documents carefully, as they describe your rights
   and restrictions with respect to this document.  Code Components
   extracted from this document must include Revised BSD License text as
   described in Section 4.e of the Trust Legal Provisions and are
   provided without warranty as described in the Revised BSD License.

Table of Contents

   1.  Introduction  . . . . . . . . . . . . . . . . . . . . . . . .   3

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

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in
   BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

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
