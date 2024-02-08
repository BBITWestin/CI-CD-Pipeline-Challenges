# Machine Learning Quote Bot Tool

2/7/2024 - Trevor Westin

## The Problem

The primary objective is to predict the charge rate for a load based on load rate data. This prediction could be influenced by various factors, including distance, load weight, type of goods, market demand, fuel prices, and seasonality.

## 1. Data Collection

Collect historical data that may influence the charge rates. This could include:

- **Internal data**: Historical load details, including weights, dimensions, distances, types of goods, pickup and delivery locations, and historical charge rates.
- **External data**: Fuel prices, weather conditions, economic indicators, and data on market demand fluctuations.

## 2. Data Preprocessing

Clean and preprocess the data:

- **Cleaning**: Handle missing values, remove outliers, and correct errors.
- **Feature Engineering**: Create new features that could help in prediction, such as day of the week, time of the year, or categorizing distances into short, medium, and long hauls.
- **Normalization/Standardization**: Scale the features to normalize their range of values.
- **Encoding**: Convert categorical variables into numeric format using [one-hot encoding](https://www.geeksforgeeks.org/ml-one-hot-encoding-of-datasets-in-python/) or [label encoding](https://www.geeksforgeeks.org/ml-label-encoding-of-datasets-in-python/).

## 3. Model Selection

Given the nature of the problem (predicting a continuous value), regression models are suitable. We may want to consider starting with simpler models and gradually moving to more complex ones only if necessary:

- **Linear Regression**: Good starting point for establishing a baseline.
- **Random Forest Regressor**: A more complex model that can capture nonlinear relationships.
- **Gradient Boosting Machines (GBM)**: XGBoost, LightGBM, or CatBoost, are popular for regression tasks.
- **Deep Learning Models**: Neural networks can be beneficial, especially with large datasets and complex relationships, but they require more computational resources.

See the two links below for why these models might be best.

## 5. Feature Selection

Use techniques to select the most relevant features to reduce model complexity and improve performance. This can be achieved through methods like:

- [Correlation analysis](https://www.geeksforgeeks.org/what-is-correlation-analysis/?ref=header_search)
- Feature importance from model

## 6. Model Training and Validation

Split the data into training, validation, and test sets to evaluate the model's performance:

- Use the training set to train the model.
- The validation set is used to tune the [hyperparameters](https://www.geeksforgeeks.org/difference-between-model-parameters-vs-hyperparameters/?ref=header_search) such as learning rate.
- The test set provides an unbiased evaluation of the final model fit. aka the model has the same preformace with test set 1 as it does with test set 2.

Use cross-validation to ensure that the model's performance is consistent across different subsets of the data.

## 7. Deployment

Deploy the trained model into a production environment where it can provide real-time predictions. This could involve:

- Creating an API that receives load details and returns the predicted charge rate.
- Developing a frontend where users can input load details and receive quotes.

#### Challenges...

See [MLOps Architecture Research](https://github.com/orgs/bbi-dev-ops/projects/9/views/1?pane=issue&itemId=52529394)

## 8. Monitoring and Updating

Regularly monitor the model's performance and update it with new data to ensure its predictions remain accurate over time. This involves setting up a pipeline for continuous learning. More research needed here... _@ Nathan_.

---

#### Technical Stack

Python with libraries such as Pandas, Scikit-learn for ML models, and TensorFlow or PyTorch for deep learning.

## Links

[Prediction of Shipping Cost on Freight Brokerage Platform Using Machine Learning](https://www.mdpi.com/2071-1050/15/2/1122)
[Utilizing Machine Learning, Predicting Shipping Costs on a Freight Brokerage Platform](https://www.hilarispublisher.com/open-access/utilizing-machine-learning-predicting-shipping-costs-on-a-freight-brokerage-platform-95342.html)
[UNLOCKING THE POWER OF MACHINE LEARNING TO FORECAST TRUCKING RATES](https://www.penskelogistics.com/industries/industrial-manufacturing/machine-learning)
