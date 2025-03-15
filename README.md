# Understanding Recipe Ratings and Building Prediction models to predict Calories
Authors: Clayton Hammen Tan, Varick Janiro Hasim

## Overview
The goal of this project is to analyze the factors of a recipe that impact the calories of the recipe and then be able to predict a recipe's calories accurately based on the other attributes of the recipe, such as average rating, number of steps, number of ingredients, etc. Using two datasets of recipes and user interactions, we will achieve our goal by first, cleaning and exploring the datasets, test our hypotheses, build our predictive models, and then ensure that our models are fair across different categories of recipes.

## Introduction
This project investigates how different factors contribute to a recipe's calories based on the recipes on Food.com, a popular recipe-sharing platform. The dataset used in this analysis comprises of two parts: a 'recipes' dataset and an 'interactions' dataset. The recipes dataset contains detailed information about over 83,000 recipes, including preparation time, nutritional details, and ingredients. The interactions dataset provides over 730,000 user reviews and ratings, offering insights into user preferences and behavior. By combining these datasets, we are able to explore the trends in a recipe's calories and then predict a recipe's calories based on its other attributes.

### Research Question
**What factors influence the calories and average rating of a recipe?**  
This question helps uncover whether or not more complex recipes have more calories and provide more satisfaction. Insights derived from this analysis can educate home cooks, food bloggers, and recipe developers on whether or not a recipe they create or follow will generally have more or less calories, and will provide them with satisfaction.

### Dataset Description
The datasets include the following columns:

<h3>Recipes Dataset</h3>
<table>
  <thead>
    <tr>
      <th>Column Name</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>name</td>
      <td>Name of the recipe, providing a unique identifier for each dish.</td>
    </tr>
    <tr>
      <td>id</td>
      <td>Unique Recipe ID.</td>
    </tr>
    <tr>
      <td>minutes</td>
      <td>Total preparation time for the recipe (in minutes).</td>
    </tr>
    <tr>
      <td>contributor_id</td>
      <td>ID of the user who submitted the recipe.</td>
    </tr>
    <tr>
      <td>submitted</td>
      <td>Date when the recipe was submitted.</td>
    </tr>
    <tr>
      <td>tags</td>
      <td>Categories associated with the recipe (e.g., cuisine or meal type).</td>
    </tr>
    <tr>
      <td>nutrition</td>
      <td>Nutritional information including calories, fat, protein, and carbohydrates.</td>
    </tr>
    <tr>
      <td>n_steps</td>
      <td>Number of steps in the recipe instructions.</td>
    </tr>
    <tr>
      <td>steps</td>
      <td>Detailed recipe instructions.</td>
    </tr>
    <tr>
      <td>description</td>
      <td>User-provided description of the recipe.</td>
    </tr>
    <tr>
      <td>ingredients</td>
      <td>List of ingredients used in the recipe.</td>
    </tr>
    <tr>
      <td>n_ingredients</td>
      <td>Total number of ingredients used.</td>
    </tr>
    <tr>
      <td>average_rating</td>
      <td>The average rating of the recipe, calculated from user interactions.</td>
    </tr>
  </tbody>
</table>


<h3>Interactions Dataset</h3>
<table>
  <thead>
    <tr>
      <th>Column Name</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>user_id</td>
      <td>ID of the user who provided the rating or review.</td>
    </tr>
    <tr>
      <td>recipe_id</td>
      <td>ID of the recipe being reviewed.</td>
    </tr>
    <tr>
      <td>date</td>
      <td>Date of the interaction (review or rating).</td>
    </tr>
    <tr>
      <td>rating</td>
      <td>Rating given to the recipe by the user.</td>
    </tr>
    <tr>
      <td>review</td>
      <td>Textual review provided by the user (optional).</td>
    </tr>
  </tbody>
</table>




### Importance of the Study
Understanding the attributes of a recipe that influence a recipe's calories has both practical and academic significance. It helps recipe developers identify traits that should be avoided, such as certain complexities and nutritional contents, and the average person with insights to estimate the calories of the recipes they encounter. Moreover, this project showcases robust data analysis and predictive modeling techniques that can be applied to other recommendation systems.

