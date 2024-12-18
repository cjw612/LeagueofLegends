# League of Legends Game Outcome Prediction
![Game Map of League of Legends](assets/SR.jpg)
*Summoner's Rift, the primary map in League of Legends*

- ### Project Objective and Overview
  This project aims to predict which team would win a League of Legends game using in-game data representing the game state at the 15-minute mark. League of Legends is an online MOBA game with two sides, a blue side and a red side, with each side consisting of five players. This project leverages four distinct models and comparing the results yielded, which, after applying data preprocessing, feature transformation, and hyperparameter tuning, resulted in an accuracy of over 78% over all four models. In particular, given the vector of the in-game statistics $X_i$ of game $i$, this project aims to to predict the outcome $Y_i$ of game $i$, in which $Y_i$ is a binary class denoting the win or loss of the blue side in that particular game.

- ### Data Source
  The dataset used for this analysis is the **"League of Legends SoloQ matches at 15 minutes 2024" (match_data_v5.csv)** dataset, created by Karlo Rusovan and Daria Komic on [Kaggle](https://www.kaggle.com/datasets/karlorusovan/league-of-legends-soloq-matches-at-10-minutes-2024/data).

- ### Data Structure
  This dataset contains 24,224 rows, with each row representing one unique game, and 29 columns, with the first column being the matchID and the last column being the indicator of whether the blue side had won that game. The remaining columns represent in-game statistics of both teams, including gold earned, wards placed, turrets destroyed, etc.

  |matchID|blueTeamControlWardsPlaced|...|redTeamControlWardsPlaced|...|blueWin|
  |-------|--------------------------|---|-------------------------|---|-------|
  |EUW1_6882489515|2|...|34|...|1|
  |...|...|...|...|...|
  |EUW1_6881140491|6|...|1|...|1|

  *Sample snapshot of dataset*

- ### Data Cleaning and Preprocessing
  The dataset does not contain missing values. However, prior to analyzing the data, necessary feature transformation and outlier deletion were performed based on domain knowledge. In particular, the following tasks are performed:

  - #### Feature Transformation:
    Created features that represent the difference between a particular feature between teams. Note that all such features are constructed by subtracting the corresponding value of the red team from the blue team. 
    For instance, the feature $goldDiff$ is created by $blueTeamTotalGold - redTeamTotalGold$.
  - #### Variable Transformation:
    Transformed necessary features into categorical variables. In particular, features $blueWin$, $blueTeamFirstBlood$, and $redTeamFirstBlood$ are transformed into binary categorical variables due to their binary nature.
  - #### Outlier Removal:
    Removed outlier games identified by winning with a significant gold deficit at 15 minutes or losing with a significant gold lead at 15 minutes. 730 games satisfy this criteria, constituting around 3% of the total games. The data points that satisfy the following criteria are filtered out prior to data analysis:
    
$$
\left( \text{goldDiff} \geq 4000 \land \text{blueWin} = 0 \right) \lor \left( \text{goldDiff} \leq -4000 \land \text{blueWin} = 1 \right)
$$

- ### Exploratory Data Analysis
  EDA in this project is aimed to address the following questions:

  - What are the correlations between features?
  - What are the distributions of quantitative features?
  - What is the difference in the distribution of quantitative features across the two target classes?

- ### Data Analysis
  Four different models are deployed in this analysis to determine which model is optimal for this dataset:
  - #### Logistic Regression with L1 penalty
    Due to this problem being a binary classification problem, Logistic Regression is deployed while incorporating an L1 penalty in the model to perform feature selection, as it is expected that only a subset of features has a significant enough impact on the outcome of a game. The regularization strength coefficient $C$ is optimized with grid search by the package *GridSearchCV*.
  - #### Linear Discriminant Analysis (LDA)
    Based on the results of the EDA, we can observe that the features in both classes roughly follow a Gaussian distribution. In addition to the low variance in this model that may alleviate overfitting, Linear Discriminant Analysis is deployed as opposed to Quadratic Discriminant Analysis.
  - #### Random Forest with Bayesian hyperparameter optimization
    Random Forest is one of the state-of-the-art machine learning methods, relying on multiple decision trees while considering only a subset of features at each split to lower the variance of the model. As the size of this dataset does not create excessive computational overheads, Random Forest is implemented with hyperparameter Bayesian optimization with the package *hyperopt*. The package _hyperopt_ is selected due to its flexibility of various parameters, in addition to the multiple parameters that are required to be optimized. 
  - #### XGBoost with Bayesian hyperparameter optimization
    XGBoost is also one of the most efficient machine learning methods, which "boosts" a tree according to the results of each iteration to alter the weight of misclassified datapoints. XGBoost was also implemented with Bayesian hyperparameter optimization by the package *hyperopt*.

  In addition, K-fold Cross-Validation with $K = 5$ is also implemented for model selection to lower the variance of the results.

- ### Results
  The results of the four models are summarized in the following table:

  |Model|Accuracy|Precision|Recall|F1-Score|
  |-----|--------|---------|------|--------|
  |Logistic Regression|0.784456|0.783151|0.781385|0.782249|
  |LDA|0.784073|0.785103|0.776931|0.780966|
  |Random Forest|0.781987|0.786864|0.768155|0.777345|
  |XGBoost|0.780327|0.785842|0.765254|0.775327|

- ### Limitations
  - As current outlier detection is based on domain knowledge, incorporating a more sophisticated outlier detection algorithm may further improve the current classification results.
  - In practice, the impact of the gold difference at the 15-minute mark is not always deterministic of the outcome of the game. For example, for team compositions that are more late game-focused, a gold deficit at the 15-minute mark may have little to even no impact on the outcome of the game. Therefore, incorporating champion composition metadata of both teams may further increase prediction accuracy.
  - Incorporating player metadata (e.g., proficiency in champion, skill level) may also increase prediction accuracy, as the current dataset follows the assumption that all players are homogenous. 

- ### References

  [1] Redback93. "SRFull-1600p." Reddit, 20 July 2025, https://www.reddit.com/r/leagueoflegends/comments/3dx2dn/extremely_high_resolution_summoners_rift_image/ \
  [2] Karlo Rusovan and Daria Komic, League of Legends SoloQ matches at 15 minutes 2024 (Kaggle, 2024), https://www.kaggle.com/datasets/karlorusovan/league-of-legends-soloq-matches-at-10-minutes-2024/data

