---
layout: post
title: "Timber Merkle Tree: Structure, Optimization, and Verification"
date: 2026-09-04
---

The Timber Merkle Tree is an optimized variation of a Merkle tree designed to reduce on-chain storage while retaining verifiable integrity. It stores selected data on-chain and emits the remaining data as events for off-chain indexing.

This balances security and scalability: the chain keeps the data needed to commit to the tree, while an off-chain store can retain the nodes needed to construct proofs.

## What Is a Timber Merkle Tree?

A normal Merkle tree represents a collection of data as a single root hash. Each leaf is a hash of data, and every internal node is computed by hashing its two child nodes.

In a Timber Merkle Tree, not every node is persisted in contract storage. Nodes outside the on-chain frontier can be emitted as events and indexed off-chain, while the frontier and Merkle root remain on-chain.

## Key Components

### Leaf nodes

Leaves are hashes of the underlying data, such as transactions. In the diagram below, `L1`, `L3`, and `L4` are emitted as events for off-chain storage, while `L2` is retained on-chain as part of the frontier.

### Internal nodes

An internal node is calculated by concatenating and hashing two child nodes:

`Hash(1-2) = Hash(L1 || L2)`

Only the internal nodes on the frontier need to be stored in the contract. In this example, the left branch is retained on-chain while the `Hash(3-4)` branch is emitted for off-chain indexing.

### Frontier

The frontier is the set of nodes the contract stores so it can efficiently update the tree as new leaves are added. Keeping this compact set on-chain reduces storage costs compared with persisting every leaf and every intermediate hash.

### Merkle root

The Merkle root is stored on-chain and commits to the full tree. It is the cryptographic anchor that lets a verifier check whether off-chain data belongs to the committed structure.

## Structure

![Timber Merkle Tree showing on-chain frontier nodes and off-chain emitted nodes]({{ '/assets/images/timber-merkle-tree-frontier.png' | relative_url }})

In this example:

- `L1`, `L3`, `L4`, `Hash(3-4)`, and its underlying off-frontier nodes are emitted and indexed off-chain.
- `L2`, the relevant on-chain hashes, and the Merkle root are retained by the contract.
- The on-chain root continues to commit to both the on-chain and off-chain branches.

## Proving That a Leaf Belongs to the Tree

A Merkle proof lets someone prove that a leaf belongs to a known Merkle tree without revealing the entire dataset.

Assume we want to prove that `L3` is part of the tree.

### 1. Collect proof components

The verifier needs:

- The hash of `L3`.
- The sibling hash `L4`.
- The sibling subtree hash `Hash(1-2)` from the on-chain frontier.
- The on-chain Merkle root to compare against the reconstructed result.

### 2. Retrieve off-chain data

`L3`, `L4`, and `Hash(3-4)` are available from the events emitted when those nodes were added. An indexer or database such as MongoDB can retrieve this data.

### 3. Reconstruct the branch

First compute the right-side internal node:

`Hash(3-4) = Hash(L3 || L4)`

Then combine it with the on-chain sibling subtree:

`Merkle Root = Hash(Hash(1-2) || Hash(3-4))`

### 4. Verify the root

Compare the reconstructed root with the root stored by the contract. If they match, `L3` belongs to the tree committed on-chain.

The left/right ordering of each sibling is part of the proof: changing the order changes the hash and should cause verification to fail.

## Why Use This Design?

### Gas efficiency

Storing only the frontier and root on-chain reduces storage writes and therefore gas costs.

### Scalability

As the tree grows, the contract updates a limited on-chain state while off-chain services index emitted nodes. This is useful when the application needs to manage a large number of leaves.

### Verifiable off-chain data

Even though many nodes live off-chain, they remain accountable to the root stored on-chain. A valid proof must still reconstruct that root.

## Conclusion

The Timber Merkle Tree is a practical pattern for applications that need both efficient on-chain state and verifiable off-chain data. By retaining the Merkle root and a compact frontier in the contract, while emitting the rest of the structure as events, it can lower storage costs without giving up cryptographic verification.
