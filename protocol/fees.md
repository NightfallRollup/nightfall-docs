&larr; [Main](../README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &rarr; [Next](./certificates.md#certificates)

# Protocol
1. [Actors](./actors.md#actors)
2. [Contracts](./contracts.md#contracts)
3. [Circuits](#circuits)
4. [MPC](./mpc.md#mpc-ceremony)
5. [Keys](./keys.md#keys)
6. [Commitments](./commitments.md#commitments)
7. [Nullifiers](./nullifiers.md#nullifiers)
8. [Secrets](./secrets.md#secrets)
9. [Transactions](./transactions.md#transactions)
10. [Fees](#fees)
11. [Certificates](./certificates.md#certificates)
12. [Sanctions List](./certificates.md#sanctions-list)


# Fees

Proposer takes incoming transaction and rolls them into a L2 block in exchange for a fee. There are two different types of fees paid to the propose depending on the transaction:
- Deposits pay fees ETH directly in L1 or in a pre-defined fee token in L2. 
- Transfers and Withdrawals pay fees in a pre-defined fee token in L2.

Fee token is a standard ERC20 Token that has been predefined at contract deployment time as the token to pay fees.
