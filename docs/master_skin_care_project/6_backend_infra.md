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
