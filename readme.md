# Guide Outline

## Finished work
[01-what-is-fault-proof](https://github.com/joohhnnn/The-book-of-optimism-fault-proof/blob/main/01-what-is-fault-proof.md)

[02-fault-dispute-game](https://github.com/joohhnnn/The-book-of-optimism-fault-proof/blob/main/02-fault-dispute-game.md)

[03-Cannon](https://github.com/joohhnnn/The-book-of-optimism-fault-proof/blob/main/03-cannon.md)

[04-op-program](https://github.com/joohhnnn/The-book-of-optimism-fault-proof/blob/main/04-op-program.md)

[05-op-challenger](https://github.com/joohhnnn/The-book-of-optimism-fault-proof/blob/main/05-op-challenger.md)

## TL;DR

- Fault Proof: Replaces the original centralized proposer system.
- Fault Proof Game: Uses binary search to find the smallest instruction where consensus is reached between the parties, and verifies it on-chain.
- Cannon: An off-chain program used to generate on-chain move/step data.
- op-program: Provides the prototype ELF files for Cannon's use and the service for supplying Pre-Image data.
- op-challenger: Automates the operation of games by invoking Cannon and op-program.

## On-chain Addresses

- [MIPS](https://etherscan.io/address/0x0f8EdFbDdD3c0256A80AD8C0F2560B1807873C9c)
- [DisputeGameFactoryProxy](https://etherscan.io/address/0xe5965Ab5962eDc7477C8520243A95517CD252fA9)
- [Game](https://etherscan.io/address/0x68bf02b87d236c07e6cf4a7c851041745f45875c)(Example)
- [PreimageOracle](https://etherscan.io/address/0xD326E10B8186e90F4E2adc5c13a2d0C137ee8b34)
- [AnchorStateRegistryProxy](https://etherscan.io/address/0x18DAc71c228D1C32c99489B7323d441E1175e443)

## Reference Materials

### Articles
- [Public-OP-Stack-Fault-Proofs-Sherlock-Competition-Handbook](https://oplabs.notion.site/Public-OP-Stack-Fault-Proofs-Sherlock-Competition-Handbook-e4cfdf210a5c45c79230af19653163cc)
- [Docs](https://docs.optimism.io/stack/protocol/fault-proofs/explainer)
- [Specs](https://github.com/ethereum-optimism/specs/blob/main/specs/fault-proof/index.md)
- [Building a Fault Proof System worthy of the Superchain](https://blog.oplabs.co/building-a-fault-proof-system/)
- [Maximizing fault proof modularity with a composable pre-image oracle](https://blog.oplabs.co/composable-pre-image-oracle/)
- [Fault Proof Deep-Dive Part 1: MIPS.sol](https://blog.oplabs.co/mips-sol/)
- [Fault Proof Deep-Dive Part 2: Cannon](https://blog.oplabs.co/fault-proof-deep-dive-part-2-cannon/)
- [CANNON CANNON CANNON: Introducing Cannon](https://blog.oplabs.co/cannon-cannon-cannon-introducing-cannon/)
- [The game’s afoot: designing modular dispute games for the OP Stack’s Fault Proof System](https://blog.oplabs.co/dispute-games/)

### Videos
- [Walkthrough: OP Stack Fault Proof System alpha release](https://www.youtube.com/watch?v=nIN5sNc6nQM&t=727s)
- [Peek a live code-walkthrough of the new OP Stack fault proof code](https://www.youtube.com/watch?v=2n9qTmsdYbQ&t=801s) (Highly recommend)
- [Onchain Summit: Fault Proof Workshop by Protolambda](https://www.youtube.com/watch?v=RGts3PSg4F8)
- [Keys in Mordor Summit: Cannon & Fault Proofs](https://www.youtube.com/watch?v=ASLMj70V0Ao&t=3218s)