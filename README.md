# VaultPay

Build a small EVM payment dApp with virtual ERC20 `tUSD`. Use local Hardhat only — do not use mainnet or real assets.

## Implementation Status

✅ **Complete** - All contract functions implemented, tested, and deployed.

### Contract Implementation

**VaultPay.sol** - Escrow-style ERC20 payment contract with the following functions:

- `createPayment` — escrow `tUSD`, store payment, emit `PaymentCreated`
  - Validates recipient address, amount, and deadline
  - Transfers tokens from payer to contract
  - Returns payment ID
- `claimPayment` — recipient only, emit `PaymentClaimed`
  - Verifies caller is the intended recipient
  - Checks payment hasn't expired
  - Transfers tokens to recipient
- `cancelPayment` — payer only after deadline, refund payer, emit `PaymentCancelled`
  - Verifies caller is the original payer
  - Checks payment has expired (after deadline)
  - Refunds tokens to payer
- `getPayment` — return payment details

**MockTUSD.sol** - ERC20 token with faucet functionality for testing.

### Test Results

All 6 tests passing:
- ✅ creates a payment and escrows tUSD
- ✅ lets the recipient claim an active payment
- ✅ blocks claims from non-recipients
- ✅ lets payer cancel only after deadline
- ✅ prevents double claim or cancel
- ✅ rejects invalid payment creation inputs

## Setup

From the repo root:

```bash
npm install
npm run compile
npm run test
```

### Quick Start (Hardhat Network)

For testing without running a local node:

```bash
npm run deploy:hardhat
```

This deploys to the built-in Hardhat network and prints contract addresses.

### Local Development Setup

Run the local stack in **three terminals** (keep each process running):

| Terminal | Command | Purpose |
|----------|---------|---------|
| 1 | `npm run node` | Hardhat JSON-RPC (port **8545**) |
| 2 | `npm run deploy:local` | Deploy `MockTUSD` + `VaultPay` (run after terminal 1 is up) |
| 3 | `cd frontend && npm install && npm run dev` | Vite dev server (port **5173**) |

After deploy, copy the printed contract addresses into `frontend/src/config.ts` (`TOKEN_ADDRESS`, `VAULTPAY_ADDRESS`).

**App (browser):** after `npm run dev`, open **[http://127.0.0.1:5173/](http://127.0.0.1:5173/)** (recommended on Windows). [http://localhost:5173/](http://localhost:5173/) should work as well once the server is listening on IPv4.

**Blockchain (MetaMask):** add a custom network with RPC `http://127.0.0.1:8545`, chain ID `31337`. Import a Hardhat test account private key from the `npm run node` output. Do not use port 5173 for MetaMask; that port is only the web UI.

**If you see `ERR_CONNECTION_REFUSED`:** the dev server is not running, or it was only bound to IPv6. Stop the frontend (`Ctrl+C`), run `npm run dev` again from `frontend/`, then use **http://127.0.0.1:5173/** (not port 8545). In a separate terminal, `netstat -ano | findstr 5173` should show `0.0.0.0:5173` or `127.0.0.1:5173`, not only `[::1]:5173`.

## Technology Stack

- **Smart Contracts:** Solidity 0.8.24, Hardhat 2.22.19
- **Security:** OpenZeppelin contracts 5.0.2 (ERC20, SafeERC20, ReentrancyGuard, Ownable)
- **Frontend:** React 18.3.1, TypeScript 5.4.5
- **Web3:** Wagmi 2.12.11, Viem 2.21.19
- **Build:** Vite 5.4.0
- **Styling:** TailwindCSS 4

## Security Features

- Reentrancy protection on all state-changing functions
- Safe ERC20 token transfers
- Input validation (zero address, zero amount, invalid deadline)
- Access control (recipient-only claims, payer-only cancellations)
- Deadline enforcement for cancellations
- Payment status tracking to prevent double operations
