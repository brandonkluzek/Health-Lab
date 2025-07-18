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
