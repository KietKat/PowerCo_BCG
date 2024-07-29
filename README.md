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
### Process:
After the feature engineering process, we are following this pipeline:
* Train-split text: We do 70-30 train-test split
* Standardize features: This step is necessary, as even though consumptions/price fields are transformed, there is still a lot of variation.
* Modeling: We are using a Random Forest Classifier. We are able to achieve a `91%` accuracy score on 10-fold cross-validation and `90%` on the test data.
  * We achieve `75%` on precision, which is how well we can label `churn` customers, and `97.8%` on recall, which indicates how well we retrieved `churn` customers.
  * In this task, I believe `recall` is a better metric to evaluate our model, as it does not hurt to misclassify `non-churn` customers to `churn` customers.
  * F1-score is `0.0402`, which is super good.
* Feature Importance: After training our model, we can find the importance of each feature in helping the model correctly predict `churn.` The top 5 driving factors are:
  * `cons_12m`
  * `forecast_meter_rent_12m`
  * `net_margin`
  * `forecast_cons_12m`
  * `margin_gross_pow_ele`
  * **image7**
 
### Model optimization:
The following approaches can be conducted to help improve our model:
* **Feature Selection**: We can deliberately choose features that contribute a lot to our model in the above picture (`>=0.01`). This improves `precision` to `82.9%` but decreases `recall` to `96.7%`, which I think is worth the trade-off.
* **PCA**: This method is relevant in reducing the dimension of our fields. With around `20-30` features, the space of `60` features can be almost `100%` explained.
* **image8/9**

### Note:
Even though the `churn` field is highly skewed, we should not oversample the minor group or undersample the majority group. While testing this approach, I was able to achieve a `95%` average accuracy rate on the cross-validation set but very poorly on the test set.
* The reason is that we should have the assumption that `churn` rate would be much lower than `non-churn`, for the sake of the company
* In Bayesian statistics, such "assumption" is the likelihood distribution, and if we decide to treat the `churn vs non-churn` as `50-50`, the posterior distribution will be affected

## Conclusion
We are answering the following questions:
* "What is the reality of `churn` customer trend?" -> It is high, at `9.7%` over `14606` customers
* "Is price a driving factor in customer churn trend?" -> No, even though it does contribute to the trend, `price`- related features are not a driving factor
* "What are the driving factors?" -> `cons_12m`,`forecast_meter_rent_12m`,`net_margin`, `forecast_cons_12m`, `margin_gross_pow_ele` are top 5 driving factors
* "What do you suggest?" -> Offering a discount to the customer who had the above values high, as `price` does affect the `churn` rate. However, we need to target specifically valued customers in those fields to keep them!
