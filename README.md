# vep

TAFFISH wrapper for Ensembl Variant Effect Predictor (VEP) 116.1, the Ensembl
runtime for predicting functional consequences of genomic variants and running
related VEP helper tools.

Package identity:

- name: `vep`
- command: `taf-vep`
- version: `116.1-r1`
- kind: `tool`
- image: `ghcr.io/taffish/vep:116.1-r1`
- upstream: Ensembl VEP tag `release/116.1`
- runtime version: `ensembl-vep 116.1`; cache/API version remains `116`
- app license: Apache-2.0
- upstream license: Apache-2.0

## What Is Packaged

This app uses the official Ensembl VEP Docker runtime
`ensemblorg/ensembl-vep:release_116.1`, pinned by OCI manifest digest. The
runtime is built by Ensembl from the `Ensembl/ensembl-vep` source tree and
includes VEP, Ensembl API modules, maintained VEP plugins, htslib tools,
Bio::DB::HTS, Bio::DB::BigFile support, Haplosaurus dependencies, and
GeneSplicer plugin support. The source tag resolves to commit
`97c24b205f4ba8b44ccc75b839512a1d59104019`; the downloaded source archive has
SHA-256 `959545e10ed443ff21d040a4fd277c640b9258bb2a4f08fe4355f702ed53b6ec`.

The container exposes:

- `vep`: default Variant Effect Predictor command
- `filter_vep`: filter VEP tabular or VCF/CSQ output
- `haplo`: Haplosaurus phased haplotype consequence tool
- `variant_recoder`: translate variant identifiers and HGVS-like encodings
- `INSTALL.pl`: Ensembl VEP installer for caches, FASTA, API, and plugins
- `bgzip`, `tabix`, `htsfile`, `perl`, `curl`, `python`, `python2`

## Usage

Show TAFFISH wrapper help:

```sh
taf-vep --help
```

Show upstream VEP help and runtime version banner:

```sh
taf-vep -- --help
taf-vep vep --help
```

`vep --version` is not an upstream VEP option. The upstream version is reported
at the top of `vep --help` as `ensembl-vep          : 116.1`.

Annotate with an already prepared cache:

```sh
taf-vep -- --cache --offline --dir_cache "$PWD/vep-data" \
  --species homo_sapiens --assembly GRCh38 \
  --format vcf --vcf --force_overwrite \
  -i variants.vcf -o variants.vep.vcf
```

Use a custom GFF3/GTF and FASTA instead of an Ensembl cache:

```sh
taf-vep -- --gff annotation.gff3.gz --fasta genome.fa.gz \
  --format vcf --vcf --force_overwrite \
  -i variants.vcf -o variants.vep.vcf
```

Run helper commands in the same app environment:

```sh
taf-vep filter_vep --format vcf --filter "Consequence is missense_variant" \
  -i variants.vep.vcf -o missense.vcf --force_overwrite

taf-vep haplo --help
taf-vep variant_recoder --help
taf-vep INSTALL.pl --help
```

## Cache and FASTA Data

This app intentionally does not bundle Ensembl VEP caches, FASTA files, dbNSFP,
ClinVar, gnomAD, CADD, AlphaMissense, SpliceAI, or other large annotation data.
Those resources are species-, assembly-, release-, and license-dependent.

For VEP 116.1, use Ensembl VEP cache version `116`. Patch release `116.1` keeps
the Ensembl API/cache generation at `116`. Prepare a persistent cache
directory in your project or another host-visible location:

```sh
mkdir -p vep-data
taf-vep INSTALL.pl -a cf -s homo_sapiens -y GRCh38 -c "$PWD/vep-data"
```

Then run VEP against that directory:

```sh
taf-vep -- --cache --offline --dir_cache "$PWD/vep-data" \
  --species homo_sapiens --assembly GRCh38 \
  -i variants.vcf -o variants.vep.txt --force_overwrite
```

The installer downloads data from Ensembl and therefore requires network
access. Flow-oriented or reproducible production runs should pre-populate the
cache directory and run VEP with `--offline`. If you use external plugin data
or custom annotations, keep those files in explicit project/reference
directories and pass the corresponding VEP options.

## Command Mode

Command mode is enabled. Option-leading arguments go to the default `vep`
command through `taf-vep -- ...`; explicit commands run inside the same
container:

```sh
taf-vep -- --help
taf-vep vep --help
taf-vep filter_vep --help
```

## Platform

The image is built for native `linux/amd64` and `linux/arm64`. The packaged
runtime comes from the official Ensembl multi-architecture Docker image.

## Boundaries

This app packages the VEP runtime and helper tools. It does not package
production annotation caches, reference FASTA bundles, Ensembl MySQL mirrors,
Nextflow, Docker/Singularity itself, or third-party databases required by
specific plugins. Commands such as `INSTALL.pl -a c`, plugin data downloads,
remote database mode, and `variant_recoder` database lookups may need network
access unless all required data is already local.

Smoke tests validate the VEP `116.1` runtime and `116.1` source constants,
helper command help, Perl modules for HTS and BigWig access, htslib utilities,
`filter_vep` on a synthetic VEP-CSQ VCF, and a real offline VEP annotation path
using tiny synthetic GFF3, FASTA, and VCF files. Smoke does not download
production caches or validate biological annotation correctness on real
genomes.

## Upstream

- homepage: <https://www.ensembl.org/info/docs/tools/vep/index.html>
- repository: <https://github.com/Ensembl/ensembl-vep>
- tag: <https://github.com/Ensembl/ensembl-vep/tree/release/116.1>
- Docker image: <https://hub.docker.com/r/ensemblorg/ensembl-vep>
- command-line documentation: <https://www.ensembl.org/info/docs/tools/vep/script/index.html>
- cache documentation: <https://www.ensembl.org/info/docs/tools/vep/script/vep_cache.html>
- upstream license: Apache-2.0

If you use Ensembl VEP in academic work, cite:

McLaren W, Gil L, Hunt SE, Riat HS, Ritchie GRS, Thormann A, Flicek P,
Cunningham F. The Ensembl Variant Effect Predictor. Genome Biology.
2016;17:122. doi: `10.1186/s13059-016-0974-4`, PMID: `27268795`.
