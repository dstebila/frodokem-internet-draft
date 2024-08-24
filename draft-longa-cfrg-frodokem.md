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

FrodoKEM [Frodo17] is a conservative yet practical post-quantum key encapsulation
mechanism (KEM) whose security derives from cautious parameterizations of the
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
parties, A and B, to derive a shared secret in an interactive fashion:
In the first pass, party B producing a ciphertext with the _Encapsulate_
operation needs to first receive or retrieve the public key from party A,
who is responsible for _KeyGen_ and _Decapsulate_.
In the second pass, this party A uses its secret key and the ciphertext
to execute the _Decapsulate_ operation.
The shared secret produced by this protocol can then be used to establish
a secure communication channel using a symmetric-key algorithm.

# Overview

The core of FrodoKEM is a public-key encryption scheme called FrodoPKE,
whose IND-CPA security is tightly related to the hardness of a corresponding
learning with errors problem. Here we briefly recall the scientific lineage
of these systems. See the surveys [Mic10, Reg10, Pei16] for further details.
The seminal works of Ajtai [Ajt96] (published in 1996) and Ajtai–Dwork [AD97]
(published in 1997) gave the first cryptographic constructions whose
security properties followed from the conjectured worst-case hardness of
various problems on point lattices in R^n. In subsequent years, these works
were substantially refined and improved. Notably, in work published in 2005,
Regev [Reg09] defined the learning with errors (LWE) problem, proved the
hardness of (certain parameterizations of) LWE assuming the hardness of
various worst-case lattice problems against quantum algorithms, and defined
a public-key encryption scheme whose IND-CPA security is tightly related to
the hardness of LWE.

Later on, in work published in 2011, Lindner and Peikert [LP11] gave a more
efficient LWE-based public-key encryption scheme that uses a square public
matrix A of dimension n with integer coefficients modulo q, instead of an
oblong rectangular one.

The FrodoPKE scheme described in this document is an instantiation and
implementation of the Lindner–Peikert scheme [LP11] with some modifications,
such as: pseudorandom generation of the public matrix A from a small seed,
more balanced key and ciphertext sizes, and new LWE parameters.

## Chosen-ciphertext security

FrodoKEM achieves IND-CCA security by way of a transformation of the
IND-CPA-secure FrodoPKE. In work published in 1999, Fujisaki and Okamoto
[FO99] gave a generic transform from an IND-CPA PKE to an IND-CCA PKE, in
the random-oracle model. At a high level, the Fujisaki–Okamoto (FO) transform
derives encryption coins pseudorandomly, and decryption regenerates these
coins to re-encrypt and check that the ciphertext is well-formed. In 2016,
Targhi and Unruh [TU16] gave a modification of the Fujisaki–Okamoto transform
that achieves IND-CCA security in the quantum random-oracle model (QROM) by
adding an extra hash.
In 2017, Hofheinz, Hovelmanns, and Kiltz [HHK17] gave several variants of the
Fujisaki–Okamoto and Targhi–Unruh transforms that in particular convert an
IND-CPA-secure PKE into an IND-CCA-secure KEM, and analyzed them in both the
classical and quantum random-oracle models, even for PKEs with non-zero
decryption error. Jiang et al. [JZC+18] show how to prove security of one of
these variant FO transforms in the QROM without requiring the extra hash from
Targhi–Unruh. FrodoKEM is constructed from FrodoPKE using a slight variant
of Jiang et al.'s transform that includes additional values in hash
computations to reduce the risk of multi-target attacks.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview

# Notation

We describe the symbols and abbreviations used throughout this document.

Z represents the set of integers and Z_q represents the set of integers
modulo q.

$\lfloor$ x $\rceil$ is	the rounding of x to the nearest integer. If
x = y + 1/2 for some y in Z, then $\lfloor$ x $\rceil$ = y + 1.

A 16-bit bit string is represented by r^(i). And a sequence of t 16-bit
bit strings r^(i) is represented by (r^(0), r^(1), ..., r^(t-1)).

AES128(k, a) denotes the 128-bit AES128 output under key k for a 128-bit
input a.

SHAKE128(x, y) and SHAKE256(x, y) denote the y first bits of SHAKE128
and SHAKE256 (resp.) output for input x.

Matrices are represented in capitals with no italics (e.g., A and C).
For an n1 \* n2 matrix C, its (i,j)th coefficient (i.e., the entry in the
ith row and jth column) is denoted by C_(i,j), where 0 <= i < n1 and
0 <= j < n2. The transpose of matrix C is denoted by C^T.

