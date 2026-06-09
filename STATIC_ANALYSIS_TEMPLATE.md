# Static Analysis Template for Suspicious npm-Style Archives

This template is for defensive analysis only. It focuses on directory enumeration, recursive text search, file hashing, and reference tracing. It does not require executing archive code, installing dependencies, or reconstructing runtime behavior.

## Goal
Rapidly identify high-risk indicators such as:
- install hooks (`preinstall`, `postinstall`, `prepare`)
- Bun launcher or runtime dropper patterns
- `.pth` loaders
- mutator modules
- credential collector strings
- exfiltration or persistence artifacts

## Step 1: Enumerate the archive
Use `find` to map the top-level layout and surface suspicious directories and filenames.

```bash
find /path/to/extracted/archive -maxdepth 2 -type f | sort
find /path/to/extracted/archive -maxdepth 3 -type d | sort
```

What to flag immediately:
- `binding.gyp`
- `scripts/`
- `src/mutator/`
- `src/providers/`
- `src/sender/`
- `src/assets/`
- loader names like `*.sh`, `*.py`, `*.ps1`, `*.mjs`, `*.pth`

## Step 2: Hash important files
Record SHA-256 hashes for major files so you can cite them later and detect duplicate or recycled payloads.

```bash
sha256sum README.md package.json binding.gyp src/**/*.ts src/**/*.js src/**/*.py src/**/*.sh
```

What to flag:
- unusually large scripts
- multiple similarly named loader files
- generated or packed asset blobs

## Step 3: Search for install-time execution hooks
Look for package lifecycle hooks and installer-triggered commands.

```bash
grep -RniE 'preinstall|postinstall|prepare|install|binding\.gyp|node-gyp|npm run|bun run' .
```

What to flag:
- lifecycle scripts in `package.json`
- wrapper scripts that launch Bun, Node, Python, or shell
- build steps that download, unpack, or execute payloads

## Step 4: Search for Bun launcher patterns
Look for references to Bun runtime startup or launcher handoff.

```bash
grep -RniE 'bun|Bun|bunx|BUN_INSTALL|bunfig|runtime dropper|launcher|bootstrap' .
```

What to flag:
- hardcoded Bun binary assumptions
- shell or JS wrappers that invoke Bun
- runtime checks that decide which loader to use

## Step 5: Search for loader and dropper artifacts
Look for files that inject execution via standard language loaders.

```bash
grep -RniE '\.pth|sitecustomize|usercustomize|require\(|import\s+|spawn\(|exec\(|child_process|os\.system|subprocess' .
```

What to flag:
- `.pth` files for Python execution on import
- JS `child_process` calls
- shell wrappers that chain into other interpreters

## Step 6: Search for collectors and exfiltration logic
Look for credential harvesting and transport keywords.

```bash
grep -RniE 'token|secret|password|credential|cookie|ssh|aws|vault|kube|github_pat_|pypi-|xox|oauth|upload\.pypi\.org|api\.anthropic\.com' .
```

What to flag:
- explicit token validation paths
- cloud provider and password manager enumeration
- upload endpoints and C2-looking domain references

## Step 7: Search for mutator and propagation modules
Look for modules that modify packages, repos, or workflows.

```bash
grep -RniE 'mutator|trojan|inject|republish|publish|workflow|action|branch|tag|repo|fork|oidc|trusted publishing' src .
```

What to flag:
- package republishing logic
- GitHub Action or workflow modification
- OIDC-based publishing or branch injection

## Step 8: Trace references across files
Use grep to follow filenames, helper names, and asset loads from one file to another.

```bash
grep -RniE 'BASH_LOADER|JS_LOADER|PYTHON_LOADER|DUMP_LINUX|GITHUB_MONITOR|DEADMAN_SWITCH|INJECT_PTH|workflow\.yml|enc_key\.pub|verify_key\.pub' .
```

What to flag:
- asset loads from `src/assets/`
- script references that connect build steps to runtime execution
- repeated names across README, architecture docs, and source code

## Step 9: Build a simple reference chain
For a suspicious file, note:
1. where it is discovered
2. what strings it contains
3. which other files mention it
4. what the surrounding docs say it does
5. whether it appears in install/build/runtime paths

This is often enough to reconstruct the high-level payload flow without parsing the code deeply.

## Step 10: Document defender-focused findings
Capture:
- file paths
- exact strings
- hashes
- likely tactic categories
- recommended detection ideas
- any limits of static-only review

## Example indicators that should raise alarms
- `preinstall`, `postinstall`, `prepare`
- `bun run`, `bunfig`, `Bun`
- `.pth`
- `workflow.yml`
- `GITHUB_MONITOR.py`
- `DEADMAN_SWITCH.sh`
- `upload.pypi.org/legacy`
- `firedalazer`
- `github_pat_`
- `pypi-`
- `VAULT_TOKEN`
- `aws_secret_access_key`
- `ssh-rsa`

## Safe reporting language
Use phrasing like:
- “Static inspection shows…”
- “The file contains references to…”
- “The archive appears designed to…”
- “No code was executed during analysis.”

Avoid operational detail, payload instructions, or reproduction steps.

