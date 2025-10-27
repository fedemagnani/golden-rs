# golden-rs

This is a mock implementation of [Golden: Lightweight Non-Interactive Distributed Key Generation](https://eprint.iacr.org/2025/1924.pdf)

> The Golden protocol is a one-round, non-interactive Distributed Key Generation (DKG) scheme designed for high efficiency. It introduces the Exponent Verifiable Random Function (eVRF), which builds on non-interactive key exchange to create publicly verifiable one-time pads. This enables participants to securely and deterministically encrypt and distribute Shamir secret shares using only standard discrete-log assumptions.

The implementation is mostly based on [commonware](https://github.com/commonwarexyz/monorepo) primitives

As a mock implementation, a core component missing is the zk-proof system (zkSNARK or Bulletproof, as recommended by the paper) through which each player in the DKG can verify the correctness of the cyphered shares broadcasted by the dealer. 

You can spawn any number of players, each connecting peer-to-peer. Every player acts as a dealer, broadcasting encrypted shares to all other participants. Using the Diffie-Hellman key exchange embedded in Golden-DKG’s eVRF, each player can decrypt their own share-limb. Once the DKG protocol completes, every player holds a share of the group’s private key, while the group public key remains publicly known. At this stage, to demonstrate threshold signature verification, players begin broadcasting partially signed “greetings” messages. Upon receiving `t = 2f + 1` such greetings, each peer aggregates the partial signatures into a full threshold signature and verifies it against the group public key. After verification, the player exits. 

The example below shows how to spawn 3 players: the first player will act as a bootstrapper letting the other two peers to discover each other (it might take a couple of seconds)

**Spawn the first player (bootstrapper)**
```rust
cargo run -- --log-level trace --worker-threads 5 --port 8545 --peer-index 0 --num-peers 3
```

**Spawn the second player**
```rust
cargo run -- --log-level trace --worker-threads 5 --port 8546 --peer-index 1 --num-peers 3 --bootstrapper 0@8545
```

**Spawn the third player**
```rust
cargo run  -- --log-level debug --worker-threads 5 --port 8547 --peer-index 2 --num-peers 3 --bootstrapper 0@8545
```