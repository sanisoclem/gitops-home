# Storage

- App config, documents and libraries: NFS. Mount the `apps` volume with `subPath: <namespace>/<app>/<dir>` and label the Deployment `nas-apps: "true"` so `components/cluster-values` fills in the server and path.
- SQLite or CouchDB state: a PVC with `storageClassName: block-backed`. It is backed up hourly and restores itself on a new cluster (see the platform repo's `volsync-block-backed` policy).
- Logs, caches, indexes you can rebuild: a PVC with no `storageClassName` (the `block-transient` default).
- Moving an existing volume to `block-backed` means a new PVC name: list the move in homelab's `scripts/migrate-volumes.sh`, run `task up` there, then push the manifest that switches to the new name.
- Sonarr, Radarr, Prowlarr and Bazarr keep their config on NFS, so switch them to the CNPG `postgres` cluster before promoting them from `staged/`; SQLite on NFS corrupts.

# Postgres

`apps/services/postgres.yaml` bootstraps from its own backup and keeps archiving to it (`cnpg.io/skipEmptyWalArchiveCheck`). Never run a second cluster against its backup path with `serverName: postgres`, and never change that `serverName`.

# Cluster values

Addresses, names and paths come from homelab's `.env` through `config/cluster-config.yaml`, which `gitops:sync` writes; manifests carry `placeholder` and `components/cluster-values` fills it in. Never write an address into a manifest.
