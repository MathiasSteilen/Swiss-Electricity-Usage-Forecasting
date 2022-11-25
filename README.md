## Forecasting Swiss End-User Electricity Demand With Supervised Machine Learning Methods

### Final Report [(Visit Link)](https://rpubs.com/MathiasSteilen/swiss-energy-forecast-supervised-ml)

The detailed report of this project was hosted on [RPubs](https://rpubs.com/MathiasSteilen/swiss-energy-forecast-supervised-ml). Below, only a very brief overview over the results of the project is given.

<br>

### Description

The goal of this project is to test the ability of supervised machine learning models, namely elastic net, gradient boosting and random forest, to predict end-user electricity consumption in Switzerland. The final product is a trained gradient boosting model that takes on a future date, as well as weather data (temperature, humidity, precipitation, cloud coverage), and gives out an accurate estimate of aggregate electricity demand in Switzerland. The final model has an $R^2$ of approximately 97% and a mean absolute percentage error (MAPE) of just under 2%.

<br>

### Tools

For this project, the programming language R was used with the _tidymodels_ package, which provides a great workflow for supervised machine learning. The _html_, which is also hosted on [RPubs](https://rpubs.com/MathiasSteilen/swiss-energy-forecast-supervised-ml), was knitted using _RMarkdown_.

<br>

### Data

The electricity demand data comes from [Swissgrid](https://www.swissgrid.ch/en/home/operation/grid-data/generation.html) and is denoted in $kwh$. Additionally, historical weather data is used as an additional predictor, which has been retrieved from [Swiss NBCN](https://data.geo.admin.ch/ch.meteoschweiz.klima/nbcn-tageswerte/liste-download-nbcn-d.csv). The target variable is daily electricity demand denoted in $mwh$. The training/testing split is 10 years to 1 year.

<p align="center">
<img src="Graphics/Split.png" alt="training/testing split" width="600px"/>
</p>

<br>

### Model Performance

Out of elastic net, random forest and gradient boosting, the latter two were chosen for hyperparameter tuning due to better initial performance with default parameters. After hyperparameter tuning for these two tree-based ensemble models, the final out-of-sample fits can be seen below.

##### Prediction vs. Actuals

<p align="center">
<img src="Graphics/predvsactual.png" alt="training/testing split" width="600px"/>
</p>

From the above chart, it becomes visually clear that the gradient boosting shows fewer outliers than the random forest, explaining the better $R^2$ of the first.

<br>

##### Out-Of-Sample Time Series

<p align="center">
<img src="Graphics/oosts_rf_gb.png" alt="training/testing split" width="600px"/>
</p>

Both models have a very good fit. The gradient boosting model has slightly higher accuracy on most weeks than random forest.

<br>

##### Comparison of Gradient Boosting to Baseline Model 

<p align="center">
<img src="Graphics/oosts_gb_baseline.png" alt="training/testing split" width="600px"/>
</p>

Clearly, the gradient boosting model shows solid performance against just predicting the historical target mean and could be used on future data points, after retraining on newer data.

<br>

### Limitations

Some limitations have to be mentioned where the model might show deficiencies.

<br>

##### Performance in unforeseen circumstances

Performance of predictive model relies heavily on the continuance of the relation of predictors with the target variable. If the relation breaks down due to unforeseen circumstances like a pandemic, then the model will act as if this event had never happened.

<p align="center">
<img src="Graphics/oos_corona_delta.png" alt="training/testing split" width="600px"/>
</p>

The cloud of red points during the first lockdown shows the model overestimating demand. This abnormal effect has not happened again as the Swiss government have made a strong effort to keep the economy running, however, the period of a couple of months showed the vulnerability of the model. It must be noted that this vulnerability is not unique to this model, virtually all models inferring from the past are subject to it.

<br>

##### General usability in the energy sector

In practice, the forecast is usually much shorter than one year. Production is adjusted almost in real-time, so forecasts over long time-periods are not useful due to higher uncertainty caused by omitted predictors. Additionally, changing conditions of the predictors have to be accounted for, which is more feasible in real-time, short-term models. This goes hand in hand with the next point.

<br>

##### Weather readings

The weather readings for the training of this model were actual values. In practice, when predicting energy demand, weather forecasts are part of the features, instead of actual values. This additional uncertainty will likely impact model performance negatively in practice.

<br>


### How To Run The Models In This Project

This project is intended as a piece of research into how machine learning models can be used to predict energy demand. In theory, after cloning this repository, you could load and use the model for making your own predictions like this:

```
# Libraries
library(tidyverse)
library(tidymodels)

# Loading Model
gb_model <- readRDS("Data/gb_final_fit.csv")

# One Holdout Observation
example_data <- structure(
    list(year = 2019, day_in_year = 120, date = structure(18016, class = "Date"),
         mwh = 156760.631422, ALT_meantemp = 9.5, ALT_precipitation = 0, 
         ALT_cloudcoverage = 75, ALT_humidity = 77.8, BAS_meantemp = 9.8, 
         BAS_precipitation = 0, BAS_cloudcoverage = 79, BAS_humidity = 77.2, 
         BER_meantemp = 9.5, BER_precipitation = 0, BER_humidity = 75.1, 
         DAV_meantemp = 1.6, DAV_precipitation = 0.2, DAV_humidity = 85.5, 
         GVE_meantemp = 11, GVE_precipitation = 0, GVE_cloudcoverage = 13, 
         GVE_humidity = 65.7, LUG_meantemp = 16.4, LUG_precipitation = 0, 
         LUG_cloudcoverage = 29, LUG_humidity = 38.2, LUZ_meantemp = 9.7, 
         LUZ_precipitation = 0, LUZ_humidity = 75.8, NEU_meantemp = 10.5, 
         NEU_precipitation = 0, NEU_humidity = 65.8, SMA_meantemp = 8.1, 
         SMA_precipitation = 0, SMA_cloudcoverage = 88, SMA_humidity = 80.9, 
         STG_meantemp = 6.2, STG_precipitation = 0.3, STG_humidity = 86.5, 
         cloudcoverage = 56.8, humidity = 72.85, meantemp = 9.23, 
         precipitation = 0.05), row.names = c(NA, -1L), 
         class = c("tbl_df", "tbl", "data.frame")
        )

# Make Prediction On Example Observation
gb_model %>% extract_workflow() %>% augment(NEW_DATA)
```

The problem in practice is getting the data to make predictions on: The model was trained with over 40 variables:

1) Variables that are easy to obtain for the user:
    `year`, `day_in_year`, `date`
2) Variables that are harder to obtain: 
    `ALT_meantemp`, `ALT_precipitation`, `ALT_cloudcoverage`, `ALT_humidity`, 
    `BAS_meantemp`, `BAS_precipitation`, `BAS_cloudcoverage`, `BAS_humidity`,
    `BER_meantemp`, `BER_precipitation`, `BER_humidity`, `DAV_meantemp`,
    `DAV_precipitation`, `DAV_humidity`, `GVE_meantemp`, `GVE_precipitation`, 
    `GVE_cloudcoverage`, `GVE_humidity`, `LUG_meantemp`, `LUG_precipitation`,
    `LUG_cloudcoverage`, `LUG_humidity`, `LUZ_meantemp`, `LUZ_precipitation`,
    `LUZ_humidity`, `NEU_meantemp`, `NEU_precipitation`, `NEU_humidity`, 
    `SMA_meantemp`, `SMA_precipitation`, `SMA_cloudcoverage`, `SMA_humidity`,
    `STG_meantemp`, `STG_precipitation`, `STG_humidity`, `cloudcoverage`,
    `humidity`, `meantemp`, `precipitation`
3) Target variable (no need to obtain for predictions): 
    `mwh`

All variables of type 2 come from the Swiss government, hence the user is dependent on these exact data points. As the data must be forecasts, but for the purpose of the analysis, actual data points were used, it will likely be hard to get the forecasts for every geolocation to use this model in practice. Therefore, as a disclaimer: This project (specifically, the [final report](https://rpubs.com/MathiasSteilen/swiss-energy-forecast-supervised-ml)), is intended as a piece of research instead of an implementation of a model that can immediately be deployed in practice.

<br>