# File-by-file Static Triage

Static triage only. This document summarizes defensive findings from file
names, source structure, strings, and static code review. Nothing here requires
or implies execution of the archive.

The table format is intentionally compact so it renders cleanly in GitHub and
Markdown preview panes. Raw grep snippets were removed because they made the
original table unreadable.

## Loader and Bootstrap Files

| File | Role | Static indicators | Defensive note |
| --- | --- | --- | --- |
| `src/utils/selfExtracting.ts` | Self-extracting Bun wrapper | Encrypted payload sections; Bun version pin; wrapper around embedded `Bun.main` content | Treat as a generated dropper wrapper, not ordinary utility code. |
| `src/assets/JS_LOADER.mjs` | Node-side Bun bootstrapper and archive launcher | Platform-specific Bun selection; unzip or PowerShell fallback; npm fallback; `index.js` launch path | Builds a runtime path for payload execution when Bun is missing. |
| `src/assets/BASH_LOADER.sh` | Shell-side Bun bootstrapper | Bun PATH check; curl/wget download path; platform-specific package selection | Shell loader used to stage Bun and run the bundled payload. |
| `src/assets/INJECT_PTH.pth` | Python site-packages loader | `.pth` import hook; `_index.js` lookup; temp marker; Bun handoff | Python startup/import-path mechanism that can trigger embedded JS execution. |
| `src/assets/DEADMAN_SWITCH.sh` | Persistence and token-monitor helper | `gh-token-monitor` config path; token file; curl dependency; service-style install behavior | Persistence-adjacent artifact that should be hunted on developer hosts. |
| `src/assets/GITHUB_MONITOR.py` | GitHub token monitor and fallback discovery helper | Runtime dependency installs; GitHub commit search strings; token monitoring logic | Background monitor/fallback component tied to GitHub token discovery. |
| `src/mutator/persist/install-monitor.ts` | Persistent monitor installer | Writes monitor script; invokes shell install flow; tracks install state | Installs the monitor artifacts as a background component. |

## Package, Repo, and Workflow Mutators

| File | Role | Static indicators | Defensive note |
| --- | --- | --- | --- |
| `src/mutator/npm/tarball.ts` | npm package trojanization | `binding.gyp` command expansion; injected `index.js`; self-extracting payload; optional Bun dependency | High-risk package-install path for npm consumers. |
| `src/mutator/pypi/wheel.ts` | PyPI wheel tampering | `.pth` payload; `_index.js`; wheel rebuild; `RECORD` updates; version bump logic | High-risk Python package startup path through wheel modification. |
| `src/mutator/rubygems/gem.ts` | RubyGems native-extension trojanization | `extconf.rb`; native extension install path; Bun download and payload execution strings | High-risk gem install path through native extension hooks. |
| `src/mutator/npmoidc/injector.ts` | npm OIDC workflow injector | Release workflow replacement; `id-token: write`; setup-bun step; `_index.js` execution | Supply-chain risk through trusted publishing workflow mutation. |
| `src/mutator/action/createEnvelope.ts` | GitHub Action YAML injector | Composite action parsing; setup-bun step insertion; `bun run` step append | Risk of action hijacking through modified action metadata. |
| `src/mutator/repository/lotp.ts` | Repository file injection | Language-specific entrypoint targets; prepended run commands; GitHub content writes | Repo poisoning path across common language ecosystems. |
| `src/mutator/ssh/sshMutator.ts` | SSH propagation helper | `ssh` and `scp` discovery; remote temp directory; loader and JS payload transfer | Lateral-movement path using accessible SSH hosts. |
| `src/mutator/claude/index.ts` | Local developer-tool settings hook | Claude settings search; hook command injection; `~/.config/index.js` payload write | Local tool-configuration abuse targeting developer workflows. |

## Collection and Provider Files

| File | Role | Static indicators | Defensive note |
| --- | --- | --- | --- |
| `src/providers/filesystem/filesystem.ts` | Local filesystem secret harvesting | Paths for SSH, AWS, Kubernetes, cloud caches, wallets, and config files | Broad local secret-discovery surface. |
| `src/providers/vault/vault-secrets.ts` | Vault, Kubernetes, AWS, and generic secret collection | Vault token sources; Kubernetes service-account token regex; AWS/cloud secret regexes; generic secret patterns | Broad credential and secret enumeration provider. |
| `src/providers/actions/workflow.ts` | GitHub Actions secret-dumping workflow provider | Workflow creation paths; branch enumeration; workflow-scope token use; artifact/secret terminology | GitHub Actions abuse path for repository secret exposure. |
| `src/collector/collector.ts` | Token validation, enrichment, and dispatch buffering | GitHub/npm/PyPI/RubyGems/JFrog token handlers; metadata enrichment; discovered-token storage | Central collection hub for discovered credentials. |

## Sender and Exfiltration Files

| File | Role | Static indicators | Defensive note |
| --- | --- | --- | --- |
| `src/sender/base.ts` | Encrypted exfiltration envelope | Compression; AES-GCM; RSA-OAEP; sender acceptance contract | Packaging layer for protected outbound data. |
| `src/sender/domain/domainSenderFactory.ts` | Primary domain sender and fallback discovery | Domain sender factory; fallback/search/verify terminology; `firedalazer` marker | Primary command-and-control style sender path with fallback discovery. |
| `src/sender/github/gitHubSenderFactory.ts` | GitHub-based fallback sender | PAT filtering; repo-scope checks; org/enterprise filtering; repo sender candidate selection | Fallback exfiltration path through GitHub-controlled resources. |

## Short Summary

| Area | Risk concentration |
| --- | --- |
| Loaders and bootstrappers | Multiple paths attempt to stage Bun and launch embedded payloads. |
| Package mutators | npm, PyPI, RubyGems, and OIDC workflows are represented. |
| Repository and workflow mutation | GitHub Actions, action metadata, and repository files are targeted. |
| Collection and sending | Credential discovery, token enrichment, encryption, and sender fallback logic are present. |

## Bottom Line

Highest risk is concentrated in loader/dropper files, package and workflow
mutators, credential collectors, and sender paths. Static evidence alone is
sufficient to treat the archive as hostile and to prioritize containment,
credential rotation, and repository/package audit work.
