
&larr; [Main](../README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &rarr; [Next](./contracts.md#contracts)

# Protocol
1. [Actors](#actors)
2. [Contracts](./contracts.md#contracts)
3. [Circuits](./circuits.md#circuits)
4. [MPC](./mpc.md#mpc-ceremony)
5. [Keys](./keys.md#keys)
6. [Commitments](./commitments.md#commitments)
7. [Nullifiers](./nullifiersmd#nullifiers)
8. [Secrets](./secrets.md#secrets)
9. [Transactions](./transactions.md#transactions)
10. [Fees](./fees.md#fees)
11. [Certificates](./certificates.md#certificates)
12. [Sanctions List](./certificates.md#sanctions-list)


# Actors
There are three main actors involved in the network:

- **Transactor -** is a regular customer of the service. They wish to make transactions, i.e. Deposit, Transfer and Withdraw privately.  These customers typically will use a web wallet or a dedicated server (Client) to perform the ZK Proofs required to generate the transactions.  

- **Proposer -**  collects the transactions from transactors and proposes new updates to Nightfall's state contract by submitting new Blocks.  Blocks contain several transactions, rolled up into a Layer 2 Block to gain scalability.  Anyone can become a Proposer, but they must post some stake intended to incentivize good behavior.  Proposers make money by providing correct Blocks, collecting fees from transactors. They are somewhat analogous to Miners in a conventional blockchain.

- **Challenger -** oversees the correctness of blocks proposed by Block Proposers within one week after the block was submitted. Anyone can be a Challenger. Challengers take the stake posted by proposers when building their blocks when they successfully submit a challenge.


## Notes
In this reference implementation, both the Proposer and the Challenger offload some functionality to one common module called Optimist.  This Optimist module provides some services to a number of Proposers and Challengers, such as generating and challenging blocks (Proposer and Challengers would need to sign these transactions), synchronizing with blockchain events, etc.

Apart from the Actors described above, there is an additional actor called Client. A Client acts as a trusted service that collects user transactions, performs the ZK Proofs on their behalf, sends transactions to the Proposer, listens to Blockchain events etc. In summary, the Client acts as a trusted relayer for a collection of users that want to offload heavy proof computation and that trust each other.

The alternative to the Client is to use the browser wallet. This wallet manages all transaction activities for a single user while maintaining privacy.

