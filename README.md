# AI Dungeon Event Stream Analysis
**Data Engineering & Analytics Take-Home Submission**

---

## 📋 **Overview**
This repository contains an exploratory data analysis (EDA) of the AI Dungeon event stream dataset covering October 1-10, 2025 (~6.1 GB, 6.9M+ events). 

The analysis addresses three core areas:
1. **Data Quality & Instrumentation** - Validation, anomaly detection, and schema recommendations
2. **First-Time User Experience** - Signup/retention metrics and activation patterns
3. **Content & UGC Marketplace** - Scenario performance and discovery surface optimization

---

## 🔧 **Technology Stack**

### **Why Polars?**
Due to the dataset size (~6.1 GB CSV), this analysis uses **Polars** instead of Pandas for:
- **Memory Efficiency**: Streaming ingestion prevents OOM errors on standard hardware
- **Performance**: 5-10x faster than Pandas for large-scale aggregations
- **Lazy Evaluation**: Optimized query planning reduces unnecessary computation

### **Dependencies**
```txt
polars==0.20.31
plotnine==0.13.6
jupyter==1.0.0
```

---

## 🚀 **Quick Start**

### **Prerequisites**
- Python 3.10+ (tested on Python 3.14)
- 8GB+ RAM recommended
- ~7 GB free disk space for dataset

### **1. Setup Environment**
```bash
# Create virtual environment
python -m venv venv

# Activate environment
# Mac/Linux:
source venv/bin/activate

# Windows:
venv\Scripts\activate

# Install dependencies
pip install polars plotnine jupyter
```

### **2. Download Dataset**
The dataset URL is embedded in the notebook. On first run, the notebook will:
- Check if `latitude_event_data.csv` exists locally
- If missing, download automatically using `wget`
- Save to project root directory

**Manual download (if needed):**
```bash
wget -O latitude_event_data.csv '<DATASET_URL>'
```

### **3. Run Analysis**
```bash
# Launch Jupyter Notebook
jupyter notebook

# Open: analysis.ipynb
# Run all cells: Cell → Run All
```

**Expected Runtime**: ~5-8 minutes (includes data loading and all computations)

---

## 📁 **Project Structure**
```
├── analysis.ipynb              # Main analysis notebook (all code & findings)
├── README.md                   # This file
└── latitude_event_data.csv     # Dataset (auto-downloaded on first run)
```

---

## 📊 **Key Findings Summary**

### **Data Quality Issues Identified**
- **3,514 duplicate events** detected and removed (0.05% of dataset)
- **No missing values** in core fields (user_id, created_at, event_name)
- **No timestamp anomalies** - consistent 24-hour coverage across 10-day period
- **83% of users** have no clear traffic source attribution (direct/organic)

**Critical Gap**: User IDs are generated client-side, causing multi-device users to appear as separate users (deflates retention metrics)

### **First-Time User Metrics**

| Metric | Value | Industry Benchmark | Status |
|--------|-------|-------------------|--------|
| **Signup Rate** | 8.04% | 15-20% | ⚠️ Below target |
| **Day-1 Retention** | 4.92% | 30-40% | 🔴 Critical |
| **Avg Actions (Guests)** | 1.1 | - | 🔴 High bounce rate |
| **Avg Actions (Registered)** | 99.9 | - | ✅ Strong engagement |

**Key Insight**: Bimodal distribution reveals a **"Make-or-Break" first session** - users either bounce immediately or become power users.

### **Registration Drivers**

**What Works**:
- Users who visit **Config/Settings Screen**: **84% more likely to register**
- Users with **>5 first-session actions**: **73% conversion rate**
- Users exploring **Discover Tab**: **78.7% registration rate** (highest of any surface)

**What Doesn't Work**:
- Homepage Banner clicks: Only **26.3% registration rate**
- Zero-action sessions: **92% never return**

### **Content Performance**

**Top Scenarios by Engagement** (avg turns per player):
1. Scenario `1828345`: **85.8 turns/player** (139 unique players)
2. Scenario `11482379`: **57.0 turns/player** (84 unique players)
3. Scenario `11507521`: **54.5 turns/player** (99 unique players)

**Most Popular Scenarios** (traffic):
1. Scenario `cj90vvdB14fn`: **2,422 unique players** (0 turns - likely tutorial/onboarding)
2. Scenario `8748087`: **1,927 unique players** (21.3 avg turns)
3. Scenario `2503121`: **1,190 unique players** (40.4 avg turns)

**Discovery Surface Performance**:
- **Discover Tab**: 78.7% registration rate, 288.3 avg actions
- **Search**: 75.6% registration rate, 287.9 avg actions  
- **Homepage Banner**: 26.3% registration rate, 72.5 avg actions (needs optimization)
- **Direct/Other**: 77.1% registration rate, 475.3 avg actions (most engaged users)

---

## 🎯 **Actionable Recommendations**

### **Priority 1: Fix "Writer's Block" for New Users** 🚨
**Problem**: 92% of guests bounce after <2 actions - they hit friction immediately

**Proposed Solution**: Implement "Beginner-Friendly" Content Filter
```
A/B Test Design:
- Control: Show standard mixed feed
- Treatment: Filter to only show scenarios with:
  • Low retry rate (<10% globally)
  • High social proof (Top 5% by unique players)
  
Success Metric: % of users reaching >5 actions in first session
Target: Increase from current 8% to 15%
```

### **Priority 2: Promote Config Screen Discovery** ⚙️
**Data**: 84% of config screen visitors register, but only 6% of guests find it

**Implementation**:
- Add **tooltip overlay** on first session: "Customize your adventure →"
- Create **onboarding checklist** including "Explore Settings" step
- A/B test tooltip placement (game screen vs. homepage)

