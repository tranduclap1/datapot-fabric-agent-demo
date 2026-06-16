# Fabric environment (deployment targets)

Fill these in for your tenant. **Do not hardcode IDs in committed artifacts** — discover them at
runtime via the Fabric REST API / `fab` CLI, or inject them as parameters. This file is a
human reference, not a credential store.

## Tenant / capacity

| Setting | Value |
|---------|-------|
| Tenant (Entra ID) | _e.g. datapot.edu.vn_ |
| Capacity (F-SKU / P-SKU) | _to fill_ |
| Default region | _to fill_ |

## Workspaces

| Purpose | Workspace name | Notes |
|---------|----------------|-------|
| Development | `datapot-bi-dev` | Where PBIP projects publish first |
| Test / UAT | `datapot-bi-test` | Promotion target |
| Production | `datapot-bi-prod` | Consumers |

> If reports source from a medallion lakehouse, keep Bronze/Silver/Gold in their own
> workspaces (see the `e2e-medallion-architecture` skill) and build Direct Lake models on **Gold**.

## Data source modes

- **Local prototype (current):** import from CSV/Parquet pasted into a report's `dataset/raw/`,
  driven by the `DataFolder` parameter. Self-contained, no Fabric needed.
- **Fabric Direct Lake (target):** point the semantic model at a Gold lakehouse SQL endpoint;
  discover the connection string from the lakehouse `properties.sqlEndpointProperties` — never hardcode.

## Publishing

```bash
# Power BI / pbir CLI
pbir publish "reports/NN-name/pbip/<Project>.Report" "datapot-bi-dev.Workspace/<Project>.Report"

# Fabric CLI
fab import "datapot-bi-dev.Workspace/<Project>.Report" -i "reports/NN-name/pbip/<Project>.Report" -f
```

## Identity & permissions

- Prefer **service principals** / **workspace identity** for automation; store secrets in Key Vault.
- Token audiences differ per workload (Fabric REST vs OneLake vs SQL vs XMLA) — see the
  skills-for-fabric `COMMON-CORE.md` "Token Audiences" table.
