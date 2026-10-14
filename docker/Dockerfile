FROM ensemblorg/ensembl-vep:release_116.3@sha256:2a46b0ba4ac544a87136aad92af92050ccbbf09098c1ce306d7e88ef7ed19aa3
# 构建来源维护：主体来自此 digest 固定的官方多架构镜像；固定身份不等于镜像内容已在本项目备份。
# 拉取失败先核对仓库、平台和 digest 可获得性，不去掉 digest 或直接追随可变化的 release tag。
# 若需替换，核实官方版本、逐平台源码身份及 cache/plugin 兼容后重走验证；已发布身份通过后继 release 修复。

ARG VEP_VERSION=116.3
ARG VEP_CACHE_VERSION=116
# SOURCE_REF/SHA256 在此记录来源身份，并未用于下载源码；不能把它们当作完整源码重建配方。
# 实际镜像源码校验由 vep-smoke deps 使用 docker/upstream-code.sha256 执行；换镜像后不能盲目重生清单放行。
ARG VEP_SOURCE_REF=f479099e2a9cd86887a48cf5c293fd7a208ddfb9
ARG VEP_SOURCE_SHA256=401c50c126e04254ec34993fe5a3c0ba122fa3631114135b1dd5d3ef889f5d42

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=vep
ENV TAFFISH_VERSION=${VEP_VERSION}
ENV TAFFISH_RELEASE=1
ENV VEP_VERSION=${VEP_VERSION}
ENV VEP_CACHE_VERSION=${VEP_CACHE_VERSION}
ENV VEP_DIR_CACHE=/opt/taffish/vep-cache
ENV VEP_SOURCE_REF=${VEP_SOURCE_REF}
ENV VEP_SOURCE_SHA256=${VEP_SOURCE_SHA256}

USER root
# 附加 Perl/Python 包来自活动 APT 仓库，官方 image digest 不锁定这次安装的依赖闭包。
# 缺包先区分网络/索引故障与仓库变化；恢复旧包需核实同版本来源，采用新依赖后重验统计解析与资源 helper。
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends libhtml-tableextract-perl python3-minimal; \
    rm -rf /var/lib/apt/lists/*

COPY docker/vep-install-data-116 /usr/local/bin/vep-install-data-116
COPY docker/vep-stats-view docker/vep-smoke docker/vep-install-plugin-data /usr/local/bin/
COPY docker/resources /opt/taffish/share/vep/resources
COPY docker/upstream-code.sha256 /opt/taffish/share/vep/upstream-code.sha256
RUN set -eux; \
    chmod 0755 /usr/local/bin/vep-install-data-116 /usr/local/bin/vep-stats-view /usr/local/bin/vep-smoke /usr/local/bin/vep-install-plugin-data; \
    mkdir -p /opt/taffish/share/vep/smoke /opt/taffish/vep-cache /opt/taffish/vep-plugin-data; \
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
    printf "%b" "##fileformat=VCFv4.2\n##VEP=\"v116.3\" API=\"v116\"\n##INFO=<ID=CSQ,Number=.,Type=String,Description=\"Consequence annotations from Ensembl VEP. Format: Allele|Consequence|IMPACT|SYMBOL|Gene|Feature_type|Feature|BIOTYPE\">\n#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\nchr1\t103\t.\tG\tA\t.\t.\tCSQ=A|missense_variant|MODERATE|GENE1|gene1|Transcript|tx1|protein_coding\nchr1\t150\t.\tG\tA\t.\t.\tCSQ=A|synonymous_variant|LOW|GENE1|gene1|Transcript|tx1|protein_coding\n" > /opt/taffish/share/vep/smoke/filter-input.vcf; \
    chmod -R a+rX /opt/taffish

USER vep
WORKDIR /data

# Build-time gates are version/import/source identity and tiny CLI only.
# Browser rendering and complete backend/resource checks belong to runtime QA.
RUN vep-smoke version && vep-smoke deps && vep-smoke gff && vep-smoke filter
