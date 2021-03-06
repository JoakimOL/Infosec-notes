# Hash functions {#sec:hash}

> A hash function, _H_, is a public function such that:\
> _H_ is simple and fast to compute\
> _H_ Takes as input a message, _m_, of arbitrary length and outputs a message digest H(_m_) of
> fixed length

Good hash functions show some properties:

1. Collision resistance
    - It should not be feasible to find two different inputs that produces the same output.
    - It's not possible to construct two values that produce the same hash
2. Second-preimage resistance
    - Given a value, it should be infeasible to find a different value that produces the same hash
    - It's not possible to construct a value that produce the same hash as a given value
3. One-way (preimage resistance)
    - Given a hash, it should be infeasible to find an input that produce the same hash.
    - You cannot find the input, given the output.

## The birthday paradox{#sec:hash:birthday}

If we choose $\sqrt{M}$ values from a set of size _M_, the probability of getting two identical values (or in
this context: hash collision) is about 50%. This is particularly useful in this course to compute
how many bits a hash should be in order to be safe.

If a hash function has an output of _k_ bits, and is regarded random to the outside world, then
$2^{\frac{k}{2}}$ attempts should yield a collision with a 50% probability. For those, including me,
who keeps forgetting math: $\sqrt{2^k} = 2^{\frac{k}{2}}$, so this is all according to the birthday
paradox.

Today (the date the slides were made) $2^{128}$ attempts is considered infeasible, so your hash
functions should output at least $2^{\frac{k}{2}} = 2^{128} \rightarrow k = 128*2 = 256$ bits in
order to be considered collision resistant.

## Iterated hash functions{#sec:hash:iterated}

Just like block ciphers, hash functions also need to be able to handle inputs of all sizes and
shapes to produce a fixed size output. Iterated hash functions solve this challenge like the
iterated block ciphers did, by splitting the input into fixed sized blocks and repeatedly using the
function.

Note that iterated hash functions operate on each block sequentially using the same function.

## Merkle-Damgård construction{#sec:hash:merkle}

> Use a fixed-size compression function applied to multiple blocks of the message

A compression function here is defined as a function that takes two _n_-bit input strings and
produces a single _n_-bit output string.

The Merkle-Damgård construction chains these together. An IV (similar to the ones used in block
ciphers) and the first block of a message, $m_1$, is input to the compression function. The output
is used as the input for the second compression, instead of the IV, in addition to the block, $m_2$.

Note that this scheme requires padding, as well as encoding of the length of _m_. In the last chain
of the process, the input is not a message block but the padding and encoded length.

In mathematical notation:\
compression function, $h(m_l, h_{l-1}) =  h_l$

Merkle-Damgård construction:\
$H(m) = h(PADDING, h_l)$\
$h_l = h(m_l, h_{l-1})$ where $h_0 = IV$\

### Properties of Merkle-Damgård construction{#sec:hash:merkle:properties}

If the compression function, _h_, is collision-resistant, then the hash function, _H_, is
collision-resistant.

The construction suffers from some weaknesses as well:\

1. Once you find a collision, it is easy to find more (length extension attack)
2. second pre-image attacks are not as hard as you'd think (construct a value that produce the same hash as a given value)
3. Collisions for multiple messages can be found without much more difficulty than collisions for 2
   messages.

Still, the Merkle-Damgård construction is used in standard and former standard hash functions (MD5,
SHA-1, SHA-2)

## Standardized hash functions{#sec:hash:standards}

Slides lack many implementational details, so I'm guessing its not important for the course. This
section only includes some basic key points.

### MDx family of hashes{#sec:hash:standards:mdx}

Old and insecure by today's standards. MD2, 4 and 5 have been used in practice, but are all easily
broken.

Is based on the Merkle-Damgård construction. They all output 128 bits. Recall the birthday paradox
and how many bits were recommended (@sec:hash:birthday).

### SHA-0 and SHA-1{#sec:hash:standards:sha}
Based off of MDx hashes (which makes it a Merkle-Damgård construction), but with added complexity and a bigger output size of 160 bits (weak).
Both are broken, but this is quite recent. First attack of SHA-1 was found in 2017.

### SHA-2 family{#sec:hash:standards:sha2}
Several versions of SHA-2 exist, hence the term "family of SHA-2 hashes". They are developed in
response to attacks on MD5 and SHA-1. Still a Merkle-Damgård construction.


|             | Hash size   | Block Size   | Security Match |
| ----        | ----------- | ------------ | -------------- |
| SHA-224     | 224 bits    | 512 bits     | 2key 3DES      |
| SHA-512/224 | 224 bits    | 1024 bits    | 2key 3DES      |
| SHA-256     | 256 bits    | 512 bits     | AES-128        |
| SHA-512/256 | 256 bits    | 1024 bits    | AES-128        |
| SHA-384     | 384 bits    | 1024 bits    | AES-192        |
| SHA-512     | 512 bits    | 1024 bits    | AES-256        |