AES128 and SHAKE are specified in [FIPS197] and [FIPS202], respectively. 

# Parameters

The FrodoKEM parameters are implicit inputs to the FrodoKEM algorithms
defined in the next sections. A FrodoKEM parameter set specifies the
following:

A positive integer D <= 16 that defines the modulus parameter q = 2^D.

Positive integers n, nHat specifying matrix dimensions. It holds that n,
nHat \equiv 0 mod 8.

A positive integer B <= D specifying the number of bits encoded in each
matrix entry.

A positive integer lenA specifying the bitlength of seeds for the
generation of the matrix A.

A positive integer lensec specifying the number of bits that match the
bit-security level. Valid values are 128, 192 and 256. This is used to
determine the bitlength of seeds (not associated to the matrix A), of hash
value outputs and of values associated to the generation of the shared
secrets.

A positive integer lenSE specifying the bitlength of the seed value seedSE.

A positive integer lensalt specifying the bitlength of the value salt.

A discrete, symmetric error distribution X on Z with support given by
S_X = \{−d, −d+1, ..., −1, 0, 1, ..., d−1, d\} for a small integer d.

A table T_X = (T_X(0), T_X(1), ..., T_X(d)) with (d+1) positive integers
based on the cumulative distribution function for X.

The values for these parameters corresponding to each parameter set are
given in Section XXXX.

# Supporting functions

## Octet encoding of bit strings

This document follows the little-endian formatting for octet encoding of
bit strings.

A bit string b = (b_0, b_1, ..., b_(|b|-1)) is converted to an octet
string by taking bits from left to right, packing those from the least
significant bit of each octet to the most significant bit, and moving to
the next octet when each octet fills up. For example, the 16-bit bit
string (b_0, b_1, ..., b_15) is converted into two octets f and g (in
this order) as

f = b_7 \* 2^7 + b_6 \* 2^6 + b_5 \* 2^5 + b_4 \* 2^4 + b_3 \* 2^3 + b_2 \* 2^2 + b_1 \* 2 + b_0
g = b_15 \* 2^7 + b_14 \* 2^6 + b_13 \* 2^5 + b_12 \* 2^4 + b_11 \* 2^3 + b_10 \* 2^2 + b_9 \* 2 + b_8

The conversion from octet string to bit string is the reverse of this
process.

For FrodoKEM, it is always the case that \|b\| is a multiple of 8 when
performing octet encoding of bit strings.

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

1. Set t = r_1 \* 2^0 + r_2 * 2^1 + ... + r_15 \* 2^14

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

      2. C_(i,j) \|\| C_(i,j+1) \|\| ... \|\| C_(i,j+7) = AES128(seed_A, b), where each matrix coefficient C_(i,j) is a 16-bit string interpreted as a nonnegative integer in the little-endian byte order, such that C_(i,j) = c_0 \* 2^0 + c_1 *\ 2^1 + ... + c_15 *\ 2^15 corresponding to the bit string (c_0, c_1, ...,c_15)

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

   2. C_(i,0) \|\| C_(i,1) \|\| ... \|\| C_(i,n-1) = SHAKE128(b, 16\*n), where each matrix coefficient C_(i,j) is a 16-bit string interpreted as a nonnegative integer in the little-endian byte order, such that C_(i,j) = c_0 \* 2^0 + c_1 *\ 2^1 + ... + c_15 *\ 2^15 corresponding to the bit string (c_0, c_1, ..., c_15)

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

2. Generate pseudorandom seed seedA = SHAKE(z, lenA)

3. Generate the matrix A = Gen(seedA)

4. Generate pseudorandom bit string (r^(0), r^(1), ..., r^(2\*n\*nHat - 1)) = SHAKE(0x5F \|\| seedSE, 32\*n\*nHat)

5. Sample error matrix S^T = SampleMatrix((r^(0), r^(1), ..., r^(n\*nHat − 1)), nHat, n)

6. Sample error matrix E = SampleMatrix((r^(n\*nHat), r^(n\*nHat + 1), ..., r^(2\*n\*nHat − 1)), n, nHat)

7. Compute matrix B = A\*S + E

8. Compute b = Pack(B)

9. Compute pkh = SHAKE(seedA \|\| b, lensec)

