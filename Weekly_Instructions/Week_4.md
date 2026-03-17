# Week 4: Current Season Predictions & Database Storage

Hey everyone, welcome to week 4! Last week we tuned our model and explored which features it leans on most. This week we'll put the model to work — running it against *current* NCAA player data to generate draft probability predictions, then storing everything in a local DuckDB database for use in next week's dashboard.

## Part 1: Saving Your Model

Before we can use the model anywhere outside of `tune.py`, we need to save it. Go back to `tune.py` and add the following line after you retrain the final model on the full training set:

```python
model.save_model('xgb_final.json')
print("Model saved to xgb_final.json")
```

XGBoost's `.save_model()` serializes the full model — trees, hyperparameters, and all — to a single file. This is what we'll load later to make predictions without retraining from scratch.

---

## Part 2: Predicting on Current Season Data

Create a new file `current_season.py`. You've been provided with the file `current_season.csv` in the data folder on github, which contains stats for players in the current NCAA season. It has the same format as your training data, just without a `Drafted` column (since the draft hasn't happened yet).

### Loading and Preprocessing

Load the current season CSV into a DataFrame. Then apply the same preprocessing steps you wrote in `preprocess.py`: fill missing `3P%` values, engineer your per-game stats, shot selection ratios, and encode `Pos` and `Class`. The feature columns must exactly match what the model was trained on — order included.

It's worth pausing here: why does this matter? If you trained on 38 features in a specific order and pass in a DataFrame with columns in a different order, XGBoost will silently make predictions using the wrong features. Reusing your `feature_cols` list from `preprocess.py` (or copying it exactly) is the safest approach.

### Loading the Model and Predicting

```python
import xgboost as xgb

model = xgb.XGBClassifier()
model.load_model('xgb_final.json')

pred_proba = model.predict_proba(curr[feature_cols])[:, 1]
curr['Draft_Prob'] = pred_proba
```

`predict_proba` returns a two-column array — probabilities for class 0 and class 1. We take `[:, 1]` to get the probability of being drafted.

**Your turn:** print the top 10 players by draft probability. Also filter for your favorite team and print their players' probabilities. Useful for a sanity check — do the results make intuitive sense?

---

## Part 3: Storing Results in DuckDB

Now that we have predictions, we want to persist them somewhere structured. We'll use **DuckDB**. It's a great fit here because it handles DataFrames natively and is fast for analytical queries.

### Quick Note: What is DuckDB?

Up until now, all of our data has lived in CSV files or pandas DataFrames — which works fine, but means every script has to reload and reprocess everything from scratch each time it runs. A database solves this by giving us a persistent, structured place to store data that any script (or eventually, a web dashboard) can query on demand.
DuckDB is a lightweight database that runs entirely inside your Python process — there's no server to set up or external software to install. Everything lives in a single .duckdb file on your computer. You write data into it once, and from that point on you can retrieve exactly what you need using SQL, a query language you may have seen before that lets you filter, sort, and aggregate data with just a few lines.
If you've never written SQL, don't worry — the queries we'll use are straightforward, and the syntax reads almost like plain English

Install it if you haven't yet:

```bash
pip install duckdb
```

Here's the basic pattern for writing a DataFrame to DuckDB:

```python
import duckdb

con = duckdb.connect(database='curr_season.duckdb')
con.execute("CREATE TABLE IF NOT EXISTS curr AS SELECT * FROM curr")
con.close()
```

A few things to note: `duckdb.connect()` creates the `.duckdb` file if it doesn't exist. `CREATE TABLE IF NOT EXISTS` makes sure it won't error if you run the script twice. DuckDB can reference a pandas DataFrame directly by name in SQL, which is what `SELECT * FROM curr` is.

**Your turn:** after writing the table, reopen the connection and run a query to verify the data was saved correctly. Try retrieving the top 10 players by `Draft_Prob` using SQL and printing the result. You can execute queries and fetch results like this:

```python
con = duckdb.connect(database='curr_season.duckdb')
result = con.execute("SELECT ...").fetchdf()
print(result)
con.close()
```

---

### Michigan State this season
Now try retrieving all of the players from Michigan State this season and their corresponding draft probabilities. Notice anything strange? 

Why do you think Jeremy Fears Jr, who leads the NCAA in assists, has such a low draft probability? Personally, I'm actually not sure, but I suspect it has to do with his awful three point shooting. Our model has likely learned that good shooting from outside the arc is a huge deal for NBA teams, especially given how important the 3 ball is in the modern league. 

Regardless, Jeremy Fears Jr serves as an important lesson when working with ML: Jeremy Fears Jr is an outlier in modern college basketball--an elite passer and point guard, who can't really shoot at all. XGBoost deals with outliers better than almost any other ML algorithm, but it has its limitations of course. We should be mindful of such limitations when using ML in the real world.

**Next Steps:** Push `current_season.py` and your updated `tune.py` to GitHub. Next week we'll build a simple web dashboard with Streamlit on top of this database to visualize draft predictions interactively.