## **Data Cleaning and Exploratory Data Analysis**

### Data Cleaning
To prepare the dataset for analysis, the following cleaning steps were performed:
1. **Merged the Datasets:**
   The `recipes` and `interactions` datasets were merged using a left join on the `id` and `recipe_id` columns. This ensured that every recipe in the `recipes` dataset was retained, even if it did not have user interactions.

2. **Replaced Invalid Ratings with Missing Values:**
   In the interactions dataset, ratings of 0 were replaced with `NaN` because a rating of 0 is likely to represent missing or invalid data rather than a true user rating.

3. **Calculated Average Ratings:**
  Computed the average rating for each recipe (grouped by 'name') and adds it as a new column 'average_rating', rounding the results to two decimal places.

4. **Extracted Nutritional Information:**
   Split the `nutrition` column (stored as a string) into separate columns for:
   - calories
   - total_fat
   - sugar
   - sodium
   - protein
   - saturated_fat
   - carbohydrates

   This allowed for easier analysis of nutritional factors and their relationship with ratings.

5. **Created additional binary columns for analysis:**
    - 'simple': True if the number of ingredients is less than or equal to 9.
    - 'easy': True if the number of steps is less than or equal to 9.
    - 'healthy': True if the ratio of saturated fat to calories is less than or equal to  0.10.

6. **Removed Outliers in the Dataset:**
    Removed the outliers in the dataset which had values that were clearly a mistake such as a recipe with 40,000 calories.

**Effect of Data Cleaning on Analysis:**
- Replacing invalid ratings and computing average ratings and grouping per recipe ensures accurate insights into a recipe's rating.
- Splitting the `nutrition` column facilitates the analysis of specific nutritional factors and their relationship with ratings.
- Merging the datasets provides a comprehensive view of recipes and user preferences, laying the foundation for further exploration.

---

### Univariate Analysis

#### Plot 1: Distribution of Number Of Ingredients
This histogram shows how many ingredients are typically used in each recipe.
- **Key Observations:**
  - There’s a clear peak around 9–10 ingredients, suggesting that most recipes fall within a moderate level of complexity.
  - Only a small fraction of recipes use more than 20 ingredients, indicating that very high-ingredient recipes are relatively rare.

<iframe
  src="assets/N_Ingredients_Distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Plot 2: Distribution of Calories in Recipes
This density-based histogram illustrates the calorie content across recipes.
- **Key Observations:**
  - The distribution is right-skewed, with a high density at the lower end (roughly under 300 calories) and a tail extending toward higher-calorie recipes.
  - While many recipes have moderate calorie counts, there is a subset with substantially higher calorie values, creating a long tail in the distribution.

<iframe
  src="assets/Calories_Distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---


### **Bivariate Analysis**

Scatter Plot: Calories vs. Average Rating

<iframe
  src="assets/CaloriesAvgRatingScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This scatter plot shows the relationship between the calorie count of recipes and their average ratings.

While there is no clear trend, most recipes with high ratings are distributed across a range of calorie values, suggesting that calorie count alone may not significantly influence ratings.

Box Plot: Average Rating by Number of Steps of a Recipe

<iframe
  src="assets/IngredientsAverageRatingBox.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The box plot displays how the number of preparation steps in a recipe affects its average rating.

Recipes with fewer ingredients tend to have more consistent ratings.
Recipes with a higher number of ingredients show greater variability in ratings, possibly reflecting the complexity and varying success of these recipes.

### **Interesting Aggregates**

**Pivot Table: Calories and Ratings by Cooking Time in Minutes**
The pivot table below shows the relationship of average rating of recipes and caloric content with cooking time in minutes. It shows how cooking time correlates with satisfaction and caloric content, with average rating decreasing and calories increasing as a recipe takes longer to cook.

|   average_rating |   calories |
|-----------------:|-----------:|
|          5       |    653.1   |
|          4.73875 |    195.705 |
|          4.72001 |    223.325 |
|          4.6944  |    253.525 |
|          4.53713 |    271.961 |

## Assessment of Missingness

