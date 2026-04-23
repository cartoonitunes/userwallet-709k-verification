# Exchange UserWallet (709k Fleet) — Bytecode Verification

Source code verification for Ethereum contract [0x24eb8813f35fa3f594d8060cd05ecb2c3ba068a7](https://etherscan.io/address/0x24eb8813f35fa3f594d8060cd05ecb2c3ba068a7).

Part of [Ethereum History](https://ethereumhistory.com) — a project to recover and verify the source code of historically significant Ethereum contracts.

## The Contract

This is an **exchange deposit wallet** — one of 709,039 contracts with identical runtime bytecode, making it the **most deployed bytecode pattern in Ethereum history**. Each contract was deployed as a dedicated deposit address for an exchange user. The deployer (`0x17bc58b788808dab201a9a90817ff3c168bf3d61`, nonce 121,422) was a high-volume exchange hot wallet.

- **Address**: `0x24eb8813f35fa3f594d8060cd05ecb2c3ba068a7`
- **Deployed**: Block 4,850,028 (January 4, 2018) — peak ICO mania era
- **Siblings**: 709,039 contracts with identical runtime bytecode
- **Deployer**: `0x17bc58b788808dab201a9a90817ff3c168bf3d61` (nonce 121,422)

## Functions

| Selector | Function | Description |
|---|---|---|
| `0x395ede4d` | `collectToken(address)` | Sweep ERC20 token balance to owner |
| `0x83197ef0` | `destroy()` | Self-destruct, sending ETH to owner |
| `0x8da5cb5b` | `owner()` | Returns the owner address |
| `0xe5225381` | `collect()` | Sweep ETH balance to owner |
| `(fallback)` | `function() payable` | Accept ETH deposits |

The owner (set to `msg.sender` at deploy time = the exchange hot wallet) could sweep funds at any time. Users deposited ETH or tokens, the exchange swept them into its treasury.

## Match Result

**Code match** — the 627-byte runtime code is an exact bit-for-bit match. The CBOR metadata suffix differs (it encodes a Swarm hash of the source metadata including original filename and whitespace, which we cannot recover).

## Compiler

```
solc 0.4.14 / 0.4.15 / 0.4.16
--optimize --optimize-runs 200
```

All three versions produce identical runtime bytecode.

## Key Archaeology Findings

The source was non-trivial to reconstruct despite its simplicity:

1. **`modifier onlyOwner`** — The access control uses a Solidity modifier with old-style `if (msg.sender != owner) throw`, not the later `require()`. The modifier generates `JUMPDEST` merge points at the `_` placeholder that produce the `5b 5b 56` opcode tail pattern in `destroy()` and `collect()`.

2. **`Token t = Token(token)` in `collectToken`** — Caching the typed reference before calling `balanceOf` and `transfer` causes the compiler to keep an extra stack slot (the storage slot `0x00`) throughout the function body. This generates the `DUP2 SWAP1` before the `CALLER` opcode and the three-POP cleanup (`5b 5b 50 50 50 56`) instead of two.

3. **`pragma solidity ^0.4.11`** — The contract accepts 0.4.11+, actual compiler was 0.4.14–0.4.16.

## Verification

```bash
# Install solc 0.4.16
solc-select install 0.4.16
solc-select use 0.4.16

# Compile
solc --optimize --optimize-runs 200 --bin-runtime UserWallet.sol

# Compare output (strip CBOR metadata suffix before comparing)
cast code 0x24eb8813f35fa3f594d8060cd05ecb2c3ba068a7 --rpc-url https://mainnet.infura.io/v3/YOUR_KEY
```

The runtime code (stripping the `a165627a7a7230...0029` CBOR suffix) matches exactly.

## Attribution

Crack performed by [EthereumHistory](https://ethereumhistory.com). Source submitted under the EthereumHistory open verification project.
