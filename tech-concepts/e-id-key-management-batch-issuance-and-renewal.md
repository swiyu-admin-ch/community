<p align ="right">CEA KRYPT, 10.06.2025<p>


<h1 align="center">e-ID<br>Key Management for e-ID - Batch Issuance and Renewal<br>V 1.0


<h2>Introduction</h2>

This document describes the cryptographic keys and their workflow related to the e-ID project. It also shows how the problem of traceability of persons can be solved when using ECDSA key pairs. 


<h2>Issuer key pair</h2>

The Swiss Confederation (Bund) generates and administrates its own key pair for issuance of e-ID verifiable credentials: the secret key $sk_{Bund}$ and the public key $pk_{Bund}$. It is an EdDSA key pair. $sk_{Bund}$ is generated and secured in a Hardware Secure Module (HSM). The public key $pk_{Bund}$ is published on the “Basisregister” of the trust infrastructure. This public key is required to verify the authenticity and integrity of the e-ID.

Issuer Key Pair:
| Secret Key (HSM) | Public Key | Key Type |
| :--------: | :--------: |:--------: |
| $sk_{Bund}$ | $pk_{Bund}$ | EdDSA on Ed448 |

Using the wallet application, the prover (i.e., the holder in the standard documentation) can access the issuer public key. This key is available in the “Basisregister”, and its address is contained in a standard SD-JWT data block (type `iss` for issuer). More precisely, it is a field contained in a standard verifiable credential defined by a “Decentralized Identifier (DID)” value. The integrity of this field is guaranteed by the chosen implementation of DIDs[^1]. 

Similarly, verifiers will use the same “Basisregister” to verify a verifiable credential of a holder, i.e., verifiers must be able to obtain the correct public key of the issuer. It is the verifier's responsibility to use the correct public key of the issuer.


<h2>Verifiable credentials (VC) and SD-JWT payload</h2>

Verifiable credentials (VCs) are containers constituted of data objects (claims), that are cryptographically hashed and signed. This allows holders to prove to a verifier, that data transferred is authentic and unaltered. In addition, key binding mechanisms (based on hardware or software) allow holders to prove possession of the associated private key and with its rightful ownership. There are a multitude of “flavours” of verifiable credentials. For the e-ID, SD-JWT is chosen as the initially supported standard. 

To understand how SD-JWTs support these capabilities, we first refer to the standard documents “*Selective Disclosure for JWTs (SD-JWT)*”[^2] :

>SD-JWT is based on an approach called “salted hashes”: For any data element that should be selectively disclosable, the Issuer of the SD-JWT does not include the cleartext of the data in the JSON-encoded payload of the JWS structure; instead, a hash digest of the data takes its place. For presentation to a Verifier, the Holder sends the signed payload along with the cleartext of those claims it wants to disclose. The Verifier can then compute the digest of the cleartext data and confirm it is included in the signed payload. To ensure that Verifiers cannot guess cleartext values of non-disclosed data elements, an additional salt value is used when creating the digest and sent along with the cleartext when disclosing it.<br><br>To prevent attacks in which an SD-JWT is presented to a Verifier without the Holder's consent, this specification additionally defines a mechanism for binding the SD-JWT to a key under the control of the Holder (Key Binding). When Key Binding is enforced, a Holder has to prove possession of a private key belonging to a public key contained in the SD-JWT itself. It usually does so by signing over a data structure containing transaction-specific data, herein defined as the Key Binding JWT. An SD-JWT with a Key Binding JWT is called SD-JWT+KB in this specification.

The issuer defines the personal data that is selectively disclosable. This data will be encoded in a JSON-Token (denoted $`SD\_JWT`$ in this paper), in a way that hides the content of the personal data (i.e., the claims). The eligible users (i.e., the prover or the holder) will use this $`SD\_JWT`$  on his mobile device. An $`SD\_JWT`$ payload contains the hashes of several claims concatenated with its salt value. The user can reveal the claims - and the associated salt - as he wishes. The sig-nature of the issuer is also part of the $`SD\_JWT`$. This signature consists of a signature of the hashes and other data such as the validity period or the identifier (DID) of the issuer. 


<h2>Renewal process</h2>

