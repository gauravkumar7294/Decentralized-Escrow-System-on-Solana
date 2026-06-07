# Decentralized Escrow System on Solana

A trustless token swap (escrow) program built with Anchor on Solana. This project demonstrates Program Derived Addresses (PDAs) as custodial vaults, Cross-Program Invocations (CPIs) using `transfer_checked`, and the Token Interface for seamless compatibility with both legacy SPL Token and Token-2022 mints.

## Why Build an Escrow?

Escrow programs are one of the best ways to learn core Solana development concepts. This project covers:

* Creating and managing PDAs as custodial token vaults
* Cross-Program Invocations (CPIs) with `transfer_checked`
* Using the Token Interface (`TokenInterface`) for SPL Token and Token-2022 compatibility
* Closing accounts and returning rent to the appropriate party
* Enforcing on-chain invariants using Anchor constraints such as `has_one`
* Secure token custody and atomic token swaps

These concepts form the foundation of more advanced DeFi applications such as AMMs, lending protocols, decentralized exchanges, and order books.

---

## Overview

Two participants — a **Maker** and a **Taker** — can exchange tokens without trusting each other or relying on a third party.

The Maker deposits Token A into a PDA-controlled vault and specifies how much Token B they expect in return. Any Taker holding Token B can fulfill the trade atomically.

If no Taker accepts the offer, the Maker can reclaim their deposited tokens at any time.

```text
Maker deposits Token A
          │
          ▼
   Vault (PDA-owned)
          │
          ▼
Taker sends Token B to Maker
          │
          ▼
Vault releases Token A to Taker
          │
          ▼
Escrow and Vault Accounts Closed
Rent Returned to Maker
```

---

---

## Features

* Trustless peer-to-peer token swaps
* PDA-owned token vaults
* Atomic escrow execution
* SPL Token support
* Token-2022 support
* Secure account validation
* CPI-based token transfers
* Rent recovery on account closure
* Custom error handling
* Modular Anchor architecture

---

## Tech Stack

* Solana
* Anchor Framework
* Rust
* SPL Token Program
* Token-2022
* LiteSVM

---

## Project Structure

```text
.
├── idls/
├── programs/
│   └── anchor_escrow/
│       ├── Cargo.toml
│       ├── Xargo.toml
│       └── src/
│           ├── instructions/
│           │   ├── make.rs
│           │   ├── take.rs
│           │   ├── refund.rs
│           │   └── mod.rs
│           ├── state.rs
│           ├── errors.rs
│           └── lib.rs
├── tests/
├── Anchor.toml
├── Cargo.toml
└── README.md
```

---

## Instructions

### make

Creates a new escrow agreement.

The Maker deposits Token A into a vault and specifies how much Token B they expect in return.

#### Arguments

| Argument | Type | Description                                            |
| -------- | ---- | ------------------------------------------------------ |
| seed     | u64  | Arbitrary PDA seed allowing multiple escrows per maker |
| receive  | u64  | Amount of Token B expected from the taker              |
| amount   | u64  | Amount of Token A deposited into the vault             |

#### Requirements

* `amount > 0`
* `receive > 0`

---

### take

Completes the escrow atomically.

The program:

1. Transfers `escrow.receive` Token B from Taker to Maker
2. Transfers all escrowed Token A from Vault to Taker
3. Closes the Vault account
4. Closes the Escrow account
5. Returns rent to the Maker

---

### refund

Cancels an escrow agreement.

Only the original Maker may invoke this instruction.

The program:

1. Transfers all Token A from the Vault back to the Maker
2. Closes the Vault account
3. Closes the Escrow account
4. Returns rent to the Maker

---

## Accounts

### Escrow PDA

Seeds:

```text
["escrow", maker_pubkey, seed]
```

#### Fields

| Field   | Type   | Description                |
| ------- | ------ | -------------------------- |
| seed    | u64    | PDA seed selected by maker |
| maker   | Pubkey | Escrow creator             |
| mint_a  | Pubkey | Token being offered        |
| mint_b  | Pubkey | Token requested            |
| receive | u64    | Amount of Token B required |
| bump    | u8     | PDA bump                   |

---

### Vault Account

Associated Token Account owned by the Escrow PDA.

Purpose:

* Holds deposited Token A
* Released during successful swap
* Returned during refund
* Closed after completion

---

## Token Interface Support

This program uses:

* `TokenInterface`
* `InterfaceAccount<Mint>`
* `InterfaceAccount<TokenAccount>`

instead of concrete SPL Token account types.

As a result, the same escrow logic works with:

* Legacy SPL Token mints
* Token-2022 mints

without requiring any code changes.

---

## Security Considerations

* PDA-controlled vault ownership
* Signer verification
* Mint validation
* Account ownership checks
* Anchor account constraints
* Atomic token transfers
* Escrow state validation
* Rent-safe account closure

---

## Error Codes

| Code          | Message        |
| ------------- | -------------- |
| InvalidAmount | Invalid amount |
| InvalidMaker  | Invalid maker  |
| InvalidMintA  | Invalid mint a |
| InvalidMintB  | Invalid mint b |

---

## Building

```bash
anchor build
```

---

## Testing

Run the test suite:

```bash
anchor test
```

The tests validate:

* Escrow creation
* PDA vault initialization
* Successful token swaps
* Escrow refunds
* Invalid account scenarios
* Token interface compatibility

---

## Learning Outcomes

This project demonstrates:

* Solana smart contract development
* Anchor framework architecture
* PDA creation and management
* CPI execution with `transfer_checked`
* SPL Token and Token-2022 interoperability
* Secure escrow design patterns
* On-chain state management

---

## License

MIT License

---

## Author

**Gaurav Kumar**

Built using Solana, Rust, Anchor, LiteSVM, SPL Token, and Token-2022 to explore secure, trustless asset exchange mechanisms on-chain.
