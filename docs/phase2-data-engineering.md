# Phase 2: Data Engineering & The Leakage Trap

> **Theme:** "Every Transformation is a Decision"

## Learning Objectives

- Internalize data leakage so deeply you can smell it in code reviews
- Understand the train/test boundary as a sacred contract
- Learn to think about cleaning pipelines as production-grade software
- Master the "fit on train, transform on all" principle

---

## Key Concepts

### Why Data Leakage is THE Critical Concept

Chip Huyen dedicates significant attention to data leakage because it's the #1 way ML projects fail silently. Here's what happens:

1. You build a model
2. Evaluation shows 95% accuracy
3. You deploy to production
4. Production metrics show 60% accuracy
5. You have no idea why

The answer is almost always leakage. You trained your model on information it won't have at prediction time. Your evaluation metrics were lies.

**The insidious nature of leakage:** There's no error. No warning. The model trains fine. Metrics look great. Everything seems perfect. Until it isn't.

This is why understanding leakage deeply - not as an abstract concept but as a visceral danger - is essential.

### The Leakage Taxonomy

**Target Leakage:**
Using features that are proxies for the label. Example: predicting mortality using "date of death" as a feature. The feature directly encodes the outcome.

In the UCI dataset, this is less likely since features are pre-defined. But always ask: could this feature only be known AFTER the outcome? Could it be caused BY the outcome rather than predicting it?

**Train-Test Contamination:**
Information from the test set bleeding into training. This is the most common form in practice.

Examples:
- Computing mean on the entire dataset, then splitting
- Feature scaling using all data statistics
- Encoding categorical variables based on all data frequencies

The test set exists to simulate "data you've never seen." If ANY information from it affects your training, the simulation is broken.

**Temporal Leakage:**
Using future data to predict the past. Critical in time series, less relevant for the UCI dataset (static, no time component). But understand it anyway:
- Training on 2023 data to predict 2022 outcomes
- Using features that are computed from future events

### The "Fit on Train, Transform on All" Principle

This is not a suggestion. It's not a best practice. It's a mathematical requirement for valid evaluation.

**The rule:**
1. Split data into train and test FIRST
2. Fit all transformations on training data only
3. Use those fitted transformations to transform both train and test

**Why it matters mathematically:**

When you compute a mean for imputation:
- Mean of entire dataset includes test set values
- Imputed training values now contain test set information
- Model "sees" test data during training
- Test evaluation no longer simulates unseen data

Same logic applies to:
- Standard scaling (mean and std)
- Min-max scaling (min and max)
- Frequency encoding (category frequencies)
- Any transformation that computes statistics from data

**The practical implementation:**
sklearn's Pipeline and ColumnTransformer exist specifically to enforce this. When you call `pipeline.fit(X_train)`, transformations learn from training data only. When you call `pipeline.transform(X_test)`, they apply those learned parameters.

### The Missingness Mechanism (Rubin's Framework)

Not all missing data is the same. Rubin's framework distinguishes:

**MCAR (Missing Completely At Random):**
- Missingness has no relationship to any data, observed or unobserved
- Example: Data entry clerk accidentally skips rows at random
- Safe to drop: remaining data is still representative
- Rare in practice

**MAR (Missing At Random):**
- Missingness depends on OTHER observed variables
- Example: Younger patients less likely to have cholesterol measured (healthy, skipped test)
- Dropping creates bias. Imputation can help if you model the relationship.
- Common in practice

**MNAR (Missing Not At Random):**
- Missingness depends on the MISSING VALUE ITSELF
- Example: Patients with very high cholesterol don't report it (embarrassment)
- Dangerous territory. Neither dropping nor simple imputation is correct.
- Requires domain knowledge and careful modeling

**For the UCI Heart Disease dataset:**
The missingness is likely MAR. Sicker patients may have more tests (so more complete data) OR may be too sick for some tests (so less complete data). The pattern matters for your imputation strategy.

**Key insight:** Your choice of how to handle missing data is a modeling choice with real consequences. Document your reasoning.

### Notebook Cleaning vs. Production Cleaning

In a notebook, you see the whole dataset. You can compute statistics on all data, look at all rows, iterate freely.

In production, you see ONE ROW at a time. A patient walks in, you get their data, you need to make a prediction. You cannot look at other patients. You cannot compute statistics on the fly.

**This constraint should shape your code from day one:**

Your cleaning pipeline must be:
- Stateless (except for parameters learned during fit)
- Able to transform a single row without seeing others
- Deterministic (same input always gives same output)

If your cleaning logic requires looking at multiple rows during inference, you have a design problem.

---

## Pattern Descriptions

### The Pipeline-First Pattern

Never apply transformations directly to DataFrames. Always through a pipeline object.

**Why:** Pipelines enforce fit/transform separation automatically. They make the flow explicit. They're serializable as a single artifact.

**The structure:**
1. Define your transformations as pipeline steps
2. Call `fit` on training data
3. Call `transform` on any data

