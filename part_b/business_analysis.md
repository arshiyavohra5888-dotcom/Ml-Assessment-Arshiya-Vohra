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
