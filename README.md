# PowerCo_BCG
This project is from Boston Consulting Group, tasking to help PowerCo to understand customer churn.

## Problem Statement
PowerCo is observing a number of their customers unsubscribe from their service to use another company's instead- customer churn. They want to suggest whether discounting is a good way to prevent this and how they should target the audience. Decomposing their request, I am trying to answer two questions:
* Is price a driving factor leading to customer churn? - In other words, calculating price sensitivity
* What critical features can we use to predict the customer churn trend?
  
-> Client and price data will be necessary to answer these questions.

## Data Analysis
Some initial views about our data:
* The `churn` rate is about `9.7%` across 14,606 customers, which is quite high.
  
   <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image1.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image1.png" alt="image1" width="300">
</a>

* The distribution of `churn` customers does not quite depend on whether they subscribed to gas or not, as they are pretty much the same

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image2.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image2.png" alt="image2" width="600">
</a>

* All `churn` customers subscribed to less than **5** of our services

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image3.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image3.png" alt="image3" width="1000">
</a>
  
* All `churn` customers  distribute across **5** of our sale channel. The `MISSING` channel means no information available.

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image4.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image4.png" alt="image4" width="1000">
</a>

For features related to consumption and price, the plot is highly skewed and spread out because the values are too big. We can clean these data in the data engineering step.

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image5.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image5.png" alt="image3" width="800">
</a>

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

Here is the final correlation heatmap: 

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image6.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image6.png" alt="image6" width="1000">
</a>

## Modeling
### Process:
After the feature engineering process, we are following this pipeline:
* **Train-test split**: We do 70-30 train-test split
* **Standardize features**: This step is necessary, as even though consumptions/price fields are transformed, there is still a lot of variation.
* **Fixing skewness**: Our data is highly skewed with **90%** `non-churn` vs **10%** `churn`. This will make the prediction highly lean towards the `non-chun` side.
* **Modeling**: We are using a Random Forest Classifier. We are able to achieve a **95%** accuracy score on 10-fold cross-validation and **43%** on the test data.
  * We achieve **11%** on precision, which is how well we can label `churn` customers, and **68.58%**. F1-score is **19.45%**
  * In this task, I believe `recall` is a better metric to evaluate our model, as it does not hurt to misclassify `non-churn` customers to `churn` customers.
  * Based on the evaluation metrics, our model is doing very well in cross-validation but extremely poor on the test set. I believe the reason is **overfitting** due to the high-dimension feature space.
* **Feature Importance**: After training our model, we can find the importance of each feature in helping the model correctly predict `churn.` The top 5 driving factors are:
  * `cons_12m`
  * `forecast_meter_rent_12m`
  * `net_margin`
  * `forecast_cons_12m`
  * `margin_gross_pow_ele`

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image7.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image7.png" alt="image7" width="1000">
</a>
 
### Model optimization:
The following approaches can be conducted to help improve our model:
* **Feature Selection**: We can deliberately choose features that contribute a lot to our model in the above picture (`>=0.015`)
  * Performance significantly increases, with **82.55%** accuracy, **80%** precision, **86.99%** recall and **83.35%** F1 score
  * This proves that the problem we had earlier with our model was due to overfitting.
* **PCA**: This method is relevant in reducing the dimension of our fields. With around `20-30` features, the space of `60` features can be almost `100%` explained.
  * Performance again sees a huge increases, with **93.02%** accuracy, **91.61%** precision, **91.61%** recall and **93.17%** F1 score

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image8.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image8.png" alt="image8" width="1000">
</a>

  <a href="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image9.png" target="_blank">
    <img src="https://github.com/KietKat/PowerCo_BCG/blob/master/BCG_image/image9.png" alt="image9" width="1000">
</a>

## Conclusion
We are answering the following questions:
* "What is the reality of `churn` customer trend?" -> It is high, at `9.7%` over `14606` customers
* "Is price a driving factor in customer churn trend?" -> No, even though it does contribute to the trend, `price`- related features are not a driving factor
* "What are the driving factors?" -> `cons_12m`,`forecast_meter_rent_12m`,`net_margin`, `forecast_cons_12m`, `margin_gross_pow_ele` are top 5 driving factors
* "What do you suggest?" -> Offering a discount to the customer who had the above values high, as `price` does affect the `churn` rate. However, we need to target specifically valued customers in those fields to keep them!
* **PCA** and **Feature selection** does help greatly improve our models
