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

| Calorie Range | Calories                |
|---------------|-------------------------|
| Low           | [0–25th percentile)     |
| Medium        | [25th-50th percentile)  |
| High          | [50th-75th percentile)  |
| Very High     | [75th-100th percentile) |

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


**Interesting Aggregate**  

|   rating |   calories (#) |   total fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated fat (PDV) |   carbohydrates (PDV) |
|---------:|---------------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|
|        1 |        486.595 |           37.0564 |       88.0094 |        44.3564 |         34.0589 |               46.6791 |               16.3962 |
|        2 |        446.598 |           32.7669 |       75.1664 |        29.5557 |         34.307  |               42.8822 |               15.0405 |
|        3 |        425.791 |           31.6405 |       65.552  |        27.938  |         34.8582 |               40.0881 |               13.6813 |
|        4 |        405.047 |           29.9404 |       56.7858 |        27.0146 |         34.0466 |               36.4327 |               12.8306 |
|        5 |        415.213 |           31.7924 |       63.0816 |        29.1707 |         32.6446 |               39.2268 |               13.038  |

**Base Model Classification Report**  
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

**Final Model Classification Report**  
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

