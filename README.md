# bun install silent exit code 1 with security scanner

Minimal reproduction for [oven-sh/bun#28193](https://github.com/oven-sh/bun/issues/28193).

## Bug

When a security scanner is configured in `bunfig.toml` and the scanner package is a `devDependency`, `bun install` silently exits with code 1 on a clean install (no existing `node_modules`).

No error message is printed:

```
bun install v1.3.10 (30e609e0)
Resolving dependencies
Resolved, downloaded and extracted [6]
Error: Process completed with exit code 1.
```

## Why

Bun tries to load the scanner package after dependency resolution but before writing `node_modules`. Since the scanner is part of the install, it doesn't exist yet — causing a silent `exit(1)` in [`install_with_manager.zig`](https://github.com/oven-sh/bun/blob/5693fc1a98116743f16a7db2952d6e436bf9ec98/src/install/PackageManager/install_with_manager.zig#L565).

This works locally because `node_modules` is cached from a previous install. In CI (clean environment), it always fails.

`BUN_LOG_LEVEL=debug` produces no additional output.

## Reproduce

```bash
git clone https://github.com/renanmav/bun-security-scanner-bug-on-ci.git
cd bun-security-scanner-bug-on-ci
rm -rf node_modules
bun install
# exits with code 1, no error message
```

Or check the [GitHub Actions workflow runs](https://github.com/renanmav/bun-security-scanner-bug-on-ci/actions).

## Workaround

Strip the scanner config before install in CI:

```bash
sed -i '/\[install\.security\]/,/scanner.*/d' bunfig.toml
bun install
git checkout bunfig.toml
```
