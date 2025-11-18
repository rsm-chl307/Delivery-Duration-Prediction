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

## Practicalities

You are required to build a model to predict the **total delivery duration (in seconds)** as defined above.  
You are encouraged to generate **additional features** from the given data to improve model performance.

Please include explanations for the following:

- Model(s) used  
- How you evaluated model performance on the historical data  
- Any data preprocessing you performed  
- Feature engineering choices you made  
- Any other information you wish to share about your modeling approach

The project is expected to take approximately **3–5 hours** in total, though you may spend more time if desired.  
You may use **any open-source packages** for this task.

