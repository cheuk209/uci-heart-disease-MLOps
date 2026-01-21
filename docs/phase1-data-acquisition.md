# Phase 1: Data Acquisition & Exploration

> **Theme:** "Know Your Data Before You Touch It"

## Learning Objectives

- Understand data provenance and why it's a first-class concern in ML systems
- Learn EDA(Exploratory Data Analysis) as a systematic discipline, not ad-hoc poking
- Develop the habit of documentation-first data work
- Internalize that exploration must precede transformation

---

## Key Concepts

### Why Data Provenance Matters

Chip Huyen emphasizes a critical truth: ML systems fail more often from data issues than model issues. Yet engineers spend 90% of their time on models and 10% on data. This ratio is backwards.

Data provenance means knowing:
- **WHERE** your data comes from (which system, which API, which file)
- **WHO** collected it (what organization, what process, what incentives)
- **WHEN** it was collected (time period, frequency, any gaps)
- **HOW** it was collected (methodology, sampling, instrumentation)
- **WHY** it was collected (original purpose, which may differ from yours)

### Takeaways from reading repository page
This database contains 76 attributes - but a subset of 14 is used for all published experiments. Very likely we will follow suit. 
The Cleveland data is the only one being used by machine learning researchers, which actually presents an interesting challenge - the mess is the opportunity. 

Data comes from multiple sources with different characteristics, how do we handle them?

This isn't optional metadata. It's essential context for every downstream decision.

**For the UCI Heart Disease dataset specifically:**
- It comes from 4 different hospitals (Cleveland, Hungarian, Switzerland, VA Long Beach)
- Collected in the 1980s (medical understanding has changed since)
- Different hospitals may have had different equipment, protocols, demographics
- Originally collected for medical research, not ML training

Every one of these facts affects how you should use this data. Treating it as "just rows and columns" is the first mistake.

### The Data Quality Dimensions

Before touching data, assess it against four dimensions:

**Completeness:** Is all the data there?
- Missing values (explicit or encoded as special values like "?")
- Missing rows (was sampling systematic or did data get lost?)
- Missing columns (are there variables that should exist but don't?)

**Consistency:** Does the data agree with itself?
- Same feature encoded differently across sources?
- Categorical values that should match but have typos or variations?
- Numerical ranges that don't make sense?

**Accuracy:** Is the data correct?
- Values that are technically present but wrong?
- Measurement error in sensors or surveys?
- Transcription errors in manual entry?

**Timeliness:** Is the data fresh enough?
- Does age of data matter for your use case?
- Has the underlying phenomenon changed since collection?
- Are you predicting something that evolves over time?

**The UCI Heart Disease dataset will fail on several of these dimensions. That's the point.** Learning to identify and handle these issues is the skill.

### EDA as Hypothesis Generation

You're not exploring data to make pretty charts. You're doing science.

The process:
1. **Observe:** Look at distributions, relationships, anomalies
2. **Hypothesize:** "I suspect the Swiss hospital has different missingness patterns than Cleveland"
3. **Test:** Actually check if your hypothesis is true
4. **Record:** Document what you found, whether expected or surprising

This is the scientific method applied to data understanding. Random poking isn't exploration - it's hoping to stumble onto insights.

**Key mindset shift:** Ask questions before looking at answers. "What do I expect this distribution to look like?" THEN look. This prevents post-hoc rationalization.

### The Data Contract Mental Model

In production, data comes from upstream systems. Those systems make implicit (sometimes explicit) promises:
- "This field will always be present"
- "Values will be between 0 and 100"
- "Updates happen daily at midnight"

When these contracts are violated, your system breaks.

Even with a static dataset like UCI Heart Disease, think this way:
- What guarantees does each column provide?
- What happens when those guarantees are violated?
- What should your system do when it encounters unexpected values?

This prepares you for production reality where upstream systems will disappoint you.

### Why Exploration Must Precede Cleaning

Every cleaning decision destroys information:
- Dropping rows: Those rows contained SOMETHING, even if inconvenient
- Imputing values: You're injecting synthetic data that may not reflect reality
- Removing outliers: Those "outliers" might be the most interesting cases

Before you can make these decisions wisely, you must understand:
- What exactly is missing or problematic?
- Is the missingness random or systematic?
- What would each cleaning choice actually remove or change?

**The irreversibility principle:** You cannot undo dropping rows. You cannot recover original values after imputation. Raw data understanding is prerequisite to deciding what transformations are even valid.

### Sampling Bias: Understanding What Your Data Can and Cannot Claim

Chip Huyen's Chapter 4 covers data collection and sampling strategies extensively. You might think: "We already have the data, so sampling isn't relevant." This is wrong.

**You didn't collect the data, but someone did.** Understanding HOW they collected it determines what your model can generalize to.

#### The UCI Heart Disease Sampling Reality

**Who was sampled?**
- Patients who visited these 4 specific cardiology clinics
- People already *suspected* of heart disease (why else would they get these tests?)
- This is **selection bias** - healthy people who never visit cardiologists aren't represented

**How were they sampled?**
- Convenience sampling (whoever happened to come to these hospitals)
- NOT a random sample of "all people who might have heart disease"
- NOT representative of any defined population

**When were they sampled?**
- 1980s data
- Medical equipment, diagnostic criteria, and treatments have changed
- Population health has changed (obesity rates, smoking rates, medication usage)

#### What This Means for Your Model

| Question | Implication |
|----------|-------------|
| Can this model generalize to a new hospital in 2024? | Uncertain - different population, different equipment, different era |
| Who is MISSING from this data? | Healthy people, uninsured people, people who died before diagnosis, people in other countries |
| Would Cleveland model work for Swiss patients? | The data suggests these populations differ significantly |
| Is this a "heart disease detector" or a "1980s cardiology clinic patient classifier"? | Think carefully about this distinction |

#### The Production Connection

Even though you're working with historical data, ask production-minded questions:

- "If I deployed this model, where would new data come from?"
- "Would that new data be sampled the same way?"
- "What happens when the sampling distribution shifts?"

This connects directly to **Phase 6 (data drift)** - your model assumes future data looks like training data. If the underlying population or sampling method changes, your model's assumptions break.

#### Key Takeaway

You're not doing sampling, but you must understand what sampling was done. The question isn't "how do I collect data?" but **"what are the consequences of how this data was collected?"**

This determines what your model can legitimately claim to predict.

---

## Pattern Descriptions

### The Data Profile Pattern

Before any manual exploration, generate an automated statistical summary:
- For each column: data type, count, unique values, missing count, missing percentage
- For numeric columns: min, max, mean, median, standard deviation, percentiles
- For categorical columns: value counts, top values, cardinality

This gives you a baseline. You know the shape of your data before diving into specifics.

**Key consideration:** Profile the raw data exactly as received. Don't clean "obvious" problems first - you might be hiding important signals.

### The Source Comparison Pattern

When data comes from multiple sources (like 4 hospitals), compare them systematically:
- Do they have the same columns?
- Do they have the same value ranges?
- Do they have the same missingness patterns?
- Do they have the same label distributions?

If sources differ significantly, you need to understand why before combining them.

**Key consideration:** Differences between sources might be real (different patient populations) or artifacts (different data collection practices). You need to distinguish these.

### The Data Diary Pattern

Keep a running log of observations as you explore:
- Date and time of observation
- What you looked at
- What you found
- What questions it raised

This serves multiple purposes:
- Prevents rediscovering the same insights
- Creates documentation for others
- Helps you track your own thinking process

**Key consideration:** Include dead ends and surprises, not just clean narratives. The messy truth is more useful than curated stories.

### The Assumption Register Pattern

Explicitly list what you're assuming about the data:
- "I assume '?' means missing, not a literal question mark"
- "I assume 'ca' (number of major vessels) should be 0-3, so 4 might be an error"
- "I assume all four hospitals used the same definition of 'heart disease'"

Every assumption is a potential failure mode. Listing them makes them visible and testable.

**Key consideration:** Revisit this register when things go wrong. Often the bug is a violated assumption you forgot you made.

---

## Concrete Exploration Steps

Follow these steps in order. Document findings as you go.

### Step 1: Download and Organize Raw Data

**Objective:** Get all 4 hospital datasets in their raw form.

- [ ] Create `data/raw/` directory
- [ ] Download all 4 `.data` files from UCI repository:
  - `processed.cleveland.data`
  - `processed.hungarian.data`
  - `processed.switzerland.data`
  - `processed.va.data`
- [ ] Download `heart-disease.names` (the data dictionary)
- [ ] Do NOT rename or modify files yet - keep them exactly as downloaded

**Record in your data diary:** Date downloaded, source URL, file sizes, any issues encountered.

### Step 2: Understand the Data Dictionary

**Objective:** Know what each column means BEFORE looking at the data.

Read `heart-disease.names` and document the 14 attributes:

| # | Attribute | Description | Type | Expected Values |
|---|-----------|-------------|------|-----------------|
| 1 | age | Age in years | Numeric | Continuous |
| 2 | sex | Sex | Binary | 0=female, 1=male |
| 3 | cp | Chest pain type | Categorical | 1-4 |
| 4 | trestbps | Resting blood pressure (mm Hg) | Numeric | Continuous |
| 5 | chol | Serum cholesterol (mg/dl) | Numeric | Continuous |
| 6 | fbs | Fasting blood sugar > 120 mg/dl | Binary | 0=false, 1=true |
| 7 | restecg | Resting ECG results | Categorical | 0-2 |
| 8 | thalach | Maximum heart rate achieved | Numeric | Continuous |
| 9 | exang | Exercise induced angina | Binary | 0=no, 1=yes |
| 10 | oldpeak | ST depression induced by exercise | Numeric | Continuous |
| 11 | slope | Slope of peak exercise ST segment | Categorical | 1-3 |
| 12 | ca | Number of major vessels colored by fluoroscopy | Numeric | 0-3 |
| 13 | thal | Thalassemia | Categorical | 3=normal, 6=fixed defect, 7=reversible defect |
| 14 | num | Diagnosis of heart disease (TARGET) | Categorical | 0=no disease, 1-4=disease severity |

**Questions to answer:**
- [ ] What does "ST depression" mean medically? (Look it up)
- [ ] What is thalassemia and why is it relevant to heart disease?
- [ ] The target has values 0-4, but most researchers binarize to 0 vs 1-4. Note this.

### Step 3: Load and Profile Each Source Separately

**Objective:** Understand each hospital's data independently before combining.

For EACH of the 4 hospital files:

- [ ] Load the raw file (note: no headers, comma-separated, "?" for missing)
- [ ] Check row count
- [ ] Check column count (should be 14, but verify)
- [ ] Generate basic statistics for each column
- [ ] Count missing values per column
- [ ] Check value distributions for categorical columns

**Create a comparison table:**

| Metric | Cleveland | Hungarian | Switzerland | VA |
|--------|-----------|-----------|-------------|-----|
| Row count | ? | ? | ? | ? |
| Column count | ? | ? | ? | ? |
| Missing values (total) | ? | ? | ? | ? |
| % with heart disease | ? | ? | ? | ? |

**Hypothesis before looking:** Which hospital do you expect to have the most missing data? Why?

### Step 4: Investigate Missingness Patterns

**Objective:** Understand WHY data is missing, not just WHERE.

For each column with missing values:

- [ ] Calculate % missing per hospital
- [ ] Is missingness consistent across hospitals or concentrated in some?
- [ ] Are certain columns missing together? (e.g., if `ca` is missing, is `thal` also missing?)
- [ ] Are patients with missing values different from those without? (older? sicker?)

**Create a missingness heatmap:** Rows = patients, Columns = features. Color = missing/present. Look for patterns.

**Key questions:**
- [ ] Which columns have the most missing values?
- [ ] Is Cleveland really more complete than the others?
- [ ] Does missingness correlate with the target variable?

### Step 5: Compare Distributions Across Hospitals

**Objective:** Determine if the 4 hospitals are actually comparable.

For each numeric column, compare across hospitals:
- [ ] Mean and median
- [ ] Standard deviation
- [ ] Min and max
- [ ] Distribution shape (histograms)

For each categorical column:
- [ ] Value counts per hospital
- [ ] Are the same categories present in all hospitals?

**Key questions:**
- [ ] Do the hospitals have similar patient demographics (age, sex)?
- [ ] Are the target distributions similar? (If Hungarian has 90% disease and Cleveland has 40%, that's a problem)
- [ ] Any columns where one hospital looks completely different?

### Step 6: Examine Edge Cases and Anomalies

**Objective:** Find the weird stuff that will break your pipeline later.

- [ ] Any values outside expected ranges? (e.g., age > 120, negative blood pressure)
- [ ] Any unexpected values in categorical columns? (e.g., `ca` = 4 when docs say 0-3)
- [ ] Any rows that are entirely or mostly missing?
- [ ] Any duplicate rows?

**For each anomaly found:**
- Document what you found
- Hypothesize why it might exist
- Note whether it's likely an error or valid edge case

### Step 7: Write Your Assumption Register

**Objective:** Make implicit assumptions explicit.

Based on your exploration, document every assumption:

```
## Assumption Register

1. "?" in the data represents missing values, not literal question marks
   - Evidence: [what you observed]
   - Risk if wrong: [what would break]

2. The 14 columns are in the order specified in heart-disease.names
   - Evidence: [how you verified]
   - Risk if wrong: [features would be mislabeled]

3. All hospitals used the same definition of "heart disease"
   - Evidence: [none - this is an assumption]
   - Risk if wrong: [labels mean different things]

4. [Continue for all assumptions...]
```

### Step 8: Summarize Findings

**Objective:** Create a one-page summary of what you learned.

Write a brief document answering:
1. How many total patients across all hospitals?
2. What are the biggest data quality issues?
3. Are the hospitals comparable enough to combine?
4. What will be the main challenges in Phase 2 (cleaning)?
5. What would you want to ask a cardiologist if you could?

---

## Common Pitfalls

### Pitfall 1: Cleaning before understanding
The urge to "fix" obvious problems is strong. Resist it. Those "?" values might encode something meaningful. Those outliers might be your most important cases. Understand first, clean second.

### Pitfall 2: Exploring in code only
Statistics are important, but so is visualization. And so is actually looking at raw rows. A mean of 54 for age is fine. But scrolling through actual patient records builds intuition that statistics miss.

### Pitfall 3: Assuming sources are comparable
Just because data comes in the same format doesn't mean it measures the same thing. Hospital A's "chest pain type 1" might not mean the same as Hospital B's.

### Pitfall 4: Forgetting to document
"I'll remember why I made this decision" - you won't. Document as you go. A week from now, you'll thank yourself.

### Pitfall 5: Looking for confirmation
If you expect the data to look a certain way, you'll see patterns that support your expectation. Always ask: "What would make me change my mind about this?" Then look for that.

---

## Questions to Answer

Before proceeding to Phase 2, you should be able to answer:

1. **What would break if a 5th hospital was added tomorrow?**
   - How do you handle a new source with potentially different characteristics?
   - What assumptions would need to be revisited?

2. **Which columns have missing values? Is the missingness random or systematic?**
   - What percentage is missing in each column?
   - Are certain types of patients more likely to have missing values?
   - Could the missingness itself be informative?

3. **Are the 4 hospital sources actually comparable? What evidence do you have?**
   - Do they have the same distributions?
   - Do they have the same missingness patterns?
   - Should they be treated as one dataset or four?

4. **What does each column MEAN, not just statistically, but semantically?**
   - What is "thal"? What do its values represent?
   - What is "oldpeak"? What does it measure?
   - Would a cardiologist agree with your interpretation?

---

## Success Criteria

You have completed this phase when:

- [ ] Data has been downloaded and stored in the appropriate directory
- [ ] A comprehensive data profile exists documenting all columns
- [ ] Each source (hospital) has been compared systematically
- [ ] Missing values have been catalogued with hypotheses about their cause
- [ ] All assumptions about the data have been explicitly listed
- [ ] A data diary documents your exploration process
- [ ] You can explain what each column represents in plain English
- [ ] You have NOT cleaned or transformed the data yet (that's Phase 2)

---

## Exercises

### Exercise 1: The Prediction Challenge
Before looking at any correlations with the target, predict which features you think will be most predictive of heart disease. Write down your predictions with reasoning. Then check. This exercises your domain intuition.

### Exercise 2: The Outlier Investigation
Identify potential outliers in numeric columns. For each one, investigate: Is this a data error or a genuine extreme value? What evidence supports your conclusion?

### Exercise 3: The Source Detective
Pick one column and trace how its values differ across the four hospitals. Generate hypotheses for why differences exist. Which are likely data issues vs. real population differences?

### Exercise 4: The Missing Data Map
Create a visualization showing the pattern of missing data across columns and rows. Can you identify patients who are missing more data? Columns that are missing together?

---

## References

- **Chip Huyen "Designing ML Systems"** - Chapter 4 (Training Data)
  - Particularly sections on data collection, data sources, and data quality

- **UCI ML Repository - Heart Disease Dataset**
  - https://archive.ics.uci.edu/dataset/45/heart+disease
  - Read the original documentation, not just the data files

- **"Tidy Data" by Hadley Wickham**
  - The principles of data organization (structure, not the R implementation)
  - https://vita.had.co.nz/papers/tidy-data.pdf

- **"Data Quality: The Field Guide" (Google)**
  - Practical approaches to assessing data quality

---

## What's Next

With a thorough understanding of your raw data, Phase 2 will focus on the critical concept of data leakage and proper train/test splitting. You'll learn why the boundary between training and test data is sacred, and how violations of this boundary silently corrupt your evaluation metrics.