### NMAR Analysis: Understanding Missingness in `average_rating`
The average_rating field appears to exhibit NMAR (Not Missing At Random) characteristics. In other words, the absence of ratings is driven by factors that are not explicitly captured in the dataset—namely, users’ personal choices—rather than by any directly observed variable.

#### Why is `average_rating` NMAR?
1. **User Behavior:** Some individuals may opt out of rating because they:
Did not strictly follow the recipe. Felt indifferent about the outcome (neither liking nor disliking it).
2. **Nature of Missingness:** These actions stem from user-specific tendencies or preferences not reflected in the data. For example, certain users might habitually refrain from rating recipes, regardless of their experience.

#### How Could We Make the Missingness into MAR?
To treat this missingness as MAR (Missing At Random), additional data about user actions and habits would be needed, such as:

**User Interaction Frequency:** How often users leave ratings.
**Contextual Details:** Whether they viewed, saved, or printed the recipe.
**Engagement Metrics:** Whether a recipe was bookmarked or favorited.

These extra insights could help explain why some users do not submit ratings, thereby linking missingness to observable factors and allowing it to be managed under the MAR framework.

### Missingness Dependency: Investigating Relationships
To further explore the missingness in `average_rating`, I conducted permutation tests to determine if the missingness depends on specific columns in the dataset. 
For each test, 
- **Null Hypothesis (H0):** is that the missingness of average_rating does not depend on the tested column.
- **Alternative Hypothesis (HA):** is that the missingness does depend on that column.
- **Test Statistic**: The difference in the means of the column between recipes with missing and non-missing `average_rating` values.

### Detailed Results

#### 1. Permutation Test: Missingness vs. `calories`
- **Results**:
  - Observed Test Statistic: **19.308042**
  - P-value: **0.000**

<iframe
  src="assets/DependencyMissingnesscalories.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `calories`.

---

#### 2. Permutation Test: Missingness vs. `healthy`
- **Results**:
  - Observed Test Statistic: **0.026784**
  - P-value: **0.004**

<iframe
  src="assets/DependencyMissingnesshealthy.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `healthy`.

---

#### 3. Permutation Test: Missingness vs. `simple`
- **Results**:
  - Observed Test Statistic: **0.024634**
  - P-value: **0.026**

<iframe
  src="assets/DependencyMissingnesssimple.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `simple`.

---

#### 4. Permutation Test: Missingness vs. `easy`
- **Results**:
  - Observed Test Statistic: **0.073634**
  - P-value: **0.000**

<iframe
  src="assets/DependencyMissingnesseasy.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `easy`.

---

#### 5. Permutation Test: Missingness vs. `sugar`
- **Results**:
  - Observed Test Statistic: **9.587501**
  - P-value: **0.000**

<iframe
  src="assets/DependencyMissingnesssugar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `sugar`.

---

#### 6. Permutation Test: Missingness vs. `protein`
- **Results**:
  - Observed Test Statistic: **0.993006**
  - P-value: **0.120**

<iframe
  src="assets/DependencyMissingnessprotein.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is greater than 0.05, we fail to reject the null hypothesis. The missingness in `average_rating` does **not** depend on `protein`.

---

#### 7. Permutation Test: Missingness vs. `sodium`
- **Results**:
  - Observed Test Statistic: **0.828399**
  - P-value: **0.622**

<iframe
  src="assets/DependencyMissingnesssodium.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is greater than 0.05, we fail to reject the null hypothesis. The missingness in `average_rating` does **not** depend on `sodium`.

---

#### 8. Permutation Test: Missingness vs. `n_ingredients`
- **Results**:
  - Observed Test Statistic: **0.259784**
  - P-value: **0.000**

<iframe
  src="assets/DependencyMissingnessn_ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `n_ingredients`.

---

#### 9. Permutation Test: Missingness vs. `n_steps`
- **Results**:
  - Observed Test Statistic: **1.471825**
  - P-value: **0.000**

<iframe
  src="assets/DependencyMissingnessn_steps.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `n_steps`.

---

#### 10. Permutation Test: Missingness vs. `saturated_fat`
- **Results**:
  - Observed Test Statistic: **3.117959**
  - P-value: **0.000**