10. Return public key pk = (seedA \|\| b) and secret key sk = (s \|\| seedA \|\| b \|\| S^T \|\| pkh).
ST = S^T is encoded row-by-row from ST_(0,0) to ST_(nHat−1,n−1), where
each matrix coefficient ST_(i,j) is a signed integer encoded as a 16-bit string
in the little-endian byte order such that ST_(i,j) = −s_15 * 2^15 + (s_0 + s_1 * 2 + s_2 * 2^2 + ... + s_14 * 2^14)
corresponding to the bit string (s_0, s_1, ..., s_15)

## Encapsulation

The encapsulation algorithm takes as input a public key pk = (seedA \|\| b), and
outputs a ciphertext c = (c1 \|\| c2 \|\| salt) and a shared secret ss.

1.  Choose uniformly random values u and salt of bitlengths lensec and lensalt (resp.)

2.  Compute pkh = SHAKE(pk, lensec)

3.  Generate pseudorandom values seedSE \|\| k = SHAKE(pkh \|\| u \|\| salt, lenSE+lensec)

4.  Generate pseudorandom bit string (r^(0), r^(1), ..., r^(2\*nHat\*n + nHat^2 - 1)) = SHAKE(0x96 \|\| seedSE, 16(2\*nHat\*n + nHat^2))

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

15. Compute ss = SHAKE(c1 \|\| c2 \|\| salt \|\|k, lensec)

16. Return ciphertext c = (c1 \|\| c2 \|\| salt) and shared secret ss

## Decapsulation

The decapsulation algorithm takes as input a ciphertext c = (c1 \|\| c2 \|\| salt) and
a secret key sk = (s \|\| seedA \|\| b \|\| S^T \|\| pkh), and outputs a shared secret ss.

1.  Compute matrix B' = Unpack(c1, nHat, n)

2.  Compute matrix C  = Unpack(c2, nHat, nHat)

3.  Compute matrix M = C - B'*S

4.  Compute u' = Decode(M)

