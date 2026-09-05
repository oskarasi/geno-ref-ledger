# geno-ref-ledger

A multi-module **cents ledger** reference app for [Geno](https://github.com/davidiach/geno-lang) 0.4.3.

Integer-only money (no floats in the core), double-entry-ish transactions
(`Deposit` / `Withdraw` / `Transfer`), balances as a list of pairs, and a
capability-free CLI driver with heavy `example` clauses.

This is deliberately larger than single-file demos: six `.geno` modules,
explicit sibling `import`s, named args for arity ≥ 3, and compile targets for
both Python and JS.

## Why this is a reference app

- **Multi-file architecture** matching `examples/apps/geno-check` (`geno.toml`
  `files` list + per-module imports).
- **Contracts where safe**: `requires` / `ensures` on helpers whose examples are
  all in-range; Result APIs that cover `Err` paths omit conflicting `requires`
  (geno-lang #73 workaround).
- **Capability-free default path**: `main` and `Cli.run(args)` need no caps so
  `geno test` / `geno run` pass in the sandbox.
- **Optional real CLI**: `@untested cli_main` uses `cli_args` / `print` — see
  below for `--unsafe --cap env,print`.

## Architecture

```
┌─────────────┐
│   Main.geno │  entry (demo) + optional cli_main
└──────┬──────┘
       │ import Cli
┌──────▼──────┐
│   Cli.geno  │  run(args) → Result[String, String]
└──┬───┬───┬──┘
   │   │   │
   │   │   └──────────────┐
   │   │                  │
┌──▼───▼──┐   ┌───────────▼──┐
│ Money   │   │   Ledger     │◄── BalPair list, apply_tx / statement
└────▲────┘   └──────▲──▲────┘
     │               │  │
     │         ┌─────┘  └─────┐
     │         │              │
┌────┴────┐ ┌──┴───┐   ┌──────┴─────┐
│ Account │ │ Tx   │   │ (imports   │
│ id/name │ │ ADT  │   │  Money,Tx, │
└─────────┘ └──────┘   │  Account)  │
                       └────────────┘
```

| Module | Role |
|--------|------|
| **Money** | `format_cents`, `parse_cents_arg` (`parse_int`), `parse_dollars`, `dollars_to_cents` |
| **Account** | `Account` record, `open_account`, labels |
| **Tx** | `Deposit` / `Withdraw` / `Transfer`, `validate_tx`, `describe_tx` |
| **Ledger** | `BalPair` list balances, `apply_tx` / `apply_all`, statement helpers, demo script |
| **Cli** | `run(args)` subcommands |
| **Main** | capability-free `main`; optional `cli_main` |

## API overview

```text
Money.format_cents(1234)            -> "$12.34"
Money.parse_dollars("12.34")        -> Ok(1234)
Money.parse_cents_arg("1234")       -> Ok(1234)

Account.open_account(id:, name:)    -> Result[Account, String]

Tx.validate_tx(Deposit("cash", 100)) -> Ok(...)
Tx.describe_tx(...)                  -> "deposit cash +$1.00"

Ledger.apply_tx(balances, tx)       -> Result[List[BalPair], String]
Ledger.apply_all(balances, txs)     -> Result[List[BalPair], String]
Ledger.format_statement(balances, txs) -> String

Cli.run(["demo" | "format" | "balance" | "apply" | "dollars", ...])
```

Subcommands:

| Args | Meaning |
|------|---------|
| `demo` | Full demo statement (seed script) |
| `format <cents>` | Format integer cents |
| `balance` | Demo ending balances only |
| `apply` | Same as `demo` (apply script + statement) |
| `dollars <amount>` | Parse `"12.34"` → cents integer string |

## Test / run

```bash
geno test .
geno run .
geno check .
```

Expected: all example clauses pass; `geno run` prints the demo statement:

```text
deposit cash +$10.00
deposit checking +$5.00
transfer cash -> checking $2.50
withdraw checking -$1.00
---
cash: $7.50
checking: $6.50
```

## Compile to Python / JS

```bash
geno compile -o /tmp/ledger.py .
geno compile --target js -o /tmp/ledger.js .
# optional profile (node-cli is also listed in geno.toml targets)
geno compile --target js --profile node-cli -o /tmp/ledger-node.js .
```

Verified on Geno **0.4.3**: both Python and JS compiles succeed for this project.

`geno.toml` targets: `python-cli`, `node-cli` (both accepted by `geno check`).

## Optional CLI (`cli_main`)

Default `main` is capability-free. For argv + print:

1. Point `entrypoint` at a wrapper that calls `cli_main`, **or** invoke
   `cli_main` from a small harness, and
2. Run with:

```bash
geno run --unsafe --cap env,print .
```

`--cap` requires `--unsafe` or `--json` (process-isolated default rejects bare
`--cap`). Pattern: keep pure `Cli.run(args)` for tests; gate I/O behind
`@untested cli_main`.

## Layout

```text
geno-ref-ledger/
  geno.toml
  Money.geno
  Account.geno
  Tx.geno
  Ledger.geno
  Cli.geno
  Main.geno
  README.md
  LICENSE
```

## License

Apache-2.0 (see `LICENSE`).
