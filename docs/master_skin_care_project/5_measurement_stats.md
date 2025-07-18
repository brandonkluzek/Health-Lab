### 5 MEASUREMENT PIPELINE

| Event/table                 | Instrument                    | Target precision  |
| --------------------------- | ----------------------------- | ----------------- |
| `photo` (`location="face"`) | 85 mm lens + X-Rite card      | pixel CV < 2 %    |
| `tewl`                      | Delfin VapoMeter (triplicate) | SD < 1 g m⁻² h⁻¹  |
| `sebum`                     | Sebumeter (T-zone)            | ±5 µg cm⁻²        |
| `erythema_idx`              | Polarised photo → CIELab      | detect ΔE*ab ≥ 1 |
| `lesion_cnt`                | OpenCV classifier             | Recall ≥ 0.9      |
| `dose_weight`               | BLE 0.01 g scale on TAAN tube | ±0.02 g          |

Timescale continuous aggregates drive Grafana panels; MLFlow logs AR(1) forecast residuals.
Power ≥ 0.80 to detect **-20 % lesion count** at Month 6 (σ 0.15 weekly).

---
