# SEHS4697 Big Data and Cloud Analytics Group Project Report
## 2025/26 Semester 2 – Class B01
### Group 7 – Unlocking Airbnb Host Profitability: A Data-Driven Approach to Pricing, Occupancy, and Host Performance

**Group Members**

- AU Txx Wxx (23057xxxS)
- CHUK Hxx Yxx (23048xxxS)
- LAM Chxx Luxx (23066xxxS)
- Ng Hxx Lxx (23066xxxS)
- SUN Hx Txx (23046xxxS)

---

## Abstract

<!-- ~150–200 words -->
<!-- Briefly state: dataset (Inside Airbnb HK 2025-09, 6,803 listings, 79 variables), four business questions, key methods (regression, classification, K-means, occupancy model), and 2–3 main findings. -->

---

## I. Introduction

### I.1 Background and Motivation

- Brief overview of Airbnb’s role in the Hong Kong short-term rental market.
- Motivation: hosts care about profitability (pricing, occupancy), reputation (Superhost), and ease of booking (Instant Bookable).

### I.2 Objectives and Research Questions

- Dataset: Inside Airbnb Hong Kong snapshot (2025-09) with 6,803 listings and 79 attributes [file:27].
- Overall objective: use data-driven analysis to help Airbnb hosts and the platform optimize pricing, occupancy, and host performance.

**Business Questions**

1. What are the determinants of listing prices, and how can hosts optimize their pricing strategy?
2. Which host characteristics and property attributes best predict Superhost status?
3. Is it possible to divide the Hong Kong Airbnb market into meaningful clusters of properties for targeted marketing?
4. Does the Instant Bookable feature have a significant effect on occupancy, controlling for price and ratings?

---

## II. Data Summary, Pre-processing, and Initial Insights

### II.1 Data Source and Description

- Data source: Inside Airbnb – Hong Kong listings (2025-09 snapshot), from:
  - Data file: `https://data.insideairbnb.com/china/hk/hong-kong/2025-09` [file:27].
  - Original source site: `https://insideairbnb.com/` [file:27].
- Number of observations and variables (rows and columns).
- Short description of key groups of variables: host-level, listing-level, review scores, calendar/availability, etc. [file:24].

**Table 1. Key Variables Used in the Project**

| Variable name              | Type      | Role        | Business Question(s) | Notes/Definition                           |
|---------------------------|-----------|-------------|----------------------|--------------------------------------------|
| `price_clean`             | Numeric   | Target      | Q1, Q3, Q4           | Cleaned numeric price (HKD)                |
| `accommodates`            | Numeric   | Predictor   | Q1                   | Guest capacity                             |
| `bathrooms`, `bedrooms`   | Numeric   | Predictor   | Q1                   | Bathroom and bedroom count                 |
| `room_type`               | Factor    | Predictor   | Q1                   | Entire home/apt, private room, etc.        |
| `neighbourhood_cleansed`  | Factor    | Predictor   | Q1                   | District name                              |
| `host_is_superhost`       | Factor    | Target      | Q2                   | Superhost status (Yes/No)                  |
| `host_response_rate_num`  | Numeric   | Predictor   | Q2                   | Host response rate (%)                     |
| `host_acceptance_rate_num`| Numeric   | Predictor   | Q2                   | Host acceptance rate (%)                   |
| `review_scores_rating`    | Numeric   | Predictor   | Q2, Q4               | Overall rating score                       |
| `number_of_reviews`       | Numeric   | Predictor   | Q2                   | Total number of reviews                    |
| `host_listings_count`     | Numeric   | Predictor   | Q2                   | Number of listings by the host             |
| `availability_365`        | Numeric   | Predictor   | Q3, Q4               | Available days per year                    |
| `number_of_reviews_l30d`  | Numeric   | Predictor   | Q3                   | Reviews in last 30 days                    |
| `instant_bookable`        | Factor    | Predictor   | Q2, Q4               | Instant Bookable (Yes/No)                  |
| `high_occupancy`          | Factor    | Target      | Q4                   | Derived from `availability_365` (<180 vs ≥180) |

*(You can adjust / extend this table based on the variables you actually use.)* [file:24][file:27]

---

### II.2 Data Quality Issues and Cleaning

Describe key issues identified and how you addressed them:

- **Price formatting**: `price` originally stored as strings with currency symbols and commas; cleaned using regex to produce numeric `price_clean` [file:27][file:24].
- **Missing values**:
  - Rows with missing `price` removed, as price is the target for Q1 and used in other questions [file:27].
  - `review_scores_rating` and `review_scores_value` imputed using median values to retain more observations [file:27].