<iframe
  src="assets/DependencyMissingnesssaturated_fat.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. The missingness in `average_rating` depends on `saturated_fat`.

---

### Test Results Summary

We performed permutation tests on various columns to explore the dependency of the missingness in `average_rating` on other recipe features. The table below summarizes the observed test statistics and p-values for each tested column:

| Tested Column    | Observed Statistic | P-Value | Conclusion                                    |
|------------------|--------------------|---------|-----------------------------------------------|
| `calories`       | 19.308042          | 0.000   | Missingness depends on `calories`             |
| `healthy`        | 0.026784           | 0.004   | Missingness depends on `healthy`              |
| `simple`         | 0.024634           | 0.026   | Missingness depends on `simple`               |
| `easy`           | 0.073634           | 0.000   | Missingness depends on `easy`                 |
| `sugar`          | 9.587501           | 0.000   | Missingness depends on `sugar`                |
| `protein`        | 0.993006           | 0.120   | Missingness does not depend on `protein`      |
| `sodium`         | 0.828399           | 0.622   | Missingness does not depend on `sodium`       |
| `n_ingredients`  | 0.259784           | 0.000   | Missingness depends on `n_ingredients`        |
| `n_steps`        | 1.471825           | 0.000   | Missingness depends on `n_steps`              |
| `saturated_fat`  | 3.117959           | 0.000   | Missingness depends on `saturated_fat`        |

---

**Summary**:  
The permutation tests indicate that the missingness in `average_rating` is significantly associated with the following features: `calories`, `healthy`, `simple`, `easy`, `sugar`, `n_ingredients`, `n_steps`, and `saturated_fat` (p-values < 0.05). In contrast, there is no statistically significant dependency between the missingness in `average_rating` and the features `protein` or `sodium` (p-values > 0.05).

## **Hypothesis Testing**


### **Objective**

The goal of this permutation test is to determine whether a recipe's number of preparation steps affects its average rating.

### **Hypotheses**

<ul>
<li><b>Null Hypothesis</b>: the mean average rating of recipes does not depend on its complexity (the number of steps needed).</li>

<li><b>Alternative Hypothesis</b>: the mean average rating of recipes does depend on its complexity (the number of steps needed).</li>

</ul>

### **Test Statistic & Significance Level**

The test statistic for our this hypothesis test is absolute difference in the mean average rating between two groups of recipes:
<ul>
<li>Easy Recipes, defined as recipes with number of steps to make less than or equal to nine steps ('n_steps' <= 9)</li>
<li>Hard Recipes, defined as recipes with number of steps to make more than nine steps ('n_steps' > 9)</li>
</ul>

#### Significance Level: 0.05</li>


### **Methodology**

<ol>
<li>Calculate the observed test statistic as the absolute difference in means between the two groups of recipes.
<li>Simulate the null hypothesis by shuffling the group labels (≤9 steps and >9 steps) 1,000 times and recalculating the test statistic for each permutation.
<li>Compute the p-value as the proportion of permuted test statistics greater than or equal to the observed test statistic.
</ol>

### **Results & Visualization**

<ul>
<li>Observed Statistic: ~0.000787
<li>P-Value: ~0.87
</ul>

Attached below is a Plotly histogram with 50 bins, depicting the distribution of test statistics generated under the null hypothesis. The observed statistic computed above is marked by a red solid vertical line. As the observed statistic is relatively near to the center of the distribution, the high p-value is reflected. 

<iframe
    src="assets/HypothesisTest.html"
    width="800"
    height="600"
    frameborder="0"
></iframe>

### **Conclusion**

As shown by the calculations and visualization above, the p-value (~0.87) exceeds the significance level (0.05), which leads us to fail to reject the null hypothesis. This result suggests that the average rating of a recipe is not significantly influenced by the number of steps it takes to make it, or in other words, how "hard" or "easy" it is to make.

