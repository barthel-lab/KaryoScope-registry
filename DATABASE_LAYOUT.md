# KaryoScope Database Layout

This document specifies the canonical on-disk layout for a KaryoScope database
archive. All databases distributed through the
[karyoscope-registry](https://github.com/barthel-lab/KaryoScope-registry)
must conform to this layout. KaryoScope validates the layout at install time
and refuses to install non-conforming archives.

## Top-level structure

A database is distributed as a single gzipped tar archive (`.tar.gz`) named
after the database `id`. When extracted, it produces a single top-level
directory of the same name:

    KS_human_CHM13_v2.tar.gz
    └── KS_human_CHM13_v2/
        ├── colors.tsv
        ├── features.tsv
        ├── hierarchy.tsv
        ├── index/
        │   ├── features.kmc_pre
        │   └── features.kmc_suf
        └── manifest.yaml

The top-level directory name **must match the `id` field** in the registry
entry and the `id` field in `manifest.yaml`.

## Files

### `colors.tsv` (required)

Tab-separated file mapping features to display colors, used by
`karyoscope karyotype` when rendering. Three columns with a header line:

    feature_set	feature	color
    chromosome	chr1	#ffacbc
    chromosome	chr2	#006400
    region	p_arm	#808080
    region	q_arm	#808080
    region	HSat1A	#91ff00
    ...

Columns:

- `feature_set` — the feature set the feature belongs to (must match a
  feature set declared in `manifest.yaml`).
- `feature` — the feature name (must match a name in `features.tsv` for
  the corresponding feature set). Internal hierarchy nodes
  (e.g. `metacentric`, `autosome`, `sex`) should also be listed.
- `color` — a CSS-style hex color (e.g. `#ff0000`). Six-digit hex required;
  three-digit shorthand and named colors are not supported.

Notes:

- The reserved feature `novel` (k-mers not present in the index) is
  always rendered with a built-in fixed color. Rows in `colors.tsv`
  that attempt to set a color for `novel` are ignored, with a warning.
- Features absent from `colors.tsv` are rendered with a fallback color (light gray) and
  generate a warning at render time. To suppress the warning, list every
  feature that may appear in the rendered output.
- The same feature name may appear in multiple feature sets with different
  colors (e.g. `rDNA` in both `acrocentric` and `region`); the `feature_set`
  column disambiguates.

### `features.tsv` (required)

Tab-separated file mapping numeric featureIDs (as emitted by `get_featureIDs`)
to feature names for each feature set. The first column is the featureID,
subsequent columns are feature names per feature set. The header line names
the feature sets:

    featureID	chromosome	haplotype	region	...
    1	chr1	hap1	p_arm	...
    2	chr1	hap1	centromere	...
    ...

FeatureID `0` is reserved for "novel" (k-mers not present in the index)
and is not listed; KaryoScope handles it implicitly.

### `hierarchy.tsv` (required)

Tab-separated file describing the feature hierarchy. Three columns with a
header line:

    feature_set	child	parent
    chromosome	chr1	metacentric
    chromosome	chr2	metacentric
    chromosome	metacentric	autosome
    chromosome	autosome	categorized
    region	p_arm	arm
    region	q_arm	arm
    ...

The root of every tree must be the literal string `categorized`.

### `index/` directory (required)

Contains the index files. The exact contents depend on the index type
declared in `manifest.yaml`:

- **KMC indexes** (`index.type: kmc`): two files sharing a common basename,
  with extensions `.kmc_pre` and `.kmc_suf`. The basename is declared in
  `manifest.yaml` (canonically `features`).

Other index types may be added in future KaryoScope versions; see the
manifest schema for the up-to-date list.

### `manifest.yaml` (required)

Operational metadata describing the database's structure and capabilities.
This is read by KaryoScope at install time and at every command invocation.
See the [Manifest schema](#manifest-schema) section below.

## Manifest schema

The `manifest.yaml` file must conform to the following structure:

```yaml
id: KS_human_CHM13_v2                  # Must match registry entry and directory name
version: "2.0.0"                       # Semver
karyoscope_min_version: "1.0.0"        # Minimum KaryoScope version required

index:
  type: kmc                            # Index format ("kmc" is the only type in v1)
  basename: index/features             # For type=kmc: KMC base path (no extension)

hierarchy: hierarchy.tsv               # Path to hierarchy file (relative)
features: features.tsv                 # Path to features file (relative)
colors: colors.tsv                     # Path to colors file (relative)

kmer:
  size: 31                             # k-mer size used to query
  type: fixed                          # "fixed" (KMC) or "variable" (future, e.g. HKS)
  max_size: 31                         # For type=fixed, equals size

feature_sets:                          # Must match what's in features.tsv header
  - chromosome
  - region
  - repeat
  - subtelomeric
  - gene
  - acrocentric

roles:                                 # Which feature_sets play special roles
  chromosome_assignment: chromosome    # Used by `scaffold` and `karyotype`
  region_assignment: region            # Used by `scaffold` and `karyotype`
  centromere_detection: region         # Used by `centromeres`

smoothing:
  recommended_window_bp: 1000          # Default smoothing window (advisory)
```

## Why this layout

A few design notes for contributors and future maintainers:

- **Filenames inside the database directory do not include the database id.**
  This makes the directory self-contained and robust to being renamed by the
  user. The id appears only in `manifest.yaml` and as the directory name.
- **The `index/` subdirectory** isolates bulky binary files from small
  metadata, and reserves a place for index formats that need multiple files.
- **The `index.type` discriminator** allows new index backends (e.g., HKS) to
  be added without breaking existing databases. Each backend declares its own
  paths inside `index:`.
- **The `roles:` section** lets non-human databases use different feature_set
  names for the same conceptual role (e.g., a database for a species without
  classical chromosome arms might still declare a `region_assignment`).

## Validating a database archive

Before submitting a registry PR for a new database, validate locally:

```bash
karyoscope info /path/to/KS_my_database_v1.tar.gz
```

This reads the manifest, checks file presence and naming, verifies the
KMC index opens, and reports any issues. Fix all reported issues before
opening a registry PR.

## Versioning

A database's `version` field follows [semver](https://semver.org/):

- Patch (`1.0.0` → `1.0.1`): bug fixes, no schema or content change.
- Minor (`1.0.0` → `1.1.0`): backwards-compatible additions (new features,
  expanded feature sets, additional colors).
- Major (`1.0.0` → `2.0.0`): breaking changes (renamed features, removed
  features, changed hierarchy semantics).

When publishing a new version, publish to a new Zenodo record (Zenodo's
"new version" feature preserves the old version's DOI), and submit a
registry PR adding a new entry. Old versions remain available indefinitely.
