# Changelog

All notable changes to **geno-ref-ledger** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] — 2026-09-05

### Added

- Multi-module cents ledger reference app for Geno 0.4.3:
  `Money`, `Account`, `Tx`, `Ledger`, `Cli`, `Main`.
- Integer-only money helpers (`format_cents`, `parse_cents_arg`, `parse_dollars`,
  `dollars_to_cents`) with Result error paths.
- Double-entry-ish transactions: `Deposit` / `Withdraw` / `Transfer` with
  `validate_tx` and statement descriptions.
- Balance list as `BalPair` rows; `apply_tx` / `apply_all` / `format_statement`.
- Capability-free `Cli.run(args)` subcommands: `demo`, `format`, `balance`,
  `apply`, `dollars`.
- Optional `@untested cli_main` for `--unsafe --cap env,print`.
- `geno.toml` targets: `python-cli`, `node-cli`.
- README architecture diagram and compile / test instructions.
- Strengthened example clauses on Account / Ledger / Cli / Tx helpers.

### Notes

- `requires` avoided on Result APIs that cover `Err` examples (geno-lang #73).
- Zero-arg helpers that need examples use a dummy `Bool` parameter or `@untested`.
