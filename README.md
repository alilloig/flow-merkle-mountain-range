# Merkle Mountain Ranges (MMRs)

## Introduction

A Merkle Mountain Range (MMR) is a data structure that extends the concept of a Merkle tree to allow for efficient appending of new elements and efficient proof generation. MMRs are particularly useful in blockchain and distributed systems where there's a need for verifiable append-only logs.

### Properties and Applications

1. **Append-Only**: Efficiently supports adding new elements
2. **Compact Proofs**: Proof size is logarithmic in the number of elements
3. **Verification Efficiency**: Proofs can be verified quickly
4. **No Rebalancing**: Unlike traditional Merkle trees, MMRs don't require rebalancing

## Basic Structure and Formation

In an MMR, nodes are arranged in a series of perfect binary trees (mountains) of decreasing height from left to right. 
When a piece of data is to be added to the MMR it gets hashed creating a node that using 1-based numbering and starting from the left is included in the MMR. Whenever a left node gets a sibling added, a parent node for both is created out of the hash of both child, following the sequentially numeration.

### Visualizing an MMR

Let's look at an MMR with 13 leaves (represented by positions 1, 2, 4, 5, 8, 9, 11, 12, 16, 17, 19, 20 and 23):

```
                               15
                              /  \
                             /    \
                            /      \
                           7        14          22
                          / \      /  \        /  \
                         /   \    /    \      /    \
                        3     6  10     13   18     21
                       / \   / \ / \   / \   / \   / \
                      1   2 4  5 8  9 11 12 16 17 19 20 23

```

## Key concepts

### Position

The number associated to each node is called node position. This numbering will be 1-based 
rather than 0-based as it will be on other data structs. While this may seem arbitrary
is a crucial aspect for the mathematical properties of MMRs allowing operations such as
obtaining the height of a node. In order to over-clarify this, we will be naming every
variable of property handling this numbering something with position in it. When we have 
to operate with this values 0-based for coding reasons, the auxiliary variables used
should be named using the index word.
(please someone that is a native english speaker write this last two sentence in a proper way)
An important note will be not to confuse this position with the order in which a piece 
of data was added to the MMR, that would be typically use to generate the node hashing 
this order along the data.

### Height and Node Types

Each node has an associated height depending on which level of the MMR they are placed.
Nodes on level 1 will always be **Leaf Nodes**, the ones containing the hashes of the actual data being logged.
The top nodes of each perfect binary tree (mountain) will be **Peak Nodes**. In our previous example peaks will be nodes 15, 22 and 23.
Given a node position, its height can be calculated by iterating through the tree.

### Size

The total amount of nodes that form the MMR

### Root

The unique identifier of a MMR at a certain point of its life, obtained by hashing together all the peaks (also known as bagging peaks).
(is size included in the calc of the root? some sources say so, im not sure what we are doing)

### Proof Path

The list of node positions needed for a certain position to be able to calculate the hash of the peak where that leave belongs
(needs accurate english writting!!) #naew?


## Proof Generation and Verification

### MMR Proofs

An MMR proof for a leaf allows verification that the leaf is part of the MMR without requiring the entire structure.

# ai generated and not reviewed yet from here
### Generating a Proof for Leaf at Position `p`

1. Start with the leaf position `p`
2. Calculate the sibling position:
   - If `p` is a left child (odd index of its level), sibling is at `p + 1`
   - If `p` is a right child (even index of its level), sibling is at `p - 1`
3. Calculate the parent position
4. Repeat steps 2-3 for the parent, moving up the tree
5. When you reach a peak, add that peak to the proof
6. Include all other peaks in the MMR

The proof consists of:
- The value of the leaf node
- The hash values of all siblings along the path from the leaf to its peak
- The hash values of all other peaks

### Proof Verification

To verify a proof for a leaf at position `p`:

1. Start with the leaf value and position
2. Calculate the hash of the leaf
3. Use the provided sibling hashes to calculate parent hashes moving up the tree
4. Reach the peak hash
5. Combine this peak with other provided peak hashes to calculate the MMR root
6. Compare the calculated root with the expected root

#### Detailed Verification Algorithm

1. Identify if the leaf is a left or right child
2. Combine with the appropriate sibling hash:
   ```
   if is_left_child(p):
       parent_hash = hash(leaf_hash + sibling_hash)
   else:
       parent_hash = hash(sibling_hash + leaf_hash)
   ```
3. Move up to the parent position
4. Repeat until reaching a peak
5. Combine all peaks using the "bagging the peaks" algorithm to calculate the final root

### "Bagging the Peaks" Algorithm

The peaks are combined from right to left:
1. Start with the rightmost peak
2. For each peak moving left:
   ```
   accumulator = hash(peak_hash + accumulator)
   ```
3. The final accumulator value is the MMR root

## Example

Let's walk through a full example with an MMR containing 5 leaves with values A, B, C, D, and E:

1. Leaf positions: 1, 2, 4, 5, 8
2. Internal nodes: 3, 6, 7
3. Peaks: 7, 8

MMR structure:
```
       7
      / \
     /   \
    3     6
   / \   / \
  1   2 4   5 8
```
### Proof for Leaf at Position 4 (value C)

1. Leaf 4 (C) is a left child
2. Its sibling is at position 5 (D)
3. Its parent is at position 6
4. Position 6's sibling is part of the path to position 3
5. Position 6's parent is at position 7 (a peak)
6. The other peak is at position 8 (E)

Proof components:
- Leaf value: C
- Sibling hashes: hash(D), hash from position 3
- Other peak hash: hash(E)

### Verification

1. Verify leaf 4 (C):
   - Calculate hash(C)
   - Combine with sibling hash(D) to get hash(C|D) for position 6
   - Combine with hash from position 3 to get hash(3|6) for position 7
   - Combine with hash(E) from position 8 to get the root hash
2. Compare the calculated root hash with the expected root hash

If they match, the proof is valid, confirming that leaf C is part of the MMR.

## Acknowledgments

Make sure to check the [CONTRIBUTORS](./CONTRIBUTORS) file for proper credit on this!