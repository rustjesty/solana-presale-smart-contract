# Solana Presale Smart Contract

A secure, feature-rich token presale program built on Solana using the Anchor framework. Supports configurable softcap/hardcap mechanics, per-wallet purchase limits, timed presale windows, and admin-controlled lifecycle management.

## Features

- **Configurable Presale Parameters** — Set token price, softcap, hardcap, per-wallet limits, and time windows at creation or via update.
- **Softcap / Hardcap Enforcement** — Automatically tracks raised SOL against defined thresholds; halts purchases once hardcap is reached.
- **Per-Address Purchase Limits** — Prevents any single wallet from exceeding a configurable token allocation.
- **Time-Gated Presale Windows** — Enforces start and end timestamps (millisecond precision) so purchases are only valid within the active window.
- **Post-Sale Token Claims** — Buyers claim purchased tokens after the presale ends via a dedicated claim instruction.
- **Admin Lifecycle Controls** — Authority can create, update, start, deposit tokens, and withdraw SOL/tokens from the vault.
- **PDA-Secured Vaults** — All funds and tokens are held in Program Derived Address accounts, ensuring trustless custody.

## Architecture

```
programs/mari-presale/src/
├── lib.rs                  # Program entry point & instruction dispatch
├── instructions/           # Instruction handlers
│   ├── create_presale.rs   # Initialize presale with parameters
│   ├── update_presale.rs   # Modify presale config (admin)
│   ├── deposit_token.rs    # Deposit SPL tokens into presale vault
│   ├── start_presale.rs    # Activate presale window
│   ├── buy_token.rs        # Purchase tokens with SOL
│   ├── claim_token.rs      # Claim tokens post-sale
│   ├── withdraw_sol.rs     # Admin SOL withdrawal
│   └── withdraw_token.rs   # Admin token withdrawal
├── state/                  # On-chain account structures
│   ├── presale_info.rs     # Presale configuration & status
│   └── user_info.rs        # Per-user purchase tracking
├── errors/                 # Custom error definitions
│   └── errors.rs
└── constants/              # PDA seeds & constants
    └── constants.rs
```

## Instructions

| Instruction        | Access  | Description                                          |
| ------------------ | ------- | ---------------------------------------------------- |
| `create_presale`   | Admin   | Initialize a new presale with token, pricing & caps  |
| `update_presale`   | Admin   | Update presale parameters before or during sale      |
| `deposit_token`    | Admin   | Transfer SPL tokens into the presale vault           |
| `start_presale`    | Admin   | Activate the presale and set the time window         |
| `buy_token`        | Public  | Purchase tokens by sending SOL during active presale |
| `claim_token`      | Public  | Claim purchased tokens after the presale ends        |
| `withdraw_sol`     | Admin   | Withdraw collected SOL from the vault                |
| `withdraw_token`   | Admin   | Withdraw unsold tokens from the vault                |

## On-Chain State

### PresaleInfo (PDA)

| Field                          | Type     | Description                        |
| ------------------------------ | -------- | ---------------------------------- |
| `token_mint_address`           | `Pubkey` | SPL token mint                     |
| `softcap_amount`               | `u64`    | Softcap threshold (lamports)       |
| `hardcap_amount`               | `u64`    | Hardcap threshold (lamports)       |
| `deposit_token_amount`         | `u64`    | Total tokens deposited by admin    |
| `sold_token_amount`            | `u64`    | Total tokens sold to buyers        |
| `start_time`                   | `u64`    | Presale start (ms since epoch)     |
| `end_time`                     | `u64`    | Presale end (ms since epoch)       |
| `max_token_amount_per_address` | `u64`    | Per-wallet purchase cap            |
| `price_per_token`              | `u64`    | Token price in lamports            |
| `is_live`                      | `bool`   | Whether presale is active          |
| `authority`                    | `Pubkey` | Admin authority pubkey             |
| `is_soft_capped`               | `bool`   | Softcap reached flag               |
| `is_hard_capped`               | `bool`   | Hardcap reached flag               |

### UserInfo (PDA)

| Field              | Type  | Description                 |
| ------------------ | ----- | --------------------------- |
| `buy_quote_amount` | `u64` | Total SOL spent by user     |
| `buy_token_amount` | `u64` | Total tokens purchased      |
| `buy_time`         | `u64` | Last purchase timestamp     |
| `claim_time`       | `u64` | Token claim timestamp       |

## Tech Stack

- **Solana** — High-performance L1 blockchain
- **Anchor** `0.29.0` — Solana development framework
- **Rust** — Smart contract language
- **SPL Token** — Solana token standard
- **TypeScript** — Test suite & client scripts

## Prerequisites

- [Rust](https://rustup.rs/) (stable)
- [Solana CLI](https://docs.solanalabs.com/cli/install) (`<1.17.0`)
- [Anchor CLI](https://www.anchor-lang.com/docs/installation) (`0.29.0`)
- [Node.js](https://nodejs.org/) (16+)
- [Yarn](https://yarnpkg.com/)

## Getting Started

### 1. Install Dependencies

```bash
yarn install
```

### 2. Build the Program

```bash
anchor build
```

### 3. Run Tests

```bash
anchor test
```

### 4. Deploy

```bash
anchor deploy
```

> The program is currently configured for **devnet** deployment. Update `Anchor.toml` to target mainnet-beta for production.

## Program ID

```
H6qBMcWs42iZu99FPT2vwhKu1AsdRQzL5JwUs3QvQhGy
```

## Presale Lifecycle

```
┌─────────────┐    ┌──────────────┐    ┌───────────────┐    ┌─────────────┐
│   Create     │───▶│   Deposit    │───▶│    Start      │───▶│  Buy Tokens │
│   Presale    │    │   Tokens     │    │   Presale     │    │  (public)   │
└─────────────┘    └──────────────┘    └───────────────┘    └──────┬──────┘
                                                                   │
                   ┌──────────────┐    ┌───────────────┐           │
                   │  Withdraw    │◀───│  Claim Tokens │◀──────────┘
                   │  SOL/Tokens  │    │  (post-sale)  │   (presale ends)
                   └──────────────┘    └───────────────┘
```

## Error Codes

| Code                  | Description                                  |
| --------------------- | -------------------------------------------- |
| `Unauthorized`        | Caller is not the presale authority           |
| `NotAllowed`          | Action is not permitted                      |
| `MathOverflow`        | Arithmetic operation overflow                |
| `AlreadyMarked`       | State flag already set                       |
| `PresaleNotStarted`   | Purchase attempted before start time         |
| `PresaleEnded`        | Purchase attempted after end time            |
| `TokenAmountMismatch` | Token amount does not match expected value   |
| `InsufficientFund`    | Not enough tokens available                  |
| `PresaleNotEnded`     | Claim attempted before presale ends          |
| `HardCapped`          | Hardcap reached — no more purchases allowed  |

## License

MIT

## Contact

Telegram: [https://t.me/soljesty](https://t.me/soljesty)