The lack of a significant disparity between permutated test statistics against the observed statistic suggests that variables like preparation duration, recipe intricacy, or individual taste inclinations might have a more pronounced influence on average ratings. This observation underscores the complex dynamics of user preferences, indicating that a broader range of recipe characteristics should be examined in future studies to more effectively discern the determinants of user ratings. Additionally, cultural background and nutritional value could also be considered as potential factors that may impact the perception and popularity of different recipes.


## **Framing a Prediction Problem**


### Prediction Problem Statement
This analysis aims to predict the total calories ('calories') of a recipe based on recipe characteristics, such as:

<ul>
<li> Number of preparation steps ('n_steps')
<li> Number of ingredients used ('n_ingredients')
<li> Protein content ('protein')
<li> Sodium content ('sodium')
<li> Saturated fat content ('saturated_fat')
<li> Sugar content ('sugar')
</ul>

Accurately predicting calories can help us understand how recipe complexity and nutritional content influence a recipe's calories. A recipe's complexity ('n_steps', 'n_ingredients') is a logical predictor for calories because a greater variety of ingredients, and more elaborate cooking methods may increase the caloric content of a dish. tts nutritional content ('protein', 'sodium', 'saturated_fat', 'sugar') are also logical predictors because they are direcly tied to a recipe's caloric value. This predictive insight can guide recipe creators in designing dishes that are healthier, and understand how combination of ingredients affect a recipe's overall caloric content.

### Type of Prediction Problem
This is a regression problem, as the response variable (calories) is continuous, ranging from 58 (min) to 1036 (max).

### Response Variable
The response variable is calories, the nutritional content of a given recipe. Extreme caloric values may influence perceptions of healthiness, and vice versa.

We chose this response variable because we wanted to explore factors that affected the overall caloric content of a recipe, and redicting calories allows us to explore the relationship between recipe attributes and overall caloric content. It captures overall "healthiness" of a recipe, which in turn affects appeal and popularity of a recipe. 

### Features Used for Prediction
The features selected for prediction include:

<ul>
<li> n_ingredients (Number of Ingredients): Represents the complexity of the recipe in terms of ingredients. Complex recipes require more ingredients, and simpler recipes require less ingredients. Appeals to cooks who seek simplicity and convenience.
<li> n_steps (Number of Preparation Steps): Represents the complexity of the recipe in terms of effort required to complete a recipe. Complex recipes require more preparation steps, and simpler recipes require less. 
<li> 'protein': Represents the protein content of the recipe. High protein values indicate healthiness, while low protein indicate less healthy foods
<li> 'sodium': Represents the sodium content of the recipe. Excessive sodium amounts makes food more salty, which in turn makes it less healthy. 
<li> 'saturated_fat': Represents the "bad fat" content of the recipe.
<li> 'sugar': Represents the sugar content of the recipe. 
</ul>

All these features are known at the time of prediction since they are inherent attributes of the recipe.

### Evaluation Metric
Our evaluation metrics for our baseline model is the Mean Absolute Error (MAE) and R-Squared Coefficient of Determination (R^2):

MAE Justification
<ul>
<li>It provides an interpretable measure of the average prediction error in the same units as the target variable.
<li>MAE is more robust to outliers, which is essential given the skewed distribution of calories.
</ul>

R^2 Justification:
<ul>
<li>It provides an easy-to-interpret measure of a model's fit to the data. Any model that performs positively will have a high R^2 value, and vice versa.
<li>It provides an indication of how well unseen samples are likely to be predicted by the model, since the coefficient generalizes the relationship between our dependent and independent variables.
</ul>

### Visual Exploration of the Problem
<ol>
<li> Distribution of calories:
<iframe
  src="assets/CaloriesDistribution2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of calories is a right/positively skewed distribution, indicating that most recipes are clustered towards lower calories, with a fewer recipes with higher calories

<li> Scatter Plot of n_ingredients and calories
<iframe
  src="assets/n_ingredientsCaloriesScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatterplot displays no clear relationship between the two variables.

<li> Scatter Plot of n_steps and calories
<iframe
  src="assets/n_stepsCaloriesScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatterplot displays no clear relationship between the two variables, but it suggests that most of the recipes in the dataset clustered towards less than or equal to 40 steps. 