5.  Generate pseudorandom values seedSE' \|\| k' = SHAKE(pkh \|\| u' \|\|salt, lenSE+lensec)

6.  Generate pseudorandom bit string (r^(0), r^(1), ..., r^(2\*nHat\*n + nHat^2 - 1)) = SHAKE(0x96 \|\| seedSE', 16
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

16.  Compute ss = SHAKE(c1 \|\| c2 \|\| salt \|\| kHat, lensec)

17.  Return shared secret ss

# FrodoKEM variants

FrodoKEM is parameterized by the pseudorandom generator (PRG) that is used for
the generation of the matrix A. As explained in Section XXXX there are two options
for PRG: AES128 and SHAKE128.

In addition, FrodoKEM consists of two main variants: a "standard" variant that does
not impose any restriction on the reuse of key pairs, and an "ephemeral" variant
that is intended for applications in which the number of ciphertexts produced
relative to any single public key is small. Concretely, the use of standard
FrodoKEM is recommended for applications in which the number of ciphertexts
produced for a single public key is expected to be equal or greater than 2^8.
Ephemeral FrodoKEM shall be used for applications in which that same figure is
expected to be smaller than 2^8.

In contrast to eFrodoKEM, standard FrodoKEM incorporates some changes to address
certain multi-ciphertext attacks [Annex]. Specifically, standard FrodoKEM doubles the
length of the seedSE value and incorporates a public random salt value into
encapsulation (see Table 2).
 

# Parameter Sets

This document specifies the following twelve parameter sets:

1. FrodoKEM-640-⟨PRG⟩ and eFrodoKEM-640-⟨PRG⟩, which match or exceed the brute-force security of AES128.

2. FrodoKEM-976-⟨PRG⟩ and eFrodoKEM-976-⟨PRG⟩, which match or exceed the brute-force security of AES192.

3. FrodoKEM-1344-⟨PRG⟩ and eFrodoKEM-1344-⟨PRG⟩, which match or exceed the brute-force security of AES256.

The label "eFrodoKEM" corresponds to the ephemeral variants.
The options for ⟨PRG⟩ are AES or SHAKE, when either AES128 or SHAKE128 (respectively)
is used for the generation of the matrix A. Thus, the first FrodoKEM variant consists
of the parameter sets FrodoKEM-640-AES, FrodoKEM-976-AES and FrodoKEM-1344-AES (and
their corresponding ephemeral variants). The second FrodoKEM variant consists of the
parameter sets FrodoKEM-640-SHAKE, FrodoKEM-976-SHAKE and FrodoKEM-1344-SHAKE (and
their corresponding ephemeral variants).

## Parameters

|   Name  | (e)FrodoKEM-640 | (e)FrodoKEM-976 | (e)FrodoKEM-1344 | Description                                    |
|--------:|:---------------:|:---------------:|:----------------:|:-----------------------------------------------|
|       D |        15       |        16       |        16        | Bitlength of q                                 |
|       q |      32768      |      65536      |      65536       | Power-of-two integer modulus                   |
|       n |       640       |       976       |       1344       | Integer matrix dimension                       |
|    nHat |        8        |        8        |         8        | Integer matrix dimension                       |
|       B |        2        |        3        |         4        | Number of bits encoded per matrix entry        |
|       d |        12       |        10       |         6        | Integer defining the support of X              |
|    lenA |       128       |       128       |        128       | Bitlength of seeds for generation of matrix A  |
|  lensec |       128       |       192       |        256       | Number of bits matching the bit-security level |
|   SHAKE |    SHAKE128     |     SHAKE256    |     SHAKE256     | SHAKE variant used for hashing                 |

                                       Table 1: Parameters for FrodoKEM.

|   Name  |   FrodoKEM-640  |  FrodoKEM-976   |   FrodoKEM-1344  | Description                                    |
|--------:|:---------------:|:---------------:|:----------------:|:-----------------------------------------------|
|   lenSE |       256       |       384       |       512        | Bitlength of seedSE in FrodoKEM                |
| lensalt |       256       |       384       |       512        | Bitlength of salt in FrodoKEM                  |

|   Name  |  eFrodoKEM-640  |  eFrodoKEM-976  |  eFrodoKEM-1344  | Description                                    |
|--------:|:---------------:|:---------------:|:----------------:|:-----------------------------------------------|
|   lenSE |       128       |       192       |       256        | Bitlength of seedSE in eFrodoKEM               |
| lensalt |        0        |        0        |        0         | No salt in eFrodoKEM                           |

                        Table 2: Additional parameters for FrodoKEM depending on variant.

|     Name     |  sigma |    0  |  +-1  |  +-2 |  +-3 |  +-4 |  +-5 | +-6 | +-7 | +-8 | +-9 |+-10|+-11|+-12| order |   divergence  |
|-------------:|:------:|------:|------:|-----:|-----:|-----:|-----:|----:|----:|----:|----:|---:|---:|---:|:-----:|:-------------:|
|  X_Frodo-640 |   2.8  |  9288 |  8720 | 7216 | 5264 | 3384 | 1918 | 958 | 422 | 164 |  56 | 17 |  4 |  1 |  200  | 0.324 x 10^-4 |
|  X_Frodo-976 |   2.3  | 11278 | 10277 | 7774 | 4882 | 2545 | 1101 | 396 | 118 |  29 |   6 |  1 |    |    |  500  | 0.140 x 10^-4 |
| X_Frodo-1344 |   1.4  | 18286 | 14320 | 6876 | 2023 |  364 |   40 |   2 |     |     |     |    |    |    | 1000  | 0.264 x 10^-4 |

                     Table 3: Error distributions. Probabilities are shown for each integer value from 0 up to +-12.
                                    The last two columns correspond to Renyi's order and divergence.

| Table entries |  FrodoKEM-640  |  FrodoKEM-976  |  FrodoKEM-1344  |
|--------------:|:--------------:|:--------------:|:---------------:|
|        T_X(0) |      4,643     |      5,638     |       9,142     |
|        T_X(1) |     13,363     |     15,915     |      23,462     |
|        T_X(2) |     20,579     |     23,689     |      30,338     |
|        T_X(3) |     25,843     |     28,571     |      32,361     |
|        T_X(4) |     29,227     |     31,116     |      32,725     |
|        T_X(5) |     31,145     |     32,217     |      32,765     |
|        T_X(6) |     32,103     |     32,613     |      32,767     |
|        T_X(7) |     32,525     |     32,731     |                 |
|        T_X(8) |     32,689     |     32,760     |                 |
|        T_X(9) |     32,745     |     32,766     |                 |
|       T_X(10) |     32,762     |     32,767     |                 |
|       T_X(11) |     32,766     |                |                 |
|       T_X(12) |     32,767     |                |                 |

            Table 4: The distribution table entries T_X(i),
                  for 0 <= i <= d, for sampling.

|                |  secret key sk  |  public key pk  |  ciphertext ct  | shared secret ss |
|---------------:|:---------------:|:---------------:|:---------------:|:----------------:|
|   FrodoKEM-640 |      19,888     |       9,616     |       9,752     |        16        |
|  eFrodoKEM-640 |      19,888     |       9,616     |       9,720     |        16        |
|   FrodoKEM-976 |      31,296     |      15,632     |      15,792     |        24        |
|  eFrodoKEM-976 |      31,296     |      15,632     |      15,744     |        24        |
|  FrodoKEM-1344 |      43,088     |      21,520     |      21,696     |        32        |
| eFrodoKEM-1344 |      43,088     |      21,520     |      21,632     |        32        |

                        Table 5: Sizes (in bits) of inputs and outputs.


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

# References

## Normative References

[FIPS197]   NIST, FIPS 197, "Advanced Encryption Standard (AES)",
            November 2001.
            https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197-upd1.pdf

[FIPS202]   NIST, FIPS 202, "SHA-3 Standard: Permutation-Based Hash and Extendable-Output Functions",
            August 2015.
            https://nvlpubs.nist.gov/nistpubs/fips/nist.fips.202.pdf

            
## Informative References

[Frodo17]   E. Alkim, J. W. Bos, L. Ducas, K. Easterbrook, P. Longa, B. LaMacchia, I. Mironov, M. Naehrig, V. Nikolaenko, C. Peikert, A. Raghunathan, and D. Stebila.
            FrodoKEM: Learning With Errors Key Encapsulation, 2017.
            NIST's Round 3 algorithm specification and supporting documentatation available at https://frodokem.org/files/FrodoKEM-specification-20210604.pdf (2021)

[Annex]     FrodoKEM Team. Annex on FrodoKEM updates, April 2023.
            https://frodokem.org/files/FrodoKEM-annex-20230418.pdf

[Mic10]     D. Micciancio. Cryptographic functions from worst-case complexity assumptions.
            Information Security and Cryptography, pages 427–452, 2010.

[Reg10]     O. Regev. The learning with errors problem (invited survey).
            In IEEE Conference on Computational Complexity, pages 191–204, 2010.
 
[Pei16]     C. Peikert. A decade of lattice cryptography.
            Foundations and Trends in Theoretical Computer Science, 10(4):283–424, 2016.

[Ajt96]     M. Ajtai. Generating hard instances of lattice problems (extended abstract).
            In 28th Annual ACM Symposium on Theory of Computing, pages 99–108, 1996.

[AD97]      M. Ajtai and C. Dwork. A public-key cryptosystem with worst-case/average-case equivalence.
            In 29th Annual ACM Symposium on Theory of Computing, pages 284–293, 1997.

[Reg09]     O. Regev. On lattices, learning with errors, random linear codes, and cryptography.
            Journal of the ACM, 56(6):34, 2009. Preliminary version in STOC 2005.

[LP11]      R. Lindner and C. Peikert. Better key sizes (and attacks) for LWE-based encryption.
            In Topics in Cryptology – CT-RSA 2011, volume 6558 of Lecture Notes in Computer Science, pages 319–339, 2011.

[FO99]      E. Fujisaki and T. Okamoto. Secure integration of asymmetric and symmetric encryption schemes.
            In Advances in Cryptology – CRYPTO’99, volume 1666 of Lecture Notes in Computer Science, pages 537–554, 1999.

[TU16]      E. E. Targhi and D. Unruh. Post-quantum security of the Fujisaki-Okamoto and OAEP transforms.
            In TCC 2016-B: 14th Theory of Cryptography Conference, Part II, volume 9986 of Lecture Notes in Computer Science, pages 192–216, 2016.

[HHK17]     D. Hofheinz, K. Hovelmanns, and E. Kiltz. A modular analysis of the Fujisaki-Okamoto transformation.
            In TCC 2017: 15th Theory of Cryptography Conference, Part I, volume 10677 of Lecture Notes in Computer Science, pages 341–371, 2017.

[JZC+18]    H. Jiang, Z. Zhang, L. Chen, H. Wang, and Z. Ma. IND-CCA-secure key encapsulation mechanism in the quantum random oracle model, revisited.
            In Advances in Cryptology – CRYPTO 2018, Part III, volume 10993 of Lecture Notes in Computer Science, pages 96–125, 2018.
