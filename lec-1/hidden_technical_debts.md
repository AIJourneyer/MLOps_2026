# Technical Debt in ML

Like money debt:

Borrow money now
→ easy today
→ repay with interest later


Technical debt:

Take shortcut now
→ faster development today
→ more bugs + complexity + maintenance later


**What:**  
Technical debt = things that make an ML system **quick and easy to build now, but harder to fix, maintain, or change later**.

**Why:**  
It happens because of **shortcuts, too many dependencies, patches, configs, pipelines, or poor testing**.

**How it grows:**  
`Quick shortcut → more complexity → harder maintenance`

**Example:**  
Instead of fixing the original model, we add another model to correct it.  
It works now, but later the system becomes harder to understand and change.

**Simple meaning:**  
`Easy today → difficult tomorrow`

---

## 1. Entanglement, Correction Cascades & Undeclared Consumers

### 1.1 Entanglement — “Changing Anything Changes Everything” (CACE)

ML features/components are often **not isolated or independent**.  
Changing one feature, dataset, preprocessing step, or model parameter can affect many other parts of the model.

**Example:**  
`location + area + bedrooms + age → house_price`

If you change how `location` is encoded, the relationships learned for `area`, `bedrooms`, etc. may also change.

**Simple meaning:**

> Change one ML component → many other behaviors may change unexpectedly.

---

### 1.2 Correction Cascades — “Model Fixing Another Model”

Instead of fixing the original model, another model is added to correct its errors.

Example:

    Input
      ↓
    Model A
      ↓
    Model B → fixes Model A
      ↓
    Model C → fixes Model B
      ↓
    Output

Now Models B and C depend on the behavior of Model A.

Changing or removing Model A may break everything after it.

**Simple meaning:**

> Keep adding patches instead of fixing the root model → the system becomes complicated and fragile.

---

### 1.3 Undeclared Consumers — “Unknown Systems Depend on Your Output”

Other teams or applications may start using your model output **without you knowing**.

Example:

    Fraud Model
         ↓
    fraud_score
         ↓
    ├── Team A
    ├── Dashboard
    ├── Another ML Model
    └── Unknown Team B

Suppose you change:

`fraud_score: 0–1`

to:

`fraud_score: 0–100`

Unknown consumers may suddenly break.

**Simple meaning:**

> Unknown systems depend on your model → changing or removing it becomes risky.

---

## 2. Data Dependency Debt — “Depending on Data You Don't Control”

Your ML model depends on datasets, tables, APIs, or features provided by another system/team.

Example:

    Team A Database
          ↓
    customer_income
          ↓
    Your ML Pipeline
          ↓
    Loan Risk Model

Team A might:

- Rename a column
- Change the datatype
- Change units
- Change the meaning
- Stop providing the data

Your pipeline may fail or, worse, continue running with incorrect data.

**Simple meaning:**

> Your model depends on external data → someone else's change can break your model.

---

## 3. Configuration Debt — “Too Many Settings to Manage”

ML systems often contain many:

- Hyperparameters
- Feature flags
- Thresholds
- Model versions
- Data paths
- Environment variables
- Preprocessing options

Example:

    learning_rate = 0.001
    batch_size = 64
    model_version = v4
    threshold = 0.72
    use_feature_A = True
    normalize = True
    data_path = train_v8.csv

Over time:

    config.yaml
    config_v2.yaml
    config_final.yaml
    config_final_new.yaml
    config_prod_latest.yaml

Eventually nobody clearly knows which configuration produced which model.

**Simple meaning:**

> Too many configurations → difficult to understand, reproduce, and safely change the system.

---

## 4. Feedback Loops — “Model Predictions Affect Future Training Data”

Sometimes model predictions change the real world, and those changes later become training data.

Example:

    Recommendation Model
            ↓
    Recommends Video A
            ↓
    More users watch Video A
            ↓
    Video A gets more engagement data
            ↓
    Model learns Video A is popular
            ↓
    Recommends Video A even more

The model is now influencing the data it learns from.

