# ubx-sdk-azure-py

Python bindings for the `hashicorp/azurerm` Terraform provider, generated
by [ubx](https://github.com/ubiquex/ubiquex) (`ubx sdk gen --lang py`),
published to PyPI as [ubx-sdk-azure](https://pypi.org/project/ubx-sdk-azure/).

Real, `pip install`-able namespace-package layout, matching real, strong
precedent (`google.cloud.*`, `azure.mgmt.*`): every package nests under
the shared `ubx` namespace, one directory per derived Azure service
boundary below that (`ubx/azurerm/kubernetes/`, `ubx/azurerm/network/`,
`ubx/azurerm/storage/`, ... — `azurerm/`, not `azure/`, matching the
real Terraform provider source, `hashicorp/azurerm`, exactly, same as
this repo's own Go/TS siblings), one file per resource type, one
`__init__.py` per service directory re-exporting that service's own
resource classes:

```python
from ubx.azurerm.kubernetes import Cluster, ClusterConfig
```

`ubx/` itself deliberately has NO `__init__.py` — a real PEP 420
implicit namespace package, not a regular one any single distribution
owns, so `ubx-sdk-aws-py`/`-google-py`/`-kubernetes-py` can each
independently contribute their own `ubx.aws`/`ubx.google`/
`ubx.kubernetes` sibling under the same shared `ubx` namespace without
conflicting. `ubx/azurerm/__init__.py` (one level down) is a real,
ordinary package, same as every service package below it.

**Install**: `pip install ubx-sdk-azure` (note the package NAME says
`azure`, matching this repo's own name, even though the internal
import path says `ubx.azurerm` — `ubx sdk gen`'s mechanical shortName
derivation writes `ubx-sdk-azurerm` on every regen, corrected back to
`ubx-sdk-azure` by a `sed` step in `version-watch.yml`, same as this
repo's Go/TS siblings; the internal `azurerm` namespace segment is
correct as-is and NOT part of that correction).
`[tool.setuptools.packages.find]`'s `namespaces = true` is what makes
`ubx` resolve as a real PEP 420 namespace package on install rather
than an ordinary one.

Regenerated automatically on a weekly schedule (and via manual dispatch)
by `.github/workflows/version-watch.yml` — it checks the real
[Terraform Registry provider-versions API](https://registry.terraform.io/v1/providers/hashicorp/azurerm/versions)
for a newer `hashicorp/azurerm` release, and opens a PR with the
regenerated diff for review (never auto-merges). The version this repo
was last generated from is tracked in `VERSION`, not `.ubx/config.hcl` —
this repo carries no ubx stack/config of its own, only generated
bindings. A separate, manually-dispatched `.github/workflows/publish.yml`
(gated `workflow_dispatch`, UBI-135) is what actually publishes a new
version to PyPI after a regen PR is reviewed and merged.

**Shared runtime (`ubx_sdk`)**: a real, published PyPI package —
[ubx-sdk](https://pypi.org/project/ubx-sdk/) (import name `ubx_sdk`,
UBI-107) — declared as an ordinary dependency in `pyproject.toml`,
resolved automatically by `pip install`; no vendoring, no `PYTHONPATH`
trick needed. This repo previously vendored a local copy at
`vendor/ubx_sdk/` before the real publish existed — that vendored copy
is gone as of the switch to the real package.

**Sibling-`_config` collision (UBI-96/108/re-export-aggregation)**:
Azure's real 1,103 types were checked directly against this
restructuring's own new collision class (two resource types in the same
service whose local names `pascalCase` identically, which this repo's
new re-export aggregation could silently shadow — the exact class
`ubx-sdk-google-py` hit for real, `google_migration_center_report` +
`_config`) — zero real hits for Azure, confirmed by regenerating and
importing every module clean, not assumed safe by analogy.

Every file under a service directory except `__init__.py` is generated —
do not hand-edit; re-run `ubx sdk gen` (or wait for the automated PR)
after a provider version bump.

This completes [UBI-103](https://github.com/ubiquex/ubiquex)'s original
four-provider (AWS/Google/Kubernetes/Azure), three-language rollout.
