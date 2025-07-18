### 7  IMPLEMENTATION DELTA vs. baseline Health-Lab plan

| Sprint wk | Delta items (scalp only)                                                      |
| --------- | ----------------------------------------------------------------------------- |
| **0.5**   | Add `scalp` domain enum; Alembic migration                                    |
| 1         | Scaffold router & Pydantic schemas                                            |
| 1.5       | Extend React tab; BLE scale integration reused                                |
| 2         | Add HMI calculation to `imagej_worker.py`; compress JPEG → S3 `scalp/` prefix |
| 2         | Grafana: HMI, redness, shed 7dma panels                                       |
| 2.5       | Blue-green deploy; smoke tests; production swap                               |

No additional AWS resources; only ~8 GB yr⁻¹ extra RDS + S3 storage.

---
