# Monero v7 Mining Pool Report

May 2018

by Josue Sneur <jsneur@gmail.com> (https://github.com/jsneur)

## Abstract

This report applies to Monero mainnet version 7 (`v7`) and showcases degraded privacy due to publicly-available metadata: mining pools' announced blocks and transactions.  Simple practices are presented which allow users and mining pools to proactively maintain privacy without disclosing less metadata.  Submissions are made to the blackball database and sample code is presented for scraping known spent outputs.

###### Formatting note

This report's final format will be LaTeX (see `monero-v7-mining-pool-report.tex` and its PDF output, `monero-v7-mining-pool-report.pdf`,) but this README is in Markdown format.  Footnotes, for example, are rendered here as simple parentheticals.  The report's text will be refined and finished here in Markdown format first and migrated to LaTeX in its final versions for presentation.

## Introduction

As of Monero version 7 (`v7`, which began at block height 1546000,) a majority of the Monero hashrate is attributable to particular mining pools.  There are at least seven or eight public pools with over 1% of the total global hashrate.  These pools advertise various statistics--metadata--including the blocks that they have mined.  All but one pool also list the payouts that they have made to their miners.  The combination of output ownership and transaction authorship allow the true member of some ring signatures to be inferred beyond a reasonable doubt in some cases, degrading one of Monero's layers of privacy.

### Monero's privacy layers

Monero provides stealth addresses, ring signatures, and Ring Confidential Transactions (RingCT) as privacy layers as of `v7`.  These layers ensure that Monero transactions are unlinkable, untraceable, and opaque.  Stealth addresses provide unlinkability by encrypting the recipients of payments such that only the recipient of a Monero transaction can detect it as addressed to them, ring signatures provide untraceability by concealing the source of a payment such that any one of a number of ring members should be equally-plausible as the actual source of a Monero transaction, and Ring Confidential Transactions provide opaqueness by encrypting a payment's input and output amounts such that a third party cannot discern anything related to a transaction's amounts other than that it did not create new coins.

### Degraded untraceability of ring signatures due to poor mining pool practices

Ring signatures should conceal the real source of a Monero transaction: there is no way to discern which ring member actually made the transaction without additional information.  Unfortunately, metadata announcements due to poor mining pool practices such as routine statistics advertisements can provide enough additional information to discern which ring member is the actual source of a transaction.

Mining pools were inadvertently revealing the real source of some payments *via* the statistics that they advertise.  Mining pools announce their blocks.  When they later announce a transaction that uses one of their blocks' outputs as a ring signature member, it is possible to deduce which is the real one spent.

By identifying an output as real in one transaction, its suitability as a decoy is degraded elsewhere, reducing the effective ring size of other users' ring signatures when included in rings after they are revealed to be known spent.

### Monero version 7's countermeasures against mining pool privacy degradations

Several tools were provided by the Monero `v7` upgrade that allow users to avoid or mitigate of the above potential degradations to their privacy outlined previously.  For example, users can avoid including any known spent outputs in their own ring signatures by using what is known as the blackball database, which contains every known spent output.  This report presents additional submissions to the blackball database, including explanations of what metadata identifies the real member of a ring, where to collect it, and example code to scrape and analyze the metadata necessary for independent verification of these results.

### A note on key image reuse and 0-decoy transactions

The impact of reckless key image reuse across blockchains and 0-decoy (0-mixin) transactions negatively impacts the privacy provided by Monero's ring signatures, and can be used in coordination of the known mining pool outputs to create a larger impact on these ring signatures. Monero noted the issues with 0-decoy transactions in January 2015 with [MRL-0004](https://lab.getmonero.org/pubs/MRL-0004.pdf), and 0-decoy transactions have been prohitied on the network since March 2016. They are no longer a concern with transactions going forward.

Blockchain forks have the potential to degrade the untraceability of ring signatures when key images are reused on both sides of a blockchain fork without taking advantage of any of the tools provided by Monero's `v7` upgrade.  Key images may be safely (at least privately) reused across blockchain forks if care is taken to construct identical rings on both sides of the fork (a process which is outside of the scope of this report but is described in detail [here](https://monero.stackexchange.com/questions/7826/how-can-individuals-safeguard-themselves-and-the-community-against-a-key-reusing).)  If users send funds on both sids of the fork without any such precautions, however, then they will inadvertently produce two rings that share only one member in common, thus identifying the common member as the real source of both transactions.  Such outputs are identified as "known spent."

One must note that not all forks will include the opportunity to reuse ring members. Suppose a imposes a restriction to only allow 0-decoy transactions. There is no non-"reckless" way of claiming funds on such a fork; any claim degrades the privacy of Monero by rendering the associated ring signature useless and impacting transactions that use this output as a decoy. However, even if this is not the case, users can still have their privacy impacted when reusing the same ring member set. If the ring member mitigation tool is not widely used, the real output can be revealed, removing the effectiveness of the ring signature on both chains and impacting transactions that use this output as a decoy.

## The State of the Hashrate

As of `v7`, a majority of the network hashrate is attributable to public mining pools.  The top 8 public Monero pools (in descending order of hashrate) are: Nanopool, SupportXMR, mineXMR.com, Mining Pool Hub, F2Pool, MinerGate, DwarfPool, and MoneroHash.  These pools represent over 80% of the combined global hashrate.  They all announce enough information to discern some outputs as known spent.  All of the pools announce their blocks and all but one (Nanopool) list their transactions.  (Nanopool does not directly announce all of their payments, but still announces enough information to identify some outputs as spent, as detailed in the Metadata collection section.  What portion of their total payments are attributable is not known at this time and is a topic for future work.)

![Global Monero Mining Network Hashrate Distribution](https://user-images.githubusercontent.com/4107993/39454120-49c38b2a-4c8e-11e8-8b05-1be9323d1985.png)

### Software centralization

TODO: List the most common mining pool stratum servers and GUIs and how code reuse enabled scraping of the data used to prepare this report.  *Thanks, `poolui`!*

[//]: # (Generated by https://www.tablesgenerator.com/markdown_tables)

##### Top 8 Public Monero Mining Pools

| Pool name       | API format             | API endpoint                    |
|-----------------|------------------------|---------------------------------|
| Nanopool        | `nanopool`             | https://api.nanopool.org/v1/xmr |
| SupportXMR      | `poolui`               | https://supportxmr.com/api      |
| mineXMR.com     | `node-cryptonote-pool` | https://p5.minexmr.com          |
| Mining Pool Hub |                        |                                 |
| F2Pool          |                        |                                 |
| MinerGate       |                        |                                 |
| DwarfPool       |                        |                                 |
| MoneroHash      |                        |                                 |

[//]: # (Make a followup table showing % hashrate-per-API)

## Metadata collection

Let `O` be the set of a pool's outputs and `T` be the set of a pool's transactions; let an output `o` be a member of `O` and a transaction `t` be a member of `T`: if `o` is the exclusive output of set 'O' used as a ring member of `t`, then that output `o` is the real member of the ring signature: it is "known spent."

### Mining pool transactions

Mining pools typically "batch" transactions. They use one transaction to send funds to several people at the same time to save fees. [Here](https://xmrchain.net/search?value=22ed46a3b15afa3d580bde985000d1e6b20eae83987c1751dbe1634a95b8ca3e) is an example transaction from SupportXMR. Exchanges tyically "batch" transactions as well.

### Scrape mining pool APIs for blocks (coinbase outputs)

All pool API formats make it easy to scrape mining pools for their blocks.  Suffix the "API call" column to the appropriate "API endpoint" column from the "Mining pool API endpoints for blocks" table and make a plain GET (where `N` is an integer.)

##### Mining pool API endpoints for a mining pools' found blocks

| Pool format   | Found blocks API      |
|---------------|-----------------------|
| Pool format   | API call              |
| poolui        | `pool/blocks?limit=N` |
| nanopool      | `pool/blocks/N`       |
| mineXMR.com   | `stats`               |

[//]: # (example responses and explain parsing)

### Scrape mining pool APIs for transactions

All pool API formats make it easy to scrape mining pools for their transactions except for Nanopool.  Nanopool's scraping is the topic of the Nanopool section.  For all other API formats, just suffix the "API call" column of the transaction to the appropriate "API endpoint" column.

##### Mining pool API endpoints for transactions

| Pool format   | Transaction history API |
|---------------|-------------------------|
| poolui        | `pool/payments?limit=N` |
| nanopool      | `none`                  |
| mineXMR.com   | `to do`

[//]: # (Add API endpoint for payments to a particular Nanopool address)

#### Nanopool

## Mitigation

We discuss several possible mitigation methods and their effectiveness. In general, these mitigation techniques seek to limit the impact of publicly-available pool output information.

### Running a stealth pool

It is possible to run a pool that reveals no information regarding what coinbase outputs it mines or what transactions it makes. However, this is unrealistic, since most miners prefer the transparency.

### Listing and blackballing all coinbase outputs

It is likely that mining pools will want to, at a minimum, publish a list of all coinbase outptus they earn. In this case, users should blackball all coinbase outptus listed by pools. As noted in section ______, pool transactions are often identifiable. Even if 

### Listing and blackballing all pool-controlled outputs

Pools could maintain a list of every output they have controlled in the past and currently control. Users should blackball all outputs in this list, since it is obvious that a user cannot spend these.

Even though the output list can be recreated by searching through the coinbase and transaction histories, there are several possible transaction constructions that may corrupt the output list in practice.

### Pool creating transactions with multiple 'o' outputs from set 'O' in its ring signatures

We considered whether a pool could reduce tracability by including multiple 'o' outputs from set 'O' in a single ring signature. Unfortunately, this is ineffective if pools publish their transaction history. Any decoys in other users' transactions would know to be fake, since they would not appear in the pool transaction list. Furthermore, this is a very delicate selection process that could easily fail if there is a mistake.

These steps may provide some protection for the pool operator, but these protections are removed if the transactions are published publicly anyway. There is little practical use for this mitigation.

### Pool creating transactions using miner payout outputs in its ring signatures

Pools could create transactions that include outputs paid to miners in ring signatures of future transactions. It would have to include the change output and one or more outputs from the same transaction in a single ring signature. If this feature is repeatedly used, then pools could publish a list of transactions without the need for users to blackball non-coinbase outptus.

### Secret churning

Pools can take advantage of "secret churning" (defined as sending transactions to a wallet the pool controls without reporting these transactions to the public) to maintain the integrity of network outputs.

In the best case, a pool can individually churn every coinbase output at least one time. This would remove the need to blackball these coinbase outputs even if they are publicly known, since there is at least some plausible deniability. Subsequent transactions posted should have enough entropy from the other outputs that they do not need to be blackballed.

If a pool groups multiple coinbase outputs together in the first churn, then users should still blackball the coinbase outputs. It is nearly impossible to construct transactions where the decoy could have conceivably been spent. Subsequent transactions posted should have enough entropy from the other outputs that they do not need to be blackballed.

## Concerns

There are several concerns introduced by these recommendations.

### Censorship

Introducing and encouraging a widely-used blackball system increases the effectiveness of censorship and node sybil attacks.

### Pool corruption

Miners prefer transparent pools, since there is a lower chance that these pools will act against the interests of its miners. Transparency recuces the risk that a pool will keep coinbase outputs for themself or make malicious withdraws. If pools reveal less information, it increases the opportunity for corruption.

### Low blackball use

In order for many of the mitigations to be effective, users must take advantage of the blackball features. If a pool makes more information publicly available to facilitate the blackball process, this easier availability of information may harm the network if hardly anyone uses the blackball features.

### Blockchain bloat

Churning as described in the mitigations increases the bandwidth, storage, and verification costs of the network.

### Blackball database submissions

TODO: Describe the https://xmreuse.daemon.network API for querying blackball database submissions

## Future work

 - Graphs & charts
 - Definitions & descriptions
 - Provide Nanopool outputs for blackballing
 - Meta
     - "Contributing" section
     - PDF build instructions
 - Direct connection to mining pools for the purpose of scraping blocks

## Conclusion

## References
