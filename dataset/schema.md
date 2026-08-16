# FogGuard Dataset Schema (v2)
Owner: Prathvik (fog: capture, extraction, Stage A/A.5/D) | Pavan (ml: MBAE, DBSCAN, IForest, LSTM)
Status: CONFIRMED

## 1. Raw Feature Vector (fog/feature_extraction/extractor.py output)
Per 10-second window, per source device.

| # | Feature | Type | Description |
|---|---|---|---|
| 1 | fc_0..fc_7 | float[8] | Normalized Modbus function code distribution |
| 2 | reg_0..reg_15 | float[16] | Normalized register address access pattern |
| 3 | packet_size_mean | float | Mean packet size (bytes) |
| 4 | packet_size_std | float | Std dev of packet size |
| 5 | inter_arrival_mean | float | Mean time between packets (s) |
| 6 | inter_arrival_std | float | Std dev of inter-arrival time |
| 7 | session_freq | int | Distinct connections/sessions in window |
| 8 | byte_rate | float | Bytes/sec |
| 9 | req_resp_ratio | float | Requests / responses |
| 10 | connection_duration | float | Active session duration (s) |

## 2. Ground Truth Labels (evaluation only, never fed into MBAE training)
| Field | Type | Values |
|---|---|---|
| device_label | string | conveyor_belt \| cctv \| robotic_arm \| cooling_system \| rogue_impersonator |
| attack_label | int | 0=Normal, 1=MAC Spoof, 2=Unauthorized Write, 3=Fake Slave Impersonation |
| safety_critical | bool | True only for cooling_system |

## 3. MBAE Output (fog-side inference)
| Field | Type | Description |
|---|---|---|
| encoding | float[24] | Concatenated latent vectors from 3 branches |
| recon_error | float | Sum of per-branch reconstruction MSE |

## 4. DBSCAN Inventory Output
| Field | Type | Description |
|---|---|---|
| cluster_id | int | -1 = noise = potential novel device |

## 5. Cloud Isolation Forest Output
| Field | Type | Description |
|---|---|---|
| iforest_anomaly | bool | True if flagged anomalous |
| iforest_score | float | Raw anomaly score |

## 6. Cloud LSTM Output
| Field | Type | Description |
|---|---|---|
| lstm_recon_error | float | Sequence reconstruction error |
| lstm_flag | bool | True if exceeds threshold |

## 7. Final Verdict
| Verdict | Meaning | Triggers |
|---|---|---|
| Normal | No anomaly signal | No action |
| Suspicious | One of IForest/LSTM flags | Quarantine |
| Critical | Both flag, or Stage A/A.5 + behavioral confirm | Stage D escalation |

## 8. Confirmed Architecture Decisions
- [x] MBAE = genuine autoencoder, unsupervised, trains on normal traffic only
- [x] Isolation Forest runs at CLOUD layer (matches official FR3/FR5)
- [x] Hardware interlock (FC05/FC06) EXCLUDES safety-critical devices — alert only
- [x] ARP/OUI/Auth (Stage A/A.5) run at fog layer, always active, zero cloud dependency
- [x] Attack source = real Kali, containerized on fogguard_net (Option B, practical version)
- [ ] "Yang et al." citation — needs verification before use in report/paper