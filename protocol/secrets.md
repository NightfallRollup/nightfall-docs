&larr; [Main](../README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &rarr; [Next](./transactions.md#transactions)

# Protocol
1. [Actors](./actors.md)
2. [Contracts](./contracts.md)
3. [Circuits](./circuits.md)
4. [MPC](./mpc.md)
5. [Keys](./keys.md)
6. [Commitments](./commitments.md)
7. [Nullifiers](./nullifiers.md#nullifiers)
8. [Secrets](#secrets)
9. [Transactions](./transactions.md)
10. [Fees](./fees.md#fees)
11. [Certificates](./certificates.md)
12. [Sanctions List](./certificates.md#sanctions-list)

# Secrets

To ensure a recipient receives the secret information required to spend the transmitted commitments, the sender
encrypts the secrets (*salt*, *value*, *tokenId*, *ercAddress*) of the commitment sent to the recipient and
proves using ZKP that they encrypted this correctly with the recipient's public key. The [**KEM-DEM**](https://eprint.iacr.org/2006/265.pdf) hybrid encryption paradigm is used.

Check [this](https://github.com/EYBlockchain/nightfall_3/blob/master/nightfall-client/src/services/kem-dem.mjs) for further details of the implementation.



