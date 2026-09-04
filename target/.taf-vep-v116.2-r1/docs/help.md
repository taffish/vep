taf-vep 116.2-r1

Purpose:
  Annotate variants with Ensembl VEP 116.2, or run its packaged helper commands.

Usage:
  taf-vep -- --help
  taf-vep -- --gff annotation.gff3.gz --fasta genome.fa.gz \
    --format vcf --vcf --force_overwrite -i variants.vcf -o out.vcf
  taf-vep filter_vep --help

Prepared cache use:
  Select exactly one cache-116 species/assembly member:
    TAFFISH_VEP_DATASET=homo_sapiens--GRCh38 \
    taf-vep -- --cache --offline --species homo_sapiens --assembly GRCh38 \
      -i variants.vcf -o out.txt --force_overwrite

  Override discovery with TAFFISH_VEP_DATA_PATH=/absolute/prepared-member.
  Disable discovery with TAFFISH_VEP_AUTO_MOUNT=0.
  RefSeq/merged members also require VEP's --refseq/--merged analysis option.

Install one cache + FASTA member:
  Preview without download or writes:
    taf-vep vep-install-data-116 --dry-run \
      --species homo_sapiens --assembly GRCh38

  Docker:
    TAFFISH_CONTAINER_BACKEND=docker \
    TAFFISH_VEP_DATA_INSTALL_ROOT="$HOME/.local/share/taffish/databases/vep" \
    taf-vep vep-install-data-116 --species homo_sapiens --assembly GRCh38

  Podman:
    TAFFISH_CONTAINER_BACKEND=podman \
    TAFFISH_VEP_DATA_INSTALL_ROOT="$HOME/.local/share/taffish/databases/vep" \
    taf-vep vep-install-data-116 --species homo_sapiens --assembly GRCh38

  Apptainer:
    TAFFISH_CONTAINER_BACKEND=apptainer \
    TAFFISH_VEP_DATA_INSTALL_ROOT="$HOME/.local/share/taffish/databases/vep" \
    taf-vep vep-install-data-116 --species homo_sapiens --assembly GRCh38

  Installation needs network and substantial free space. Analysis reuses the
  completed member read-only and should normally use --offline. The helper never
  downloads all members implicitly.

Manual prepared-member mounts:
  Docker:
    TAFFISH_VEP_AUTO_MOUNT=0 \
    TAFFISH_DOCKER_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
    taf-vep -- --cache --offline -i variants.vcf -o out.txt

  Podman:
    TAFFISH_VEP_AUTO_MOUNT=0 \
    TAFFISH_PODMAN_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
    taf-vep -- --cache --offline -i variants.vcf -o out.txt

  Apptainer:
    TAFFISH_VEP_AUTO_MOUNT=0 \
    TAFFISH_APPTAINER_RUN_ARGS="--bind /abs/member:/opt/taffish/vep-cache:ro" \
    taf-vep -- --cache --offline -i variants.vcf -o out.txt

Inputs:
  VCF, VEP text, HGVS/identifier/coordinate forms, or indexed GFF3/GTF plus FASTA.

Key outputs:
  VEP text, VCF with CSQ annotations, JSON, statistics and command-specific files.

Immediate notes:
  vep has no --version option; taf-vep -- --help shows ensembl-vep 116.2.
  Cache/API generation remains 116. Plugin data are not bundled.
  Invalid explicit resource roots fail closed; no selector means no auto-mount.

More help:
  taf-vep vep --help
  taf-vep INSTALL.pl --help
  https://www.ensembl.org/info/docs/tools/vep/script/index.html
  https://github.com/taffish/vep

Wrapper options:
  taf-vep --help       Show this TAFFISH help.
  taf-vep --version    Show TAFFISH wrapper version.
  taf-vep --compile    Print the generated wrapper shell.
  taf-vep -- --help    Pass option-leading arguments to default vep.
