# Development Fixtures

Curated samples from public datasets and real Australian government documents
for extraction pipeline development. Each subset preserves the original ground
truth format of its source dataset.

## Datasets

### FUNSD

Form Understanding in Noisy Scanned Documents.

- **Authors:** Guillaume Jaume, Hazim Kemal Ekenel, Jean-Philippe Thiran
- **Paper:** [FUNSD: A Dataset for Form Understanding in Noisy Scanned Documents](https://arxiv.org/abs/1905.13538) (ICDAR-OST 2019)
- **License:** See dataset terms at <https://guillaumejaume.github.io/FUNSD/>
- **Format:** PNG images + JSON annotations (`{form: [{box, text, label, words, linking}]}`)
- **Transcripts:** `transcripts/` contains plain-text human transcripts for selected samples

### IAM Handwriting Database (line level)

- **Authors:** U. Marti, H. Bunke (University of Bern)
- **Paper:** [The IAM-database: an English sentence database for offline handwriting recognition](https://doi.org/10.1007/s100320200071) (IJDAR 2002)
- **Homepage:** <https://fki.tic.heia-fr.ch/databases/iam-handwriting-database>
- **License:** MIT (HuggingFace distribution by [Teklia](https://teklia.com))
- **Format:** PNG images + `.gt.txt` sidecar files (standard HTR convention)

### DocLayNet

- **Authors:** Birgit Pfitzmann, Christoph Auer, Michele Dolfi, Ahmed S. Nassar, Peter Staar (IBM Research)
- **Paper:** [DocLayNet: A Large Human-Annotated Dataset for Document-Layout Segmentation](https://arxiv.org/abs/2206.01062) (KDD 2022)
- **License:** CDLA-Permissive 2.0
- **Homepage:** <https://github.com/DS4SD/DocLayNet>
- **Format:** PNG images + JSON sidecar files (`{bboxes, labels, words, split}`)
- **Transcripts:** `*_transcript.txt` sidecar files contain human-readable text for selected samples

### Womblex Collection

Real Australian government documents covering the formats womblex targets.

- **Source:** Australian government document releases and open data portals
- **License:** Commonwealth of Australia (various Crown copyright terms)
- **Format:** Mixed — PDF, DOCX, CSV, XLSX with `_transcript.txt` sidecar files

#### Documents (`_documents/`)

| File | Format | Pages | Description | Ground Truth |
|------|--------|-------|-------------|--------------|
| `00768-213A-...Redacted` | PDF | 3 | ACT regulatory decision letter (redacted) | Human transcript with `<REDACTED>` tags |
| `Auditor-General_Report_2020-21_19` | PDF | 406 | ANAO Major Projects Report (Defence) | Womblex transcript, pending review |
| `foreign-affairs-and-trade-2025-26-portfolio-budget-statements` | DOCX | 18 | DFAT portfolio budget statements | Womblex transcript, pending review |

#### Spreadsheets (`_spreadsheets/`)

| File | Format | Rows | Description | Ground Truth |
|------|--------|------|-------------|--------------|
| `Approved-providers-au-export_20260204` | CSV | 10,859 | Childcare provider registry | Womblex transcript, pending review |
| `mso-statistics-sept-qtr-2025` | XLSX | 7 sheets | Fuel stockholding statistics (DCCEEW) | Womblex transcript, pending review |
| `2021_GCP_all_for_NT_short-header` | ZIP | — | ABS GCP census data for NT (short header) | — |
| `IRSAD Indicators of disadvantage` | CSV | — | SEIFA IRSAD indicator variable descriptions | — |
| `Statistical Area Level 2, Indexes, SEIFA 2021` | XLSX | — | ABS SEIFA 2021 indexes at SA2 level | — |

Some spreadsheets have Dublin Core JSON metadata sidecars (`*.json`) describing
title, date, subject, and access rights.

#### PSV (`_psv/`)

Pipe-separated value datasets from the G-NAF geocoded national address file
(Feb 2026, GDA2020). Trimmed to ACT only, with each table truncated to
header + 20 rows for repo size. The full dataset covers all states.

Contents:
- `Authority Code/` — 16 lookup tables (address types, geocode reliability, etc.)
- `Standard/ACT_*.psv` — 19 ACT address tables (detail, geocode, locality, mesh block, etc.)
- `Contents.txt` — G-NAF release manifest
- `Extras/` — SQL table creation and view scripts

#### Shapefiles (`_SHP/`)

ESRI Shapefile components from the National Native Title Determinations
register (NNTT). Truncated to 20 polygon records for repo size.

| File | Description |
|------|-------------|
| `NTD_Register_Nat.shp` | Geometry (polygon boundaries) |
| `NTD_Register_Nat.dbf` | Attribute table |
| `NTD_Register_Nat.shx` | Spatial index |
| `NTD_Register_Nat.prj` | Coordinate reference system |
| `NTD_Register_Nat.cpg` | Character encoding |

#### Extracted (`_extracted/`)

Serialised `ExtractionResult` objects from running the real extraction pipeline
against each source document. Each JSON file contains the full page-level text,
text blocks with positions, tables, metadata, and computed `full_text`.

| File | Source | Results | Pages | Chars |
|------|--------|---------|-------|-------|
| `throsby-redacted_extracted.json` | Redacted PDF | 1 | 3 | 4,898 |
| `auditor-general_extracted.json` | ANAO PDF (first 30 pages) | 1 | 30 | 76,514 |
| `dfat-budget-statements_extracted.json` | DFAT DOCX | 1 | 1 | 99,706 |
| `mso-statistics_extracted.json` | Fuel MSO XLSX | 34 | 34 | 34,786 |
| `approved-providers_extracted.json` | Provider CSV | 10,859 | 10,859 | 2,422,497 |

#### Chunks (`_chunks/`)

Pre-chunked versions of the extracted text, generated with a word-count
tokeniser at chunk_size=200 (matching the test suite baseline). Each JSON
file contains metadata, chunking config, stats, and per-extraction-result
chunk arrays with character offsets back into the extracted `full_text`.

| File | Source | Chunks | Avg Chars |
|------|--------|--------|-----------|
| `throsby-redacted_chunks.json` | Redacted PDF | 5 | 978 |
| `auditor-general_chunks.json` | ANAO PDF (30 pages) | 64 | 1,194 |
| `dfat-budget-statements_chunks.json` | DFAT DOCX | 80 | 1,246 |
| `mso-statistics_chunks.json` | Fuel MSO XLSX | 36 | 966 |
| `approved-providers_chunks.json` | Provider CSV | 10,867 | 222 |

Regenerate both with: `uv run python scripts/generate_chunk_fixtures.py`

### Isaacus (`isaacus/`)

Real API responses from the Isaacus legal NLP platform, generated from the
chunk fixtures above. See [`fixtures/isaacus/README.md`](fixtures/isaacus/README.md)
for full schema documentation.

⚠️ These fixtures are expensive to recreate (API costs + hours of calls).
Only regenerate when chunk fixtures change significantly.

#### Enrichment (`isaacus/enrichment/`)

Isaacus kanon-2-enricher responses containing entities, segments, terms,
dates, and document structure per chunk (ILGS schema).

| File | Source | Description |
|------|--------|-------------|
| `throsby-redacted_enrichment.json` | Redacted PDF | 3-page regulatory decision |
| `auditor-general_enrichment.json` | ANAO PDF | 30-page defence projects report |
| `dfat-budget-statements_enrichment.json` | DFAT DOCX | Portfolio budget statements |
| `mso-statistics_enrichment.json` | Fuel MSO XLSX | Fuel stockholding statistics |

Regenerate with: `uv run python scripts/generate_enrichment_fixtures.py`

#### Embeddings (`isaacus/embeddings/`)

1792-dimensional L2-normalised vectors from kanon-2-embedder, one per chunk.

| File | Source |
|------|--------|
| `throsby-redacted_embeddings.json` | Redacted PDF |
| `auditor-general_embeddings.json` | ANAO PDF |
| `dfat-budget-statements_embeddings.json` | DFAT DOCX |
| `mso-statistics_embeddings.json` | Fuel MSO XLSX |

#### Graph (`isaacus/graph/`)

JSONL property graphs built from enrichment + chunk data. Each line is a
node or edge JSON object.

| File | Description |
|------|-------------|
| `graph.jsonl` | Combined graph (auditor-general, dfat, mso-statistics) |
| `pii_graph.jsonl` | PII-focused variant |

Regenerate with: `uv run python scripts/generate_graph_fixtures.py`

### MiniLM Embeddings (`MiniLM_embeddings/`)

384-dimensional embeddings from all-MiniLM-L6-v2 (local model, no API key
required). Same chunk sources as the Isaacus embeddings above.

| File | Source |
|------|--------|
| `throsby-redacted_embeddings.json` | Redacted PDF |
| `auditor-general_embeddings.json` | ANAO PDF |
| `dfat-budget-statements_embeddings.json` | DFAT DOCX |
| `mso-statistics_embeddings.json` | Fuel MSO XLSX |

Regenerate with: `uv run python scripts/generate_minilm_embeddings.py`

## Fixture selection

Samples were selected for variety across difficulty and content type:

| Dataset | Sample | Description |
|---------|--------|-------------|
| FUNSD | `85540866` | Sparse form (25 words) |
| FUNSD | `82200067_0069` | Median density (181 words) |
| FUNSD | `87594142_87594144` | Dense form (434 words) |
| FUNSD | `87528321` | Form-heavy (60 question fields) |
| FUNSD | `87528380` | Header-heavy (11 headers, 108 fields) |
| IAM-line | `short_1602` | Single word, narrow image |
| IAM-line | `median_15` | Typical 9-word line |
| IAM-line | `long_4` | Long 22-word sentence |
| IAM-line | `wide_1739` | Widest image (4369px) |
| IAM-line | `narrow_1163` | Narrowest image (187px) |
| DocLayNet | `diverse_layout_49` | 7 layout label types |
| DocLayNet | `table_0` | Table-heavy document |
| DocLayNet | `formula_29` | Contains formulas |
| DocLayNet | `sparse_text_344` | Minimal text (13 words) |
| DocLayNet | `dense_text_548` | Dense text (413 words) |
| womblex-collection | `Throsby...Redacted` | Redacted native PDF, 3 pages (730 words) |
| womblex-collection | `Auditor-General_Report_2020-21_19` | Native PDF, 406 pages (193k words) |
| womblex-collection | `foreign-affairs-and-trade-2025-26-portfolio-budget-statements` | DOCX portfolio budget statements |
| womblex-collection | `Approved-providers-au-export` | CSV, 10,859 provider records |
| womblex-collection | `mso-statistics-sept-qtr-2025` | XLSX, 7 sheets of fuel statistics |
| womblex-collection | `2021_GCP_all_for_NT_short-header` | ZIP, ABS GCP census data for NT |
| womblex-collection | `IRSAD Indicators of disadvantage` | CSV, SEIFA IRSAD indicator descriptions |
| womblex-collection | `Statistical Area Level 2, Indexes, SEIFA 2021` | XLSX, SEIFA 2021 indexes at SA2 level |
| womblex-collection | `g-naf_feb26 (ACT only)` | PSV, G-NAF geocoded address file (19 tables, ~135 MB) |
| womblex-collection | `NTD_Register_Nat` | SHP, NNTT native title determinations (20 polygons) |