The electronic identities described in this document are very similar to the X.509 certificates used in standard TLS communication or by smartcards of employees of the Swiss Confederation. In both situations, a key renewal process has been defined. For TLS, this process works automatically using the standard ACME (Automated Certificate Management Environment) protocol, which is available from many recognized certification authorities such as Let's Encrypt. The server has to generate a new key pair when he starts the renewal process and can then request a new certificate. Note that the validity of TLS certificates has been greatly reduced over time (from 3 years by 2015 to 45 days by 2027) in order to reduce issues related to certificate revocation list (CRL). By the second example, an employee of the Swiss Confederation has to renew his smartcard after 3 years. Here the user smartcard has already a batch of key pairs that can be used to request new certificates. This batch of keys pairs has been generated by the central system and stored on the smartcard during the prestaging step. This re-newal process works only if the current authentication certificate is valid. 

For the e-ID, it would be possible that the system works without a renewal process, but in this case, it would require the user to repeat the entire identity verification and issuing process when their e-ID has technically expired. This disadvantage would force the project to carry out this renewal less frequently. Moreover, when using the same ECDSA key during a long period, it becomes easy to trace a user during that time. To eliminate these two drawbacks (too frequent repetition of the entire issuing process and tracking of users), we propose to introduce an automated key renewal process with a dedicated renewal key pair and the issuance of a batch of credentials to avoid tracking.

An e-ID has a validity period, so this payload must be renewed when it expires. Over time, the user will have to renew his SD_JWT, and we will denote these different data packages as follows:

$`SD\_JWT_1`$, $`SD\_JWT_2`$, $`SD\_JWT_3`$, ..., $`SD\_JWT_n`$


<h2>Holder key pairs and SD-JWT</h2>

To avoid theft of digital credentials as well as protecting against key extraction and transmissibility of the e-ID it is necessary to ensure cryptographic binding of the generated key pair to a hardware device. It is foreseen, that the e-ID will only be issued to devices where this binding is given. 

Each data package $`SD\_JWT_i`$ $(1≤ i≤n)$ will be associated to the following key material on the mobile device of the holder:

| Secret Key |	Public Key |	Key Type |	Key Attestation | SD-JWT |	Signature (of issuer)|
| :----: | :----: | :----: | :----: | :----: | :----: |
| $sk_i$ |	$pk_i$ |	ECDSA NIST p-256 |	$ka_i$ | $`SD\_JWT_i`$ |	$sig_i$|


**Remarks:**
- Each key pair $sk_i$/ $pk_i$ is an ECDSA key pair. Each secret key $sk_i$ is generated and secured by the secure enclave of the mobile device. 
- Key attestation $ka_i$ consists of a signature of $pk_i$. This signature has been generated by a secret key contained in the secure enclave. The corresponding public key is contained in a certificate that ensures the location of keys generated in the secure enclave. This certificate has been signed by a trustworthy Certification Authority, i.e., by a deemed trustworthy company (Apple, Samsung, Google, …). Be aware that the certificate associated with the secure enclave of the user's mobile device can be used to establish an unambiguous link with this device. Consequently, the key attestation should only be used by the issuer to verify the location of the keys, but it should be not integrated into a $`SD\_JWT`$ payload.
- Key pairs $sk_i$/ $pk_i$  of the holder are generated on their mobile device during the registration process (or the renewal process). 
- $`SD\_JWT_i`$ of the e-ID payload must contain:
    - the public key $pk_i$
    - a time stamp (`iat`)
    - validity period: `nbf` (Not Before) and `exp` (Expiration Time)
    - the identifier of the issuer (in a DID field)
    - the hashes of the claims concatenated with their salt values
- The signature $sig_i$ is done by Bund (issuer); it ensures the integrity of the $`SD\_JWT_i`$ payload. 
- By the creation of $`SD\_JWT_i`$, all the salts must be newly and randomly generated, i.e., these salt values must be all different with a very high probability
- Before this signature $sig_i$ is produced by the issuer, the issuer verifies that:
  - The holder has initiated the process
  - The holder is in possession of the private key $sk_i$  (e.g., by sending a self-signed request)
  - $ka_i$ is valid for $pk_i$
