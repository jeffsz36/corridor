# Norhaven Corridor Map — Claude Code Handoff Brief

**Project:** `norhaven-corridors` (sister project to `Nigeria Asset Intelligence — SWF Edition`)
**Type:** Single-file Leaflet.js interactive map, standalone HTML
**Owner:** Jeff / Revitau / Norhaven
**Status:** New build, ready to execute

---

## 0. What this is, in one paragraph

The Norhaven research produced two things: an asset inventory of NiPost's 774-LGA real-estate footprint, and a commodity-corridor plan mapping 10 commodities (5 crops + 5 minerals) end-to-end from origin to Atlantic seaport. A designed PDF report already exists. **This project turns the report into a tangible, interactive map.** A user picks a commodity, sees the corridor today (with its chokepoints visible), then flips a toggle to see what happens when the specific intervention for that commodity lands. A second global toggle flips the whole map between **pitch view** (clean, aspirational, sells the opportunity) and **ops view** (chokepoints, extortion costs, security overlay, honest routing data). Single HTML file, no build step, no framework.

---

## 1. Interaction model (the core spec)

### 1.1 Two global toggles at the top

```
┌─────────────────────────────────────────────────────┐
│  [ PITCH | OPS ]      [ 2024 STATE | UNLOCKED ]     │
└─────────────────────────────────────────────────────┘
```

- **PITCH / OPS toggle** (global, persistent across all commodities)
  - **Pitch:** subtle chokepoints, prominent intervention markers, hero numbers, clean polylines, muted basemap (CartoDB Positron)
  - **Ops:** all risk data on, security-zone polygons rendered, checkpoint markers with ₦ annotations, rail-rolling-stock caveats visible, standard OSM basemap

- **2024 STATE / UNLOCKED toggle** (per-commodity; flips when a commodity is selected)
  - **2024 State:** current corridor with chokepoints lit up
  - **Unlocked:** intervention(s) fire, chokepoints fade, corridor polyline redraws (sometimes via different waypoints — e.g. Lekki rail replaces Apapa road)

### 1.2 Commodity sidebar (left rail)

```
CROPS
◉ Cocoa            SW belt → Lagos
◯ Sesame           NE/NC → Lagos
◯ Cassava          Nationwide → Lagos
◯ Ginger           Kaduna → Lagos
◯ Hibiscus         NW → Lagos

MINERALS
◯ Gold             Osun + Zamfara → Lagos (air)
◯ Lithium          NC pegmatite belt → Onne
◯ Tin/columbite    Jos Plateau → Lagos
◯ Lead-zinc        Ebonyi → Calabar/Onne
◯ Barite           Nasarawa Azara → Onne
```

- Each chip shows the commodity name and "origin region → destination port"
- Click a chip:
  1. Map pans and fits-bounds to the corridor extent
  2. Corridor polyline animates in from origin to port (use `polyline.setStyle` with staged drawing or CSS animation)
  3. Chokepoints render as pulsing markers along the route
  4. Right-side **insight panel** appears with the commodity's key numbers and its unlock explanation
  5. The per-commodity unlock toggle becomes active

### 1.3 Right-side insight panel

Collapsible panel that shows:
- Commodity name + producing states + annual volume/value
- Current corridor distance, modality breakdown (road/rail/river km), estimated transit days
- Top 3 chokepoints with severity and type
- The unlock card: "When [intervention] deploys, this corridor becomes [benefit]"
- Cites the source page in the PDF report

### 1.4 Per-commodity unlock behavior

When user flips a selected commodity's 2024 → Unlocked toggle:
- Chokepoints relevant to the unlock fade to 20% opacity (not removed — still visible faintly)
- Intervention markers (e.g. "Lekki rail spur", "Cold chain hub — Kachia", "Segilola-style formal buying centre") animate in
- The corridor polyline redraws if the modality changes (e.g. cocoa reroutes through Lekki rail instead of Apapa road)
- Insight panel swaps from "Current state" to "After unlock" copy
- A subtle success animation (pulse the corridor green once) signals the change

### 1.5 Map-level layer controls (bottom-left, always visible)

```
BASE LAYERS (toggle on/off independently)
☑ Rail network (narrow + standard gauge, status-colored)
☑ Major road corridors (A1, A2, A3)
☐ Inland waterways (Niger, Benue, Cross River)
☐ NiPost backbone (774 LGA points — link to main Norhaven map)
☐ Security risk zones (NW banditry, NE insurgency, SE sit-at-home, SS militancy)
```

Security zones toggle should auto-enable when OPS mode is selected.

---

## 2. File structure

Single HTML file, matching the existing Norhaven map pattern:

```
norhaven-corridors.html
```

Inside, follow this structure:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Norhaven Corridor Map</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
  <style>
    /* All styles inline */
  </style>
</head>
<body>
  <div id="app">
    <header>...</header>           <!-- global toggles -->
    <aside id="commodity-list">...</aside>
    <main id="map"></main>
    <aside id="insight-panel">...</aside>
    <div id="layer-controls">...</div>
  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    // All data (commodities, chokepoints, interventions, ports) inline as JS objects
    const DATA = { ... };
    // App logic
  </script>
