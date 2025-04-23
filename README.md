By: Ken Fukutomi 
Email: [kfukutom@umich.edu](mailto:kfukutom@umich.edu)  
Course: EECS 398 — Final Project  
Website: https://kfukutom.github.io/boston-farecast/

---

## Introduction 

This project investigates the dynamics of rideshare pricing in Boston, MA using a dataset of nearly 700,000 combined Uber and Lyft trips. With the growing use of on-demand mobility services, understanding how fare pricing works - and what influences it — is crucial for both riders and active municipal officials. By understanding this kind of synergy, I aim to try and understand what kinds of variables impact such delays in trip time, quality, and how such information could lead to a novel insight into accurate predictions of trip fares on such large dataset.

The key question guiding this project is:  
**What factors most strongly influence trip price, and how accurately can we predict ridefare using those variables?**

This question is not only practically valuable for cost-conscious riders, but also relevant for understanding traffic demand, pricing models, and urban mobility patterns in a major U.S. city (as such, Boston MA).

Several real-world questions naturally emerge while exploring the data:

- During peak hours, should I choose Uber or Lyft to minimize my fare?  
- How does surge pricing vary by time of day and service type?  
- What is the relationship between trip distance and fare for each cab type?  
- Do weather conditions (e.g., temperature, UV index) affect average fares?  
- Which trip origins tend to have higher or lower ride costs?  

For the predictive portion of this project, I will first establish a baseline model, and then refine it to improve accuracy — allowing us to predict trip fares based on contextual factors like time, distance, service type, and location.

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
