# Appendix C – ggplot2 Code for Figures

This appendix contains ggplot2 code for the main plots used in the report and presentation:
- Price distribution
- Room type and Superhost composition
- Price vs room type
- Variable importance (price and Superhost models)
- K-means elbow and cluster scatter
- Instant Book vs high occupancy

Assumes:
- `df_clean` is prepared as in Appendix A.
- Models `fit_rpart`, `fit_logit` are fitted as in Appendix B.

```r
library(ggplot2)
library(dplyr)
library(caret)
library(tibble)
library(scales)
```

---

## C.1 Price Distribution – Histogram

```r
ggplot(df_clean, aes(x = price_clean)) +
  geom_histogram(binwidth = 200, fill = "steelblue", color = "white") +
  labs(
    title = "Distribution of Listing Prices (HKD)",
    x = "Price (HKD)",
    y = "Count"
  ) +
  theme_minimal()
```

Optional log-scale version:

```r
ggplot(df_clean, aes(x = price_clean)) +
  geom_histogram(binwidth = 0.1, fill = "steelblue", color = "white") +
  scale_x_log10() +
  labs(
    title = "Distribution of Listing Prices (log scale)",
    x = "log(Price)",
    y = "Count"
  ) +
  theme_minimal()
```

---

## C.2 Room Type Composition – Bar Chart

```r
ggplot(df_clean, aes(x = room_type)) +
  geom_bar(fill = "darkorange") +
  labs(
    title = "Distribution of Room Types",
    x = "Room Type",
    y = "Number of Listings"
  ) +
  theme_minimal()
```

---

## C.3 Superhost Proportion – Bar Chart

```r
ggplot(df_clean, aes(x = host_is_superhost)) +
  geom_bar(fill = "seagreen") +
  labs(
    title = "Superhost vs Non-Superhost Listings",
    x = "Host is Superhost",
    y = "Number of Listings"
  ) +
  theme_minimal()
```

---

## C.4 Price by Room Type – Boxplot

```r
ggplot(df_clean, aes(x = room_type, y = price_clean)) +
  geom_boxplot(fill = "skyblue") +
  labs(
    title = "Listing Price by Room Type",
    x = "Room Type",
    y = "Price (HKD)"
  ) +
  coord_cartesian(ylim = c(0, quantile(df_clean$price_clean, 0.95, na.rm = TRUE))) +
  theme_minimal()
```

The `coord_cartesian` call trims extreme prices (top 5%) so boxes are clearer.

---

## C.5 Variable Importance – Price Regression (Tree Model)

Assumes you fitted a decision tree regression `fit_rpart` with caret.

```r
imp_price <- varImp(fit_rpart)$importance %>%
  rownames_to_column("variable") %>%
  arrange(desc(Overall))

ggplot(imp_price, aes(x = reorder(variable, Overall), y = Overall)) +
  geom_col(fill = "steelblue") +
  coord_flip() +
  labs(
    title = "Variable Importance – Price Regression (Tree)",
    x = "Variable",
    y = "Importance"
  ) +
  theme_minimal()
```

---

## C.6 Variable Importance – Superhost Classification

Assumes you fitted a logistic regression `fit_logit` with caret.

```r
imp_superhost <- varImp(fit_logit)$importance %>%
  rownames_to_column("variable") %>%
  arrange(desc(Overall))

ggplot(imp_superhost, aes(x = reorder(variable, Overall), y = Overall)) +
  geom_col(fill = "darkorange") +
  coord_flip() +
  labs(
    title = "Variable Importance – Superhost Classification",
    x = "Variable",
    y = "Importance"
  ) +
  theme_minimal()
```

---

## C.7 Elbow Plot – K-means Clustering

Assumes you computed `wss` for k = 2:6.

```r
k_values <- 2:6
elbow_df <- data.frame(
  k = k_values,
  wss = wss
)

ggplot(elbow_df, aes(x = k, y = wss)) +
  geom_line(color = "steelblue") +
  geom_point(color = "steelblue") +
  labs(
    title = "Elbow Plot for K-means Clustering",
    x = "Number of Clusters (k)",
    y = "Total Within-Cluster Sum of Squares"
  ) +
  theme_minimal()
```

---

## C.8 Cluster Scatter Plot – Price vs Recent Reviews

Assumes you created `df_clust_res` with `cluster` as a factor.

```r
ggplot(df_clust_res,
       aes(x = price_clean,
           y = number_of_reviews_l30d,
           color = cluster)) +
  geom_point(alpha = 0.6) +
  labs(
    title = "Clusters of Listings by Price and Recent Reviews",
    x = "Price (HKD)",
    y = "Number of Reviews in Last 30 Days",
    color = "Cluster"
  ) +
  theme_minimal()
```

You can instead put `availability_365` on the y-axis if that tells a clearer story.

---

## C.9 Instant Book vs High Occupancy – Bar Chart

```r
ib_summary <- df_clean %>%
  group_by(instant_bookable) %>%
  summarise(
    pct_high_occupancy = mean(high_occupancy == 1, na.rm = TRUE),
    .groups = "drop"
  )

ggplot(ib_summary, aes(x = instant_bookable, y = pct_high_occupancy)) +
  geom_col(fill = "seagreen") +
  scale_y_continuous(labels = percent_format()) +
  labs(
    title = "Share of High-Occupancy Listings by Instant Bookable",
    x = "Instant Bookable",
    y = "Percentage of High-Occupancy Listings"
  ) +
  theme_minimal()
```

End of Appendix C.