: summary of SHA-2 family. Taken from lecture 10 slides of spring 2019.

The SHA-2 family is still a Merkle-Damgård construction so it needs a padding scheme. First off it needs a
field for the message length encoding. This field is 64 bits long if the block length is 512 bits, and
128 bits long if the block length is 1024 bits. After the message length field, there is padding.
There is always at least one bit of padding. After the first `1` in the padding, enough `0` are
added to get a complete block.

Since the padding requires at least 1 bit of pad and either 64 or 128 bits of encoding, this
sometimes results in adding a new block.


### SHA-3{#sec:hash:standards:sha3}
The MDx and previous SHA hashes were based on the same design, which has encountered unexpected
attacks. The SHA-3 hash is the result of a competition (just like AES), held in 2007-2008. This
ended up with a new function that was standardized in 2015 and is NOT based on the Merkle-Damgård
construction. It uses a sponge construction, whatever that is.


## HMAC{#sec:hash:hmac}
A MAC constructed from any iterated cryptographic hash function (like SHA256 etc).
HMAC is defined as: $HMAC(M,K) = H( (K \oplus opad) || H (( K \oplus ipad ) || M ) )$\

- M: Message to be authenticated
- K: Key padded with zeros to the blocksize of _H_
- opad: hardcoded string
- ipad: hardcoded string

HMACs are secure if _H_ is collision resistant or if H is a pseudorandom function. It is designed to
resist length extension attacks, even if _H_ is a Merkle-Damgård construction (which are vulnerable
to such attacks).

HMAC is often used as a pseudorandom function for deriving keys (since they are deterministic but
seem random)

# Authenticated encryption {#sec:authencryption}

## Combining encryption and MAC{#sec:authencryption:combining}

How do you ensure both confidentiality (no one can read your messages) and integrity (the messages
aren't tampered with)? A proposed solution is to split your assumed established shared key, _K_,
into two parts - one for encryption and one to obtain a MAC.

There are three possible ways to combine encryption and MACs:

1. Encrypt-and-MAC
    - Encrypt message, apply MAC to message and send the two results
    - $C \leftarrow Enc(M,K_1)$
    - $T \leftarrow MAC(M,K_2)$
    - Send $C||T$
2. MAC-then-encrypt
    - Apply MAC to message to get tag. Then encrypt message concatenated with tag and send the
      ciphertext.
    - $T \leftarrow MAC(M,K_1)$
    - $C \leftarrow Enc(M||T,K_2)$
    - Send $C$
3. encrypt-then-MAC
    - encrypt message to get ciphertext. Then apply MAC to ciphertext and send the two results
    - $C \leftarrow Enc(M,K_1)$
    - $T \leftarrow MAC(C,K_2)$
    - Send $C||T$

Encrypt-then-MAC is the safest of the three. Encrypt-and-MAC is the worst because the MAC cannot
guarantee that it wont leak information. MAC-then-encrypt has the same weakness, but at least it
encrypts the MAC.

Some schemes do, however, provide both confidentiality and integrity with one key.

## Modes for Authenticated encryption {#sec:authencryption:modes}
There are two types of input data:

- Payload data: both encrypted and authenticated
- Associated data: only authenticated

Both modes of operation use the CTR-mode to add confidentiality, but add integrity in different
ways. They also allow some data to be only authenticated, not encrypted, providing ``authenticated
encryption with associated data`` (AEAD). This property is important in newer versions of TLS.

### Counter with CBC-MAC mode (CCM) {#sec:authencryption:modes:CCM}
This mode offers confidentiality for the payload, and authentication for both payload and associated
data.

The nonce, _N_, payload, _P_ and associated data, _A_ needs to be formatted to produce a set of
blocks. The format is complex and different implementations exist in different standards. By blindly
following the slides, we see that the lengths of _N_ and _P_ are included in the first block. If _A_
is non-zero, it is formatted from the second block up to its length.

After formatting, we compute a $T_{len}$ bits long tag for these blocks. Using normal counter mode,
we compute _m_ blocks, where $m = \lceil{\frac{P_{len}}{128}}\rceil$

Output $C = (P \oplus MSB_{plen}(S)) || (T \oplus MSB_{tlen}(S_0))$,\
    where $S = S_1, ... , S_m$,\
    and   $MSB_{n}(S)$ returns the _n_ most significants bits from _S_.


### Galois Counter mode (GCM){#sec:authencryption:modes:GCM}
A problem with CCM is that the formatting of _N_,_A_ and _P_ requires the length of both _A_ and
_P_. This prevents it from being used in a streaming application. GCM overcomes this limitation. It
combines CTR mode on a block cipher, _E_ (AES is a good and common choice), with a hash function called
`GHASH`.

Skipping implementational details.

The receiver receives the ciphertext, _C_, the nonce, _N_, the tag, _T_, and the authenticated data,
_A_. This is all the receiver needs to recompute _T_ and check whether it matches the received _T_.
If it does, the receiver can decrypt _C_ just like in CTR-mode.

This is used in TLS 1.2.
