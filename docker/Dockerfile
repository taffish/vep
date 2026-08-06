FROM ensemblorg/ensembl-vep:release_116.1@sha256:6972667883d89b9fa55e5cbe94f1b4c259ebc6bbd178310cef78f617e055eb75

ARG VEP_VERSION=116.1
ARG VEP_CACHE_VERSION=116
ARG VEP_SOURCE_REF=97c24b205f4ba8b44ccc75b839512a1d59104019
ARG VEP_SOURCE_SHA256=959545e10ed443ff21d040a4fd277c640b9258bb2a4f08fe4355f702ed53b6ec

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=vep
ENV TAFFISH_VERSION=${VEP_VERSION}
ENV TAFFISH_RELEASE=1
ENV VEP_VERSION=${VEP_VERSION}
ENV VEP_CACHE_VERSION=${VEP_CACHE_VERSION}
ENV VEP_SOURCE_REF=${VEP_SOURCE_REF}
ENV VEP_SOURCE_SHA256=${VEP_SOURCE_SHA256}

USER root
RUN set -eux; \
    mkdir -p /opt/taffish/share/vep/smoke; \
    perl -e 'print ">chr1\n", "N" x 99, "ATG", "GAA" x 20, "TAA", "N" x 200, "\n"' > /opt/taffish/share/vep/smoke/ref.fa; \
    printf "%s\n" \
      "##gff-version 3" \
      "chr1	taf	gene	100	165	.	+	.	ID=gene1;Name=GENE1" \
      "chr1	taf	mRNA	100	165	.	+	.	ID=tx1;Parent=gene1;Name=TX1" \
      "chr1	taf	exon	100	165	.	+	.	ID=exon1;Parent=tx1" \
      "chr1	taf	CDS	100	165	.	+	0	ID=cds1;Parent=tx1" \
      > /opt/taffish/share/vep/smoke/genes.gff3; \
    bgzip -c /opt/taffish/share/vep/smoke/genes.gff3 > /opt/taffish/share/vep/smoke/genes.gff3.gz; \
    tabix -p gff /opt/taffish/share/vep/smoke/genes.gff3.gz; \
    printf "%b" "##fileformat=VCFv4.2\n#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\nchr1\t103\t.\tG\tA\t.\t.\t.\n" > /opt/taffish/share/vep/smoke/input.vcf; \
    printf "%b" "##fileformat=VCFv4.2\n##VEP=\"v116.1\" API=\"v116\"\n##INFO=<ID=CSQ,Number=.,Type=String,Description=\"Consequence annotations from Ensembl VEP. Format: Allele|Consequence|IMPACT|SYMBOL|Gene|Feature_type|Feature|BIOTYPE\">\n#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\nchr1\t103\t.\tG\tA\t.\t.\tCSQ=A|missense_variant|MODERATE|GENE1|gene1|Transcript|tx1|protein_coding\nchr1\t150\t.\tG\tA\t.\t.\tCSQ=A|synonymous_variant|LOW|GENE1|gene1|Transcript|tx1|protein_coding\n" > /opt/taffish/share/vep/smoke/filter-input.vcf; \
    chmod -R a+rX /opt/taffish

USER vep
WORKDIR /data

RUN set -eux; \
    vep --help 2>&1 | grep -F "ENSEMBL VARIANT EFFECT PREDICTOR" >/dev/null; \
    vep --help 2>&1 | grep -F "ensembl-vep          : ${VEP_VERSION}" >/dev/null; \
    haplo --help 2>&1 | grep -F "HAPLOSAURUS" >/dev/null; \
    variant_recoder --help 2>&1 | grep -F "VARIANT RECODER" >/dev/null; \
    filter_vep --help 2>&1 | grep -F "consequence data" >/dev/null; \
    INSTALL.pl --help 2>&1 | grep -F "default = ${VEP_CACHE_VERSION}" >/dev/null; \
    perl -MBio::EnsEMBL::VEP::Constants -e 'print join(qq(.), $Bio::EnsEMBL::VEP::Constants::VEP_VERSION, $Bio::EnsEMBL::VEP::Constants::VEP_SUB_VERSION), qq(\n)' | grep -Fx "${VEP_VERSION}" >/dev/null; \
    perl -MJSON -e 'print qq(JSON ok\n)' | grep -Fx "JSON ok" >/dev/null; \
    perl -MBio::DB::HTS -e 'print qq(HTS ok\n)' | grep -Fx "HTS ok" >/dev/null; \
    perl -MBio::DB::BigFile -e 'print qq(BigFile ok\n)' | grep -Fx "BigFile ok" >/dev/null; \
    command -v bgzip >/dev/null; \
    command -v tabix >/dev/null; \
    command -v htsfile >/dev/null; \
    command -v genesplicer >/dev/null; \
    test -s /plugins/CADD.pm; \
    test -s /plugins/AlphaMissense.pm

RUN set -eux; \
    tmp=/tmp/taf-vep-build-check; \
    rm -rf "$tmp"; \
    mkdir -p "$tmp"; \
    cp /opt/taffish/share/vep/smoke/ref.fa /opt/taffish/share/vep/smoke/genes.gff3.gz /opt/taffish/share/vep/smoke/genes.gff3.gz.tbi /opt/taffish/share/vep/smoke/input.vcf /opt/taffish/share/vep/smoke/filter-input.vcf "$tmp"/; \
    vep --gff "$tmp/genes.gff3.gz" --fasta "$tmp/ref.fa" --format vcf --vcf --force_overwrite --no_stats --species homo_sapiens -i "$tmp/input.vcf" -o "$tmp/out.vcf" > "$tmp/run.log" 2>&1; \
    grep -F "##VEP=\"v${VEP_VERSION}\"" "$tmp/out.vcf" >/dev/null; \
    grep -F "missense_variant" "$tmp/out.vcf" >/dev/null; \
    grep -F "GENE1" "$tmp/out.vcf" >/dev/null; \
    filter_vep --format vcf --filter "Consequence is missense_variant" -i "$tmp/filter-input.vcf" -o "$tmp/filter.vcf" --force_overwrite; \
    grep -F "missense_variant" "$tmp/filter.vcf" >/dev/null; \
    if grep -F "synonymous_variant" "$tmp/filter.vcf"; then exit 1; fi; \
    rm -rf "$tmp"
