# Cross Sell Prediction

**Objective** 
---

- Predicting whether an existing health insurance customer will be cross-sold car insurance

**Data Sources**
---
- Kaggle and Analytics Vidhya Competition; [Link Here](https://www.kaggle.com/anmolkumar/health-insurance-cross-sell-prediction)

**Background**
---

- Companies typically win most of their customers through paid channels and implicitly have incurred costs for every customer they win.  For instance, Facebook ads, direct mail or the sales team directly engaging with customers.  This is known as customer acquisition costs or "CAC".

- Many companies have payback periods, which represents the number of months to pay back customer acquisition costs, of 5+ months.  Given this high payback period, cross-selling existing customers is a great way to win high margin revenue.

- Through cross-sell prediction from this project, the sales and marketing team could pre-screen customers and place an emphasis on customers with a high probability of being cross-sold.


## Featured Techniques & Models
- Feature Engineering and Selection 
- Supervised Machine Learning
- Imbalanced Techniques
- Logistic Regression
- K-Nearest Neighbors 
- Random Forest 
- Naive Bayes (Gaussian and Categorical)
- Balanced Random Forest
- Balanced Bagging Classifier 
- XGBoost (Binary Logistic)

**Results**
---

F2 score was chosen as the optimization metric because predicting the cross-sold customers is the main priority, but some precision is still necessary as the funnel of cross-sold customers should not become too wide or marketing efficiencies wil be lost. 

The top performing model was a logistic regression (threshold = 0.2) with a 61% F2 score, 31% precision and 81% recall.  The top features were whether the customer had a vehicle with damage, was sold health insurance through policy sales channel 26 and was in Region Code 28.  The second highest performing model was a balanced Random Forest with a 60% F2 score, 29% precision and 82% recall.
