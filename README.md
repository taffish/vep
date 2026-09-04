# vep

`vep` packages Ensembl Variant Effect Predictor (VEP) for TAFFISH.

Package identity:

- name: `vep`
- command: `taf-vep`
- kind: `tool`
- version: `116.2-r1`
- image: `ghcr.io/taffish/vep:116.2-r1`
- native platforms: `linux/amd64`, `linux/arm64`
- TAFFISH app license: Apache-2.0
- upstream: <https://github.com/Ensembl/ensembl-vep/tree/release/116.2>

## What This App Packages

The runtime is based on the official multi-architecture image
`ensemblorg/ensembl-vep:release_116.2`, pinned to OCI index digest
`sha256:b5c4839c194cfcc148dd6a34393623fd4fdb0239d03d29af10c5b5a461ec366a`.
The upstream tag resolves to commit
`2cb0bbe216bb31c75de8f8000e2da7ff4fb7b451`; the official tag archive has
SHA-256 `834e9c9c82ed1b656a655ab126a4596e4808dcd984f5cb72ecb88b48c7ed28f9`.

The image contains:

- `vep`: the default variant consequence annotator
- `filter_vep`: filtering for VEP text and VCF/CSQ output
- `haplo`: Haplosaurus phased-haplotype analysis
- `variant_recoder`: variant identifier and representation conversion
- `INSTALL.pl` and `convert_cache.pl`: upstream data utilities
- `vep-install-data-116`: TAFFISH's explicit, shared-safe cache/FASTA installer
- maintained VEP plugins, Ensembl Perl APIs, Bio::DB::HTS, Bio::DB::BigFile,
  htslib commands and GeneSplicer support inherited from the official image

The TAFFISH layer adds `HTML::TableExtract`, which the official HTTPS cache and
FASTA installer path requires but the upstream runtime image does not contain.

## Scope

This app supports the packaged command-line programs, offline Ensembl cache
annotation, custom GFF3/GTF plus FASTA annotation, compressed output, remote
database mode when the user intentionally enables networking, and explicit
plugin data supplied by the user.

The upstream repository also includes a Nextflow entry point, while Ensembl
offers hosted web and REST VEP services. This container does not package a
Nextflow runtime, a local Ensembl website, or a local REST/web service. Those
are separate execution and service surfaces, not an omitted local GUI. The GUI
and long-running service checklists are therefore not applicable to this CLI
app.

## Usage

Show wrapper and upstream help:

```sh
taf-vep --help
taf-vep -- --help
taf-vep vep --help
```

`vep` has no `--version` option. Its help banner reports
`ensembl-vep          : 116.2`; `taf-vep --version` reports the TAFFISH wrapper
version.

Annotate with a prepared cache selected from a standard TAFFISH data root:

```sh
TAFFISH_VEP_DATASET=homo_sapiens--GRCh38 \
taf-vep -- --cache --offline --species homo_sapiens --assembly GRCh38 \
  --format vcf --vcf --force_overwrite \
  -i variants.vcf -o variants.vep.vcf
```

Use explicit custom annotations instead of an Ensembl cache:

```sh
taf-vep -- --gff annotation.gff3.gz --fasta genome.fa.gz \
  --format vcf --vcf --force_overwrite \
  -i variants.vcf -o variants.vep.vcf
```

Run another packaged command through automatic command mode:

```sh
taf-vep filter_vep --format vcf \
  --filter "Consequence is missense_variant" \
  -i variants.vep.vcf -o missense.vcf --force_overwrite
taf-vep haplo --help
taf-vep variant_recoder --help
```

## Inputs and Outputs

VEP accepts VCF, its default whitespace-delimited format, HGVS expressions,
identifiers and coordinate forms according to upstream options. Custom
GFF3/GTF sources should be bgzip-compressed, tabix-indexed and paired with the
matching FASTA.

Outputs include VEP text, VCF with `CSQ`, JSON, statistics and warning files.
`filter_vep`, `haplo` and `variant_recoder` have their own output contracts.
Explicit output paths and temporary files are written through the ordinary
TAFFISH working-directory and `/tmp` mappings; the installed runtime and
prepared cache mount remain read-only during analysis.

## VEP Cache and FASTA Resources

VEP `116.2` still uses Ensembl cache/API generation `116`. Production cache and
FASTA data are large, species- and assembly-specific resources and are not
embedded in the runtime image. Plugin datasets such as CADD, AlphaMissense,
SpliceAI, dbNSFP, ClinVar and gnomAD remain separately selected resources.

