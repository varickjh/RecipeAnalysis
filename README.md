# Understanding Recipe Ratings and Building Prediction Models to Predict Calories

**Authors:** Clayton Hammen Tan, Varick Janiro Hasim

## Overview

The goal of this project is to analyze which recipe factors impact calories and to predict a recipe's calories accurately based on its other attributes (e.g., average rating, number of steps, number of ingredients). Using two datasets (recipes and user interactions), we clean and explore the data, test hypotheses, build predictive models, and ensure fairness across different recipe categories.

## Introduction

This project investigates how various factors contribute to a recipe's calories using data from Food.com. The datasets include:

- **Recipes Dataset:** Over 83,000 recipes with details such as preparation time, nutritional information, and ingredients.
- **Interactions Dataset:** Over 730,000 user reviews and ratings.

By merging these datasets, we explore trends in calorie content and predict calories based on recipe attributes.

### Research Question

**What factors influence the calories and average rating of a recipe?**

This question helps determine if more complex recipes tend to have higher calories and different satisfaction levels. The insights can guide home cooks, food bloggers, and recipe developers.

### Dataset Description

#### Recipes Dataset

| Column Name      | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| **name**         | Name of the recipe, providing a unique identifier for each dish.           |
| **id**           | Unique Recipe ID.                                                           |
| **minutes**      | Total preparation time (in minutes).                                        |
| **contributor_id** | ID of the user who submitted the recipe.                                  |
| **submitted**    | Date when the recipe was submitted.                                         |
| **tags**         | Categories associated with the recipe (e.g., cuisine or meal type).         |
| **nutrition**    | Nutritional information including calories, fat, protein, and carbohydrates.|
| **n_steps**      | Number of steps in the recipe instructions.                                 |
| **steps**        | Detailed recipe instructions.                                               |
| **description**  | User-provided description of the recipe.                                    |
| **ingredients**  | List of ingredients used.                                                   |
| **n_ingredients** | Total number of ingredients used.                                         |
| **average_rating** | Average rating, calculated from user interactions.                      |

#### Interactions Dataset

| Column Name | Description                                           |
|-------------|-------------------------------------------------------|
| **user_id** | ID of the user providing the rating or review.       |
| **recipe_id** | ID of the reviewed recipe.                         |
| **date**    | Date of the interaction (review or rating).           |
| **rating**  | Rating given by the user.                             |
| **review**  | Textual review provided by the user (optional).       |

### Importance of the Study

Understanding which recipe attributes influence calories has practical and academic benefits. It helps recipe developers and everyday cooks estimate recipe calories and demonstrates robust data analysis and predictive modeling techniques applicable to other recommendation systems.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The following cleaning steps were performed:

1. **Merged the Datasets:**  
   The `recipes` and `interactions` datasets were merged using a left join on `id` and `recipe_id` so that all recipes were retained.

2. **Replaced Invalid Ratings:**  
   Ratings of 0 in the interactions dataset were replaced with `NaN`, treating them as missing rather than true ratings.

3. **Calculated Average Ratings:**  
   Average ratings were computed for each recipe (grouped by `name`) and added as a new column `average_rating`, rounded to two decimals.

4. **Extracted Nutritional Information:**  
   The `nutrition` column (a string) was split into separate columns for calories, total_fat, sugar, sodium, protein, saturated_fat, and carbohydrates.

5. **Created Additional Binary Columns:**  
   - `simple`: True if the number of ingredients ≤ 9.
   - `easy`: True if the number of steps ≤ 9.
   - `healthy`: True if the ratio of saturated fat to calories ≤ 0.10.

6. **Removed Outliers:**  
   Outliers (e.g., recipes with 40,000 calories) were removed.

*Effect on Analysis:*  
- Replacing invalid ratings and computing average ratings provides more accurate insights.  
- Splitting nutrition data facilitates focused analysis of individual nutritional factors.  
- Merging datasets offers a comprehensive view of recipes and user preferences.

---

### Univariate Analysis

#### Plot 1: Distribution of Number of Ingredients

This histogram shows the typical number of ingredients per recipe.

- **Key Observations:**
  - A clear peak around 9–10 ingredients suggests moderate complexity.
  - Very high-ingredient recipes (over 20) are rare.

<iframe
  src="assets/N_Ingredients_Distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Plot 2: Distribution of Calories in Recipes

This density-based histogram shows the calorie content across recipes.

