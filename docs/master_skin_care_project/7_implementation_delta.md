### 7 DELTA IMPLEMENTATION PLAN (leveraging previous merge)

| Sprint day | Task                                                                           |
| ---------- | ------------------------------------------------------------------------------ |
| **0–0.5**  | Extend `domain` enum → add `"skin"`, Alembic migration                         |
| 0.5–1      | Scaffold `domains/skin/` router & Pydantic schemas                             |
| 1–1.5      | React tab: wash, vit-C, SPF, TAAN, mask forms; camera reuse                    |
| 1.5–2      | Wire BLE TEWL meter & TAAN tube scale via MQTT topics `skin/tewl`, `skin/dose` |
| 2          | `barrier_guard.py` Lambda; unit tests for TEWL ≥ +2 σ rule                     |
| 2          | Grafana panels: TEWL, redness, lesion count, adherence                         |
| 2.5        | Stage deploy → smoke tests (forms, automations, reports)                       |
| 3          | Production ALB swap; old skincare repo archived                                |

No new AWS infra; only additional S3 prefixes (`skin/`) & Timescale rows.

---
