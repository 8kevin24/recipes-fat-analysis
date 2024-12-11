# Recipes and Ratings: Data Analysis and Predictive Model of Saturated Fat Percentage
By Kevin Zhu
## Project Overview
This is a data science project on investigating how different factors affect the saturated fat percentage for a recipe
The datasets used for this project can be find [here](https://dsc80.com/project3/recipes-and-ratings/food.com)

---
## Investigating Topic and Introduction
One of the core ways in which technology has changed society is its influence on the foods we eat. Processed food has taken over a large majority of American diets, and a major indicator of unhealthy food is its saturated fat content (proven in many studies to cause cardiovascular diseases), which processed foods are particularly high in. However, I want to know if foods with longer preparation times are unhealthier(often fried, grilled, roasted courses with heavy oil or seasoning) or if shorter, pre-processed or at least pre-prepared meals are more or less healthy. **In order to find out whether the length of preparation time influences saturated fat content, we will conduct an analysis of the data at hand**.

### Introduction to the Datasets in this Study
The first data set we are using contains the information of 83782 recipes from 2008 to 2018 on food.com, with 10 columns recording the following information:

|Column	                 |Description|
|---                     |---        |
|`'name'	`            |Recipe name|
|`'id'`	                 |Recipe ID|
|`'minutes'`	         |Minutes to prepare recipe|
|`'contributor_id'`	     |User ID who submitted this recipe|
|`'submitted'`	            | Date recipe was submitted|
|`'tags'`	              |Food.com tags for recipe|
|`'nutrition'`	          |Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein    (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”|
|`'n_steps'`	          |Number of steps in recipe|
|`'steps'`	              |Text for recipe steps, in order|
|`'description'`	     | User-provided description|

The other data set we used contains people's opinions to recipes on food.com, which is consists of 731,927 total reviews. It consists of 5 different columns as listed below:

|Column|Description|
|---|---|
|`'user_id'`	|User ID|
|`'recipe_id'`	|Recipe ID|
|`'date'`	|Date of interaction|
|`'rating'`	|Rating given|
|`'review'`	|Review text|

I mainly focused on the `nutrition` column in the first dataset, which contains the the saturated fat content as a daily percentage value (DPV) in addition to other information about the recipe like calories, carbohydrates, sugar, and more. I seperated them into different columns and assigned each category their respective portion data but we will mainly use `sat_fat (#)`
---
## Cleaning Data and Exploratory Data Analysis
### Data Cleaning
Here are some highlights of the data cleaning process:

1. **Left Merging the recipe and review data sets into one dataframe**: This is for the purpose of matching comments with corresponding recipes to make comparison easier.
2. **Fill the ratings of 0 with N/A**: Ratings on [food.com](food.com) cannot have rating 0. 0s in the data are most likely autofills by a faulty program. 
3. **Convert columns into correct types**: This allows later dataframe maniuplations to be much smoother. For example, the `date` and `submitted` columns are converted into DateTime type, `minutes` to float type, `n_steps` and `n_ingredients` into integer type.
4. **Extract reviewed year information from `nutrition` column**: unpacked this column into 6 seperate columns, calories for the `cal` column, saturated fat in the `sat_fat` column, and so on.
5. **discretized minutes taken to prepare**: This allowed for later steps to be far easier, as oftentimes I was comparing time to prepare with saturated fat content when making graphs and conducting hypothesis tests. By turning time, a continous numerical variable into a discrete one by using percentiles, this made the visualization and testing process much smoother. I split it into 10 percentiles, so granularity is still largely perserved in the column   `time_percentile`.

After data cleaning, the combined DataFrame contains these relevant columns (only showing the first 5 rows for illustration, the actual combined DataFrame has 234429 records):

| sat_fat | minutes | time_percentile | cal   | submitted   |
|---------|---------|-----------------|-------|-------------|
| 19.0    | 40      | 5               | 138.4 | 2008-10-27  |
| 51.0    | 45      | 5               | 595.1 | 2011-04-11  |
| 36.0    | 40      | 5               | 194.8 | 2008-05-30  |
| 36.0    | 40      | 5               | 194.8 | 2008-05-30  |
| 36.0    | 40      | 5               | 194.8 | 2008-05-30  |


Note: the dataframe has more than just these columns, but these 5 are the most relevant as of now

### Univariate Analysis
Time for some exploratory data analysis now that we are done with cleaning!
#### Distribution of Saturated Fat
Here is a histogram on the distribution of the the years when recipes are commented in our data set. 

It has a left-skewed shape, with most recipes containing saturated fat contents in the 20-30% DPV range. This makes sense because we eat 3 meals a day, so our daily percentage of any nutritional content should be in that range. However, there are definitely outliers, with some very fatty meals lying in the right tail at the 60% mark.

<iframe src="assets/fig2.html" width=800 height=600 frameBorder=0></iframe>

### Bivariate Analysis


<iframe src="assets/fig1.html" width=800 height=600 frameBorder=0></iframe>

To truly investigate the relationship between time to cook and saturated fat content, we simply just plot them out against each other by first grouping by `time_percentile` and then plotting it against each group's average saturated fat content. The graph has a moderate pattern of left skewness, meaning that it might be possible that higher times to prepare means higher saturated fat content. This makes sense since many long recipes involve processes like frying, grilling, or roasting that would involve heavy amounts of oil, butter, and other fatty ingredients.


### Interesting Aggregates
As a side quest, I also wondered if people preferred foods with higher saturated fat content. It's common to hear people say that unhealthy foods taste the best, but is this really true? (I love my fried chicken but my favorite, favorite food is steamed fish, very healthy)

To do so, I generated a pivot table with ratings as the index, showing us the average minutes taken to prepare and the average saturated fat content for recipes of that rating. 

| Rating | Minutes   | Saturated Fat (sat_fat) |
|--------|-----------|-------------------------|
| 1.0    | 99.672474 | 46.679094               |
| 2.0    | 98.021537 | 42.882179               |
| 3.0    | 87.497630 | 40.088120               |
| 4.0    | 91.585038 | 36.432680               |
| 5.0    | 106.923878| 39.226773               |

Turns out, it's entirely True! 5 star rating recipes have an average saturated fat content nearly 7 percent lower than 1 star ratings.
---
## Assessment of Missingness
### NMAR Analysis
One of the columns in our dataset with missing values that is possibly NMAR is the `review` column. This column's missingness is most likely entire dependent on itself, since people who really like a recipe (5 stars) would feel compelled to give a review, and people who really hate a recipe (1 stars) would feel compelle to give a review, but if a recipe is "mid" to someone, they are not very likely to take the time out of their day to log into food.com and write a sentence or two explaining just how mediocre the recipe was. It can happen, just not very likely.

If we had another column directly related to how someone felt about a recipe, similar to `review`, like "satisfaction" or "linger time" (which we do not have in our dataset), THEN review would be MAR based on that column. But there aren't, so it is completley Not Missing At Random.

### Missingness Dependency
The previous pivot table we had about ratings and people's love for unhealthy food had me thinking-since people dislike healthy food so much, could that be a reason to leave `rating` entirely blank? They dislike the recipe so much they don't even want to leave a review?

I constructed permutation tests to determine the relationship. I'm testing the dependency between the missingness of `rating` with two columns: `minutes` and `calories (#)`.

#### 1. Rating and Minutes (MCAR)
Null Hypothesis: The missingness of rating does not depend on minutes

Alternative Hypothesis: The missingness of rating depend on minutes

I created a new column indicating the missingness status of `rating`, and shuffled this column for permutation. Since `minutes` is numeric, I used the absolute difference in means of `minutes` when `rating` is and is not missing as our test statistics. If in our permutations we got results that were mostly not as extreme as the observed difference, then `rating` is MAR on `minutes`, otherwise not.

Below are distributions of minutes with and without rating. From the histogram below, we notice that the two distributions are very similar.

<iframe src="assets/fig2-1.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/fig2-2.html" width=800 height=600 frameBorder=0></iframe>

Empirical distribution of our test statistics in 30 permutations, the red line indicates the observed test statistics.

<iframe src="assets/fig4.html" width=800 height=600 frameBorder=0></iframe>

From the graph above and out permutation test, we get a p-value that's greater than our significance level of 5% at 0.067, so we fail to reject the null hypothesis.

**Therefore, we conclude that it is highly possible that the missingness of `rating` *<u>does not</u>* depend on `minutes` column.**

#### 2. Rating and Calories (MAR)

Null Hypothesis: The missingness of rating does not depend on calories (#)

Alternative Hypothesis: The missingness of rating depend on calories (#)

I repeated this process with the column `calories` and created a new column indicating the missingness of `rating`, and shuffled this column for permutation. Likewise, because `calories` is a numerical variable, I used the absolute mean difference of `calories` when `rating` is and is not missing as the test statistic

The distribution of calories when rating is missing is slightly greater than the distribution of calories when rating is not missing.

<iframe src="assets/fig2-3.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/fig2-4.html" width=800 height=600 frameBorder=0></iframe>


Empirical distribution of our test statistics in 30 permutations, the red line indicates the observed test statistics.

<iframe src="assets/fig3.html" width=800 height=600 frameBorder=0></iframe>

From this graph and our test, we get a p-value of 0.0. For those unfamiliar with mathematics, that's significantly less than our significance level of 0.05, so we reject the null hypothesis.

**Hence, we conclude that the missingness of `rating` *<u>depends</u>* on the `calories` column.**

---
## Hypothesis Testing
### Permutation Test
In this case, we are investigating whether the average saturated fat content for recipes in the "short" and "long" time categories (as defined by the `time_long_short` column) is different. Based on the initial analysis, we observed a potential difference in saturated fat content between recipes in the two categories. To further validate this observation, we need to conduct a permutation test to assess whether the observed difference is statistically significant, or if it might just be due to random chance.

**Null Hypothesis**: The average saturated fat content for recipes in the "short" and "long" time categories (as defined by the `time_long_short` column) are the same. Any observed difference is due to random chance.

**Alternative Hypothesis**: The average saturated fat content for recipes in the "long" time category is more than the saturated fat content for recipes in the "short" time category.

**Test Statistics**: For this test, we will use the difference in the mean saturated fat content between the "short" and "long" time categories as our test statistic. The null hypothesis assumes that the observed difference is due to random variation, and our permutation test will help us assess if this observed difference is statistically significant.

**Significance Level**: We will use a significance level of 5% to ensure that we have sufficient evidence to reject the null hypothesis if the test result is statistically significant.

The plot below shows the empirical distribution of our test statistics in 1000 permutations, with the red line indicating the observed test statistic.

<iframe src="assets/fig5.html" width=800 height=600 frameBorder=0></iframe>

From the figure above, we can see that the observed difference in saturated fat content between the two time categories is unlikely to be due to random chance, as the observed test statistic is wayyy to the right of the null distribution. This is further supported by our p-value of 0, which is less than the significance level of 0.05. Hence, we **reject the null hypothesis**.

## Conclusion:
Based on the results of the permutation test, we conclude that the observed data provides strong evidence against the null hypothesis, indicating that the difference in saturated fat content between recipes in the "short" and "long" time categories is statistically significant. Therefore, it is most likely that recipes with higher times to prepare have higher saturated fat content percentages. Although we do not know the true reason for this or even if it's a causal relationship, some theories include
    -recipes with longer prep times might serve more than just 1 meal
    -recipes with longer prep times might serve more than 1 person
    -recipes with longer prep times involve more oil-heavy processes(frying grilling)

## Now we move onto our Prediction Model!
## Framing the Problem

My prediction problem is to predict the saturated fat content of a given recipe, and this is a regression problem since saturated fat content as a daily percentage value is a continuous numeric variable. I chose to predict saturated fat content because it is in direct relation with the previous parts of my analysis and also because I think it is the most useful thing to predict, other than perhaps calories. (Predicting something like rating is not very useful to the average person since taste is truly subjective, and predicting time is not very useful either since humans have a good natural intuition of how long a recipe will take. However, humans are very bad at intuitively guessing how healthy a food is, saturated fat content wise). I will be using R^2 to evaluate my model since it is more reader friendly than RMSE(as RMSE is relative between models while R^2 is always between 0 and 1, with 0 being awful and 1 being amazing).

## Baseline Model

My baseline model uses average rating, date of submission, and number of steps. Submission date is a categorical ordinal variable, and to transform this for prediction, I am using ordinal encoding to make the first year-month-day combination into integers(eg. January 1st 2008 is 1, January 1st 2009 is 365, so on and son). Effectively, this means "how many days since January 1st 2008 has passed to the submission date".(Note: I do realize that this approach does not account for leap years, but leap years' effect on our model is negliglent since we have about 4 thousand days worth of data). While I at first thought there was no need for feature engineering for n_steps and rating, it turns out that rating is MAR on calories (as we discovered earlier). In order to account for this, I could either conduct probabilistic imputation, which is very computation exhausting, or perform single imputation to transform our data into something usable for a model.

In the end, I decided with single imputation. It resulted in much, much faster model training times, and the performance tradeoff between probabilistic and single imputatation was negligible since our data had over 234,429 rows and there were were only about 15,000 missing - which is about 6% missing, so replacing the missing values with the mean of all ratings would only marginally affect our model performance.

The performance of my model was not ideal, with an R^2 of close to 0. However, since this is a baseline model, and only includes 3 features, it makes sense that even an advanced machine learning model cannot predict saturated fat content of a recipe given its submission time, number of steps and average rating. Expecting this model to predict with extreme accuracy given these features is like expecting someone to tell you if it's going to rain based on the menu at McDonald's - somewhat plausible, maybe Mickey D's likes to serve Filet-O-Fishes on rainy days, but not very realistic.

## Final Model

To improve upon my baseline model, I added two new features - calories and minutes. I chose calories because it is probably the most heavily correlated with saturated fat content(no justification for this, simply gut feeling) and minutes because the relationship between minutes and saturated fat content was something I heavily explored in the earlier part of this project. However, calories has an extremely wide range and also contains many outliers, so to counter this, I discretized it and turned it into quantiles and incorporated that into my pipeline. I binarized time because for the model to be useful to the average human, it has to be user-friendly, and most people, when given a recipe and asked to estimate the amount of time they will spend on it, they will probably say something like "over/under 30 minutes" compared to "22.456 minutes". After all, our model needs to be practical, so I binarized the "minutes" column.

I also ran into the issue of choosing which hyperparameters were best. I had to change my model from being based on a LinearRegression model to a Lasso model in my search of optimizing hyperparameters, which led to better performance as well. I did some research on Lasso models, and essentially, it's a linear regression model that selectively chooses some weights to shrink to 0, improving model variance and generalizability. It chooses which weights to shrink based on an alpha value. If the alpha value is too large, the model underfits, and too small means it overfits. Eventaully, after splitting the training data into validation folds and conducting cross-validation, I found out that the best hyperparameter was the second smallest in my list-a sweet 0.001.

This final model improved much, much more compared to the original baseline model. This makes sense- the two features I added should correlate much more heavily with saturated fat content on paper- calories and time to prepare. Using the previous analogy, it's like asking someone if it's going to rain based on the clothing people in the streets are wearing-much more plausible and more likely to give you a correct answer. 

If I were to use the "Linkedin approach" to boasting of my model's improvement, I'd tell you it improved by with a increased R^2 of nearly 1000 percent. However, I will gladly humbly tell you the the undecorated version that it went from having an R^2 of about 0.02 to 0.2. That's still an incredibly significant jump, though, and definitely worth celebrating.

## Fairness Analysis
Model fairness is important, and so to make sure that no recipes are left behind, I needed to answer the question, "does my model perform worse for individuals in Group X than it does for individuals in Group Y?”.

So, I conducted a permutation test.

My two groups were simply the binarization of time to prepare I had used earlier. Essentially, if time to prepare was below average, versus if time to prepare was below or at average. My evaluation metric was R^2, for reasons meantioned earlier, so my test statistic is the difference of means between "short" recipes and "long" recipes(short-long). Therefore, my null and alternative hypotheses are:

**Null Hypothesis**: There is no significant difference in the performance of the model (e.g., R²) when predicting the saturated fat content for recipes with preparation times above/at average compared to those with preparation times below average. Any observed difference is due to random chance.

**Alternative Hypothesis**: The performance of the model is significantly worse for recipes with preparation times above/at average compared to those with preparation times below average. This indicates a potential bias or unfairness in the model's predictions based on preparation time.

For the significance level, I chose 0.05 since this is a commonly accepted statistic threshold for significance. Other signifiance levels are used for specific industries(eg. 0.01 for cancer treatments or spaceship construction) and recipes analysis is not exactly on the same level as NASA or Pfizer tests.

<iframe src="assets/fig6.html" width=800 height=600 frameBorder=0></iframe>

The permutation test resulted in a p-value of 0.019, falling below our threshold for signifiance and therefore leading to us to conclude that we should reject our null hypothesis. Essentially, our model does favor recipes with preparation times that are below average compared to those at/above average, and it is an unfair model in that sense.

