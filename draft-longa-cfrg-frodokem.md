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

# Supporting functions

## Matrix encoding of bit strings

We define how bit strings are encoded as mod-q integer matrices.

Recall that 2^B <= q. The encoding function ec() encodes an integer
0 <= val < 2^B as an element in Z_q by multiplying it by q/2^B = 2^(D-B):

ec(val) = val \* q/2^B.

Using this function, the function Encode(b) encodes a given bit string
b = (b_0, ..., b_(l-1)) of length l = B \* nHat^2 as an nHat \* nHat
matrix C with coefficients C_(i,j) in Z_q by applying ec(\cdot) to B-bit
sub-strings sequentially and filling the matrix row by row entry-wise.
The function Encode(b) is defined as follows.

1. For i = 0 to nHat - 1 do

   1. For j = 0 to nHat - 1 do

      1. val = 0

      2. For k = 0 to B - 1 do

         1. val = val + b_((i\*nHat + j)B + k) \* 2^k

      3. End for

      4. Set C_(i,j) = val \* q/2^B

   2. End for

2. End for

3.  Return C

The corresponding decoding function Decode(C) decodes an nHat \* nHat
matrix C into a bit string of length l = B \* nHat^2. It extracts B bits
from each entry by applying the function dc():

dc(c) = $\lfloor$ c \* 2^B/q $\rceil$ mod 2^B.

That is, the Z_q-entry is interpreted as an integer, then divided by q/2^B
and rounded. This amounts to rounding to the B most significant bits of
each entry. With these definitions, it is the case that dc(ec(val)) = val
for all 0 ≤ val < 2^B.
The function Decode(C) is defined as follows.

1. For i = 0 to nHat - 1 do

   1. For j = 0 to nHat - 1 do

      1. c = $\lfloor$ C_(i,j) \* 2^B/q $\rceil$ mod 2^B

      2. Set c = c_0 * 2^0 + c_1 * 2^1 + ... + c_(B-1) * 2^(B-1)

      3. For k = 0 to B - 1 do

         1. b_((i*nHat + j)B + k) = c_k

      4. End for

   2. End for

2. End for

3. Return (b_0, ..., b_(l-1)).

## Packing matrices modulo q

We define packing and unpacking functions to transform matrices with entries
in Z_q to bit strings and vice versa.

The function Pack packs an n1 \* n2 matrix C with entries C_(i,j) in Z_q to an
octet string by concatenating the D-bit matrix coefficients.
The function Pack(C) is defined as follows.

1. For i = 0 to n1 - 1 do

   1. For j = 0 to n2 - 1 do

      1. Set C_(i,j) = c_0 * 2^0 + c_1 * 2^1 + ... + c_(D-1) * 2^(D-1)

      2. For k = 0 to D - 1 do

         1. b_((i\*n2 + j)D + k) = c_(D-1-k)

      3. End for

   2. End for

   3. End for

2. Output the octet string corresponding to the bit string b = (b_0, b_1, ..., b_(D\*n1\*n2 - 1)), as per XXXX.

The function Unpack does the reverse of this process to transform an octet string o
to an n1 \* n2 matrix C with entries C_(i,j) in Z_q, converting the input to a bit
string, and then extracting D-bit strings and storing each as matrix coefficients
C_(i,j) for 0 <= i < n1 and 0 <= j < n2 (row-by-row from C_(0,0) to C_(n1-1,n2-1)).
The function Unpack(o, n1, n2) is defined as follows:

1. Convert the input octet string o to a bit string b = (b_0, b_1, ..., b_(D\*n1\*n2 - 1)), as per XXXX.

2. For i = 0 to n1 - 1 do

   1. For j = 0 to n_2 - 1 do

      1. C_(i,j) = 0

      2. For k = 0 to D - 1 do

         1. C_(i,j) = C(i,j) + b_((i\*n2 + j)D + k) \* 2^(D-1-k)

      3. End for

   2. End for

3. End for

4. Output C

## Sampling from the error distribution

The error distribution X used in FrodoKEM is a discrete, symmetric distribution on
Z, centered at zero and with small support, which approximates a rounded continuous
Gaussian distribution.

The support of X is S_X = \{−d, −d+1, ..., −1, 0, 1, ..., d−1, d\} for a positive
integer d. The probabilities X(z) = X(−z) for z in S_X are given by a discrete
probability density function, which is described by a table

T_X = (T_X(0), T_X(1), ..., T_X(d))

of d+1 positive integers related to the cumulative distribution function.

