taf-vep 116.1-r1

Purpose:
  Run Ensembl Variant Effect Predictor (VEP) 116.1 and related helper tools
  inside a TAFFISH container.

Usage:
  taf-vep -- --help
  taf-vep -- --cache --offline --dir_cache "$PWD/vep-data" -i variants.vcf -o out.vcf
  taf-vep vep --help
  taf-vep filter_vep --help
  taf-vep COMMAND [ARGS...]

Common workflows:
  taf-vep -- --cache --offline --dir_cache "$PWD/vep-data" \
    --species homo_sapiens --assembly GRCh38 --format vcf --vcf \
    --force_overwrite -i variants.vcf -o variants.vep.vcf

  taf-vep -- --gff annotation.gff3.gz --fasta genome.fa.gz \
    --format vcf --vcf --force_overwrite -i variants.vcf -o variants.vep.vcf

  taf-vep filter_vep --format vcf --filter "Consequence is missense_variant" \
    -i variants.vep.vcf -o missense.vcf --force_overwrite

Packaged commands:
  vep              Default Variant Effect Predictor command.
  filter_vep       Filter VEP tabular or VCF/CSQ output.
  haplo            Haplosaurus phased haplotype consequence tool.
  variant_recoder  Translate variant IDs and HGVS-like encodings.
  INSTALL.pl       Download/install VEP caches, FASTA files, API, or plugins.

Upstream help and version:
  taf-vep -- --help
  taf-vep vep --help
  taf-vep INSTALL.pl --help

  VEP does not provide a vep --version option. The runtime version is printed
  in the vep --help banner as:
    ensembl-vep          : 116.1

Cache and FASTA:
  Production VEP annotation usually needs an Ensembl cache and often a FASTA.
  This app does not bundle those large, species-specific data files.

  Prepare human GRCh38 cache/FASTA for VEP 116:
    mkdir -p vep-data
    taf-vep INSTALL.pl -a cf -s homo_sapiens -y GRCh38 -c "$PWD/vep-data"

  Reuse it offline:
    taf-vep -- --cache --offline --dir_cache "$PWD/vep-data" \
      --species homo_sapiens --assembly GRCh38 -i variants.vcf -o out.txt

Inputs:
  VCF, VEP default input, HGVS-like identifiers, or coordinates depending on
  VEP options. GFF3/GTF custom transcript sources should be bgzip/tabix indexed
  and paired with a matching FASTA.

Key outputs:
  VEP tabular text, VCF with CSQ annotations, JSON, statistics files, and helper
  outputs depending on selected upstream options.

Platform:
  Native linux/amd64 and linux/arm64 images are built from the official Ensembl
  VEP release_116.1 Docker runtime. VEP 116.1 still uses cache/API version 116.

Boundaries:
  No production cache, FASTA bundle, Ensembl MySQL mirror, dbNSFP, ClinVar,
  gnomAD, CADD, AlphaMissense, SpliceAI, or plugin data is bundled.
  Commands that download caches or query remote databases require network
  access. For reproducible flows, pre-stage cache data and run with --offline.

Detailed documentation:
  https://www.ensembl.org/info/docs/tools/vep/index.html
  https://www.ensembl.org/info/docs/tools/vep/script/index.html
  https://www.ensembl.org/info/docs/tools/vep/script/vep_cache.html

Wrapper options:
  taf-vep --help       Show this TAFFISH help.
  taf-vep --version    Show TAFFISH wrapper version.
  taf-vep --compile    Compile the TAFFISH wrapper.
  taf-vep -- --help    Pass option-leading arguments to default vep.
