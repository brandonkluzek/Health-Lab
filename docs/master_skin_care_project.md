# MASTER SKIN-CARE PROJECT — *Facial-Skin* Domain Module

**Version r2 – 21 Jun 2025**
*This specification converts the stand-alone Skin-Care v r1 protocol into a fully namespaced **`domain="skin"`** module inside the unified **Health-Lab monorepo** (which already hosts the “beard” and “scalp” domains).  All tables, event names, blackout windows, Lambda guards, React screens and Grafana panels align 1-for-1 with the merged architecture.*

---

### 1 PRODUCT FORMULARY → `ProductBatch.kind`

| `kind` key    | Commercial label             | Actives                                        | Mechanism(s)                         | Strength / pH |
| ------------- | ---------------------------- | ---------------------------------------------- | ------------------------------------ | ------------- |
| `cleanser`    | **Cami-Team Daily Cleanser** | Coco-betaine, glycerin                         | Non-stripping surfactant + humectant | pH ≈ 5.5      |
| `vitc_serum`  | **Vanicream Vitamin C**      | L-ascorbic acid 10 %, ceramides                | Antioxidant, pro-collagen, barrier   | pH ≈ 3.2      |
| `moisturizer` | **Vanicream Daily Facial**   | HA, ceramides, squalane                        | Humectant + occlusive                | —             |
| `taan`        | **TAAN compounded cream**    | Tretinoin 0.03 %, azelaic 4 %, niacinamide 4 % | Retinoid renewal + anti-inflamm.     | 0.03/4/4 %    |
| `spf`         | **Broad-Spectrum SPF 50+**   | ZnO 12 %, Tinosorb M                           | UV-A/B scatter + absorb              | —             |
| `bcarotene`   | β-Carotene caps              | 7 500 µg RAE                                   | Pro-vit A antioxidant                | 1 cap day⁻¹   |

All batches recorded via `/product-batches/` API.

---

### 2 CLINICAL EXECUTION GRID (`domain="skin"`)

| Day     | 07 : 00 – Morning sequence (`Event.type`)                       | 21 : 00 – Night sequence                                                                                      | Black-out / guard rules                                                         |
| ------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Sun** | `wash` → `vitc_serum` (0.6 mL) → `moisturizer` (0.8 mL) → `spf` | *Scalp microneedle* triggers **`taan_blackout` 24 h** → *skip TAAN*; apply `moisturizer` after gentle cleanse | `barrier_guard` enforces 24 h TAAN hold when a scalp-`microneedle` event exists |
| **Mon** | Same AM routine                                                 | `taan` 0.5 mL                                                                                                 | `barrier_guard` blocks if TEWL > +2 σ                                           |
| **Tue** | Same AM                                                         | `moisturizer` only                                                                                            | —                                                                               |
| **Wed** | Same AM                                                         | `taan` 0.5 mL                                                                                                 | —                                                                               |
| **Thu** | Same AM                                                         | `moisturizer` only                                                                                            | —                                                                               |
| **Fri** | Same AM                                                         | `taan` 0.5 mL                                                                                                 | —                                                                               |
| **Sat** | Same AM                                                         | `mask` (panthenol 5 %, 20 min) → `moisturizer`                                                                | —                                                                               |

*Annual cadence: Cleanser 730, Vit-C 365, Moisturizer 730, TAAN 156, SPF 365.*

---

### 3 SUPPLY & COST (JSON rows → dashboard)

*(identical numbers, now stored with `domain="skin"` tag)*

| Item              | Uses wk⁻¹ | mL use⁻¹ | mL yr⁻¹  | Units yr⁻¹ | Est. cost      |
| ----------------- | --------- | -------- | ------- | ---------- | -------------- |
| Cleanser 237 mL   | 14        | 2        | 730      | 4          | $60           |
| Vitamin‑C 35 mL   | 7         | 0.6      | 219      | 7          | $175          |
| Moisturizer 89 mL | 14        | 0.8      | 409      | 5          | $60           |
| TAAN 30 g         | 3         | 0.5      | 78       | 3          | $180          |
| SPF 90 mL         | 7         | 1        | 365      | 5          | $100          |
| β-Carotene caps   | 7         | —        | 365 caps | 4 bottles  | $45           |
| **Total ±10 %**   | —         | —        | —       | —          | **≈ $620 yr** |

---

### 4 AUTOMATED RISK RULES (Lambda `barrier_guard.py`)

| Query                              | Action                                                           |
| ---------------------------------- | ---------------------------------------------------------------- |
| **TEWL > +2 σ for 7 d**            | Reduce TAAN frequency to 2 × wk (`taan_schedule=2`)              |
| **Redness ΔE*ab > 3**             | Switch `vitc_serum` to alt-day; inject nightly `panthenol` event |
| **New pustules > 10/wk**           | Insert `benzoyl_wash` events Mon / Wed / Fri AM                  |
| **CIELab b* > 2 σ*                 | Flag `bcarotene_hold` 14 d                                       |
| **SPF adherence < 90 % / quarter** | Open GitHub Issue, push mobile reminder                          |
| **KPI lag ≥ 2** (semester cron)    | Suggest retinal-aldehyde or 10 % azelaic foam                    |

---

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

### 6 MONOREPO PATHS & CODE ADD-ON

```
healthlab-monorepo/
├── backend/app/
│   ├── domains/
│   │   └── skin/
│   │       ├── router.py          # /skin/events  /skin/photos
│   │       ├── schemas.py         # Pydantic models
│   │       └── rules.py           # TAAN blackout, TEWL limits
├── automations/
│   ├── barrier_guard.py           # TEWL & redness logic
│   ├── lesion_anomaly.py          # future
│   └── quarterly_skin_report.py   # Pandoc → PDF
└── frontend/tabs/Skin.tsx         # React/Expo forms + camera
```

*`domain="skin"` hard-coded in tab; all emitted events partition correctly.*

---

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

### 8 QUICK‑START (Applies only to `skin` domain)

1. Capture baseline TEWL + color-calibrated photos (`/skin/photo` events).
2. Patch-test **TAAN** on mandibular line 24 h (`/skin/event type="taan_test"`).
3. Set TAAN label: “QOD PM; hold 24 h after *scalp* microneedling”.
4. Enable SPF reminder push at 10 : 00 local (`/alerts/spf`).
5. Enter lot numbers for all five topical products + β-carotene caps.
6. Pair BLE devices (TEWL, TAAN tube scale) in PWA; confirm MQTT ingestion.
7. Create Grafana user; copy `skin_dash.json`; enable Slack `#alerts-skin`.

---

## Outcome

The **Facial-Skin module** is now a first-class **`domain="skin"`** inside your Health-Lab platform:

* **Clinical layer** – minimalist but evidence-rich routine (cleanse, Vit-C, TAAN, SPF, barrier guard) with blackout synchronised to scalp microneedling.
* **Operational layer** – automated inventory & adherence checks across all domains.
* **Data layer** – unified TimescaleDB hypertables, shared analytics, 6-month power to detect ≥ 20 % acne reduction.
* **Dev-Ops layer** – single IaC state, single CI/CD, TLS 1.3, S3 Object-Lock, cross-region backups.

With ≥ 90 % adherence you will generate a publication-ready, inter-domain dataset revealing how skin-barrier metrics interact with scalp and beard interventions—while keeping operational overhead minimal.

*(Blueprint & engineering spec — not individual medical advice; confirm formulations with your dermatologist.)*
---
