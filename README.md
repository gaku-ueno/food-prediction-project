# Analysis of Food.com recipe features and their ratings

Authors: Gaku Ueno & Neil Dewan

## Introduction
In a world increasingly shaped by the internet, one constant remains: people's love for food. Online recipe platforms like Food.com have become essential resources for home cooks, culinary enthusiasts, and professional chefs alike. With over 500,000 user-generated recipes and ratings, an important question arises—what factors contribute to a high recipe rating? Out of the features in the dataset, we will examine which ones contribute the most to Ratings, specifically looking at the calories and t

This research aims to analyze recipe and review data from Food.com to determine the relationship between calories and user ratings. Using recipes and ratings posted since 2008 on Food.com, originally utilized in the recommender system research paper Generating Personalized Recipes from Historical User Preferences by Majumder et al., we will apply multiple linear regression to uncover patterns and correlations. By understanding this relationship, our study seeks to provide insights that can help home cooks, food bloggers, and nutrition-conscious users optimize their recipes for better engagement and higher ratings.

## Dataframe
The first dataset, recipe, contains 83782 rows for unique recipes, with 10 columns recording the following:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe   

The second dataset, interactions, contains 731927 rows where each row is a review from the user on a recipe. The columns are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

**Cleaned Data Frame**
| name                                 |     id |   minutes | submitted   |   rating |   avg_rating_per_recipe |   calories (#) |   total fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated fat (PDV) |   carbohydrates (PDV) | calorie_range   |
|:-------------------------------------|-------:|----------:|:------------|---------:|------------------------:|---------------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|:----------------|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27  |        4 |                       4 |          138.4 |                10 |            50 |              3 |               3 |                    19 |                     6 | very_low        |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11  |        5 |                       5 |          595.1 |                46 |           211 |             22 |              13 |                    51 |                    26 | very_high       |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |        5 |                       5 |          194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 | low             |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |        5 |                       5 |          194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 | low             |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |        5 |                       5 |          194.8 |                20 |             6 |             32 |              22 |                    36 |                     3 | low             |

**Interesting Aggregate**
|   rating |   calories (#) |   total fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated fat (PDV) |   carbohydrates (PDV) |
|---------:|---------------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|
|        1 |        486.595 |           37.0564 |       88.0094 |        44.3564 |         34.0589 |               46.6791 |               16.3962 |
|        2 |        446.598 |           32.7669 |       75.1664 |        29.5557 |         34.307  |               42.8822 |               15.0405 |
|        3 |        425.791 |           31.6405 |       65.552  |        27.938  |         34.8582 |               40.0881 |               13.6813 |
|        4 |        405.047 |           29.9404 |       56.7858 |        27.0146 |         34.0466 |               36.4327 |               12.8306 |
|        5 |        415.213 |           31.7924 |       63.0816 |        29.1707 |         32.6446 |               39.2268 |               13.038  |
