# Contextual UI Redesign — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Make the protocol page time-aware so the user sees what to do *right now* when they open it.

**Architecture:** Add a "Now Card" component at the top of each tab that reads `new Date()` and renders context-specific content. Convert existing content blocks into collapsible sections (closed by default, auto-opened for current time slot). All in a single `index.html` — no build step, no dependencies.

**Tech Stack:** Vanilla HTML/CSS/JS, CSS transitions for collapse animation.

---

### Task 1: Add CSS for Now Card and collapsible sections

**Files:**
- Modify: `/tmp/protocol-tmp/index.html` — `<style>` block (lines 11–163)

**Step 1: Add Now Card CSS**

Add after line 163 (before `</style>`):

```css
/* ══════════ NOW CARD ══════════ */
.now-card {
  margin: 20px 28px 0;
  padding: 20px;
  border-radius: 16px;
  background: rgba(200,251,69,0.06);
  border: 1px solid rgba(200,251,69,0.15);
}
.now-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 9px;
  font-weight: 500;
  letter-spacing: 4px;
  text-transform: uppercase;
  color: var(--lime);
  margin-bottom: 10px;
}
.now-title {
  font-size: 20px;
  font-weight: 700;
  letter-spacing: -0.3px;
  margin-bottom: 8px;
}
.now-items {
  display: grid;
  gap: 4px;
}
.now-item {
  font-size: 13.5px;
  color: var(--soft);
  line-height: 1.5;
  display: flex;
  align-items: baseline;
  gap: 8px;
}
.now-item::before {
  content: '\00b7';
  color: var(--muted);
  font-weight: 800;
  font-size: 16px;
  flex-shrink: 0;
}
.now-item-supp {
  font-size: 13px;
  font-weight: 600;
  color: var(--lime);
}
.now-item-warn {
  color: var(--rose);
  font-weight: 600;
}
.now-next {
  margin-top: 10px;
  font-size: 12px;
  color: var(--muted);
  font-style: italic;
}
```

**Step 2: Add collapsible section CSS**

Add right after the Now Card CSS:

```css
/* ══════════ COLLAPSIBLE ══════════ */
.collapsible-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
  padding: 20px 28px;
  border-bottom: 1px solid var(--line);
}
.collapsible-head .block-head {
  margin-bottom: 0;
  pointer-events: none;
}
.collapsible-chevron {
  color: var(--muted);
  font-size: 14px;
  transition: transform 0.25s ease;
  flex-shrink: 0;
  margin-left: 8px;
}
.collapsible-head.open .collapsible-chevron {
  transform: rotate(90deg);
}
.collapsible-body {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease;
}
.collapsible-body.open {
  max-height: 800px;
}
.collapsible-body-inner {
  padding: 0 28px 20px;
}
```

**Step 3: Verify in browser**

Open `index.html` in browser. No visual changes yet (CSS only, not applied). No errors in console.

**Step 4: Commit**

```bash
git add index.html
git commit -m "Add CSS for now-card and collapsible sections"
```

---

### Task 2: Build the Now Card for Nutrition tab

**Files:**
- Modify: `/tmp/protocol-tmp/index.html` — HTML (after tabs div) + JS (script block)

**Step 1: Add Now Card HTML placeholder**

In the `tab-nutrition` div, right after the banners (after line 195), add:

```html
<div id="now-nutrition" class="now-card">
  <div class="now-label">Maintenant</div>
  <div class="now-title" id="now-nutri-title"></div>
  <div class="now-items" id="now-nutri-items"></div>
  <div class="now-next" id="now-nutri-next"></div>
</div>
```

**Step 2: Add JS to populate the Now Card**

In the `<script>` block, add a new function `renderNowNutrition()` that:

1. Gets current hour via `new Date().getHours()`
2. Determines which time slot we're in
3. Populates `#now-nutri-title`, `#now-nutri-items`, `#now-nutri-next`

Time slot data (array of objects):

