# ubx-sdk-azure-py

Python bindings for the `hashicorp/azurerm` Terraform provider, generated
by [ubx](https://github.com/ubiquex/ubiquex) (`ubx sdk gen --lang py`). One
directory per derived Azure service boundary (`kubernetes/`, `network/`,
`storage/`, ...), one file per resource type, one `__init__.py` per service
directory (also the real Python package marker), all nested under
`azurerm/` (UBI-106 shape) — matches the real Terraform provider source,
`hashicorp/azurerm`, exactly.

Regenerated automatically on a weekly schedule (and via manual dispatch) by
`.github/workflows/version-watch.yml` — it checks the real
[Terraform Registry provider-versions API](https://registry.terraform.io/v1/providers/hashicorp/azurerm/versions)
for a newer `hashicorp/azurerm` release, and opens a PR with the
regenerated diff for review (never auto-merges). The version this repo was
last generated from is tracked in `VERSION`, not `.ubx/config.hcl` — this
repo carries no ubx stack/config of its own, only generated bindings.

**Shared runtime (`ubx_sdk`)**: a real, published PyPI package —
[ubx-sdk](https://pypi.org/project/ubx-sdk/) (import name `ubx_sdk`,
UBI-107) — declared as an ordinary dependency in `pyproject.toml`. This
repo previously vendored a local copy at `vendor/ubx_sdk/` before the
real publish existed — that vendored copy is gone as of the switch to
the real package.

**Python sibling to `ubx-sdk-azure-go` (UBI-115) / `ubx-sdk-azure-ts`
(UBI-116) — real findings, checked directly for Python, not assumed to
match Go/TS by analogy:**

- **A real, pre-emptively fixed bug — checked BEFORE the first dispatch,
  not discovered via a bad PR diff (UBI-116's own lesson)**: `ubx sdk
  gen`'s mechanical shortName derivation (the provider source's own last
  `/`-segment, `"hashicorp/azurerm"` → `"azurerm"`) writes
  `pyproject.toml`'s `name` as `ubx-sdk-azurerm` on every generation —
  confirmed directly by running the generator locally before touching
  this repo. Azure is the first provider in this family whose repo
  shortname (`azure`) diverges from its mechanically-derived one
  (`azurerm`) — UBI-116 found and fixed the identical class of bug in
  `ubx-sdk-azure-ts`'s `package.json` `name` field, and UBI-115 in
  `ubx-sdk-azure-go`'s `go.mod` module path; this repo's `pyproject.toml`
  carries the same latent risk and is corrected the same way, both in
  this initial commit and in `version-watch.yml`'s regeneration step
  (a `sed` correction applied every run, since `pyproject.toml`'s
  `description` field legitimately needs the live version-stamp update
  and so can't just be excluded from regen like `VERSION`).
- **The ticket's own real ask — UBI-115/116's v5-protocol finding
  confirmed for Python's own generation path too, checked directly not
  assumed**: `hashicorp/azurerm@5.0.0` negotiates tfplugin **protocol
  v5** for `--lang py` generation too — reconfirmed via a raw handshake
  against the real provider binary (`TF_PLUGIN_MAGIC_COOKIE` +
  `PLUGIN_PROTOCOL_VERSIONS=6,5`, response `1|5|unix|...|grpc`), the same
  direct method used for the earlier AWS/Google binaries. Expected, since
  protocol negotiation happens once in `provider/` — shared,
  language-independent code that runs identically regardless of
  `--lang` — but checked live rather than assumed. Kubernetes remains
  the only v6 provider in this whole rollout; v5 is the norm across
  AWS/Google/Azure (UBI-116's own correction).
- **Real schema scale, confirmed independently**: **1,103 resource
  types** for `hashicorp/azurerm@5.0.0`, exactly matching Go's and TS's
  own counts — 144 derived service packages, 1,247 files, real recursive
  `importlib.import_module` of every generated module, zero errors
  (largest file `azurerm/kubernetes/cluster.py`, 1,060 lines — no
  compiler-crash-class issue).
- **Sibling-`_config` collision (UBI-96/108) and bare-version-suffix
  collision (UBI-112)**: both wire-name-level facts, language-independent
  — UBI-115's own Go-side check (0/1,103 candidates) carries over
  unchanged, reconfirmed directly here (0/1,103 candidates against the
  real generated local-name list; zero bare `v<N>.py`-style filenames
  anywhere in the tree).
- **Zero `ubiquex`-core changes needed this session.**

Every file under a service directory except `__init__.py` is generated —
do not hand-edit; re-run `ubx sdk gen` (or wait for the automated PR)
after a provider version bump.

This completes [UBI-103](https://github.com/ubiquex/ubiquex)'s original
four-provider (AWS/Google/Kubernetes/Azure), three-language rollout.