- **Type conversions**:
  - `host_response_rate` and `host_acceptance_rate` converted from percentage strings to numeric [file:24].
  - `host_is_superhost`, `instant_bookable`, `room_type`, `property_type`, `neighbourhood_cleansed` converted to factors [file:24][file:27].
- **Derived variable**:
  - `high_occupancy` defined as 1 if `availability_365 < 180`, else 0, then converted to factor [file:27].

**Table 2. Missing Values Before and After Cleaning**

| Variable                 | Missing (raw) | Missing (cleaned) |
|--------------------------|---------------|-------------------|
| price                    | …             | …                 |
| review_scores_rating     | …             | …                 |
| review_scores_value      | …             | …                 |
| host_response_rate_num   | …             | …                 |
| host_acceptance_rate_num | …             | …                 |
| availability_365         | …             | …                 |

*(Fill in counts from R.)*

---

### II.3 Exploratory Data Analysis and Initial Insights

Include key descriptive statistics and visualizations:

- **Table 3. Summary Statistics of Key Numeric Variables**

  Show mean, median, min, max, sd for: `price_clean`, `availability_365`, `number_of_reviews`, `review_scores_rating`, `host_response_rate_num`, etc.

- Plots (described briefly in text):
  - Histogram of `price_clean` (and optionally log(price)).
  - Bar plot of `room_type` distribution.
  - Bar plot of `host_is_superhost` share (class imbalance).
  - Boxplot of `price_clean` by `room_type` or `neighbourhood_cleansed`.

**Initial Insights (examples)**

- Entire homes/apartments have higher median prices than private rooms.  
- Superhosts represent a minority of all hosts, confirming class imbalance for Q2.  
- Some districts show higher typical prices than others, suggesting a location effect.

---

## III. Data Analysis

### III.1 Q1 – Determinants of Listing Price (Regression)

#### III.1.1 Methodology

- Models: multiple linear regression and decision tree regression.
- Target: `price_clean`.
- Predictors: `accommodates`, `bathrooms`, `bedrooms`, `beds`, `room_type`, `neighbourhood_cleansed`.  
- Train–test split: 70% training, 30% testing; evaluation using RMSE and \(R^2\) [file:13][file:27].
- Brief explanation of why these predictors and metrics are appropriate.

#### III.1.2 Results

**Table 4. Regression Performance (Test Set)**

| Model              | RMSE  | R²    |
|--------------------|------:|------:|
| Linear regression  | …     | …     |
| Decision tree      | …     | …     |

**Table 5. Key Regression Coefficients / Variable Importance**

| Variable              | Effect / Importance | Interpretation                  |
|-----------------------|---------------------|---------------------------------|
| accommodates          | …                   |                                 |
| room_type: Entire home| …                   |                                 |
| neighbourhood_cleansed| …                   |                                 |

*(Fill with your actual output.)*

#### III.1.3 Interpretation

- Highlight which features most strongly increase or decrease price (capacity, room type, neighborhood, etc.).
- Comment on RMSE relative to average price (e.g. “RMSE corresponds to around X% of mean price, indicating moderate predictive accuracy.”).
- Link back to practical pricing recommendations for hosts.

---

### III.2 Q2 – Predicting Superhost Status (Classification)

#### III.2.1 Methodology

- Target: `host_is_superhost` (factor).
- Predictors: `host_response_rate_num`, `host_acceptance_rate_num`, `review_scores_rating`, `instant_bookable`, `number_of_reviews`, `host_listings_count` [file:27][file:24].
- Train–test split: 70/30.
- Handle class imbalance using up-sampling with caret during training [file:27][file:15].
- Model: logistic regression (optionally compare with decision tree).
- Evaluation metrics: accuracy, precision, recall, confusion matrix, and (optionally) ROC AUC [file:15].

#### III.2.2 Results

**Table 6. Superhost Classification Metrics (Test Set)**

| Model              | Accuracy | Precision | Recall | ROC AUC |
|--------------------|---------:|----------:|-------:|--------:|
| Logistic regression| …        | …         | …      | …       |

**Table 7. Confusion Matrix (Logistic Regression)**

|                      | Predicted Non-Superhost | Predicted Superhost |
|----------------------|------------------------:|--------------------:|
| Actual Non-Superhost | …                      | …                  |
| Actual Superhost     | …                      | …                  |