Given a random bit string r = (r_0, r_1, ..., r_15), the function Sample(r) returns
a sample e from FrodoKEM’s error distribution X via inversion sampling using a
table T_X, as follows (note that T_X(d) is never accessed):

1. Set t = r_1 \* 2^0 + r_2 * 2^1 + ... + r_15 \* 2^14 \equiv (r_1, r_2, ..., r_15, 0), interpreted as a nonnegative integer in the little-endian byte order

2. e = 0

3. For i = 0 to d - 1 do

   1. If t > T_X(i) then

      1. e = e + 1

   2. End if

4. End for

5. e = (-1)^(r_0) \* e

6. Output e

The output of the algorithm is a small integer in the range
\{-d, -d+1, ..., -1, 0, 1, ..., d-1, d\}. The tables T_X corresponding to each of
FrodoKEM’s parameter sets are given in Table XXXX.

We emphasize that it is important to perform this sampling in constant time to
avoid exposing timing side-channels, which is why the for-loop of the algorithm
does a complete loop through the entire table T_X. Similarly, the comparison in
the if-loop needs to be implemented in a constant-time manner.

## Matrix sampling from the error distribution

We define the function SampleMatrix which samples an n1 \* n2 matrix using the
function Sample. 

Given (n1 \* n2) 16-bit random strings r^(i) and the dimension values n1 and n2,
SampleMatrix((r^(0), ..., r^(n1\*n2 - 1)), n1, n2) generates an n1 \* n2 matrix
E row-by-row from E_(0,0) to E_(n1-1,n2-1) by successively calling the function
Sample n1 \* n2 times, as follows:
	
1. For i = 0 to n1 - 1 do

   1. For j = 0 to n2 - 1 do

      1. E_(i,j) = Sample(r^(i\*n2 + j))

   2. End for

2. End for

3. Output E

## Pseudorandom matrix generation

The function Gen takes as input a seed, seedA, of length lenA=128 bits and an
implicit dimension n that is fixed per parameter set, and outputs an n \* n
pseudorandom matrix A, where all the coefficients are in Z_q.
There are two options for instantiating Gen: one based on AES128 and another
based on SHAKE128. 
In both cases, the matrix A is generated row-by-row from A_(0,0) to A_(n-1,n-1).

### Matrix A generation with AES128

The algorithm for the case using AES128 is shown below. Each call to AES128
generates 8 coefficients.

1. For i = 0 to n - 1 do

   1. For j = 0 to n - 1 step 8 do

      1. b = i\|\|j\|\|0\|\|0\|\|0\|\|0\|\|0\|\|0, where each concatenated element is encoded as a 16-bit string represented in the little-endian byte order such that, e.g., (i_0, i_1, ..., i_15) \equiv i_0 \* 2^0 + i_1 \* 2^1 + ... + i_15 *\ 2^15, and \|b\| = 128

	    2. C_(i,j) \|\| C_(i,j+1) \|\| ... \|\| C_(i,j+7) = AES128(seed_A, b), where each matrix coefficient C_(i,j) is a 16-bit string interpreted as a nonnegative integer in the little-endian byte order, such that C_(i,j) = (c_0, c_1, ...,c_15) \equiv c_0 \* 2^0 + c_1 *\ 2^1 + ... + c_15 *\ 2^15

      3. For k = 0 to 7 do 

   	    1. A_(i,j+k) = C_(i,j+k) mod q

	    4. End for

  2. End for

2. End for

3. Output A
   
### Matrix A generation with SHAKE128

The algorithm for the case using SHAKE128 is shown below. Each call to SHAKE128
generates n coefficients (i.e., a full matrix row).

1. For i = 0 to n - 1 do

   1. b = i\|\|seed_A, where i is encoded as a 16-bit string represented in the little-endian byte order such that (i_0, i_1 , ..., i_15) \equiv i_0 \* 2^0 + i_1 \* 2^1 + ... + i_15 *\ 2^15, and hence the bitlength of b is \|b\| = lenA+16

   2. C_(i,0) \|\| C_(i,1) \|\| ... \|\| C_(i,n-1) = SHAKE128(b, 16\*n), where each matrix coefficient C_(i,j) is a 16-bit string interpreted as a nonnegative integer in the little-endian byte order, such that C_(i,j) = (c_0, c_1, ..., c_15)\equiv c_0 \* 2^0 + c_1 *\ 2^1 + ... + c_15 *\ 2^15

   3. For j = 0 to n - 1 do

      1. A_(i,j) = C_(i,j) mod q

   4. End for

2. End for

3. Output A


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
