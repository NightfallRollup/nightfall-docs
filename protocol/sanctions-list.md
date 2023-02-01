&larr; [Main](../README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &rarr; [Tools](../tools/explorer.md#explorer)

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
11. [Certificates](./certificates.md#certificates)
12. [Sanctions List](#sanctions-list)


# Sanctions List
A related topic to whitelisting and self certification is that of Sanctions list. Nightfall allows to screen the sender address
agains Chainalysis santions screening oracle [contract](https://go.chainalysis.com/chainalysis-oracle-docs.html#:~:text=The%20Chainalysis%20oracle%20is%20a,included%20in%20a%20sanctions%20designation).

Although this check may arguably form part of the self-certification checkm it is not done via the X509 interface. This is because the sanctions contract already exposes its own interface, and manages its own blacklisting, thus there is nothing for Nightfall to do, other than to check the mapping held by the Chainalysis contract via the [`SanctionsListInterface.sol` interface](https://github.com/EYBlockchain/nightfall_3/blob/master/nightfall-deployer/contracts/SanctionsListInterface.sol).
