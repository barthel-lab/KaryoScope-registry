# karyoscope-registry

Registry of pre-built KaryoScope databases.

This repository serves as the central index of KaryoScope databases available
for download via `karyoscope download`. Each database entry points to a hosted
archive (typically on Zenodo) and includes the metadata needed for KaryoScope
to validate, install, and use the database.

## Contents

- `registry.yaml` — the registry of available databases
- `schema/registry.schema.json` — JSON Schema describing valid registry entries

## Adding a database

To contribute a new database (community or lab-built):

1. Build and upload the database archive to a stable host (Zenodo recommended).
2. Compute the SHA-256 checksum of the archive.
3. Open a pull request adding an entry to `registry.yaml` under `databases`
   (for official) or `community_databases` (for community-contributed).
4. Ensure the entry validates against `schema/registry.schema.json`.

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## Used by

- [KaryoScope](https://github.com/barthel-lab/KaryoScope)

## License

Registry metadata in this repository is released under CC0 1.0 (public domain).
See [LICENSE](LICENSE).

Databases referenced by this registry have their own licenses; consult the
`citation` and source links for each entry.
