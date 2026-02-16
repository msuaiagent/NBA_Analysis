# Week 1: Getting Started

Hey everyone, welcome to week 1 of the NBA Draft Prediction model!

This week, we'll cover data manipulation skills in Pandas, as well as some basic exploratory data analysis before we begin training our model next week. 

## Part 1: Setting up your development environment

### Creating a new Git Repository

Begin by creating a new git repository on GitHub titled whatever you want but making sure it's informative (something like "NBA_Draft_Prediction_Model). Add a README as well, we can worry about filling it out later on. 

Then, like as described in the Week 0 instructions, clone the repository to your local device.

### Creating a .venv

Open the repository you just cloned in VSCode and initialize a virtual environment. Open a new terminal window to ensure your virtual environment is active, you should see something like `.venv` at the the start of each new line. In VSCode, your .venv should activate automatically, but in case it doesn't, type `source .venv/bin/activate` on MacOS/Linux or `.venv\Scripts\Activate.ps1` on Windows into your terminal. 

With your virtual environment activated, its time to install all of the packages you will need for this project. In the Github page for this project, you will find a file titled `requirements.txt`. Download this file and upload it to the directory you are currently working in. Now run `pip install -r requirements.txt` in your terminal. This command will install all of the packages listed in the requirements file (by the way `-r` is what is known as a flag variable, a way to tell the `pip install` command that `requirements.txt` is a file with packages that need to be installed). 

### adding a .gitignore.

If you recall from the Week 0 instructions, it is important to have a gitignore to make sure git is only tracking the files that you care about (as to not overload the git tracking system). 

