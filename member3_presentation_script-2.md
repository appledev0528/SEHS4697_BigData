# Member 3 — Presentation Script

> **Role:** Member 3 | **Topic:** Q2 — Classification Problem  
> **Model:** Decision Tree (`rpart`) | **Technique:** 10-fold CV + Up-Sampling  
> **Dataset:** `listings_clean.csv` · **5,401 listings** · Inside Airbnb HK 2025-09-23

---

## Slide 1 — Title Slide
**⏱ ~20 seconds**

> "My section is Question 2 — the classification problem. The business question is: can we predict whether a host will become a Superhost, and more importantly, what are the key factors that drive that status? Superhosts earn over HK$92,000 more per year — so this question has real financial impact."

---

## Slide 2 — Business Question & Data Split
**⏱ ~35 seconds**

> "Superhosts charge HK$748 median per night and are occupied 173 days a year. Regular hosts charge HK$394 and are only occupied 95 days. That is a 90% price gap and 82% more bookings — so Superhost status is worth pursuing. I split the data 80/20 using `set.seed(56109111)`, giving 4,357 training rows and 1,044 test rows."

### Code
```r
set.seed(56109111)
pname     = sample(x=c("train","test"), size=nrow(df), replace=TRUE, prob=c(0.8,0.2))
data_train = df[pname=="train",]   # ~4,357 rows
data_test  = df[pname=="test", ]   # ~1,044 rows
```

### Output — Business Impact (from `superhost.summary`)
```
  host_is_superhost Count Median.Price Avg.Occupied
1 No                 4747          394         94.9
2 Yes                 654          748.       173.
```
> HK$394 × 94.9 = **HK$37,391/yr** vs HK$748 × 173 = **HK$129,404/yr** → **+HK$92,013 uplift**

---

## Slide 3 — Data Challenge + Modeling Approach
**⏱ ~45 seconds**

> "`table(data_train$host_is_superhost)` showed 3,815 Non-Superhosts vs only 542 Superhosts — 87.5% vs 12.5%. That is a severe class imbalance. Without fixing it, a model could hit 89.3% accuracy just by always saying 'No' — and never find a single Superhost. I fixed this with `sampling='up'` from Lecture 6, which duplicates Superhost rows until both classes are balanced. Then I trained a Decision Tree with 10-fold cross-validation using five predictors."

### Code
```r
# Check imbalance
table(data_train$host_is_superhost)

# Fix with up-sampling (Lecture 6) + 10-fold CV
train_ctl = trainControl(method="cv", number=10, sampling="up")

# Train Decision Tree
model <- train(host_is_superhost ~ price + accommodates +
               review_scores_rating + number_of_reviews + instant_bookable,
               data=data_train, trControl=train_ctl, method="rpart")

rpart.plot(model$finalModel)   # Visualise decision rules
```

### Output — Class Distribution
```
  No  Yes
3815  542        # 87.5% vs 12.5% → imbalanced
```

---

## Slide 4 — Confusion Matrix & Results
**⏱ ~55 seconds**

> "On the 1,044 test rows, the model achieved 84.1% accuracy. More importantly, **Sensitivity is 72.3%** — it correctly identifies 7 out of 10 actual Superhosts. The **Kappa is 0.41**, which confirms genuine learning. The No Information Rate — always guessing 'No' — gives 89.3% accuracy but Kappa of zero. Our model trades a small drop in raw accuracy for a massive gain in actually finding Superhosts."

### Code
```r
# Cross-validated result (training folds)
print(confusionMatrix(model))

# Test partition result (held-out data)
test_accuracy = confusionMatrix(predict(model, newdata=data_test),
                                data_test$host_is_superhost, positive="Yes")
print(test_accuracy)
```

### Output — Cross-Validated (training)
```
Accuracy (average) : 0.8331
```

### Output — Test Partition (KEY RESULT)
```
          Reference
Prediction  No Yes
       No  797  31
       Yes 135  81

Accuracy    : 0.8410      NIR  : 0.8927
Kappa       : 0.4106      Sensitivity : 0.72321
Specificity : 0.85515
```

---

## Slide 5 — Variable Importance
**⏱ ~35 seconds**

> "`varImp(model)` shows review score rating is the #1 predictor at 100, followed by number of reviews at 90.77. Most strikingly, **Instant Bookable has zero importance — 0.00**. This means enabling Instant Bookable has no effect on Superhost status at all. These are independent strategies — which is exactly what Member 5 will explore next."

### Code
```r
print(varImp(model))
```

