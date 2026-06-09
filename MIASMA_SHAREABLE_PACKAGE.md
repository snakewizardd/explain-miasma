---
classification: Defensive static analysis only
scope: No execution / no network / no token validation
audience: Security leadership, incident response, supply-chain defense
---
# Miasma Shareable Defensive Package
## Executive summary
Miasma is a multi-stage malware framework described in its own documentation as a worm for credential theft, exfiltration, persistence, and propagation across developer ecosystems. Static inspection shows broad coverage of GitHub, package registries, cloud services, Kubernetes, SSH, and local developer tooling. The archive contains modules for token harvesting, encrypted exfiltration, repository and package tampering, and workflow or hook injection.
## What this is
- A worm-like malware project in TypeScript, executed via Bun, with accompanying build and utility scripts.
- The documentation explicitly describes exfiltration, propagation, and supply-chain abuse.
- The codebase includes collectors, mutators, senders, and supporting scripts for persistence and reinfection.
## Why it matters
- It targets high-value credentials and secrets used in source control, CI/CD, and cloud environments.
- It can use stolen access to spread into packages, repositories, workflows, and hosts.
- The archive includes mechanisms for encrypted transport and fallback discovery of C2-related infrastructure.
## Affected ecosystems
- GitHub repositories and GitHub Actions
- npm packages
- PyPI packages and Trusted Publishing/OIDC
- RubyGems and RubyGems OIDC
- JFrog Artifactory npm repositories
- HashiCorp Vault
- AWS, Azure, and GCP
- Kubernetes clusters and service accounts
- SSH hosts and developer workstations
- Password managers and runner processes
- Claude Code / local developer settings
## Most dangerous capabilities
- Secret harvesting from files, environment variables, cloud metadata, runners, and password stores
- Token validation and metadata enrichment for GitHub, npm, PyPI, RubyGems, and JFrog
- Encrypted exfiltration over HTTPS or GitHub-based fallback paths
- Repository poisoning, branch and tag mutation, and payload injection
- Package trojanization and workflow manipulation
- OIDC-based supply-chain injection for package publishing
- SSH propagation and background persistence helpers
## Observed indicators
### Names and strings
- `GITHUB_MONITOR.py`
- `DEADMAN_SWITCH.sh`
- `INJECT_PTH.pth`
- `workflow.yml`
- `enc_key.pub`
- `verify_key.pub`
- `firedalazer`
- `DontRevokeOrItGoesBoom`
- `TheBeautifulSandsOfTime`
- `thebeautifulmarchoftime`
- `api.anthropic.com`
- `v1/api`
- `github_pat_`
- `pypi-`
- `hvs.`
- `xox`
- `VAULT_TOKEN`
- `upload.pypi.org/legacy`
### Static hashes
- `README.md` — `5c68c39bd97369ea2c7c808bcead11f2b62e32eb40dd260744259032f3f2b686`
- `ARCHITECTURE.MD` — `64f0876cd29afd8719962d83377bb74b290c9c8932f1fbcac19dc03fc331c729`
- `src/collector/collector.ts` — `bc506fe456bd873e29434b2beae95961141d1222444695641a29edb1c51e060b`
- `src/providers/vault/vault-secrets.ts` — `42a5ee9936da0da096821c640ce294cae34f0532144a727f001a2c5dda7ba601`
- `src/sender/base.ts` — `4f6199bbad461f069827560cfbf5db961db3501c5c6d91f1d0f01c64255e5445`
## File-by-file capability index
### High-risk source files
#### `src/collector/collector.ts`
This file validates and retains tokens and enriches metadata.
Static citations from appendix grep:
- `handleGhTokens`, `handleFgGhTokens`, `handleNpmTokens`, `handleRubygemsTokens`, `handlePypiTokens`
- `upload.pypi.org`
- `tokenMetadata`
- `discoveredTokens`
- `jfrogtoken`
- `jfrogreftoken`
#### `src/providers/vault/vault-secrets.ts`
This file is built to discover Vault tokens and multiple secret types.
Static citations from appendix grep:
- `VAULT_TOKEN`
- `VAULT_AUTH_TOKEN`
- `VAULT_API_TOKEN`
- `VAULT_TOKEN_PATH`
- `kubernetes.io/serviceaccount/token`
- `aws_access_key_id`
- `aws_secret_access_key`
- `privateKey`
- `sshKey`
- `genericSecret`
- `base64Blob`
#### `src/sender/base.ts`
This file handles encrypted packaging before transport.
Static citations from appendix grep:
- `gzip`
- `AES-256-GCM`
- `RSA-OAEP`
- `encrypt`
- `bundle`
- `send`
#### `src/sender/domain/domainSenderFactory.ts`
This file implements the domain sender and fallback discovery.
Static citations from appendix grep:
- `firedalazer`
- `commit`
- `search`
- `verify`
- `fallback`
- `DOMAIN`
#### `src/mutator/npm/index.ts`
This file is associated with package trojanization.
Static citations from appendix grep:
- `preinstall`
- `tarball`
- `publish`
- `token`
- `inject`
#### `src/mutator/pypi/index.ts`
This file is associated with wheel modification and republishing.
Static citations from appendix grep:
- `pth`
- `wheel`
- `upload`
- `inject`
- `publish`
- `macaroon`
#### `src/mutator/rubygems/index.ts`
This file is associated with gem modification and republishing.
Static citations from appendix grep:
- `gem`
- `publish`
- `native`
- `inject`
- `OIDC`
#### `src/mutator/action/actionMutator.ts`
This file is associated with GitHub Action hijacking.
Static citations from appendix grep:
- `action`
- `tag`
- `force`
- `checkout`
- `inject`
- `composite`
- `workflow`
#### `src/mutator/npmoidc/index.ts`
This file is associated with OIDC-based supply-chain injection.
Static citations from appendix grep:
- `OIDC`
- `workflow`
- `tool`
- `inject`
- `provenance`
- `sigstore`
#### `src/mutator/rubygemsoidc/index.ts`
This file is associated with RubyGems OIDC workflow manipulation.
Static citations from appendix grep:
- `OIDC`
- `workflow`
- `inject`
- `branch`
- `provenance`
- `detector`
#### `src/providers/actions/workflow.ts`
This file is associated with secret-dumping workflow operations.
Static citations from appendix grep:
- `workflow`
- `artifact`
- `secret`
- `dispatch`
- `poll`
- `download`
## MITRE ATT&CK-style defender mapping
- Credential Access: token harvesting, cloud credential discovery, password manager access
- Discovery: filesystem, repository, cloud, package registry, and cluster enumeration
- Persistence: workflows, hooks, monitors, and dead-man-switch style mechanisms
- Defense Evasion: obfuscation, encrypted payloads, selective token retention
- Exfiltration: HTTPS transport and GitHub-based fallback transport
- Supply Chain Compromise: package trojanization, action hijacking, repo poisoning
- Lateral Movement: SSH propagation and credential reuse across systems
## Immediate containment recommendations
- Quarantine the archive and extracted copies.
- Do not execute any scripts or package workflows from the archive.
- Rotate potentially exposed GitHub, npm, PyPI, RubyGems, JFrog, Vault, cloud, and SSH credentials.
- Audit repositories, Actions workflows, and package publishing history for unauthorized changes.
- Check developer hosts for persistence artifacts and access to secret stores.
- Preserve logs and artifacts for incident response and scoping.
## Defensive detection priorities
- Alert on orphan commits, unusual public repo creation, and commit messages containing `firedalazer`.
- Monitor for GitHub Actions workflow edits that dump secrets or inject build steps.
- Detect reads of `~/.ssh`, `~/.aws`, `~/.kube`, Vault token locations, password managers, and runner memory.
- Watch for outbound requests to `api.anthropic.com/v1/api` and `upload.pypi.org/legacy`.
- Flag package publish events that introduce preinstall hooks, `.pth` loaders, workflow files, or other injection artifacts.
- Review branch force-pushes, tag mutations, and repository file injection patterns.
- Look for persistence artifacts such as `GITHUB_MONITOR.py` and `DEADMAN_SWITCH.sh`.
## Confidence and limitations
### Directly evidenced
- The archive documentation explicitly describes worm behavior, exfiltration, persistence, and propagation.
- The static triage report and appendix contain cited evidence from specific files and line-oriented grep output.
- Named files, strings, and module paths listed above are present in the archive.
### Inferred
- The exact real-world success rate of each capability is not proven by static analysis alone.
- The operational impact depends on execution environment, credentials available, and target defenses.
- Live network behavior and token validity were not tested.
## Final statement
All conclusions in this package are based on static review only. Live effectiveness was not tested.
