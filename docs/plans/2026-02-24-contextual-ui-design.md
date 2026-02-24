# Contextual UI Redesign — Protocol Page

## Problem

The page shows all protocol info at once. User opens it when they have a doubt about what to do now or how to do a specific step. Too much scrolling to find the relevant info.

## Solution: "Now Card" + Collapsible Sections

### Now Card

A prominent card at the top of each tab that detects current time + day and shows only what's relevant right now.

#### Nutrition Tab — Time Slots

| Slot | Content |
|------|---------|
| 4h00–4h59 | Pre-workout: banana, olive oil + morning supps (Ashwa if ON, Omega3, VitD, SP, VitC) |
| 5h00–6h29 | Training: electrolytes + EAA intra |
| 6h30–11h59 | Post-workout: whey, oats, creatine, collagen |
| 12h00–14h59 | Lunch: chicken, cauliflower, no starch |
| 15h00–18h59 | Snack: 2 eggs |
| 19h00–21h59 | Dinner: steak, rice, veggies + evening supps (Boron if weekday, Zinc, Magnesium) |
| 22h00–3h59 | Sleep: 7-8h min |

#### Hair Tab — Time Slots

| Slot | Content |
|------|---------|
| 4h00–4h59 | Rosemary massage: 3-5 drops in jojoba, 5 min on recession zones |
| 5h00–7h59 | Shower at gym: Nizoral 1% (Mon/Wed/Fri) or gentle shampoo (other days) |
| 8h00–20h59 | Idle — "Next: minoxidil tonight" or "No minox tonight" (Sun/Mon) |
| 21h00–3h59 | Minoxidil 5% foam OR Sunday: microneedling protocol |

### Collapsible Sections

All existing content blocks become collapsible (closed by default):
- Tap title to toggle open/close
- Chevron indicator (`>` rotates to `v` when open)
- The block matching the current Now Card time slot is **open by default**
- Smooth CSS transition on open/close

### Unchanged

- Header + KPIs
- Tabs (Nutrition / Cheveux)
- Totals macros section (nutrition)
- Cycles section (nutrition)
- Notes, alerts, timeline, budget (hair)
- Design system (colors, typography, dark theme)

## Technical Notes

- Single index.html file, no build step
- Time detection via `new Date().getHours()`
- Day detection already exists (`today.getDay()`)
- Collapsible via CSS `max-height` transition + JS toggle
- Hair tab rosemary timing: morning (before gym), not post-shower