The smallest unit managed by `vep-install-data-116` is one exact
`SPECIES--ASSEMBLY` member containing the indexed cache and matching primary
assembly (or upstream toplevel fallback) FASTA. `all`, wildcard and implicit
selection are rejected. For example, preview a human GRCh38 install without
writing or accessing the network:

```sh
taf-vep vep-install-data-116 --dry-run \
  --species homo_sapiens --assembly GRCh38
```

### Personal installation

Choose Docker, Podman or Apptainer with the ordinary backend selector. The
wrapper turns `TAFFISH_VEP_DATA_INSTALL_ROOT` into the app-intrinsic writable
`/data-install` mount; the helper refuses an ephemeral unmounted install.

```sh
ROOT="$HOME/.local/share/taffish/databases/vep"
mkdir -p "$ROOT"
TAFFISH_CONTAINER_BACKEND=docker \
TAFFISH_VEP_DATA_INSTALL_ROOT="$ROOT" \
taf-vep vep-install-data-116 \
  --species homo_sapiens --assembly GRCh38
```

Use `TAFFISH_CONTAINER_BACKEND=podman` or `apptainer` for the same helper. The
completed member is
`$ROOT/release-116/homo_sapiens--GRCh38`. Installation uses an exclusive lock,
a same-filesystem staging directory, the official Ensembl HTTPS installer and
cache `CHECKSUMS`, then records an actual per-file SHA-256 inventory, manifest
and ready marker before atomic promotion. Existing data are accepted only
after deep verification; use `--verify-only` to audit and `--force` only for an
intentional atomic replacement. A killed transaction is retained as an exact
staging/backup path and blocks later installation until an administrator has
inspected and removed or recovered it.

The upstream selectable catalog does not publish one immutable whole-family
size in this app. `--dry-run` therefore reports size as unknown and states the
selection and integrity policy; installation enforces an adjustable free-space
floor (`--min-free-gb`, default 10 GiB). A production human cache can be much
larger than that floor, so operators should inspect the current Ensembl member
size before downloading.

### Site-wide installation

An administrator can prepare one standard root through a designated installer
account, then make it root-owned and read-only for ordinary users:

```sh
sudo mkdir -p /usr/local/share/taffish/databases/vep
sudo chown "$(id -u):$(id -g)" /usr/local/share/taffish/databases/vep
TAFFISH_VEP_DATA_INSTALL_ROOT=/usr/local/share/taffish/databases/vep \
taf-vep vep-install-data-116 --species homo_sapiens --assembly GRCh38
sudo chown -R root:root /usr/local/share/taffish/databases/vep
sudo chmod -R a+rX,go-w /usr/local/share/taffish/databases/vep
```

`/opt/taffish/databases/vep` is the second standard site root. Ordinary users
do not need root, network or write access after the administrator completes the
installation.

### Discovery, override and disable controls

Set `TAFFISH_VEP_DATASET` to an exact member ID. The wrapper searches, in order:

1. `~/.local/share/taffish/databases/vep/release-116/<member>`
2. `/usr/local/share/taffish/databases/vep/release-116/<member>`
3. `/opt/taffish/databases/vep/release-116/<member>`

It validates the manifest, inventory and ready marker, resolves symlinks to a
physical host path and mounts only that member read-only at
`/opt/taffish/vep-cache`. VEP reads that stable path through its supported
`VEP_DIR_CACHE` setting. It does not choose a species or assembly for the user.
Set `TAFFISH_VEP_DATA_PATH` to an exact prepared member for an explicit
override, or `TAFFISH_VEP_AUTO_MOUNT=0` to disable discovery. An invalid
explicit root or member fails closed rather than silently falling back to
another dataset.

For RefSeq or merged cache members, use the exact upstream species member name
when installing and pass VEP's corresponding `--refseq` or `--merged` option at
analysis time.

### Manual backend fallback

Automatic resource mounts use the same stable container path on all three
backends. If site policy requires manual mounts, disable automatic discovery:

```sh
TAFFISH_VEP_AUTO_MOUNT=0 \
TAFFISH_DOCKER_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
taf-vep -- --cache --offline -i variants.vcf -o out.txt

TAFFISH_VEP_AUTO_MOUNT=0 \
TAFFISH_PODMAN_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
taf-vep -- --cache --offline -i variants.vcf -o out.txt

TAFFISH_VEP_AUTO_MOUNT=0 \
TAFFISH_APPTAINER_RUN_ARGS="--bind /abs/member:/opt/taffish/vep-cache:ro" \
taf-vep -- --cache --offline -i variants.vcf -o out.txt
```

## Backend and Platform Matrix