```js
var nutriSlots = [
  { start: 4, end: 4, id: 'preworkout', title: 'Pré-workout',
    items: ['Café noir', '1 banane moyenne', '1 cuillère d\'huile d\'olive'],
    supps: ['Ashwagandha KSM-66 600mg', 'Oméga 3 ESN x3', 'Vitamine D3 (4 gouttes)', 'Saw Palmetto 320mg', 'Vitamine C 300mg'],
    next: 'Prochain : entraînement à 5h00' },
  { start: 5, end: 6, id: 'training', title: 'Entraînement PPL',
    items: ['1 pastille d\'électrolytes', '1 scoop EAA'],
    supps: [],
    next: 'Prochain : shake post-workout' },
  { start: 6, end: 11, id: 'postworkout', title: 'Shake post-workout', startLabel: '~6h30',
    items: ['1 scoop whey ON Gold Standard (30g)', '50g flocons d\'avoine en poudre', 'Créatine monohydrate (5g)', 'Collagène Collamin (10g)'],
    supps: [],
    next: 'Prochain : déjeuner vers 12h00' },
  { start: 12, end: 14, id: 'lunch', title: 'Déjeuner',
    items: ['200g blanc de poulet (mariné)', 'Pas de féculent', '200g chou-fleur'],
    supps: [],
    next: 'Prochain : snack' },
  { start: 15, end: 18, id: 'snack', title: 'Snack',
    items: ['2 oeufs'],
    supps: [],
    next: 'Prochain : dîner vers 19h00' },
  { start: 19, end: 21, id: 'dinner', title: 'Dîner',
    items: ['200g entrecôte (marinée)', '60g riz / pâtes / ebly (cru)', '150g légumes variés'],
    supps: ['Zinc 14mg', 'Magnésium Glycinate 400mg'],
    next: 'Prochain : sommeil à 22h00' },
  { start: 22, end: 3, id: 'sleep', title: 'Sommeil · 7-8h min',
    items: [],
    supps: [],
    next: '' }
];
```

Logic:
- Ashwagandha: only add to preworkout supps if ashwa cycle is ON (reuse existing `runAshwa` result — store in a variable `ashwaIsOn`)
- Boron: only add to dinner supps if not weekend (reuse existing `isWE` variable)
- For the 22–3 slot, check `hour >= 22 || hour <= 3`
- For the 6–11 slot, start at `hour >= 6 && hour < 7` to cover the ~6h30 label, then extend to 11

**Step 3: Verify in browser**

Open page at different simulated hours (temporarily hardcode `var hour = 12;` to test). The Now Card should show "Déjeuner" with items.

**Step 4: Commit**

```bash
git add index.html
git commit -m "Add Now Card for nutrition tab with time-based content"
```

---

### Task 3: Build the Now Card for Hair tab

**Files:**
- Modify: `/tmp/protocol-tmp/index.html` — HTML + JS

**Step 1: Add Now Card HTML placeholder**

In the `tab-cheveux` div, right after the banner, add:

```html
<div id="now-hair" class="now-card" style="border-color: rgba(108,180,255,0.15); background: rgba(108,180,255,0.05);">
  <div class="now-label" style="color: var(--sky);">Maintenant</div>
  <div class="now-title" id="now-hair-title"></div>
  <div class="now-items" id="now-hair-items"></div>
  <div class="now-next" id="now-hair-next"></div>
</div>
```

**Step 2: Add JS to populate Hair Now Card**

Add function `renderNowHair()` using:

```js
var hairSlots = [
  { start: 4, end: 4, id: 'rosemary', title: 'Massage romarin' },
  { start: 5, end: 7, id: 'shower', title: '' },  // dynamic: Nizoral or gentle
  { start: 8, end: 20, id: 'idle', title: 'Rien pour le moment' },
  { start: 21, end: 3, id: 'evening', title: '' }  // dynamic: minox or needling
];
```

Dynamic content:
- **shower**: if `isNizoral` → title "Nizoral 1%", items ["Laisser poser 3-5 min, rincer"]. Else → title "Shampoing doux", items ["Sans SLS — Ducray Anaphase+ ou Bioderma Nodé"]
- **idle**: `now-next` shows "Prochaine étape : minoxidil ce soir" or "Pas de minoxidil ce soir" (Sun/Mon)
- **evening on Sunday**: title "Microneedling", full protocol items. **evening other days**: if `minoxTonight` → title "Minoxidil 5%", items ["Mousse Regaine sur golfes et zones de recul", "1 noisette, masser 1 min", "Sécher 10-15 min, ne pas rincer, dormir avec"]. If Monday → "Pas de minoxidil — 24h post-needling"

**Step 3: Verify in browser**

Test with simulated hours.

**Step 4: Commit**

```bash
git add index.html
git commit -m "Add Now Card for hair tab with time and day awareness"
```

---

### Task 4: Convert Nutrition blocks to collapsible sections

**Files:**
- Modify: `/tmp/protocol-tmp/index.html` — Nutrition tab HTML blocks + JS

**Step 1: Wrap each nutrition block in collapsible structure**

For each block (Pré-workout, Entraînement, Intra, Post-workout, Déjeuner, Snack, Dîner, Sommeil), replace the current structure:

Before:
```html
<div class="block">
  <div class="block-head">
    <div class="block-hour">4h30</div>
    <div class="block-title">Pré-workout</div>
  </div>
  <div class="items">...</div>
  <div class="macros">...</div>
</div>
```