- **Key Observations:**
  - The distribution is right-skewed, with many recipes under 300 calories.
  - A tail exists for recipes with higher calories.

<iframe
  src="assets/Calories_Distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

### Bivariate Analysis

**Scatter Plot: Calories vs. Average Rating**

<iframe
  src="assets/CaloriesAvgRatingScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot suggests that while recipes with high ratings span a range of calorie values, calorie count alone may not drive ratings.

**Box Plot: Average Rating by Number of Steps**

<iframe
  src="assets/IngredientsAverageRatingBox.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This box plot indicates that recipes with fewer ingredients have more consistent ratings, while those with more ingredients show greater variability.

### Interesting Aggregates

**Pivot Table: Calories and Ratings by Cooking Time (Minutes)**

The table below shows how cooking time correlates with satisfaction and caloric content:

| average_rating | calories   |
|----------------|------------|
| 5              | 653.1      |
| 4.73875       | 195.705    |
| 4.72001       | 223.325    |
| 4.6944        | 253.525    |
| 4.53713       | 271.961    |

---

## Assessment of Missingness

### NMAR Analysis: Understanding Missingness in `average_rating`

The `average_rating` field exhibits NMAR (Not Missing At Random) behavior, meaning its missingness is driven by factors not explicitly captured in the dataset.

#### Why is `average_rating` NMAR?

- **User Behavior:**  
  Some users may not rate recipes if they did not follow the recipe strictly or felt indifferent.

- **Nature of Missingness:**  
  Such behaviors stem from user-specific preferences not reflected in the data.

#### How Could We Make It MAR?

Additional data (e.g., user interaction frequency, contextual details, engagement metrics) could link the missingness to observable factors, allowing it to be treated as MAR (Missing At Random).

### Missingness Dependency: Investigating Relationships

Permutation tests were conducted to determine if the missingness of `average_rating` depends on other recipe features. For each test:

- **H₀ (Null Hypothesis):** Missingness does not depend on the tested column.
- **Hₐ (Alternative Hypothesis):** Missingness depends on the tested column.
- **Test Statistic:** Difference in means of the column between recipes with missing and non-missing `average_rating`.

#### Detailed Results

1. **Missingness vs. `calories`**  
   - Observed Test Statistic: 19.308042  
   - P-value: 0.000  
   <iframe src="assets/DependencyMissingnesscalories.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Depends on `calories`.

2. **Missingness vs. `healthy`**  
   - Observed Test Statistic: 0.026784  
   - P-value: 0.004  
   <iframe src="assets/DependencyMissingnesshealthy.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Depends on `healthy`.

3. **Missingness vs. `simple`**  
   - Observed Test Statistic: 0.024634  
   - P-value: 0.026  
   <iframe src="assets/DependencyMissingnesssimple.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Depends on `simple`.

4. **Missingness vs. `easy`**  
   - Observed Test Statistic: 0.073634  
   - P-value: 0.000  
   <iframe src="assets/DependencyMissingnesseasy.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Depends on `easy`.

5. **Missingness vs. `sugar`**  
   - Observed Test Statistic: 9.587501  
   - P-value: 0.000  
   <iframe src="assets/DependencyMissingnesssugar.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Depends on `sugar`.

6. **Missingness vs. `protein`**  
   - Observed Test Statistic: 0.993006  
   - P-value: 0.120  
   <iframe src="assets/DependencyMissingnessprotein.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Does not depend on `protein`.

7. **Missingness vs. `sodium`**  
   - Observed Test Statistic: 0.828399  
   - P-value: 0.622  
   <iframe src="assets/DependencyMissingnesssodium.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Does not depend on `sodium`.

8. **Missingness vs. `n_ingredients`**  
   - Observed Test Statistic: 0.259784  
   - P-value: 0.000  
   <iframe src="assets/DependencyMissingnessn_ingredients.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Depends on `n_ingredients`.

9. **Missingness vs. `n_steps`**  
   - Observed Test Statistic: 1.471825  
   - P-value: 0.000  
   <iframe src="assets/DependencyMissingnessn_steps.html" width="800" height="600" frameborder="0"></iframe>  
   **Conclusion:** Depends on `n_steps`.

10. **Missingness vs. `saturated_fat`**  
    - Observed Test Statistic: 3.117959  
    - P-value: 0.000  
    <iframe src="assets/DependencyMissingnesssaturated_fat.html" width="800" height="600" frameborder="0"></iframe>  
    **Conclusion:** Depends on `saturated_fat`.

#### Summary Table

