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

Handover Presentation

January 2026

---

# Project Goal

- Track carbon footprint of Ada workspaces
- Attribute carbon usage to users and groups
- Visualize carbon data in the Ada UI
- Enable time-shifting to low-carbon periods

---

# What Was Built

- Backend API for carbon calculations
- Prometheus integration for CPU metrics
- UK Grid carbon intensity integration
- User attribution via MongoDB workspace ownership
- Group attribution via cloud project + machine type
- Svelte UI components for visualization

---

# Architecture Overview

```
ada-ui (Svelte)
    |
    v
ada-api (FastAPI proxy)
    |
    v
ada-carbon-monitoring-api (FastAPI)
    |
    +---> Prometheus (CPU metrics)
    +---> MongoDB via ada-db-interface (workspace data)
    +---> UK Carbon Intensity API (grid carbon)
```

---

# Repositories

| Repository | Purpose |
|------------|---------|
| ada-carbon-monitoring-api | Carbon calculations, API |
| ada-api | Proxy routes to carbon API |
| ada-ui | Svelte frontend components |
| ada-db-interface | MongoDB workspace queries |

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

**Group Attribution:**
- `GET /groups` - List all groups (cloud_project + machine)
- `GET /groups/{name}/usage` - Group carbon usage
- `GET /groups/{name}/summary` - Group summary

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

# CarbonDashboard

Located in Labs sidebar (ISIS, CLF platforms)

Features:
- Summary cards (energy, carbon, workspaces, CPU time)
- Carbon intensity forecast with best 3-hour window
- Stacked bar chart (day/month/year views)
- Year heatmap
- Workspace breakdown table

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

# Group Attribution

Groups = cloud_project + machine_name

Examples:
- IDAaaS_Muon

API returns all workspaces in a group with combined carbon footprint.

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
[API]
prometheus_url = https://host-172-16-100-248.nubes.stfc.ac.uk
ada_db_interface_url = http://localhost:5002

[POWER]
busy_power_w = 12.0
idle_power_w = 1.0

[FAKE_DATA]
use_fake_prometheus = false
use_fake_mongodb = false
```

---

# Running the Services

**ada-carbon-monitoring-api:**
```bash
cd ada-carbon-monitoring-api
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000
```

**ada-api:**
```bash
cd ada-api
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 5001
```

Set `CARBON_API_URL=http://localhost:8000` in ada-api config.

---

# Demo Mode

To run with fake data:

1. Switch to `carbon-labs-demo` branch in both repos
2. Set in `ada-carbon-monitoring-api.ini`:
```ini
[FAKE_DATA]
use_fake_prometheus = true
use_fake_mongodb = true
```
3. UI shows orange "Demo Mode" banner

---



# Documentation

Updated documentation at:
`Ada_Carbon_Monitoring_Implementation_Documentation/`

Key pages:
- Quickstart - Getting started
- API Reference - All API endpoints
- Backend 
- Frontend - Svelte components

---

# Key Files

**ada-carbon-monitoring-api:**
- `src/calculators/electricity_estimator.py`
- `src/calculators/carbon_calculator.py`
- `src/models/workspace_tracker.py`
- `src/api/carbon/` - All route handlers

**ada-ui:**
- `src/components/Carbon/` - All Svelte components
- `src/api/carbon.js` - API client

---

# Next Steps

For the Ada team:

1. Review the carbon-labs-platform branches
2. Merge into main when ready
3. Deploy ada-carbon-monitoring-api to production
4. Configure Prometheus recording rules for faster queries
5. Consider adding more equivalencies or visualizations

---

# Recording Rules

For faster Prometheus queries, add recording rules:

```yaml
groups:
  - name: carbon_aggregations
    interval: 5m
    rules:
      - record: carbon:cpu_busy_seconds:by_project
        expr: sum by (cloud_project_name) (
          node_cpu_seconds_total{mode!="idle"}
        )
```

See `prometheus-preprod/prometheus/recording_rules.yml`



---


# Repositories
- github.com/prodmodfour/ada-carbon-monitoring-api
- github.com/prodmodfour/ada-api
- github.com/ral-facilities/ada-ui
- github.com/prodmodfour/Ada_Carbon_Monitoring_Implementation_Documentation