</body>
</html>
```

Leaflet from CDN, CartoDB Positron and OSM tile layers. No webpack, no build, no npm.

---

## 3. Visual design system

Match the Norhaven aesthetic. The existing SWF Edition map uses mineral-diamond markers colored by sector, amber pill toggles (Taraba TMP2050 pattern), and has a serious/institutional feel.

### 3.1 Color tokens

```css
:root {
  /* Brand */
  --forest: #1a3a2e;
  --forest-deep: #0f2a20;
  --terracotta: #c04a1a;
  --gold: #c89932;
  --sage: #5a7a5a;
  --clay: #a85a3a;
  --cream: #faf8f3;
  --charcoal: #1f1f1f;

  /* Functional */
  --severity-critical: #c13a2a;   /* security, banditry */
  --severity-high:     #d97826;   /* infrastructure, bottlenecks */
  --severity-medium:   #c89932;   /* regulatory, cost */
  --intervention:      #3a8a5c;   /* green = unlock */

  /* Modality line colors */
  --modality-road: #6b6b63;
  --modality-rail: #c04a1a;
  --modality-river: #3a7590;
  --modality-air: #c89932;
  --modality-sea: #1a3a2e;
}
```

### 3.2 Marker conventions

- **Origin/farmgate/mine-head:** circle, 10px, filled with commodity color
- **Aggregation/market:** square, 10px, outlined
- **Processing plant:** diamond (match existing Norhaven convention for minerals), 12px, sector-colored fill
- **Bonded warehouse / dry port:** rounded square, 12px, cream-filled with dark outline
- **Seaport:** anchor icon, 18px, forest-green
- **Chokepoint:** triangle, 12px, severity-colored, pulsing in 2024 state
- **Intervention:** hexagon, 14px, intervention-green, glow effect when activated

### 3.3 Polyline conventions

- Solid line = road
- Dashed line (8,4 pattern) = rail
- Dotted line (2,4 pattern) = river/waterway
- Stroke-dasharray animation on draw-in (SVG `stroke-dashoffset` animation, 1.2s ease-out)
- Stroke weight 3px normally, 5px when that commodity is actively selected
- Opacity 0.85 for active commodity, 0.25 for background context

### 3.4 Typography

- Sans (UI): `-apple-system, "Inter", "Helvetica Neue", sans-serif`
- Serif (numbers/headlines in panel): `Georgia, "Iowan Old Style", serif`
- Monospace (corridor stats): `"SF Mono", "Menlo", monospace`

### 3.5 Pitch vs Ops mode visual diffs

| Element | Pitch | Ops |
|---|---|---|
| Basemap | CartoDB Positron (muted) | OSM Standard |
| Chokepoints | 6px, 30% opacity, no labels | 12px, 100% opacity, ₦ cost labels visible |
| Security zones | Hidden unless explicitly toggled | Shown by default, semi-transparent fill |
| Intervention markers | Large, animated, gold-glow | Normal size, no glow |
| Insight panel tone | Opportunity-forward ("$2B unlock") | Data-forward ("3.5 days road transit + ₦100k checkpoint") |
| Corridor stroke | Confident, single weight | Segment-colored by risk (green clean / amber risky / red dangerous) |

---

## 4. Data specification

All data lives inline as JS objects. Structure below. Where lat/lng is listed as `REQUIRES_GEOCODING`, Claude Code should make best-effort geocoding using the place name + state, and flag uncertainty in a code comment. **No fabricated coordinates** — if a place can't be located, use the state capital as a labeled approximation.

### 4.1 Ports (destination terminals)

```js
const PORTS = [
  { id: "apapa",     name: "Lagos Port Complex (Apapa)",  lat: 6.4450, lng: 3.3650, type: "seaport", state: "Lagos" },
  { id: "tincan",    name: "Tin Can Island Port",          lat: 6.4300, lng: 3.3500, type: "seaport", state: "Lagos" },
  { id: "lekki",     name: "Lekki Deep Sea Port",          lat: 6.4200, lng: 4.1000, type: "seaport", state: "Lagos" },
  { id: "onne",      name: "Onne Port",                    lat: 4.7200, lng: 7.1600, type: "seaport", state: "Rivers" },
  { id: "ph",        name: "Port Harcourt Port",           lat: 4.8100, lng: 7.0000, type: "seaport", state: "Rivers" },
  { id: "calabar",   name: "Calabar Port",                 lat: 4.9700, lng: 8.3300, type: "seaport", state: "Cross River" },
  { id: "mmia",      name: "Murtala Muhammed Airport (air cargo)", lat: 6.5770, lng: 3.3210, type: "airport", state: "Lagos" }
];
```

### 4.2 Intermediate nodes (aggregation, processing, dry ports)

```js
const NODES = [
  // Inland dry ports
  { id: "dala-kano",  name: "Dala Inland Dry Port",    lat: 12.0000, lng: 8.5200, type: "dry-port", state: "Kano" },
  { id: "kaduna-icd", name: "Kaduna Inland Container Depot", lat: 10.5200, lng: 7.4400, type: "dry-port", state: "Kaduna" },
  { id: "funtua-idp", name: "Funtua Inland Dry Port",  lat: 11.5200, lng: 7.3100, type: "dry-port", state: "Katsina" },
  { id: "ibadan-icnl",name: "Ibadan Inland Terminal",   lat: 7.3900,  lng: 3.9000, type: "dry-port", state: "Oyo" },

  // Major markets / aggregation
  { id: "dawanau",    name: "Dawanau International Grains Market", lat: 12.0810, lng: 8.5070, type: "market", state: "Kano",
    notes: "West Africa's largest grain market; sesame, hibiscus, sorghum primary aggregation" },
  { id: "mile12",     name: "Mile 12 Market (Lagos)",   lat: 6.5770, lng: 3.3830, type: "market", state: "Lagos" },
  { id: "zaki-ibiam", name: "Zaki Ibiam Yam Market",    lat: 7.0500, lng: 9.4000, type: "market", state: "Benue",
    notes: "REQUIRES_GEOCODING confirmation" },

  // Processing plants (known or planned)
  { id: "eleme",      name: "Indorama Eleme Petrochemical", lat: 4.7800, lng: 7.1100, type: "processing", state: "Rivers",
    commodity: ["urea","fertiliser"] },
  { id: "dangote-lekki", name: "Dangote Refinery & Fertiliser Lekki", lat: 6.4200, lng: 4.1500, type: "processing", state: "Lagos",
    commodity: ["urea","crude"] },
  { id: "ftn-cocoa",  name: "FTN Cocoa Processors",     lat: 7.2500, lng: 5.2000, type: "processing", state: "Ondo",
    commodity: ["cocoa"], notes: "Akure area — REQUIRES_GEOCODING" },
  { id: "segilola",   name: "Segilola Gold Mine",       lat: 7.5800, lng: 4.8000, type: "mine", state: "Osun",
    commodity: ["gold"], notes: "Iperindo area, Osun. Only industrial-scale gold producer." },
  { id: "avatar-nasarawa", name: "Avatar New Energy Lithium Plant", lat: 8.5400, lng: 7.7100, type: "processing", state: "Nasarawa",
    commodity: ["lithium"], notes: "Nasarawa town area — REQUIRES_GEOCODING" },

  // Add more as needed from research reports
];
```

### 4.3 The 10 commodities (the heart of the spec)

Each commodity has: `id, name, category, origin_belt (lat/lng + radius for halo), producing_states, annual_volume, annual_value, waypoints (ordered array of node/port IDs and inline lat/lngs), modality_per_segment, chokepoints, unlocks, insight_copy`.

```js
const COMMODITIES = [

  // ========== CROPS ==========
  {
    id: "cocoa",
    name: "Cocoa",
    category: "crop",
    icon: "🌰",
    origin_belt: { lat: 7.25, lng: 5.20, radius_km: 150, label: "Ondo–Cross River cocoa belt" },
    producing_states: ["Ondo","Cross River","Edo","Osun","Ekiti"],
    annual_volume: "344,000 tonnes (2024/25)",
    annual_value: "₦3.6 trillion (June 2024–June 2025)",
    dest_port: "apapa",
    waypoints_2024: [
      { lat: 7.25, lng: 5.20, label: "Ondo farmgate (Akure area)", type: "origin" },
      { lat: 7.55, lng: 5.10, label: "Ile-Oluji fermentation racks", type: "processing" },
      { lat: 7.15, lng: 3.35, label: "Ibadan aggregation", type: "aggregation" },
      { port: "apapa" }
    ],
    waypoints_unlocked: [
      { lat: 7.25, lng: 5.20, label: "Ondo farmgate (EUDR geo-tagged)", type: "origin" },
      { lat: 7.55, lng: 5.10, label: "Ile-Oluji fermentation + traceability", type: "processing" },
      { port: "lekki", via_rail: true }   // Lekki rail spur activates
    ],
    modality_2024: ["road","road","road"],
    modality_unlocked: ["road","rail","sea"],
    distance_km_2024: 450,
    distance_km_unlocked: 420,
    transit_days_2024: 3.5,
    transit_days_unlocked: 1.8,
    chokepoints: ["apapa-gridlock","eudr-deadline","tree-age","lagos-ibadan-toll"],
    unlocks: ["lekki-rail-spur","eudr-traceability"],
    insight: {
      opportunity: "Cocoa earns ₦3.6T/year; Netherlands buys 46% of value. Lekki rail + EUDR compliance captures the EU premium and avoids the 15–25% discount to lower-tier markets.",
      current_state: "Road-only evacuation through Apapa gridlock; Ondo farms not yet GPS-verified; Dec 2025 EUDR deadline approaching.",
      after_unlock: "Lekki rail spur + EUDR-compliant origin contracts preserve EU access at premium. Replanting still a 4–7 year capex gap, but export mechanism secured."
    },
    pdf_page: 5
  },

  {
    id: "sesame",
    name: "Sesame seed",
    category: "crop",
    icon: "⚪",
    origin_belt: { lat: 11.50, lng: 9.00, radius_km: 250, label: "Northern sesame belt (Jigawa, Nasarawa, Yobe, Benue)" },
    producing_states: ["Jigawa","Nasarawa","Yobe","Benue","Kano"],
    annual_volume: "510,000 tonnes (2022 est.)",
    annual_value: "World's largest sesame exporter by value, 2024 ($2,290/tonne)",
    dest_port: "apapa",
    waypoints_2024: [
      { lat: 11.76, lng: 9.34, label: "Dutse / Malam-Madori (Jigawa)", type: "origin" },
      { node: "dawanau", label: "Dawanau aggregation, Kano" },
      { lat: 10.52, lng: 7.44, label: "Kaduna road bottleneck", type: "chokepoint" },
      { lat: 9.06,  lng: 7.49, label: "Abuja–Lokoja A2 corridor" },
      { lat: 7.39,  lng: 3.90, label: "Ibadan A1 corridor" },
      { port: "apapa" }
    ],
    waypoints_unlocked: [
      { lat: 11.76, lng: 9.34, label: "Dutse / Malam-Madori (Jigawa)", type: "origin" },
      { node: "dawanau" },
      { node: "dala-kano", label: "Dala IDP rail loading" },
      { port: "apapa", via_rail: true }   // Lagos-Kano narrow gauge
    ],
    modality_2024: ["road","road","road","road","road"],
    modality_unlocked: ["road","road","rail"],
    distance_km_2024: 1150,
    distance_km_unlocked: 1124,
    chokepoints: ["nw-banditry","a2-checkpoint-extortion","processing-gap-lagos-concentrated"],
    unlocks: ["lagos-kano-rail-freight","kano-hulling-capacity"],
    insight: {
      opportunity: "Nigeria is the world's top sesame exporter by value. Kano hulling + rail freight captures the 60% value gap currently leaking to Chinese and Turkish processors.",
      current_state: "1,150 km road trip through bandit-exposed NW; ~₦100k checkpoint extortion; 97% ships as raw seed.",
      after_unlock: "Lagos–Kano narrow gauge (reopened June 2024) handles bulk; Kano-sited hulling captures processing margin locally."
    },
    pdf_page: 7
  },

  {
    id: "cassava",
    name: "Cassava & derivatives",
    category: "crop",
    icon: "🥔",
    origin_belt: { lat: 7.0, lng: 5.5, radius_km: 400, label: "Nationwide cassava (concentrated SW, SS)" },
    producing_states: ["Ogun","Oyo","Ondo","Cross River","Benue","Kogi"],
    annual_volume: "~60 million tonnes (world's largest producer)",
    annual_value: "HQCF, starch, ethanol — primarily domestic; export value limited",
    dest_port: "apapa",
    waypoints_2024: [
      { lat: 7.00, lng: 4.00, label: "Ogun–Oyo cassava belt", type: "origin" },
      { lat: 7.58, lng: 4.25, label: "Psaltry International (Oyo)", type: "processing", notes: "REQUIRES_GEOCODING" },
      { port: "apapa" }
    ],
    waypoints_unlocked: [
      { lat: 7.00, lng: 4.00, label: "Ogun–Oyo cassava belt", type: "origin" },
      { lat: 7.58, lng: 4.25, label: "Psaltry-model HQCF plant (scaled)", type: "processing" },
      { port: "lekki" }
    ],
    modality_2024: ["road","road"],
    modality_unlocked: ["road","road"],
    chokepoints: ["processing-gap-hqcf","currency-export-competitiveness"],
    unlocks: ["processing-plant-network","hqcf-export-standards"],
    insight: {
      opportunity: "Nigeria produces more cassava than Thailand but exports almost none. Psaltry-model HQCF plants can unlock a multi-hundred-million-dollar export stream.",
      current_state: "Volume enormous, export value near zero because Nigeria lacks industrial starch/HQCF capacity at scale.",
      after_unlock: "Processing plants near farmgate + HQCF export standards open EU, Middle East, Asian markets for cassava derivatives."
    },
    pdf_page: 6
  },

  {
    id: "ginger",
    name: "Ginger",
    category: "crop",
    icon: "🫚",
    origin_belt: { lat: 9.87, lng: 7.95, radius_km: 80, label: "Southern Kaduna (Kachia LGA belt)" },
    producing_states: ["Kaduna"],
    annual_volume: "COLLAPSED — 2,500+ ha wiped out, 85–95% yield loss (2023 blight)",
    annual_value: "₦6.28 billion (first 9 months 2024, −74% YoY)",
    dest_port: "apapa",
    waypoints_2024: [
      { lat: 9.87, lng: 7.95, label: "Kachia farmgate (blight-affected)", type: "origin", status: "damaged" },
      { lat: 9.62, lng: 6.55, label: "Minna aggregation (limited)" },
      { port: "apapa" }
    ],
    waypoints_unlocked: [
      { lat: 9.87, lng: 7.95, label: "Kachia — blight-free certified seed", type: "origin" },
      { lat: 9.87, lng: 7.95, label: "Kachia cold chain + washing plant", type: "processing" },
      { lat: 10.52, lng: 7.44, label: "Kaduna bonded warehouse" },
      { port: "apapa", via_rail: true }
    ],
    modality_2024: ["road","road"],
    modality_unlocked: ["road","road","rail"],
    chokepoints: ["ginger-blight","seed-scarcity","processing-gap-powder"],
    unlocks: ["gbect-blight-program","kachia-cold-chain","blight-free-certification"],
    insight: {
      opportunity: "Nigeria was the world's 2nd largest ginger producer. Blight-free seed + cold chain + in-Kachia processing rebuilds the ₦40B+ export market.",
      current_state: "Fungal blight wiped 85–95% of yields. Ethiopia, China, India captured the lost Europe/Middle East demand.",
      after_unlock: "GBECT-distributed clean seed + in-belt cold chain + powder/oleoresin processing. 2 crop cycles to rebuild offtake credibility."
    },
    pdf_page: 8
  },

  {
    id: "hibiscus",
    name: "Hibiscus (zobo)",
    category: "crop",
    icon: "🌺",
    origin_belt: { lat: 12.50, lng: 8.50, radius_km: 300, label: "NW hibiscus belt (Jigawa, Kano, Katsina, Bauchi, Kebbi, Sokoto)" },
    producing_states: ["Jigawa","Kano","Katsina","Bauchi","Kebbi","Sokoto"],
    annual_volume: "~₦48B grower earnings target",
    annual_value: "Farmgate ₦1.7–2M/tonne (post-2021 Mexico reopening)",
    dest_port: "apapa",
    waypoints_2024: [
      { lat: 12.99, lng: 7.60, label: "Katsina / Jigawa farmgate", type: "origin" },
      { node: "dawanau" },
      { lat: 9.06,  lng: 7.49, label: "Abuja A2 corridor" },
      { port: "apapa" }
    ],
    waypoints_unlocked: [
      { lat: 12.99, lng: 7.60, label: "Jigawa/Katsina solar-drying standard", type: "origin" },
      { node: "dawanau" },
      { node: "dala-kano", label: "Dala IDP rail loading" },
      { port: "apapa", via_rail: true },
      // Air cargo optional branch to Mexico
      { port: "mmia", via_air: true, branch: true, label: "MMIA air cargo → Mexico direct" }
    ],
    modality_2024: ["road","road","road"],
    modality_unlocked: ["road","road","rail", "air-branch"],
    chokepoints: ["nw-banditry","solar-drying-quality","mexico-dependence"],
    unlocks: ["solar-drying-standardization","lagos-kano-rail-freight","mexico-direct-contracts"],
    insight: {
      opportunity: "Mexico takes 85% of Nigerian hibiscus for Agua de Jamaica. Standardized solar drying + rail freight + direct Mexican buyer contracts hit the $3B ambassador target at scale.",
      current_state: "Quality inconsistency forces price discounts; rail capacity insufficient; 100% dependent on Mexico demand.",
      after_unlock: "Solar-drying SOP + rail + diversified buyers (EU herbal tea, US wellness) de-risks the Mexico concentration."
    },
    pdf_page: 7
  },

  // ========== MINERALS ==========
  {
    id: "gold",
    name: "Gold (industrial + artisanal)",
    category: "mineral",
    icon: "◆",
    origin_belt: { lat: 9.0, lng: 6.5, radius_km: 500, label: "Zamfara–Niger–Kaduna–Osun gold zone" },
    producing_states: ["Osun","Zamfara","Kebbi","Niger","Kaduna"],
    annual_volume: "Segilola formal: 91,910 oz (2025); Artisanal: estimated multiples",
    annual_value: "Segilola ~$230M; estimated informal ~$2B flowing to UAE",
    dest_port: "mmia",
    waypoints_2024: [
      { node: "segilola", label: "Segilola (Osun) — only formal producer" },
      { lat: 12.11, lng: 5.93, label: "Zamfara artisanal (Anka, Bukkuyum)", type: "origin", status: "informal" },
      { lat: 11.00, lng: 6.00, label: "Informal flow → Togo/Dubai", type: "chokepoint", status: "illegal" },
      { port: "mmia", label: "MMIA formal air export (Segilola)" }
    ],
    waypoints_unlocked: [
      { node: "segilola" },
      { lat: 12.11, lng: 5.93, label: "Zamfara artisanal → LICENSED buying centres", type: "origin" },
      { lat: 10.52, lng: 7.44, label: "Kaduna central refining/assay", type: "processing" },
      { port: "mmia", label: "MMIA formal air export — UAE direct" }
    ],
    modality_2024: ["air"],
    modality_unlocked: ["road","air"],
    chokepoints: ["informal-gold-flows","zamfara-banditry-legacy","lead-poisoning","togo-smuggling-route"],
    unlocks: ["zamfara-reopening-2024","licensed-buying-centres","uae-bilateral-formalization"],
    insight: {
      opportunity: "If even partial formalisation of the $2B/year informal flow happens, declared mineral exports could double inside 2 years.",
      current_state: "Segilola declares properly; Zamfara artisanal output flows informally via Togo to UAE at massive value leakage.",
      after_unlock: "Zamfara reopened Dec 2024; licensed buying centres + Nigeria–UAE bilateral channel converts informal flows to declared exports."
    },
    pdf_page: 9
  },

  {
    id: "lithium",
    name: "Lithium (spodumene)",
    category: "mineral",
    icon: "◆",
    origin_belt: { lat: 8.50, lng: 7.50, radius_km: 150, label: "NC pegmatite belt (Nasarawa, Kwara, Kogi, Oyo, Ekiti)" },
    producing_states: ["Nasarawa","Kwara","Kogi","Oyo","Ekiti"],
    annual_volume: "Rising — Avatar + Ganfeng + Jupiter plants coming online",
    annual_value: "Unprocessed export currently banned; processed value rising",
    dest_port: "onne",
    waypoints_2024: [
      { lat: 8.54, lng: 7.71, label: "Nasarawa spodumene", type: "origin" },
      { lat: 8.50, lng: 7.70, label: "Small-scale sorting (informal)", type: "processing", status: "informal" },
      { lat: 7.00, lng: 7.50, label: "Road evacuation south" },
      { port: "onne" }
    ],
    waypoints_unlocked: [
      { lat: 8.54, lng: 7.71, label: "Nasarawa spodumene", type: "origin" },
      { node: "avatar-nasarawa", label: "Avatar/Ganfeng/Jupiter chemical plants" },
      { lat: 7.50, lng: 7.00, label: "Processed Li carbonate bonded warehouse" },
      { port: "onne" }
    ],
    modality_2024: ["road","road","road"],
    modality_unlocked: ["road","road","road"],
    chokepoints: ["unprocessed-export-ban","informal-processing-leakage","plant-construction-delays"],
    unlocks: ["avatar-plant-commissioning","ganfeng-plant-operational","export-ban-enforcement"],
    insight: {
      opportunity: "Global EV demand + Nigerian government pressure for in-country processing = capture the refining margin Chinese players take elsewhere.",
      current_state: "Unprocessed ore smuggled despite the export ban; processing plants under construction but not yet at capacity.",
      after_unlock: "Fully operational chemical plants + enforced unprocessed ban = Li carbonate exports to global battery supply chains."
    },
    pdf_page: 9
  },

  {
    id: "tin-columbite",
    name: "Tin / Columbite / Tantalite",
    category: "mineral",
    icon: "◆",
    origin_belt: { lat: 9.80, lng: 8.87, radius_km: 100, label: "Jos Plateau legacy tin belt" },
    producing_states: ["Plateau","Nasarawa","Bauchi"],
    annual_volume: "Artisanal only since 1980s; 3T minerals also present",
    annual_value: "Informal; declared exports minor",
    dest_port: "apapa",
    waypoints_2024: [
      { lat: 9.80, lng: 8.87, label: "Bukuru artisanal pits", type: "origin", status: "informal" },
      { lat: 9.92, lng: 8.89, label: "Jos informal buyers (often Chinese)", type: "aggregation", status: "informal" },
      { lat: 9.06, lng: 7.49, label: "Abuja/Lagos road" },
      { port: "apapa" }
    ],
    waypoints_unlocked: [
      { lat: 9.80, lng: 8.87, label: "Bukuru formalised ASM cooperative", type: "origin" },
      { lat: 9.92, lng: 8.89, label: "Jos licensed buying centre + 3T certification", type: "processing" },
      { lat: 10.52, lng: 7.44, label: "Kaduna bonded warehouse" },
      { port: "apapa" }
    ],
    modality_2024: ["road","road","road"],
    modality_unlocked: ["road","road","road"],
    chokepoints: ["3t-certification-gap","chinese-informal-buyers","plateau-legacy-mining-damage"],
    unlocks: ["3t-certification-deployment","licensed-buying-centres","ree-co-production-kanam"],
    insight: {
      opportunity: "Jos Plateau has massive legacy infrastructure and known deposits. 3T certification unlocks conflict-free premium markets.",
      current_state: "Informal Chinese buyers capture most margin; no 3T certification; legacy environmental issues.",
      after_unlock: "Licensed cooperatives + 3T cert + REE co-production from Kanam monazite = legitimate export stream."
    },
    pdf_page: 9
  },

  {
    id: "lead-zinc",
    name: "Lead & Zinc",
    category: "mineral",
    icon: "◆",
    origin_belt: { lat: 6.32, lng: 8.11, radius_km: 80, label: "Abakaliki–Enyigba–Ameka belt" },
    producing_states: ["Ebonyi","Nasarawa","Benue","Plateau","Cross River"],
    annual_volume: "Artisanal + 3 foreign-backed companies",
    annual_value: "Mostly to Chinese smelters",
    dest_port: "calabar",
    waypoints_2024: [
      { lat: 6.32, lng: 8.11, label: "Enyigba artisanal pits (Ebonyi)", type: "origin", status: "informal" },
      { lat: 6.32, lng: 8.11, label: "Informal concentration", type: "processing" },
      { lat: 4.97, lng: 8.33, label: "Calabar road evacuation" },
      { port: "calabar" }
    ],
    waypoints_unlocked: [
      { lat: 6.32, lng: 8.11, label: "Ebonyi formal mining + environmental compliance", type: "origin" },
      { lat: 6.32, lng: 8.11, label: "Licensed concentration + sorting", type: "processing" },
      { lat: 5.00, lng: 8.00, label: "Calabar Free Trade Zone bonded smelter" },
      { port: "calabar" }
    ],
    modality_2024: ["road","road","road"],
    modality_unlocked: ["road","road","road"],
    chokepoints: ["child-labour-ebonyi","environmental-damage","calabar-shallow-draft"],
    unlocks: ["environmental-compliance-regime","calabar-ftz-smelter","formalization"],
    insight: {
      opportunity: "Proximity to Calabar FTZ + oil-services demand + battery recycling economics = domestic smelting rather than Chinese export.",
      current_state: "Artisanal pits with child-labour incidents; environmental damage; ore leaves cheap to Chinese smelters.",
      after_unlock: "FTZ smelter + formal mining permits + environmental regime = declared exports of refined lead/zinc."
    },
    pdf_page: 9
  },

  {
    id: "barite",
    name: "Barite (API-grade for drilling)",
    category: "mineral",
    icon: "◆",
    origin_belt: { lat: 8.00, lng: 9.24, radius_km: 120, label: "Azara + Benue + Taraba barite fields" },
    producing_states: ["Nasarawa","Benue","Taraba","Cross River","Plateau"],
    annual_volume: "22M+ tonnes inferred reserves (Azara); Nigeria still imports drilling barite",
    annual_value: "Domestic oil-services demand + export potential",
    dest_port: "onne",
    waypoints_2024: [
      { lat: 8.00, lng: 9.24, label: "Azara (Awe LGA, Nasarawa) — below API spec", type: "origin" },
      { lat: 7.00, lng: 9.00, label: "Benue/Taraba higher-grade sites (96.5% BaSO4)" },
      { lat: 5.50, lng: 7.50, label: "Truck route south" },
      { port: "onne" }
    ],
    waypoints_unlocked: [
      { lat: 8.00, lng: 9.24, label: "Azara + beneficiation plant (API upgrade)", type: "origin" },
      { lat: 7.00, lng: 9.00, label: "Taraba 96.5% BaSO4 direct extraction" },
      { lat: 5.50, lng: 7.50, label: "Onne OGFZ grinding mill (API-13A compliant)" },
      { port: "onne" }
    ],
    modality_2024: ["road","road"],
    modality_unlocked: ["road","road"],
    chokepoints: ["api-spec-gap","import-substitution-incomplete","transport-cost"],
    unlocks: ["beneficiation-capacity","onne-ogfz-grinding","api-13a-certification"],
    insight: {
      opportunity: "Nigeria imports drilling barite it has in the ground. Onne OGFZ-sited API-grade grinding mill substitutes imports AND exports to West African offshore.",
      current_state: "Azara doesn't meet API spec; higher-grade Taraba too remote; oil companies import from China/Morocco.",
      after_unlock: "Beneficiation + OGFZ grinding = domestic supply + export to Ghana, Angola, Congo offshore operations."
    },
    pdf_page: 9
  }
];
```

### 4.4 Chokepoints registry

```js
const CHOKEPOINTS = [
  { id: "apapa-gridlock", lat: 6.44, lng: 3.37, severity: "high", type: "infrastructure",
    label: "Apapa port gridlock", cost_note: "2–7 day truck queues; ₦200k+ demurrage typical" },
  { id: "eudr-deadline", lat: 7.25, lng: 5.20, severity: "medium", type: "regulatory",
    label: "EUDR compliance deadline (Dec 2025)", cost_note: "15–25% price discount if non-compliant" },
  { id: "tree-age", lat: 7.25, lng: 5.20, severity: "medium", type: "structural",
    label: "Aging cocoa trees past peak yield", cost_note: "4–7 year replanting capex" },
  { id: "lagos-ibadan-toll", lat: 6.90, lng: 3.60, severity: "medium", type: "infrastructure",
    label: "Lagos–Ibadan expressway toll/congestion" },

  { id: "nw-banditry", lat: 12.00, lng: 6.00, severity: "critical", type: "security",
    label: "NW banditry corridor", zone: [[12.5,5.0],[13.0,8.0],[11.0,8.5],[10.8,5.5]],
    cost_note: "Kidnapping risk; road impassable seasonally" },
  { id: "a2-checkpoint-extortion", lat: 9.50, lng: 7.50, severity: "high", type: "extortion",
    label: "A2 checkpoint extortion", cost_note: "~₦100,000 per trip in peak zones" },
  { id: "processing-gap-lagos-concentrated", lat: 6.50, lng: 3.40, severity: "high", type: "structural",
    label: "Processing capacity concentrated in Lagos/Kano only",
    cost_note: "97% sesame ships as raw seed; value leaks to China/Turkey" },

  { id: "processing-gap-hqcf", lat: 7.00, lng: 4.00, severity: "high", type: "structural",
    label: "No industrial HQCF/starch capacity at scale" },
  { id: "currency-export-competitiveness", lat: 6.50, lng: 3.40, severity: "medium", type: "macro",
    label: "Naira volatility impacts export contract stability" },

  { id: "ginger-blight", lat: 9.87, lng: 7.95, severity: "critical", type: "biological",
    label: "Fungal blight — Kachia belt", cost_note: "2,500+ ha wiped; 85–95% yield loss" },
  { id: "seed-scarcity", lat: 9.87, lng: 7.95, severity: "high", type: "input",
    label: "Blight-free ginger seed in short supply" },
  { id: "processing-gap-powder", lat: 9.87, lng: 7.95, severity: "medium", type: "structural",
    label: "No ginger powder/oleoresin processing near farmgate" },

  { id: "solar-drying-quality", lat: 12.00, lng: 8.50, severity: "medium", type: "quality",
    label: "Hibiscus solar drying quality inconsistency" },
  { id: "mexico-dependence", lat: 6.50, lng: 3.40, severity: "medium", type: "market",
    label: "85% hibiscus buyer concentration (Mexico)" },

  { id: "informal-gold-flows", lat: 12.11, lng: 5.93, severity: "critical", type: "smuggling",
    label: "Informal gold → UAE via Togo", cost_note: "Est. $2B/year Nigerian share" },
  { id: "zamfara-banditry-legacy", lat: 12.11, lng: 5.93, severity: "high", type: "security",
    label: "Zamfara legacy banditry (reopened Dec 2024 but fragile)" },
  { id: "lead-poisoning", lat: 12.11, lng: 5.93, severity: "critical", type: "health",
    label: "Ore-bound lead poisoning epidemic" },
  { id: "togo-smuggling-route", lat: 7.00, lng: 2.50, severity: "critical", type: "smuggling",
    label: "Togo pass-through declared origin for Nigerian gold" },

  { id: "unprocessed-export-ban", lat: 8.54, lng: 7.71, severity: "medium", type: "regulatory",
    label: "Unprocessed lithium export ban (enforcement patchy)" },
  { id: "informal-processing-leakage", lat: 8.54, lng: 7.71, severity: "high", type: "smuggling",
    label: "Spodumene smuggled as 'sand' to bypass ban" },
  { id: "plant-construction-delays", lat: 8.54, lng: 7.71, severity: "medium", type: "structural",
    label: "Avatar/Ganfeng construction pace" },

  { id: "3t-certification-gap", lat: 9.80, lng: 8.87, severity: "high", type: "regulatory",
    label: "No 3T (tin/tantalum/tungsten) certification regime in place" },
  { id: "chinese-informal-buyers", lat: 9.92, lng: 8.89, severity: "medium", type: "market",
    label: "Chinese informal buyers capture margin at origin" },
  { id: "plateau-legacy-mining-damage", lat: 9.92, lng: 8.89, severity: "medium", type: "environmental",
    label: "Jos Plateau legacy tin mining environmental damage" },

  { id: "child-labour-ebonyi", lat: 6.32, lng: 8.11, severity: "critical", type: "social",
    label: "Child-labour incidents in Ebonyi pits" },
  { id: "environmental-damage", lat: 6.32, lng: 8.11, severity: "high", type: "environmental",
    label: "Unregulated lead/zinc extraction damage" },
  { id: "calabar-shallow-draft", lat: 4.97, lng: 8.33, severity: "medium", type: "infrastructure",
    label: "Calabar ~8m draft limits vessel size" },

  { id: "api-spec-gap", lat: 8.00, lng: 9.24, severity: "high", type: "quality",
    label: "Azara barite below API-13A drilling spec" },
  { id: "import-substitution-incomplete", lat: 5.50, lng: 7.50, severity: "medium", type: "structural",
    label: "Nigeria imports drilling barite despite having reserves" },
  { id: "transport-cost", lat: 7.00, lng: 9.00, severity: "medium", type: "infrastructure",
    label: "Remote Taraba barite sites have high truck costs" }
];
```

### 4.5 Interventions registry

```js
const INTERVENTIONS = [
  { id: "lekki-rail-spur", lat: 6.42, lng: 4.05, label: "Lekki rail spur (planned)",
    type: "infrastructure", status: "planned", unlocks: ["cocoa","sesame","hibiscus"],
    narrative: "Direct container evacuation from Lekki Deep Sea Port without Apapa gridlock." },
  { id: "eudr-traceability", lat: 7.25, lng: 5.20, label: "EUDR geo-tag deployment",
    type: "compliance", status: "urgent", unlocks: ["cocoa"],
    narrative: "Farm-level GPS verification to preserve EU market access." },

  { id: "lagos-kano-rail-freight", lat: 10.00, lng: 7.50, label: "Lagos–Kano rail freight (reopened June 2024)",
    type: "infrastructure", status: "operational", unlocks: ["sesame","hibiscus"],
    narrative: "1,124 km narrow-gauge line moving 40-ft containers Apapa ↔ Dala IDP." },
  { id: "kano-hulling-capacity", lat: 12.00, lng: 8.52, label: "Kano sesame hulling capacity",
    type: "processing", status: "gap", unlocks: ["sesame"],
    narrative: "In-belt processing captures margin currently leaking to China/Turkey." },

  { id: "processing-plant-network", lat: 7.20, lng: 4.10, label: "Processing plant network (farmgate-proximate)",
    type: "processing", status: "gap", unlocks: ["cassava","ginger","sesame","hibiscus"],
    narrative: "Psaltry-model: HQCF, powder, oleoresin, hulling plants within 200 km of farmgate." },
  { id: "hqcf-export-standards", lat: 7.00, lng: 4.00, label: "HQCF export standards + certification",
    type: "compliance", status: "gap", unlocks: ["cassava"] },

  { id: "gbect-blight-program", lat: 9.87, lng: 7.95, label: "GBECT ginger blight program (₦1.6B)",
    type: "program", status: "active", unlocks: ["ginger"],
    narrative: "Federal taskforce distributing clean seed; 5,000 farmers reached." },
  { id: "kachia-cold-chain", lat: 9.87, lng: 7.95, label: "Kachia cold chain + washing plant",
    type: "infrastructure", status: "gap", unlocks: ["ginger"] },
  { id: "blight-free-certification", lat: 9.87, lng: 7.95, label: "Blight-free ginger certification",
    type: "compliance", status: "gap", unlocks: ["ginger"] },

  { id: "solar-drying-standardization", lat: 12.50, lng: 8.50, label: "Solar drying SOP deployment",
    type: "quality", status: "gap", unlocks: ["hibiscus"] },
  { id: "mexico-direct-contracts", lat: 6.58, lng: 3.32, label: "Mexico direct buyer contracts",
    type: "market", status: "active", unlocks: ["hibiscus"] },

  { id: "zamfara-reopening-2024", lat: 12.11, lng: 5.93, label: "Zamfara mining ban lifted (Dec 2024)",
    type: "regulatory", status: "operational", unlocks: ["gold"],
    narrative: "5-year ban lifted after security improvements and Sububu capture." },
  { id: "licensed-buying-centres", lat: 12.11, lng: 5.93, label: "Licensed gold buying centres",
    type: "regulatory", status: "gap", unlocks: ["gold"],
    narrative: "LBMA-compliant assay + pricing + KYC at source." },
  { id: "uae-bilateral-formalization", lat: 6.58, lng: 3.32, label: "Nigeria–UAE gold bilateral channel",
    type: "diplomacy", status: "in-progress", unlocks: ["gold"],
    narrative: "Minister Alake's Oct 2024 engagement converting informal flows to declared exports." },

  { id: "avatar-plant-commissioning", lat: 8.54, lng: 7.71, label: "Avatar New Energy plant commissioning",
    type: "processing", status: "in-progress", unlocks: ["lithium"] },
  { id: "ganfeng-plant-operational", lat: 8.54, lng: 7.71, label: "Ganfeng lithium plant operational",
    type: "processing", status: "in-progress", unlocks: ["lithium"] },
  { id: "export-ban-enforcement", lat: 8.54, lng: 7.71, label: "Unprocessed export ban enforcement",
    type: "regulatory", status: "patchy", unlocks: ["lithium"] },

  { id: "3t-certification-deployment", lat: 9.80, lng: 8.87, label: "3T (tin-tantalum-tungsten) certification",
    type: "compliance", status: "gap", unlocks: ["tin-columbite"] },
  { id: "ree-co-production-kanam", lat: 9.50, lng: 8.90, label: "REE co-production from Kanam monazite",
    type: "processing", status: "emerging", unlocks: ["tin-columbite"],
    narrative: "Nigeria's rare-earth oxide output up ~80% to 13,000 tonnes in 2024." },

  { id: "environmental-compliance-regime", lat: 6.32, lng: 8.11, label: "Environmental compliance regime (Ebonyi)",
    type: "regulatory", status: "gap", unlocks: ["lead-zinc"] },
  { id: "calabar-ftz-smelter", lat: 4.97, lng: 8.33, label: "Calabar FTZ licensed smelter",
    type: "processing", status: "gap", unlocks: ["lead-zinc"] },
  { id: "formalization", lat: 6.32, lng: 8.11, label: "ASM formalization permits",
    type: "regulatory", status: "in-progress", unlocks: ["lead-zinc","tin-columbite","gold"] },

  { id: "beneficiation-capacity", lat: 8.00, lng: 9.24, label: "Azara barite beneficiation plant",
    type: "processing", status: "gap", unlocks: ["barite"] },
  { id: "onne-ogfz-grinding", lat: 4.72, lng: 7.16, label: "Onne OGFZ API-grade grinding mill",
    type: "processing", status: "gap", unlocks: ["barite"] },
  { id: "api-13a-certification", lat: 4.72, lng: 7.16, label: "API-13A certification for drilling barite",
    type: "compliance", status: "gap", unlocks: ["barite"] }
];
```

### 4.6 Security zones (polygon overlay)

Four semi-transparent polygons, each roughly the published banditry/insurgency footprint:

```js
const SECURITY_ZONES = [
  { id: "nw-banditry", name: "NW banditry corridor",
    color: "#c13a2a",
    polygon: [[13.2,5.0],[13.5,7.5],[11.0,8.8],[10.8,5.2]],  // Zamfara/Katsina/Kebbi/NW Kaduna rough
    states: ["Zamfara","Katsina","Kebbi","Niger","Kaduna"] },
  { id: "ne-insurgency", name: "NE insurgency zone",
    color: "#8b2020",
    polygon: [[13.5,13.5],[13.8,14.7],[10.5,14.3],[10.2,12.5]],  // Borno/Yobe/Adamawa rough
    states: ["Borno","Yobe","Adamawa"] },
  { id: "se-sit-at-home", name: "SE sit-at-home corridor",
    color: "#a85a3a",
    polygon: [[6.8,7.0],[6.8,8.2],[5.3,8.5],[5.2,7.0]],
    states: ["Anambra","Imo","Enugu","Abia","Ebonyi"] },
  { id: "ss-militancy", name: "SS Niger Delta militancy",
    color: "#6b2a2a",
    polygon: [[5.8,5.0],[5.5,7.5],[4.3,7.8],[4.5,5.2]],
    states: ["Rivers","Bayelsa","Delta"] }
];
```

---

## 5. Build checklist (ordered)

Build in this order — each step is independently testable:

1. **Shell:** HTML skeleton, Leaflet map centered on Nigeria (9.0, 8.5, zoom 6), CartoDB Positron basemap, two tile layer definitions for switching Pitch/Ops
2. **Ports + basic nodes:** render the 7 ports, basic NODES as markers with tooltips
3. **Security zones:** render 4 polygons, hidden by default unless OPS mode
4. **Commodity sidebar:** static list of 10 chips, hover state, active state
5. **Commodity selection logic:** on chip click, pan/fit-bounds to that commodity's extent, show corridor polyline (2024 state only first)
6. **Chokepoint markers:** render chokepoints for selected commodity, with severity colors
7. **Insight panel:** right-side collapsible, populate from commodity.insight.current_state
8. **Per-commodity unlock toggle:** flipping toggle swaps waypoints_2024 → waypoints_unlocked, redraws polyline, fades chokepoints, renders intervention markers
9. **Global Pitch/Ops toggle:** swap basemaps, change marker sizes, show/hide security zones, swap insight panel tone
10. **Polish:** polyline draw-in animation (stroke-dashoffset), intervention marker glow, transitions between states, responsive layout
11. **Layer controls:** bottom-left toggle panel for rail/road/waterways/NiPost backbone/security
12. **Cross-link to main map:** subtle link bottom-right: "View full Norhaven Asset Intelligence Map →"

### 5.1 Testing milestones

- [ ] Map renders and pans smoothly at zoom 6–12
- [ ] All 7 ports visible with correct tooltips
- [ ] Clicking "Cocoa" pans to SW, draws polyline, shows chokepoints
- [ ] Unlock toggle on cocoa reroutes through Lekki with rail modality
- [ ] Pitch mode hides security zones; Ops mode shows them
- [ ] Insight panel updates when commodity changes
- [ ] Works in Chrome, Firefox, Safari (desktop); degrades acceptably on iPad
- [ ] Single file under 500 KB (excluding Leaflet CDN)

---

## 6. Source references

Two existing research artifacts live in context and should be referenced when ambiguity arises:

1. **NiPost asset inventory research** (prior turn in the Norhaven project chat)
   - 774-LGA footprint
   - Rail-adjacent GPOs (Ibadan Dugbe, Enugu Ogui, PH Station Rd, Kano Post Office Rd, Lagos MMIA IMPC)
   - Used for the `☐ NiPost backbone` layer toggle

2. **Nigeria export corridor plan research** (prior turn, longer)
   - Full corridor specs for the 10 commodities
   - Port concessions and terminal operators
   - Chokepoint taxonomy
   - Intervention inventory

3. **Designed PDF report** (`nigeria_export_report.pdf`)
   - Hero numbers and insight panel copy should reference PDF page numbers in tooltips (`pdf_page` field on each commodity)
   - Design palette (forest/terracotta/gold/cream) is the source of truth for colors

---

## 7. Constraints & gotchas

- **No fabricated GPS coordinates.** If a place can't be verified, use the state capital and flag `REQUIRES_GEOCODING` in a code comment.
- **Do NOT import NiPost asset data into this map wholesale.** Just provide the toggle link to the main Norhaven map. Keep scope tight.
- **No build step, no npm, no framework.** Leaflet from CDN, vanilla JS, inline CSS, single HTML file.
- **Security zones are approximations.** Polygon vertices in §4.6 are rough outlines; do not represent them as precise threat assessments in the UI copy.
- **All figures must cite source.** Every number in the insight panel should include a `(PDF p. X)` or `(NBS Q1 2025)` style caveat.
- **Mobile is secondary.** Desktop-first; tablet acceptable; phone degrades to "please view on larger screen" if needed.
- **Respect the Jeff style.** Terse, execution-focused, no marketing fluff in UI copy. The Insight panel should read like a briefing note, not a pitch deck (even in Pitch mode — "Pitch mode" here means *aspirational data*, not breathless prose).

---

## 8. Acceptance criteria

When Claude Code considers the build done, all of these should be true:

- [ ] One HTML file, opens in any browser, works offline after first load
- [ ] 10 commodities visible as chips, all clickable
- [ ] Each commodity shows at least one 2024-state chokepoint and one unlock intervention
- [ ] Unlock toggle produces a visible change (chokepoints fade, polyline redraws, interventions appear)
- [ ] Pitch ↔ Ops toggle produces visible differences as tabled in §3.5
- [ ] All 7 ports + 4 dry ports + 3+ processing plants visible in base layer
- [ ] Security zones polygon overlay works and is off by default in Pitch
- [ ] Insight panel cites source (PDF page or data source) for every figure
- [ ] Corridor polylines use modality-based styling (road/rail/river/air distinct)
- [ ] Bottom-right: subtle link back to main Norhaven asset intelligence map

---

## 9. Stretch goals (do last, only if time)

- Animated droplet along the polyline showing commodity flow direction
- Mini-timeline showing when each intervention is expected (2024 operational, 2026 planned, etc.)
- "Economics" tab in insight panel showing cost/tonne breakdown road vs rail vs unlocked
- Share-state URL hash (`#cocoa=unlocked&mode=pitch`) for embedding
- Export current view as PNG for slide decks

