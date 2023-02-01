&larr; [Main](../README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &rarr; [Next](./keys.md#keys)

# Protocol
1. [Actors](./actors.md)
2. [Contracts](./contracts.md)
3. [Circuits](./circuits.md)
4. [MPC](#mpc)
5. [Keys](./keys.md)
6. [Commitments](./commitments.md)
7. [Nullifiers](./nullifiers.md#nullifiers)
8. [Secrets](./secrets.md)
9. [Transactions](./transactions.md)
10. [Fees](./fees.md#fees)
11. [Certificates](./certificates.md)
12. [Sanctions List](./certificates.md#sanctions-list)

# MPC Ceremony
Zero-knowledge proofs require a trusted setup. This setup generates a so-called "toxic waste" which could potentially allow to create fake proofs. To avoid this, a ceremony needs to be held where this setup is generated via multi-party computation (MPC).

Nightfall provides a [repository](https://github.com/NightfallRollup/nightfall-phase2ceremony) to enable a Multi-Party Computation (MPC) following the same principles of the Perpetual Powers of Tau Ceremony. 