### **Priority 3: Fix Guest User Identity** 🔐
**Problem**: Multi-device usage inflates unique user count, deflates retention

**Solution**: Implement probabilistic identity stitching
```python
# Pseudocode
identity_key = hash(IP_address + User_Agent + temporal_pattern)
# Link user_ids with matching identity_key within 24-hour window
```

### **Priority 4: Optimize Homepage Banner** 📊
**Data**: Only 26.3% registration rate (vs. 78.7% for Discover)

**Hypothesis**: Generic banners don't match user intent

**Test**: 
- **Variant A**: Personalized banner based on search history
- **Variant B**: "Most Played This Week" social proof banner
- **Variant C**: "Quick Start" action-focused banner

---

## 📈 **Methodology**

### **Data Processing Pipeline**
```
1. Ingestion → Polars lazy CSV scan with streaming collection
2. Cleaning → Deduplicate on (user_id, created_at, event_name, metadata)
3. Feature Engineering → Extract scenario_id, calculate user-level aggregates
4. Segmentation → Compare verified vs. guest user populations
5. Surface Attribution → Parse metadata 'source' field for discovery analysis
```

### **Metrics Definitions**
- **Signup Rate**: `(verified_users / total_unique_user_ids) * 100`
- **Day-1 Retention**: `(users_with_event_on_day_N+1 / users_created_on_day_N) * 100`
- **Engagement Depth**: `avg(submit_button_pressed count per user)`
- **Surface Registration Rate**: `(verified_users_from_surface / total_users_from_surface) * 100`

### **Statistical Approach**
- **Comparative Segmentation**: Verified vs. Guest population analysis
- **Scenario-Level Aggregation**: Traffic × Engagement composite scoring
- **Surface Attribution**: Metadata parsing with fallback logic (source → event_name)

---

## ⚠️ **Limitations & Caveats**

1. **Temporal Coverage**: Analysis covers October 1-10, 2025 only (10-day snapshot)
   - **Impact**: Seasonal patterns, marketing campaigns, and growth trends not captured
   - **Mitigation**: Metrics should be validated on 30-90 day dataset before productionization

2. **Guest ID Inflation**: Multi-device users counted as separate user_ids
   - **Impact**: Signup and retention rates are **understated** (true rates likely 2-3x higher)
   - **Mitigation**: Implement identity stitching before using for exec reporting

3. **Session Logic Missing**: No native `session_id` field in schema
   - **Impact**: Cannot accurately measure session duration or session-level conversions
   - **Mitigation**: Reconstruct sessions using 30-minute timeout heuristic

4. **Metadata Sparsity**: Key fields (device, region, referrer) buried in JSON
   - **Impact**: Cannot segment by device type (iOS vs. Android vs. Web)
   - **Mitigation**: Extract metadata fields into typed columns (schema improvement)

5. **No Statistical Significance Testing**: 
   - **Impact**: Cannot determine if surface performance differences are statistically significant
   - **Mitigation**: Apply chi-square tests for categorical comparisons before A/B test launch

6. **Survivor Bias**: Dataset only includes users who generated ≥1 event
   - **Impact**: True bounce rate (users who visit but never trigger event) is unknown
   - **Mitigation**: Cross-reference with web analytics (Google Analytics) for full funnel

---

## 🔍 **Future Work**

### **Data Engineering Improvements**
- [ ] Extract metadata fields into typed columns (session_id, adventure_id, device_type, region)
- [ ] Implement session reconstruction algorithm (30-min inactivity timeout)
- [ ] Build identity graph for cross-device user stitching
- [ ] Add data validation pipeline (e.g., Great Expectations) for ongoing monitoring

### **Advanced Analytics**
- [ ] **Cohort Retention Curves**: Day 1, 7, 30 retention by signup date cohort
- [ ] **Content Recommendation Engine**: Collaborative filtering based on scenario completion patterns
- [ ] **Churn Prediction Model**: XGBoost classifier using engagement features
- [ ] **LTV Projection**: Revenue modeling for premium conversion funnel

### **Experimentation Framework**
- [ ] A/B test infrastructure for beginner content filter
- [ ] Multi-armed bandit for dynamic surface optimization
- [ ] Statistical power calculator for experiment sizing

---

## 📚 **Technical References**

- **Polars Documentation**: https://pola-rs.github.io/polars/
- **Event Schema Best Practices**: Segment's Good Data Guide
- **Retention Benchmarks**: Lenny Rachitsky's SaaS Metrics Newsletter

---

## 🐛 **Known Issues**

1. **Cell [42] Error**: References non-existent `session_id` column
   - **Status**: Commented out pending session reconstruction implementation
   - **Workaround**: Use temporal clustering as proxy for sessions

2. **Plotnine Deprecation Warnings**: `streaming` parameter warning in Polars
   - **Status**: Cosmetic only, does not affect results
   - **Fix**: Update to `engine='streaming'` syntax in future version

---

## ⏱️ **Time Investment**

**Total Analysis Time**: ~4 hours (as per assignment guidelines)

Breakdown:
- Data exploration & quality validation: 1.5 hours
- User behavior & segmentation analysis: 1.5 hours
- Content performance & surface analysis: 1 hour
- Documentation (README): 30 minutes (not counted toward 4-hour limit)

---

## 📞 **Contact**

**Submission Date**: December 8, 2025  
**Dataset Period**: October 1-10, 2025  
**Total Events Analyzed**: 6,944,203 (after deduplication)

---

## 📄 **License & Confidentiality**

This analysis is submitted as part of a take-home assignment for Latitude (AI Dungeon).  
Dataset provided under confidentiality terms - not for public distribution.

---

**Last Updated**: December 8, 2025