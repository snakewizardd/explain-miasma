# Additional Static Findings on Miasma

## What stands out most
The funniest part from a malware-analysis standpoint is how openly self-aware the project is. The docs read like a brag sheet for a worm, and the source tree follows through with matching modules for almost every stage of the chain: harvest, validate, wrap, exfiltrate, persist, mutate, and reinfect.

## Most notable static findings
### 1. The project is explicit about being a worm
The README and architecture docs do not hide intent. They describe:
- credential theft
- supply-chain propagation
- encrypted exfiltration
- fallback C2 discovery
- repo and package reinfection

This is not a subtle “dual use” tool; it is labeled and engineered like a spreader.

### 2. Bun is the common handoff point
A recurring design choice is to use Bun as the runtime pivot. Multiple files show the same pattern:
- detect or download Bun
- wrap the payload around Bun.main content
- launch Bun through shell, JS, Python, Ruby, or workflow hooks

That makes the project easy to reason about statically because the runtime handoff is repeated in many ecosystems.

### 3. The archive is layered but repetitive
The same core ideas show up over and over:
- a loader or hook file
- a self-extracting or self-wrapping payload
- a platform-specific execution path
- a persistence or propagation step
- a source of credentials or repo access

That repetition is useful for defenders because once you recognize the pattern in one language, you can hunt for it everywhere else.

### 4. Install-time and hook-time execution are central
The most important static clues are not in any single “payload” file, but in the places where code is meant to run automatically:
- npm `preinstall` / `binding.gyp`
- RubyGems `extconf.rb`
- PyPI `.pth`
- GitHub Actions workflows
- Claude / editor settings hooks
- SSH propagation scripts
- persistent service installers

The archive is designed to get code execution at trust boundaries.

### 5. The loader files are almost comically direct
Examples from static inspection:
- `src/assets/BASH_LOADER.sh` downloads Bun if needed
- `src/assets/JS_LOADER.mjs` falls back to `npm install bun`
- `src/assets/INJECT_PTH.pth` searches for `_index.js` and runs Bun
- `src/mutator/rubygems/gem.ts` writes an `extconf.rb` that downloads Bun during install

Those are unmistakable handoff points.

### 6. There are lots of “supporting” pieces that are actually the payload chain
Files like `collector.ts`, `sender/base.ts`, `domainSenderFactory.ts`, and `gitHubSenderFactory.ts` are not side utilities. They form the core chain:
- collect secrets
- validate and enrich them
- encrypt them
- send them out through primary or fallback transports

### 7. The fallback logic is as interesting as the primary logic
The archive keeps trying to survive:
- primary domain sender
- GitHub repo sender
- commit-search fallback via signed messages
- host-level persistence
- background monitors and dead-man switches

That means even if one path is blocked, there are several others.

## Defenders should care about
- package install hooks
- workflow changes
- republished packages with hooks or loaders
- repo branch/tag mutations
- `.pth` or `extconf.rb` artifacts
- token collection from cloud, password, and developer tooling paths
- any GitHub commit search around `firedalazer`

## Simple static takeaway
The project is funny in the same way an overly elaborate heist blueprint is funny: it is so verbose, modular, and self-congratulatory that the intent is exposed everywhere. The whole archive is built to spread through trusted developer workflows.
