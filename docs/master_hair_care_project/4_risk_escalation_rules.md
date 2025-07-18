### 4  RISK & ESCALATION RULES (Lambda → Slack)

| Trigger query                | Auto action                                                              |
| ---------------------------- | ------------------------------------------------------------------------ |
| `shed_7dma > baseline + 2σ`  | Slack “shed anomaly”; prompt `photo` event                               |
| `redness_index > 2σ`         | Patch `Event.payload['contact_min'] = 2` on next `wash`                  |
| `dryness_score > 3` for 14 d | Auto-insert Saturday protein mask event                                  |
| KPI lag ≥ 2 (quarterly cron) | Flag choice: enable LLLT cap OR raise `ghk_serum` concentration to 0.2 % |
| `bottle_weight` dev > 10 %   | Open GitHub Issue `#adherence`                                           |

---
