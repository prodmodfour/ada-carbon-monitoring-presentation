---
theme: default
title: Ada Carbon Monitoring
info: Carbon monitoring implementation for Ada service
author: Ashraf
class: text-white
drawings:
  persist: false
transition: fade
---

<style>
:root {
  --slidev-theme-default-background: #000000;
  --slidev-theme-default-text: #ffffff;
}
.slidev-layout {
  background: #000000 !important;
  color: #ffffff !important;
}
h1, h2, h3, h4, h5, h6 {
  color: #ffffff !important;
}
li {
  color: #ffffff !important;
}
p {
  color: #ffffff !important;
}
code {
  background: #1a1a1a !important;
}
table {
  color: #ffffff !important;
}
th, td {
  border-color: #333333 !important;
}
</style>

# Ada Carbon Monitoring

Handoff Presentation


---




# Architecture Overview

<img src="/system.png" class="mx-auto" style="max-height: 420px;" />

---



# Branch Structure

**ada-ui:**
- `carbon-labs-platform` - Production, real data
- `carbon-labs-demo` - Demo mode with fake data banner

**ada-carbon-monitoring-api:**
- `carbon-labs-platform` - Real Prometheus/MongoDB
- `carbon-labs-demo` - Fake data enabled

---

# Calculation Pipeline

```
CPU Seconds (Prometheus)
    |
    v
Electricity (kWh)
    Formula: (12W x busy_seconds + 1W x idle_seconds) / 3,600,000
    |
    v
Carbon (gCO2eq)
    Formula: kWh x carbon_intensity (from UK Grid API)
    |
    v
Equivalencies
    Example: miles driven, smartphone charges, tree-days
```

---

# Backend API Endpoints

**Carbon Intensity:**
- `GET /carbon/intensity/current` - Current UK grid intensity
- `GET /carbon/intensity/forecast` - 48-hour forecast

**Calculations:**
- `POST /carbon/calculate` - Calculate from CPU seconds
- `GET /carbon/electricity/estimate` - Electricity only
- `GET /carbon/equivalencies/{gco2eq}` - Real-world equivalencies

---

# Backend API Endpoints (continued)

**User Attribution:**
- `GET /users/me/summary` - Current user's carbon summary
- `GET /users/me/usage` - User's workspace breakdown
- `POST /workspaces/track` - Track workspaces for a user


---

# Frontend Components

| Component | Purpose |
|-----------|---------|
| CarbonDashboard | Main dashboard with all views |
| CarbonIntensityForecast | Line chart with best window |
| CarbonStackedBarChart | Busy/idle breakdown |
| CarbonHeatmap | GitHub-style year heatmap |
| CarbonEquivalencies | Real-world comparisons |
| CarbonBadge | Compact badge for workspace cards |

---

# Carbon Intensity Forecast

Shows 48-hour forecast from UK Carbon Intensity API with recommended low-carbon windows.

<img src="/forecast.png" class="mx-auto" style="max-height: 380px;" />

---

# Usage Breakdown Chart

Stacked bar chart showing idle vs busy electricity/carbon usage by day, month, or year.

<img src="/usage_bar_chart.png" class="mx-auto" style="max-height: 380px;" />

---

# Carbon Heatmap

GitHub-style year view showing daily carbon emissions intensity.

<img src="/heatmap.png" class="mx-auto" style="max-height: 300px;" />

---

# Workspace Breakdown

Per-workspace energy and carbon attribution.

<img src="/workspace_breakdown.png" class="mx-auto" style="max-height: 350px;" />

---

# Carbon Equivalencies

Real-world comparisons to make carbon footprint tangible.

<img src="/carbon_equivalencies.png" class="mx-auto" style="max-height: 380px;" />

---

# Workspace Carbon Badge

Carbon badge integrated into workspace cards for at-a-glance emissions.

<img src="/workspaces.png" class="mx-auto" style="max-height: 420px;" />

---

# CarbonDashboard

<div class="flex gap-4">
<div class="flex-1">

Located in Labs sidebar (ISIS, CLF platforms)

Features:
- Summary cards (energy, carbon, workspaces, CPU time)
- Carbon intensity forecast with best 3-hour window
- Stacked bar chart (day/month/year views)
- Year heatmap
- Workspace breakdown table

</div>
<div>
<img src="/sidebar.png" style="height: 200px;" />
</div>
</div>

<img src="/dashboard.png" class="mt-4" style="max-height: 180px;" />

---

# User Attribution

How it works:
1. Get user's federal ID 
2. Query MongoDB for workspaces where owner matches
3. Get hostnames from workspace data
4. Query Prometheus for CPU metrics by hostname
5. Calculate electricity and carbon
6. Return aggregated results


---

# Data Sources

**Prometheus:**
- Metric: `node_cpu_seconds_total`
- Labels: instance, mode, cpu
- Note: Ignore data before March 2025 (label changes)

**MongoDB:**
- Collections: workspaces, groups
- Via ada-db-interface API

**UK Carbon Intensity API:**
- URL: https://carbonintensity.org.uk/
- Updates every 30 minutes

---

# Configuration

File: `ada-carbon-monitoring-api.ini`

```ini
[POWER]
busy_power_w = 12.0
idle_power_w = 1.0

[FAKE_DATA]
use_fake_prometheus = false
use_fake_mongodb = false
```




---


# Repositories
- github.com/prodmodfour/ada-carbon-monitoring-api
- github.com/prodmodfour/ada-api
- https://github.com/prodmodfour/ada-db-interface
- github.com/ral-facilities/ada-ui
- github.com/prodmodfour/Ada_Carbon_Monitoring_Implementation_Documentation
