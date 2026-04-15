# Shot Quality Auditor

A Python notebook that loads StatsBomb open event data for a La Liga season, computes shot quality metrics, and produces structured tables and shot map visualizations for selected players and teams.

---

## What this project does

This notebook was built to explore whether xG shot maps can confirm or challenge assumptions about how players and teams actually play—without watching the matches.

It loads StatsBomb open data for a full season, filters shots by a defined scope of players and teams, computes goals-per-90 and goal-type breakdowns, and renders two types of output:

- **Player shot maps** — half-pitch, with dot opacity encoding xG and green outlines marking goals
- **Team shot maps** — full pitch, same visual encoding, laid out in a 2×2 grid

The interpretation section at the end of the notebook explains what the visuals revealed and where the data ran out of answers.

---

## Setup

1. Create and activate a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate  # On macOS/Linux
# .venv\Scripts\activate   # On Windows
```

2. Install dependencies using the provided `requirements.txt`

```bash
pip install -r requirements.txt
```

---

## Running the pipeline

### Step 1 — Load and save the competition data

Run the following in a separate cell or standalone script **before** opening the main notebook. This loads all matches and events for the competition and saves them as Parquet files.

```python
from statsbombpy import sb
import pandas as pd
import os

# --- Configuration ---
COMPETITION_ID = 11   # La Liga
SEASON_ID      = 27   # 2015/16

DATA_DIR = "data"
os.makedirs(DATA_DIR, exist_ok=True)

# Load all matches for the competition
matches = sb.matches(competition_id=COMPETITION_ID, season_id=SEASON_ID)

# Load events for every match and concatenate into one dataframe
all_events = []
for match_id in matches["match_id"]:
    events = sb.events(match_id=match_id)
    all_events.append(events)

events_df = pd.concat(all_events, ignore_index=True)

# Save to Parquet
matches.to_parquet(f"{DATA_DIR}/la_liga_1516_matches.parquet", index=False)
events_df.to_parquet(f"{DATA_DIR}/la_liga_1516_events.parquet", index=False)

print(f"Saved {len(matches)} matches and {len(events_df)} events.")
```

> **Note:** This will generate a `NoAuthWarning` from statsbombpy when accessing open data. This is expected and can be ignored.

### Step 2 — Run the notebook

Open `shot_quality_auditor.ipynb` and run all cells in order.

The notebook reads from `data/la_liga_1516_events.parquet`. If you saved your Parquet file under a different name or path, update the path in **Cell 1**.

### Changing teams or players

The notebook uses two hardcoded variables that control which data is analysed and visualised. If you load a different competition, update both:

```python
# Cell 3
TEAMS_IN_SCOPE = ['Barcelona', 'Real Madrid', 'Atlético Madrid', 'Levante UD']
PLAYERS_IN_SCOPE = ['Luis Alberto Suárez Díaz', 'Cristiano Ronaldo dos Santos Aveiro']
```

> **Important:** Player and team names must match the StatsBomb strings exactly, including accents and full legal names unless using nickname attribute. Run `all_shots['player'].unique()` and `all_shots['team'].unique()` on your loaded data to confirm the exact strings before editing these variables.

If you change the competition, also update the Parquet filename in Cell 1 and the minutes-played lookup dictionary in Cell 4 (`PLAYER_MINUTES`), which is currently hardcoded for Suárez and Ronaldo in La Liga 2015/16.

---

## Reading the notebook

The notebook is structured sequentially:

| Cell         | Purpose                                                      |
| ------------ | ------------------------------------------------------------ |
| 1            | Load events from Parquet, confirm required fields            |
| 2            | Validate shot outcomes and field names                       |
| 3            | Filter shots by scoped teams and players                     |
| 4            | Compute summary tables (goals, xG, goals per 90, goal types) |
| 5            | Render player shot maps                                      |
| 6            | Render team shot maps                                        |
| 7 (Markdown) | Written interpretation of outputs                            |

### Outputs

Both visualisations are saved to the `outputs/` directory:

- `outputs/striker_shot_maps.png` — Suárez and Ronaldo side by side
- `outputs/team_shot_maps.png` — 2×2 grid for all four teams

The summary tables are displayed inline in the notebook (Cells 4).

---

## Data

This project uses **StatsBomb open data**, available under the StatsBomb Open Data Licence.

→ [https://github.com/statsbomb/open-data](https://github.com/statsbomb/open-data)

Data is accessed via the `statsbombpy` Python library and is not committed to this repository. Run the loading script in Step 1 to regenerate the Parquet files locally.