**Key consideration:** This feels like overhead at first. It's not. It's insurance against leakage that scales.

### The Column Transformer Pattern

Different column types need different treatments:
- Numeric columns: scaling, imputation with statistics
- Categorical columns: encoding, imputation with mode
- Binary columns: might pass through unchanged

Group columns by type. Apply appropriate transformations to each group. The ColumnTransformer handles this composition.

**Key consideration:** Think carefully about which columns belong in which group. A numeric ID that shouldn't be scaled isn't really "numeric" for transformation purposes.

### The Leakage Test Pattern

After training a model:
1. Record your test set metrics
2. Randomly shuffle your test set labels
3. Re-evaluate (without retraining)

If metrics stay high on shuffled labels, you have leakage. The model has memorized something about the test set that isn't the actual relationship.

**Why this works:** Shuffled labels destroy the signal. If the model still performs well, it's using something other than the intended features-to-label relationship.

### The Decision Log Pattern

Every cleaning choice should be documented:
- What decision was made (e.g., "Impute missing 'ca' with median")
- Why it was made (e.g., "MAR assumption: missingness correlates with age")
- What alternatives were considered
- What evidence supported the choice

This creates an audit trail. When things go wrong, you can trace back to see which decision might be responsible.

**Key consideration:** Include things that DIDN'T work too. Understanding why approaches failed is valuable.

---

## Common Pitfalls

### Pitfall 1: Scaling before splitting
Classic leakage. You compute mean and standard deviation on all data, then split. The test set's statistics are now baked into your training data. Always split first.

### Pitfall 2: Encoding categories using all data
If category frequencies affect your encoding, and you compute frequencies on all data, you've leaked. Fit encoders on training data only.

### Pitfall 3: Feature selection using all data
You select features based on correlation with the target, using all data. Then you split. Your selection was influenced by test data. Leakage.

### Pitfall 4: Cross-validation done wrong
Each fold should be completely isolated. Preprocessing should be fit inside the CV loop, not before it. This is tedious. It's also required.

### Pitfall 5: Overlooking temporal order
Even if your data doesn't have explicit timestamps, was there an implicit order? Did data collection change over time? Shuffling might mix fundamentally different data.

---

## Questions to Answer

Before proceeding to Phase 3, you should be able to answer:

1. **If you impute with the mean, whose mean? Can you explain why?**
   - Walk through exactly what would happen if you used the global mean
   - What information would leak?

2. **Write down 3 ways leakage could sneak into this project. Now prevent them.**
   - Be specific to the UCI dataset and your processing choices
   - What mechanisms prevent each leakage mode?

3. **What type of missingness does each column likely have? What's your evidence?**
   - MCAR, MAR, or MNAR?
   - What analysis would strengthen or weaken your hypothesis?

4. **Could you apply your cleaning pipeline to a single new patient record? How?**
   - What parameters would the pipeline need?
   - What happens if the patient has a value outside ranges seen in training?

---

## Success Criteria

You have completed this phase when:

- [ ] Data is split into train and test sets BEFORE any processing
- [ ] All transformations are fit on training data only
- [ ] Preprocessing is encapsulated in a Pipeline or equivalent
- [ ] You can explain exactly why each transformation is applied
- [ ] Missing value handling strategy is documented with rationale
- [ ] You've tested your pipeline on a single row to verify it works
- [ ] The decision log documents all cleaning choices and their rationale

---

## Exercises

### Exercise 1: Leakage Injection
Intentionally introduce leakage by fitting a scaler on all data before splitting. Compare metrics to correctly implemented preprocessing. How much does leakage inflate your metrics?

### Exercise 2: The Single Row Test
Take one row from your test set. Run it through your pipeline in isolation. Verify it produces the same result as running the whole test set and extracting that row.

### Exercise 3: Missingness Analysis
For each column with missing values, test whether missingness is associated with other variables. Build a simple model predicting "is missing" from other features. What does this tell you about the mechanism?

### Exercise 4: The Time Travel Test
Imagine you deploy this model. Tomorrow, a patient comes in. Walk through exactly what happens to their data. What transformations apply? What parameters are used? Where do those parameters come from?

---

## References

- **Chip Huyen "Designing ML Systems"** - Chapter 4 (Training Data)
  - Section on Data Leakage is essential reading

- **Scikit-learn Documentation**
  - Pipeline: https://scikit-learn.org/stable/modules/compose.html
  - ColumnTransformer: https://scikit-learn.org/stable/modules/compose.html#columntransformer-for-heterogeneous-data

- **"Data Leakage in Machine Learning" (Kaggle)**
  - Practical examples with code

- **Rubin's Missing Data Framework**
  - Original statistical treatment of missingness mechanisms
  - Rubin, D.B. (1976). "Inference and missing data"

---

## What's Next

With proper train/test separation and a leakage-free pipeline foundation, Phase 3 will focus on feature engineering. You'll learn how features encode domain knowledge, how to handle production edge cases like unseen categories, and why the model and its preprocessor must travel together as one artifact.
