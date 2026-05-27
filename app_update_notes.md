# Succession Planning Engine — Dataset v2 Update Notes
**Generated:** 2026-05-26  
**Framework:** Korn Ferry 72-Dimension Leadership Framework  
**Employee Pool:** 118 employees (HRIS synchronized)

---

## 1. What Changed

### A. Korn Ferry Framework (NEW)
The previous framework has been replaced with the full Korn Ferry 72-dimension model across 13 categories. All dimensions, scale types, and scoring logic are defined in `kf_competencies_reference.csv`.

| Category | Dims | Scale |
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
| **TOTAL** | **72** | mixed |

> Note: The spec referenced "61 dimensions" at the top-level; the full sub-dimension expansion yields 72 scored items. All 72 are included.

### B. Critical Roles (Updated to 12)
`CEO, COO, CFO, CHRO, CIO/CTO, CSO (Sales), CISO, Chief Strategy Officer, Business Unit Head, Geography Head, Vertical Head, Chief Marketing Officer`

### C. Succession Pool Constraints
- Each role: **5–10 eligible candidates** (enforced)
- Grade eligibility windows:
  - C-Suite roles: M5, E1, E2 only
  - BU Head / Geo Head / Vertical Head: M4, M5, E1
- Minimum performance threshold: ≥ 3.0 (1–5 scale)

---

## 2. Output Files

| File | Rows | Columns | Description |
|---|---|---|---|
| `employees_master_v2.csv` | 118 | 91 | HRIS + all 72 KF dimension scores (wide format) |
| `kf_competencies_detail.csv` | 8,496 | 9 | Long format: 118 × 72 scored records |
| `kf_competencies_reference.csv` | 72 | 8 | Dimension metadata + behavioural descriptors + assessment methods |
| `org_structure.csv` | 16 | 11 | Org chart with 12 critical roles + management layers |
| `promotion_history.csv` | 292 | 10 | Individual promotion records derived from HRIS data |
| `succession_pools.csv` | 93 | 10 | 5–10 ranked successors per critical role |

---

## 3. app.py Update Requirements

### 3.1 Data Loading
```python
# Replace old employee load with:
emp_df = pd.read_csv("data/employees_master_v2.csv")

# Load KF reference for dimension metadata
kf_ref = pd.read_csv("data/kf_competencies_reference.csv")

# Load long-format competency detail
kf_detail = pd.read_csv("data/kf_competencies_detail.csv")

# Load pre-computed succession pools
pool_df = pd.read_csv("data/succession_pools.csv")
```

### 3.2 KF Column Naming Convention
All 72 KF dimension columns in `employees_master_v2.csv` follow this pattern:
```
KF_{Category_snake_case}__{Dimension_snake_case}
```
Examples:
- `KF_Strategic_Thinking__Visionary_Thinking`
- `KF_Individual_Success_Profile__Achievement_Orientation`
- `KF_Risk_Factors__Volatility_Emotional_Instability`

To extract KF columns programmatically:
```python
kf_cols = [c for c in emp_df.columns if c.startswith("KF_")]
ordinal_cols = kf_ref[kf_ref["Scale_Type"]=="ordinal_4"]["KF_Dimension"].tolist()
scale100_cols = kf_ref[kf_ref["Scale_Type"]=="scale_100"]["KF_Dimension"].tolist()
risk_cols = kf_ref[kf_ref["Scale_Type"]=="scale_10_inverse"]["KF_Dimension"].tolist()
```

### 3.3 Scoring Logic (Normalization for composite scores)
```python
def normalize_kf_score(score, scale_type):
    """Normalize any KF dimension to 0–1 for composite scoring."""
    if scale_type == "ordinal_4":
        return (score - 1) / 3.0
    elif scale_type == "scale_100":
        return (score - 1) / 99.0
    elif scale_type == "scale_10":
        return (score - 1) / 9.0
    elif scale_type == "scale_10_inverse":
        return 1.0 - (score - 1) / 9.0  # Invert: lower risk = higher score
```

