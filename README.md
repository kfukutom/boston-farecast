By: Ken Fukutomi  
Email: [kfukutom@umich.edu](mailto:kfukutom@umich.edu)  
Course: EECS 398 - Final Project  
Repository: [https://github.com/kfukutom/boston-farecast](https://github.com/kfukutom/boston-farecast)

---

## Introduction

This project investigates the dynamics of rideshare pricing in Boston, MA using a dataset of nearly 700,000 combined Uber and Lyft trips. With the growing use of on-demand mobility services, understanding how fare pricing works — and what influences it — is crucial for both riders and active municipal officials. By analyzing this synergy, I aim to explore the variables that impact trip time, quality, and pricing — and ultimately generate insights that can power fare prediction at scale.

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

When first cleaning my data, I noticed the dataset came with a ton of information — which initially felt promising for predictive modeling. Features like trip origin and destination, cab type, ride distance, datetime, and weather conditions (UV index, wind strength, etc.) stood out as useful. However, there were also duplicated or redundant columns, some of which needed cleaning due to encoding issues or lack of unique value. Using regex and column comparisons, I removed these duplicates.

One of the biggest challenges was missing values in the `price` column. I asked:  
(1) Are the prices missing due to canceled or invalid trips?  
(2) Are the trips long enough to expect pricing?  
The answer to both was yes — the trips were valid. So I applied two imputation strategies: one based on probabilistically sampling from observed prices, and another using median values from distance-based bins. Both were reasonable, but I moved forward with the probabilistic approach.

*Here is the visualization comparing the original prices with both imputed strategies:*

<iframe src="assets/fare-imputation-comparison.html" width="800" height="425" frameborder="0"></iframe>

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

To finalize my cleaning, I dropped unnecessary columns that had minimal or no impact on our hypothesis or predictive model. These included variables that were redundant, improperly formatted, or unrelated to fare prediction — such as `timezone`, `icon`, `summary`, `sunriseTime`, `sunsetTime`, and several detailed weather-related timestamp fields (e.g., `temperatureHighTime`, `uvIndexTime`).

*Here’s a preview of the cleaned DataFrame:*

|   hour |   distance |   price | cab_type   | name         | destination   |
|-------:|-----------:|--------:|:-----------|:-------------|:--------------|
|      9 |       0.44 |       5 | Lyft       | Shared       | North Station |
|      2 |       0.44 |      11 | Lyft       | Lux          | North Station |
|      1 |       0.44 |       7 | Lyft       | Lyft         | North Station |
|      4 |       0.44 |      26 | Lyft       | Lux Black XL | North Station |
|      3 |       0.44 |       9 | Lyft       | Lyft XL      | North Station |

### Univariate Analysis

The plot below shows the *log-normalized distribution* of trip distances before and after missing values were imputed. Most trips in the dataset remain clustered at shorter distances (under 5 miles), and the alignment between the original and imputed distributions confirms that the imputation process preserved the shape and scale of the data. This supports the assumption that filling missing `distance` values using a probabilistic approach won’t distort downstream modeling.

<iframe src="assets/distance.html" width="800" height="400" frameborder="0"></iframe>

<iframe src="assets/log.html" width="800" height="400" frameborder="0"></iframe>

### Bivariate Analysis and Aggregates



<!-- Load MathJax for LaTeX rendering -->
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