<li> Scatter Plot of protein and calories
<iframe
  src="assets/proteinCaloriesScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatterplot shows that recipes with higher protein levels often are higher in calories, showing a direct positive relationship.

<li> Scatter Plot of sodium and calories
<iframe
  src="assets/sodiumCaloriesScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatterplot displays no clear relationship between sodium and calories, where there is an even distribution of varying sodium content and all calorie counts. 

<li> Scatter Plot of saturated_fat and calories
<iframe
  src="assets/saturated_fatCaloriesScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatterplot shows that recipes with higher satured fat levels often are higher in calories, showing a direct positive relationship.

<li> Scatter Plot of sugar and calories
<iframe
  src="assets/sugarCaloriesScatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatterplot shows that recipes with higher sugar levels often are higher in calories, showing a direct positive relationship.

</ol>

### Importance of This Prediction Problem
Predicting calories caters to many audiences, including:
<ul>
<li>Health Enthusiasts: predicting calories can help health enthusiasts tailor their diet to specific health goals, such as weight loss, muscle gain, or medical conditions, ensuring the right type of food is consumed.
<li>Cooks: Accurate calorie predictions allow them to craft recipes that align with various dietary needs and preferences, making their dishes appealing to a broader audience looking for health-conscious options. 
<li>Recipes Insights: understanding the quantitative basis behind how a recipe's inner complexity and nutritional content on its overall caloric content are new and useful insights.
</ul>

## Baseline Model

The baseline model we used for this prediction problem is a Linear Regression model, because it is effective for predicting continuous numerical outcomes like 'calories', and relationship of features as significant predictors of the outcome can be compared clearly. To build the model, we first split the data into training and test sets. Then, we used a Pipeline, where preprocessing steps and data transformations were done so that the model is ready to be used.

### Model Features

#### Numerical Features (6):

<ul>
<li>n_steps: Number of preparation steps required for the recipe.
As a countable, numeric value indicating how many steps are involved in the recipe preparation, n_steps is a quantitative variable that can be used in calculations.

<li>n_ingredients: Number of ingredients used in the recipe.
Similar to n_steps, this feature counts the number of ingredients used, n_ingredients is a quantitative variable that can be used in calculations.

<li>protein: Protein content (as a percentage of the daily value).
The protein content is measured as a percentage of daily value, making it a quantitative variable that reflects the nutritional content of the recipe in terms of protein.

<li>sodium: Sodium content (as a percentage of the daily value).
The sodium content is quantified as a percentage of the daily value, making it a quantitative variable of the sodium intake provided by the recipe.

<li>saturated_fat: Saturated fat content (as a percentage of the daily value).
The saturated fat content is quantified as a percentage of the daily value, making it a quantitative variable of the harmful fat intake provided by the recipe.

<li>sugar: Sugar content (as a percentage of the daily value).
The sugar content is quantified as a percentage of the daily value, making it a quantitative variable of the sugar intake provided by the recipe.
</ul>

These numerical features were left as is since standardizing features (i.e. using StandardScaler) in a regression model does not change predictions, but only changes coefficients. Hence, it has no significant use in this specific scenario since we are using a LinearRegression model.

#### Categorical Features (1):

<ul>
<li>ingredients: Comma-separated tags describing varying individual ingredients of a certain recipe (e.g., 'white sugar', 'brown sugar', 'salt', and more).
</ul>
Ingredients in a recipe are considered nominal variables as each ingredient is treated as a distinct category without any inherent order or ranking. This allows for the classification of recipes based on the presence or absence of specific ingredients, useful for filtering or identifying recipes based on dietary preferences or restrictions. 
<br></br>
The 'ingredients' feature was processed using OneHotEncoder, which creates binary columns for each unique ingredient. This encoding ensures that categorical information is represented in a numerical format usable by the Linear Regression model.

### Model Performance:
**Metrics**
<ul>
<li>R-squared: 0.7624704207215018
<li>Mean Absolute Error (Baseline Model): 76.76761605484424
</ul>

This result indicates that the model’s predictions deviate, on average, by approximately 0.5 points from the true recipe ratings. Considering the target variable (average_rating) ranges from 1 to 5, this represents a reasonably good baseline performance.