- Using a given $`SD\_JWT_i`$ many times, a given holder can be tracked since he will use the same $pk_i$, the same signature $sig_i$ and the same salted hashes. If one of these values is known, it is then possible to uniquely identify an $`SD\_JWT_i`$, i.e., these actions were carried out by the same person.
-  Technically, the issuer can store all $`SD\_JWT_i`$. With the corresponding $pk_i$ and collaborating with verifiers, they could track credentials issued by themselves. To avoid this, it would suffice to forbid the issuer to store the $`SD\_JWT_i`$ and any other information, that could be used for such a tracking, i.e., the public keys contained in the $`SD\_JWT_i`$ or the salt values. However, to ensure the e-ID issuers capability to revoke or support holders in cases of fraud or identity theft it is foreseen, that certain relevant data generated during issuance (e.g., the hash of the generated VC as well as the revocation index) will be stored. However, the e-ID issuer has no legal basis to correlate this data with verifiers.
<br>


**Regarding iOS:**

It is important to note, that iOS Devices currently do not support key attestation, but an alternative mechanism named “app attestation” is available. App attestation and key attestation are both methods to verify the integrity and trustworthiness of a device or application, but they differ in scope and security. <br>
**Key attestation** provides a high level of trust by verifying that a cryptographic key was generated and securely stored within a trusted hardware-backed environment (like a TPM or Secure Element) and has not been tampered with. This ensures that cryptographic operations are anchored to hardware security, making it the preferred and more robust choice. <br>
In contrast, **app attestation** focuses on verifying the integrity of the application itself, ensuring it hasn’t been modified or run in an untrusted environment. However, it offers weaker guarantees since it’s less tightly bound to hardware, and for it to be effective, the correct implementation of cryptographic functions must be manually assessed before granting attestation. While key attestation should be the default due to its stronger security posture, app attestation will serve as a fallback in scenarios where hardware-backed key attestation isn’t available. In the context of the e-ID, this means applications running on iOS will require certification by the Confederation before being granted attestation. In the case of the present concept, an app attesta-tion is used instead of a key attestation.

App attestation: On iOS devices, comparable security to key attestation on Android is achieved by using a combination of mechanisms provided by Apple:
- Initially, a dedicated key pair (for attestation) is generated in the Secure Enclave using a specific function provided by iOS[^3].
- This key pair is then certified as having a valid origin by a remote Apple server and receives an attestation proving that the key pair is rooted in Apple hardware and the utilized version of the app is uncompromised.
- This attestation can then be transmitted to a backend (e.g., app backend or the issuer) and allows independent verification through this service. If the verification of the attestation is successful, proof has been provided that the app is a valid instance, and the hardware-bound nature of the key is confirmed. A key identifier and the associated public key are contained in the attestation.
- Associating the extracted public key/key identifier with a holder allows the issuer to ensure future requests (e.g., during renewal) have been triggered by the same device used during initial issuance. Since the attestation of the key is in both cases (one-time issuance vs. batch issuance) created primarily for the app and not for the key pair included in the VC, security during renewal does not deteriorate compared to first-time issuance. The integrity of the app and proof of possession of the initial key are equivalently verified during renewal.
- The dedicated key pair is persistent over updates; however, it does not survive app re-installation, device migration, or restoration from a backup. In these cases, a new key pair will need to be generated, thus also requiring reissuance of the e-ID.
- It is recommended to solely use the app attestation key for the described conformance checks and to create specific hardware-bound key pairs for renewal and individual VCs. This allows the implementation of independent logic per key and enables dedicated re-newal keys per issuer. In addition, it is advisable to ensure wallets create a unique client attestation/key pair per issuer to avoid introducing new linkable identifiers.


<h2>Use case: presentation of an SD-JWT payload</h2>

When a holder wants to show to a service provider that he is in possession of a claim contained in his $`SD\_JWT_i`$, he must first sign a challenge from the service provider using his corresponding secret key $sk_i$. This proves, that he is in possession of the private key corresponding to the public key $pk_i$ contained in the $`SD\_JWT_i`$. Note: When the wallet of the holder is opened, the holder can use all his secret keys[^4]. 
For each selected claim $VC_j$  $(1≤ j≤m)$ contained in $`SD\_JWT_i`$, that the holder wants to reveal, he will provide the value and the associated salt to the verifier. In details, the holder sends the following objects to the verifier:
- $`SD\_JWT_i`$ (the public key $pk_i$ and the hash values of all claims are included in this payload)
- the signature $sig_i$ of $`SD\_JWT_i`$
- the values of the m selected claims and the salt values associated to these claims
- the signature of the challenge (signed with $sk_i$) and the signature $sig_i$ of $`SD\_JWT_i`$ (signed by the issuer)
The verifier will consider the credentials as valid if:
- The signature $sig_i$ is valid, i.e., the values in $`SD\_JWT_i`$ are valid and $pk_i$ belongs to the holder. This verification is computed using the public key of the issuer available on the “Basisregister”.
- $HASH(salt_j ||VC_j) = h_j$ for all $j=1,2,…,m$ (the hash values $h_j$ are contained in $`SD\_JWT_i`$)
- The signature of the challenge is valid using $pk_i$


