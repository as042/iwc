# Whole Genome Alignment Workflows

This directory contains Galaxy workflows for building a whole genome alignment (WGA) of multiple
genomes. The pipeline computes a pairwise distance matrix to choose an appropriate aligner for
each genome pair, aligns the genomes, builds a guide tree, and combines everything into a
multiple genome alignment.

> **Status: work in progress.** Two of the workflows below are implemented (adapted from a
> KegAlign-only pipeline to support both KegAlign and FastGA); the rest are planned. Several step
> connections are still being finalized — see the notes per workflow.

## Pipeline overview

1. **Distance matrix** — `dashing2` produces a pairwise distance matrix over the input genomes,
   used to decide which aligner (FastGA or KegAlign) to use for each pair. _(planned)_
2. **Pairwise alignment + liftover** — KegAlign and/or FastGA align a genome pair, followed by
   chaining/netting into liftover chains. _(implemented, WIP)_
3. **Neighbor-joining tree** — the distance matrix is turned into a guide tree. _(planned)_
4. **Multiple sequence alignment** — the pairwise alignments are combined into a final
   multiple genome alignment. _(planned)_
5. **Top-level orchestrating workflow** — chains the steps above end to end. _(planned)_

## Workflows

### WGA: pairwise align and liftover (`wga-pairwise-align-and-liftover.ga`) — implemented (WIP)

Aligns a single TARGET/QUERY pair and builds liftover chains in **both** directions
(normal and reverse). Two aligners are present:

- **KegAlign → Batched LASTZ → axtChain** — wired through to the `wga-chain-to-liftover`
  subworkflow. *(KegAlign's own TARGET/QUERY inputs are not yet connected — in progress.)*
- **FastGA** — present and produces a PAF alignment, but **intentionally not yet connected to
  axtChain**. The axtChain version currently on usegalaxy cannot consume FastGA's output; this
  branch will be wired once the required axtChain update is installed (see
  [Notes on FastGA → axtChain](#notes-on-fastga--axtchain)).

- **Inputs:** `TARGET_Sequence`, `QUERY_Sequence` (masked FASTA).
- **Outputs:** `liftover_chain_file_normal`, `liftover_chain_file_reverse`, plus intermediate
  aligner/chain outputs.

This is the WorkflowHub "main" workflow for the folder.

### WGA: chain to liftover (`wga-chain-to-liftover.ga`) — implemented

Subworkflow used by the workflow above. Takes a pairwise alignment (from Batched LASTZ) and a
chain, converts the genomes to 2bit, computes sequence lengths, and runs the UCSC chain/net
toolchain (chainAntiRepeat → chainPreNet → chainSort → chainNet → netSyntenic → netFilter →
netChainSubset) to produce a net-filtered liftover chain (`liftover_chain_file`).

### Distance Matrix (`wga-distance-matrix.ga`) — planned

Will compute a pairwise distance matrix with `dashing2`, used to select the aligner per pair.

### Neighbor-Joining Tree (`wga-neighbor-joining-tree.ga`) — planned

Will build a neighbor-joining guide tree from the pairwise distance matrix.

### Multiple Sequence Alignment (`wga-msa.ga`) — planned

Will combine the pairwise alignments into a final multiple genome alignment.

## Notes on FastGA → axtChain

FastGA emits PAF (or a binary `.1aln`), which the axtChain build currently on usegalaxy
(`ucsc_axtchain/482+galaxy1`) cannot read — it expects AXT. Until an updated axtChain that
accepts FastGA's output is available, the FastGA branch is left disconnected from axtChain. When
revisiting, note that FastGA's native chaining path is `ALNchain` on `.1aln` files
(`tools-iuc/tools/fastga/alnchain.xml`), which may be the better route than axtChain.

## Test Data

The `test-data/` directory holds the toy inputs used by the Planemo test files. Use the smallest
inputs that still exercise each workflow. For large or non-deterministic outputs, prefer
assertion-based tests over committing full output files. The current `*-tests.yml` files are
stubs with `TODO` paths to be filled in once the workflows are finalized.

## Running Tests

Each `.ga` has a matching `<name>-tests.yml` ([Planemo test format](https://planemo.readthedocs.io/en/latest/test_format.html)).
To lint a workflow and its test file:

```bash
planemo workflow_lint wga-pairwise-align-and-liftover.ga
```

To run the tests against a Galaxy instance that has the required tools installed:

```bash
planemo test --galaxy_url <galaxy_server> --galaxy_user_key <api_key> wga-pairwise-align-and-liftover.ga
```

## References

- FastGA: https://github.com/thegenemyers/FASTGA
- KegAlign: https://github.com/galaxyproject/KegAlign
- UCSC chain/net tools (Kent utilities): https://genome.ucsc.edu/goldenPath/help/chain.html
- dashing2: Baker & Langmead, _Genome Research_ (2023), https://doi.org/10.1101/gr.277655.123
- Neighbor-joining tool: _(TODO)_
- Multiple sequence aligner: _(TODO)_
