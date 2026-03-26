# Resolv Labs — Unauthorized USR Minting via AWS KMS Compromise — PoC

> **Educational Purpose Only** — This PoC is created for security research and education
> purposes only. It uses local mocks, not mainnet.

## Quick Start

```bash
git clone https://github.com/NomosLabs-Security/poc-resolv-labs-2026
cd poc-resolv-labs-2026
forge install foundry-rs/forge-std --no-git
forge test -vvvv
```

## Details
- **Exploit Date:** 2026-03-22
- **Chain:** Ethereum
- **Loss:** ~$24.5M
- **Technique:** AWS KMS infrastructure compromise — unauthorized USR minting via SERVICE_ROLE key
- **First Exploit TX:** [0xfe37f25efd67d0a4da4afe48509b258df48757b97810b28ce4c649658dc33743](https://etherscan.io/tx/0xfe37f25efd67d0a4da4afe48509b258df48757b97810b28ce4c649658dc33743)
- **Second Exploit TX:** [0x41b6b9376d174165cbd54ba576c8f6675ff966f17609a7b80d27d8652db1f18f](https://etherscan.io/tx/0x41b6b9376d174165cbd54ba576c8f6675ff966f17609a7b80d27d8652db1f18f)
- **Attacker:** [0x04a288a7789dd6ade935361a4fb1ec5db513caed](https://etherscan.io/address/0x04a288a7789dd6ade935361a4fb1ec5db513caed)
- **USR Counter Contract:** [0xa27a69Ae180e202fDe5D38189a3F24Fe24E55861](https://etherscan.io/address/0xa27a69Ae180e202fDe5D38189a3F24Fe24E55861)
- **USR Token:** [0x66a1e37c9b0eaddca17d3662d6c05f4decf3e110](https://etherscan.io/token/0x66a1e37c9b0eaddca17d3662d6c05f4decf3e110)
- **Expected Output:** Tests demonstrating how a compromised SERVICE_ROLE key enables 500x inflated minting with no on-chain validation, and how ratio checks + multisig + caps prevent the attack
- **Full Analysis:** [NomosLabs Security Intelligence Archive](https://nomoslabs.io/archive/resolv-labs-2026)

## Vulnerability

The Resolv Labs USR stablecoin used a USR Counter contract (`0xa27a69Ae...`) with a `completeSwap()` function controlled by a single SERVICE_ROLE EOA. The signing key was stored in AWS KMS.

The attacker compromised Resolv's AWS KMS environment and used the SERVICE_ROLE key to call `completeSwap()` with inflated output amounts:
- **TX 1:** Deposited 100,000 USDC → received 50,000,000 USR (500x expected amount)
- **TX 2:** Minted an additional 30,000,000 USR

The contract had **no on-chain validation** of the output amount:
1. No oracle check to verify the deposit-to-output ratio
2. No per-transaction or daily minting caps
3. No circuit breaker to auto-pause on anomalous activity
4. Single EOA as SERVICE_ROLE (not multisig)

USR crashed to $0.025 on Curve Finance within 17 minutes. The attacker extracted ~$24.5M.

## Tests

| Test | Description |
|------|-------------|
| `test_exploit_firstMint_50M_USR` | First TX: 100K USDC → 50M USR (500x, no validation) |
| `test_exploit_fullAttackFlow` | Complete attack: both mints totaling 80M unbacked USR |
| `test_exploit_noAmountValidation` | 1 USDC → 1B USR accepted (no ratio check) |
| `test_fixedCounter_ratioCheckBlocks500x` | Fix: on-chain ratio validation rejects 500x |
| `test_fixedCounter_legitimateSwapWorks` | Fix: 1:1 swap works within bounds |
| `test_fixedCounter_multisigRequired` | Fix: 2-of-3 multisig blocks single-key swap |
| `test_fixedCounter_perTxCapEnforced` | Fix: per-TX cap rejects oversized swaps |
| `test_fixedCounter_circuitBreaker` | Fix: pause mechanism blocks all swaps |
| `test_comparison_attackImpact` | Side-by-side: 50M minted (vulnerable) vs 0 (fixed) |

## License

MIT — For educational use only.
