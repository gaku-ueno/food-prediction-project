# Analysis of Food.com recipe features and their ratings

Authors: Gaku Ueno & Neil Dewan

## 📖 Introduction

In a world increasingly shaped by the internet, one constant remains: people's love for food. Online recipe platforms like **Food.com** have become essential resources for home cooks, culinary enthusiasts, and professional chefs alike. With over **500,000 recipes**, Food.com provides a rich dataset to explore user preferences and culinary trends.

This project investigates the relationship between recipe characteristics—particularly **calories**—and user-generated **ratings**. Using recipes and ratings posted since 2008 on [Food.com](https://www.food.com/), originally utilized in the recommender system research paper *"Generating Personalized Recipes from Historical User Preferences"* by Majumder et al., we will apply statistical modeling techniques to uncover meaningful patterns. Specifically, we will build predictive models to understand how nutritional content influences user ratings.

---

## 📚 Dataset Overview

### 🍽️ Recipes Dataset  

The recipes dataset contains information about recipes posted on Food.com. It includes **231,637** rows with the following 10 columns:

| Column Name       | Description                                             |
|-------------------|---------------------------------------------------------|
| `name`            | Recipe name                                             |
| `id`              | Recipe ID                                               |
| `minutes`         | Minutes required to prepare the recipe                  |
| `contributor_id`  | User ID who submitted this recipe                       |
| `submitted`       | Date when the recipe was submitted                      |
| `tags`            | Tags associated with the recipe                         |
| `nutrition`       | Nutrition info: `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]` |
| `n_steps`         | Number of steps in recipe                               |
| `steps`           | Text for recipe steps                                   |
| `description`     | User-provided description                               |

---

### ⭐️ Ratings Dataset  

The ratings dataset contains user-generated reviews and ratings for recipes. It includes **731,927** rows with the following columns:

| Column Name   | Description                      |
|---------------|----------------------------------|
| `user_id`     | User ID                          |
| `recipe_id`   | Recipe ID                        |
| `date`        | Date of interaction              |
| `rating`      | Rating given (1-5 stars)         |
| `review`      | Review text provided by user     |

---

## 🎯 Question  

We aim to answer the question: What are the most important features when predicting the rating?


Let's dive into the analysis! 🚀

## 🧹 Data Cleaning and Exploratory Data Analysis (EDA)

### 🔗 **Merging Datasets**

The first step we took was merging the **recipe** and **review** dataframes to match each review with its corresponding recipe. This allowed us to have a complete record for each observation—both the recipe's attributes and its associated reviews.

### ⭐️ **Handling Missing Ratings**

Next, we inspected the `rating` column. Since ratings are provided on a 5-point scale (**1–5**), any entry with a **0 rating** indicates that a rating was not actually submitted in the review. To accurately reflect this, we replaced these 0-star ratings with `np.nan`. This approach prevents artificially lowering the average rating during statistical analyses.

### 🍎 **Extracting Nutritional Information**

Following the treatment of ratings, we turned our attention to the `nutrition` column. Originally, this column contained a composite string with multiple nutritional components combined together. To facilitate analysis and feature engineering, we decomposed this field into several distinct columns. Specifically, we extracted values for:

- ✅ **Calories (#)**
- ✅ **Total fat (PDV)**
- ✅ **Saturated fat (PDV)**
- ✅ **Protein (PDV)**
- *(and other relevant nutritional attributes)*

The percent daily value (**PDV**) indicates how much a nutrient in a serving contributes to a typical daily diet.


### 📊 **Feature Engineering: `calorie_range`**

We introduced an additional categorical feature, `calorie_range`, believing that calorie content could significantly influence user ratings. We classified recipes into four categories based on calorie content:

| Calorie Range | Calories                 |
|---------------|--------------------------|
| very_low      | [0–25th percentile)      |
| low           | [25th-50th percentile)   |
| high          | [50th-75th percentile)   |
| very_high     | [75th-100th percentile\] |

**Cleaned Data Frame**  

| name                                 |     id |   minutes | submitted   |   rating |   avg_rating_per_recipe |   calories (#) |   total fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated fat (PDV) |   carbohydrates (PDV) | calorie_range   |
|:-------------------------------------|-------:|----------:|:------------|---------:|------------------------:|---------------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|:----------------|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27  |        4 |                       4 |          138.4 |                10 |            50 |              3 |               3 |                    19 |                     6 | very_low        |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11  |        5 |                       5 |          595.1 |                46 |           211 |             22 |              13 |                    51 |                    26 | very_high       |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |        5 |                       5 |          194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 | low             |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |        5 |                       5 |          194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 | low             |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |        5 |                       5 |          194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 | low             |

### Univariate Analysis  

We created 3 plots for univariate analysis  

Our first plot is a bar plot of the counts of the recipes of each rating in the Dataset. 

<iframe
  src="assets/ratings_count_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>  

As shown by the plot, most of the recipes in the ratings have a rating of 5, with over 169K recipes given the rating of a 5. We need to keep this in mind, as it could lead to our models not performing well for prediciting ratings below 5.  

The following plot is a box plot of the calorie values in the dataset.  

<iframe
  src="assets/calorie_box.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>  

As we can see here, there are a lot of outlier values in the dataset that make it difficult to see the general distribution. To be able to better understand how the data is distributed, we will remove outliers based on if the data point falls outside of the range [-1.5 * IQR, 1.5 * IQR\] where IQR is the interquartile range (75th percentile - 25th percentile).  

The histogram of the calories is as follows:  

<iframe
  src="assets/calories_histogram.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>  

The histogram shows that the distribution of the data is heavily right skewed, with the most data points falling between 125-175 calories. After reviewing the data, it was found that almost 200000 entries were considered outliers based on our criteria. We will keep them in the data for the analysis that follows.  

### Bivariate Analysis

For our bivariate analysis, we looked at the two variables, calories and ratings to see if there are any relationships.  

<iframe
  src="assets/cal_rating_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>  

The scatter plot above explores the relationship between calories and ratings. We can see that a majority of the points are densely packed between 0 to 10,000 calories. From this plot we can also see a slight trend in higher stars being related higher caloric content. Could this mean that people enjoy higher calorie recipes more?  

To answer this question, we created a stacked bar chart of the count of recipes within the calorie ranges and the ratings distribution within the calories ranges. This allows us to see whether or not people give higher ratings to recipes with higher calories.  

<iframe
  src="assets/stacked_bar_chart.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>  

The plot above shows the count of the number of recipes for each calorie range, but each bar is then further separated into ratings. As we can see, across all calorie ranges, there is an even spread of ratings scores, with most recipes across all calorie ranges having a rating of 5.  

### Interesting Aggregate  

A good indicator that a rating is going to be good or not is the nutritional components of the food. For example, a food with balanced nutritional values or foods that are fried may have higher ratings than foods that are low calorie. Interestingly, when we aggregate accross rating and get the mean of all the nutritional values, we can see that higher calorie foods recieved lower ratings. Along with sugary and high sodium recipes.  

|   rating |   calories (#) |   total fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated fat (PDV) |   carbohydrates (PDV) |
|---------:|---------------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|
|        1 |        486.595 |           37.0564 |       88.0094 |        44.3564 |         34.0589 |               46.6791 |               16.3962 |
|        2 |        446.598 |           32.7669 |       75.1664 |        29.5557 |         34.307  |               42.8822 |               15.0405 |
|        3 |        425.791 |           31.6405 |       65.552  |        27.938  |         34.8582 |               40.0881 |               13.6813 |
|        4 |        405.047 |           29.9404 |       56.7858 |        27.0146 |         34.0466 |               36.4327 |               12.8306 |
|        5 |        415.213 |           31.7924 |       63.0816 |        29.1707 |         32.6446 |               39.2268 |               13.038  |  

## Asessment of Missngness  

### NMAR Analysis  

The column that we believe could be NMAR is the `description` column. This is due to the fact that people writing the recipes would be more likely to not write a description if they believe the name is descriptive enough to not warrent a description. Which meets the criteria that the missingness is dependent on what the value would have been if it was not missing.  

### Missingness Dependency  

For the analysis of missingness, we decided to use the `ratings` column.  

> Ratings vs Calories  

We will first investigate if the missingness of the `ratings` column depends on the `calories` column.  

- **Null Hypothesis:** The missingness of ratings does not depend on the number of calories in the recipe.  
- **Alternative Hypothesis:** The missingness of ratings does depend on the number of calories in the recipe.  

- **Test Statistic:** The test statistic that we are going to use for this permutation test is the K-S test statistic. Since the K-S test statistic only captures the difference between the 2 groups, it is an ideal test statistic for this situation.  

<iframe
  src="assets/perm_test_ratings_calories.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>  

The observed statistic was 0.053. When we ran the permutation test, we got a p-value of $7.96 \times 10^{-35}$, there is significant evidence that the missingness in the `rating` column is dependent on the `calories` column. The plot above also shows this fact. All of the distribution of the test statistics lies to the left of the test statistic, indicating that we can reject the null hypothesis, and that there is significant evidence toward the alternate hypothesis that the missingness of the `ratings` column depends on the `calories` column.  

> Ratings vs Minutes  

We also looked at whether the missingness of the `ratings` columnd depends on the `minutes` column.  

- **Null Hypothesis:** The missingness of `ratings` does not depend on the number of `minutes` it takes to cook the recipe.  
- **Alternate Hypothesis:** The missingness of `ratings` does depend on the number of `minutes` it takes to cook the recipe.  

- **Test Statistic:** For this permutation, we also used a K-S test statistic.  

We calculated an observed statistic of 0.0935 and a p-value of 0.135, which is greater than the significance level of 0.05. This means that we can not reject the null hypothesis that the missingness of the `ratings` column does not depend on the `minutes` column.  

## Hypothesis Testing

From the aggregate by ratings dataframe, we observe that the average calorie count of lower rated foods is higher than the average calorie count of higher rated foods. We decided to run a hypothesis test to see if there is any statistical merit to this observation or not.  

We are going to set up a permutation test with the following hypotheses:  
- Null Hypothesis ($H_0$): The average rating of high-calorie recipes is equal to the average rating of low-calorie recipes.  
- Alternative Hypothesis ($H_A$): The average rating of high-calorie recipes is lower than the average rating of low-calorie recipes.

The test statistic we used is the difference in mean ratings between high-calorie and low-calorie recipes.  

Mean Rating (High Calorie) − Mean Rating (Low Calorie)  

*Note we will be combining the `very_low` column to `low` and `very_high` column to `high` for this permutation test.*  

<iframe
  src="assets/hypo_high_low_cal_avg_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>  

Significance Level: 
- We set our significance level ($\alpha$) to 0.05, which is a common threshold for determining statistical significance.  

Results  
- Our observed difference in mean ratings was -0.0172, indicating that high-calorie recipes have slightly lower average ratings than low-calorie recipes.  
- After running a permutation test with 1000 simulations, we derived a p-value of 0.0, indicating that we can reject the null hypothesis.  

Conclusion  
- Since the p-value ($0.0$) is much smaller than our significance level ($\alpha = 0.05$), we reject the null hypothesis. This means there is statistical evidence to suggest that high-calorie recipes receive lower ratings than low-calorie recipes.  

While there may appear to be a slight trend in the scatter plot, our statistical test confirms that this trend is likely due to random variation rather than a meaningful relationship between calorie content and recipe ratings.  

It could be the case that high calorie recipes are rated lower than low calorie recipes because people that cook their own food are usually very health conscious, which means that they want to lower their calorie intake and rate recipes that are high in calories lower than recipes that are lower in calorie.  

## Framing a prediction problem   
### 🎯 Prediction Problem  

We aim to predict the rating of a recipe based mostly on intrinsic characteristics of the food itself. Specifically, we will build a **multiclass classification model** that predicts whether recipes receive ratings of **1, 2, 3, 4, or 5 stars**.  

---  

### 📌 Response Variable  

Our response variable is **recipe rating**, an ordinal categorical variable with possible values: `{1, 2, 3, 4, 5}`.  

---  

### 🤔 Why Predict Ratings?  

Predicting user ratings helps food websites recommend recipes more effectively, improves user experience by highlighting high-quality recipes, and guides content creators toward developing recipes that users prefer.  

---  
  
### 📋 Features Used  

We will exclude columns that do not directly relate to intrinsic food properties or that are unavailable at prediction time. Specifically, we will exclude columns such as:  

- `name`
- `id`
- `contributor_id`
- `submitted`
- `tags`
- `steps`
- `ingredients`
- `description`
- `user_id`  

and any other columns dependent on user-upload style or external user actions.  

We will use recipe features such as:  

- ✅ **Nutritional values** (protein, fat, sugar content)
- ✅ **Preparation time** (`minutes`)
- ✅ **Number of steps** (`n_steps`)
- ✅ **Number of ingredients** (`n_ingredients`)  

These features are known at the moment a recipe is published and thus valid for predictive modeling.  

---

### 📊 Evaluation Metric  

We will use the **F1-score** as our evaluation metric because our dataset has imbalanced ratings (most recipes have ratings of 4 or 5 stars). Accuracy alone would be misleadingly high due to this imbalance. The F1-score balances precision and recall and provides a more informative measure of performance for imbalanced multiclass classification problems.  

## Baseline Model

## 🧪 Baseline Model  

Our baseline model will use the features **calories** (quantitative continuous) and **minutes** (quantitative continuous) to predict **ratings** (categorical ordinal: 1, 2, 3, 4, or 5). These features were chosen because we believe they are simple and directly related to user preferences for recipes. For instance, recipes with fewer calories may appeal to health-conscious users, while recipes that take less time to prepare might be more convenient and receive higher ratings.  

---  

### ⚙️ Handling Missing Values  

To start, we will handle these missing values by imputing them with their respective medians, as  

---  

### 🌳 Modeling Algorithm  

Since this is a classification task, we chose a **Decision Tree Classifier** for predictions. Decision trees are interpretable and well-suited for handling both numerical and categorical data. Additionally, they can capture non-linear relationships between features and the target variable.  

---  

### Base Model Classification Report  
Weighted F1-score: 0.680   

|             | Precision | Recall | F1-Score | Support |
|-------------|-----------|--------|----------|---------|
| **1.0**    | 0.09      | 0.06   | 0.07     | 576     |
| **2.0**    | 0.03      | 0.02   | 0.02     | 501     |
| **3.0**    | 0.06      | 0.04   | 0.05     | 1467    |
| **4.0**    | 0.26      | 0.19   | 0.22     | 7436    |
| **5.0**    | 0.80      | 0.86   | 0.83     | 33899   |
| **Accuracy** |         |        | 0.70     | 43879   |
| **Macro Avg** | 0.25   | 0.24   | 0.24     | 43879   |
| **Weighted Avg** | 0.66| 0.70   | 0.68     | 43879   |

| Feature | Weight |
|--------------|---------|
| Calories (#) | 0.7573  |
| Minutes | 0.2427 |  

## 📈 Analysis of Decision Tree Results (Baseline Model)

### ⚠️ **Class Imbalance**

The classification report clearly shows significant class imbalance in our dataset:

- **Rating 5.0**: 33,899 samples (majority class)
- **Rating 4.0**: 7,436 samples
- **Rating 3.0**: 1,467 samples
- **Rating 2.0**: 501 samples
- **Rating 1.0**: 576 samples

Due to this imbalance, the model tends to predict the majority class (**5.0**) more frequently, resulting in:

- ✅ High precision and recall for rating **5.0**
- ❌ Very low precision and recall for underrepresented classes (**1.0**, **2.0**, **3.0**, and **4.0**)

---

### 📊 **Performance Metrics**

- **Accuracy (0.70)**:  
  At first glance, accuracy seems decent; however, this is misleading due to class imbalance. Predicting the majority class (**5.0**) correctly inflates accuracy.

- **Macro Average (Precision: 0.25, Recall: 0.24, F1-score: 0.24)**:  
  This metric treats all classes equally and clearly shows that the model struggles significantly with minority classes.

- **Weighted Average (Precision: 0.66, Recall: 0.70, F1-score: 0.68)**:  
  This metric accounts for class imbalance by weighting each class according to its frequency in the dataset. It provides a more realistic assessment of overall performance—still moderate but notably higher than the macro average due to strong performance on the dominant class (**5.0**).

---

### 🌳 **Feature Importances**

The decision tree analysis provided the following feature importances:

| Feature          | Importance |
|------------------|------------|
| `calories (#)`   | **75.73%** |
| `minutes`        | **24.27%** |

This indicates that the feature **calories (#)** plays a significantly larger role in predicting recipe ratings compared to preparation time (**minutes**).

---

### 🎯 **Conclusion & Next Steps**

Our baseline model demonstrates moderate overall performance (weighted F1-score = **0.677**) but clearly struggles with minority rating classes due to severe class imbalance.

To improve our model in subsequent analyses, we decided to:

- ✅ Incorporating additional relevant features beyond calories and preparation time.
- ✅ Tuning hyperparameters  by incorporating more features and using **GridSearchCV** to find the optimal hyperparameters.

### Final Model Classification Report  
Test Weighted F1-score: 0.686  

|               | Precision | Recall | F1-Score | Support |
|---------------|-----------|--------|----------|---------|
| 1.0           | 0.12      | 0.03   | 0.05     | 574     |
| 2.0           | 0.08      | 0.01   | 0.02     | 474     |
| 3.0           | 0.08      | 0.01   | 0.02     | 1434    |
| 4.0           | 0.29      | 0.07   | 0.11     | 7461    |
| 5.0           | 0.78      | 0.96   | 0.86     | 33936   |
| Accuracy      |           |        | 0.75     | 43879   |
| Macro Avg     | 0.27      | 0.22   | 0.21     | 43879   |
| Weighted Avg  | 0.66      | 0.75   | 0.69     | 43879   |

| Feature             | Importance |
|---------------------|------------|
| minutes             | 0.074      |
| n_steps             | 0.062      |
| n_ingredients       | 0.053      |
| user_id             | 0.363      |
| calories (#)        | 0.072      |
| total fat (PDV)     | 0.054      |
| sugar (PDV)         | 0.075      |
| sodium (PDV)        | 0.073      |
| protein (PDV)       | 0.073      |
| saturated fat (PDV) | 0.058      |
| carbohydrates (PDV) | 0.044      |

## 🎯 Final Model 

To increase our F1 score from our baseline model, we added the following features:

| 🧩 **Feature**         | 📊 **Type**           | 💡 **Reason**                                                                                     |
|-------------------------|-----------------------|---------------------------------------------------------------------------------------------------|
| n_steps                 | Quantitative nominal  | Represents the complexity of the recipe; recipes with fewer steps might be rated higher due to convenience. |
| n_ingredients           | Quantitative nominal  | Indicates simplicity of the recipe; simpler recipes may appeal to more users.                   |
| total fat (PDV)         | Quantitative ordinal  | Higher fat content may negatively impact ratings due to health concerns.                        |
| sugar (PDV)             | Quantitative ordinal  | Recipes with high sugar content may be rated lower due to health risks.                         |
| sodium (PDV)            | Quantitative ordinal  | High sodium levels may reduce ratings also due to health-related risks.                         |
| protein (PDV)           | Quantitative ordinal  | Recipes high in protein may appeal to fitness-conscious users, improving ratings.               |
| saturated fat (PDV)     | Quantitative ordinal  | High saturated fat content may negatively impact ratings for health-conscious users.            |
| carbohydrates (PDV)     | Quantitative ordinal  | Also a nutritional value that can affect how people view the recipe.                            |

---

### 🧹 Data Cleaning

To start, we dropped the rows with `np.nan` in the ratings column. We did this because if we imputed the value with the average rating or median, this would introduce bias. This is because people who viewed the recipe negatively would likely not want to leave a review. By dropping the values, we opted to take a safer route without introducing bias.

---

### ⚙️ Modeling Approach

For our actual model, we used **GridSearchCV** to find the best hyperparameters for three different algorithms. We ran GridSearchCV on **KNNClassifier**, **DecisionTreeClassifier**, and **RandomForestClassifier** and found our **DecisionTreeClassifier** had the highest F1 score of **0.686**. 

We also found our optimal hyperparameters:
> 🛠️ `'classifier__max_depth': 10`  
> 🛠️ `'classifier__min_samples_split': 14`

Our average precision and recall are:
> 📏 `avg_precision`: 0.66  
> 📏 `avg_recall`: 0.75  

---

### 📊 Baseline vs Final Model Performance

#### 🟢 Baseline Model:
- **Features:** `['calories (#)', 'minutes']`  
- **Algorithm:** Decision Tree Classifier  
- **Weighted F1 Score:** ~0.680  

**F1 Scores for Each Rating:**
> ⭐ Rating 1: 0.07  
> ⭐ Rating 2: 0.02  
> ⭐ Rating 3: 0.05  
> ⭐ Rating 4: 0.22  
> ⭐ Rating 5: 0.83  

---

#### 🔵 Final Model:
- **Algorithm:** Decision Tree Classifier  
- **Weighted F1 Score:** ~0.686  

**F1 Scores for Each Rating:**
> ⭐ Rating 1: 0.05  
> ⭐ Rating 2: 0.02  
> ⭐ Rating 3: 0.02  
> ⭐ Rating 4: 0.11  
> ⭐ Rating 5: 0.86  

---

### 🌟 Top 3 Optimal Features Ranked by SKLearn Importance Metric:

| 🧩 Feature          | 🔑 Importance |
|---------------------|--------------|
| user_id            | 0.363        |
| sugar (PDV)        | 0.075        |
| sodium (PDV)       | 0.073        |

---

### 📌 Conclusion

The final model improved the F1 score slightly from **0.680** to **0.686**. However, we likely did not achieve the improvements we were hoping for simply because of how skewed the dataset is for **5-star ratings**. This is also reflected in our F1 score per rating, as the F1 score for **5-star ratings** was the only one that increased significantly from our baseline model.

## 🎯 Fairness Analysis

### **Groups**
- **Group 1:** Low-calorie recipes (`calorie_range == 'low'`)
- **Group 2:** High-calorie recipes (`calorie_range == 'high'`)

We chose these groups because calorie content is the most important feature, and we want to evaluate whether the model performs differently for recipes with lower calories compared to those with higher calories.

---

### **Evaluation Metric**
The evaluation metric for this fairness analysis is the **F1-score**. This metric balances precision and recall, making it appropriate for imbalanced datasets like ours.

---

### **Hypotheses**
- **Null Hypothesis (H₀):** The model is fair. Its F1-scores for predicting ratings of low-calorie recipes and high-calorie recipes are the same, and any differences are due to random chance.
- **Alternate Hypothesis (H₁):** The model is not fair. Its F1-scores for predicting ratings of low-calorie recipes are different from those of high-calorie recipes.

---

### **Test Statistic**
The test statistic is the difference in weighted F1-scores:
$$
\text{Test Statistic} = \text{F1}_{\text{high-calorie}} - \text{F1}_{\text{low-calorie}}
$$

---

### **Significance Level**
The significance level for this test is:
$$
\alpha = 0.05
$$

---

### **Results**
In our test set, we observe:
- **Observed Test Statistic:** `-0.0144`
    - This negative value indicates that the model performs slightly better on low-calorie recipes compared to high-calorie recipes.
- **P-value:** `0.01`

After running 1,000 simulations under the null hypothesis, we observe a p-value of `0.01`

---

### **Conclusion**
Since the p-value of (`0.01`) is less than our significance level (`α = 0.05`), we can safely reject the null hypothesis.

This means that our model is likely unfair and is most likely due to the lack of variance in low calorie recipes. 

