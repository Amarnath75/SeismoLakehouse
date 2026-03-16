# 🌍 SeismoLakehouse

> An end-to-end, production-grade Data Engineering pipeline built on **Databricks Free Edition** — ingesting real-time earthquake data from the USGS API, processing it through the Medallion Architecture, and visualizing insights on an interactive dashboard.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)
- [Tech Stack](#-tech-stack)
- [Pipeline Layers](#-pipeline-layers)
- [Dashboard](#-dashboard)
- [CI/CD with GitHub Actions](#-cicd-with-github-actions)
- [Getting Started](#-getting-started)
- [Configuration](#-configuration)
- [Environments](#-environments)
- [Security](#-security)
- [Contributing](#-contributing)

---

## 🔭 Overview

**SeismoLakehouse** is a fully automated, governed, and version-controlled data engineering project that:

- 📡 Fetches **real-time earthquake data** from the [USGS Earthquake API](https://earthquake.usgs.gov/fdsnws/event/1/)
- 🏗️ Processes data through the **Medallion Architecture** (Bronze → Silver → Gold)
- 📊 Visualizes insights on a **Databricks SQL Dashboard** — auto-refreshed after every pipeline run
- 🔁 Orchestrates the full pipeline using **Databricks Jobs**
- 🚀 Deploys using **Databricks Asset Bundles (DAB)** and **GitHub Actions**

> Built entirely on **Databricks Free Edition** — zero cost, 100% production mindset.

---

## 🏛️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA FLOW                                │
│                                                                 │
│   USGS REST API                                                 │
│        │                                                        │
│        ▼                                                        │
│  ┌──────────────┐                                               │
│  │ BRONZE LAYER │  Python Notebook → Raw JSON → Databricks     │
│  │              │  Volume (Serverless Compute)                  │
│  └──────┬───────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │ SILVER LAYER │  Auto Loader + Delta Live Tables →            │
│  │              │  Cleaned Delta Table                          │
│  └──────┬───────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │  GOLD LAYER  │  Databricks SQL Dashboard →                   │
│  │              │  Seismo_EarthQuake_Dashboard                  │
│  └──────────────┘                                               │
│                                                                 │
│         Governed by Unity Catalog                               │
│         Deployed via DAB + GitHub Actions                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
SeismoLakehouse/
├── SeismoLakehouse_bundle/
│   ├── resources/
│   │   ├── Bronze_Silver_earthquake.pipeline.yml   # DLT Pipeline definition
│   │   ├── EarthQuake_Dashboard.yml                # Dashboard resource
│   │   └── Earthquake_end_end.job.yml              # Job orchestration
│   ├── src/
│   │   ├── Notebooks/
│   │   │   ├── Ingestion_Bronze.ipynb              # Bronze ingestion notebook
│   │   │   └── Bronze_Silver/
│   │   │       └── transformations/                # DLT transformation notebooks
│   │   └── dashboard/
│   │       └── Seismo_EarthQuake_Dashboard.lvdash.json
│   ├── databricks.yml                              # Main DAB configuration
│   ├── pyproject.toml
│   └── .gitignore
├── .github/
│   └── workflows/
│       └── deploy.yml                              # GitHub Actions CI/CD
└── README.md
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Data Source** | USGS Earthquake REST API |
| **Ingestion** | Python (requests), Databricks Notebooks |
| **Raw Storage** | Databricks Volumes (Bronze) |
| **Incremental Load** | Auto Loader (`cloudFiles`) |
| **Transformation** | Delta Live Tables (DLT) |
| **Storage Format** | Delta Lake / Delta Tables |
| **Visualization** | Databricks SQL Dashboard |
| **Governance** | Unity Catalog |
| **Orchestration** | Databricks Jobs |
| **IaC / Deployment** | Databricks Asset Bundles (DAB) |
| **CI/CD** | GitHub Actions |
| **Security** | Service Principal (SPN) |
| **Compute** | Serverless |

---

## 🔄 Pipeline Layers

### 🥉 Bronze Layer — Raw Ingestion
- **Task:** `Bronze_layer`
- **Notebook:** `src/Notebooks/Ingestion_Bronze.ipynb`
- **What it does:**
  - Calls the USGS Earthquake REST API
  - Stores raw JSON response into a **Databricks Volume**
  - No transformations — raw data preserved as-is
- **Compute:** Serverless

### 🥈 Silver Layer — Cleaning & Transformation
- **Task:** `Silver_layer`
- **Pipeline:** `Bronze_Silver_EarthQuake` (Delta Live Tables)
- **What it does:**
  - **Auto Loader** monitors the Volume for new JSON files
  - Parses nested JSON fields (geometry, properties)
  - Casts data types, handles nulls, standardizes timestamps
  - Writes clean data as a managed **Delta Table**
- **Features:** Photon enabled, Serverless, Unity Catalog governed

### 🥇 Gold Layer — Visualization
- **Task:** `Gold_layer`
- **Dashboard:** `Seismo_EarthQuake_Dashboard`
- **What it does:**
  - Auto-refreshes the SQL Dashboard after every pipeline run
  - Serves insights to end users via Databricks SQL

---

## 📊 Dashboard

The **Seismo_EarthQuake_Dashboard** provides:

| Widget | Description |
|---|---|
| 🗺️ **World Map** | Earthquake epicenters plotted by lat/long |
| 🍩 **Magnitude Chart** | Distribution: Micro (66.2%) \| Minor (22.2%) \| Light (7.4%) \| Moderate+ |
| 🌎 **Countries Chart** | Earthquake frequency by country (CA, Alaska, Japan, Chile, Argentina...) |
| 📋 **Event Table** | Paginated list with magnitude, location, timestamp & USGS source links |

---

## 🚀 CI/CD with GitHub Actions

The project uses a **branch-based CI/CD strategy**:

```
Develop branch  →  GitHub Actions  →  databricks bundle deploy --target dev
main branch     →  GitHub Actions  →  databricks bundle deploy --target prod
```

### Workflow File: `.github/workflows/deploy.yml`

```yaml
name: Deploy SeismoLakehouse Bundle

on:
  push:
    branches:
      - Develop
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - run: pip install databricks-cli

      - name: Deploy to DEV
        if: github.ref == 'refs/heads/Develop'
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
        run: databricks bundle deploy --target dev

      - name: Deploy to PROD
        if: github.ref == 'refs/heads/main'
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
        run: databricks bundle deploy --target prod
```

### Required GitHub Secrets

| Secret | Description |
|---|---|
| `DATABRICKS_HOST` | Databricks workspace URL |
| `DATABRICKS_TOKEN` | Databricks Personal Access Token |

---

## 🏁 Getting Started

### Prerequisites

- Databricks Free Edition account
- GitHub account
- Unity Catalog enabled workspace
- SQL Warehouse created

### 1. Clone the Repository

```bash
git clone https://github.com/Amarnath75/SeismoLakehouse.git
cd SeismoLakehouse
```

### 2. Install Databricks CLI

```bash
pip install databricks-cli
databricks configure --token
```

### 3. Deploy to Dev

```bash
cd SeismoLakehouse_bundle
databricks bundle deploy --target dev
```

### 4. Run the Pipeline

```bash
databricks bundle run SeismoLakehouse_End_to_End_pipeline --target dev
```

---

## ⚙️ Configuration

The `databricks.yml` defines all variables and targets:

```yaml
bundle:
  name: SeismoLakehouse_bundle

include:
  - resources/Bronze_Silver_earthquake.pipeline.yml
  - resources/EarthQuake_Dashboard.yml
  - resources/Earthquake_end_end.job.yml

variables:
  catalog:
    description: The catalog to use
  schema:
    description: The schema to use
  warehouse_id:
    description: WarehouseID

targets:
  dev:
    mode: development
    default: true
    variables:
      catalog: dev_catalog
      warehouse_id: <your_dev_warehouse_id>

  prod:
    mode: production
    variables:
      catalog: prod_catalog
      warehouse_id: <your_prod_warehouse_id>
```

---

## 🌐 Environments

| Environment | Branch | Target | Prefix |
|---|---|---|---|
| Development | `Develop` | `dev` | `[dev username]` |
| Production | `main` | `prod` | No prefix |

---

## 🔐 Security

- **Service Principal (SPN)** is used for all automated job executions — no personal credentials in production
- **Unity Catalog** manages all data access with fine-grained permissions
- **GitHub Secrets** store all sensitive credentials for CI/CD
- **Least-privilege access** — SPN granted only required permissions on catalogs, volumes, and warehouses

---

## 🤝 Contributing

1. Create a feature branch from `Develop`
2. Make your changes
3. Push to `Develop` → auto deploys to `dev` for testing
4. Open a Pull Request to `main`
5. After review & merge → auto deploys to `prod`

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 🙌 Acknowledgements

- [USGS Earthquake API](https://earthquake.usgs.gov/fdsnws/event/1/) for providing free real-time seismic data
- [Databricks Free Edition](https://www.databricks.com/try-databricks) for the full Lakehouse platform at zero cost
- Tutorial reference: [End-to-End Databricks Data Engineering Project](https://www.youtube.com/watch?v=xkrUAcRcpjc)

---

<p align="center">Built with ❤️ using Databricks Lakehouse Platform</p>
