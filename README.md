By: Ken Fukutomi  
Email: [kfukutom@umich.edu](mailto:kfukutom@umich.edu)  
Course: EECS 398 - Final Project  
Repository: [https://github.com/kfukutom/boston-farecast](https://github.com/kfukutom/boston-farecast)

---

## Introduction

This project investigates the dynamics of rideshare pricing in Boston, MA using a dataset of nearly 700,000 combined Uber and Lyft trips. With the growing use of on-demand mobility services, understanding how fare pricing works - and what influences it - is crucial for both riders and active municipal officials. By analyzing this synergy, I aim to explore the variables that impact trip time, quality, and pricing - and ultimately generate insights that can power fare prediction at scale.

The key question guiding this project is:  
**"What factors most strongly influence trip price, and how accurately can we predict ridefare using those variables?"**

Several real-world questions naturally emerge while exploring the data:

- During peak hours, should I choose Uber or Lyft to minimize my fare?  
- How does surge pricing vary by time of day and service type?  
- What is the relationship between trip distance and fare for each cab type?  
- Do weather conditions (e.g., temperature, UV index) affect average fares?  
- Which trip origins tend to have higher or lower ride costs?

Dataset overview:  
- Number of rows: 693,071  
- Relevant columns:
  - `timestamp`, `hour`: When the ride occurred  
  - `cab_type`: Whether the ride was an Uber or Lyft  
  - `price`: The cost of the trip  
  - `surge_multiplier`: Whether surge pricing was active  
  - `distance`: The length of the trip in miles  
  - `source`, `destination`: Start and end neighborhoods  
  - `product_id`: Type of ride service (e.g. Shared, XL, Comfort)

---

## Data Cleaning and Exploratory Data Analysis

### Preprocessing + Cleaning

When first cleaning my data, I noticed the dataset came with a ton of information - which initially felt promising for predictive modeling. Features like trip origin and destination, cab type, ride distance, datetime, and weather conditions (UV index, wind strength, etc.) stood out as useful. However, there were also duplicated or redundant columns, some of which needed cleaning due to encoding issues or lack of unique value. Using regex and column comparisons, I removed these duplicates.

One of the biggest challenges was missing values in the `price` column. I asked:  
(1) Are the prices missing due to canceled or invalid trips?  
(2) Are the trips long enough to expect pricing?  
The answer to both was yes - the trips were valid. So I applied two imputation strategies: one based on probabilistically sampling from observed prices, and another using median values from distance-based bins. Both were reasonable, but I moved forward with the probabilistic approach.

*Here is the visualization comparing the original prices with both imputed strategies:*

<iframe src="assets/fare-imputation-comparison.html" width="800" height="425" frameborder="0"></iframe>

---

The probabilistic imputation strategy samples missing values directly from the empirical distribution of observed values:

$$
x_i^{(\mathrm{imputed})} = x_j,\quad
j \sim \mathrm{Uniform}\left(\{k \mid x_k \neq \mathrm{NaN}\}\right)
$$

Equivalently, each observed value has equal probability:

$$
P\left(x_i^{(\mathrm{imputed})} = x_j\right)
= \frac{1}{n_{\mathrm{observed}}}
$$

Since the two imputation methods yielded nearly identical distributions and the percentage of missing data was low, I chose the probabilistic method. While I could have left `NaN` values in place, having a complete column ensures better performance in downstream modeling tasks.

To finalize my cleaning, I dropped unnecessary columns that had minimal or no impact on our hypothesis or predictive model. These included variables that were redundant, improperly formatted, or unrelated to fare prediction - such as `timezone`, `icon`, `summary`, `sunriseTime`, `sunsetTime`, and several detailed weather-related timestamp fields (e.g., `temperatureHighTime`, `uvIndexTime`).

*Here’s a preview of the cleaned DataFrame:*

|   hour |   distance |   price | cab_type   | name         | destination   |
|-------:|-----------:|--------:|:-----------|:-------------|:--------------|
|      9 |       0.44 |       5 | Lyft       | Shared       | North Station |
|      2 |       0.44 |      11 | Lyft       | Lux          | North Station |
|      1 |       0.44 |       7 | Lyft       | Lyft         | North Station |
|      4 |       0.44 |      26 | Lyft       | Lux Black XL | North Station |
|      3 |       0.44 |       9 | Lyft       | Lyft XL      | North Station |

---

### Univariate Analysis

The plot below shows the *log-normalized distribution* of trip distances before and after missing values were imputed. Most trips in the dataset remain clustered at shorter distances (under 5 miles), and the alignment between the original and imputed distributions confirms that the imputation process preserved the shape and scale of the data. This supports the assumption that filling missing `distance` values using a probabilistic approach won’t distort downstream modeling.

<iframe src="assets/distance.html" width="800" height="400" frameborder="0"></iframe>

<iframe src="assets/log.html" width="800" height="400" frameborder="0"></iframe>

I experimented with the imputed results and found no significant difference between the two methods. Additionally, none of the imputed price values were below zero, and since the distribution wasn’t strongly right-skewed, I decided to move forward using the original (non-log-transformed) pricing data. I also created a geospatial visualization of trip distributions, shown below:

<iframe src="assets/trip-frequency-map.html" width="800" height="400" frameborder="0"></iframe>

---

### Bivariate Analysis and Aggregates

To better understand relationships between continuous variables, I created a correlation heatmap across numerical features in the dataset. This visualization helps reveal multicollinearity and patterns that may not be immediately obvious from scatterplots alone. Notably, variables like `temperature` and `apparentTemperature` show strong positive correlation, as expected, while `distance` and `price` also share a moderate positive relationship - reinforcing their usefulness in fare prediction.

<iframe src="assets/corr.html" width="800" height="400" frameborder="0"></iframe>

#### An Interesting Aggregate

*Lyft Data Table*

| destination       | cab_type   |   price_mean |   price_median |   distance_mean |   distance_median |
|:------------------|:-----------|-------------:|---------------:|----------------:|------------------:|
| Boston University | Lyft       |        20.32 |           19.5 |            3.18 |              3.07 |
| Back Bay          | Lyft       |        16.89 |           16.5 |            2.07 |              1.97 |
| North Station     | Lyft       |        17.77 |           16.5 |            2.26 |              3.17 |
| Beacon Hill       | Lyft       |        16.88 |           16.5 |            2.19 |              2.42 |
| West End          | Lyft       |        17.08 |           16.5 |            2.12 |              2.77 |

*Uber Data Table*

| destination             | cab_type   |   price_mean |   price_median |   distance_mean |   distance_median |
|:------------------------|:-----------|-------------:|---------------:|----------------:|------------------:|
| Boston University       | Uber       |        17.5  |           15   |            2.88 |              2.8  |
| Fenway                  | Uber       |        16.96 |           14   |            2.79 |              2.84 |
| Northeastern University | Uber       |        16.9  |           14   |            2.66 |              2.56 |
| Financial District      | Uber       |        17.09 |           13.5 |            2.64 |              1.22 |
| Back Bay                | Uber       |        15.69 |           13   |            2.1  |              1.78 |

I’ve sorted + aggregated both tables in descending order by average fare, which immediately highlights that the top three destinations differ between Uber and Lyft. This divergence underscores how each service’s pricing varies by neighborhood and reinforces the importance of including destination as a feature in our predictive model-mean price alone already reveals distinct patterns that our model can learn from!

---

## Framing a Prediction Problem

I'm tackling a regression task: predicting the ride fare price (`price`) for each trip in Boston.  

- Response variable: `price`-this is the value riders see and platforms optimize for, and it must be known at booking time.  
- Features used: only data available when the ride is requested-trip distance, pickup hour/day/month, surge multiplier, cab type & service tier, origin & destination neighborhoods, and forecasted weather metrics. No in-ride or post-trip information is included to avoid leakage.  
- Evaluation metric: RMSE as our primary score (to heavily penalize large dollar‐value mistakes) and mean absolute error for a straightforward average-error interpretation in dollars.  

By restricting inputs and using RMSE/MAE, I can ensure that my model mimics real-world fare estimation and prioritizes minimizing costly prediction outliers.

---

## Baseline Model

<!-- Load MathJax for LaTeX rendering -->
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
