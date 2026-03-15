# Energy Island Dashboard

**Live:** [agus-sam.github.io/energy_island_dashboard](https://agus-sam.github.io/energy_island_dashboard/)

An interactive web dashboard for visualising and comparing results of the [Energy Island PyPSA-HiGHS](https://github.com/agus-sam/energy-island-pypsa-highs) optimisation model across multiple objective scenarios. Built with vanilla HTML, CSS, Chart.js, and Plotly — no build tools, no framework, no server required.

---

## Preview

The dashboard reads up to three scenario JSON files exported from the optimisation notebook and renders a full suite of charts, KPIs, and energy flow diagrams. Switching between scenarios via the **Objective dropdown** in the header reloads all charts instantly without a full page refresh. All remaining scenarios are silently pre-fetched in the background so the comparison panel populates automatically.

---

## Live Dashboard

🔗 **[agus-sam.github.io/energy_island_dashboard](https://agus-sam.github.io/energy_island_dashboard/)**

To update results, replace or add JSON files in this repository and push. GitHub Pages serves the updated data within seconds — no code changes required.

---

## Multi-Scenario Setup

The dashboard compares three optimisation objectives, each backed by its own JSON file:

| Dropdown label | JSON file | Objective |
|---|---|---|
| Lowest LCOE | `data_lcoe.json` | Minimise system levelised cost of energy |
| Lowest CO₂ | `data_co2.json` | Minimise annual direct CO₂ emissions |
| Most Diversified | `data_diversified.json` | Maximise technology mix diversity |

Selecting a scenario in the header dropdown triggers a soft reload — a blurred overlay appears briefly while the new JSON is fetched, then all charts, KPIs, and insight cards update simultaneously. Once all three scenarios have loaded, the **Scenario Comparison** panel auto-populates with a side-by-side visual breakdown.

To change the scenario labels or filenames, edit the `SCENARIOS` object at the top of the `<script>` block in `index.html`:

```js
const SCENARIOS = {
  'Lowest LCOE':      'data_lcoe.json',
  'Lowest CO₂':       'data_co2.json',
  'Most Diversified': 'data_diversified.json',
};
```

Adding a fourth scenario is a single line here — no other changes needed.

---

## Generating Results

1. Run the optimisation notebook: [Energy Island PyPSA-HiGHS](https://github.com/agus-sam/energy-island-pypsa-highs)
2. For each objective, execute **Step 7 — File Export**:
   ```python
   # Lowest LCOE scenario
   model.export_dashboard_json("data_lcoe.json")

   # Lowest CO₂ scenario
   model.export_dashboard_json("data_co2.json")

   # Most Diversified scenario
   model.export_dashboard_json("data_diversified.json")
   ```
3. Copy all three JSON files into this repository
4. Commit and push — the live dashboard updates automatically

> **Note:** Use `export_dashboard_json()`, not the older `export_json()`. The dashboard format uses lowercase snake_case keys and includes additional fields (`curtailment_gwh`, split `gross_demand_mw` / `net_demand_mw`, cost decomposition fields, and optionally `geographic`) that the older exporter does not produce.

---

## Dashboard Sections

| Section | Content |
|---|---|
| **Header** | Objective dropdown, currency, discount rate, grid loss factor, technology count, PDF export button, and an optional Leaflet map showing the project location |
| **KPI Strip** | System LCOE, net demand served (with gross in sub-label and monthly sparkline), renewable share (with animated SVG ring), CO₂ intensity, peak demand, total system CAPEX (with monthly sparkline) |
| **System Narrative** | Auto-generated one-sentence summary of the optimal system: installed capacities, LCOE, RE share, CO₂ intensity, and upfront capital requirement |
| **Scenario Comparison** | Visual card panel comparing all loaded scenarios across LCOE, RE share, CO₂ intensity, overnight CAPEX, and annualised system cost — with mini bar charts and best/worst highlighting |
| **Auto-insights** | 8+ cards computed dynamically: worst RE month, critical week, peak demand, cheapest source, dominant technology, VRE curtailment, system load factor, total system CAPEX |
| **A — Installed Capacity Mix** | Installed MW per technology + stacked potential-vs-installed bar showing resource utilisation headroom |
| **B — Hourly Dispatch** | Filterable hourly dispatch by technology and month (with optional week zoom); single-tech view uses Chart.js, all-technologies view uses Plotly stacked area; auto-detected 168-hour worst week shown below |
| **C — Demand vs Generation** | Annual generation mix donut + animated energy balance bars (sources → net load, grid loss, storage charge, RTE loss, curtailment); monthly stacked bar vs net demand line; annual Sankey energy flow |
| **D — Storage Operation** | Load duration curve with peak/avg/min insights; storage state-of-charge trajectory; storage utilisation table (MW, MWh, E/P ratio, CAPEX/yr, OPEX/yr, round-trip η, annual cycles); demand heatmap (hour-of-day × month) |
| **E — System Cost Breakdown** | Switchable cost view (total annualised / annualised CAPEX / overnight CAPEX / OPEX / fuel cost / LCOE); stacked cost composition bar (CAPEX + OPEX + fuel per technology); system-level cost insight cards |
| **F — Performance & Emissions** | Capacity factor bars; average diurnal generation profile; CO₂ emissions by technology; resource utilisation chart (dispatched vs available potential); monthly capacity factor heatmap grid with CSV export |

---

## Key Features

**Multi-scenario comparison via dropdown**
Three JSON files, one per objective. Switching scenarios replaces all charts, KPIs, and insight cards without reloading the page. All remaining scenarios are pre-fetched silently in the background 800 ms after first load — the comparison panel appears automatically once two or more scenarios are available. A blurred overlay prevents interaction during a scenario fetch.

**Geographic map in header**
If the JSON includes a `geographic` block (name, latitude, longitude, optional max height), a Leaflet map is rendered in the top-right corner of the header and the project name updates to match. The map re-centres on scenario switch without reinitialising the Leaflet instance.

**Zero hardcoded technology names**
All charts are built from whatever technologies exist in the loaded JSON. Adding or removing a technology from the model adds or removes it from every chart automatically — no HTML or JS edits needed.

**Correct energy accounting**
All figures use the same definitions throughout the dashboard:

| Term | Definition |
|---|---|
| Gross demand | Raw PyPSA load = net demand × (1 + grid loss factor) |
| Net demand | Actual consumer demand, after stripping distribution losses |
| Grid loss | Gross demand − net demand (= GLF fraction) |
| Storage RTE loss | Total charge − total discharge (round-trip inefficiency) |
| VRE curtailment | Available potential − dispatched, Solar and Wind only |

The Sankey, energy balance bars, and KPIs all use the same derived values and are guaranteed to agree.

**Sankey energy balance**
The Sankey is fully balanced: Grid Bus inflow = Grid Bus outflow = gross demand exactly. Storage round-trip losses and grid distribution losses are merged into a single Losses sink node. Curtailment is restricted to Solar and Wind. Node y positions are explicitly computed to prevent clipping at any technology count, and chart height scales automatically with the number of generators.

**Switchable cost view**
Section E provides a dropdown to switch the cost breakdown chart between six views: total annualised cost, annualised CAPEX, overnight CAPEX (upfront), annual OPEX, annual fuel cost, and LCOE/LCOS. The chart title and sub-label update in sync.

**Hourly dispatch explorer**
Section B provides a full 8,760-hour dispatch explorer: select any technology (or all technologies) and any month, with optional week zoom. Single-technology mode renders a Chart.js line chart; all-technologies mode switches to a Plotly stacked area chart with a unified hover tooltip.

**Demand heatmap**
Section D includes a canvas-rendered 24 × 12 heatmap of average hourly gross demand by hour-of-day and month. The heatmap redraws on window resize and uses a two-tone colour gradient from deep blue (low) to teal (high).

**Monthly capacity factor grid**
Section F includes a colour-coded grid of monthly capacity factors per technology (red < 30%, blue 30–60%, green > 60%) with a CSV export button that downloads the full hourly dispatch for the current scenario.

**Chart download buttons**
Each panel has a hover-activated download button that saves the Chart.js chart as a PNG.

**Keyboard navigation**
Press **1–6** to jump to sections A–F. Use **← →** to switch between scenarios. A hint bar appears briefly on first load.

**Scroll reveal animations**
All section grids fade and slide into view as they enter the viewport, with staggered delays for sibling panels.

**Chart registry**
Every Chart.js instance is tracked in a registry (`_charts`). Before re-rendering on scenario switch, all existing instances are destroyed and Plotly is purged — preventing ghost traces and canvas reuse errors.

**Auto-detected operational parameters**

| Parameter | Method |
|---|---|
| Worst RE month | Month with lowest (RE generation / total generation) ratio |
| Best RE month | Month with highest RE share |
| Critical week | Sliding 168-hour window maximising fossil/backup generation (O(8,760) scan) |
| Peak / minimum demand | Scanned from `gross_demand_mw` across all 8,760 hourly rows |
| Cheapest / most expensive source | Scanned from `lcoe_summary` per-technology LCOE/LCOS |
| Dominant source | Highest `share_pct` in `energy_mix` |
| System load factor | Average gross demand ÷ peak gross demand |

**Automatic cache-busting**
Every JSON file is fetched with `?v=<timestamp>` so GitHub Pages always serves the latest version without browser caching issues.

---

## JSON Data Format

The dashboard consumes the JSON exported by `export_dashboard_json()` in the PyPSA-HiGHS notebook. Expected top-level keys:

```json
{
  "meta": {
    "objective": "Lowest LCOE",
    "currency": "€",
    "discount_rate": 0.08,
    "grid_loss_factor": 0.04,
    "technologies": { "generation": [...], "storage": [...], "balancing": [...] }
  },
  "geographic": {
    "name": "Pulau Contoh",
    "latitude": -8.3405,
    "longitude": 115.0920,
    "max_height_m": 2924
  },
  "capacities": {
    "<tech>": {
      "type": "generator",
      "capacity_mw": 120.5,
      "potential_mw": 250.0
    },
    "<storage>": {
      "type": "storage",
      "power_mw": 30.0,
      "energy_mwh": 120.0,
      "max_hours": 4
    }
  },
  "energy_mix": {
    "<tech>": {
      "annual_gwh": 95.2, "share_pct": 48.1,
      "lcoe_per_mwh": 62.4, "annualised_cost": 4820000,
      "co2_tco2": 0.0, "curtailment_gwh": 8.3
    },
    "<storage>": {
      "annual_gwh": 11.8, "share_pct": 5.9,
      "discharge_gwh": 11.8, "charge_gwh": 15.5,
      "rte": 0.762, "lcos_per_mwh": 118.0, "annualised_cost": 1390000
    }
  },
  "lcoe_summary": {
    "<tech>": {
      "lcoe_per_mwh": 62.4,
      "annualised_cost": 4820000,
      "annualised_capex": 3100000,
      "total_capex": 38750000,
      "annual_opex": 720000,
      "annual_fuel_cost": 0
    },
    "_system": {
      "system_lcoe_per_mwh": 78.2,
      "total_annualised_cost": 15600000,
      "total_demand_mwh": 206200,
      "total_annualised_capex": 9800000,
      "total_annual_opex": 2100000,
      "total_annual_fuel": 0,
      "total_capex": 122500000
    }
  },
  "co2_summary": {
    "<tech>": { "annual_tco2": 0.0 },
    "_system": { "total_tco2": 0.0, "emission_intensity_gco2_per_kwh": 0.0 }
  },
  "hourly": [
    {
      "hour": 0,
      "gross_demand_mw": 23.5,
      "net_demand_mw": 22.6,
      "gen_<tech>_mw": 18.2,
      "dis_<storage>_mw": 4.1,
      "chg_<storage>_mw": 0.0,
      "soc_<storage>_mwh": 88.4
    }
  ]
}
```

**Optional but recommended fields:**

| Field | Used by |
|---|---|
| `geographic` | Header Leaflet map, project name |
| `capacities[tech].potential_mw` | Section A — Potential vs Installed chart |
| `lcoe_summary[tech].annualised_capex` | Section E cost views, storage table, CAPEX KPI |
| `lcoe_summary[tech].total_capex` | Section E overnight CAPEX view, system narrative, auto-insights |
| `lcoe_summary[tech].annual_opex` | Section E OPEX view, storage table |
| `lcoe_summary[tech].annual_fuel_cost` | Section E fuel cost view, stacked cost bar |
| `lcoe_summary._system.total_capex` etc. | Section E system cost insight cards |

The dashboard also accepts the older `export_json()` columnar `dispatch` format for backwards compatibility. Key differences from the old format are the use of lowercase snake_case keys, the split `gross_demand_mw` / `net_demand_mw` fields, `curtailment_gwh` per VRE technology, and the extended cost decomposition fields in `lcoe_summary`.

---

## Running Locally

```bash
git clone https://github.com/agus-sam/energy_island_dashboard.git
cd energy_island_dashboard
python -m http.server 8000
```

Then open [http://localhost:8000](http://localhost:8000) in your browser.

> **Note:** Opening `index.html` directly via `file://` will fail due to browser CORS restrictions on `fetch()`. A local server is required.

---

## Repository Structure

```
energy_island_dashboard/
│
├── index.html               # Dashboard — single self-contained file
├── data_lcoe.json           # Scenario: Lowest LCOE
├── data_co2.json            # Scenario: Lowest CO₂
├── data_diversified.json    # Scenario: Most Diversified
└── README.md
```

---

## Dependencies

All loaded from CDN — no installation required.

| Library | Version | Purpose |
|---|---|---|
| [Chart.js](https://www.chartjs.org/) | 4.4.1 | Bar, line, donut, LDC, and dispatch charts |
| [Plotly.js](https://plotly.com/javascript/) | 2.27.0 | Sankey energy flow diagram; all-technologies stacked area dispatch |
| [Leaflet](https://leafletjs.com/) | 1.9.4 | Geographic location map in header |
| [DM Sans / DM Mono / Fraunces](https://fonts.google.com/) | — | Typography |

---

## Browser Support

Works in all modern browsers (Chrome, Firefox, Edge, Safari). Requires JavaScript enabled.

---

## Related Repository

The optimisation model that produces the scenario JSON files:
**[Energy Island PyPSA-HiGHS](https://github.com/agus-sam/energy-island-pypsa-highs)** — LP capacity-expansion and dispatch model built with PyPSA and solved with HiGHS.

---

## Disclaimer

Developed for educational and research purposes. Results should not be used as the sole basis for investment or planning decisions.

---

*Author: Agus Samsudin — Energy Systems Modelling · Optimisation · Renewable Energy*
