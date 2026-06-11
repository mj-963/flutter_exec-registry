# flutter_exec package registry

This directory is the **package registry** for the `pkg` command. It is a static
site: a JSON index plus (optionally) per-package files. The app fetches
`index.json`, and downloads the binary artifacts it references.

```
registry/
├── index.json     ← master package list (schema 1)
├── sources.list   ← default community sources (one index.json URL per line)
└── README.md      ← this file
scripts/
├── build_pkg_jq.sh        ← cross-compiles jq (Android ELF + iOS WASI wasm)
├── build_pkg_fastfetch.sh ← cross-compiles fastfetch (Android ELF only)
└── release_pkg.sh         ← uploads built artifacts to the registry Releases (gh)
```

## Hosting (resolved)

Artifacts are served over **anonymous HTTPS** from the public companion repo
[`flutter_exec-registry`](https://github.com/mj-963/flutter_exec-registry):

| Asset | Where | URL |
|---|---|---|
| `index.json` | committed to the public repo | `https://raw.githubusercontent.com/mj-963/flutter_exec-registry/main/index.json` |
| Binaries | that repo's **Releases** (tag `pkgs-v1`) | `…/releases/download/pkgs-v1/<artifact>` |

The main plugin repo stays **private**; only the compiled binaries + index are
public. `PackageManager.defaultSources` points at the raw `index.json` URL
above (live immediately — no Pages setup needed). Enabling GitHub Pages on the
registry repo later is an optional CDN upgrade; the raw URL works fine.

The plugin treats the registry as a single configurable URL
(`PackageManager.defaultSources`, or `pkg source add <url>`), so swapping hosts
is a one-line change — no code beyond the constant.

## Schema (`index.json`)

```jsonc
{
  "schema": 1,
  "updated": "2026-06-07",
  "packages": [
    {
      "name": "jq",
      "version": "1.7.1",
      "description": "Command-line JSON processor",
      "homepage": "https://jqlang.github.io/jq/",
      "platforms": {
        "android": {
          "arm64-v8a":   { "url": "…/jq-arm64-v8a",   "sha256": "…", "size": 812304 },
          "armeabi-v7a": { "url": "…/jq-armeabi-v7a", "sha256": "…", "size": 701234 },
          "x86_64":      { "url": "…/jq-x86_64",      "sha256": "…", "size": 905120 }
        },
        "ios": { "url": "…/jq.wasm", "sha256": "…", "size": 1203440 }
      }
    }
  ]
}
```

- `sha256` is **mandatory** — the app rejects a download whose hash doesn't match.
- Android artifacts are native ELF, keyed by ABI. iOS artifacts are a single
  WASI `.wasm` (iOS can't exec native binaries in the sandbox), so native-only
  tools are simply omitted from the `ios` key (and show as "android only").

## Per-package notes

| Package | Android | iOS | Notes |
|---|---|---|---|
| `jq` 1.7.1 | ✅ 3 ABIs | ✅ WASI wasm | Pure compute, ports cleanly to WASI. Kept as the reference package — it exercises the custom pipeline on both platforms (and is the only iOS path, since Termux is Android-only). |

> **Android-native tools are no longer curated here.** With the Termux apt repo
> wired in (`PackageManager.defaultSources`), ~2800 Android packages —
> `fastfetch` included — install straight from Termux. The custom index now
> carries only things Termux can't serve: iOS WASI builds, and `jq` as a
> cross-platform demonstrator. `scripts/build_pkg_fastfetch.sh` stays for
> reference but its artifact is dropped from `index.json`.

**WASI cross-compile gotcha (macOS):** force the wasi-sdk's archive tools —
`AR=$WASI_SDK_PATH/bin/llvm-ar RANLIB=…/llvm-ranlib NM=…/llvm-nm`. macOS's
default `ar`/`ranlib` write a BSD-style archive symbol table that `wasm-ld`
crashes on (in `readULEB128`). `build_pkg_jq.sh` already does this.

## Publishing a package

1. Build artifacts — see `scripts/build_pkg_<name>.sh` (jq, fastfetch provided).
2. Upload them to the registry Releases: `bash scripts/release_pkg.sh pkgs-v1 <files…>`
   (uses `gh`; the token lives in your keychain/CI, never in the app).
3. Put the resulting URLs + `sha256` + `size` into `index.json`.
4. Commit `index.json` to the public registry repo (the app reads it via raw URL).
