# MASTER HAIR-CARE PROJECT — *Scalp* Domain Module

**Version r6 – 21 Jun 2025**
*This is the “scalp” slice of the unified **Health-Lab monorepo** that now hosts beard, skin, and scalp workflows behind a single FastAPI service, TimescaleDB instance, and React/Expo front-end. All identifiers, blackout windows, event names, and infrastructure paths match the merged code-base previously specified.*

---

### 1  PRODUCT FORMULARY (unchanged chemistry, new `product_kind` keys)

| `product_kind` (DB) | Commercial label                                                   | Key actives                                                                                             | Mechanism(s)                                                            | Strength / note |
| ------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | --------------- |
| `shampoo`           | **BLEND Shampoo 1.5 % KZ** (120 mL Intelligent + 120 mL Rx 2 % KZ) | KZ 1.5 %, black-cumin, pumpkin-seed, caffeine, peppermint, biotin                                       | Antifungal, mild 5-AR inhibition, vasodilation                          | KZ final 1.5 %  |
| `conditioner`       | **Intelligent Super-Volumizing Conditioner**                       | Panthenol, biotin, hydrolysed silk, sea-kelp, antioxidants, argan, tea-tree                             | Moisture, keratin support, antioxidant                                  | proprietary     |
| `growth_serum`      | **Growth-Serum Foam (night)**                                      | Minoxidil 5 %, dutasteride 1 %, tretinoin 0.01 %, fluocinolone 0.01 %, caffeine 2 %, latanoprost 0.01 % | Vasodilator, type I/II 5-AR block, enhanced penetration, PGF₂α analogue | 0.75 mL dose    |
| `ghk_serum`         | **GHK-Cu Peptide Serum (morning)**                                 | GHK-Cu 0.1 %, HA + niacinamide base                                                                     | Wnt/β-catenin, VEGF, anti-inflammatory                                  | 1 mL dropper    |
| `argon_oil`         | 100 % cosmetic-grade argan                                         | Linoleic & oleic acids, tocopherols                                                                     | Occlusive & antioxidant                                                 | 2-3 drops scalp |

*Lot numbers go in `/product-batches/`.*

---

### 2  CLINICAL EXECUTION GRID (`domain="scalp"`)

| Day     | 07 : 00 — Morning sequence *(PWA form → `Event` rows)*                                | 21 : 00 — Night sequence                                          | Black-out logic (`blackout_guard`)                                          |
| ------- | ------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Sun** | Shampoo 10 mL → Conditioner 8 mL → Argan 2–3 drops → **`ghk_serum` 1 mL**             | **`microneedle` depth 1–1.5 mm; 4 passes** → sterile-saline rinse | **24 h scalp blackout**: No `ghk_serum` or `growth_serum` until Mon 21 : 00 |
| **Mon** | — *(within blackout)*                                                                 | **`growth_serum` 0.75 mL**                                        | 24 h window satisfied                                                       |
| **Tue** | Shampoo 10 mL (≥ 5 min contact) → Conditioner → Argan → `ghk_serum`                   | `growth_serum`                                                    | —                                                                           |
| **Wed** | Rinse only → Conditioner → Argan → `ghk_serum`                                        | `growth_serum`                                                    | —                                                                           |
| **Thu** | Shampoo → Conditioner → Argan → `ghk_serum`                                           | `growth_serum`                                                    | —                                                                           |
| **Fri** | Rinse only → Conditioner → Argan → `ghk_serum`                                        | `growth_serum`                                                    | —                                                                           |
| **Sat** | Shampoo → Conditioner **or** protein/oil mask ≥ 60 min → `ghk_serum` *(skip if mask)* | `growth_serum` *(skip if mask)*                                   | —                                                                           |

*Annual cadence: Shampoo 156×, Conditioner 365×, Growth-Serum 312×, GHK-Cu 312×.*

---

### 3  SUPPLY & COST FORECAST (JSON rows → `ProductBatch`)

