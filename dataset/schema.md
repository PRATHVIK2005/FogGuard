# FogGuard Dataset Schema
Owner: Prathvik (fog/feature_extraction) | Consumer: Pavan (ml/mbnn, ml/lstm)
Status: DRAFT — confirm with Pavan before training starts

## Feature Vector (per 10-second window, per device)

| # | Feature Name | Type | Range/Format | Description |
|---|---|---|---|---|
| 1 | fc_distribution | vector[8] | float[0-1], sums to 1 | Normalized frequency of each Modbus function code (FC01,02,03,04,05,06,15,16) seen in window |
| 2 | register_pattern | vector[16] | float[0-1] | Normalized access frequency across register address buckets (bucketed in 16 ranges) |
| 3 | packet_size_mean | float | bytes | Mean packet size in window |
| 4 | packet_size_std | float | bytes | Std deviation of packet size (captures burstiness) |
| 5 | inter_arrival_mean | float | seconds | Mean time between packets |
| 6 | inter_arrival_std | float | seconds | Std deviation of inter-arrival time |
| 7 | session_freq | int | count | Number of distinct connections/sessions in window |
| 8 | byte_rate | float | bytes/sec | Total bytes transferred / window duration |
| 9 | req_resp_ratio | float | ratio | Requests sent / responses received |
| 10 | connection_duration | float | seconds | Duration of active TCP session in window |

## Metadata (attached to every row, not fed to model directly)
| Field | Type | Description |
|---|---|---|
| timestamp | ISO8601 string | Window start time |
| src_ip | string | Source IP |
| src_mac | string | Source MAC |
| window_id | string | Unique ID for this feature window |

## Label Schema (for MBNN — device identity)
| Label | Device Type |
|---|---|
| 0 | CCTV |
| 1 | Robotic Arm |
| 2 | Conveyor Belt |
| 3 | Cooling System |
| -1 | UNKNOWN (open-set / untrained device — MBNN confidence below threshold) |

## Attack Label Schema (for evaluation / RF baseline, not MBNN input)
| Label | Meaning |
|---|---|
| 0 | Normal |
| 1 | MAC Spoof |
| 2 | Unauthorized Write (FC06/16) |
| 3 | Fake Slave Impersonation |

## File Format
- Stored as: `dataset/processed/features_<date>.csv` (or `.parquet` if volume gets large)
- One row = one (device, window) pair
- Columns: [metadata fields] + [feature 1-10] + [device_label] + [attack_label]

## Open Questions for Pavan (confirm before training)
- [ ] Is vector[8]/vector[16] flattened into separate columns (fc_0, fc_1...) or stored as a single JSON/array column? — **Recommend flattened columns, simpler for sklearn/pandas**
- [ ] Does MBNN want raw features or normalized/scaled? — Recommend I hand off raw, Pavan scales in his pipeline (keeps my extractor simpler and reusable for RF baseline too)
- [ ] Confirm the -1 UNKNOWN label threshold logic sits in Pavan's model output, not my extractor (extractor only produces features, not predictions)