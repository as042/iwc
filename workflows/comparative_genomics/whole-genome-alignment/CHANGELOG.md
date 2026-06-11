# Changelog

## [0.1] - 2026-06-11

### Added
- Initial scaffold of the whole-genome-alignment workflow repository.
- `wga-pairwise-align-and-liftover` workflow (adapted from a KegAlign-only "FastGA 2x liftover"
  pipeline by Penn State University / Martin Čech): aligns a TARGET/QUERY pair and builds
  liftover chains in both directions, with both KegAlign and FastGA aligners present.
  - Wired the KegAlign branch (Batched LASTZ output) into axtChain.
  - FastGA is present but intentionally left disconnected from axtChain pending an axtChain
    update on usegalaxy that can consume FastGA output.
- `wga-chain-to-liftover` subworkflow (adapted from "chain to liftover"): runs the UCSC chain/net
  toolchain to produce a net-filtered liftover chain.
