# flutter_exec-registry

Public package registry for the [`flutter_exec`](https://github.com/mj-963/flutter_exec)
Flutter plugin's `pkg` command.

- **`index.json`** — the master package list (schema 1). Served via GitHub Pages
  at `https://mj-963.github.io/flutter_exec-registry/index.json`.
- **Binaries** — attached to this repo's [Releases](../../releases). Android
  artifacts are native ELF (per ABI); iOS artifacts are WASI `.wasm`.

Only the compiled binaries and this index are public; the plugin's source stays
in its (private) repo. Downloads are anonymous — no token required.

## Schema

```jsonc
{
  "schema": 1,
  "updated": "YYYY-MM-DD",
  "packages": [
    {
      "name": "jq",
      "version": "1.7.1",
      "description": "Command-line JSON processor",
      "homepage": "https://jqlang.github.io/jq/",
      "platforms": {
        "android": {
          "arm64-v8a":   { "url": "…", "sha256": "…", "size": 0 },
          "armeabi-v7a": { "url": "…", "sha256": "…", "size": 0 },
          "x86_64":      { "url": "…", "sha256": "…", "size": 0 }
        },
        "ios": { "url": "…", "sha256": "…", "size": 0 }
      }
    }
  ]
}
```

`sha256` is mandatory — the client rejects any download whose hash doesn't match.

## Publishing

Built in the plugin repo via `scripts/build_pkg_<name>.sh`, which prints the
`sha256`/`size` for each artifact. Upload the artifacts to a Release here, paste
the URLs + hashes into `index.json`, and commit.
