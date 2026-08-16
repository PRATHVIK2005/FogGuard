# FogGuard
A Multi-Layer Rogue Device Detection Framework for IoT Networks
23IC6PWPP1 — Major Project Phase 2, AY 2025-26, BMSCE

## Status
Phase 0 complete — repo structure + schema locked. Building Phase 1 (device fleet).

## Architecture
Edge (Docker device fleet, Modbus TCP) → Fog (capture, ARP/OUI, MBAE feature extraction, DBSCAN inventory) → Cloud (Isolation Forest + LSTM, verdict, dashboard)

See `dataset/schema.md` for the full data contract between components.