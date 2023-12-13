# Predicting-Saturated-Fat-Content

This is a project at UCSD for DSC 80. We trained a model to predict the saturated fat content of a recipe based on the amount of carbohydrates, total fat, and calories of the recipe.

---

### Framing the Problem

For our problem, we decided to predict saturated fat content of recipes based on other nutritional values within the recipe. Since our variables were all quantitative, we decided to use Regression as our method of prediction, and RMSE as our test metric. We chose RMSE over R2, as RMSE performs better on unseen data, and we wanted to ensure we could generate robust predictions. Saturated fat was the perfect response variable, as it is known to be produced and found in the body as a result of excess carbs and calories, but this relationship has not been quantified. While some fats are essential for the body, saturated fats are responsible for high blood pressure, and result in higher risk of heart-related diseases. Furthermore, saturated fat is not always known as a nutritional value, but carbohydrates, calories, and total fat are almost always reported on ingredient labels, making them good variables to use as predictors. Therefore, being able to estimate saturated fat from these other nutritional values allows for healthier choices overall when considering recipes. We trained our model on a nutritional values dataset, containing 83,193 rows of different recipes.

---

### Baseline Model

Since we chose regression as our means of prediction, we used a simple Linear Regression model as our baseline. All of our feature variables –carbohydrates, calories, and total fat (as daily value percentages)– are quantitative, so we did not have to make many feature adjustments to the existing data. We converted calories to a percentage of the daily value (PDV) based on a 2000-calorie diet, and used this to predict saturated fat PDV. Based on our training and test data, we got a training RMSE of 92.68, and a test RMSE of 40.42. This seems like a good prediction model due to the low test RMSE, but it falls short when looking at the whole dataset, yielding an RMSE of 102.85. It seems as though the RMSE is a function of size, rather than a true estimator, due to the simplicity of the model. Therefore, the current model is not adequate or “good”, as it has large variation when it comes to the size of the dataset. 

---

### Final Model

The first feature we added was the transformation of the carbohydrates column. We first noticed that there was a nonlinear relationship between carbohydrates and the given values of saturated fat, so we decided to square the data based on the Turkey Mosteller diagram. Squaring the carbohydrates yielded a much more linear relationship, while cubing the data did not show any improvement. We therefore decided to transform the data using an exponent scale of 1.70, based on looped scoring with polynomial degrees between 1 and 2. We used R2 for this test, as we were only training this parameter on existing data, and it did not directly affect the predictions of the model, so R2 is a better measure of how linear the transformed feature could predict. Once the carbohydrates feature was transformed, we used a GridSearchCV with Polynomial Features to determine the ideal degree for the rest of the data, eventually landing on a degree of 2. We also added 5-fold cross validation, in order to avoid overfitting to the same training dataset we used in the baseline model. 
The final model was therefore a Polynomial Regression of degree 2, and an exponentiation of carbohydrates by a factor of 1.7, and yielded RMSEs of 32.24 and 32.36 for the training and test datasets, as well as a total dataset RMSE of 32.27. This is a large improvement over the initial model; our baseline’s largest issue was its RMSE dependence on the dataset size,  which the final model solved by generating robust predictions, regardless of the length of the frame. Furthermore, the model’s predictions were lower in terms of RMSE, meaning that there was overall less error in the final model predictions as compared to those of the baseline.

---

### Fairness Analysis

Now that our predictions had low error, regardless of dataset size, we wondered if the caloric content of the recipe would cause a difference in the error of the model. We therefore decided to use calorie content, with our two groups being recipes with over ⅓ of daily values of calories and recipes with less than ⅓ of daily caloric content. We chose 33% of daily caloric content as our threshold for binarization based on the idea of healthy eating habits being 3 square meals a day. After binarization, we conducted 1000 randomized permutation tests to confirm the reproducibility of our results. 
Our null hypothesis was that the distributions of low and high calorie RMSEs would show no difference, while our alternative hypothesis was that there would be a negative difference between the RMSEs. We chose a one-sided hypothesis test, as there were more low-calorie recipes (nL = 71,462) than high-calorie ones (nH = 11,731), and decided on a significance of 0.01 for our test. We selected the difference of RMSEs between the low and high calorie predictions as our test statistic due to RMSE being the main indicator of error in the predictions, and chose RMSElow – RMSEhigh as there were simply more instances of low calorie recipes than high calorie ones. 
α = 0.01
H0: 𝜃 = 0
Ha: 𝜃 < 0
𝜃 = RMSELOW – RMSEHIGH


This is the distribution of differences in RMSE between high and low calorie recipes from generating random samples under the null hypothesis.

<iframe src="assets/fig_1.html" width=800 height=600 frameBorder=0></iframe>


With a p-value of 0.0 for 1000 trials, we were able to reject the null hypothesis at a significance of 0.01. We were therefore able to conclude that it is very statistically likely that the model predicts at a lower RMSE for low-calorie recipes as compared to high-calorie recipes. As explained in the reasoning behind the alternative hypothesis, there were more low-calorie recipes, which we believe led to the results. Furthermore, there was a greater spread of saturated fat content observed in higher-calorie recipes, with a standard deviation of 172.76% DV, as compared to low-calorie recipes (standard deviation of 26.02% DV), which we believe was another factor in the differences in the distributions of prediction errors. 