### Output
```
review_scores_rating  100.00   ← #1 — most critical
number_of_reviews      90.77   ← #2 — trust signal
price                  31.75   ← #3 — links to Member 2
accommodates           22.63
instant_bookableYes     0.00   ← ZERO — independent of Superhost
```

---

## Slide 6 — Business Impact
**⏱ ~30 seconds**

> "Putting it all together: a Superhost earns HK$129,404 per year versus HK$37,391 for a regular host — a HK$92,013 annual uplift. Our model tells you exactly how to get there: maintain a high review score, collect more reviews, and price competitively using Member 2's model."

---

## Presentation Timeline

| Slide | Topic | Key Figure | Time |
|-------|-------|-----------|------|
| 1 | Title | 84.1% accuracy, +HK$92K | 20s |
| 2 | Business Question | HK$748 vs HK$394, 173 vs 94.9 days | 35s |
| 3 | Imbalance + Model | No=3,815 / Yes=542 → sampling="up" | 45s |
| 4 | Confusion Matrix | Accuracy 84.1%, Sensitivity 72.3%, Kappa 0.41 | 55s |
| 5 | Variable Importance | rating=100, instant_bookable=0 | 35s |
| 6 | Business Impact | HK$129,404 vs HK$37,391 = +HK$92,013 | 30s |
| **Total** | | | **~3 min 20s** |

---

## Q&A Preparation

### Q1: Why Decision Tree over Logistic Regression?
> "Decision Tree produces readable if-else rules shown by `rpart.plot` — directly actionable for a host. Logistic Regression gives coefficients that are harder to translate into concrete advice."

### Q2: Accuracy 84.1% is below the 89.3% baseline — isn't the model worse?
> "The 89.3% baseline comes from always predicting 'No' — it never finds a single Superhost. Kappa is zero. Our model gets **Sensitivity 72.3%** and **Kappa 0.41**, proving it genuinely learns Superhost patterns."

### Q3: What does `sampling='up'` do?
> "It duplicates Superhost rows in training data until both classes balance. Our 542 Superhost rows are replicated up to ~3,815 so the model sees equal examples in each of the 10 CV folds."

```r
# Before up-sampling:
table(data_train$host_is_superhost)
#   No  Yes
# 3815  542   ← 87.5% vs 12.5%

train_ctl = trainControl(method="cv", number=10, sampling="up")
#                                                ^^^^^^^^^^^^ Lecture 6 fix
```

### Q4: Why is Instant Bookable zero importance?
> "Airbnb's Superhost criteria are based on response rate, completion rate, rating, and stay count — not booking settings. `instant_bookableYes = 0.00` means it adds no predictive power. Member 5 examines whether it independently affects occupancy, which is a separate question."

### Q5: What is Kappa?
> "Kappa measures agreement beyond chance. 0 = no better than random, 1 = perfect. Our **Kappa = 0.41** is moderate agreement. The baseline model's Kappa = 0.00. So despite slightly lower raw accuracy, our model is genuinely learning."

### Q6: Could the model be improved?
> "Yes — tune the `cp` hyperparameter with a grid search, try Random Forest (`method='rf'`), or add `neighbourhood_cleansed` as a feature. For this project the Decision Tree gives a transparent, reproducible result."

### Q7: How does this connect to other members?
> "Member 1 created `occupancy_proxy` and cleaned `host_is_superhost` — both used here. `price` having importance 31.75 means Member 2's pricing model feeds into achieving Superhost status. `instant_bookable = 0.00` sets up Member 5's analysis."

---

## Key Concepts Cheat Sheet

| Concept | Definition | Evidence |
|---------|-----------|----------|
| Class Imbalance | 87.5% vs 12.5% in training data | `table()` → No=3815, Yes=542 |
| Up-Sampling | Duplicate minority class to balance training | `trainControl(sampling="up")` |
| 10-fold CV | Rotate 10 train/validate splits | `method="cv", number=10` |
| Sensitivity | % of actual Superhosts found | **72.3%** — `Sensitivity : 0.72321` |
| Specificity | % of Non-Superhosts correctly rejected | **85.5%** — `Specificity : 0.85515` |
| Kappa | Agreement beyond chance | **0.41** — `Kappa : 0.4106` |
| NIR | Baseline — always predict majority | **89.3%** — `NIR : 0.8927` |
| varImp | Feature importance from model | rating=100, instant_bookable=0 |

---
*SEHS4697 Big Data and Cloud Analytics · Group 7 · Member 3*  
*Verified against actual R output · Reproducible with set.seed(56109111)*
