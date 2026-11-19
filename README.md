# Raydium CPI adapters — Anchor 1.2.x fork

Fork of [raydium-io/raydium-cpi](https://github.com/raydium-io/raydium-cpi) for programs on
Anchor 1.2.x.

Upstream pins `anchor-lang`/`anchor-spl` to `=1.0.2`. Because the pin is exact, cargo cannot
unify it with any other 1.x release, so an Anchor 1.2.2 program fails to resolve these crates
even though 1.0 and 1.2 are semver compatible. This branch relaxes the pins to `1.2` and
changes nothing else — no source changes were needed.

Upstream's branches (`anchor-0.29.0` … `anchor-0.32.1`) remain the right choice for older
Anchor versions. Use `master` for upstream's current pin.

## Environmental requirements

```
anchor-cli 1.2.2
solana-cli 4.1.2
platform-tools v1.54 (rustc 1.89.0)
```

## Usage

Call CPMM through CPI.
```toml
[dependencies]
anchor-lang = "1.2"
anchor-spl = "1.2"
raydium-cpmm-cpi = { git = "https://github.com/alex-js-ltd/raydium-cpi", package = "raydium-cpmm-cpi", branch = "anchor-1.2.2" }
```

Call CLMM through CPI.
```toml
raydium-clmm-cpi = { git = "https://github.com/alex-js-ltd/raydium-cpi", package = "raydium-clmm-cpi", branch = "anchor-1.2.2" }
```

Call AMM through CPI.
```toml
raydium-amm-cpi = { git = "https://github.com/alex-js-ltd/raydium-cpi", package = "raydium-amm-cpi", branch = "anchor-1.2.2" }
```

Call Launchpad through CPI.
```toml
raydium-launch-cpi = { git = "https://github.com/alex-js-ltd/raydium-cpi", package = "raydium-launch-cpi", branch = "anchor-1.2.2" }
```

## Note on the `metadata` feature

`cpmm-cpi` previously enabled `anchor-spl`'s `metadata` feature, which it never used. That
feature pulls in `mpl-token-metadata`, which depends on `solana-program 3.0.0`. With that crate
in the dependency graph, `anchor build` installs and activates Solana 3.0.0 — downgrading the
toolchain to platform-tools v1.51 / rustc 1.84.1, below the 1.89.0 that Anchor 1.2.2 programs
require.

`cpmm-cpi` now enables only `memo`, which it does use (`memo_program` in `context.rs`). This
drops `solana-program` from its graph entirely.

The other adapters have been audited. `cpmm-cpi` was the only one enabling `metadata` without
using it:

| crate | uses `metadata` | uses `memo` | declares |
| --- | --- | --- | --- |
| `cpmm-cpi` | no | yes | `memo` |
| `clmm-cpi` | yes | yes | `metadata`, `memo` |
| `amm-cpi` | no | no | neither |
| `launch-cpi` | yes | no | `metadata` |
| `locking-cpi` | yes | yes | `memo`, `metadata` |

`clmm-cpi`, `launch-cpi`, and `locking-cpi` use `anchor_spl::metadata::Metadata` as a
`metadata_program` account, so they need the feature and will pull `solana-program 3.0.0` into
your graph. Depending on any of them reintroduces the toolchain downgrade described above.

`locking-cpi` previously declared only `memo` despite using `Metadata`; it compiled because its
path dependency on `clmm-cpi` enabled the feature and cargo unified it. It now declares both.

Usage examples live in upstream's
[example repository](https://github.com/raydium-io/raydium-cpi-example).