After:
```html
<div class="collapsible-head" data-slot="preworkout">
  <div class="block-head">
    <div class="block-hour">4h30</div>
    <div class="block-title">Pré-workout</div>
  </div>
  <span class="collapsible-chevron">›</span>
</div>
<div class="collapsible-body" data-slot="preworkout">
  <div class="collapsible-body-inner">
    <div class="items" style="padding-left: 0;">...</div>
    <div class="macros" style="padding-left: 0;">...</div>
  </div>
</div>
```

Map each block to a `data-slot` matching the nutriSlots IDs: `preworkout`, `training`, `postworkout`, `lunch`, `snack`, `dinner`, `sleep`.

The `block-simple` blocks (training, sleep) become collapsible heads too but their body is minimal.

**Step 2: Add JS to handle collapsible toggle**

```js
document.querySelectorAll('.collapsible-head').forEach(function(head) {
  head.addEventListener('click', function() {
    var slot = this.getAttribute('data-slot');
    var body = document.querySelector('.collapsible-body[data-slot="' + slot + '"]');
    this.classList.toggle('open');
    body.classList.toggle('open');
  });
});
```

**Step 3: Auto-open the current time slot**

After rendering the Now Card, get the current slot ID and auto-open its collapsible:

```js
var currentHead = document.querySelector('.collapsible-head[data-slot="' + currentSlotId + '"]');
var currentBody = document.querySelector('.collapsible-body[data-slot="' + currentSlotId + '"]');
if (currentHead && currentBody) {
  currentHead.classList.add('open');
  currentBody.classList.add('open');
}
```

**Step 4: Verify in browser**

- All sections start collapsed except the current one
- Tapping a section header opens/closes it smoothly
- Chevron rotates on open

**Step 5: Commit**

```bash
git add index.html
git commit -m "Convert nutrition blocks to collapsible sections with auto-open"
```

---

### Task 5: Convert Hair tab blocks to collapsible sections

**Files:**
- Modify: `/tmp/protocol-tmp/index.html` — Hair tab HTML blocks + JS

**Step 1: Wrap hair blocks in collapsible structure**

Same pattern as Task 4 for: Douche, Massage romarin, Minoxidil, Microneedling.

Use `data-slot` values: `rosemary`, `shower`, `minoxidil`, `needling`.

**Step 2: Auto-open current hair slot**

Same logic — after `renderNowHair()`, open the matching collapsible.

**Step 3: Keep non-collapsible sections as-is**

These sections stay unchanged (not collapsible):
- Weekly table
- Entretien matériel notes
- Suivi notes
- Timeline
- Alertes
- Budget card
- "Règle d'or" note

**Step 4: Verify in browser**

- Hair blocks collapse/expand correctly
- Current slot auto-opens
- Bottom sections (notes, table, etc.) remain always visible

**Step 5: Commit**

```bash
git add index.html
git commit -m "Convert hair blocks to collapsible sections with auto-open"
```

---

### Task 6: Update hair tab with corrected rosemary timing

**Files:**
- Modify: `/tmp/protocol-tmp/index.html` — Hair tab HTML + JS

**Step 1: Reorder hair blocks**

Current order: Douche → Massage romarin → Minoxidil → Microneedling

New order: Massage romarin (4h30, au réveil) → Douche (au fit, après entraînement) → Minoxidil (soir) → Microneedling (dimanche soir)

Update block hours:
- Romarin: `4h30` (au réveil)
- Douche: `~7h00` (au fit)
- Minoxidil: `Soir`
- Microneedling: `Dim. soir`

**Step 2: Update romarin block content**

Remove the "Alternative : faire avant la douche pour rincer après" line since the timing is now confirmed.

**Step 3: Verify and commit**

```bash
git add index.html
git commit -m "Reorder hair blocks: rosemary at wakeup, shower at gym"
```

---

### Task 7: Final cleanup and push

**Files:**
- Modify: `/tmp/protocol-tmp/index.html`

**Step 1: Remove dead JS comments**

Clean up the leftover comments in the script block (the long minoxidil timing comments around lines 710-715).

**Step 2: Test all scenarios**

Manually verify by temporarily changing hour/day:
- Nutrition: test hours 4, 5, 7, 12, 15, 19, 23
- Hair: test hours 4, 6, 14, 22 + test Sunday + Monday
- Collapsible toggle works on all blocks
- Tab switching works
- Ashwagandha cycle still works
- Boron weekend logic still works

**Step 3: Commit and push**

```bash
git add index.html
git commit -m "Clean up comments and finalize contextual UI"
git push origin main
```
