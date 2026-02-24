# Week 2: Preprocessing & Model Training

Hey everyone, welcome to week 2! Last week we explored our data visually and developed intuition for what separates drafted from undrafted players. This week, we'll clean and engineer features in `preprocess.py`, then train an XGBoost classifier in `train.py`.

## Part 1: Preprocessing

Create a new file `preprocess.py` in your main directory. This is where all of our data cleaning and feature engineering will live.

### Handling Missing Values

Before doing anything with the data, we need to handle `NaN` values. A quick check with `ncaa.isnull().sum()` will show you which columns have missing data. In our dataset, `3P%` is null whenever a player attempted zero three-pointers — dividing by zero produces `NaN`. The fix is straightforward: fill those nulls with 0.

```python
ncaa['3P%'] = ncaa['3P%'].fillna(0)
```

Check the other columns for missing values and handle them appropriately before moving on.

### Feature Engineering

Raw counting stats (total points, rebounds, etc.) are misleading because players play different numbers of games. A player with 400 points in 30 games is very different from one with 400 points in 15 games. We fix this by normalizing to a per-game basis:

```python
ncaa['PPG']  = ncaa['PTS'] / ncaa['GP']
ncaa['RPG']  = ncaa['TRB'] / ncaa['GP']
ncaa['APG']  = ncaa['AST'] / ncaa['GP']
```

**Your turn:** calculate per-game stats for steals, blocks, turnovers, and minutes played. Also calculate `GS_rate` (games started divided by games played), which captures how often a player is a starter.

We can also engineer **shot selection ratios** that tell us about a player's role and style:

- `3PAr` (three-point attempt rate): what fraction of a player's field goal attempts are threes
- `FTr` (free-throw attempt rate): how often a player gets to the line relative to field goal attempts

Try to implement these yourself.

### Encoding Categorical Variables

Machine learning models need numbers, not strings. We have two categorical columns — `POS` and `Class` — that need to be converted. We use a **dictionary map** to assign each category an integer:

```python
pos_map = {'G': 0, 'F': 1, 'C': 2}
ncaa['POS_enc'] = ncaa['POS'].map(pos_map)
```

**Your turn:** do the same for `Class` (FR, SO, JR, SR → 0, 1, 2, 3). Note: the column name has a trailing space, so you'll need to call `.str.strip()` on it first.

### Selecting Features and Target

Once your features are engineered, define your feature matrix `X` and target vector `y`:

```python
feature_cols = [
    # your full list of columns here
]

X = ncaa[feature_cols]
y = ncaa['Drafted']
```

Think carefully about which columns to include. Raw totals and per-game versions of the same stat are highly correlated — including both can cause redundancy. For now, include both and we'll revisit this in a later week when we discuss feature selection.

### Train/Test Split

#### Why Split at All?

The goal of a machine learning model isn't to memorize the data it was trained on — it's to generalize to new, unseen examples. If we trained and evaluated on the same data, the model could achieve near-perfect accuracy simply by overfitting: essentially memorizing which players were drafted rather than learning meaningful patterns. This would look great on paper but fail completely when applied to a new season of players.

The solution is to withhold a portion of the data before training and only use it for final evaluation. The model never sees the test set during training, so performance on it is an honest estimate of how the model would perform in the real world. Think of it like studying for an exam with a practice test — you learn from the practice problems, but your actual grade comes from questions you haven't seen before.

#### Stratification

A standard random split can accidentally skew the class balance between train and test. Since only ~8-10% of players in our dataset are drafted, a bad random split could end up with, say, 6% drafted players in the test set and 10% in the training set, making evaluation unreliable. Setting `stratify=y` tells sklearn to preserve the original draft rate in both splits:

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=67, stratify=y
)
```

The `random_state=42` seeds the random number generator so the split is reproducible — you and your teammates will get the same split every time.

Finally, save your train and test splits to `Data/train.csv` and `Data/test.csv` so that `train.py` can load them independently.

---

## Part 2: Training the Model

Create a new file `train.py`. Load your saved splits at the top:

```python
train_df = pd.read_csv('Data/train.csv')
test_df  = pd.read_csv('Data/test.csv')

X_train = train_df.drop(columns=['Drafted'])
y_train = train_df['Drafted']
X_test  = test_df.drop(columns=['Drafted'])
y_test  = test_df['Drafted']
```

### XGBoost

We'll use **XGBoost**, a gradient boosting algorithm that builds an ensemble of decision trees sequentially, where each tree tries to correct the errors of the previous one. It's one of the most widely used models in tabular data competitions and industry for good reason — it handles mixed feature types, is robust to outliers, and rarely needs heavy tuning to get strong baseline results.

One important consideration: our target is heavily imbalanced. Only a small percentage of players get drafted, which means a naive model could achieve very high accuracy by just predicting "not drafted" for everyone. XGBoost has a `scale_pos_weight` parameter to address this — it upweights the minority class (drafted players) during training:

```python
scale_pos_weight = (y_train == 0).sum() / (y_train == 1).sum()
```

With that calculated, here's how to construct the classifier:

```python
model = XGBClassifier(
    n_estimators=300,
    max_depth=4,
    learning_rate=0.05,
    subsample=0.8,
    colsample_bytree=0.8,
    scale_pos_weight=scale_pos_weight,
    eval_metric='logloss',
    random_state=67,
)
```

**Your turn:** fit the model on the training data. If you're unsure of the method, check the [XGBoost documentation](https://xgboost.readthedocs.io/en/stable/python/python_api.html#xgboost.XGBClassifier).

### Evaluation

Accuracy alone is a misleading metric for imbalanced problems (see above). Instead, we look at a broader set of metrics that give a clearer picture of model performance. After generating predictions, here's how to calculate a few:

```python
y_pred = model.predict(X_test)
y_pred_prob = model.predict_proba(X_test)[:, 1]

print(f"Accuracy : {accuracy_score(y_test, y_pred):.4f}")
print(f"Precision: {precision_score(y_test, y_pred):.4f}")
print(f"Recall   : {recall_score(y_test, y_pred):.4f}")
```

**Your turn:** also compute the F1 score, ROC-AUC, and print the full `classification_report` and `confusion_matrix`. All of these are available in [`sklearn.metrics`](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.metrics) — the docs will show you exactly what to import and how to call each one.

---

**Next Steps:** Commit both `preprocess.py` and `train.py` to GitHub. Next week we'll look at hyperparameter tuning and cross-validation to squeeze more performance out of this model.
