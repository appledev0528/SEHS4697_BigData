# SEHS4697 Group 7 – R Code (Airbnb HK Project)

This file contains R code snippets for data loading, cleaning, and analysis:
- Data cleaning & feature engineering
- Price regression (Q1)
- Superhost classification (Q2)
- K-means clustering for segments (Q3)
- Instant Book vs occupancy (Q4)

---

## 1. Setup and Data Loading

```r
# Load required packages
library(dplyr)
library(readr)
library(stringr)
library(caret)

# Load the Airbnb listings data
df_raw <- read_csv("listings-2.csv")
```

---

## 2. Select Key Variables

```r
# Keep only variables needed for your four questions
df <- df_raw %>%
  select(
    id,
    price,
    accommodates,
    bathrooms,
    bedrooms,
    beds,
    room_type,
    property_type,
    neighbourhood_cleansed,
    host_is_superhost,
    host_response_rate,
    host_acceptance_rate,
    host_listings_count,
    number_of_reviews,
    number_of_reviews_ltm,
    number_of_reviews_l30d,
    review_scores_rating,
    review_scores_value,
    availability_365,
    instant_bookable
  )
```

---

## 3. Data Cleaning and Type Conversion

```r
# Clean price, convert percentages, create factors, and derive occupancy
df_clean <- df %>%
  # 1) Clean price: remove currency symbols and commas, convert to numeric
  mutate(
    price_clean = str_replace_all(price, "[^0-9.]", ""),
    price_clean = as.numeric(price_clean)
  ) %>%
  # 2) Convert percentage strings (e.g. "95%") to numeric 0–100
  mutate(
    host_response_rate_num   = as.numeric(str_replace(host_response_rate, "%", "")),
    host_acceptance_rate_num = as.numeric(str_replace(host_acceptance_rate, "%", ""))
  ) %>%
  # 3) Convert booleans and categories to factors
  mutate(
    host_is_superhost      = as.factor(host_is_superhost),
    instant_bookable       = as.factor(instant_bookable),
    room_type              = as.factor(room_type),
    property_type          = as.factor(property_type),
    neighbourhood_cleansed = as.factor(neighbourhood_cleansed)
  ) %>%
  # 4) Create high_occupancy from availability_365 (<180 vs ≥180)
  mutate(
    high_occupancy = ifelse(availability_365 < 180, 1, 0),
    high_occupancy = as.factor(high_occupancy)
  )

# Handle missing values
# 1) Drop rows with missing price (target for regression)
df_clean <- df_clean %>%
  filter(!is.na(price_clean))

# 2) Median imputation for review scores
median_rating <- median(df_clean$review_scores_rating, na.rm = TRUE)
median_value  <- median(df_clean$review_scores_value,  na.rm = TRUE)

df_clean <- df_clean %>%
  mutate(
    review_scores_rating = ifelse(is.na(review_scores_rating),
                                  median_rating,
                                  review_scores_rating),
    review_scores_value  = ifelse(is.na(review_scores_value),
                                  median_value,
                                  review_scores_value)
  )

# Quick structure and missing check
str(df_clean)

colSums(is.na(df_clean[, c(
  "price_clean",
  "accommodates",
  "bathrooms",
  "bedrooms",
  "beds",
  "host_is_superhost",
  "host_response_rate_num",
  "host_acceptance_rate_num",
  "review_scores_rating",
  "review_scores_value",
  "availability_365",
  "instant_bookable"
)]))
```

---

## 4. Q1 – Price Regression

### 4.1 Data Preparation and Train–Test Split

```r
set.seed(123)

df_reg <- df_clean %>%
  select(
    price_clean,
    accommodates,
    bathrooms,
    bedrooms,
    beds,
    room_type,
    neighbourhood_cleansed
  ) %>%
  na.omit()

train_index <- createDataPartition(df_reg$price_clean, p = 0.7, list = FALSE)
train_reg   <- df_reg[train_index, ]
test_reg    <- df_reg[-train_index, ]
```

### 4.2 Linear Regression Model

```r
# Linear regression
fit_lm <- train(
  price_clean ~ .,
  data   = train_reg,
  method = "lm"
)

# Summary of linear model
summary(fit_lm$finalModel)

# Predictions and performance on test set
pred_price <- predict(fit_lm, newdata = test_reg)
reg_metrics <- postResample(pred_price, test_reg$price_clean)
reg_metrics
```

### 4.3 Decision Tree Regression (Optional)

```r
# Decision tree regression with cross-validation
fit_rpart <- train(
  price_clean ~ .,
  data   = train_reg,
  method = "rpart",
  trControl = trainControl(method = "cv", number = 5)
)

# Variable importance
varImp(fit_rpart)
```

---

## 5. Q2 – Superhost Classification

### 5.1 Data Preparation and Train–Test Split