**Coefficients**
<ul>
<li>n_steps: 0.2255, this coefficient suggests that, all else being equal, each additional step in a recipe is associated with an increase of about 0.2255 calories in the dish. The effect is positive but relatively small, indicating that the complexity or length of the recipe as measured by the number of steps has a slight impact on the caloric content.
<li>n_ingredients: 4.2917, this coefficient indicates a more substantial impact, with each additional ingredient added to a recipe increasing its caloric content by approximately 4.2917 calories. This suggests that recipes with more ingredients tend to be higher in calories, which could be due to the cumulative addition of caloric content from each ingredient.
<li>protein: 3.4293, for each unit increase in protein (likely measured in grams), the calorie count of the recipe increases by about 3.4293 calories. This relationship highlights protein as a significant contributor to the caloric total, which aligns with the fact that proteins have a known caloric value per gram.
<li>sodium: 0.1105, each unit increase in sodium is associated with a slight increase in calories, by about 0.1105 calories.
<li>saturated_fat: 3.0411, each unit increase in saturated fat increases the calorie count by approximately 3.0411 calories.
<li>sugar: 0.9368, each unit increase in sugar increases the calorie count by approximately 0.9368 calories.
<li>ingredients: 0.0000, seemingly showing that no particular ingredient significantly stands out in affecting the calorie prediction on its own.
</ul>

Overall, the model suggests that the number of ingredients, protein, and saturated fat are significant predictors of the calorie content in recipes.

### Model's Strengths
<ol>
<li>Reproducibility and Scalability:
Linear regression, particularly when combined with a well-structured pipeline, can be computationally efficient, consistent, and scalable to larger datasets. This efficiency makes it suitable for reproducing the model using different features rapidly.
<li>Handling of Categorical Data:
By using one-hot encoding in the pipeline, the model can effectively handle categorical data, transforming it into a format suitable for linear regression. This expands the model's applicability to datasets that include nominal variables without imposing an ordinal relationship among categories.
<li>Interpretability:
Linear regression models remain relatively simple and interpretable compared to more complex models, as how input variables affect the predicted outcome is clearly represented by the coefficients.
</ol>

### Model's Weaknesses
<ol>
<li>Prone to Outliers:
Outliers can disproportionately influence the model’s coefficients, leading to less accurate predictions as relationships between dependent and independent variables can be distorted.
<li>Linear Assumption:
As a model, Linear Regression assumes that there is a linear relationship between the dependent and independent variables. This can lead to poor model performance if the true relationship is non-linear, which can be the case for variables like 'n_steps' and 'n_calories'
</ol>

## Final Model

### Added Features

To improve the model's predictions, we decided to add the following features from the perspective of the data-generating process:
<ol>
<li> 'complexity': 'n_steps' + 'n_ingredients'
<ul>
<li> This new feature is an interaction term combining the number of preparation steps and the number of ingredients in a recipe. As both features encapsulate a form of complexity of recipes, combining them as a new feature encompasses complexity better as it can account for more recipe complexity instances, such as one many ingredients, and many preparation steps.
<li> In relation to the target variable, new scopes of measuring recipe complexity may also influence calories, since more ingredients and steps can logically add to caloric content. So, this feature allows the model to account for these new and unique combinations.
</ul>
<li> 'bad_nutrition': 'sugar' + 'saturated_fat' + 'sodium'
<ul>
<li>By aggregating sugar, saturated fat, and sodium into a single metric, the model can more effectively capture the cumulative negative impact these components may have on health, which is reflected in caloric content.
<li>By reducing the number of individual predictors, the model can improve on generalization, as there is a high chance that individual effects of sugar, fat, and sodium are correlated based on ingredients used for recipes in our dataset, making capturing broader impact better than isolating their individual contributions.
</ul>
</ol>

### Modeling Algorithm

For our final model, the Random Forest Regressor was used. A Random Forest Regressor is defined as an ensemble learning method that uses multiple decision trees during training and outputs the average prediction of the individual trees to make better predictions in comparison to a single decision tree.

