# 🎯 Succession Planning Engine

A Streamlit-based internal HR tool for succession planning, powered by the **Korn Ferry 72-Dimension Leadership Framework** and live LPS (Leadership Potential Score) computation.

---

## 📁 Repository Structure

```
succession-planning-engine/
│
├── app.py                          # Main Streamlit application
├── requirements.txt                # Python dependencies
├── README.md                       # This file
│
├── .streamlit/
│   └── config.toml                 # Theme & server settings
│
├── data/                           # Sample / reference datasets
│   ├── employees_master_v2.csv     # 118-employee HRIS + KF scores (wide format)
│   ├── kf_competencies_detail.csv  # Long format: 118 × 72 KF dimension scores
│   ├── kf_competencies_reference.csv # KF framework reference + behavioural descriptors
│   ├── org_structure.csv           # Org hierarchy with 12 critical roles
│   ├── promotion_history.csv       # Individual promotion records
│   └── succession_pools.csv        # Pre-computed 5–10 successors per role
│
└── docs/
    └── app_update_notes.md         # Dataset v2 change log & app.py integration guide
```

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone <your-repo-url>
cd succession-planning-engine
```

### 2. Create a virtual environment (recommended)
```bash
python -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate           # Windows
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Run the app
```bash
streamlit run app.py
```

The app opens at `http://localhost:8501`

---

## 📂 Uploading Data

The app uses **file uploaders** in the sidebar. Upload these 6 CSVs on startup:

| Uploader label | File to upload | Required? |
|---|---|---|
| `employees_master.csv` | `employees_master_v2.csv` | ✅ Required |
| `kf_kfalp_detail.csv` | *(KF KFALP detail file)* | Optional |
| `kf_viaedge_detail.csv` | *(KF viaEdge detail file)* | Optional |
| `kf_attribute_reference.csv` | `kf_competencies_reference.csv` | Optional |
| `promotion_history.csv` | `promotion_history.csv` | Optional |
| `org_structure.csv` | `org_structure.csv` | Optional |

> Only `employees_master.csv` is required to activate the engine. All other uploads enable additional tabs and features.

---

## 🧠 Korn Ferry Framework

The engine uses the **KF 72-Dimension Leadership Framework** across 13 categories:

| Category | Dimensions | Scale |
|---|---|---|
| Strategic Thinking | 3 | Ordinal 1–4 |
| Operational Excellence | 3 | Ordinal 1–4 |
| Decision Effectiveness | 3 | Ordinal 1–4 |
| People Leadership | 3 | Ordinal 1–4 |
| Leading Change | 3 | Ordinal 1–4 |
| Stakeholder Engagement | 3 | Ordinal 1–4 |
| Individual Success Profile | 20 | 1–100 |
| Agreeableness | 3 | 1–100 |
| Agility | 5 | 1–10 |
| Presence | 5 | 1–100 |
| Learning Agility | 5 | 1–10 |
| Drivers | 5 | 1–10 |
| Risk Factors | 11 | 1–10 (inverse) |

---

## 📊 LPS — Leadership Potential Score

LPS (0–100) is a weighted composite of 5 clusters, configurable via sidebar sliders:

| Cluster | Default Weight | What it measures |
|---|---|---|
| C1 — Performance | 25% | 3yr avg rating, last rating, trajectory |
| C2 — KF Assessment | 30% | KF Blended (KFALP + viaEdge) |
| C3 — Career Velocity | 20% | Promotion speed (career + last 5yr) |
| C4 — Leadership Breadth | 15% | Cross-functional, international, project scope |
| C5 — Readiness | 10% | Grade proximity, mobility, flight risk |

### Readiness Bands

| Band | LPS Range | Colour | Meaning |
|---|---|---|---|
| Band 4 — Ready Now | ≥ 80 | 🟢 Dark Green | Can step in immediately |
| Band 3 — Ready in 1–2 Years | 65–79 | 🟩 Light Green | Near-ready, short development |
| Band 2 — Ready in 2–3 Years | 50–64 | 🟡 Amber | Developing, medium horizon |
| Band 1 — Not Ready | < 50 | 🔴 Red | Not yet in succession window |

---

## 🏢 Critical Roles (12)

`CEO · COO · CFO · CHRO · CIO/CTO · CSO (Sales) · CISO · Chief Strategy Officer · Business Unit Head · Geography Head · Vertical Head · Chief Marketing Officer`

Each role has a **succession pool of 5–10 ranked candidates** generated live from LPS scores within the eligible grade window.

---

## ⚙️ Python Version

Python **3.9+** recommended. Tested on 3.10 and 3.11.

---

## 📄 Licence

Internal use only — LTM Succession Planning Engine v2.
