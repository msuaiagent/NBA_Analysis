# Week 3: Hyperparameter Tuning & Model Interpretability

Hey everyone, welcome to week 3! Last week we trained a baseline XGBoost model and evaluated it on a held-out test set. This week we'll properly tune the model using a validation split, then dig into what our model has actually learned using feature importances

## Part 1: Hyperparameter Tuning

Right now, we are simply training with 80% of our data, and testing with 20%. That was fine for our initial model, but a more robust approach would involve tuning what are known as *hyperparameters*. Our XGBoost model learns how to best predict if a college basketball player gets drafted, but this process of learning itself is influenced by these hyperparameters, which we can experiment with for better outcomes. In other words, hyperparameters control *how our model learns from data*. 

### tune.py

We'll use a method called **`GridSearchCV`** from sklearn to handle hyperparameter selection for us. Basically, this tool will test out the model learned from the training data with various hyperparameter combinations. The hyperparameter combinations with the best performance will then be used for our final model.

This method is called GridSearch because it evaluates all possible combinations of hyperparameters from a predefined grid. The CV stands for Cross-Validation, a technique where our data is split into a predefined number of groups, a number which we will call *k*. *k-1* groups will be used to train our data, and the final remaining group will be used to test our data. The hyperparameters that do best on their specific testing group will be used in our final model.

We will be evaluating the following two hyperparameters:

n_estimators - as described last week, XGBoost works by building a series of *decision trees* that work together to predict an outcome. n_estimators controls how many of these trees we will use.

max_depth - max_depth controls how far down each individual tree allowed to go down. The deeper a given tree, the more granular its evaluation of the data, which might end up being a good or bad thing (that's why we're experimenting after all)

Create a new `tune.py` file. First load in the training data from earlier and calculate `scale_pos_weight` like last time. Then, paste in the following code: 

```python
from sklearn.model_selection import GridSearchCV
import numpy as np


param_grid = {
    'n_estimators': [200, 300, 400],
    'max_depth': [3, 4, 5],
}

search = GridSearchCV(
    estimator=XGBClassifier(
        scale_pos_weight=scale_pos_weight,
        colsample_bytree=0.8,
        eval_metric='logloss',
        random_state=67,
    ),
    param_grid=param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    verbose=1,
)

search.fit(X_train, y_train)

print("Best params:", search.best_params_)
print("Best val accuracy", search.best_score_)
```

**Your turn:** once the search completes, retrain a final model using `search.best_params_` on the full `X_train` and evaluate it on the test set. Print the same metrics you computed in `train.py` — accuracy, precision, recall, F1, ROC-AUC, classification report, and confusion matrix — and compare them to your Week 2 baseline. Did tuning help?

**One Last Point**: at the end of this project, the winner will be chosen based on how well their model performs on an evaluation dataset that I've kept separate. A great way to improve the performance of your model is to experiment with additional hyperparameters (which you can research) that I've chosen not to include in this code snippet. Keep that in mind...

---

## Part 2: Feature Importances

We know our model is capable of making predictions with reasonable accuracy, but why is our model making the decisions that it is? Which variables are the most important? Which variables are the least? We can answer these questions to some extent by examining our model's feature importances. Add the following code to the end of your `tune.py` file:

```python
import matplotlib.pyplot as plt

importances = model.get_booster().get_score(importance_type='weight')
importances = dict(sorted(importances.items(), key=lambda x: x[1], reverse=True))

plt.figure(figsize=(10, 6))
plt.barh(list(importances.keys())[:15], list(importances.values())[:15])
plt.xlabel('Weight')
plt.title('Top 15 Feature Importances')
plt.gca().invert_yaxis()
plt.tight_layout()
plt.show()
```

Do these results make sense? Which stats are the model leaning on most?

---

**Next Steps:** 
Be sure to push your new code to GitHub. Next week, we'll upload player data and model predictions to a local database to eventually use in our web dashboard.
