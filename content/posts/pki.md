---
title: "Public key infrastructure and digital certificates" 
date : "2023-11-11"
---

In this post, I will talk about public key cryptography, how encryption/decryption happens in asymmetric key cryptography, and how digital signatures and certificates are useful.

### Public key cryptography
* Also, known as asymmetric key cryptography. Asymmetric because there are 2 keys - public and private. 
* Public key is available to anyone in public. But the private key remains with the original owner. 
* Can be used to encrypt/decrypt data or to verify the authenticity of the data(digital signatures). 
* _The thing to note is that public/private can be used interchangeably (they are similar in that sense) i.e. data encrypted by one can be decrypted by another. For the sake of convention, in this post, we will use the terms public key for encryption and private key for decryption._
* RSA is an example of one such cryptosystem.

### Encryption and decryption
* In asymmetric key cryptography. The public key is available to the public. 
* The act of encrypting the data involves using some cryptographic algorithm with inputs as the public key and data being encrypted. 
* The end result is seemingly random-looking data. Obtaining original data from this random-looking data involves using the cryptographic algorithm with inputs as the private key and encrypted data.
* Since public keys are available to the public, the sender can encrypt the data which can only be decrypted by the receiver.

### Digital signatures
* Digital signatures are generally used to verify the integrity of the data. Practically speaking it follows, that party A generates some data known as a digital signature for the actual data. And when sending the actual data this digital signature is also sent with it.
Other parties now can actually verify that the received data is correct and is really sent by party A by using the digital signature.
* The way this works using public key cryptography is : 
    1. The sender generates the digital signature(can be thought of as a process similar to encrypting) using its private key (using some crypto algo).
    2. The receiver now has this digital signature of the data, the data and, the public key of the sender. We have already discussed that the public and private keys are interchangeable in that data encrypted by one can be decrypted using the other.
    3. Due to this receiver can retrieve the data using the sender's public key and the digital signature. If the sent data by the sender and retrieved data are the same, then the verification is successful, suggesting that the data was indeed sent by the sender.
* Practically, the same digital signature can be performed on the subset of the data (or even the hash of the data!) and we could still perform similar verification.

### Digital certificate
* This is an application of digital signatures. Used to establish the identity of the owner(of the certificate) and also its public key. Digital certificates will be associated with a pair of public and private keys. The server/owner will have the private key and the public key will be distributed along with the digital certificate.
* A question arises, with just this information in the digital certificate, can some malicious party pose as another entity using a forged digital certificate? i.e. we have yet to establish/verify that the public key really belongs to the said owner.
* To solve this problem we have issuers of the digital certificate, and the issuer itself has its public/private key pair. And their job is to verify the ownership and sign the digital certificate using their private key. This issuer's digital signature is also present in the digital certificate.
* To understand this end-to-end on how an end user can verify the digital cert, consider this flow :
    1. The user wants to verify the server A digital certificate. The certificate has server A public key, issuer A details, and issuer A digital signature.
    2. Assume that the user trusts issuer A and also has its public key. Then it can verify using the issuer's digital signature that the server A digital certificate was indeed signed by the issuer thereby confirming if the digital certificate is valid.
    3. It can be clearly seen here that a chain of trust is established, that is to verify the digital certificate of server A we have to verify/trust the issuer's digital signature.
* The issuer here acts as the *Certificate Authority(CA)*. And the job of a CA is to verify the identities and issue digital certificates for the entities. In our case, Issuer A was the CA for server A digital certificate. What this really means is that if you trust a CA then you automatically trust certificates signed/issued by this CA!
* *The process of signing a digital certificate by an issuer effectively means signing it using the issuer's private key or using the issuer's own digital certificate(both are the same technically, just different uses of language).* FYI, the Digital certificate owner is also known as subject in technical terms.
* X.509 is one of the standards for public key certificates.

### Digital certificate types
* Generally there are 3 types :
    1. *Root certificate* : 
        * These certificates are at the top of the chain of trust and therefore are self-signed (i.e. signed using their own private key).
        * These are also trust anchors because the trust for these is to be assumed and we cannot verify or derive the trust for these.
    2. *Intermediate certificate* : 
        * These are signed by the root certificate(same as root CA). e.g. issuer A in the above example. 
        * To verify intermediate certificates, clients will use root CA cert public keys.
    3. *Leaf certificate* : 
        * Signed by the intermediate certificate (same as intermediate CA). 
        * These are terminal nodes i.e. these cannot be used to issue new digital certificates. (The entity owning this is not a CA!)
        * The role of intermediate CA is to delegate/proxy the role of signing from root CA.

* _I am using CA and their digital certificates interchangeably here. Although the CA is the one actually owning its digital certificate (so technically not the same) the act of signing itself involves the CA digital certificate(or the private key)._

Till now we have discussed how subjects can acquire digital certificates(for their public keys) from the CA. And how those digital certificates can be verified by other parties by establishing the chain of trust till root CA. 

> _*It is apparent that the applications (e.g. web browsers) need to trust some CA certs for which verification cannot be derived (e.g root CA certs). To facilitate the validation of these operating systems and applications have something called trust stores containing the information about CAs they can trust!*_

Hopefully, this gave you an overview of Public key cryptography, digital signatures, digital certificates and Certification authority. These concepts are widely used. One notable application is TLS, but that is a story for a different post.
