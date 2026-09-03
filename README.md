# DARPA TC E5 ground truth (CADETS, THEIA, TRACE)

Node labels for DARPA Transparent Computing Engagement 5, taken from ubc-provenance/PIDSMaker.
Pulled 2026-08-28.

| Source | Repo | Commit | Date |
|---|---|---|---|
| PIDSMaker | https://github.com/ubc-provenance/PIDSMaker | `2289cd9` | 2026-07-30 |
| Orthrus | https://github.com/ubc-provenance/orthrus | `e7f25df` | 2026-05-11 |
| ground-truth (Orthrus submodule) | https://github.com/ubc-provenance/ground-truth | `59d2d1a` | 2025-05-19 |

- ground-truth repo moved from `ProvenanceAnalytics/ground-truth` to `ubc-provenance/ground-truth`. Old URL redirects.
- Orthrus pins the submodule at `012f321`, older than `main`. `012f321..main` only adds the OpTC files (h051, h201, h501). E5 files identical.

## Sources

Orthrus: labels are in the submodule at `Ground_Truth/darpa`. Empty after a plain clone, needs
`--recurse-submodules`. No TRACE labels.

PIDSMaker: labels committed in-tree under `Ground_Truth/`, three versions: `orthrus/`, `reapr/`,
`threatrace/`.

PIDSMaker `Ground_Truth/orthrus/` = the 15 files of the standalone repo (cmp identical) + 36 more,
including E5-TRACE (only published copy). All four files here are from PIDSMaker.

`reapr/` and `threatrace/`: E3 only, no E5 file anywhere in their git history. `orthrus/` is the
only E5 ground truth.

## Labels

| Dataset | Attack | Date | Rows | Distinct attribute strings | subject / file / netflow | Attack window (US/Eastern) |
|---|---|---|---:|---:|---|---|
| CADETS E5 | Nginx_Drakon_APT | 05-16 | 19 | 10 | 9 / 5 / 5 | 09:31:00 to 10:12:00 |
| CADETS E5 | Nginx_Drakon_APT_17 | 05-17 | 107 | 14 | 96 / 6 / 5 | 10:15:00 to 15:33:00 |
| THEIA E5 | THEIA_1_Firefox_Drakon_APT_BinFmt_Elevate_Inject | 05-15 | 70 | 14 | 64 / 2 / 4 | 14:47:00 to 15:08:00 |
| TRACE E5 | Trace_Firefox_Drakon | 05-14 | 71 | 35 | 39 / 28 / 4 | 10:17:00 to 11:45:00 |

- One row per node UUID. No duplicate UUIDs within a file.
- Repeated program names are separate subject nodes (new subject node per fork/exec in CDM).
  CADETS 05-17: 56 of 96 subject rows are `main`. THEIA: 51 of 64 are sshd.
- Distinct attribute strings = number of distinct column-2 values = `distinct_entities` in
  `manifest.csv`.
- The two CADETS files share two UUIDs (`908C1871-9556-355C-9695-1D084C35A52A`,
  `EDAD853A-31D9-ED54-9931-889CC4ED15B6`, both `{'file': 'None'}`): 126 rows, 124 unique nodes.
  Published TP+FN for CADETS E5 is 123.
- Excluded upstream as failed attacks: THEIA E5 `Firefox_Drakon_APT` (05-14), TRACE E5
  `Azazel APT` (05-17). Per-attack notes: `sources/pidsmaker-Ground_Truth/orthrus/readme.md`.

## Splits (PIDSMaker defaults)

| Dataset | Train | Val | Test |
|---|---|---|---|
| CADETS E5 | 05-08, 05-09, 05-11 | 05-12 | 05-16, 05-17 |
| THEIA E5 | 05-08, 05-09, 05-10 | 05-11 | 05-14, 05-15 |
| TRACE E5 | 05-08, 05-09 | 05-11 | 05-14, 05-15 |

## Format

No header. Three columns:

1. `UUID`, from the raw CDM records
2. `attributes`, Python dict literal, one key = node type:
   - `{'subject': '<path> <cmdline>'}` process
   - `{'file': '<path>'}` file
   - `{'netflow': '<src_ip>:<port>-><dst_ip>:<port>'}` network flow
3. `index_id`, node id in the Orthrus/PIDSMaker Postgres database

```
CED86807-77E2-11E9-A28B-D4AE52C1DBD3,{'netflow': '128.55.12.51:80->128.55.12.167:58356'},111583
00F33638-7043-11E9-B41B-D4AE52C1DBD3,{'subject': 'None nginx'},238149
```

- Join on UUID. `index_id` is a row number in `file_node_table` / `netflow_node_table` /
  `subject_node_table` of a database built by `create_database`, only valid in that database.
  Both codebases map UUID through `uuid2nids` and ignore column 3 (`get_ground_truth` in
  `orthrus/src/labelling.py` and `pidsmaker/utils/labelling.py`).
- Column 2 has no commas or quotes. `csv.reader` returns 3 fields per row.
- No label expansion at evaluation. Positive class = these nodes
  (`node_evaluation.py` L115, L227). Recall denominator = `sum(len(nodes_set))`
  (`evaluation_utils.py` L1409).
- Edge labels: `malicious_edge_selection: src_node` by default (`config/default.yml` L164),
  edge is malicious if its source node is in the GT set. Needs the Postgres database, not included.
- Attack windows are US/Eastern, converted to ns with `datetime_to_ns_time_US()`.

## Layout

```
E5-ground-truth/           CADETS, THEIA, TRACE E5 labels
  manifest.csv               counts and attack windows, one row per attack
  CADETS/  node_Nginx_Drakon_APT.csv, node_Nginx_Drakon_APT_17.csv
  THEIA/   node_THEIA_1_Firefox_Drakon_APT_BinFmt_Elevate_Inject.csv
  TRACE/   node_Trace_Firefox_Drakon.csv

sources/
  pidsmaker-Ground_Truth/    PIDSMaker Ground_Truth/: orthrus/ (E3, E5, OpTC, ATLASv2, Carbanak),
                             reapr/ (E3), threatrace/ (E3). orthrus/readme.md has per-attack notes.
                             orthrus/ also has E5-CLEARSCOPE and E5-FIVEDIRECTIONS (4 files each).
  orthrus-ground-truth-repo/ standalone ubc-provenance/ground-truth

docs/
  TA51_Final_report_E5.pdf              DARPA E5 ground truth report
  TC_Ground_Truth_Report_E3_Update.pdf  DARPA E3 ground truth report
                                        (both from PIDSMaker Ground_Truth/)
```

## Download

```bash
# PIDSMaker, labels in-tree
git clone https://github.com/ubc-provenance/PIDSMaker.git
ls PIDSMaker/Ground_Truth/orthrus/E5-{CADETS,THEIA,TRACE}

# standalone ground-truth repo (Orthrus submodule, no TRACE)
git clone https://github.com/ubc-provenance/ground-truth.git

# Orthrus with submodule
git clone --recurse-submodules https://github.com/ubc-provenance/orthrus.git
```

Database dumps (large, not included): `PIDSMaker/download_datasets.sh <datasets> <google-oauth-token>`.
`trace_e5` dump is in two parts, the script concatenates them. Postgres dumps of the parsed graph, not
raw DARPA data.