| Item                    | Uses wk⁻¹ | mL use⁻¹ | mL yr⁻¹ | Units yr⁻¹  | Est. cost        |
| ----------------------- | --------- | -------- | ------- | ----------- | ---------------- |
| Intelligent Shampoo     | 3         | 5        | 780     | 3 × 295 mL  | $95             |
| Rx 2 % KZ Shampoo       | 3         | 5        | 780     | 4 × 200 mL  | $160            |
| Intelligent Conditioner | 7         | 8        | 2920    | 10 × 295 mL | $320            |
| Growth-Serum Foam       | 6         | 0.75     | 234     | 4 × 60 g    | $900            |
| GHK-Cu Serum            | 6         | 1        | 312     | 4 × 100 mL  | $220            |
| Argan Oil               | 7         | 0.5      | 180     | 3 droppers  | $45             |
| **Total ±10 %**         | —         | —        | —       | —           | **≈ $1 740 yr** |

---

### 4  RISK & ESCALATION RULES (Lambda → Slack)

| Trigger query                | Auto action                                                              |
| ---------------------------- | ------------------------------------------------------------------------ |
| `shed_7dma > baseline + 2σ`  | Slack “shed anomaly”; prompt `photo` event                               |
| `redness_index > 2σ`         | Patch `Event.payload['contact_min'] = 2` on next `wash`                  |
| `dryness_score > 3` for 14 d | Auto-insert Saturday protein mask event                                  |
| KPI lag ≥ 2 (quarterly cron) | Flag choice: enable LLLT cap OR raise `ghk_serum` concentration to 0.2 % |
| `bottle_weight` dev > 10 %   | Open GitHub Issue `#adherence`                                           |

---

### 5  MEASUREMENT & STATS (Timescale hypertables → MLflow)

| Signal → table  | Instrument                 | Precision target |
| --------------- | -------------------------- | ---------------- |
| `photo` (scalp) | Tripod + 60 mm lens + grid | pixel CV < 2 %   |
| `hmi`           | HairCheck 200, 3 passes    | CV ≤ 3 %         |
| `shed_combtst`  | 60 s comb test             | SD ≈ 20 hairs    |
| `mn_depth`      | Wax block, BLE pen         | ±0.05 mm         |
| `dose_weight`   | BLE 0.01 g scale           | ±0.02 g          |

**Power** ≥ 0.80 to detect **+5 % HMI** at Month 9 under AR(1) σ 0.03.

---

### 6  BACKEND & INFRA (MONOREPO PATHS)

```
healthlab-monorepo/
├── backend/app/
│   ├── domains/
│   │   ├── scalp/           # ← THIS MODULE
│   │   │   ├── router.py    # /scalp/events /scalp/photos
│   │   │   └── rules.py     # contact-time downgrade, 24 h blackout
│   └── ...
├── automations/
│   ├── blackout_guard.py    # domain-aware (24 h for scalp)
│   ├── shed_anomaly.py      # scalp + beard + skin
│   └── blend_contact_timer.py
└── frontend/tabs/Scalp.tsx   # React/Expo forms & camera
```

* `domain="scalp"` is hard-coded in the Scalp tab; every event the PWA emits therefore lands in the correct hypertable partition.*

---

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

### 8  QUICK-START (Affects only scalp domain)

1. Patch-test **GHK-Cu 0.1 %** inner arm 24 h.
2. Blend shampoo; label bottle “1.5 % KZ – Sun Tue Thu Sat”.
3. Load first lot numbers (shampoo, conditioner, serums, argan) → `/product-batches/`.
4. Stock **8** sterile microneedle cartridges (scalp); create inventory threshold 2.
5. Pair BLE scale & trichometer with PWA; verify MQTT events.
6. Capture baseline scalp photo set; verify ImageJ hair-count pipeline.
7. Enable Slack alerts & Grafana dashboard **#alerts-scalp**.

---

## **Outcome**

*Scalp-hair care* is now a first-class **domain** inside the unified Health-Lab stack:

* Shared **auth, storage, CI/CD, observability** with beard & skin modules.
* Domain-specific **blackout rules (24 h)** enforced centrally.
* Single PDF **weekly / quarterly report** with beard, skin, scalp sections.
* Deterministic **burn-rate automation** keeps consumables stocked and costs predictable (≈ $1 740 yr).
* End-to-end data lineage (Timescale → MLflow → Pandoc) lets you publish peer-review-grade single-subject time-series for scalp density, shedding, sebum, and redness.

Maintain ≥ 90 % adherence, keep cartridges single-use, and your integrated dataset will decisively test whether this multi-modal scalp protocol yields the planned **≥ +5 % Hair-Mass Index at 9 months** while dovetailing with beard and skin regimens.

*(Blueprint & engineering spec — not personalized medical advice; confirm drug formulations with your dermatologist.)*
