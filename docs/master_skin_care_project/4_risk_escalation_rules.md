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