Another example:

    Loan Model → rejects customers
                     ↓
              No repayment data
                     ↓
          Future training dataset

The model only sees outcomes from people it previously approved.

**Simple meaning:**

> Model predictions influence future data → future models may reinforce their own previous decisions.

---

## 5. Pipeline Jungles + Glue Code — “Too Many Connected Steps”

Real ML systems are usually much more than:

`Data → Model → Prediction`

They may become:

    Database
       ↓
    SQL
       ↓
    Python Script
       ↓
    Cleaning
       ↓
    Feature Engineering
       ↓
    Feature Store
       ↓
    Training Script
       ↓
    Model
       ↓
    API
       ↓
    Post-processing
       ↓
    Another Service
       ↓
    Dashboard

Over time, many temporary scripts and connections are added.

This extra code connecting systems is called **glue code**.

Example:

    script1.py
       ↓
    script2.py
       ↓
    temp_fix.py
       ↓
    convert_data.py
       ↓
    model.py
       ↓
    final_fix.py

Nobody wants to remove anything because nobody knows what depends on what.

**Simple meaning:**

> Too many pipelines, scripts, and connections → the ML system becomes a jungle that is hard to maintain.

---

## 6. Data Testing + Reproducibility Debt — “Can We Trust and Recreate the Model?”

### Data Testing Debt

ML systems depend heavily on data, but teams sometimes test code without properly testing the data.

Example problems:

    age = -25
    salary = "unknown"
    country = null
    transaction_amount = 999999999
    target column suddenly missing

Without data validation, the model may silently produce bad predictions.

We should test:

- Missing values
- Data types
- Value ranges
- Schema changes
- Duplicate records
- Distribution changes
- Feature relationships

### Reproducibility Debt

Suppose a model produced:

`Accuracy = 94%`

Six months later, nobody can reproduce it because they don't know:

- Which dataset version was used
- Which code commit was used
- Which features were used
- Which model version was used
- Which hyperparameters were used
- Which random seed was used
- Which library versions were installed

**Simple meaning:**

> No data testing → bad data can silently damage the model.  
> No reproducibility → you cannot recreate the same experiment/model later.

---

## 7. External World Changes + Monitoring — “The World Changes After Deployment”

A model may work perfectly today but become inaccurate later because the real world changes.

Example:

A fraud model trained in:

`2024 data → Fraud Model`

But by 2026:

- Customer behavior changes
- New fraud techniques appear
- New products are introduced
- Economic conditions change
- Data distributions change

This can cause **data drift** or **concept drift**.

Example:

    Training Data
    avg_transaction = ₹2,000

    Production Data
    avg_transaction = ₹8,000

The production world is now different from the training world.

Therefore ML systems need monitoring for:

- Data drift
- Prediction drift
- Model accuracy
- Latency
- Errors
- Missing features
- Data quality
- Model failures
- Business KPIs

Example:

    Production Data
          ↓
        Model
          ↓
      Predictions
          ↓
      Monitoring
       ↙      ↘
    Healthy    Drift/Error
                 ↓
          Alert / Retrain

**Simple meaning:**

> The real world keeps changing → continuously monitor the model and retrain/update when necessary.

---

# Quick Memory

| Technical Debt | Easy Meaning |
|---|---|
| **Entanglement** | Change one thing → many things change |
| **Correction Cascades** | Model fixes model fixes model |
| **Undeclared Consumers** | Unknown systems use your model output |
| **Data Dependency Debt** | Depending on data you don't control |
| **Configuration Debt** | Too many settings and configurations |
| **Feedback Loops** | Model decisions influence future training data |
| **Pipeline Jungles + Glue Code** | Too many scripts and connected components |
| **Data Testing Debt** | Bad data enters without being detected |
| **Reproducibility Debt** | Cannot recreate the same model later |
| **External World Changes** | Production world changes over time |
| **Monitoring Debt** | Problems are not detected after deployment |

## One-Line Mental Model

**ML Technical Debt = Model + Data + Configuration + Dependencies + Pipelines + Feedback + Real-World Changes becoming difficult to control over time.**