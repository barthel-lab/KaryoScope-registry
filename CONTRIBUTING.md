# Contributing to karyoscope-registry

This repository is the source of truth for what databases are available via
`karyoscope download`. Contributions are welcome from anyone who has built
and shared a KaryoScope-compatible database.

## How to add a database

1. **Build your database** using `karyoscope build` (or by hand, following the
   layout in [DATABASE_LAYOUT.md](DATABASE_LAYOUT.md)). See below for validating
   a database.
2. **Upload the database tarball** to a stable host. We strongly recommend
   [Zenodo](https://zenodo.org/) because each upload gets a permanent DOI.
3. **Compute the SHA-256 checksum** of the tarball:
```bash
   sha256sum your_database.tar.gz
```
4. **Fork this repository** and add an entry under either `databases:` (for
   databases maintained by recognized groups; subject to maintainer review)
   or `community_databases:` (for community-contributed entries, accepted
   more permissively).
5. **Validate your entry** against the schema:
```bash
   # If you have jsonschema installed:
   yq -o=json registry.yaml | jsonschema -i - schema/registry.schema.json
```
6. **Open a pull request.** Include in the PR description: the intended use
   case, a brief description of how the database was built, and any caveats
   users should know.

### Validating a database

Before submitting a registry entry, ensure your database archive conforms
to the canonical layout described in [DATABASE_LAYOUT.md](DATABASE_LAYOUT.md).
Validate locally with `karyoscope info <tarball>` before opening a PR.

## Review process

- Schema-valid PRs from new contributors are reviewed within ~1 week.
- We verify the URL resolves, the checksum matches, and the metadata is accurate.
- We do not vet the scientific validity of community databases — that's on
  the maintainer.

## Maintainers

- Timothy Rhyker Ranallo-Benavidez ([@tbenavi1](https://github.com/tbenavi1))
- Floris P. Barthel ([@fpbarthel](https://github.com/fpbarthel))

## Code of conduct

Be kind. We follow the [Contributor Covenant v2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/).
