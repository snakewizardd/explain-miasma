# explain-miasma

This repository is a defensive explanation package for the Miasma worm archive.
It was generated in a safe sandbox on June 8, 2026 by snakewizardd, goblin, and
GPT 5.5 working as a team.

## Provenance

The analysis was produced by goblin CLI after inspecting the extracted contents
of the published Miasma zip archive. The archive was analyzed statically: files,
strings, structure, documentation, and source code were reviewed, but the worm
was not executed.

The team setup was:

- snakewizardd: human direction, review, and release context.
- goblin: the safe hands, operating the sandboxed CLI workflow and keeping the
  work constrained to static analysis.
- GPT 5.4 mini: the model under the hood for goblin CLI.
- GPT 5.5 with extra thinking: additional reasoning and synthesis support.

## Safety boundary

This repo is for defensive understanding, incident response, and security
communication. It should not be treated as a runnable malware workspace.

The analysis intentionally avoided:

- Running the worm or its scripts.
- Validating tokens or credentials.
- Contacting command-and-control, package registry, cloud, or GitHub endpoints.
- Reconstructing or testing propagation behavior.
- Publishing, building, or installing anything from the archive.

## Contents

- `MIASMA_SHAREABLE_PACKAGE.md`: a shareable defensive summary of the static
  findings, affected ecosystems, indicators, capability mapping, containment
  guidance, and limitations.

## Intended use

Use this repository to brief defenders, security leadership, incident response
teams, and supply-chain owners on what the Miasma archive appears designed to
do based on static evidence.

All conclusions are bounded by static analysis. Live effectiveness, token
validity, network behavior, and real-world operational impact were not tested.