<h2>Prevent tracking</h2>

The main drawback of the e-ID solution with ECDSA is the possible tracking of individuals by service providers: if the same $`SD\_JWT_i`$ is used several times, the service provider will know, that it is the same person since the same public key $pk_i$  and the same signature $sig_i$ have been used. If several verifiers would compare the $`SD\_JWT_i`$ they got, the user could even be tracked by one as well as over multiple service providers.
A simple solution to avoid this tracking is to generate, for a given holder, several key pairs (and so several $`SD\_JWT`$) for the same set of attributes and to use a key pair only once. But once all these public keys have been used, either the holder is blocked, or tracking becomes possible again, since he would be obliged to use a key more than once. It is important to note that the size of the secure enclave is limited and so only a limited number of keys can be stored. Consequently, the solution presented so far does not solve the problem of tracking in general. The solution is to renew the e-ID payloads more frequently and automatically. For security reasons, such a renewal should be done by using a dedicated key pair for each holder (see next section for details).
Moreover, claims involving temporal information, such as `iat`, `exp` and `nbf`, MUST either be randomized within a time frame considered appropriate (for example, randomize `iat` within the last 24 hours and calculate exp accordingly), or rounded (for example, rounded to the beginning of the day or the week).


<h2>Credential renewal using a dedicated key</h2>

To renew their e-ID, a holder must have access to a special key pair $sk_{renew}$ and $pk_{renew}$ in their mobile device and a dedicated payload $JWT_{renew}$. Both the keypair and the $JWT_{renew}$ are generated during the initial issuance of the e-ID by the holder. The special key pair and $JWT_{renew}$ will only be used to communicate with the issuer and will not be disclosed otherwise. Both will ensure the integrity of the exchanged data. The JWT_renew will be signed by the issuer. This signature will be denoted $sig_renew$. JWT_{renew}$ will not be selectively disclosable and solely contains its issuance date, the expiry date of the renewal key and a unique identifier simplifying key management for parties involved. 
The holder will use their renew key pair and JWT_renew to get a new $SD_{JWT}$ associated to a new key pair in their wallet. The payload of the new $SD_{JWT}$ will again contain hashes of their personal data combined with new salts.
Before issuing a new $SD_{JWT}$ to the holder, the issuer must make sure that:
- it can identify and authenticate the holder
- it has the personal data belonging to the holder of the key pair $pk_{renew}$/ $sk_{renew}$
- the key attestation of the new public key belongs to the same device as the key attestation of the $pk_{renew}$
- *if* key attestation is not available (e.g. iOS): the attestation provided is valid and confirms the version of the app is unaltered and the holder is still in possession of the app attestation key provided during initial issuance.
- the holder holds the private key $sk_{renew}$ associated to the public key $pk_{renew}$
- the holder holds the private key associated to the new public key; this new public key will be included in the new $SD_{JWT}$.

The implementation of the renewal protocol must ensure that no replay attack or other typical attacks are possible, for example by using a challenge for each renewal.
Observe that the validity of the renewal key will be defined in the $JWT_{renew}$ payload. This validity should in fact define the validity of an e-ID before a completely new generation is required. The dedicated renewal key will never be renewed. If it is lost or expires a new identity verification and issuing is required. 

Note that all the private keys of the holder mentioned in this document offer the same level of security, as they are protected in the same way by the mobile device.


<h2>General solution against tracking </h2>

To solve the problem of traceability (or linkability) with ECDSA effectively, it is therefore necessary to have several different key pairs for the same set of attributes encoded in multiple verifiable credentials, and to be able to renew them securely and automatically. Below, we present a way of managing ECDSA keys that prevents the holder from being tracked.
Let $d$ be the number of key pairs on the mobile device associated to the same verifiable credentials. The e-ID wallet should contain the following material: 

