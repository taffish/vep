# taf-vep

Ensembl Variant Effect Predictor **116.3**, TAFFISH release **116.3-r1**.
Annotate variants with VEP, run its packaged CLI helpers, or view its text
statistics in an independent standalone offline HTML page.

## Identity and upstream scope

| Identity | Value |
| --- | --- |
| Upstream tag | `release/116.3` |
| Source commit | `f479099e2a9cd86887a48cf5c293fd7a208ddfb9` |
| Fixed commit archive SHA-256 | `401c50c126e04254ec34993fe5a3c0ba122fa3631114135b1dd5d3ef889f5d42` |
| Official base OCI index | `sha256:2a46b0ba4ac544a87136aad92af92050ccbbf09098c1ce306d7e88ef7ed19aa3` |
| Cache / Ensembl API generation | `116`, not `116.3` |
| Native platforms | `linux/amd64`, `linux/arm64` |
| App image | `ghcr.io/taffish/vep:116.3-r1` |

The image extends the pinned official VEP image, preserving upstream annotation
code and packaged plugins. Build and runtime checks compare core CLI files,
modules and LICENSE against a SHA-256 inventory from the fixed official source.
No upstream scientific algorithm is patched. This release includes upstream fixes
for allele-frequency matching/zero values, ncRNA_gene GFF records and custom VCF
annotations after 3-prime shifting.
[Upstream release](https://github.com/Ensembl/ensembl-vep/releases/tag/release/116.3).

VEP, filter_vep, Haplosaurus, Variant Recoder, INSTALL.pl and convert_cache.pl
remain callable through automatic command mode. INSTALL.pl remains unmodified;
TAFFISH resource helpers add persistent acquisition, verification and sharing.
The official image excludes the LoF plugin; it is not silently advertised here.
Optional plugins retain their own data, dependency, assembly and license needs.
The hosted Ensembl web interface and REST API are external services, not local
servers bundled by this app; their network availability is not guaranteed.

## Usage

Install this immutable app release: `taf install vep 116.3-r1`.
Installed-user help: `taf-vep --help`. Upstream help: `taf-vep -- --help`.
VEP itself has no standalone `--version`; its help identifies 116.3.

```sh
export TAFFISH_CONTAINER_BACKEND=docker  # or podman / apptainer
taf-vep -- --gff genes.gff3.gz --fasta genome.fa.gz --format vcf --vcf \
  --stats_text --stats_file statistics.txt -i variants.vcf -o annotated.vcf
taf-vep filter_vep --format vcf -i annotated.vcf -o missense.vcf \
  --filter "'Consequence is missense_variant'"
taf-vep vep-stats-view --input statistics.txt --output statistics.html
```

Inputs may be VCF, VEP text, HGVS/identifier/coordinate forms, or custom indexed
GFF3/GTF plus FASTA. Outputs include VEP text, annotated VCF/CSQ, JSON, statistics
and warnings. Use new output names by default. VEP overwrites only when
`--force_overwrite` is requested; the viewer uses `--force`.
Keep inputs/reference data readable and output directories writable.

The current TAFFISH automatic command mode requires literal quotes for values
containing spaces: `--title "'My VEP statistics'"` or
`--output "'report with spaces.html'"`. This is a wrapper argument boundary,
not a VEP input-format change. Resource root paths must not contain spaces,
quotes, colons or control characters.

## Static statistics viewer

VEP's original HTML contains legacy online Google Charts and remote logos.
It is preserved, but is not an offline visualization contract. For offline work
request upstream `--stats_text --stats_file statistics.txt`, then invoke
`vep-stats-view`. The helper adds a local, standalone HTML presentation and
does not modify annotation output or upstream HTML.

It preserves every original section, row, label and value; numeric two-column
sections are also shown as horizontal bars in upstream order. Bars use a zero
baseline and a per-section maximum, not a cross-section or percentage scale.
Counts of variants and transcript consequences do not necessarily share a
denominator. Original TXT download is byte-identical to the input, with its
SHA-256 displayed. This viewer is not scientific validation or a diagnostic tool.

The page needs no JavaScript, CDN, font download, service, port, GPU or display.
Native HTML navigation/details and an original-data download work from file://.
The same generation commands apply to Docker, Podman and Apptainer by selecting
`TAFFISH_CONTAINER_BACKEND`; copy the resulting HTML to a local browser if
analysis ran on SSH. No noVNC/service lifecycle is applicable. Reports may contain
sample metadata, paths and command arguments; apply your normal access policy.

## Cache and FASTA resources

The smallest managed unit is one explicit cache-116 species/assembly plus its
matching reference FASTA, e.g. `homo_sapiens--GRCh38`.
RefSeq/merged caches use their exact species suffix and the corresponding VEP
`--refseq`/`--merged` analysis option. No implicit `all` selection is accepted.

```sh
mkdir -p "$HOME/.local/share/taffish/databases/vep"
export TAFFISH_VEP_DATA_INSTALL_ROOT="$HOME/.local/share/taffish/databases/vep"
taf-vep vep-install-data-116 --dry-run --species homo_sapiens --assembly GRCh38
# Optional online metadata resolution; no archive download or disk writes:
taf-vep vep-install-data-116 --dry-run --resolve --species homo_sapiens --assembly GRCh38
taf-vep vep-install-data-116 --species homo_sapiens --assembly GRCh38
unset TAFFISH_VEP_DATA_INSTALL_ROOT
TAFFISH_VEP_DATASET=homo_sapiens--GRCh38 taf-vep -- --cache --offline \
  --species homo_sapiens --assembly GRCh38 -i variants.vcf -o annotated.vcf --vcf
```

Set the backend explicitly to Docker, Podman or Apptainer before either command.
The wrapper binds the explicit installation parent at /data-install and rejects
an absent parent. The helper requires both the marker and an actual mount; a
marker alone cannot turn an ephemeral container directory into a managed install.
Completed members are `release-116/SPECIES--ASSEMBLY` below that parent.

The helper fetches official HTTPS CHECKSUMS and exactly selected archives, verifies
both `sum -r` fields, records downloaded SHA-256, checks cache archive paths/types,
then delegates unpacking/indexing to unmodified INSTALL.pl using local archives.
FASTA selection is primary_assembly, then toplevel, with an exact assembly match;
GRCh37 FASTA comes from Ensembl release 75. Missing checksums/FASTA/index fail closed.
Official CHECKSUMS is a transport-integrity check, not a cryptographic publisher
signature; acquisition is protected by HTTPS and recorded SHA-256 thereafter.

Downloads/partials remain in a private .downloads directory for retry/resume.
A release lock, private staging, deep inventory checks and atomic promotion guard
installation; `--force` first verifies the old member and preserves it until the
replacement is ready. A failed preparation leaves that old member unchanged.
Corrupt complete downloads are retained for diagnosis with an exact retry message.
The ordinary dry-run is an offline identity/source plan. `--dry-run --resolve`
fetches official CHECKSUMS and HTTP HEAD metadata, reporting exact selected cache
and FASTA names/URLs, official sum-r/block checksums, catalog SHA-256 and compressed
bytes when Content-Length is supplied. Otherwise it reports unknown bytes and the
official block upper bound explicitly. It does not download archives or create a
lock/staging directory; a missing or ambiguous catalog entry fails closed.
The default 10 GiB free-space threshold is only a minimum, not an estimate of human
cache size. Allow for compressed archives, unpacked data/indices, and the old member
during replacement. A resolved plan records metadata at that time, not a promise
that the upstream catalog can never change; installation records its own receipts.

After a normal download failure, retry the same installation command; valid partial
archives resume. If the process was killed or a staging/backup remains, keep the
same explicit installation parent and run:

```sh
taf-vep vep-install-data-116 --species homo_sapiens --assembly GRCh38 --recover
# Then rerun installation, adding --force only when intentionally replacing data.
```

Recovery takes the release lock, verifies the current member before discarding old
transactions, or restores the single verified backup when the member is absent.
It discards only this member's owned, non-symlink staging/backup directories and
retains download cache. Invalid current data, ambiguous backups, unsafe paths or
foreign ownership stop recovery for manual review; it does not guess or erase them.
After successful promotion, failure to delete an old backup prints **replacement
committed; backup cleanup pending** and exits successfully: new READY remains valid.
Correct only the named backup's permissions if appropriate, then use `--recover`.
Do not start recovery while an installation is running or manually delete a sole
backup. Recovery/installation uses the same Docker/Podman/Apptainer selection above.

The manifest records exact member/API identity, source receipts, file count/bytes
and inventory SHA-256. A ready marker is written only after successful checks.
`--verify-root /opt/taffish/vep-cache --species ... --assembly ...` performs full
read-only validation without lock, chmod or a free-space requirement.

### Personal and system-wide reuse

Administrators can prepare either /usr/local/share/taffish/databases/vep or
/opt/taffish/databases/vep once, using the same explicit install-root variable.
They must first create and own the installation parent; this app never invokes
sudo or changes unrelated site permissions. Completed member directories/files
are 0755/0644. Protect parent ownership so ordinary users cannot modify shared data.

For `TAFFISH_VEP_DATASET`, discovery checks personal, /usr/local/share and
/opt/taffish roots in that order. An incomplete or broken-symlink higher-priority root
fails closed. `TAFFISH_VEP_DATA_PATH` overrides discovery; no selector means no
automatic cache. The wrapper checks identity/ready metadata and inventory digest,
then binds the physical root read-only at /opt/taffish/vep-cache.
This lightweight selection check does not rehash multi-gigabyte payloads;
run the deep verifier before first use and after suspected storage corruption.
Use `TAFFISH_VEP_AUTO_MOUNT=0` to disable all automatic data mounts.

Manual fallbacks (choose the exact species/assembly in the VEP arguments):

```sh
TAFFISH_CONTAINER_BACKEND=docker TAFFISH_VEP_AUTO_MOUNT=0 \
 TAFFISH_DOCKER_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
 taf-vep -- --cache --offline -i variants.vcf -o out.txt
TAFFISH_CONTAINER_BACKEND=podman TAFFISH_VEP_AUTO_MOUNT=0 \
 TAFFISH_PODMAN_RUN_ARGS="-v /abs/member:/opt/taffish/vep-cache:ro" \
 taf-vep -- --cache --offline -i variants.vcf -o out.txt
TAFFISH_CONTAINER_BACKEND=apptainer TAFFISH_VEP_AUTO_MOUNT=0 \
 TAFFISH_APPTAINER_RUN_ARGS="--bind /abs/member:/opt/taffish/vep-cache:ro" \
 taf-vep -- --cache --offline -i variants.vcf -o out.txt
```

Apptainer requires a native Linux host; it is not a macOS runtime.
Automatic Docker/Podman execution uses the caller's UID; Podman also uses keep-id.

## Plugin data, models and custom references

External plugin resources are separate from VEP cache/API identity. This app does
not load a GPU model; AlphaMissense uses precomputed scores. Public mechanical
resources are not exempted merely because they are optional. The versioned
`vep-install-plugin-data` helper accepts an explicit fixed recipe and installs one
member under `plugins/ID`, with resume/retry, checksum and byte-size checks, lock,
private staging, generated indices, SHA-256 inventory and atomic promotion.
Existing matching complete members are reused; different/corrupt members fail
closed. There is no destructive --force; choose a new immutable ID for new data.

A bundled recipe pins AlphaMissense hg38 to official GCS generation
1691073413649109 (2023-08-03), 642961469 compressed bytes, official MD5.
After installation all files and generated tabix indices have SHA-256 inventory.

```sh
taf-vep vep-install-plugin-data --member alphamissense-hg38-20230803 --dry-run
# Set TAFFISH_VEP_DATA_INSTALL_ROOT as for cache installation above.
# Review resource terms before explicitly opting in:
taf-vep vep-install-plugin-data --member alphamissense-hg38-20230803 --accept-terms
unset TAFFISH_VEP_DATA_INSTALL_ROOT
export TAFFISH_VEP_PLUGIN_DATASET=alphamissense-hg38-20230803
# Add to the regular VEP annotation command:
# --plugin AlphaMissense,file=/opt/taffish/vep-plugin-data/AlphaMissense_hg38.tsv.gz
taf-vep vep-install-plugin-data --verify-root /opt/taffish/vep-plugin-data
```

The personal/site parent search order is the same as cache data, but under
`plugins/ID`. Override via `TAFFISH_VEP_PLUGIN_DATA_PATH`; the read-only container
path is /opt/taffish/vep-plugin-data. Manual mounts use the same three-backend
examples above with this destination. The wrapper validates manifest digest;
the verifier validates every payload/index. No plugin is enabled implicitly.

Other mechanical resources use `--spec recipe.json`: schema
`taffish.vep.plugin-recipe.v1`; explicit `id`, `version`, `assembly`, `license`,
`citation`, and a nonempty `files` list. Each member has a safe basename `name`,
HTTPS `url` without embedded credentials, exact `bytes`, and `sha256` (preferred)
or official `md5`. Optional `index` is `vcf` or `alphamissense`; provide other
prebuilt indices as explicit files. The bundled JSON is a complete example.
A recipe selects known files; it is not an arbitrary shell recipe or model selector.

Resource licensing and boundaries:

| Resource | Smallest reusable unit and handling |
| --- | --- |
| Ensembl cache + FASTA | Exact species/assembly/API member; managed helper above; Ensembl data/source-provider terms, separate from software license. |
| AlphaMissense | Assembly-specific fixed score table + tabix index; managed fixed recipe; DeepMind CC-BY-4.0 and Google Cloud terms, retain citation. |
| CADD / CADD-SV | Version/assembly/type score file + index; non-commercial restrictions apply. Obtain appropriate permission and provide a pinned recipe or manually prepared read-only files; no blanket permission is implied. |
| dbNSFP / SpliceAI | Version/assembly-specific files; constituent predictors and resource licenses differ. User must choose lawful edition/usage, then use recipe acquisition where allowed, or manual read-only import if access requires personal credentials/acceptance. |
| ClinVar / gnomAD / custom annotations | Version/assembly-specific indexed data; pinned recipes can acquire allowed public files. Scientific field/filter/assembly selection remains explicit. |
| Project-specific GFF/GTF/FASTA | User input, not an automatically chosen public database; pre-index and mount read-only, retaining input checksums. |
| Remote Ensembl database / REST | Explicit online mode, not offline resource installation or a local shared database. |

Plugin data licenses are not inherited from the Apache-2.0 plugin code.
The helper records terms but does not grant usage rights or authenticate restricted
services. No production human cache/all-plugin validation is claimed merely from
tiny fixtures or one representative member.
[AlphaMissense plugin/data terms](https://github.com/Ensembl/VEP_plugins/blob/release/116/AlphaMissense.pm),
[CADD plugin](https://github.com/Ensembl/VEP_plugins/blob/release/116/CADD.pm),
[Ensembl data terms](https://www.ensembl.org/info/about/legal/disclaimer.html).

## Runtime write map and image contents

The host working directory (wrapper's same-path bind): user outputs/statistics/warnings. Custom FASTA/GFF
indices must exist or be generated in a writable copy. /tmp: independent smoke
scratch and temporary files. /data-install: explicit persistent resource
transactions/downloads only. /opt/vep, /plugins and prepared resource mounts:
read-only during analysis. No build-time or smoke writes to bundled fixtures.

The official image retains Perl/HTS/BigFile, Python2 for upstream plugins,
GeneSplicer and upstream installation/build dependencies. Python3 hosts the
TAFFISH standalone viewer/resource helper. We do not remove upstream plugin or
installation capabilities solely to make a small smoke pass. New apt metadata is
removed in its installation layer. Image size/profile and any retained build
dependencies are checked before release; later-layer deletions are not claimed as
image slimming.

## Validation and release procedure

The 2026-10-11 candidate passed native amd64 and arm64 app-root builds. Exact
manifest probes passed in fresh offline Docker/Podman containers (normal and
read-only roots) on both architectures, and in the same amd64 OCI's read-only
Apptainer SIF: 351 probes total. Final generated-wrapper tests passed on amd64
Docker/Podman/Apptainer and arm64 Docker/Podman: 120 checks including persistent
outputs, quoting, ordinary UID, original HTML, offline viewer and read-only cache
annotation. Earlier local Docker EOF/Podman timeout attempts are retained as
diagnostics; final reruns passed without changing upstream code.

One official Ciona cache/FASTA member was downloaded, checksum-verified, indexed
and used for annotation on both architectures. The repaired helper also completed
a real online --force replacement, followed by 17 three-backend verification,
annotation, lock/idempotence and different-ordinary-UID checks. Synthetic tiny
resources with unmodified INSTALL.pl cover 140 transaction/metadata checks across
the five tested platform/backend combinations: download failure/resume, force,
cleanup-pending success, promotion rollback, process interruption, safe recovery
and temporary names containing underscores. HTTPS transport and promotion errors
are injected only by private QA fixtures, never by patching production code.

72 further checks cover personal/site/opt discovery, priority, explicit override,
disable, incomplete/dangling roots, physical symlink resolution, deep verification
and read-only mounts on all three backends. Site paths are isolated in private
user/mount namespaces; real host site directories are unchanged. These mechanism
tests do not claim an actual privileged production site installation.

Earlier preparation of this release installed/indexed the fixed full AlphaMissense
table and verified it through three amd64 backends. The plugin helper, recipe and
upstream runtime are unchanged by this repair; that large resource was not
redownloaded for the transaction repair or separately repeated on arm64. Current
plugin discovery/verification, native tabix and helper negative checks pass. No
validation of every human cache/plugin or biological use case is claimed.

The independent viewer passed data-preservation/escaping/overwrite tests and
maintainer-confirmed file-based visual/interaction acceptance. The viewer source
has no external runtime requests or JavaScript. Browser automation/console capture
was unavailable; manual acceptance is recorded separately, not presented as an
automated browser test. Native arm64 Apptainer was not separately validated.

Final image sizes are 933,786,916 bytes (amd64) and 904,644,237 bytes (arm64).
Actual upstream notices and Debian copyright files are retained. The fixed
official base's installation/compiler/plugin dependencies remain intentional;
deleting them in a child layer would not reduce distributed base layers. New apt
lists are removed in the installation layer. No unsafe capability pruning or
image-size reduction claim is made. Publication and subsequent Action/Index
verification remain maintainer-controlled.

Maintainer commands from the app root:

```sh
taf check
taf build
docker build --platform linux/arm64 -f docker/Dockerfile -t ghcr.io/taffish/vep:116.3-r1 .
# Run exact manifest exist/tests fresh, offline; D/P read-only and actual SIF too.
taf publish --build --release --dry-run
```

Both declared native platforms must use app-root context, matching the canonical
Action. Build-time tests are source/version/import/tiny CLI checks, not browser
rendering or timing-sensitive GUI assertions. Final cleanup must leave only
target/.gitkeep after the last build/wrapper/dry-run. Canonical Action is unchanged
and byte-identical to the fresh taf new template. Publication is manual.

## License and citation

VEP and the TAFFISH packaging use Apache-2.0; upstream dependency and resource
licenses remain separate. Preserve upstream notices supplied in the fixed image.
Cite McLaren et al. (2016), *The Ensembl Variant Effect Predictor*,
Genome Biology 17:122, DOI [10.1186/s13059-016-0974-4](https://doi.org/10.1186/s13059-016-0974-4),
PMID 27268795, plus any selected plugin/database references.
