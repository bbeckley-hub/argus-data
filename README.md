# argus-data

Versioned data releases for the **ARGUS** family of genomic surveillance tools
(**A**ntimicrobial **R**esistance **G**enomic **U**nified **S**urveillance).

This repository holds the data bundles separately from the code so that each
command-line interface can be versioned, released, and upgraded independently
of its dataset. Every bundle is published as a GitHub release with a SHA256
checksum sidecar, so installations are verifiable and reproducible.
`https://github.com/bbeckley-hub/argus-data` is the single download endpoint
for all ARGUS command-line tools, present and future.

---

## The ARGUS family

ARGUS is a family of open typing-based genome retrieval resources covering the
WHO priority ESKAPE pathogens. Each product has its own web resource, CLI
package, data release cycle, and DOI, while sharing one technical foundation
(master table + long-format module tables) and this one data repository.

| Product  | Organism                | Status        | Data tag prefix    | CLI package     |
|----------|-------------------------|---------------|--------------------|-----------------|
| SAURUS   | *Staphylococcus aureus* | **Released**  | `saurus-data-`     | [`saurus-cli`](https://pypi.org/project/saurus-cli/) |
| ECLIPSE  | *Escherichia coli*      | Released (web); CLI forthcoming | `eclipse-data-` | — |
| ABYSS    | *Acinetobacter baumannii* | Released (web); CLI forthcoming | `abyss-data-` | — |
| KLEOS    | *Klebsiella pneumoniae* | Planned       | `kleos-data-`      | — |
| PSEUDA   | *Pseudomonas aeruginosa*| Planned       | `pseuda-data-`     | — |
| EMPIRE   | *Enterococcus faecium*  | Planned       | `empire-data-`     | — |
| ENTROPY  | *Enterobacter* spp.     | Planned       | `entropy-data-`    | — |

Umbrella site: <https://argus-db.org/>
SAURUS web resource: <https://saurus-db.org/>

**New ARGUS data will be published in this repository as it is produced.**
Releases are namespaced by tag prefix (`<product>-data-vX.Y.Z`), so each CLI
resolves its own latest data release from this shared repository without
interference: `saurus init` looks for the newest `saurus-data-*` tag, and the
future `eclipse init` will look for the newest `eclipse-data-*` tag, and so on.

---

## How releases work

Each data release is a GitHub release in this repository with:

- `<product>_data.zip` — the data bundle (zipped module TSVs, see below)
- `<product>_data.zip.sha256` — SHA256 checksum sidecar
- a release manifest recording the NCBI snapshot date, source datasets,
  build tools and their versions, and file-level provenance

Tag format: `<product>-data-vMAJOR.MINOR.PATCH`, e.g. `saurus-data-v1.1.0`.

Versioning semantics:

- **MAJOR** — breaking change to the schema (column names, module layout);
  older CLIs may not be able to read the bundle
- **MINOR** — content update (new genomes, re-typing, corrected annotations);
  read-compatible with existing CLIs
- **PATCH** — defect fix in the data with no schema or coverage change

Data versioning is fully independent of CLI versioning. A CLI upgrade never
forces a data upgrade and vice versa.

---

## Installing

### With the CLI (recommended)

```bash
pip install saurus-cli

saurus init                # latest saurus-data-* release
saurus init -v v1.1.0      # a specific pinned release
saurus init --force        # reinstall over an existing install
```

The CLI downloads the release zip, verifies the `.sha256` sidecar, and
extracts it to the appropriate writable data directory (six-tier search:
`--data-dir` > `$SAURUS_DATA_DIR` > existing install > XDG data home >
`~/.saurus/data` > `./data`). A checksum mismatch is always fatal; an install
is never performed unverified unless you explicitly pass `--allow-unverified`.

### Manual download

```bash
curl -LO https://github.com/bbeckley-hub/argus-data/releases/download/saurus-data-v1.1.0/saurus_data.zip
curl -LO https://github.com/bbeckley-hub/argus-data/releases/download/saurus-data-v1.1.0/saurus_data.zip.sha256
sha256sum -c saurus_data.zip.sha256
unzip saurus_data.zip -d ~/.saurus/data
```

### Offline install from a local zip

```bash
saurus init -f /path/to/saurus_data.zip          # requires the .sha256 sidecar next to it
saurus init -f /path/to/saurus_data.zip --allow-unverified   # explicit opt-out
```

### Pinning for reproducibility

```bash
export SAURUS_DATA_DIR=/pinned/saurus-v1.1.0
saurus init -v v1.1.0 -o "$SAURUS_DATA_DIR"
saurus --lineage USA300 --count --config query.json
saurus --version   # reports both CLI and data version; record both in your methods
```

---

## What is inside a SAURUS bundle (v1.1.0)

16 files, 16.5 MB. Master table: 134,273 genomes × 20 columns.

| Path | Content |
|------|---------|
| `DATA_VERSION` | Release tag, reported by `saurus --version` |
| `mrsa/mrsa_all.tsv.gz` | Master table: one row per genome, all modules joined |
| `mlst/mlst_all.tsv.gz` | MLST: sequence type, clonal complex (mlst 2.35.2) |
| `mlst/cc.txt.gz` | ST → clonal complex mapping |
| `spa/spa_all.tsv.gz` | spa types (Ridom spaTyper) |
| `sccmec/sccmec_cge_all.tsv.gz` | SCCmec, CGE SCCmecFinder 1.2.0 |
| `sccmec/sccmec_rpet_all.tsv.gz` | SCCmec, RPet (Staphopia-derived) |
| `agr/agr_all.tsv.gz` | agr types (agrVATE) |
| `capsule/capsule_all.tsv.gz` | Capsule polysaccharide type |
| `gene_annotation/amr_all.tsv.gz` | Long-format AMR gene annotations (ABRicate 1.4.0) |
| `gene_annotation/amr_gene_freq.tsv.gz` | Per-gene frequency catalogue |
| `gene_annotation/vfdb_all.tsv.gz` | Long-format VFDB virulence annotations |
| `gene_annotation/vfdb_gene_freq.tsv.gz` | Per-gene frequency catalogue |
| `gene_annotation/plasmid_all.tsv.gz` | Long-format plasmid replicon annotations |
| `gene_annotation/plasmid_gene_freq.tsv.gz` | Per-gene frequency catalogue |
| `metadata/metadata.tsv.gz` | Country, source, host, year, accession namespaces |

Note that the gene-annotation tables cover genomes **beyond** the 134,273-row
master collection (for v1.1.0: 98,948 VFDB-annotated, 90,264 AMR-annotated and
86,299 plasmid-annotated assemblies). Standalone gene queries therefore
resolve in the annotated-universe space, while queries that combine a gene
filter with any typing or metadata filter resolve in the master-table space.
The two denominators are documented in the SAURUS-CLI manuscript and are
reported by `saurus --about argus`.

---

## Provenance and scope

- SAURUS gene annotations derive from the Shelenkov et al. (2025) pan-GPS
  dataset, keyed by GenBank (GCA) assembly accession; typing is resolved
  RefSeq-first, then GenBank. Provenance is recorded per file in the release
  manifest.
- The gene-annotation layer is **static**: it reflects the Shelenkov snapshot
  and is not incrementally extended. Missing annotation means "not annotated
  in the source dataset", not "gene absent".
- **Genome sequences and raw reads are not redistributed here.** The bundles
  carry accession lists; users fetch assemblies from NCBI with the `datasets`
  tool or raw reads from the SRA with their client of choice. This keeps every
  release small (tens of MB) and free of redistribution constraints.
- Each bundle is rebuildable from public NCBI data. - Each bundle is rebuildable from public NCBI data. The full reconstruction
  and curation pipeline (StaphScope 2.1.0-based: HPC download/typing lane,
  batch checkpoints, merge and gene-curation scripts, and all site
  generators) ships in the [argus-code](https://github.com/bbeckley-hub/argus-code) repository
  (subdirectory `saurus/`), so the dataset can be regenerated or
  independently extended from the snapshot recorded in the manifest.
  
---

## Rebuilding or extending a release

The `saurus/` directory of
[argus-code](https://github.com/bbeckley-hub/argus-code) contains the
complete checkpoint state of the SAURUS pipeline: 14 gzipped StaphScope
2.1.0 batch summaries covering the 134,160 typed genomes, the merged table
seeds, the gene-curation chain, and every web-resource generator. A fresh
clone reproduces the shipped state with two commands:
`scripts/restore_seeds.sh` then `scripts/Regenerate_all.sh`. To extend the
dataset with new genomes, follow `saurus/HPC_scripts/README.md` (download
+ typing lane), drop the finished batch into `saurus/batches/`, and rerun
the merge-and-generate chain documented in `saurus/scripts/README.md`.

---

## Maintenance policy

Data releases are planned on a semi-annual cadence, with out-of-cycle releases
when a defect is found. Releases are archived on Zenodo with a DOI per
version. Because the master table is derived entirely from public genome
sequences and annotations, the dataset remains reproducible even if central
maintenance lapses.

---

## Citation

If you use a data bundle from this repository, please cite the corresponding
product's publication and the data release DOI:

- SAURUS: Beckley B. SAURUS-CLI: a reproducible command-line interface for
  typing-based genome retrieval in *Staphylococcus aureus*. BMC Bioinformatics
  (under review). Data release: saurus-data-v1.1.0,
  <https://github.com/bbeckley-hub/argus-data/releases/tag/saurus-data-v1.1.0> `[DOI to follow]`
- ECLIPSE / ABYSS: see <https://argus-db.org/> for current references.

---

## License

Data in this repository is released under the same terms as the
corresponding CLI (`saurus-cli`: MIT). Upstream annotation datasets (VFDB,
CARD/MEGARes via ABRicate, Shelenkov et al. pan-GPS) remain under their own
licenses; the manifest lists each upstream source.

---

## Support

- Issues: <https://github.com/bbeckley-hub/argus-data/issues>
- SAURUS-CLI issues: <https://github.com/bbeckley-hub/saurus-cli/issues>
- Contact: Brown Beckley, University of Ghana Medical School
  (brownbeckley94@gmail.com)
