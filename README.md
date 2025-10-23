# SOLAX

Zero-Knowledge Proof System for Solana State Validation — lightweight starter scaffold

This repository is a compact, ready-to-post. It includes minimal starter files for a Solana + Anchor program, a TypeScript orchestrator, a small CLI, Docker support, and a README explaining how to extend and deploy.

---

## Repository layout

```
SOLAX/
├── README.md                # this document (entrypoint)
├── .gitignore
├── Anchor.toml
├── package.json
├── programs/
│   └── solax/
│       ├── Cargo.toml
│       └── src/lib.rs
├── orchestrator/
│   ├── package.json
│   └── src/index.ts
├── cli/
│   ├── package.json
│   └── src/cli.ts
├── docker/
│   └── Dockerfile
└── env.example
```

---

## README (usage)

Use this scaffold to quickly start a SOLAX repo. The included example Anchor program is minimal and intended to compile and deploy to a local validator (or testnet/devnet) once you fill in keys and configuration. The orchestrator is a tiny Node service showing how to call the on-chain program. The CLI demonstrates submitting a basic transaction.

### Quick start (high level)

1. Install prerequisites: `rust`, `cargo`, `anchor-cli`, `node` (v18+), `pnpm` or `npm`.
2. Fill `env.example` -> `.env` with your RPC and DB (if needed).
3. From `programs/solax` run `anchor build` then `anchor deploy` (or use local validator).
4. From `orchestrator` run `pnpm install` then `pnpm start` to run the TypeScript service.
5. Use `cli` to interact with the orchestrator or submit transactions.

---

## Key files (starter contents)

### .gitignore

```
/node_modules
/dist
/target
/.env
/.env.local
*.log
.artifacts
/*.key
```

---

### Anchor.toml

```toml
[programs.localnet]
solax = "ReplaceWithProgramPubkey"

[provider]
cluster = "localnet"
wallet = "~/.config/solana/id.json"
```

---

### env.example

```
PORT=3000
NODE_ENV=development
RPC_URL=https://api.devnet.solana.com
PROGRAM_ID=ReplaceWithProgramPubkey
DATABASE_URL=postgresql://user:password@localhost:5432/solax
```

---

### programs/solax/Cargo.toml

```toml
[package]
name = "solax"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "lib"]

[dependencies]
anchor-lang = "0.28.0"
```

> Note: adjust `anchor-lang` version to match your local Anchor toolchain.

---

### programs/solax/src/lib.rs

```rust
use anchor_lang::prelude::*;

declare_id!("ReplaceWithProgramPubkey");

#[program]
pub mod solax {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>, bump: u8) -> Result<()> {
        let state = &mut ctx.accounts.state;
        state.authority = *ctx.accounts.authority.key;
        state.bump = bump;
        Ok(())
    }

    pub fn ping(ctx: Context<Ping>) -> Result<()> {
        let state = &mut ctx.accounts.state;
        state.counter = state.counter.checked_add(1).unwrap_or(0);
        Ok(())
    }
}

#[account]
pub struct State {
    pub authority: Pubkey,
    pub bump: u8,
    pub counter: u64,
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = authority, space = 8 + 32 + 1 + 8)]
    pub state: Account<'info, State>,
    #[account(mut)]
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Ping<'info> {
    #[account(mut)]
    pub state: Account<'info, State>,
}
```

---

### orchestrator/package.json

```json
{
  "name": "solax-orchestrator",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "start": "node ./dist/index.js",
    "dev": "ts-node src/index.ts",
    "build": "tsc"
  },
  "dependencies": {
    "@solana/web3.js": "^1.73.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "ts-node": "^10.0.0"
  }
}
```

---

### orchestrator/src/index.ts

```ts
import { Connection, PublicKey, Keypair, Transaction, SystemProgram } from "@solana/web3.js";
import dotenv from "dotenv";

dotenv.config();

const RPC = process.env.RPC_URL || "https://api.devnet.solana.com";
const PROGRAM_ID = new PublicKey(process.env.PROGRAM_ID || "ReplaceWithProgramPubkey");

async function main() {
  const conn = new Connection(RPC, "confirmed");
  console.log("SOLAX orchestrator connected to", RPC);

  // Example usage placeholder — extend for proof orchestration
}

main().catch(e => console.error(e));
```

---

### cli/package.json

```json
{
  "name": "solax-cli",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "start": "ts-node src/cli.ts"
  },
  "dependencies": {
    "@solana/web3.js": "^1.73.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "ts-node": "^10.0.0",
    "typescript": "^5.0.0"
  }
}
```

---

### cli/src/cli.ts

```ts
import { Connection, Keypair, PublicKey, SystemProgram, Transaction } from "@solana/web3.js";
import dotenv from "dotenv";

dotenv.config();

const RPC = process.env.RPC_URL || "https://api.devnet.solana.com";
const conn = new Connection(RPC, "confirmed");

async function run() {
  console.log("SOLAX CLI — connecting to", RPC);
  // Example: create a dummy transaction to illustrate usage
  const payer = Keypair.generate();
  const tx = new Transaction().add(
    SystemProgram.transfer({
      fromPubkey: payer.publicKey,
      toPubkey: payer.publicKey,
      lamports: 0,
    })
  );
  console.log("Prepared sample transaction (no-op)");
}

run().catch(console.error);
```

---

### docker/Dockerfile

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY orchestrator/package.json orchestrator/package-lock.json* ./
RUN npm install --production
COPY orchestrator/dist ./dist
CMD ["node", "dist/index.js"]
```

---