| Secret Key |	Public Key |	Key Type |	Key Attestation |	SD-JWT |	Signature (of issuer) |
| :----: | :----: | :----: | :----: | :----: | :----: |
| $sk_{i+1}$ |	$pk_{i+1}$ |	ECDSA NIST p-256 |	$ka_{i+1}$ |	$`SD\_{JWT}_{i+1}`$	| $sig_{i+1}$ |
| $sk_{i+2}$ |	$pk_{i+2}$ |	ECDSA NIST p-256 |	$ka_{i+2}$ |	$`SD\_{JWT}_{i+2}`$	| $sig_{i+2}$ |
| ... | ... | ... | ... | ... | ... |
| $sk_{i+d}$ |	$pk_{i+d}$ |	ECDSA NIST p-256 |	$ka_{i+d}$ |	$`SD\_{JWT}_{i+d}`$	| $sig_{i+d}$ |
| $sk_{renew}$ |	$pk_{renew}$ |	ECDSA NIST p-256 |	$ka_{renew}$ |	$JWT_{renew}$	| $sig_{renew}$ |

The $d$ first key pairs and associated $SD_{JWT}$ will be used to present standard verifiable credentials to a service provider. The values of the claims associated to the $`SD\_{JWT}_k`$ payloads $(k=i+1,i+2,…,i+d)$ should be all the same.  

To ensure renewal of the batch after a certain threshold of remaining SD-JWTs is reached, the wallet must automatically contact the issuer and use $JWT_{renew}$ as well as the $pk_{renew}$/ $sk_{renew}$ key pair to request a new batch of VCs. $JWT_{renew}$ allows to verify the validity of the e-ID and to know the identifier(s) associated with the user. Furthermore, as $JWT_{renew}$ is signed by the issuer, its content remains authentic. By using the key attestations or app attestations, the issuer can verify that all the keys, including the new key pairs, are in the same secure enclave (i.e., on the same mobile device). To ensure this property during renewal, $JWT_{renew}$ should be sent to the issuer with a proof of possession of $sk_{renew}$. Note that $ka_{i+1}$, …, $ka_{i+d}$ must also be sent to the issuer during renewal. 

With such a key management, it is possible to avoid tracking and use an ECDSA key pair only once. Even if it is not possible to renew keys often enough to ensure a single-use of credentials and associated keys, the fact that multiple key pairs are available greatly reduce the possibility of tracking people, for example by randomly rotating keys for each presentation. 
It is crucial for the issuer to ensure that the “issued at time” (`iat`) and expiration (`exp`) do not lead to new linkability. Thus, dates must be encoded with a generic time (e.g. 20260701 / 00:00:00) or must be randomized. 

From a privacy perspective, **single-use credentials** are the preferred approach, as they minimize the risk of linkability and tracking across multiple interactions. The proposal demonstrates that the use of a renewal key does not deteriorate security, as each new credential remains securely bound to the holder through the renewal key pair (**$pk_{renew}$/ $sk_{renew}$**). Since this key pair is exclusively shared between the **holder** and the **issuer**, its security properties remain intact over time, ensuring that the renewal process does not degrade the overall security of the system. This balance allows for enhanced privacy without compromising the integrity or trustworthiness of the credentials.




<br>
<h3> Authors </h3>

Dr. François Weissbaum <br> Kryptologe <br> Kdo Cy <br> Schweizer Armee

Andreas Frey Sang  <br> Verantwortlicher Produktstrategie  <br> Fachbereich e-ID  <br> Bundesamt für Justiz

Dr. Alexandra Höß <br> Verantwortliche Technologiestrategie <br> Fachbereich e-ID <br> Bundesamt für Justiz

Jonas Niestroj <br> System Architekt <br> Bundesamt für Informatik und Telekommunikation

<br>


[^1]: https://identity.foundation/didwebvh/v1.0/
[^2]: https://www.ietf.org/archive/id/draft-ietf-oauth-selective-disclosure-jwt-17.txt (page 3-4).
[^3]: DCAppAttestService.generateKey(completionHandler)
[^4]: Option: the holder must provide a PIN or a biometric mean each time he uses a secret key
