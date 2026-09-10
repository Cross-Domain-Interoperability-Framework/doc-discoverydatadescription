# CDIF Data Description Profile

Repository for the CDIF Data Description profile — extends the Discovery profile with detailed variable documentation and data structure description using DDI-CDI concepts.

## Specification

- **[CDIFDataDescriptionImplementationGuide.md](CDIFDataDescriptionImplementationGuide.md)** — Implementation guide (Markdown source; companion **.docx** regenerated via pandoc with `--reference-doc` to preserve Word styles)
- **[CDIFDataDescriptionProfileStructuredSchema.json](CDIFDataDescriptionProfileStructuredSchema.json)** — JSON Schema for validation (synced from `metadataBuildingBlocks/_sources/profiles/cdifProfiles/CDIFDataDescriptionProfile/resolvedSchema.json`)
- **[CDIFDataDescription-frame.jsonld](CDIFDataDescription-frame.jsonld)** — JSON-LD frame used by `FrameAndValidate.py`
- **[dataDescriptionRules.shacl](dataDescriptionRules.shacl)** — Merged SHACL validation shapes (synced from `metadataBuildingBlocks` — see [Merged SHACL section](#shacl-validation) below)

## What Data Description adds to Discovery

The Data Description profile composes cdifCore + Discovery's variable / coverage / quality properties + additional Data-Description-only constraints:

- **`schema:variableMeasured`** items must have `@id` and `@type` including `cdi:InstanceVariable`, extended with the full DDI-CDI InstanceVariable / RepresentedVariable property set. See the [InstanceVariable across CDIF profiles](CDIFDataDescriptionImplementationGuide.md#variablemeasured) note in the IG for what Discovery, DataDescription, and DataStructure each carry. Key Data-Description-level properties:
  - `cdif:physicalDataType`, `cdif:role`, `cdif:simpleUnitOfMeasure`, `cdif:uses`
  - `cdi:function`, `cdi:platformType`, `cdi:source`, `cdi:hasIntendedDataType`, `cdi:describedUnitOfMeasure`, `cdi:qualifies`
  - **Value-domain links** (added at the DataDescription profile level, not on the base InstanceVariable): `cdi:takesSentinelValuesFrom` (→ `cdif:SentinelValueDomain` only), `cdi:takesSubstantiveValuesFrom` (→ `cdif:SubstantiveValueDomain` only). Each value-domain node carries either an enumerated `cdif:takesValuesFrom` list (a `cdif:EnumerationDomain` referencing a SKOS scheme) or one or more `cdif:recommendedDataType` XSD type tokens.
  - **Per-variable summary statistics**: `cdif:isDescribedBy_StatisticsCollection` → `cdif:StatisticsCollection` (groups one or more `cdi:Statistics` nodes, indexed by InstanceVariables).
- **`schema:distribution`** items can be typed as `cdi:TabularTextDataSet`, `cdi:LongStructureDataSet`, or `cdi:StructuredDataSet` with CSVW properties (`csvw:delimiter`, `csvw:header`, `csvw:headerRowCount`, etc.) and CDIF file characterization (`cdi:characterSet`, `cdif:fileSize`, `cdif:fileSizeUofM`).
- **`cdif:hasPhysicalMapping`** on distributions links physical columns/paths to `schema:variableMeasured` items via `cdif:formats_InstanceVariable` references, with `cdif:index` / `cdif:locator`, `cdif:format`, `cdif:physicalDataType`, plus `cdi:length`/`nullSequence`/`defaultValue`/`scale`/`decimalPositions`/etc. Each mapping item conforms to [CdifPhysicalMapping](CDIFDataDescriptionImplementationGuide.md#sec-cdifphysicalmapping).
- **WebAPI distributions** — the `schema:potentialAction.schema:result` (a `schema:DataDownload`) can additionally be typed `cdi:PhysicalDataSet` (or subclass) and carry the same physical-realization properties as a sibling DataDownload distribution: `cdi:characterSet`, `cdif:fileSize`/`fileSizeUofM`, `cdif:hasPhysicalMapping` (whose `cdif:formats_InstanceVariable` references the parent dataset's `schema:variableMeasured` @ids — the API response is another physical realization of the same conceptual variables). `cdi:PhysicalDataSet` typing belongs on the result, NOT on the WebAPI distribution itself.
- **Dataset-level extensions**: `cdif:hasPrimaryKey` (a `cdif:Key` whose `cdif:isComposedOf` is an ordered list of `cdi:ComponentPosition` wrappers — matches the canonical DDI-CDI PrimaryKey shape), `cdif:statistics` (dataset-wide `cdi:Statistics` / `cdif:StatisticsCollection`).
- **`dcterms:conformsTo`** on the catalog record (`schema:subjectOf`) must include `https://w3id.org/cdif/manifest/1.1` and `https://w3id.org/cdif/data_description/1.1` in addition to core and discovery URIs.

## Examples

The `examples/` directory contains 7 validated JSON-LD examples at the DataDescription level. All use DataDescription-specific properties (not just Discovery-level metadata). Records that only had `cdi:InstanceVariable` typing without physical data types or mappings have been moved to Discovery or packaging repositories.

### DDI Codebook conversions (Harvard Dataverse)

Converted from DDI Codebook 2.5 XML via `validation/DDI/ddi_to_cdif.py`. Tab file headers fetched from Dataverse to build physical mappings. File sizes and MD5 checksums from Dataverse file API.

| File | Domain | Vars | Files | Rows | DD properties |
|------|--------|------|-------|------|---------------|
| `ddi-cost-of-cash-datadesc.json` | Economics | 75 | 3 | 1,256 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |
| `ddi-expert-party-space-datadesc.json` | Political science | 74 | 2 | 401 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |
| `ddi-zoonotic-disease-datadesc.json` | Climate & disease | 7 | 1 | 315 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |
| `ddi-bird-monitoring-datadesc.json` | Biodiversity | 82 | 5 | 2,429 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |

### Simplified from validation MetadataExamples

Provenance extensions stripped; retain only DataDescription-specific properties.

| File | Domain | DD properties |
|------|--------|---------------|
| `cdif-datadesc-example.json` | General | physicalDataType, hasPhysicalMapping, TabularTextDataSet, StructuredDataSet, csvw |
| `nwis-longdata-datadesc.json` | Water quality | physicalDataType, role, hasPhysicalMapping, LongStructureDataSet, csvw |
| `xas-na2so4-datadesc.json` | XAS spectroscopy | physicalDataType, uses |

All validated against CDIFDataDescriptionProfile structured schema.

## Sources of variable-level metadata

Investigation of repositories for variable-level metadata (April 2026):

| Source | Variable-level DDI? | Notes |
|--------|---------------------|-------|
| **Harvard Dataverse** | **Yes** | DDI export includes full `<dataDscr>` with variable names, labels, formats, categories for ingested .tab files |
| **Borealis Dataverse** | No | DDI export available but no variable descriptions |
| **SND / researchdata.se** | No | Study-level only in all formats (DDI 2.5, DDI 3.3, JSON-LD). Reports `variableQuantity` but doesn't expose variable list |
| **CESSDA Data Catalogue** | No | OAI-PMH DDI 2.5 is study-level only |
| **UK Data Service** | No | DDI via OAI-PMH has no `<dataDscr>` section |
| **GESIS** | Inaccessible | API blocked by SPA |
| **ICPSR** | Inaccessible | Cloudflare bot protection |
| **PANGAEA** | Partial | `variableMeasured` (names, units) but no DDI-CDI structure |

### Harvard Dataverse DDI export

```
GET https://dataverse.harvard.edu/api/datasets/export?exporter=ddi&persistentId={doi}
```

Returns DDI Codebook 2.5 with `<dataDscr>` including `<var>` elements (name, label, format, interval, summary statistics, file reference) and `<fileDscr>` with dimensions (`caseQnty`, `varQnty`). Only for datasets with ingested .tab files.

### Conversion tool

`validation/DDI/ddi_to_cdif.py` converts DDI Codebook to CDIF DataDescription JSON-LD:

```bash
python DDI/ddi_to_cdif.py input.xml --doi https://doi.org/10.7910/DVN/XXX \
  --fetch-headers --fetch-file-meta -o output.json
```

## JSON-LD Framing and Validation

**`FrameAndValidate.py`** frames a JSON-LD document against the Data Description profile schema and optionally validates it:

```bash
# Frame and validate
python FrameAndValidate.py examples/cdif-datadesc-example.json --validate

# Frame and save output
python FrameAndValidate.py examples/cdif-datadesc-example.json -o framed.json
```

The script uses **`CDIFDataDescription-frame.jsonld`** to frame JSON-LD documents into the expected property structure. Context prefixes from the input document are automatically merged into the frame.

**Requirements:** `pyld`, `jsonschema` (`pip install pyld jsonschema`)

## SHACL Validation

**`dataDescriptionRules.shacl`** contains self-contained SHACL shapes for validating CDIF Data Description profile instances (renamed from generic `rules.shacl` so it is discoverable in the release repo without ambiguity). The file is **generated**, not hand-curated: it bundles the `rules.shacl` of every Building Block in the profile's `$ref`-closure — i.e. exactly the BBs the profile actually composes (`schema.yaml` `$ref` graph), and nothing else. (Notably this does **not** include `cdifArchiveDistribution`, which the Data Description profile does not compose; its `CDIFManifestConformsToShape` therefore is not pulled in.)

Regenerate it from [`metadataBuildingBlocks`](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks) whenever a source `rules.shacl` changes:

```bash
python tools/validate_shacl.py CDIFDataDescriptionProfile \
  --emit-shapes <path-to>/datadescription/dataDescriptionRules.shacl
```

The merge uses **shape-name precedence**: when the same named SHACL shape is defined in multiple sources, the profile's own `rules.shacl` wins (so it can override base shapes like `cdifd:CDIFDatasetMandatoryShape` / `cdifd:CDIFCatalogRecordShape`), otherwise the BB whose directory name appears in the shape's local name wins (e.g. `definedTerm` owns `cdifd:CDIFDefinedTermShape`); the winner's full subgraph is emitted and the losers dropped. This avoids the "PropertyShape with multiple sh:path" error pyshacl raises on a naive union.

## Related repositories

- **[cdif-core](https://github.com/Cross-Domain-Interoperability-Framework/core)** — CDIF Core profile
- **[Discovery](https://github.com/Cross-Domain-Interoperability-Framework/Discovery)** — CDIF Discovery profile
- **[packaging](https://github.com/Cross-Domain-Interoperability-Framework/packaging)** — Archive bundles and RO-Crate (records moved here that are bundles without DD properties)
- **[data-structure-description](https://github.com/Cross-Domain-Interoperability-Framework/data-structure-description)** — DDI-CDI mapping development and test data
- **[metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks)** — Building block schemas
- **[validation](https://github.com/Cross-Domain-Interoperability-Framework/validation)** — Validation tools, DDI converter, DCAT converter, harvester

## Changelog — v1.1.0

Released 2026-09-10 as `v1.1.0`. Content synced from the CDIF
**metadataBuildingBlocks** source; see the
[release](../../releases/tag/v1.1.0) for the tagged snapshot and
`git log v1.1.0` for the per-commit history:

- **Populated from metadataBuildingBlocks** — `*StructuredSchema.json`, merged SHACL,
  JSON-LD frame, examples, and the normative `FrameAndValidate.py` generated from the
  building-block source; `Examples/` renamed to `examples/`.
- **CDIF v1.1** — profile conformance URIs migrated `/1.0` → `/1.1`.
- **License** standardized on CC-BY-4.0.
- **`@id`-reference tightening** — bare `{@id}` reference slots sealed
  (`additionalProperties: false` + `required: ['@id']`); a canonical `objectReference`
  building block introduced as the strict node reference.
- **`prov:used` wrapper reconciliation** — the base `generatedBy.prov:used` accepts
  role-keyed wrappers (`schema:instrument` / `bios:computationalTool` / `prov:reagent`)
  alongside string / `{@id}` / inline `prov:Entity`; profiles pin a wrapper's shape via
  a constraint-only `if/then` (never a narrowed `anyOf`).
- **`skos:notation` → single string** at concept level (consistent with the codelist
  single-notation design).
- **`FrameAndValidate.py`** (normative, drift-checked against
  `Cross-Domain-Interoperability-Framework/validation`) — two-frame root-`@type`
  selection, context-aware `schema:about`, `--conformance` detection, `cdif:`-`@id`
  re-expansion, and (2026-08) reference-collapse on all document types + blank-node
  dedupe + agent `schema:identifier` unwrap, so `@embed:@always`-framed documents
  validate against the tightened schemas.
- **Examples** conformed to the tightened schemas throughout (PrimaryKey →
  `cdi:ComponentPosition`, reference slots → `{@id}`, CVE `hasIntendedDataType` →
  string, `skos:notation` → string, `schema:additionalType` URI → `{@id}`).


## Branches

`main` is the **current release** — GitHub Pages serves it, so the published
URLs always show the newest release. It is protected: changes reach it only by
pull request, which means the merge *is* the release.

New work goes on the **`updates`** branch and is merged to `main` when a release
is cut, then tagged `v1.1.n`. The former `reviewRevision202606` branch is retained
as **`archive202609`**.


## License

See [LICENSE](LICENSE).
