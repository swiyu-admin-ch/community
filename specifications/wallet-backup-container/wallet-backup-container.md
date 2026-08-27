# Wallet Backup Container

Status: Draft<br/>
Version: 0.4<br/>

## Introduction

This specification defines a flexible and secure backup container designed for digital identity wallets.

## File format

The backup file is identified by the file extension `wbak`.

The backup file **MUST** be a zip file which **SHOULD** contain the following files:

| File                      | Required  | Description                                                                                |
|---------------------------|-----------|--------------------------------------------------------------------------------------------|
| meta.json                 | Mandatory | File containing metadata information about the backup file.                                |
| container_encryption.json | Optional  | File containing additional information about the encryption of the backup container files. |
| wbak-*.json               | Optional	| Files containing a JSON data structure.                                                    |
| wbak-*.jwe                | Optional	| Files containing a encrypted JSON data structure encoded as JWE.                           |

- The wildcard of the `wbak-*` files  **MUST** be interpreted as numerical increment to differentiate all files. the increment starts at 0.
- The file order **MUST NOT** be relevant for the recovery process.

<br/>

Example content of a `wbak` file:

```
meta.json
container_encryption.json
wbak-0.jwe
wbak-1.jwe
wbak-2.jwe
```

## meta.json  - Metadata file

The Metadata file **MUST** contain a JSON data structure with the following content:  

- `type`: Version of the Wallet Backup Container specification and **MUST** be `WalletBackupContainerV1`.
- `creationDate`: Creation date of the backup, **MUST** be defined and **MUST** use the [ISO 8601 date format](https://www.iso.org/iso-8601-date-and-time-format.
html).
- `profile_version`: VC portability can only be guaranteed when the importing wallet is supporting the version given by this value

<br/>

Example of `meta.json` file:

```json
{
  "type": "WalletBackupContainerV1",
  "creationDate": "2025-01-01T15:50+00Z",
  "profile_version": "swiss-profile-portability:1.0.0"
}
```

## container_encryption.json - Additional container encryption information

Contains a map of per-file salts recommended for the generation of the encryption keys, when using a KDF(Key Derivation Function).

If the encryption key is generated with a KDF and the input has less than 128 bits of entropy (e.g. Pin, Password), the Encryption file **SHOULD** be defined. The file **MUST** contain a JSON data structure with the following content:

- `type`: Version of the Encryption Container specification and **MUST** be `ContainerEncryptionContainerV1`.
- `salts`: Map of encrypted file names and their respective salt values. Each salt **MUST** be unique, have a minimum length of 256 bits, be encoded in [base64url](https://datatracker.ietf.org/doc/html/rfc4648) and **MUST** be appended to the low entropy KDF input.

<br/>

Example of `container_encryption.json` file:

```json
{
  "type": "ContainerEncryptionContainerV1",
  "salts": {
    "wbak-0.jwe": "-zNe8UtdtuGWMD25pVZoPphywDGZeOqpso8f3Z5q7VE",
    "wbak-1.jwe": "JGKJ6lhODH0bpgGhGzkopHA-unxJulRe5UqN02HsLMM",
    "wbak-2.jwe": "FUE0zlMvqOPK7REPhhGu_6E7-P01zPUWTGO9c2RRiWg",
  }
}
```

## wbak-* - Wallet Backup Container files
The Wallet Backup Container files **MUST** either finish with:

- `json` file extension, when the file is unencrypted and contains a JSON data structure.
- `jwe` file extension, when the file's content is encrypted and encoded in the [JWE format](https://datatracker.ietf.org/doc/html/rfc7516). When decrypted, it **MUST** contains a JSON data structure.


The JSON data structure **MUST** contain:

- `type`: Type of the wallet backup container. Each type defines the additional content present in their specific JSON data structure.

<br/>

Example of `wbak-0.json` file:

```json
{
  "type": "VerifiableCredentialContainerV1"
  ...
}
```

## Container Types

### VerifiableCredentialContainerV1
Container specified with the property type `VerifiableCredentialContainerV1` **MUST** include the following property:

- `vcs`: Array of verifiable credential batches, optionally with their holder binding key.
    - `id`: Unique identifier
    - `format`: Format of the verifiable credential. Format **SHOULD** be part of the [Credential Format Profile from OpenID4VCI](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html#name-credential-format-profiles) specification.
    - `credentials`: Array of VC batch items.
        - `vc`: Verifiable Credential encoded in Base64URL.
        - `jwk`: (optional) private software holder binding key in [JWK format (RFC7517)](https://datatracker.ietf.org/doc/html/rfc7517) necessary to create holder binding proofs.

In case of a hardware-bound batch credential, only a single VC batch item **MUST** be exported.

<br/>

Example of a `VerifiableCredentialContainerV1`  container:

```json
{
  "type": "VerifiableCredentialContainerV1",
  "vcs": [
    {
      "id": "270f003b-ea23-4819-b9ec-bf1f238859a3", // credential with no key binding, only one credential in credential-batch
      "format": "vc+sd-jwt",
      "credentials": [
        {
          "vc": "..."
        }
      ]
    },
    {
      "id": "ffbf6101-dee3-48d2-83c3-e625339660de", // hardware bound credential, therefore no exported holder binding jwk
      "format": "vc+sd-jwt",
      "credentials": [
        {
          "vc": "..."
        }
      ]
    },
    {
      "id": "3c6b9350-4f67-41bc-8db5-c9603c947e1d", // software bound credential with holder binding key, multiple credential in credential-batch
      "format": "vc+sd-jwt",
      "credentials": [
        {
          "vc": "...",
          "jwk": {
            ...
          }
        },
        {
          "vc": "...",
          "jwk": {
            ...
          }
        }
      ]
    }
  ]
}
```

### CredentialAuthenticationContainerV1
Container specified with the property type `CredentialAuthenticationContainerV1` **MUST** include the following property:

- `credentialAuthentications`: Array of credential authentications with their associated verifiable credential identifier.
    - `vcId`: Unique identifier of the associated verifiable credential.
    - `refreshToken`
    - `tokenType`: **MUST** be either `bearer` or `dpop`.
    - `dpop`: (optional) DPoP binding. **MUST** be present when `tokenType` is `dpop`.
        - `jwk`: Software-bound private key

A credential is treated as refreshable, when it has an authentication entry in this container.

<br/>

Example of a `CredentialAuthenticationContainerV1` container:

```json
{
  "type": "CredentialAuthenticationContainerV1",
  "credentialAuthentications": [
    {
      "vcId": "270f003b-ea23-4819-b9ec-bf1f238859a3",
      "refreshToken": "...",
      "tokenType": "bearer"
    },
    {
      "vcId": "3c6b9350-4f67-41bc-8db5-c9603c947e1d",
      "refreshToken": "...",
      "tokenType": "dpop",
      "dpop": {
        "jwk": {
          ...
        }
      }
    }
  ]
}
```

### OIDIssuerMetadataContainerV1
Container specified with the property type `OIDIssuerMetadataContainerV1` **MUST** include the following property:

- `metadata`: Array of OID metadata with their associated verifiable credential identifier.
    - `vcId`: Unique identifier of the associated verifiable credential
    - `data`: OID Issuer Metadata encoded in Base64URL.

<br/>

Example of a `OIDMetadataContainerV1` container:


```json
{
  "type": "OIDIssuerMetadataContainerV1"
  "metadata": [
    {
       "vcId": "270f003b-ea23-4819-b9ec-bf1f238859a3",
       "data": "...."
    },
    {
       "vcId": "ffbf6101-dee3-48d2-83c3-e625339660de",
       "data": "...."
    },
    {
       "vcId": "3c6b9350-4f67-41bc-8db5-c9603c947e1d",
       "data": "...."
    },
  ]
}
```

### OCAContainerV1
Container specified with the property type `OCAContainerV1` **MUST** include the following property:

- `metadata`: Array of OCA metadata with their associated verifiable credential identifier.
    - `vcId`: Unique identifier of the associated verifiable credential
    - `data`: OCA encoded in Base64URL.

<br/>

Example of a `OCAContainerV1` container

```json
{
  "type": "OCAContainerV1 "
  "metadata": [
    {
       "vcId": "270f003b-ea23-4819-b9ec-bf1f238859a3",
       "data": "...."
    },
    {
       "vcId": "ffbf6101-dee3-48d2-83c3-e625339660de",
       "data": "...."
    },
    {
       "vcId": "3c6b9350-4f67-41bc-8db5-c9603c947e1d",
       "data": "...."
    }
  ]
}
```

## References

**ISO8601**<br/>
https://www.iso.org/iso-8601-date-and-time-format.html

**OpenID4VCI credential format profiles**<br/>
https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html#name-credential-format-profiles

**RFC4648**<br/>
https://datatracker.ietf.org/doc/html/rfc4648

**RFC7516**<br/>
https://datatracker.ietf.org/doc/html/rfc7516

**RFC7517**<br/>
https://datatracker.ietf.org/doc/html/rfc7517
