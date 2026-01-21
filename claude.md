# MLOps Learning Project - Handoff Document

## Who Is This Person?

An MLOps engineer with strong DevOps and Kubernetes background, currently learning ML fundamentals. They're using Chip Huyen's "Designing Machine Learning Systems" as their guide. Strong on infrastructure, building skills on the data and ML side.

Their company works on AI-powered floor plan optimization (placing components like lighting, vents, radiators), likely using constraint optimization or graph neural networks—but this project is separate personal learning.

---

## What We're Building

A complete MLOps pipeline using the **UCI Heart Disease dataset** to predict heart disease (binary classification).

**Critical philosophy:** The model should be intentionally simple (Logistic Regression). We do not care about accuracy or fancy algorithms. The entire focus is on **process, infrastructure, and MLOps practices**. A perfectly tuned XGBoost with garbage pipeline is worthless. A mediocre model with bulletproof MLOps? That's production-ready.

---

## The Dataset

**UCI Heart Disease Dataset** from https://archive.ics.uci.edu/dataset/45/heart+disease

Why this dataset is perfect for learning:
- Has 4 different hospital sources (Cleveland, Hungarian, Switzerland, VA Long Beach) with inconsistencies between them
- Missing values encoded as "?" strings, not proper NaN
- Mix of numeric and categorical features
- Small enough to iterate fast, messy enough to be realistic
- Real medical data with real-world quirks

---

## Key Learning Areas

### 1. Data Collection & Exploration

**What to learn:**
- How to handle multiple data sources with different formats
- The importance of exploring BEFORE cleaning (don't assume)
- Documenting what you find—data reports are real deliverables

**Questions to wrestle with:**
- What does "missing" actually mean in each column? Random or systematic?
- Are the 4 hospital sources actually comparable?
- What would break if a 5th hospital was added tomorrow?

---

### 2. Data Wrangling & Cleaning

**What to learn:**
- Real data is ugly—strings where numbers should be, inconsistent encodings
- Every cleaning decision is a choice that should be documented and justified
- There's no "correct" way to handle missing values, only tradeoffs

**Questions to wrestle with:**
- Impute with mean, median, mode, or drop? What are the tradeoffs?
- Should cleaning logic be hardcoded or configurable?
- How do you test that your cleaning pipeline actually works?

---

### 3. Data Leakage (THIS IS CRITICAL)

**What to learn:**
- Why you must fit preprocessing (scalers, imputers, encoders) ONLY on training data
- How leakage silently inflates your metrics and destroys production performance
- The difference between "works in notebook" and "works in production"

**Questions to wrestle with:**
- If you impute missing values with the mean, whose mean? Train set only!
- Where else can leakage sneak in? (Hint: feature engineering, time series)
- How would you detect leakage if it happened?

---

### 4. Feature Engineering

**What to learn:**
- sklearn Pipelines and ColumnTransformers for maintainable preprocessing
- Why feature engineering code must be versioned and reproducible
- The concept of a "feature store" even at small scale

**Questions to wrestle with:**
- Should the 'source' hospital be a feature? What are implications?
- How do you handle a categorical value in production that wasn't in training?
- What happens when feature engineering logic changes—how do you track that?

---

### 5. Experiment Tracking

**What to learn:**
- Why "I tried some stuff and this worked" is not acceptable
- Systematic experimentation with proper logging (MLflow)
- What to log: parameters, metrics, artifacts, data versions, code versions

**Questions to wrestle with:**
- How do you reproduce an experiment from 3 months ago?
- What metrics actually matter for heart disease? (Hint: is false negative worse than false positive?)
- How many experiments is "enough"?

---

### 6. Data Versioning & Reproducibility

**What to learn:**
- DVC for versioning data alongside code
- Pipeline definitions that capture the full DAG of dependencies
- "Works on my machine" is not reproducibility

**Questions to wrestle with:**
- If you change one cleaning parameter, what needs to re-run?
- How do you roll back to last week's data AND last week's code?
- What's the difference between reproducibility and replicability?

---

### 7. Model Serving & Containerization

**What to learn:**
- Packaging a model with its preprocessing as a single deployable unit
- API design for ML services (input validation, error handling, versioning)
- Docker for consistent environments

**Questions to wrestle with:**
- Model and preprocessor must travel together—why?
- What should your /health endpoint actually check?
- How do you handle malformed input without crashing?

---

### 8. Kubernetes Deployment

**What to learn:**
- Resource limits and requests for ML workloads
- Readiness vs liveness probes for model services
- Rolling updates without dropping predictions

**Questions to wrestle with:**
- How much memory does your model actually need?
- What happens during deployment—do requests fail?
- How would you do canary deployments for a new model version?

---

### 9. Monitoring & Observability

**What to learn:**
- What metrics to expose (latency, throughput, prediction distribution)
- Data drift detection—when inputs change over time
- Model performance monitoring (when you have ground truth)

**Questions to wrestle with:**
- How do you know your model is failing if you don't have labels yet?
- What does "drift" look like? How much is too much?
- When should a human be alerted vs. automatic rollback?

---

## Suggested Approach

1. **Go phase by phase**, not all at once. Complete data loading before moving to cleaning.

2. **Document decisions** as you make them. A `docs/decisions.md` file explaining "why" is valuable.

3. **Break things intentionally.** Introduce leakage on purpose, see how metrics change. Deploy a broken model, see what happens.

4. **Keep the model stupid.** Resist the urge to try XGBoost or neural nets. LogisticRegression is your friend here.

5. **Treat it like production.** Pretend this will serve real patients. What would you need to trust it?

---

## Questions to Answer After Completion

Write a reflection document covering:

1. What surprised you most about working with real messy data?
2. What would break if this went to production tomorrow?
3. What's missing that a real MLOps system would need?
4. How would your approach change if the dataset was 1000x larger?
5. What would you do differently if you started over?

---

## Tech Stack (Suggested, Not Mandated)

- Python, pandas, scikit-learn for the basics
- MLflow for experiment tracking
- DVC for data versioning
- FastAPI for serving
- Docker for containerization
- Kubernetes for orchestration
- Prometheus for metrics

These are suggestions—if you prefer alternatives, use them. The concepts matter more than the tools.

---

## Notes on Working Style

- Prefers practical hands-on over theory
- Learns best by building and breaking things
- Has strong infrastructure intuition, building ML intuition
- Appreciates when things are explained simply without jargon
- Values understanding "why" not just "how"

---

*The goal is not to build the best heart disease predictor. The goal is to understand what it takes to put ML into production reliably. The model is just the excuse to learn the surrounding machinery.*