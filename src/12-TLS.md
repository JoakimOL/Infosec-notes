# TLS
Is one of the most widely used security protocol of today.

It is the successor of SSL, which is now advised against, and is still being developed - the newest
version, TLS 1.3, was released in 2018. TLS is based on the use of a PKI and is often used to
establish secure sessions with web servers, among other things. The design goal is to secure
reliable end-to-end services over TCP (although UDP-variants exist).

TLS consists of three higher level protocols:

1. TLS Handshake protocol to set up secure sessions
2. TLS alert protocol to signal events such as failures
3. TLS change cipher spec protocol to change the cryptographic algorithms in use

As well as the TLS record protocol which provides basic services to the higher level protocols.

## Record protocol
The record protocol aims to provide two services for TLS connections, that we have discussed earlier
in the course:

1. Message confidentiality:
    - message contents cannot be read in transit
2. message integrity:
    - detect tampering with messages

These services are provided by a symmetric encryption scheme and a MAC. Later TLS-versions (1.2+)
also offer these services by using authenticated encryption modes (CCM and GCM, see own section).
Necessary keys are set up by the handshake protocol.

The record protocol format contains a header describing the type and length of the content, as well
as a specification of which TLS version in use. Legal types include:

- change-cipher-spec
- alert
- handshake
- application-data

After the header comes an encrypted part containing the message, optionally
compressed (compression removed in version 1.3, due to CRIME 2012 attack), and a MAC field in case authenticated encryption is not used.

The record protocol splits each application layer message in blocks of $2^{14}$ bytes or less.

### Cryptographic algorithms in TLS

MAC:

The HMAC scheme is used to provide integrity in TLS, with an agreed-upon hash functions
    - SHA2 is only allowed in TLS 1.2+
    - MD5 and SHA-1 not allowed in TLS 1.3

Encryption:

An agreed-upon block cipher in CBC-mode or stream cipher is used. Most commonly AES.

    - RC4 and 3DES is supported in TLS 1.2
    - Both removed from TLS 1.3 (in order to avoid a handful of attacks)

Authenticated encryption:

Using authenticated encryption is also allowed from TLS 1.2, replacing the MAC and encryption algorithm.
In TLS 1.3, the only allowed configuration is AES in CCM or GCM modes. Authenticated additional data
is the header and implicit record sequence number.

