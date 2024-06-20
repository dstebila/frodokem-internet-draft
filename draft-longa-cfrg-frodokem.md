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
outputs the keypair (pk, sk) = (seedA \|\| b, s \|\| seedA \|\| b \|\| S^T \|\| pkh).

1. Choose uniformly random seeds s, seedSE and z of bitlengths lensec, lenSE and lenA (resp.)

2. Generate pseudorandom seed seedA = SHAKE256(z, lenA)

3. Generate the matrix A = Gen(seedA)

4. Generate pseudorandom bit string (r^(0), r^(1), ..., r^(2\*n\*nHat - 1)) = SHAKE256(0x5F \|\| seedSE, 32\*n\*nHat)

5. Sample error matrix S^T = SampleMatrix((r^(0), r^(1), ..., r^(n\*nHat − 1)), nHat, n)

6. Sample error matrix E = SampleMatrix((r^(n\*nHat), r^(n\*nHat + 1), ..., r^(2\*n\*nHat − 1)), n, nHat)

7. Compute matrix B = A\*S + E

8. Compute b = Pack(B)

9. Compute pkh = SHAKE256(seedA \|\| b, lensec)

10. Return public key pk = (seedA \|\| b) and secret key sk = (s \|\| seedA \|\| b \|\| S^T \|\| pkh).
ST = S^T is encoded row-by-row from ST_(0,0) to ST_(nHat−1,n−1), where
each matrix coefficient ST_(i,j) is a signed integer encoded as a 16-bit string
in the little-endian byte order such that (s_0, s_1, ..., s_15) = ST_(i,j) =
−s_15 * 2^15 + (s_0 + s_1 * 2 + s_2 * 2^2 + ... + s_14 * 2^14)

## Encapsulation

The encapsulation algorithm takes as input a public key pk = (seedA \|\| b), and
outputs a ciphertext c = (c1 \|\| c2 \|\| salt) and a shared secret ss.

1.  Choose uniformly random values u and salt of bitlengths lensec and lensalt (resp.)

2.  Compute pkh = SHAKE256(pk, lensec)

3.  Generate pseudorandom values seedSE \|\| k = SHAKE256(pkh \|\| u \|\| salt, lenSE+lensec)

4.  Generate pseudorandom bit string (r^(0), r^(1), ..., r^(2\*nHat\*n + nHat^2 - 1)) = SHAKE256(0x96 \|\| seedSE, 16(2\*nHat\*n + nHat^2))

5.  Sample error matrix S' = SampleMatrix((r^(0), r^(1), ..., r^(nHat\*n - 1)), nHat, n)

6.  Sample error matrix E' = SampleMatrix((r^(nHat\*n), r^(nHat\*n + 1), ..., r^(2\*nHat\*n - 1)), nHat, n)

7.  Generate the matrix A = Gen(seedA)

8.  Compute matrix B' = S'\*A + E'

9.  Compute c1 = Pack(B')

10. Sample error matrix E" = SampleMatrix((r^(2\*nHat\*n), r^(2\*nHat\*n + 1), ..., r^(2\*nHat\*n + nHat^2 - 1)), nHat, nHat)

11. Compute matrix B = Unpack(b, n, nHat)

12. Compute matrix V = S'\*B + E"

13. Compute matrix C = V + Encode(u)

14. Compute c2 = Pack(C)

15. Compute ss = SHAKE256(c1 \|\| c2 \|\| salt \|\|k, lensec)

16. Return ciphertext c = (c1 \|\| c2 \|\| salt) and shared secret ss


## Decapsulation

The decapsulation algorithm takes as input a ciphertext c = (c1 \|\| c2 \|\| salt) and
a secret key sk = (s \|\| seedA \|\| b \|\| S^T \|\| pkh), and outputs a shared secret ss.

1.  Compute matrix B' = Unpack(c1, nHat, n)

2.  Compute matrix C  = Unpack(c2, nHat, nHat)

3.  Compute matrix M = C - B'*S

4.  Compute u' = Decode(M)

5.  Generate pseudorandom values seedSE' \|\| k' = SHAKE256(pkh \|\| u' \|\|salt, lenSE+lensec)

6.  Generate pseudorandom bit string (r^(0), r^(1), ..., r^(2\*nHat\*n + nHat^2 - 1)) = SHAKE256(0x96 \|\| seedSE', 16
(2\*nHat\*n + nHat^2))

7.  Sample error matrix S' = SampleMatrix((r^(0), r^(1), ..., r^(nHat\*n - 1)), nHat, n)

8.  Sample error matrix E' = SampleMatrix((r^(nHat\*n), r^(nHat\*n + 1), ..., r^(2\*nHat\*n - 1)), nHat, n)

9.  Generate the matrix A  = Gen(seedA)

10.  Compute matrix B" = S'\*A + E'

11.  Sample error matrix E" = SampleMatrix((r^(2\*nHat\*n), r^(2\*nHat\*n + 1), ..., r^(2\*nHat\*n+ nHat^2 - 1)),
nHat, nHat)

12.  Compute matrix B = Unpack(b, n, nHat)

13.  Compute matrix V = S'\*B + E"

14.  Compute matrix C' = V + Encode(u')

15.  If B' = B" and C = C' then kHat = k' else kHat = s

16.  Compute ss = SHAKE256(c1 \|\| c2 \|\| salt \|\| kHat, lensec)

17.  Return shared secret ss


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