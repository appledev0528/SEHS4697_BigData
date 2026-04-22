# Appendix C – Figures (ggplot2 Code, Titles, and Captions

This appendix contains ggplot2 code and the corresponding figure titles and captions used in the report and presentation.

Assumptions:
- `df_clean` is prepared as in Appendix A.
- Models `fit_rpart` (price tree) and `fit_logit` (Superhost logistic) are fitted as in Appendix B.

```r
library(ggplot2)
library(dplyr)
library(caret)
library(tibble)
library(scales)
```

---

## Figure 1 – Price Distribution

**Figure title**  
Figure 1. Distribution of Airbnb listing prices in Hong Kong.

**Caption**  
This figure shows the distribution of cleaned listing prices in Hong Kong on the Inside Airbnb dataset, highlighting the overall price range and skewness of the market. It motivates the use of regression techniques to model price and potentially transform the variable when building predictive models.

**Code**

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

## Figure 2 – Room Type Composition

**Figure title**  
Figure 2. Distribution of room types among Hong Kong listings.

**Caption**  
This figure displays the counts of different room types (e.g. entire home/apt, private room) in the Hong Kong Airbnb dataset. It illustrates that entire homes and private rooms dominate the market, justifying the inclusion of room_type as a key predictor of price.

**Code**

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

## Figure 3 – Superhost Proportion

**Figure title**  
Figure 3. Proportion of Superhost vs non-Superhost listings.

**Caption**  
This figure visualizes the number of listings belonging to Superhosts compared with non-Superhosts in the dataset. The clear minority share of Superhosts supports the need for up-sampling techniques when training the Superhost classification model.

**Code**

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

## Figure 4 – Price by Room Type

**Figure title**  
Figure 4. Listing prices by room type.

**Caption**  
This figure compares the distribution of prices across room types, showing that entire homes and apartments tend to command higher prices than private rooms. The observed differences provide early evidence that room_type is an important driver of price and should be included in the regression model.

**Code**

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

---

## Figure 5 – Variable Importance for Price Model

**Figure title**  
Figure 5. Variable importance in the price regression tree model.

**Caption**  
This figure presents the relative importance of predictors such as accommodates, room_type, and neighbourhood_cleansed in the decision tree regression model for price. It indicates which listing characteristics contribute most to explaining price differences across properties.

**Code**

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

## Figure 6 – Variable Importance for Superhost Model

**Figure title**  
Figure 6. Variable importance in the Superhost classification model.

**Caption**  
This figure shows the most influential variables for predicting Superhost status, including host_response_rate, host_acceptance_rate, review_scores_rating, and number_of_reviews. The results highlight the key host behaviors and listing attributes associated with achieving Superhost recognition.

**Code**

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

## Figure 7 – Elbow Plot for K-means

**Figure title**  
Figure 7. Elbow plot for selecting the number of clusters.

**Caption**  
This figure plots the total within-cluster sum of squares against the number of clusters k for the K-means algorithm applied to price, availability_365, and number_of_reviews_l30d. The “elbow” point guides the choice of an appropriate number of clusters that balances model simplicity and segment separation.

**Code**

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

## Figure 8 – Cluster Scatter Plot

**Figure title**  
Figure 8. Clusters of listings by price and recent review activity.

**Caption**  
This figure plots listings by price and number_of_reviews_l30d, colored by their assigned K-means cluster. It illustrates distinct segments such as budget high-demand listings and premium lower-demand listings that can be targeted with different marketing strategies.

**Code**

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

---

## Figure 9 – Instant Book vs High Occupancy

**Figure title**  
Figure 9. Share of high-occupancy listings by Instant Bookable status.

**Caption**  
This figure compares the proportion of high-occupancy listings between Instant Bookable and non-Instant Bookable properties, based on the derived high_occupancy variable from availability_365. The observed differences support the investigation of Instant Bookable as an important factor in the occupancy classification model.

**Code**

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
