# E-Commerce Sales Forecasting for S&OP

## 1. Data Background (The Problem)
In the rapidly evolving world of online shopping, delivering products to customers efficiently is a complex logistical challenge. Uncertainty in supply chains often leads to stockouts, delayed deliveries, and inflated operational costs

This project is built for the Sales & Operations Planning (S&OP) team of a multinational e-commerce company. The goal is to forecast product demand to prepare for upcoming end-of-the-year sales and promotional events. By predicting how much of each product will be sold, the company can optimize inventory management and ensure prompt deliveries 

The dataset used (`Online Retail.csv`) contains transactional data including product stock codes, quantities, pricing, customer locations (countries), and detailed time-based metrics (year, month, week, day)

## 2. Analysis Method (Solution Thinking Flow)
To handle the large volume of transactional data and forecast future sales, we implemented a Big Data pipeline using **PySpark**:
1. **Data Cleaning & Date Formatting:** Reconstructed a clean `InvoiceDate` from existing year, month, and day columns. Filtered out invalid data, such as null dates and negative quantities (e.g., returns or errors)
2. **Data Aggregation:** Grouped the raw transactional data into daily summaries per product (`StockCode`) and `Country`. We summed up the total `Quantity` sold and averaged the `UnitPrice` for those groupings
3. **Time-Based Splitting:** Instead of a random split, the data was divided chronologically at a specific date (`2011-09-25`). Data before this date was used to train the model, and data after was used to test its forecasting ability
4. **Machine Learning Pipeline:** * Converted text categories (Country, StockCode) into numeric indices
   * Combined all predictive features (indices, month, year, day, week) into a single feature vector
   * Trained a **Random Forest Regressor** to predict the continuous numerical target (`Quantity`)
5. **Business Application:** Aggregated the final test predictions to find the total expected sales volume for a specific promotional week (Week 39)

## 3. Libraries Used & Desired Data Structure
Unlike standard Pandas/Scikit-Learn workflows, this project utilizes **Apache Spark (PySpark)** to handle potentially massive datasets in a distributed manner. The goal was to transform raw rows into an assembled "Feature Vector" structure required by Spark's machine learning engine

* **`SparkSession`**: The core entry point to use Spark, allowing us to process large data efficiently
* **`pyspark.sql.functions`**: Used extensively (`col`, `to_date`, `concat_ws`, etc.) to clean and manipulate data columns on the fly (e.g., stitching the date back together)
* **`StringIndexer`**: Spark's equivalent of label encoding. It converts text categories (like "United Kingdom" or stock code "85123A") into numeric indices
* **`VectorAssembler`**: A crucial Spark ML tool that takes multiple columns (our numeric features) and collapses them into a single array-like column called `features`
* **`RandomForestRegressor`**: The chosen algorithm. Because we are predicting a continuous number (quantity of items), we use a Regressor instead of a Classifier
* **`Pipeline`**: Chains our indexing, assembling, and modeling steps together so the data flows smoothly from raw text to final prediction
* **`RegressionEvaluator`**: Used to calculate the Mean Absolute Error (MAE) of our predictions

## 4. Results & Conclusion
* **Dataset Scale:** The chronological split yielded **175,452** daily aggregated records for training and **64,524** records for testing
* **Model Performance (MAE):** The Random Forest Regressor achieved a Mean Absolute Error of **~9.60**. This means that, on average, the model's daily prediction for a specific product's sales volume was off by about 9 to 10 units
* **Business Actionability (Week 39 Promo):** The model was successfully used to forecast the total aggregated sales for the upcoming promotional period in Week 39. The predicted quantity sold for this week across all regions is **88,580 units**
* **Conclusion:** The S&OP team can use this 88,580 unit benchmark to inform their supply chain ordering, ensuring warehouses are adequately stocked for the promotion while minimizing excess inventory costs. Future iterations could attempt to lower the MAE by including promotional flags or economic indicators as additional features
