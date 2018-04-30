# Monero v7 Mining Pool Report

May 2018

by sneurlax <sneurlax@gmail.com> (https://github.com/sneurlax)

## Abstract

An overview of Monero v7 mining pools with a current example of degraded privacy in one of Monero's privacy layers due to poor practices on the part of mining pools, a report on mitigation strategies including submissions to the blackball database, and a request for comments on best practices or solutions going forward.  This report applies to Monero mainnet version 7 (v7) and showcases degraded privacy due to publicly-available metadata: mining pools' announced finds (blocks) and payments (transactions.)  Mining pools are recommended to improve "privacy by obscurity" by disclosing less information publicly, but simple practices are presented in order to allow mining pools to proactively improve their privacy while maintaining the current level of metadata disclosure.

###### Formatting note

This report's final format will be LaTeX (see `monero-v7-mining-pool-report.tex` and its PDF output, `monero-v7-mining-pool-report.pdf`,) but this README is in Markdown format.  The report's text will be refined and finished here in Markdown format first and migrated to LaTeX in its final versions for presentation.

## Introduction

As of Monero v7 (which began at block height 1546000,) a majority of the Monero hashrate is attributable to particular mining pools and there are seven or eight public pools with over 1\% of the total global hashrate.  These pools advertise various statistics---metadata---including the blocks that they have mined, and all but one pool (Nanopool) list the payouts that they have made to their miners.  (Nanopool does not directly announce all of their payments, but still does announce enough information to identify some of their transactions.  What portion of their total payments are attributable is not known at this time and is a topic for future work.)  The combination of output ownership and transaction authorship allow the true member of a ring signature to be inferred beyond a reasonable doubt in some cases, degrading one of Monero's layers of privacy.

### Monero's privacy protections 

Describe them and identify which layer is being degraded

### Compromising Confidential Ring Signatures

Describe how blockchain forks (*eg.* Monero Classic and MoneroV) and these mining pool practices compromise Confidential Ring Signatues 

### Monero v7's countermeasures against privacy degradations

Describe how the introduction of the blackball database mitigates the effects of these privacy degradations upon end-users

## The State of the Hashrate

List the pools in descending order of hashrate

### Hashrate distribution

List as many sources / references for hashrate distribution as is convenient

### Software centralization

List the most common mining pool stratum servers and GUIs.  *Thanks, `poolui`!*

## Metadata collection

Describe the metadata that will be collected, where it will be collected from, and how it will be used.

### `poolui`

### Nanopool

## Conclusion

Pool operators: either announce less information *or* churn prior to paying out to miners.

## Future work

 - Graphs & charts
 - Definitions & descriptions
    "...one of Monero's three layers of privacy..."  Describe Monero's "three layers of privacy."
 - Provide Nanopool outputs for blackballing
 - References
     - v7 fork height & date
     - List specific pools
     - Monero hashrate distribution & attributability
 - Meta
     - "Contributing" section
     - PDF build instructions
