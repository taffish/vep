taf-vep 116.3-r1

Annotate variants with VEP; generate an optional standalone offline statistics viewer.
Choose a backend: export TAFFISH_CONTAINER_BACKEND=docker (or podman/apptainer).
Apptainer requires Linux. Prepared resources and input files must be readable.

Usage:
  taf-vep -- --gff genes.gff3.gz --fasta genome.fa.gz --format vcf --vcf \
    --stats_text --stats_file statistics.txt -i variants.vcf -o annotated.vcf
  taf-vep vep-stats-view --input statistics.txt --output statistics.html
  Open statistics.html in your host browser. No server or network is needed.
  Output exists? Choose a new name; VEP uses --force_overwrite, the viewer --force.
  Original VEP HTML uses online Google Charts; use --stats_text for offline viewing.

Use a prepared cache-116 member:
  TAFFISH_VEP_DATASET=homo_sapiens--GRCh38 \
    taf-vep -- --cache --offline --species homo_sapiens --assembly GRCh38 \
    -i variants.vcf -o annotated.vcf --vcf --stats_text --stats_file statistics.txt
  TAFFISH_VEP_DATA_PATH=/absolute/prepared-member overrides standard discovery.
  TAFFISH_VEP_AUTO_MOUNT=0 disables automatic cache and plugin-data mounts.
  RefSeq/merged caches also require VEP --refseq/--merged. Do not mix assemblies.

Install exactly one cache + FASTA member (network and free disk required):
  mkdir -p "$HOME/.local/share/taffish/databases/vep"
  export TAFFISH_VEP_DATA_INSTALL_ROOT="$HOME/.local/share/taffish/databases/vep"
  taf-vep vep-install-data-116 --dry-run --species homo_sapiens --assembly GRCh38
  Add --resolve for online exact filenames, checksums and compressed-size metadata.
  Docker:
    TAFFISH_CONTAINER_BACKEND=docker taf-vep vep-install-data-116 \
      --species homo_sapiens --assembly GRCh38
  Podman:
    TAFFISH_CONTAINER_BACKEND=podman taf-vep vep-install-data-116 \
      --species homo_sapiens --assembly GRCh38
  Apptainer:
    TAFFISH_CONTAINER_BACKEND=apptainer taf-vep vep-install-data-116 \
      --species homo_sapiens --assembly GRCh38
  unset TAFFISH_VEP_DATA_INSTALL_ROOT
  Retry ordinary download failures. For killed processes or stale staging/backups:
    set the same install root; run the same species/assembly with --recover first.
  --force replaces verified data; cleanup-pending means new data are ready: recover.
  Administrators may use /usr/local/share/taffish/databases/vep or
  /opt/taffish/databases/vep once, then grant ordinary users read/traverse access.
  Deep-verify without writes (select the same species/assembly member):
    TAFFISH_VEP_DATASET=homo_sapiens--GRCh38 taf-vep vep-install-data-116 \
      --species homo_sapiens --assembly GRCh38 --verify-root /opt/taffish/vep-cache

Install optional plugin data into the same explicit install root:
  taf-vep vep-install-plugin-data --member alphamissense-hg38-20230803 --dry-run
  Review the terms; set TAFFISH_VEP_DATA_INSTALL_ROOT as above, then run:
    taf-vep vep-install-plugin-data --member alphamissense-hg38-20230803 --accept-terms
  Unset the install root before analysis; select the prepared plugin member:
    export TAFFISH_VEP_PLUGIN_DATASET=alphamissense-hg38-20230803
  Add to VEP: --plugin AlphaMissense,file=/opt/taffish/vep-plugin-data/AlphaMissense_hg38.tsv.gz
  Override with TAFFISH_VEP_PLUGIN_DATA_PATH=/absolute/plugin-member.
  Custom pinned resource recipes use --spec recipe.json; see README for format.
  Verify read-only: taf-vep vep-install-plugin-data --verify-root /opt/taffish/vep-plugin-data

Manual cache mount fallback (plugin data use /opt/taffish/vep-plugin-data instead):
  Docker:
    TAFFISH_CONTAINER_BACKEND=docker TAFFISH_VEP_AUTO_MOUNT=0 \
    TAFFISH_DOCKER_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
    taf-vep -- --cache --offline -i variants.vcf -o out.txt
  Podman:
    TAFFISH_CONTAINER_BACKEND=podman TAFFISH_VEP_AUTO_MOUNT=0 \
    TAFFISH_PODMAN_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
    taf-vep -- --cache --offline -i variants.vcf -o out.txt
  Apptainer:
    TAFFISH_CONTAINER_BACKEND=apptainer TAFFISH_VEP_AUTO_MOUNT=0 \
    TAFFISH_APPTAINER_RUN_ARGS="--bind /abs/member:/opt/taffish/vep-cache:ro" \
    taf-vep -- --cache --offline -i variants.vcf -o out.txt

Filter and quoting:
  taf-vep filter_vep -i annotated.vcf -o missense.vcf --format vcf \
    --filter "'Consequence is missense_variant'"
  For a spaced title/path, retain literal quotes: --title "'My VEP statistics'".
  Keep resource roots free of spaces, quotes, colons and newlines.

Outputs: annotated VCF/CSQ, VEP text or JSON; statistics, warnings and chosen files.
Inputs and results remain yours; control report access if they contain sample metadata.
Cache generation stays 116. No database, prediction score or model is bundled.

More help:
  taf-vep -- --help                    Upstream VEP help (no --version option).
  taf-vep --help                       This installed-user help.
  taf-vep --version                    Wrapper identity.
  taf-vep --compile                    Generated command.
  https://github.com/taffish/vep