| Tested Column    | Observed Statistic | P-Value | Conclusion                                   |
|------------------|--------------------|---------|----------------------------------------------|
| `calories`       | 19.308042          | 0.000   | Depends on `calories`                        |
| `healthy`        | 0.026784           | 0.004   | Depends on `healthy`                         |
| `simple`         | 0.024634           | 0.026   | Depends on `simple`                          |
| `easy`           | 0.073634           | 0.000   | Depends on `easy`                            |
| `sugar`          | 9.587501           | 0.000   | Depends on `sugar`                           |
| `protein`        | 0.993006           | 0.120   | Does not depend on `protein`                 |
| `sodium`         | 0.828399           | 0.622   | Does not depend on `sodium`                  |
| `n_ingredients`  | 0.259784           | 0.000   | Depends on `n_ingredients`                   |
| `n_steps`        | 1.471825           | 0.000   | Depends on `n_steps`                         |
| `saturated_fat`  | 3.117959           | 0.000   | Depends on `saturated_fat`                   |

**Summary:**  
Missingness in `average_rating` is significantly associated with `calories`, `healthy`, `simple`, `easy`, `sugar`, `n_ingredients`, `n_steps`, and `saturated_fat` (p < 0.05) but not with `protein` or `sodium`.

---

## Hypothesis Testing

### Objective

Determine whether a recipe's number of preparation steps affects its average rating.

### Hypotheses

- **Null Hypothesis:** The mean average rating of recipes does not depend on the number of steps (recipe complexity).
- **Alternative Hypothesis:** The mean average rating of recipes does depend on the number of steps.

### Test Statistic & Significance Level

The test statistic is the absolute difference in the mean average rating between:
- **Easy Recipes:** `n_steps` ≤ 9.
- **Hard Recipes:** `n_steps` > 9.

**Significance Level:** 0.05

### Methodology

1. Calculate the observed test statistic (absolute difference in means).
2. Shuffle the group labels (≤9 steps vs. >9 steps) 1,000 times to simulate the null hypothesis.
3. Compute the p-value as the proportion of permuted statistics ≥ the observed statistic.

### Results & Visualization

- **Observed Statistic:** ~0.000787  
- **P-Value:** ~0.87  

<iframe
  src="assets/HypothesisTest.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Conclusion

Since the p-value (~0.87) is greater than 0.05, we fail to reject the null hypothesis. This suggests that the number of preparation steps does not significantly influence the average rating.

---

## Framing a Prediction Problem

### Prediction Problem Statement

Predict the total calories (`calories`) of a recipe based on its characteristics:
- Number of preparation steps (`n_steps`)
- Number of ingredients (`n_ingredients`)
- Protein content (`protein`)
- Sodium content (`sodium`)
- Saturated fat content (`saturated_fat`)
- Sugar content (`sugar`)

Predicting calories helps understand how complexity and nutritional content contribute to a recipe’s caloric value, guiding recipe creation and dietary choices.

### Type of Prediction Problem

This is a **regression** problem since the target variable (`calories`) is continuous (ranging from 58 to 1036).

### Response Variable

**Calories** – Represents the nutritional content and indirectly the healthiness of the recipe.

### Features Used for Prediction

- **n_ingredients:** Recipe complexity in terms of ingredients.
- **n_steps:** Recipe complexity in terms of preparation steps.
- **protein:** Protein content.
- **sodium:** Sodium content.
- **saturated_fat:** Saturated fat content.
- **sugar:** Sugar content.

All features are known at the time of prediction.

### Evaluation Metric

We use:
- **Mean Absolute Error (MAE):** Provides an average error in the same units as calories and is robust to outliers.
- **R² (Coefficient of Determination):** Indicates how well the model fits the data.

### Visual Exploration of the Problem

1. **Distribution of Calories:**  
   <iframe src="assets/CaloriesDistribution2.html" width="800" height="600" frameborder="0"></iframe>  
   (Right-skewed distribution with most recipes having lower calories.)

2. **Scatter Plot: n_ingredients vs. Calories:**  
   <iframe src="assets/n_ingredientsCaloriesScatter.html" width="800" height="600" frameborder="0"></iframe>  
   (No clear relationship observed.)

3. **Scatter Plot: n_steps vs. Calories:**  
   <iframe src="assets/n_stepsCaloriesScatter.html" width="800" height="600" frameborder="0"></iframe>  
   (Most recipes have ≤40 steps; relationship is unclear.)

