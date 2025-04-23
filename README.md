By: Ken Fukutomi  
Email: [kfukutom@umich.edu](mailto:kfukutom@umich.edu)  
Course: EECS 398 - Final Project  
Repository: [https://github.com/kfukutom/boston-farecast](https://github.com/kfukutom/boston-farecast) 

---

## Introduction 

This project investigates the dynamics of rideshare pricing in Boston, MA using a dataset of nearly 700,000 combined Uber and Lyft trips. With the growing use of on-demand mobility services, understanding how fare pricing works - and what influences it - is crucial for both riders and active municipal officials. By understanding this kind of synergy, I aim to try and understand what kinds of variables impact such delays in trip time, quality, and how such information could lead to a novel insight into accurate predictions of trip fares on such large dataset.

The key question guiding this project is:  
**"What factors most strongly influence trip price, and how accurately can we predict ridefare using those variables?"**

This question is not only practically valuable for cost-conscious riders, but also relevant for understanding traffic demand, pricing models, and urban mobility patterns in a major U.S. city (as such, Boston MA).

Several real-world questions naturally emerge while exploring the data:

- During peak hours, should I choose Uber or Lyft to minimize my fare?  
- How does surge pricing vary by time of day and service type?  
- What is the relationship between trip distance and fare for each cab type?  
- Do weather conditions (e.g., temperature, UV index) affect average fares?  
- Which trip origins tend to have higher or lower ride costs?  

For the predictive portion of this project, I will first establish a baseline model, and then refine it to improve accuracy - allowing us to predict trip fares based on contextual factors like time, distance, service type, and location.

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

When first cleaning my data, I noticed the dataset came with a ton of information — which at first felt promising for predictive modeling. Features like trip origin and destination, cab type, ride distance, datetime, and even weather conditions (UV index, wind strength, etc.) stood out as potentially useful. However, there were also a number of duplicated or redundant columns, some of which needed cleaning due to encoding issues or lack of unique value. Using regex and column comparisons, I dropped these duplicates.

One of the biggest challenges was missing values in the `price` column. To address this, I asked: (1) Are the prices missing because of trip cancellations? (2) Are the distances valid enough to expect a fare? The answer to both was yes — the trips were valid. So I applied two imputation strategies: one based on probabilistically sampling the price distribution, and another based on distance quantile bins. Both were reasonable, but I ultimately selected the probabilistic approach.

Here is the visualization showcasing the differences between the original and imputed price distributions:

<iframe src="assets/fare-imputation-comparison.html" width="800" height="600" frameborder="0"></iframe>

Given that the two imputation methods yield nearly identical distributions and the overall percentage of missing values is low, we can safely conclude that the choice of imputation strategy will not materially affect our analysis. Therefore, we adopt a probabilistic imputation approach, sampling each missing value from the empirical distribution of observed values:

$$
x_i^{(\mathrm{imputed})} = x_j,\quad
j \sim \mathrm{Uniform}\bigl(\{k \mid x_k \neq \mathrm{NaN}\}\bigr).
$$

Equivalently, each observed value has equal probability:

$$
P\bigl(x_i^{(\mathrm{imputed})} = x_j\bigr)
= \frac{1}{n_{\mathrm{observed}}},
$$

Regardless, my selection for which method doesn't affect the overall distribution. Hence, I could've just kept variables as NaN, but I preferred to have it imputed, in order to have a full column of information (pricing, as it becomes relevant later).

