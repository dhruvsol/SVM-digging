# Gossip Protocol Solana

Gossip in solana does 2 things

- Store Data
- Send/Receive Request

## Storing Data

### Gossip Tables

- Solana's uses gossip tables to store data in agave client they are callend ( Cluster Replicated Data Store ).

Main Sections

- `GossipData` # enum covering various data types ( CrdsData in agave )
- `SignedGossipData` # Struct contains signature & GossipData ( CrdsValue in agave )
- `GossipTable` # Data store ( Crds in agave )

GossipTable is a IndexMap where the key is `GossipKey` ( CrdsValueLabel in agave ) and value is `GissupVersoinedData` ( VersionedCrdsValue in agave )

`GossipKey` defines how data is stored, since its a enum

The Currnt Keys are following

```

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

`GossipVersionedData` has `SignedGossipData` inserted with other metadata.

cursor is use to read the new data

### Removing old data

Old data is preiodically trimmed to maintain max number of pubkeys, based on the timestamp

# Sending / Receiving Request

we receives raw bytes of messages from the network, so have to deserlialize and verify sig over them.

### Message types over gossip

we have 4 main types of request we can receive from gossip

- Pull
- Push
- Prune
- Ping/Pong

## Pull Messages

we have 2 types of Pull messages that we can receive - PullRequest - PullResponse

### Building Pull Request

A pull request includes a bloom filter over the valuess stored in that nodes GossipTable to represet the `SignedGossipData`, so at the time of reciving the data the node can just look at
the bloom filter can parse to get missing data and send them to the requesting node's

Bloom filter is not one Large filter it has multiple of them based on N bytes of `SignedGossipData`.

For example if we have 3 bits of hash we could use 8 bloom filter ( 2^N ) # This seems similar to descriminator we use in anchor accounts

### Building Pull Responses

Just checks the missing bloom filter from the original request and sends that in response.
To find the filter we use `GossipTableShards`

## GossipTableShards ( CrdsShards in agave )
