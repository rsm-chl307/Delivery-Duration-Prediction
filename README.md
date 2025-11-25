# Assignment

When a consumer places an order on DoorDash, we show the expected time of delivery. It is very important for DoorDash to get this right, as it has a big impact on consumer experience.  
In this exercise, you will build a model to predict the estimated time taken for a delivery.

---

## Objective

For a given delivery, you must predict the **total delivery duration (in seconds)**, i.e., the time taken from:

- **Start:** the time the consumer submits the order (`created_at`)  
- **End:** when the order will be delivered to the consumer (`actual_delivery_time`)

---

## Data Description

The attached file `historical_data.csv` contains a subset of deliveries received at DoorDash in early 2015 from a subset of cities.  
Each row in this file corresponds to one unique delivery.  
Noise has been added to the dataset to obfuscate certain business details.  

Each column corresponds to a feature as explained below.  
Note: all **money values** are given in **cents**, and all **time durations** are given in **seconds**.

The **target value** to predict is the total number of seconds between `created_at` and `actual_delivery_time`.

---

## Columns in `historical_data.csv`

### Time Features
| Column | Description |
|--------|--------------|
| `market_id` | A city/region in which DoorDash operates (e.g., Los Angeles), represented as an ID |
| `created_at` | UTC timestamp when the order was submitted by the consumer to DoorDash. (Note: this timestamp is in UTC, but the actual timezone of the region is US/Pacific) |
| `actual_delivery_time` | UTC timestamp when the order was delivered to the consumer |

---

### Store Features
| Column | Description |
|--------|--------------|
| `store_id` | An ID representing the restaurant where the order was placed |
| `store_primary_category` | Cuisine category of the restaurant (e.g., Italian, Asian) |
| `order_protocol` | A numeric ID representing the mode/protocol by which the store receives orders |

---

### Order Features
| Column | Description |
|--------|--------------|
| `total_items` | Total number of items in the order |
| `subtotal` | Total value of the order (in cents) |
| `num_distinct_items` | Number of distinct items included in the order |
| `min_item_price` | Price of the least expensive item in the order (in cents) |
| `max_item_price` | Price of the most expensive item in the order (in cents) |

---

### Market Features
DoorDash operates as a marketplace, so we also have information about the **state of the marketplace** at the time an order was placed.  
These features can help estimate delivery time.  
All values correspond to the moment of `created_at` (order submission time).

| Column | Description |
|--------|--------------|
| `total_onshift_dashers` | Number of available dashers within 10 miles of the store at order creation |
| `total_busy_dashers` | Number of dashers currently working on an order (subset of the above) |
| `total_outstanding_orders` | Number of orders within 10 miles that are currently being processed |

---

### Predictions from Other Models
These features are predictions generated from other models for various stages of the delivery process and can be used as inputs.

| Column | Description |
|--------|--------------|
| `estimated_order_place_duration` | Estimated time for the restaurant to receive the order from DoorDash (in seconds) |
| `estimated_store_to_consumer_driving_duration` | Estimated driving time between store and consumer (in seconds) |

---

## 🛠 Data Preparation & Cleaning  

### 1. Outlier and Data Quality Checks  

- Removed rows where the following variables had **negative values** (logically impossible):  
  - `min_item_price`  
  - `total_onshift_dashers`  
  - `total_busy_dashers`  
  - `total_outstanding_orders`  

- Trimmed extreme outliers in the target by removing observations above the **99th percentile** of delivery duration.

### 2. Log Transformation  

The target and many numeric features were **heavily right-skewed**.  
To stabilize variance and reduce the impact of extreme values we applied:

- `log_target = log1p(target)`  
- `log_subtotal`  
- `log_min_item_price`  
- `log_max_item_price`  
- `log_total_items`  
- `log_num_distinct_items`  
- `log_total_onshift_dashers`  
- `log_total_busy_dashers`  
- `log_total_outstanding_orders`  
- `log_estimated_order_place_duration`  

