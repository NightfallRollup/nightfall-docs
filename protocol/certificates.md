&larr; [Main](../README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &rarr; [Next](./sanctions-list.md#sanctions-list)

# Protocol
1. [Actors](./actors.md#actors)
2. [Contracts](./contracts.md#contracts)
3. [Circuits](./circuits.md#circuits)
4. [MPC](./mpc.md#mpc-ceremony)
5. [Keys](./keys.md#keys)
6. [Commitments](./commitments.md#commitments)
7. [Nullifiers](./nullifiers.md#nullifiers)
8. [Secrets](./secrets.md#secrets)
9. [Transactions](./transactions.md#transactions)
10. [Fees](./fees.md#fees)
11. [Certificates](#certificates)
12. [Sanctions List](./sanctions-list.md#sanctions-list)


# Certificates

Nightfall now incorporates the ability to manage a whitelist of accounts. When whitelisting is enabled, only accounts that are added to the whitelist are able to move funds from Layer 1 to Layer 2 and to withdraw Layer 1 funds from the Shield contract.

The contract that manages which accounts are whitelisted is [`X509.sol`](https://github.com/EYBlockchain/nightfall_3/blob/master/nightfall-deployer/contracts/X509.sol).

## Self Certification in Nightfall
Self cerficiation functionality has been added to Nightfall to assist with regulatory compliance requirements. A user that 
wants to perform a deposit or a withdrawal operation needs to have a valid credential to be whitelisted. At this point in time,
this credential is a valid X509 certificate from a set of accepted providers. 

Self certification is architected as a layer on top of Nightfall's whitelisting functionality, and it operates only when Whitelisting is enabled.

Different forms of self certification other than X509 may be added if required by using a different smart contract . The contract must extend the Whitelist manager and expose the [X509 interface](https://github.com/EYBlockchain/nightfall_3/blob/master/nightfall-deployer/contracts/X509Interface.sol).  It is the responsibility of the user to determine if this is adequate for their particular regulatory environment.

## X509
[`X509.sol`]((https://github.com/EYBlockchain/nightfall_3/blob/master/nightfall-deployer/contracts/X509.sol)) can check the validity of a passed-in X509 certificate (in DER format) against a root of trust that the contract knows (these can be added by the contract owner).  If the certificate is valid (in date, signature good, appropriate key usage, appropriate extensions, links to known trusted public key), then the owner of the certificate will be whitelisted. Note that only RSA cryptography is currently supported. A certificate chain can be added by passing in successive certificates until the end-user certificate is reached.

Of course, certificates are public, so we need to be sure that it's actually the certificate owner that passed it to the contract and not anyone else. To achieve that, we get the owner to sign their ethereum address with the private key that corresponds to their certificate. This must also be the address that passes in the certificate (all done as one transaction), so that front-running is prevented. The main contract API functions are:

`function validateCertificate(bytes calldata certificate, uint256 tlvLength, bytes calldata addressSignature, bool addAddress, uint256 oidGroup) external;`

This function takes in: a DER encoded X509 (RSA) certificate (`bytes calldata certificate`); the number of Tag-Length-Value items in the certificate (which can be calculated by calling `function computeNumberOfTlvs(certificate, 0)` and passing the certificate in); a signature over `msg.sender` using the private key corresponding to the certificate (PKCS #1 format); and a `bool` which indicates whether this is an intermediate certificate in a chain, or the end-user certificate, in which case `msg.sender` will be whitelisted following successful validation. Note that, if a chain of certificates is required, these should be passed in in order, with the first certificate being one that is signed by a key trusted by the smart contract.  Once that certificate is validated, its public key will be added to the list of trusted keys and the next certificate in the chain can be passed in, until finally the end-user certificate is reached. The function will reject any intermediate certificate which does not have the Certificate Sign key usage flag set and any end-user certificate which does not have the Digital Signature and Non-Repudiation key usage flags set. It will also reject any certificates that are not in-date or whose signature does not verify against a know trusted key. The variable `oidGroup` is explained below.

`function setUseageBitMaskEndUser(bytes1 _usageBitMask) external onlyOwner;`

This function can be used by the contract owner to change the key usage flags that are applied to an end-user certificate. The value should be computed by calculating the bitmask for each flag and reversing the bits after calculating to make it little endian. (DER flag encoding is somewhat complex).

`function setUseageBitMaskIntermediate(bytes1 _usageBitMask) external onlyOwner;`

As above, but for an intermediate certificate.

 `function setTrustedPublicKey( RSAPublicKey calldata trustedPublicKey, bytes32 keyIdentifier) external onlyOwner;`

 This function can be used to add a trusted public key `RSAPublicKey` is a struct `{ bytes modulus, uint256 exponent}`, along with an identifier for the key, which should normally be the same as the one in the corresponding certificate because that is used to look up a key to check a certificate's signature.

 `function x509Check(address user) external view returns (bool);`

 This is required by the KYC Interface and returns true, in the present case, if the user is whitelisted and their certificate is still in date and hasn't been revoked locally.

 `function revokeKey(bytes32 subjectKeyIdentifier) external;`

 This function allows the address that passed in a certificate to revoke its corresponding key. Note that only the owner of the certificate or the owner of the contract can do this. It's intended to be used in the event of a key compromise.

`function addExtendedKeyUsage(bytes32[] calldata oids) external onlyOwner;`

Generally, a key usage check is insufficient, certificates can have the same key usage but vary widely in terms of the amount of identity validation that the CA has engaged in. Thus we require specific extended key usages, to narrow down the exact type of certificate that we are dealing with. This function allows a set of extended key usage OIDs to be added and the smart contract will check that these are present. Each CA has its own key usage extensions as defined in its Certificate Practice Statement, so the correct group of OIDs to look for must be added separately for each CA being used. The correct group to check a certificate against is identified by the `oidGroup` variable above.  These are assigned by `X509.sol` in sequence i.e. the first group of OIDs to be added is assigned `oidGroup = 0` the next `1` and so on.

Note that these MUST be provided, certificates without any extended key usage will not validate.

`function addCertificatePolicies(bytes32[] calldata oids) external onlyOwner;`

This is similar to the previous function, and the same remarks apply, but it applies to the certificate policies extension rather than extended key usage.  These must be provided in the same order as the extended key usages (e.g. private both the extended key usage and certificate policies for CA 1 before moving on to CA 2) because they share the oidGroup. 

`function removeExtendedKeyUsage() external onlyOwner;`

Used to delete the extended key usage OIDS.  Note that it will delete them all and new ones must be re-applied before certificates can be validated.

`function removeCertificatePolicies() external onlyOwner;`

As for the previous function but applies to certificate policies.


### Adding X509 certificates

#### How to add a Certificate Authority and certificate chain to X509.sol

The owner of X509.sol is able to add Certificate Authorities (CAs) to the contract. Once a CA root of trust is established, it is possible to add intermediate CAs and end user certificates which take their trust from that root.

##### Adding a CA

We start with the Root Certificate of the CA, which can usually be downloaded from their website.  Most CAs have several so it is important to get the right one. We require the certificate to be in binary Destingished Encoding Rules format (DER). If it's in PEM format then it must be converted with a suitable utility (e.g. openssl):

```sh
openssl x509 -in /path/to/cert.pem -out /path/to/cert.der -outform DER 
```

Note that only RSA certificates are supported currently.

We need the public key modulus and exponent from this certificate, and the Subject Key Identifier. These can easily be obtained by printing the certificate, for example, with openssl:

```sh
openssl x509 -in /path/to/certificate.crt -noout -text -inform DER
```

Here is an example:

```sh
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 157588887309935488 (0x22fde4210a86380)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=GB, O=Nightfall, CN=Nightfall test root
        Validity
            Not Before: Oct 28 10:53:00 2022 GMT
            Not After : Oct 28 10:53:00 2032 GMT
        Subject: C=GB, O=Nightfall, CN=Nightfall test root
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c6:cd:ae:b4:4c:7b:8f:e6:97:a3:b8:a2:69:79:
                    91:76:07:8a:e3:cb:06:50:10:f5:5a:1f:1a:83:9f:
                    f2:03:b1:e7:85:d6:78:2e:b9:c0:4e:0e:1c:f6:3e:
                    c7:ef:21:c6:d3:20:1c:81:86:47:b8:ce:a4:76:11:
                    24:63:ca:a8:33:9f:03:e6:78:21:2f:02:14:c4:a5:
                    0d:e2:1c:ab:c8:00:1e:f2:69:ee:f4:93:0f:cd:1d:
                    d2:91:1b:a4:0d:50:5f:ce:e5:50:8b:d9:1a:79:aa:
                    dc:70:cc:33:c7:7b:e1:49:08:b1:c3:2f:88:0a:8b:
                    b8:e2:d8:63:83:8c:fa:6b:d4:44:c4:7d:d3:0f:78:
                    65:0c:af:1d:d9:47:ad:cf:48:b4:27:53:6d:29:42:
                    40:d4:03:35:ea:ee:5d:b3:13:99:b0:4b:38:93:93:
                    6c:c4:1c:04:60:2b:71:36:03:52:6a:1e:00:31:12:
                    bf:21:3e:6f:5a:99:83:0f:a8:21:78:33:40:c4:65:
                    97:e4:81:e1:ee:4c:0c:6b:3a:ca:32:62:8b:70:88:
                    6a:39:6d:73:75:37:bc:fa:e5:ba:51:df:d6:ad:d1:
                    72:8a:a6:bd:e5:ae:b8:c2:72:89:fb:8e:91:15:69:
                    a4:1c:3e:3f:48:b9:b2:67:1c:67:3f:aa:c7:f0:85:
                    a1:95
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier:
                EF:35:55:58:D6:FD:EE:0D:5D:02:A2:2D:07:8E:05:7B:74:64:4E:5F
            X509v3 Key Usage:
                Certificate Sign, CRL Sign
            Netscape Cert Type:
                SSL CA, S/MIME CA, Object Signing CA
            Netscape Comment:
                xca certificate
    Signature Algorithm: sha256WithRSAEncryption
         af:15:b8:6a:61:6a:da:03:7a:50:7a:da:48:65:f6:5b:95:54:
         a7:e6:ea:e6:9d:c7:22:e5:ed:51:ef:e5:dc:89:ba:a9:63:ea:
         ab:98:c0:c9:fb:72:cd:72:5a:b9:3a:bf:68:de:c2:12:b9:9c:
         94:da:ce:ea:46:57:9b:20:9f:8f:8e:ad:3c:b4:d6:01:18:42:
         6e:38:4e:ec:fc:38:f3:1a:9c:a5:de:b5:fe:d4:21:89:fe:98:
         53:6e:d9:8c:45:b4:91:4d:e3:d3:27:34:37:0a:84:c9:82:dc:
         3b:cf:60:b1:1c:88:bf:30:62:56:68:67:40:96:22:29:8d:8c:
         fc:65:63:ee:03:82:48:f1:5e:36:9e:f7:57:d6:5f:80:49:f0:
         b2:6e:b4:02:01:ba:b4:29:e6:00:23:1d:bc:0d:68:ee:a9:db:
         d9:87:2e:29:8a:6d:5d:fe:b2:96:4a:c6:8c:8e:73:69:e2:d6:
         cc:05:2c:62:40:4f:9f:68:95:05:cb:67:b6:ba:a0:e1:40:25:
         a8:dd:9b:6b:c0:e2:ee:2b:c2:a8:75:be:22:23:1b:0e:70:4d:
         9d:6b:6e:8c:90:f5:ab:62:1a:6b:dc:68:39:e9:bf:fc:1a:91:
         89:32:30:d5:b0:16:51:cb:db:57:da:1f:8a:7f:56:b8:b7:73:
         57:07:2c:64
```

You will need to do some editing to turn the key modulus and Subject Key Identifier into hex strings of the form `0x....`, with no carriage returns or colons. DO NOT miss out leading zeros; the smart contract treats these values as byte arrays.

These can then be added to the Smart contract using the setter:

```
function setTrustedPublicKey(
        RSAPublicKey calldata trustedPublicKey,
        bytes32 authorityKeyIdentifier) external onlyOwner;
```

`RSAPublicKey` has the form { bytes modulus, uint256 exponent}; `authorityKeyIdentifier` is the `Subject Key Indentifier` you've recovered from the certificate but padded to a bytes32 with leading zeros (we may remove the padding requirement in the future as it's superflous really). The latter is used as a lookup for the RSAPublicKey. It's renamed from Subject Key Identifier to Authority Key Identifier because that is how the next certificate in the chain will identify it.

##### Adding a certificate chain

Once the trust root is added, it's possible to add any certificate that uses the trust root as its Authority Key Identifier, provided it meets any other requirements for validity that `X509.sol` places on it (these are discussed in the next section).

Once _those_ certificate(s) are added, the next certificates down the certificate chain may be added and so on until an end user certificate is reached.

##### Specifying acceptable certificates

`X509.sol` does a number of checks for validity of a certificate:

1. The certificate's signature is valid
1. The certificate is in date
1. The certificate has `Key Usage` a set by the contract owner (cannot be omitted). Note that the settings can be different for and intermediate and end-user certificate
1. The `Extended Key Usage` is as set by the contract owner (cannot be ommitted but only checked if the certificate is an end-user one)
1. The `Certificate Policies` are as set by the contract owner (cannot be omitted but only checked if the certificate is an end-user one)

Note that default values for `Key Usage` are set during contract initialisation as `Certificate Sign, CRL Sign` for intermediate certificates and `Digital Signature` for end-user certificates. These should not normally be changed but there are setters in the contract for that purpose.

The values used for `Extended Key Usage` and `Certificate Policies` are unique to each CA and have to be obtained from the Certificate Practice Statement and list of Object Identifiers (OIDs) used by them. This allows tailoring of the level of checks that the owner of the certificate has to undergo, and hence the level of identity assurance that the certificate confers. Note that at least one OID MUST be chosed for each case, they cannot simply be omitted because such a certificate would be unlikely to confer meaningful assurance of identity. Note also that a separate group of OIDs must be specificed for each CA. See the information in [X509.md](./x509.md) about `oidGroups` for more information on how to add them.

The encoding of OIDs is a little tricky but there is a good utility [here](https://misc.daniel-marschall.de/asn.1/oid-converter/online.php). Use this to binary encode the OID and then right pad the result to 32 bytes.  So, for example

```sh
2.16.840.1.114412.3.21.2
```
becomes
```sh
0x060a6086480186fd6c0315020000000000000000000000000000000000000000
```

This is in the correct form to add to the contract.  