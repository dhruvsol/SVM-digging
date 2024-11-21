# Solana Gossip Protocol

The Solana gossip protocol serves two primary functions:

1. Data Storage
2. Message Communication (Send/Receive)

## Data Storage Architecture

### Gossip Tables Overview

Solana implements gossip tables for data storage, known as Cluster Replicated Data Store in Agave client. The architecture consists of three main components:

1. **GossipData** (CrdsData in Agave)

   - An enum covering various data types

2. **SignedGossipData** (CrdsValue in Agave)

   - A struct containing:
     - Signature
     - GossipData

3. **GossipTable** (Crds in Agave)
   - Implemented as an IndexMap where:
     - Key: `GossipKey` (CrdsValueLabel in Agave)
     - Value: `GossipVersionedData` (VersionedCrdsValue in Agave)

### GossipKey Structure

GossipKey, implemented as an enum, defines the data storage structure. Current key variants include:

```rust
#[derive(PartialEq, Hash, Eq, Clone, Debug)]
pub enum CrdsValueLabel {
    LegacyContactInfo(Pubkey),
    Vote(VoteIndex, Pubkey),
    LowestSlot(Pubkey),
    LegacySnapshotHashes(Pubkey),
    EpochSlots(EpochSlotsIndex, Pubkey),
    AccountsHashes(Pubkey),
    LegacyVersion(Pubkey),
    Version(Pubkey),
    NodeInstance(Pubkey),
    DuplicateShred(DuplicateShredIndex, Pubkey),
    SnapshotHashes(Pubkey),
    ContactInfo(Pubkey),
    RestartLastVotedForkSlots(Pubkey),
    RestartHeaviestFork(Pubkey),
}
```

### Data Management

- `GossipVersionedData` includes `SignedGossipData` along with additional metadata
- A cursor is used to track and read new data
- Old data is periodically trimmed based on timestamps to maintain a maximum number of pubkeys

## Message Communication

### Message Processing

Network messages are received as raw bytes and must be:

1. Deserialized
2. Signature verified

### Message Types

The protocol supports four main message types:

1. Pull
2. Push
3. Prune
4. Ping/Pong

### Pull Message System

#### Pull Request Structure

- Includes a bloom filter representing `SignedGossipData` stored in the node's GossipTable
- Uses multiple bloom filters based on N bytes of `SignedGossipData`
- Example: With 3 bits of hash, 8 bloom filters would be used (2^N)
- Similar to discriminator usage in Anchor accounts

#### Pull Response Mechanism

- Checks missing bloom filters from the original request
- Uses `GossipTableShards` (CrdsShards in Agave) to locate filters
- Sends missing data in response## GossipTableShards ( CrdsShards in agave )

## GossipTableShards

todo





--------------------------- Solana Spotlight Gossip Protocol Notes -------------------

### PAS ( Push Active Set )
Vec of 25 nodes 
PAS set is decided by Stake Bucket which means Log2 of node's stake // Might have been changed  
#### PASE ( Push Active Set Entry )
IndexMap<Pubkey, Bloom<Origins>>  // Using the bloom filter to quickly check the value

Pruning maintains 2 IndexMap of origin list 
1. Act as we can send messages to the origin 
2. Act as we should not send messages to this origin

Pruning happens based on the origin