The original numeric columns were dropped after transformation, keeping only their log versions.

### 3. Encoding Categorical Variables  

- `store_id` and `store_primary_category`  
  - **Target encoding** to convert high-cardinality categories into informative numeric representations.  

- `market_id` and `order_protocol`  
  - **One-hot encoding** (low cardinality, suitable for dummy variables).  

---

## 🧩 Feature Engineering  

We engineered more than **15 business-driven features** to capture order complexity, marketplace pressure, and restaurant efficiency. A few key examples:

- `fe_difficulty_score`  
  - Composite index combining order complexity, demand pressure, and preparation time.  
  - Designed to measure how operationally “difficult” an order is to fulfill.

- `fe_store_demand_intensity`  
  - Interaction between store behavior (`store_id` encoding) and outstanding orders.  
  - Highlights stores that perform poorly when demand is high.

- `fe_dasher_load_ratio`  
  - Approximates the ratio of busy dashers to on-shift dashers in log space.  
  - Captures courier overload and market congestion.

- `fe_demand_pressure`  
  - Outstanding orders relative to available dashers.  
  - Higher values imply longer expected wait times.

- `fe_order_value_index`  
  - Combines subtotal and item diversity as a proxy for large, complex, high-value orders.

- `fe_prep_pressure`  
  - Sum of log(total items), log(distinct items), and log(prep duration).  
  - Measures restaurant-side preparation difficulty.

- `fe_distance_adjusted_pressure` and `fe_market_congestion`  
  - Combine demand pressure and estimated driving duration to reflect both kitchen and travel constraints.

All engineered features were added to `df_train_log_fe` and `df_test_log_fe`.

---

## 🧠 Models Explored  

### 1. Baseline Linear Regression (OLS)  

- Fitted on the **original (non-log) target**.  
- Exhibited strong heteroscedasticity and non-normal residuals.  
- R² around **0.29**, confirming the need for transformation.

### 2. Log-Transformed Linear Regression (OLS)  

- Response: `log_target`  
- Predictors: original engineered and transformed features.  
- Residual diagnostics improved substantially (more normal, more constant variance).  
- R² in the low **0.30s** on the log scale.

### 3. Ridge Regression (Regularized Linear Model) — *Explored, Not Selected as Final*  

- Used scikit-learn `RidgeCV` with cross-validated α values.  
- Helped stabilize coefficients under multicollinearity but **did not deliver a major lift** in out-of-sample R² or RMSE compared with the simpler OLS on the engineered features.  
- For interpretability and simplicity, we chose to keep the **non-regularized OLS** as the final model.

### 4. Multi-Layer Perceptron (Neural Network) — *Explored, Not Selected as Final*  

- Architecture examples:
  - Hidden layers: `(3, 3)`  
  - Activation: `tanh`  
  - Solver: `lbfgs`  
  - Alpha: 1  
  - Max iterations: 1,000,000  

- Performance (on log target):  
  - R² ≈ **0.35**  

- When predictions were transformed back to the original scale (`expm1`),  
  - R² ≈ **0.29**, very close to the linear model.  

Because the neural network introduced additional complexity without material performance gains, it was **not chosen as the final production model**.

---

## 🏁 Final Model  

**Final model:**  
> Ordinary Least Squares Regression  
> Using **log-transformed target** and the **feature-engineered dataset** (`df_train_log_fe`, `df_test_log_fe`),  
> **without Ridge regularization**.

Predictions are generated in log space and then transformed back to the original scale:

\[
\hat{y}_{\text{seconds}} = \exp(\hat{y}_{\text{log}}) - 1
\]

### Final Performance (Original Scale)

On the test set (`df_test_log_fe`):

```text
Test RMSE (original scale): 829.866 seconds
Test MAE  (original scale): 620.762 seconds
Test R²   (original scale): 0.292