### 3.4 Succession Pool Generation
The pre-computed `succession_pools.csv` can be used directly. If real-time recomputation is needed:
```python
def compute_succession_pool(role, emp_df, min_pool=5, max_pool=10):
    C_SUITE = ["CEO","COO","CFO","CHRO","CIO/CTO","CSO (Sales)","CISO",
               "Chief Strategy Officer","Chief Marketing Officer"]
    MID = ["Business Unit Head","Geography Head","Vertical Head"]
    
    grade_map = {"M1":1,"M2":2,"M3":3,"M4":4,"M5":5,"E1":6,"E2":7}
    
    eligible = emp_df.copy()
    if role in C_SUITE:
        eligible = eligible[eligible["Grade"].map(grade_map) >= 5]
    elif role in MID:
        eligible = eligible[eligible["Grade"].map(grade_map) >= 4]
    
    eligible = eligible[eligible["Performance_Rating_Last_Year"] >= 3.0]
    
    # Score candidates (customize weights as needed)
    eligible["pool_score"] = (
        eligible["Performance_Rating_Last_Year"] * 18 +
        eligible["Grade"].map(grade_map) * 4
    )
    
    return eligible.nlargest(max_pool, "pool_score").head(max_pool)
```

### 3.5 Behavioural Descriptors Display
```python
def get_descriptor(dimension, score, kf_ref_df):
    row = kf_ref_df[kf_ref_df["KF_Dimension"] == dimension]
    if row.empty: return ""
    desc_text = row.iloc[0]["Behavioural_Descriptors"]
    # Parse: "[1] text; [2] text; ..." format
    import re
    parts = re.findall(r'\[(\d)\]\s([^;]+)', desc_text)
    desc_dict = {int(k): v.strip() for k, v in parts}
    return desc_dict.get(int(score), "")
```

### 3.6 Risk Factor Handling
Risk factors are **inverse** — lower is better. Always invert when displaying or sorting:
```python
risk_dims = kf_ref[kf_ref["Scale_Type"]=="scale_10_inverse"]["KF_Dimension"].tolist()

# Display label for risk scores
def risk_label(score):
    if score <= 3: return "Low Risk"
    elif score <= 6: return "Moderate Risk"
    elif score <= 8: return "Elevated Risk"
    else: return "Critical Concern"
```

### 3.7 Grade Windows by Role (Updated)
```python
GRADE_ELIGIBILITY = {
    # C-Suite
    "CEO": ["M5","E1","E2"],
    "COO": ["M5","E1","E2"],
    "CFO": ["M5","E1","E2"],
    "CHRO": ["M5","E1","E2"],
    "CIO/CTO": ["M5","E1","E2"],
    "CSO (Sales)": ["M5","E1","E2"],
    "CISO": ["M5","E1","E2"],
    "Chief Strategy Officer": ["M5","E1","E2"],
    "Chief Marketing Officer": ["M5","E1","E2"],
    # Leadership roles
    "Business Unit Head": ["M4","M5","E1"],
    "Geography Head": ["M4","M5","E1"],
    "Vertical Head": ["M4","M5","E1"],
}
```

---

## 4. Score Distribution Reference

| Scale Type | Expected Mean | Std Dev | Notes |
|---|---|---|---|
| Ordinal 1–4 | ~2.4 | ~0.8 | Correlated with perf + grade |
| Scale 1–100 | ~52 | ~13 | Midpoint 50; ±15–25 variance |
| Scale 1–10 | ~5.9 | ~1.4 | Midpoint ~5.5; perf-correlated |
| Risk Factors (inverse) | ~2.8 | ~1.2 | Lower = better; perf-inverse |

---

## 5. Department → KF Category Affinity Map

Employees in these departments receive a boost in the listed KF categories:

| Department | High-Affinity KF Categories |
|---|---|
| Strategy & Corporate Development | Strategic Thinking, Decision Effectiveness, Stakeholder Engagement |
| Finance & Accounting | Operational Excellence, Decision Effectiveness, Risk Factors |
| Technology & Engineering | Operational Excellence, Agility, Learning Agility |
| Human Resources | People Leadership, Agreeableness, Presence |
| Sales & Business Development | Stakeholder Engagement, Drivers, Presence |
| Operations | Operational Excellence, Decision Effectiveness, Drivers |
| Marketing & Brand | Stakeholder Engagement, Presence, Individual Success Profile |
| Legal & Compliance | Decision Effectiveness, Risk Factors, Stakeholder Engagement |
| Risk & Audit | Operational Excellence, Decision Effectiveness, Risk Factors |
| Information Security | Operational Excellence, Agility, Risk Factors |
| Product Management | Strategic Thinking, Agility, Learning Agility |
| Supply Chain | Operational Excellence, Decision Effectiveness, Drivers |
