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

Since the two imputation methods yielded nearly identical distributions and the percentage of missing data was low, I chose the probabilistic method. While I could have left `NaN` values in place, having a complete column ensures better performance in downstream modeling tasks. To finalize my cleaning, I dropped every other unnecessary column that might lead to minimal or no impact on our hypothesis or predictive model. These included variables that were either redundant, improperly formatted, or irrelevant to the pricing task — such as `timezone`, `icon`, `summary`, `sunriseTime`, `sunsetTime`, and several detailed weather-related time columns (e.g., `temperatureHighTime`, `uvIndexTime`). *Here's my finalized dataframe.head():*

|   hour |   day |   month | datetime                      | source           | destination   | cab_type   | name         |   price |   distance |   surge_multiplier |   latitude |   longitude |   temperature |   apparentTemperature |   precipIntensity |   precipProbability |   windSpeed |   windGust |   visibility |   apparentTemperatureHigh |   apparentTemperatureHighTime |   apparentTemperatureLow |   apparentTemperatureLowTime | icon                |   pressure |   windBearing |   cloudCover |   sunriseTime |   sunsetTime |   precipIntensityMax |   uvIndexTime |   temperatureMin |   temperatureMax |   temperatureMaxTime | dist_bin      |   most_common |
|-------:|------:|--------:|:------------------------------|:-----------------|:--------------|:-----------|:-------------|--------:|-----------:|-------------------:|-----------:|------------:|--------------:|----------------------:|------------------:|--------------------:|------------:|-----------:|-------------:|--------------------------:|------------------------------:|-------------------------:|-----------------------------:|:--------------------|-----------:|--------------:|-------------:|--------------:|-------------:|---------------------:|--------------:|-----------------:|-----------------:|---------------------:|:--------------|--------------:|
|      9 |    16 |      12 | 2018-12-16 09:30:07.890000128 | Haymarket Square | North Station | Lyft       | Shared       |       5 |       0.44 |                  1 |    42.2148 |     -71.033 |         42.34 |                 37.12 |            0      |                   0 |        8.66 |       9.17 |       10     |                     37.95 |                    1544968800 |                    27.39 |                   1545044400 | partly-cloudy-night |    1021.98 |            57 |         0.72 |    1544962084 |   1544994864 |               0.1276 |    1544979600 |            39.89 |            43.68 |           1544968800 | (0.019, 1.16] |             0 |
|      2 |    27 |      11 | 2018-11-27 02:00:23.676999936 | Haymarket Square | North Station | Lyft       | Lux          |      11 |       0.44 |                  1 |    42.2148 |     -71.033 |         43.58 |                 37.35 |            0.1299 |                   1 |       11.98 |      11.98 |        4.786 |                     43.92 |                    1543251600 |                    36.2  |                   1543291200 | rain                |    1003.97 |            90 |         1    |    1543232969 |   1543266992 |               0.13   |    1543251600 |            40.49 |            47.3  |           1543251600 | (0.019, 1.16] |             0 |
|      1 |    28 |      11 | 2018-11-28 01:00:22.197999872 | Haymarket Square | North Station | Lyft       | Lyft         |       7 |       0.44 |                  1 |    42.2148 |     -71.033 |         38.33 |                 32.93 |            0      |                   0 |        7.33 |       7.33 |       10     |                     44.12 |                    1543320000 |                    29.11 |                   1543392000 | clear-night         |     992.28 |           240 |         0.03 |    1543319437 |   1543353364 |               0.1064 |    1543338000 |            35.36 |            47.55 |           1543320000 | (0.019, 1.16] |             0 |
|      4 |    30 |      11 | 2018-11-30 04:53:02.749000192 | Haymarket Square | North Station | Lyft       | Lux Black XL |      26 |       0.44 |                  1 |    42.2148 |     -71.033 |         34.38 |                 29.63 |            0      |                   0 |        5.28 |       5.28 |       10     |                     38.53 |                    1543510800 |                    26.2  |                   1543575600 | clear-night         |    1013.73 |           310 |         0    |    1543492370 |   1543526114 |               0      |    1543507200 |            34.67 |            45.03 |           1543510800 | (0.019, 1.16] |             0 |
|      3 |    29 |      11 | 2018-11-29 03:49:20.223000064 | Haymarket Square | North Station | Lyft       | Lyft XL      |       9 |       0.44 |                  1 |    42.2148 |     -71.033 |         37.44 |                 30.88 |            0      |                   0 |        9.14 |       9.14 |       10     |                     35.75 |                    1543420800 |                    30.29 |                   1543460400 | partly-cloudy-night |     998.36 |           303 |         0.44 |    1543405904 |   1543439738 |               0.0001 |    1543420800 |            33.1  |            42.18 |           1543420800 | (0.019, 1.16] |             0 |

### Univariate Analysis

Testing here

### Bivariate Analysis and Aggregates

Hello world!

<!-- Load MathJax for LaTeX rendering -->
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
