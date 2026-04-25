# Part B: Business Case Analysis

# B1. Problem Formulation

(a) This is a supervised regression problem.  
The target variable we need to predict is `items_sold` (the number of items sold in each transaction).  
The input features include promotion_type, location_type, store_size, competition_density, is_weekend, is_festival, and the date features we created (year, month, day_of_week, is_month_end).

I chose regression because we are predicting a continuous numerical value (how many items will be sold), not a category like yes/no.

(b) The company currently looks at total sales revenue, but I think `items_sold` is a better target variable.  
Revenue can change a lot just because of price changes or discounts, even if the actual customer response to the promotion is the same.  
`items_sold` directly shows how effective the promotion was in terms of volume.  
This is a good example of why we should always choose a target variable that truly reflects the business goal and is less affected by external noise.

(c) A junior analyst suggested building one single global model for all 50 stores.  
I don’t think that’s the best approach. Stores in urban, semi-urban, and rural areas are very different in terms of customer behaviour, income levels, and local competition.  
A better strategy would be to build separate models for each location_type (Urban, Semi-urban, Rural) or to include strong interaction terms between store location and other features.  
This way the model can learn that the same promotion works differently in different types of stores.









## B2. Data and EDA Strategy

(a) The raw data comes in four separate tables: transactions, store attributes, promotion details, and a calendar table that has weekend and festival flags.

To combine them, I would:
- Start with the transactions table as the main table.
- Join the store attributes table using `store_id`.
- Join the promotion details table using `promotion_type` (or promotion_id if available).
- Join the calendar table using `transaction_date` to bring in weekend and festival information.

The final modelling dataset would have **one row per transaction**.  
Before modelling, I would aggregate some features like average competition_density or total promotions per store if needed, but mostly keep it at the transaction level since our target is `items_sold` per transaction.

(b) Before building the model, I would do the following EDA:

1. **Distribution of target variable (`items_sold`)** – histogram and boxplot. I would check if it’s skewed and decide whether to apply log transformation.
2. **Promotion effectiveness** – bar chart or boxplot of `items_sold` by `promotion_type`. This would tell me which promotions actually increase sales.
3. **Impact of store location and size** – boxplots of `items_sold` by `location_type` and `store_size`. This helps understand if certain stores respond better to promotions.
4. **Correlation heatmap** (especially with competition_density, is_festival, is_weekend). I would look for strong relationships that can be used as features.

These insights would guide my feature engineering — for example, creating interaction features like `promotion_type × location_type` if I see that the same promotion works differently in different locations.

(c) It’s mentioned that 80% of transactions happened without any promotion. This creates a strong class imbalance in the promotion column.

This could make the model biased towards “no promotion” and underestimate the effect of actual promotions.  
To handle this, I would:
- Use techniques like SMOTE or class weights in the model.
- Or oversample the transactions that had promotions.
- Also create a separate feature that clearly flags “no promotion” so the model can learn its baseline effect.


## B3. Model Evaluation and Deployment

(a) Since we have monthly store-level data spanning three years, I would use a **temporal train-test split**. I would train the model on the first two years (or first 80% of the time period) and test it on the most recent months.  

A random split would be inappropriate here because it could put future data into the training set and past data into the test set. This would cause data leakage and make the model look unrealistically good during testing, but perform poorly in real life when predicting the future.

For evaluation, I would mainly look at **RMSE** and **MAE** because our target is `items_sold` (a continuous number). I would also check **MAPE** (Mean Absolute Percentage Error) to see the error in percentage terms, which is easier for the business team to understand. In the business context, lower error means the model is giving more accurate promotion recommendations, which directly impacts sales.

(b) The model is recommending different promotions for the same store (Store 12) in December vs March. To investigate this, I would look at the **feature importance** from the Random Forest model.  

I would check which features have high importance in those months — for example, `is_festival` might be high in December (due to Christmas or year-end sales), while `competition_density` or weather-related factors might be different in March. I would also look at interaction effects (e.g., promotion_type × month).  

I would explain to the marketing team: “The model is not confused — it’s actually smart. It learned that the same store responds differently depending on the month because of seasonal factors and external conditions. So instead of giving the same promotion every month, we should adapt based on the time of year.”

(c) For deployment, I would save the trained model using `joblib` or `pickle`. Every month, a new batch of data would be prepared (with the same feature engineering steps we did during training). This new data would be passed through the same preprocessing pipeline and fed into the saved model to get promotion recommendations for all 50 stores.

To monitor performance, I would:
- Track actual `items_sold` vs model-predicted values every month.
- Set up alerts if the error (RMSE/MAE) suddenly increases beyond a threshold.
- Retrain the model every 3–6 months or when performance drops significantly.

This way the model stays up-to-date without needing to be retrained from scratch every single month.
