# Week 6: Deploying to Streamlit Community Cloud

Hey everyone, welcome to week 6! Last week we built a Streamlit dashboard on top of our DuckDB database. This week we'll deploy it so anyone can access it from a browser, no local setup required. We'll use Streamlit Community Cloud, which is free and connects directly to your GitHub repo.

---

## Part 1: Create a Separate Deployment Repo

Your existing repo contains training scripts, raw data, model files, and a bunch of other things that have nothing to do with running the dashboard. We want our deployment repo to be clean and only contain only the files Streamlit actually needs. This also keeps your deployment history tidy and avoids accidentally exposing unrelated work.

Create a new public GitHub repository called something like `nba-draft-dashboard`. Then, locally, create a new folder and initialize it:

```bash
mkdir nba-draft-dashboard
cd nba-draft-dashboard
git init
git remote add origin https://github.com/yourusername/nba-draft-dashboard.git
```

Copy only the following files into this new folder from your existing project:

- `dashboard.py`
- `curr_season.duckdb`
- `requirements.txt` (covered below)

### Why commit the .duckdb file?

When Streamlit Cloud clones your repo to run your app, it only has access to what's in the repo — there's no local filesystem to pull from. Since our database is a single self-contained file, the simplest solution is to commit it directly. This works well here because the data is static.

### Creating requirements.txt

Streamlit Cloud needs to know which packages to install. Create a `requirements.txt` in the new folder:

```
streamlit
duckdb
pandas
```

You don't need to pin versions unless you run into compatibility issues. Once all three files are in place, commit and push:

```bash
git add .
git commit -m "initial deployment"
git branch -M main
git push -u origin main
```

---

## Part 2: Check Your Database Path

Your `dashboard.py` currently connects to the database like this:

```python
duckdb.connect(database='curr_season.duckdb', read_only=True)
```

On Streamlit Cloud, apps run from the repo root, so this relative path works without any changes. Keep everything flat at the repo root and you're fine.

---

## Part 3: Deploy on Streamlit Community Cloud

1. Go to [share.streamlit.io](https://share.streamlit.io) and sign in with GitHub.
2. Click **"Create app"** in the top right.
3. Select **"Deploy a public app from GitHub"**.
4. Fill in the fields:
   - **Repository:** your repo (e.g. `yourusername/nba-draft-predictor`)
   - **Branch:** `main`
   - **Main file path:** `dashboard.py`
5. Click **Deploy**.

Streamlit will clone your repo, install your dependencies, and launch the app. This usually takes a minute or two. Once it's live, you'll receive a public URL you can share with anyone.

---

## Part 4: Verify the Deployment

Once the app is live, run through the following checks:

- Search for a player you know is in the database and confirm their stats and draft probability display correctly.
- Try a search that returns no results and confirm the warning message appears.


---


**Conclusion:** Congrats on finishing the NBA Draft Prediction Project! I hope the weekly content has been educational and that you've been able to learn at least a little. And if you have any feedback (good or bad) whatsoever, please do not hesitate to share!
