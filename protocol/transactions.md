&larr; [Main](../README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &rarr; [Next](./fees.md#fees)

# Protocol
1. [Actors](./actors.md)
2. [Contracts](./contracts.md)
3. [Circuits](./circuits.md)
4. [MPC](./mpc.md)
5. [Keys](./keys.md)
6. [Commitments](./commitments.md)
7. [Nullifiers](./nullifiers.md#nullifiers)
8. [Secrets](./secrets.md)
9. [Transactions](#transactions)
10. [Fees](./fees.md#fees)
11. [Certificates](./certificates.md)
12. [Sanctions List](./certificates.md#sanctions-list)

# Transactions

Nightfall transactions are defined [here](https://github.com/EYBlockchain/nightfall_3/blob/master/nightfall-deployer/contracts/Structures.sol#L45)

```
    // a struct representing a generic transaction, some of these data items
    // will hold default values for any specific tranaction, e.g. there are no
    // nullifiers for a Deposit transaction.
    struct Transaction {
        uint256 packedInfo;
        uint256[] historicRootBlockNumberL2;
        bytes32 tokenId;
        bytes32 ercAddress;
        bytes32 recipientAddress;
        bytes32[] commitments;
        bytes32[] nullifiers;
        bytes32[2] compressedSecrets;
        uint256[4] proof;
    }
```

Every transaction included in a block will be available as call data on L1.