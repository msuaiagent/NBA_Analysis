# Week 5: Building a Streamlit Dashboard

Hey everyone, welcome to week 5! Last week we ran our model against current NCAA season data and stored everything in a DuckDB database. This week we'll put it all together — building an interactive web dashboard in Streamlit that lets you search for any player and instantly see their stats and draft probability.

## What is Streamlit?

Streamlit is a Python library that lets you build web apps with nothing but Python. You write a Python script, and Streamlit handles all the rendering and interactivity for you. It's widely used for data science and ML demos, and it connects to our DuckDB database cleanly.

Install it if you haven't:

```bash
pip install streamlit
```

---

## Part 1: App Setup

Create a new file called `dashboard.py` in your project root. Start by importing your dependencies and loading the database:

```python
import streamlit as st
import duckdb
import pandas as pd

st.set_page_config(
    page_title="NBA Draft Predictor",
    page_icon="🏀",
    layout="wide",
)

@st.cache_resource
def get_connection():
    return duckdb.connect(database='curr_season.duckdb', read_only=True)

con = get_connection()
```

A few things to note: `st.set_page_config` controls the browser tab title and icon, and sets the layout to `wide` so we have more horizontal space for stats. `@st.cache_resource` is important — it tells Streamlit to create the database connection once and reuse it across re-renders, rather than reconnecting every time the user interacts with the app. Without it, your app would open a new database connection on every keystroke.

---

## Part 2: The Search Bar

Add a header and a search input below your setup code:

```python
st.title("NBA Draft Predictor")
st.caption("2026 Draft Class")

search = st.text_input(
    label="Search for a player",
    placeholder="e.g. Jeremy Fears Jr.",
)
```

`st.text_input` renders a text box and returns whatever the user has typed as a string. Each time the user types a character, Streamlit re-runs the entire script from top to bottom — that's just how Streamlit works. Since we cached our database connection, this re-run is fast.

---

## Part 3: Querying the Database

Below your search bar, add the query logic:

```python
if search:
    query = """
        SELECT *
        FROM curr
        WHERE Player LIKE ?
        ORDER BY Draft_Prob DESC
    """
    results = con.execute(query, [f"%{search}%"]).fetchdf()

    if results.empty:
        st.warning("No players found. Try a different name or partial name.")
    else:
        st.success(f"Found {len(results)} player(s)")
        selected_name = st.selectbox(
            "Select a player",
            options=results["Player"].tolist(),
        )
        player = results[results["Player"] == selected_name].iloc[0]
```

A few things worth understanding here:

- `Player LIKE ?` lets users search case with partial names. Typing "Coen" will still find "Coen Carr".
- The `?` placeholder is a *parameterized query* — this is the correct way to pass user input into SQL. Never use f-strings to build SQL queries directly, as this opens you up to SQL injection attacks.
- `st.selectbox` handles the case where a search matches multiple players — it lets the user pick which one they want to see.

---

## Part 4: Displaying Player Info

Once a player is selected, display their information. Add this inside the `else` block, after `player` is defined:

```python
        # Header row
        col1, col2 = st.columns([2, 1])
        with col1:
            st.subheader(player["Player"])
            st.write(f"**Team:** {player['Team']}  |  **Position:** {player['Pos']}  |  **Class:** {player['Class']}")
        with col2:
            draft_prob = float(player["Draft_Prob"])
            st.metric(
                label="Draft Probability",
                value=f"{draft_prob:.1%}",
            )

        st.divider()

        # Stats section
        st.subheader("Season Stats")
        stat_cols = st.columns(5)
        stats = {
            "PPG": "PPG",
            "APG": "APG",
            "RPG": "RPG",
            "SPG": "SPG",
            "BPG": "BPG",
        }

        for i, (label, col_name) in enumerate(stats.items()):
            with stat_cols[i]:
                if col_name in player:
                    st.metric(label=label, value=f"{float(player[col_name]):.1f}")

        st.divider()

        # Shooting section
        st.subheader("Shooting")
        shoot_cols = st.columns(3)
        shooting_stats = {
            "FG%": "FG%",
            "3P%": "3P%",
            "FT%": "FT%",
        }
        for i, (label, col_name) in enumerate(shooting_stats.items()):
            with shoot_cols[i]:
                if col_name in player and pd.notna(player[col_name]):
                    st.metric(label=label, value=f"{float(player[col_name]):.1%}")
                else:
                    st.metric(label=label, value="N/A")

        st.divider()

        # Full data expander
        with st.expander("View all stats"):
            st.dataframe(player.iloc[1:-3].to_frame().T, hide_index = True, use_container_width=True)
```

`st.columns` is how you create side-by-side layouts in Streamlit. `st.metric` is a pre-built component that displays a large number with a label below it — perfect for individual stats. The `st.expander` at the bottom lets users drill into the full raw row without cluttering the main view.

---

## Part 5: Running the App

To launch the dashboard, run:

```bash
streamlit run dashboard.py
```

Streamlit will open the app automatically in your browser at `http://localhost:8501`. Any time you save `dashboard.py`, the app will hot-reload with your changes.

---

## Part 6: Your turn 

Implement the following three features within your dashboard:
1. Add a new subsection with games played and games started information
2. Add a new section that displays a dataframe of all player information
3. Make player search case insensitive