It was chosen as the final model as the final model considering its strengths and our baseline model's weaknesses:
<ul>
<li>Since our input features contain both numerical and categorical data, Random Forest Regressor is known for its ability to handle both numerical and categorical data.
<li>In comparison to our Linear Regression baseline model's nature to only capture linear relationships. Random Forest Regressor has the ability to capture non-linear relationships between features as well, possibly making better predictions.
<li>In a Random Forest Regressor model, there is opportunity for impactful use of hyperparameters and optimization, which is not present in our Linear Regression baseline model. 
</ul>

### Hyperparameters Tuning:

For our final model, we used three hyperparameters:
<ul>
<li>max_depth, defined as taximum depth of each tree.
<li>min_samples_split, defined as the minimum number of samples required to split an internal node.
<li>n_estimators, defined as tumber of trees in the forest.
</ul>

To tune our hyperparameters, we employed GridSearchCV with 3-fold cross-validations to identify the optimal set among various combinations of hyperparameters.

As a result, the best hyperparameters we identified were:
<ul>
<li>max_depth: 20
<li>min_samples_split: 10
<li>n_estimators: 200
</ul>
These hyperparameters strike a balance between model complexity and generalizability, ensuring good performance on unseen data.

Performance Comparison
The final model demonstrated a measurable improvement over the baseline model in both of our evaluation metrics:

Baseline Model Performance:
Mean Absolute Error (MAE): 76.7585
Correlation Coefficient (R^2): 0.7634

Final Model Performance:
Mean Absolute Error (MAE): 67.0454
Correlation Coefficient (R^2): 0.8076

### Conclusion
This improvement in MAE and R^2 indicates that the final model's predictions are more accurate than the baseline model's, indicating less average error and better correlation between dependent and indepdendent variables. This improvement can be clearly attributed to multiple things. First, the addition of new features 'complexity' and 'bad_nutrition' was able to help the final model better represent recipe complexity and capture broader impact of nutritional content on caloric content of a recipe. Additionally, the use of hyperparameters in a Random Forest Regressor model that weren't possible for Linear Regresion and its optimization significantly improved the model's accuracy.

The bar chart below compares the performance of the baseline and final models, showcasing the reduction in MAE and R^2:

<iframe
  src="assets/MAEPerformance.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/R2Performance.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Fairness Analysis
Clearly state your choice of Group X and Group Y, your evaluation metric, your null and alternative hypotheses, your choice of test statistic and significance level, the resulting 
p
-value, and your conclusion.

Optional: Embed a visualization related to your permutation test in your website.

### Overview
To ensure that the model performs equitably across different groups, a fairness analysis is important for a predictive model. In this project, I analyzed the "fairness" of the final model by comparing its performance between two groups of recipes separated based on the number of ingredients in them:
<ul>
<li>Simple Recipes: Recipes with ≤ 9 ingredients.
<li>Complex Recipes: Recipes with > 9 ingredients.
</ul>

### Hypotheses

<ul>
<li>Null Hypothesis (H₀): The model’s performance (measured by MAE) does not differ significantly between Simple Recipes and Complex Recipes.
<li>Alternative Hypothesis (Hₐ): The model’s performance differs significantly between the two groups.
</ul>

The Mean Absolute Error (MAE) was used to measure the model’s performance for each group because it provides an interpretable measure of average prediction error.

### Test Statistic
The absolute difference in MAE between the two groups (Simple Recipes & Complex Recipes)

### Significance Level
A significance level of α = 0.05 was used.

### Results
<ul>
<li>Observed MAE Difference: ~0.0031
<li>P-Value: 0.0
</ul>

As the permutation test's p-value of 0 is less than the chosen significance level of 0.05, we reject the null hypothesis. Hence,  the observed difference in MAE is statistically significant, and it indicates that the model's performance does differ between the two groups recipes, performing slightly better for one group of recipes. Hence, there is potential unfairness in the model between Simple and Complex Recipes, suggesting areas for improvement. 

The histogram below illustrates the distribution of permutation test statistics under the null hypothesis, with the observed difference marked by the red dashed line.

<iframe
  src="assets/FairnessAnalysis.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