| Capability | Docker | Podman | Apptainer | Boundary |
| --- | --- | --- | --- | --- |
| CLI and custom GFF/FASTA | Native amd64/arm64 image | Native amd64/arm64 image | OCI-derived SIF on native Linux | Exact validation status is recorded below |
| Prepared cache member | Automatic read-only volume | Automatic read-only volume | Automatic read-only bind | Explicit member selection; stable `VEP_DIR_CACHE=/opt/taffish/vep-cache` |
| Data installation | Writable `/data-install` plus host UID/GID | Writable `/data-install` with keep-id | Writable `/data-install` under caller identity | Network is required only during installation |
| Remote database/REST/plugin lookup | Requires explicit network policy | Requires explicit network policy | Requires explicit network policy | Offline reproducibility should use prepared local data |

Docker and Podman run VEP under the caller's UID/GID (Podman also uses
`keep-id`), so inputs and outputs inside private user directories keep host
ownership and remain accessible. Apptainer already executes as the caller.

The platform and backend evidence axes are independent: both declared native
OCI platforms must run real VEP functionality, while each public backend must
pass exact smoke and resource-mount behavior on an architecture-matched native
Linux host. Uncoupled combinations not separately exercised are reported as
such rather than treated as native failures.

## Resource License Boundary

Ensembl states that project-generated data are available without restriction,
while some incorporated data can retain third-party constraints. The cache,
reference FASTA and plugin datasets therefore remain governed by the Ensembl
data disclaimer and any source-database terms; the VEP Apache-2.0 software
license is not used as a substitute for those data terms. See:

- <https://www.ensembl.org/info/about/legal/disclaimer.html>
- <https://www.ensembl.org/info/about/legal/index.html>

The helper downloads from Ensembl's official release-116 FTP-over-HTTPS paths.
It does not download third-party plugin datasets.

## Boundaries

- `INSTALL.pl`, remote database mode, REST calls and plugin-data acquisition can
  require network access. Normal smoke and recommended production annotation
  are offline.
- Packaging QA installed and deep-verified the real Ensembl cache/FASTA member
  `ciona_savignyi--CSAV2.0`; it did not download a human production member or
  every selectable species/assembly. This runtime evidence does not establish
  biological correctness for other resources or analyses.
- Upstream plugin Perl files in `/plugins` do not imply that licensed or large
  plugin datasets are bundled.
- Nextflow, the hosted Ensembl web VEP and hosted REST service are not local
  components of this CLI app.

## Validation

The release smoke contract checks exact VEP `116.2` identity, cache generation
`116`, main helper commands, Perl/HTS/BigWig modules, HTTPS installer parsing,
plugin files and compression/index utilities. It runs `filter_vep` on synthetic
VCF/CSQ data, real offline VEP annotation with tiny GFF3/FASTA/VCF fixtures, and
the repaired `--compress_output gzip` path. Resource tests additionally cover
explicit selection, no-`all`, mounted installation refusal, manifest/inventory
validation, read-only reuse and backend mount construction.

The final candidate passed every manifest command in disconnected containers:
Docker and Podman each passed 62/62 direct plus read-only-root checks on native
amd64 Linux, and the OCI-derived read-only SIF passed 31/31 checks with
Apptainer. The native arm64 OCI candidate independently passed the same 62/62
Docker matrix. The final wrapper then passed explicit and standard-path
resource discovery, actual writable install binds, invalid-resource
fail-closed behavior, and read-only reuse on Docker, Podman and Apptainer. A
real `ciona_savignyi--CSAV2.0` member (453 files; 87,722,369 bytes) was installed
through the helper, deep-verified from its SHA-256 inventory, and used for an
offline cache/FASTA annotation through all three backends. Podman on arm64 and
Apptainer on arm64 were not separately validated; no backend-specific coupling
was found that requires those extra combinations.

These are packaging and runtime checks, not complete scientific validation on
production genomes or external annotation databases.

## Upstream, License and Citation

- VEP repository/tag: <https://github.com/Ensembl/ensembl-vep/tree/release/116.2>
- VEP documentation: <https://www.ensembl.org/info/docs/tools/vep/index.html>
- CLI documentation: <https://www.ensembl.org/info/docs/tools/vep/script/index.html>
- cache documentation: <https://www.ensembl.org/info/docs/tools/vep/script/vep_cache.html>
- official image: <https://hub.docker.com/r/ensemblorg/ensembl-vep>
- upstream software license: Apache-2.0

For academic use, cite:

McLaren W, Gil L, Hunt SE, Riat HS, Ritchie GRS, Thormann A, Flicek P,
Cunningham F. The Ensembl Variant Effect Predictor. Genome Biology.
2016;17:122. doi: `10.1186/s13059-016-0974-4`; PMID: `27268795`.