```r
df_cls <- df_clean %>%
  select(
    host_is_superhost,
    host_response_rate_num,
    host_acceptance_rate_num,
    review_scores_rating,
    instant_bookable,
    number_of_reviews,
    host_listings_count
  ) %>%
  na.omit()

set.seed(123)

train_index_cls <- createDataPartition(df_cls$host_is_superhost, p = 0.7, list = FALSE)
train_cls <- df_cls[train_index_cls, ]
test_cls  <- df_cls[-train_index_cls, ]

# Ensure factor levels are consistent
train_cls$host_is_superhost <- factor(train_cls$host_is_superhost)
test_cls$host_is_superhost  <- factor(test_cls$host_is_superhost)
```

### 5.2 Train Logistic Regression with Up-sampling

```r
# Control with up-sampling and ROC summary
ctrl_up <- trainControl(
  method = "cv",
  number = 5,
  sampling = "up",
  classProbs = TRUE,
  summaryFunction = twoClassSummary
)

# Logistic regression model
fit_logit <- train(
  host_is_superhost ~ .,
  data   = train_cls,
  method = "glm",
  family = "binomial",
  trControl = ctrl_up,
  metric = "ROC"
)

# Predict probabilities and classes on test set
pred_prob <- predict(fit_logit, newdata = test_cls, type = "prob")
# Assume the positive class is the second level
pos_class <- levels(test_cls$host_is_superhost)[1]

pred_cls  <- ifelse(pred_prob[, pos_class] >= 0.5, pos_class,
                    levels(test_cls$host_is_superhost))[2]
pred_cls  <- factor(pred_cls, levels = levels(test_cls$host_is_superhost))

# Confusion matrix and metrics
cm_superhost <- confusionMatrix(pred_cls, test_cls$host_is_superhost,
                                positive = pos_class)
cm_superhost

# Variable importance
varImp(fit_logit)
```

---

## 6. Q3 – K-means Clustering (Market Segments)

### 6.1 Data Preparation and Scaling

```r
set.seed(123)

df_clust <- df_clean %>%
  select(
    price_clean,
    availability_365,
    number_of_reviews_l30d
  ) %>%
  na.omit()

# Scale features
df_clust_scaled <- scale(df_clust)
```

### 6.2 Elbow Method to Choose k

```r
wss <- sapply(2:6, function(k) {
  kmeans(df_clust_scaled, centers = k, nstart = 20)$tot.withinss
})
wss

# Optional: plot elbow curve
plot(2:6, wss, type = "b",
     xlab = "Number of clusters k",
     ylab = "Total within-cluster sum of squares")
```

### 6.3 Fit Final K-means Model

```r
# Example: choose k = 3 (adjust if you decide otherwise)
k <- 3
km3 <- kmeans(df_clust_scaled, centers = k, nstart = 20)

# Attach cluster labels
df_clust_res <- df_clust %>%
  mutate(cluster = factor(km3$cluster))

# Summarise clusters
cluster_summary <- df_clust_res %>%
  group_by(cluster) %>%
  summarise(
    avg_price            = mean(price_clean),
    avg_availability_365 = mean(availability_365),
    avg_reviews_30d      = mean(number_of_reviews_l30d),
    n_listings           = n(),
    .groups = "drop"
  )

cluster_summary
```

---

## 7. Q4 – Instant Book vs Occupancy

### 7.1 Data Preparation and Train–Test Split

```r
df_ib <- df_clean %>%
  select(
    high_occupancy,
    instant_bookable,
    price_clean,
    review_scores_value
  ) %>%
  na.omit()

set.seed(123)

train_index_ib <- createDataPartition(df_ib$high_occupancy, p = 0.7, list = FALSE)
train_ib <- df_ib[train_index_ib, ]
test_ib  <- df_ib[-train_index_ib, ]

train_ib$high_occupancy <- factor(train_ib$high_occupancy)
test_ib$high_occupancy  <- factor(test_ib$high_occupancy)
```

### 7.2 Logistic Regression Model for Occupancy

```r
ctrl_ib <- trainControl(
  method = "cv",
  number = 5,
  classProbs = TRUE,
  summaryFunction = twoClassSummary
)

fit_ib <- train(
  high_occupancy ~ .,
  data   = train_ib,
  method = "glm",
  family = "binomial",
  trControl = ctrl_ib,
  metric = "ROC"
)

# Predictions and confusion matrix
pred_ib <- predict(fit_ib, newdata = test_ib)
cm_ib   <- confusionMatrix(pred_ib, test_ib$high_occupancy)
cm_ib

# Variable importance (effect of instant_bookable vs controls)
varImp(fit_ib)
```

---

## 8. Simple Descriptive Comparison: Instant Bookable vs Non-Instant

```r
ib_summary <- df_clean %>%
  group_by(instant_bookable) %>%
  summarise(
    avg_availability_365 = mean(availability_365, na.rm = TRUE),
    pct_high_occupancy   = mean(high_occupancy == 1, na.rm = TRUE),
    n_listings           = n(),
    .groups = "drop"
  )

ib_summary
```

---

End of R code file.