Now that you have your development environment configured, its time to actually explore our data. Copy this [gitignore template](https://github.com/github/gitignore/blob/main/Python.gitignore) add add it to a `.gitignore` file in your main directory. Then commit your changes, and push to your remote git repository. You should see both the `requirements.txt` file and the `.gitignore` in the remote repo. 


## Part 2: Data and Jupyter Notebooks

Cool, now its time to actually take a look at our NBA draft prospect data. On the Github repo with the instructions for this project, go to the Data folder and grab the file titled `ncaa_data.csv`. This file contains all NCAA Men's Basketball player seasons from the 2015 to 2024 college basketball seasons, but only containing players from the Power Conferences(Big 10, SEC, Big 12, Big East, and Pac-12 Prior to dissolution) plus a few notable Mid-major programs. I've excluded international draft prospects to simplify the modeling and data collection work.

Upload this file to a new `Data` folder in your own directory. Then create a new file titled `EDA.ipynb`. Pay special attention to the `.ipynb` file extension, this type of file is called a jupyter notebook. Jupyter notebooks allow us to execute individual cells of python code to see the contents of our data as we programmatically modify it. Of course, this contrasts with a normal `.py` file, where the code runs all at once. This will be particularly useful as we conduct exploratory analysis on our data, the first step of any machine learning project.

Now, take some time to understand the various buttons at the top of the notebook and what they do. Research online how to create a new code cell, how to create a new markdown cell, and what it means to restart the notebook kernel.

Create a new code cell in your new notebook . Add a comment at the top of the cell with the info `#Imports`. From now on, we will designate this cell to be used to import all of the packages we will use in this notebook, which is a best practice when working in jupyter notebooks. We will first import the module `pandas` with the line `import pandas as pd`. Here `pd` is what is known as an alias: we will often import functions and classes from the `pandas` module, so in order to not type `pandas` over and over again, we will use `pd` instead, which references the `pandas` module for us. 

For context, `pandas` is a python library used specifically for data manipulation. With `pandas` we can create objects known as DataFrames, which are basically spreadsheets. We can then use code to observe, modify, filter, and make new or modified copies of our data.

Now open another new code cell. Type the line `ncaa = pd.read_csv('Data/ncaa_data.csv')`. Follow this up with another line `ncaa.head()`. The `read_csv` command pulls the data from the `ncaa_data.csv` file and loads it into a DataFrame object called `ncaa`. Then `ncaa.head()` displays the first few rows from this DataFrame so that we can inspect them. 

## Part 3: Exploratory Data Analysis

Now that you've loaded the data, let's explore it. EDA is all about understanding your dataset before building models.

### Basic DataFrame Operations

First, let's get an overview of our data:

```python
# See all columns and their data types
ncaa.info()
```

This shows you 32 columns with stats like GP (games played), PTS (points), AST (assists), TRB (total rebounds), and our target variable `Drafted` (1 = drafted, 0 = not drafted).

```python
# Statistical summary of numeric columns
ncaa.describe()
```

This gives you mean, standard deviation, min, max, and quartiles for each stat.

Now check to see if we have any missing values.

```python
# Check for missing values
ncaa.isnull().sum()
```


### Selecting and Filtering

Let's work with specific columns and filter players:

```python
# Select specific columns
scoring = ncaa[['Player', 'Team', 'PTS', 'GP']]
scoring.head()
```

```python
# Filter high scorers (more than 500 points)
high_scorers = ncaa[ncaa['PTS'] > 500]
print(f"High scorers: {len(high_scorers)}")

# Multiple conditions: Guards with 400+ points
# Use & for AND, | for OR, wrap each condition in parentheses
elite_guards = ncaa[(ncaa['POS'] == 'G') & (ncaa['PTS'] >= 400)]
elite_guards[['Player', 'PTS', 'POS']].head()
```

### Creating New Columns

Raw totals can be misleading since players play different numbers of games. Calculate per-game stats:

```python
# Points per game
ncaa['PPG'] = ncaa['PTS'] / ncaa['GP']

# Rebounds and assists per game
ncaa['RPG'] = ncaa['TRB'] / ncaa['GP']
ncaa['APG'] = ncaa['AST'] / ncaa['GP']

ncaa[['Player', 'PPG', 'RPG', 'APG']].head()
```

### Grouping and Aggregation

Group data to find patterns:

```python
# Average stats by position
ncaa.groupby('POS')[['PTS', 'AST', 'TRB']].mean()
```

This reveals that Guards average more assists, Centers get more rebounds - exactly what you'd expect!

```python
# How many players per position?
ncaa['POS'].value_counts()

# Draft rate by position
ncaa.groupby('POS')['Drafted'].mean()
```

```python
# Compare drafted vs non-drafted players
ncaa.groupby('Drafted')[['PPG', 'RPG', 'APG']].mean()
```

### Sorting

Find top performers:

```python
# Top 10 scorers
ncaa.nlargest(10, 'PTS')[['Player', 'Team', 'PTS', 'PPG']]

# Top 10 in PPG (minimum 20 games to qualify)
qualified = ncaa[ncaa['GP'] >= 20]
qualified.nlargest(10, 'PPG')[['Player', 'Team', 'PPG']]
```

### Your EDA Tasks

Complete these three tasks in your notebook:

- **Task 1**: What percentage of players in the dataset were drafted?
- **Task 2**: What Schools send the most players to the NBA Draft?
- **Task 3** : Which position has the highest draft rate?


## Part 4: Visualization with Matplotlib

Numbers tell one story, but visualizations make patterns instantly clear. Let's create some plots with Matplotlib. For context, Matplotlib is a python library that allows us to create graphs and plots to visually explore our data. 

### Setting Up Matplotlib

Add this to your imports cell at the top:

```python
import matplotlib.pyplot as plt
```

### Basic Plotting

Start with a simple histogram to see the distribution of points:

```python
# Distribution of total points
plt.figure(figsize=(10, 6))
plt.hist(ncaa['PTS'], bins=40, edgecolor='black', alpha=0.7)
plt.xlabel('Total Points')
plt.ylabel('Frequency')
plt.title('Distribution of Points Scored')
plt.show()
```

Then let's explore the data in terms of position.

```python
# Compare PPG by position
plt.figure(figsize=(10, 6))
ncaa.boxplot(column='PPG', by='POS')
plt.suptitle('')  # Remove default title
plt.xlabel('Position')
plt.ylabel('Points Per Game')
plt.title('PPG by Position')
plt.show()
```

Boxplots show you the median, quartiles, and outliers for each position.

### Scatter Plots for Relationships

```python
# Do more minutes lead to more points?
plt.figure(figsize=(10, 6))
plt.scatter(ncaa['MP'], ncaa['PTS'], alpha=0.5)
plt.xlabel('Total Minutes Played')
plt.ylabel('Total Points')
plt.title('Minutes vs Points')
plt.show()
```

You should see a clear positive relationship - more playing time equals more scoring opportunity.

```python
# Color by draft status
drafted = ncaa[ncaa['Drafted'] == 1]
not_drafted = ncaa[ncaa['Drafted'] == 0]

plt.figure(figsize=(10, 6))
plt.scatter(not_drafted['PPG'], not_drafted['RPG'], alpha=0.5, label='Not Drafted', s=30)
plt.scatter(drafted['PPG'], drafted['RPG'], alpha=0.7, label='Drafted', s=30, color='red')
plt.xlabel('Points Per Game')
plt.ylabel('Rebounds Per Game')
plt.title('PPG vs RPG by Draft Status')
plt.legend()
plt.show()
```

Notice how drafted players (red) tend to be closer to the upper right - higher scoring AND rebounding.

### Your Visualization Task

Create a figure with 4 subplots (2x2 grid) that shows:

1. **Histogram** of field goal percentage (FG%)
2. **Boxplot** comparing total assists (AST) across different classes (FR, SO, JR, SR)
3. **Scatter plot** of 3-point attempts (3PA) vs 3-point percentage (3P%) - color by draft status
4. **Bar chart** showing the count of drafted players by position

Hint: Use `plt.subplot(2, 2, 1)` for the first plot, `plt.subplot(2, 2, 2)` for the second, etc. Use `plt.tight_layout()` at the end to prevent overlap.

---

**Next Steps:**  Commit your `EDA.ipynb` to GitHub with a message like "Complete Week 1 EDA".

Next week, we'll use these insights to engineer features and build our first classification model!
