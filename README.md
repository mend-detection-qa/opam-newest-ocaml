# opam-newest-ocaml

## Purpose

Mend SCA detection probe targeting the newest-compiler/newest-dep coverage pattern.
Forces the opam resolver to select the latest OCaml 5.x compiler (`>= 5.2`, resolves to
`5.6.0` as of 2026-05-27) and the newest stable versions of three widely-used ecosystem
packages. This exposes whether Mend's UA correctly identifies all transitive deps that
ship with a modern compiler switch.

## Feature exercised

Compiler-line pin via `ocaml >= 5.2` combined with newest-stable direct deps, exercising
the resolver's ability to walk the full transitive closure of `eio 1.3` (which itself
requires `ocaml >= 5.2.0`) through its build-time and runtime sub-graph.

## Package selection rationale

| Package | Version | Why chosen |
|---|---|---|
| `dune` | 3.23.1 | Standard build system; newest stable; always present in any switch |
| `eio` | 1.3 | Effect-based IO; requires `ocaml >= 5.2.0` — strongest OCaml 5 signal; pulls in the richest transitive sub-graph of any single library |
| `cmdliner` | 2.1.1 | CLI argument parser; newest stable; zero runtime deps beyond `ocaml` — pure leaf node, validates Mend reports single-dep packages cleanly |

## Expected dependency tree

### Direct dependencies (4)

| artifactId | version | constraint in manifest |
|---|---|---|
| `ocaml` | 5.6.0 | `>= 5.2` |
| `dune` | 3.23.1 | `>= 3.23.1` |
| `eio` | 1.3 | `>= 1.3` |
| `cmdliner` | 2.1.1 | `>= 2.1.1` |

### Transitive dependencies (16)

Pulled in entirely via `eio 1.3` and `fmt 0.11.0`:

| artifactId | version | via |
|---|---|---|
| `bigstringaf` | 0.10.0 | eio → bigstringaf |
| `cstruct` | 6.2.0 | eio → cstruct |
| `domain-local-await` | 1.0.1 | eio → domain-local-await |
| `dune-configurator` | 3.23.1 | bigstringaf (build) |
| `fmt` | 0.11.0 | eio → fmt |
| `hmap` | 0.8.1 | eio → hmap |
| `lwt-dllist` | 1.1.0 | eio → lwt-dllist |
| `mtime` | 2.1.0 | eio → mtime |
| `ocamlbuild` | 0.16.1 | fmt (build), mtime (build) |
| `ocamlfind` | 1.9.8 | fmt (build), mtime (build) |
| `optint` | 0.3.0 | eio → optint |
| `psq` | 0.2.1 | eio → psq |
| `thread-table` | 1.0.0 | domain-local-await → thread-table |
| `topkg` | 1.1.1 | fmt (build), mtime (build) |
| `base-threads` | base | ocaml switch virtual |
| `base-unix` | base | ocaml switch virtual |

## Mend config

No `.whitesource` file is emitted. The UA resolver for OCaml is not in the
`install-tool` versioning list and opam has no `scanSettings.versioning` key.
Detection is purely manifest-driven (`*.opam`). The UA must have `ocaml.resolveDependencies=true`
set in `whitesource.config` (or via UA auto-recommendation when `.opam` is found)
and `ocaml.runPreStep=false` (default) since we do not run `opam install` as a
pre-step — Mend reads the manifest directly.

## Probe metadata

```json
{
  "probe_id": "opam/newest-compiler-20260527-111245",
  "pattern": "newest-compiler",
  "pm": "opam",
  "generated_at": "2026-05-27T11:12:45Z",
  "pm_version_tested": null,
  "ocaml_constraint": ">= 5.2",
  "resolved_ocaml_version": "5.6.0",
  "target": "remote",
  "remote_url": "https://github.com/mend-detection-qa/opam-newest-ocaml"
}
```