---

## 10. Notes for the implementer

- Corridor polylines should NOT follow actual roads or rail lines at first pass. Straight-line segments between waypoints are acceptable and arguably clearer. Only upgrade to road-following polylines if it adds clarity (it often doesn't).
- The "pulse" animation on chokepoints: SVG circle with `animate` tag on `r` between 6 and 10px, 1.5s infinite. Simple and effective.
- Test the Pitch/Ops transition early — it's easy to build in a way that forces a full re-render. Use CSS classes on the map container + Leaflet layer visibility, not tile-layer swaps if possible.
- The insight panel copy in §4.3 is a *starting point*. Edit for voice fit as you build — shorter is better.
- Don't over-engineer the marker icon system. Leaflet's built-in `divIcon` with SVG-in-HTML is enough; no need for Leaflet.awesome-markers or similar.

---

## 11. Done criteria, one sentence

> A user lands on the page, sees Nigeria with ten commodity chips on the left, clicks "Cocoa," watches a polyline flow from the Ondo belt through Ibadan to Apapa with chokepoints pulsing; they flip the unlock toggle and watch the polyline reroute through Lekki rail while the chokepoints fade; they flip the global Pitch → Ops toggle and the security zones and extortion cost labels appear; they close the tab knowing exactly what Nigeria's export corridors look like today and what needs to happen for them to open.