**Variable Importance Table/Plot**

- Show relative importance of each predictor (from `varImp`).

#### III.2.3 Interpretation

- Discuss whether accuracy meets or approaches your target (e.g. 80%+).
- Interpret which host behaviors and listing features are most linked to Superhost status (e.g. high response rate, more reviews).
- Derive recommendations for hosts seeking to become Superhosts.

---

### III.3 Q3 – Clustering Listings for Market Segmentation (K-means)

#### III.3.1 Methodology

- Variables used: `price_clean`, `availability_365`, `number_of_reviews_l30d` [file:27].
- Preprocessing: scale/standardize variables before clustering [file:7].
- Choose number of clusters k using elbow method (k from 2 to 6) [file:7].
- Final choice of k (e.g. k = 3) with a short justification.

#### III.3.2 Results

- Elbow plot description (k vs total within-cluster sum of squares).
- Summary of each cluster.

**Table 8. Cluster Summary**

| Cluster | Avg price | Avg availability_365 | Avg reviews (30d) | n listings | Segment label                |
|---------|----------:|---------------------:|------------------:|----------:|------------------------------|
| 1       | …         | …                    | …                 | …         | e.g. Budget high-demand      |
| 2       | …         | …                    | …                 | …         | e.g. Premium low-demand      |
| 3       | …         | …                    | …                 | …         | e.g. Mid-range stable demand |

- Optional scatter plots: price vs reviews or price vs availability colored by cluster.

#### III.3.3 Interpretation

- Describe each cluster’s characteristics (price level, availability, recent demand).
- Explain how Airbnb might use these segments for targeted promotions, search ranking, or dynamic pricing [file:27].

---

### III.4 Q4 – Instant Bookable and Occupancy (Binary Classification)

#### III.4.1 Methodology

- Target: `high_occupancy` derived from `availability_365` (<180 = 1, ≥180 = 0) [file:27].
- Predictors: `instant_bookable`, `price_clean`, `review_scores_value` [file:27][file:24].
- Model: logistic regression (and/or decision tree).
- Evaluation: accuracy and confusion matrix; examine coefficient/importance for `instant_bookable`.

#### III.4.2 Results

**Table 9. High Occupancy Classification Metrics (Test Set)**

| Model              | Accuracy | Precision | Recall |
|--------------------|---------:|----------:|-------:|
| Logistic regression| …        | …         | …      |

- Table comparing average `availability_365` and proportion of `high_occupancy` for Instant vs non-Instant listings.

**Table 10. Occupancy vs Instant Bookable (Descriptive)**

| Instant_bookable | Avg availability_365 | % high_occupancy |
|------------------|---------------------:|-----------------:|
| No               | …                    | …                |
| Yes              | …                    | …                |

#### III.4.3 Interpretation

- Discuss whether Instant Bookable has a strong and positive association with high occupancy after controlling for price and review scores [file:27].
- Approximate the magnitude of the effect (e.g. change in odds or probability).
- Link back to your business argument: why Airbnb should encourage Instant Bookable among hosts.

---

## IV. Discussion

### IV.1 Summary of Findings by Question

- Q1: Key drivers of price and how reliable the pricing model is.
- Q2: Important predictors of Superhost status and model performance.
- Q3: Main segments discovered and their strategic meaning.
- Q4: Evidence regarding the impact of Instant Bookable on occupancy.

### IV.2 Business Implications

- Recommendations for individual hosts (pricing, host behavior, Instant Book use).
- Recommendations for Airbnb platform (training program design, marketing segmentation, Instant Book promotion) [file:27].

### IV.3 Limitations and Future Work

- Limitations: single snapshot, observational data (not causal), missing or noisy information, no text review analysis, etc. [file:19].
- Future work: time-series analysis, text mining of reviews, more advanced ML models, causal inference, or experimentation.

---

## V. References

- Inside Airbnb. (Year). *Inside Airbnb: Adding data to the debate*. Retrieved from https://insideairbnb.com/ [file:27].
- Course lecture notes on regression, classification, clustering, and evaluation [file:13][file:15][file:7][file:5][file:4].
- Any additional academic or technical references you used (in APA style).

---

## VI. Appendix

- Detailed R code for data cleaning, regression, classification, clustering, and Instant Book analysis.
- Full model summaries and additional tables/plots not included in the main 10 pages.
- Any other supplementary material such as variable lists or extended diagnostics.
