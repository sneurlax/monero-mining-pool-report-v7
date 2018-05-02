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

Ring signatures should conceal the real source of a Monero transaction: there is no way to discern which ring member actually made the transaction without additional information.  Unfortunately, metadata announcements due to poor mining pool practices such as blockchain forks and routine statistics advertisements can provide enough additional information to discern which ring member is the actual source of a transaction.

Blockchain forks have the potential to degrade the untraceability of ring signatures when key images are reused on both sides of a blockchain fork without taking advantage of any of the tools provided by Monero's `v7` upgrade.  Key images may be safely (at least privately) reused across blockchain forks if care is taken to construct identical rings on both sides of the fork (a process which is outside of the scope of this report but is described in detail [here](https://monero.stackexchange.com/questions/7826/how-can-individuals-safeguard-themselves-and-the-community-against-a-key-reusing).)  If users send funds on both sids of the fork without any such precautions, however, then they will inadvertently produce two rings that share only one member in common, thus identifying the common member as the real source of both transactions.  Such outputs are identified as "known spent."

Before any blockchain forks had incentivized key image reuse, mining pools were inadvertently revealing the real source of some payments *via* the statistics that they advertise.  Mining pools announce their blocks.  When they later announce a transaction that uses one of their blocks' outputs as a ring signature member, it is most likely that their output is the real one and thus known spent.

By identifying an output as real in one transaction, its suitability as a decoy is degraded elsewhere, reducing the effective ring size of other users' ring signatures when included in rings after they are revealed to be known spent.


### Monero version 7's countermeasures against mining pool privacy degradations

Several tools were provided by the Monero `v7` upgrade that allow users to avoid or mitigate of the above potential degradations to their privacy outlined previously.  For example, users can avoid including any known spent outputs in their own ring signatures by using what is known as the blackball database, which contains every known spent output.  This report presents additional submissions to the blackball database, including explanations of what metadata identifies the real member of a ring, where to collect it, and example code to scrape and analyze the metadata necessary for independent verification of these results.

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

Let `O` be the set of a pool's outputs and `T` be the set of a pool's transactions; let an output `o` be a member of `O` and a transaction `t` be a member of `T`: if `o` is used as a ring member of `t`, then that output `o` is probably the real member of the ring signature: it is "known spent."

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

TODO: Pool operators: either announce less information *or* churn prior to paying out to miners.

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
