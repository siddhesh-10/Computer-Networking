# Networking — Phase 15: TLS & HTTPS

> **Goal:** Understand how TLS provides confidentiality, integrity, authentication, and secure HTTPS communication.

## 1. Why TLS?

Without TLS:

```text
Client ───── plaintext ─────→ Server
```

With TLS:

```text
Client ─── encrypted data ───→ Server
```

TLS provides:

```text
Confidentiality
Integrity
Authentication
```

---

## 2. Symmetric Encryption

The same secret key is used for encryption and decryption.

```text
Plaintext
   ↓
Secret key
   ↓
Encryption
   ↓
Ciphertext
   ↓
Secret key
   ↓
Decryption
```

Symmetric encryption is efficient and is used for bulk application data.

---

## 3. Asymmetric Encryption

Asymmetric cryptography uses:

```text
Public key
Private key
```

It is useful for:

```text
Authentication
Digital signatures
Key establishment
```

It is generally more computationally expensive than symmetric encryption.

---

## 4. Hashing

A cryptographic hash produces a fixed-size digest:

```text
Data
 ↓
Hash function
 ↓
Digest
```

Hashing is not encryption:

```text
Encryption → designed for reversible protection with a key
Hashing    → one-way digest function
```

Hashes are used in many TLS cryptographic operations.

---

## 5. Digital Signatures

A digital signature helps prove:

```text
Who signed the data
+
Data was not modified
```

Conceptually:

```text
Data
 ↓
Hash
 ↓
Private key
 ↓
Signature
```

The receiver can verify it using the corresponding public key.

---

## 6. Certificates

A TLS certificate binds an identity such as:

```text
example.com
```

to a public key.

It contains information such as:

```text
Domain / subject
Public key
Issuer
Validity period
Signature
```

The client uses it to help authenticate the server.

---

## 7. Certificate Authorities

A **Certificate Authority (CA)** signs certificates.

```text
CA
 ↓ signs
Server certificate
 ↓
example.com + public key
```

Browsers and operating systems trust a set of root CA certificates.

---

## 8. Certificate Chain

Validation often follows:

```text
Root CA
   ↓
Intermediate CA
   ↓
Server certificate
   ↓
example.com
```

The client verifies signatures and trust relationships up to a trusted root.

---

## 9. TLS Handshake

Simplified:

```text
Client
  ↓
ClientHello
  ↓
ServerHello
  ↓
Server Certificate
  ↓
Key establishment
  ↓
Handshake verification
  ↓
Encrypted application data
```

The exact messages differ between TLS versions.

---

## 10. TLS 1.2

TLS 1.2 supports:

```text
Cipher suite negotiation
Certificate authentication
Key exchange
Session keys
```

Simplified:

```text
ClientHello
     ↓
ServerHello
     ↓
Certificate
     ↓
Key exchange
     ↓
Finished
     ↓
Encrypted data
```

The exact exchange depends on the chosen configuration and key exchange.

---

## 11. TLS 1.3

TLS 1.3 reduces handshake overhead compared with older TLS versions and removes many legacy cryptographic options.

Simplified:

```text
ClientHello
     ↓
ServerHello + handshake data
     ↓
Handshake completion
     ↓
Encrypted application data
```

---

## 12. HTTPS

**HTTPS = HTTP over TLS.**

Traditional HTTPS:

```text
HTTP
 ↓
TLS
 ↓
TCP
 ↓
IP
```

For HTTP/3:

```text
HTTP/3
 ↓
QUIC
 ↓
UDP
 ↓
IP
```

TLS is integrated into the QUIC connection.

---

## 13. SNI

**SNI (Server Name Indication)** allows the client to indicate the hostname it wants during TLS setup.

Example:

```text
Client
  ↓
SNI = www.example.com
  ↓
Server
```

This matters when multiple HTTPS domains share an IP:

```text
One IP
 ├── example.com
 ├── api.example.com
 └── shop.example.com
```

The server can select the appropriate certificate/configuration.

---

## 14. ALPN

**ALPN (Application-Layer Protocol Negotiation)** allows TLS to negotiate the application protocol.

Examples:

```text
h2        → HTTP/2
http/1.1  → HTTP/1.1
```

Conceptually:

```text
Client:
"I support h2 and HTTP/1.1"

Server:
"Use h2"
```

For HTTP/3, protocol negotiation is associated with QUIC/TLS and commonly identifies `h3`.

---

## 15. Practical: Inspect TLS

Run:

```powershell
curl.exe -v https://example.com
```

Example-style output:

```text
* Connected to example.com
* SSL connection using TLSv1.3
* Server certificate:
*  subject: ...
*  issuer: ...
```

Exact output depends on the curl version and server.

---

## 16. Practical: Test HTTPS

Run:

```powershell
curl.exe -I https://example.com
```

Example:

```text
HTTP/1.1 200 OK
Content-Type: text/html
```

The HTTP response is being obtained through a TLS-protected connection.

---

## 17. Opening an HTTPS URL

For:

```text
https://www.example.com
```

a simplified sequence is:

```text
1. Parse URL
       ↓
2. DNS lookup
       ↓
3. Get server IP
       ↓
4. Routing / next-hop resolution
       ↓
5. TCP handshake
       ↓
6. TLS handshake
       ↓
7. Certificate validation
       ↓
8. Secure session established
       ↓
9. HTTP request
       ↓
10. HTTP response
       ↓
11. Browser renders content
```

For HTTP/3:

```text
DNS
 ↓
UDP
 ↓
QUIC + TLS
 ↓
HTTP/3
```

---

## 18. Key Mental Model

```text
                 HTTPS
                   ↓
                  HTTP
                   ↓
                  TLS
          ┌────────┴────────┐
          ↓                 ↓
   Authentication      Encryption
          ↓                 ↓
   Certificate        Session keys
          ↓                 ↓
       Trusted        Encrypted data
        server
```

Core relationship:

```text
Certificate
    ↓
Authenticates server identity

Asymmetric cryptography
    ↓
Authentication / key establishment

Symmetric cryptography
    ↓
Efficient bulk encryption

Hash / signatures
    ↓
Integrity + authentication mechanisms
```
