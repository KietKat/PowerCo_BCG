# PowerCo_BCG
This project is from Boston Consulting Group, tasking to help PowerCo to understand customer churn.

## Problem Statement
PowerCo is observing a number of their customers unsubscribe from their service to use another company's instead- customer churn. They want to suggest whether discounting is a good way to prevent this and how they should target the audience. Decomposing their request, I am trying to answer two questions:
* Is price a driving factor leading to customer churn? - In other words, calculating price sensitivity
* What critical features can we use to predict the customer churn trend?
  
-> Client and price data will be necessary to answer these questions.

## Data Analysis
Some initial views about our data:
* The `churn` rate is about `9.7%` across 14,606 customers, which is quite high. **image1**
* The distribution of `churn` customers does not quite depend on whether they subscribed to gas or not, as they are pretty much the same **image2**
* All `churn` customers subscribed to less than **5** of our services **image3**
* All `churn` customers  distribute across **5** of our sale channel. The `MISSING` channel means no information available **image4**

For features related to consumption and price, the plot is highly skewed and spread out because the values are too big. We can clean these data in the data engineering step. **image5** 

## Feature Engineering
### Creating new features:
Given the information that customer churn during the month of December after seeing the change in price of December, we proceed to add the following features as they are directly involved in this claim:
* `peak_diff_dec_january_energy`: the energy price difference between the peak period of Jan and Dec
* `peak_diff_dec_january_power`: the power price difference between the peak period of Jan and Dec
* `midpeak_diff_dec_january_energy`: the energy price difference between the mid-peak period of Jan and Dec
* `midpeak_diff_dec_january_power`: the power price difference between the mid-peak period of Jan and Dec
* `offpeak_diff_dec_january_energy`: the energy price difference between the off-peak period of Jan and Dec
* `offpeak_diff_dec_january_power`: the energy price difference between the off-peak period of Jan and Dec

Standing on a customer's viewpoint, I am very likely to change to another service if the price fluctuates too much within a month. We proceed to add the following features to our analysis.
* `max_diff_offpeak_peak_var`: the max difference in price between off-peak and peak periods within a month (var)
* `max_diff_offpeak_midpeak_var`: the max difference in price between off-peak and mid-peak periods within a month (var)
* `max_diff_midpeak_peak_var`: the max difference in price between mid-peak and peak period within a month (var)
* `max_diff_offpeak_peak_fix`: the max difference in price between off-peak and peak periods within a month (fix)
* `max_diff_offpeak_midpeak_fix`: the max difference in price between off-peak and mid-peak periods within a month (fix)
* `max_diff_midpeak_peak_fix`: the max difference in price between mid-peak and peak periods within a month (fix)

Intuitively, how long a customer has been with the company is also a good metric. Therefore, we add `tenure` as well.

### Converting numerical features:
As discussed above, the consumption-related features and price_feature are widely spread and skewed. Therefore, we take their `log10` to make it more feasible for the later model. The list of features are:
* Consumptions: `cons_12m`,`cons_gas_12m`, `cons_last_month`, `forecast_cons_12m`, `forecast_cons_year`. 
* Price: `forecast_discount_energy`, `forecast_meter_rent_12m`, `forecast_price_energy_off_peak`, `forecast_price_energy_peak`,`
forecast_price_pow_off_peak`

### Converting categorical features:
These columns are converted into numerical data to be more feasible for the machine: `origin_up,` `channel_sales,` and `churn` using `OneHotEncoder.`

### Dropping data:
`DateTime` features are not feasible for the model, so we proceed to drop: `date_activ,` `date_end,` `date_modif_prod,` and `date_renewal.` We also proceeded to fill out the NA with 0.

Here is the final correlation heatmap: **image6**

## Modeling

## Conclusion
