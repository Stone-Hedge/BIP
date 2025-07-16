BIP:        XXXX
Title:      On‑Chain Recording of Average Transaction Fee per Block
Author:     Daniel Ames (danamesuk@gmail.com)
Status:     Draft
Type:       Standards Track
Created:    2025‑07‑16

## Abstract

This BIP proposes adding an on‑chain field (avg_fee) to each block that records the arithmetic mean of all transaction fees paid in that block.  Nodes verify the correctness of this field as part of block validation.  This enables wallets, explorers, and smart contracts to read accurate, tamper‑proof fee signals directly from the blockchain.

## Motivation

1. Fee estimation today relies on off‑chain mempool snapshots or external oracles.  
2. Users overpay due to volatility and first‑price auction inefficiency.  
3. On‑chain average fee data improves transparency, wallet fee estimation, mempool management, and enables protocol‑level fee‑adaptive behaviors.

## Specification

### Block Format Extension

Two alternative deployment paths are defined:

1. **Hard/Soft‑Fork Header Field**  
   - Add a new 64‑bit unsigned field `avg_fee` to the block header (after the nonce).  
   - `avg_fee` is equal to ⌊(∑(tx_input_sum – tx_output_sum)) ÷ n_txs⌉, where n_txs is the number of non‑coinbase transactions.  
   - Nodes must recalculate total fees and n_txs upon receipt; if the computed average differs from the header’s `avg_fee`, the block is invalid.

2. **Coinbase OP_RETURN** (Soft‑Fork Friendly)  
   - Miners include a single OP_RETURN output in the coinbase transaction, pushing an 8‑byte little‑endian `avg_fee`.  
   - Full nodes parse the coinbase script, extract the value, recompute their own `avg_fee`, and reject the block if mismatched.  
   - No header change; enforcement is handled via policy once a majority of hashing power signals readiness.

### Miner Algorithm

1. Collect candidate transactions.  
2. Compute `total_fees = Σ (input_value – output_value)` over all non‑coinbase txs.  
3. Compute `avg_fee = round(total_fees / n_txs)`.  
4. Embed `avg_fee` in the header field or OP_RETURN.  
5. Proceed to mine and broadcast.

### Node Validation

Upon receiving a block, a node:

1. Sums all non‑coinbase tx fees and counts transactions.  
2. Calculates `local_avg = round(total_fees / n_txs)`.  
3. If header’s or coinbase’s `avg_fee ≠ local_avg`, reject the block.

## Backwards Compatibility

- Header‑based approach requires a hard or soft fork with a version bit.  
- OP_RETURN approach can activate via miner signalling and soft‑fork rules (e.g. BIP9). Legacy nodes ignore the extra output but do not enforce validity until activation.

## Deployment Considerations

- Feature flag via version bit ensures coordinated upgrade.  
- Wallets and explorers must be updated to read and display the field.  
- Soft‑fork path lowers activation barriers by avoiding header changes.

## Rationale Against Cheating

Because block validity depends on matching the recalculateduted average, any incorrect claim results in immediate block rejection and orphaning — miners gain nothing by lying.

## Reference Implementation

A prototype patch for Bitcoin Core is available at:  


## Rejection Criteria

This proposal should be rejected if:

- It unduly complicates block validation without clear real‑world benefit.  
- Fee‑market behavior can be efficiently improved via off‑chain or layer‑2 mechanisms.

## Copyright

This document is placed in the public domain.  