4. **Scatter Plot: Protein vs. Calories:**  
   <iframe src="assets/proteinCaloriesScatter.html" width="800" height="600" frameborder="0"></iframe>  
   (Positive relationship observed.)

5. **Scatter Plot: Sodium vs. Calories:**  
   <iframe src="assets/sodiumCaloriesScatter.html" width="800" height="600" frameborder="0"></iframe>  
   (No clear relationship.)

6. **Scatter Plot: Saturated Fat vs. Calories:**  
   <iframe src="assets/saturated_fatCaloriesScatter.html" width="800" height="600" frameborder="0"></iframe>  
   (Positive relationship observed.)

7. **Scatter Plot: Sugar vs. Calories:**  
   <iframe src="assets/sugarCaloriesScatter.html" width="800" height="600" frameborder="0"></iframe>  
   (Positive relationship observed.)

### Importance

Predicting calories benefits:
- **Health Enthusiasts:** For tailored dietary plans.
- **Cooks:** To design recipes that meet specific nutritional requirements.
- **Recipe Insights:** Understanding how complexity and nutrition combine to affect calorie content.

---

## Baseline Model

The baseline model is a Linear Regression model built using a preprocessing pipeline.

### Model Features

#### Numerical Features

- **n_steps**
- **n_ingredients**
- **protein**
- **sodium**
- **saturated_fat**
- **sugar**

(Standardization was not applied since it does not affect predictions in linear regression.)

#### Categorical Feature

- **ingredients:** Processed via OneHotEncoder.

### Model Performance

**Metrics:**
- **R²:** 0.7624704207215018
- **MAE:** 76.76761605484424

**Coefficients:**
- **n_steps:** 0.2255  
- **n_ingredients:** 4.2917  
- **protein:** 3.4293  
- **sodium:** 0.1105  
- **saturated_fat:** 3.0411  
- **sugar:** 0.9368  
- **ingredients:** 0.0000  

The results suggest that `n_ingredients`, `protein`, and `saturated_fat` are significant predictors of calorie content.

### Model's Strengths and Weaknesses

**Strengths:**
1. Reproducible and scalable.
2. Effective handling of categorical data via one-hot encoding.
3. Interpretability of coefficients.

**Weaknesses:**
1. Prone to outliers.
2. Assumes linear relationships, which may not hold for all predictors.

---

## Final Model

### Added Features

1. **complexity:** `n_steps` + `n_ingredients`  
   *(Captures overall recipe complexity.)*
2. **bad_nutrition:** `sugar` + `saturated_fat` + `sodium`  
   *(Captures cumulative negative nutritional impact.)*

### Modeling Algorithm

A **Random Forest Regressor** was used because it:
- Handles both numerical and categorical data.
- Captures non-linear relationships.
- Allows hyperparameter tuning.

### Hyperparameter Tuning

Using GridSearchCV (3-fold CV), the best hyperparameters were:
- **max_depth:** 20
- **min_samples_split:** 10
- **n_estimators:** 200

### Performance Comparison

**Baseline Model:**
- **MAE:** 76.7585
- **R²:** 0.7634

**Final Model:**
- **MAE:** 67.0454
- **R²:** 0.8076

### Conclusion

The final model’s lower MAE and higher R² demonstrate improved accuracy. The enhancements are attributed to the new features and the non-linear modeling capability of the Random Forest Regressor.

<iframe src="assets/MAEPerformance.html" width="800" height="600" frameborder="0"></iframe>  
<iframe src="assets/R2Performance.html" width="800" height="600" frameborder="0"></iframe>

---

## Fairness Analysis

A fairness analysis was conducted by comparing model performance (MAE) between two groups:

- **Simple Recipes:** ≤ 9 ingredients.
- **Complex Recipes:** > 9 ingredients.

### Hypotheses

- **Null Hypothesis (H₀):** The model’s MAE does not differ significantly between Simple and Complex Recipes.
- **Alternative Hypothesis (Hₐ):** The model’s MAE differs significantly between the two groups.

### Test Statistic

The absolute difference in MAE between the two groups.

### Significance Level

α = 0.05

### Results

- **Observed MAE Difference:** ~0.0031  
- **P-Value:** 0.0  

Since the p-value is below 0.05, we reject H₀, indicating that the model's performance differs between Simple and Complex Recipes. This suggests potential unfairness that warrants further investigation.

<iframe src="assets/FairnessAnalysis.html" width="800" height="600" frameborder="0"></iframe>
