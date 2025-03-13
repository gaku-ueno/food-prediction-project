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

