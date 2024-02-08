# Machine Learning Quote Bot Research

2/8/2024 - Trevor Westin

The following are my notes / summary from [Prediction of Shipping Cost on Freight Brokerage Platform Using Machine Learning](https://www.mdpi.com/2071-1050/15/2/1122).

## Abstract

This study derives the main variables that influence the setting of the shipping costs and presents the recommended shipping cost given by a price prediction model using machine learning methods.

The cost prediction model was built using four algorithms: multiple linear regression, deep neural network, XGBoost regression, and LightGBM regression. R-squared was used as the performance evaluation index. In view of the results of this study, LightGBM was chosen as the model with the greatest explanatory power and the fastest processing. Furthermore, the range of the predicted shipping costs was determined considering realistic usage patterns. The confidence interval was used as the method of calculation for the range of the predicted shipping costs, and, for this purpose, the dataset was classified using the K-fold cross-validation method. This paper could be used to set the shipping costs on freight brokerage platforms and to improve utilization rates.

## Intro

Currently, the criteria for setting the shipping costs simply consider distance and vehicle tonnage.

We present a method for setting the range of predicted fares considering realistic usage behaviors; the fares should be presented as a range rather than as a single value to the user.

The machine learning algorithms we used were multiple linear regression (MLR), deep neural networks (DNNs), extreme gradient boosting (XGBoost) regression, and light gradient boosting machine (LightGBM) regression. LightGBM is a model that reduces the learning time compared to the XGBoost model.

A total of 30 training sets were generated using k-fold cross-validation. Assuming that the 30 derived predicted values follow a normal distribution, a confidence interval was calculated, and an appropriate fare range was presented.

R-squared was used as the performance evaluation index for the predictive model.

## Literature Review

### Prior Studies

- Kovács calculated road freight shipping costs, which previously had only been estimated. Transport-related factors such as “distance,” “fuel,” “price,” and “highway toll” were selected to calculate the cost. A predictive model based on multiple regression analysis was built using the selected factors, and it demonstrated an excellent predictive performance.
- Lindsey et al. investigated factors that affect truck fare rates in North America and found that factors such as “distance” and “truck type” were the most important factors for determining the shipping costs.

### Theoretical Background

According to the learning method, machine learning is categorized as supervised learning and unsupervised learning. Supervised learning is a learning algorithm that learns data with input and output values, and it predicts output values for unseen data or future data. It is used for classification or regression analysis.

![ML_Algorithms](ML_Algorithms.png)

## Data Collection and Preprocessing

### Data Collections

For this study, we collected freight brokerage data that were registered on the freight brokerage platform within the 6 months from April to September 2020.

The dataset consists of 1,885,033 data observations and 78 variables used by freight brokerages, such as cargo information, vehicle type, vehicle tonnage, loading date and time, and unloading date and time.

### Data Preprocessing

#### Creating and Removing Variables

In the dataset, variables that were related to the personal information of the cargo owners and vehicle owners, such as name, vehicle number, and phone number, were deleted, as they were judged to be irrelevant to the shipping cost prediction.

The latitude and longitude of the upper and lower location were calculated using the haversine distance formula and were added as a “linear distance.” It was judged that detailed date and time information could affect the cost setting, so the arrival and departure dates were subdivided into the month, day, day of the week, and time, and new variables were created for each. In addition, we added the precipitation amount as a new factor, considering that the weather conditions at the time of the cargo transport would affect the shipping cost. The precipitation data of the Korea Meteorological Administration were used, and the precipitation value was added by considering the loading and unloading locations and dates.

#### Removing Data Outliers

The interquartile range (IQR) was used to remove outliers in the data. Outlier removal was applied only to continuous variables.

#### Handling Missing Data

After applying two methods for managing missing values, we compared which method was more useful. Before the processing of missing values, factors for which more than 50% of the data were missing were determined to be factors that did not have a great influence on the prediction and were, thus, removed. We removed 20 factors, including “load/unload name address,” “summary,” and “order number.” For the missing value treatment, listwise deletion and the mean imputation were used, and a dataset was created to which each treatment for missing values was applied. The listwise deletion removed all data with missing values, and the mean imputation replaced the missing values with the average value of each factor.

![alt text](ML_Missing Data.png)

## Derivation of Key Factors

Correlational analysis and step selection were applied as a means to derive the major factors.

### Correlational Analysis

In this study, independent variables with a correlation coefficient of 0.1 or higher, which is judged to indicate a linear relationship between the independent variable and the dependent variable, were judged to be the main factors.

### Stepwise Method

The stepwise method was one of the methods used for selecting several independent variables to be included in the regression model. It is a method that is used to find the variable constituting the optimal regression model by repeating the addition and removal of variables. The selected variable was judged to be a strong predictor in the prediction model.

![Independent variables](Independent_vars.png)

## Model Construction and Analysis Results

### Data Preparation

To ensure the accuracy of the model, all variables were normalized to the same scale. Min–max normalization, which converts all continuous variable data to values between 0 and 1, was used for normalization. We attempted to derive the shipping cost as a range using the confidence interval. In the case of freight, some characteristics were not fully expressed in the data. Therefore, if the recommended cost is presented as a single value, it has limited means to reflect the volatility of reality. To derive the shipping cost as a range, a 95% confidence interval was calculated for the cost value predicted by the shipping cost prediction model. To ensure that the distribution of the predicted values follows a normal distribution, we increased the predicted values (number of samples) using K-fold cross-validation.

For the suitability of the model, 80% of the collected data were allocated to a training set and 20% to a test set, and the datasets were then used for the model construction and verification. At this time, to derive the cost range, the training set was divided into 30 folds through K-fold cross-validation, and 30 predicted values were derived by predicting the test set for each iteration. For each iteration, 29 training sets and 1 validation set were used. After that, the model was trained on the training set of each fold, and the process of predicting with the test set was repeated until 30 predicted values were derived. Afterward, the maximum and minimum values in the confidence interval were determined as the upper and lower limits of the prediction interval to estimate the predicted cost range. Overall configuration of a dataset for range prediction is shown in Figure 2.
