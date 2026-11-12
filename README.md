# clone-lin-compound-xrate

**EXPERIMENTAL** LIN clone of Compound v2
[`CToken.exchangeRateStoredInternal`](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/CToken.sol).

This is **not** a cToken, not a Compound replacement, and not uint256.
Proofs live in [kbelludoo/lin-open](https://github.com/kbelludoo/lin-open).

## Current kernel (Compiler 0, SCALE=1e9, explicit scale)

- `src/lin_compound_exrate.lin` — cash, borrows, reserves, supply, scale and
  initial are arguments. `(cash+borrows-reserves)*scale/supply` uses a 128-bit
  schoolbook product. Quotient that does not fit i64 fail-closes to 0.
- `test/oracles/compound_exrate_c11.c` — independent C11 `__int128` XOR portable
  32-bit limbs. Not part of the LIN compiler TCB.
- `getCashPrior` / ERC-20 `balanceOf` / `address(this).balance` are **not**
  implemented. The Solidity/C/Rust transpilers in lin-open fail-close those
  as `REJ_*_CASH_PRIOR`.

Named goldens (SCALE=1e9, initial=0.02*SCALE=2e7):

| vector | rate |
|---|---|
| cash=supply=1e9, zero borrows/reserves | 1000000000 |
| 10% borrows | 1100000000 |
| 10% reserves | 900000000 |
| totalSupply == 0 | 20000000 |
| cash+borrows < reserves | 0 (Solidity 0.8 reverts) |

```bash
gcc -O2 -std=c11 -o compound_exrate_c11 test/oracles/compound_exrate_c11.c
./compound_exrate_c11 selftest
# PASS
```

## Proofs (lin-open only)

```bash
make compound-exrate-proof
# or: python3 test/prove_compound_exrate_external.py
```

Event receipt: `docs/events/EVENT_COMPOUND_EXRATE_CLONE_LIN.rulel`

This run: **19 PASS / 0 FAIL / 1 SKIP** (in-memory libtcc JIT unavailable).
C11 `__int128` == limbs == `lin_c0 vm` == Python bigint. Pinned upstream
bytes of `CToken.sol` + `ExponentialNoError.sol` recomputed live.

## Upstream (hash-pinned, not vendored)

| Field | Value |
|---|---|
| Repo | https://github.com/compound-finance/compound-protocol |
| Path | `contracts/CToken.sol` + `contracts/ExponentialNoError.sol` |
| Commit | `a3214f67b73310d547e00fc578e8355911c9d376` |
| CToken sha256 | `b64790f41ef851b09856af286ce8253b95ef34fda4ae836f924a519e8f792785` |
| CToken blob | `b8c878c79c75855808f6e7303099a367c8bad98b` |
| ExponentialNoError sha256 | `dd272760b2d9082db9c94f6198896402fe3681bf41fc3f9d8bba31ada290c3c5` |
| ExponentialNoError blob | `23f833760db898ebdc6b2fef9654cb9bc1dd21c3` |
| License (upstream) | BSD-3-Clause |

## Related

September 2026 SCALE=1e18 variant (`cer_*`, hardcoded 1e18):
https://github.com/kbelludoo/clone-lin-compound-exrate

## Non-claims

- Solidity uint256 / mainnet expScale=1e18 / live cToken parity
- LIN replacing Compound or the EVM
- Any TVL or dollar-savings figure
- zk / computational soundness of Merkle receipts
- `getCashPrior` ERC-20/ETH balance reads
- `accrueInterest` (see [clone-lin-compound-accrue](https://github.com/kbelludoo/clone-lin-compound-accrue))
