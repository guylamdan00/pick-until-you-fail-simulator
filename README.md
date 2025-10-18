# PUYF Simulation: Pick Until You Fail

This project simulates a game mechanic called **Pick Until You Fail (PUYF)**. It models player progression through a series of levels, where each "pick" may either result in a prize or a fail. The simulation accounts for difficulty progression, fail cooldowns, gem costs, and balance management.

---

## 📌 Simulation Overview

Players start with a randomly assigned gems balance and attempt to progress through a set of levels. Each pick has probabilistic outcomes based on level and difficulty. Players who fail will receive adjusted rewards and may continue based on affordability.

---

## ⚙️ Key Components

### 1. Configuration Data
Loaded from Google Sheets:
- `main config`: Level reward weights and probabilities.
- `cooldown settings`: Fail factors and difficulty tiers.
- `gems slope`: Reference for gem conversion rates.
- `resource valuation`: Converts in-game rewards to gem equivalents.
- `cost config`: Cost multiplier per level.

### 2. Player Initialization
- 10,000 simulated players.
- Gems balance assigned by percentile (from 0 to 3500+).
- Difficulty starts at 0.

### 3. Simulation Loop
For each level:
- Generate random picks.
- Match picks against weighted probabilities to determine outcome.
- Adjust difficulty on fail or success.
- Record outcomes, rewards, fail states, and cumulative cost.

### 4. Post-Simulation Analysis
Data is aggregated to produce insights including:
- Gems cost per fail (`fails_agg`)
- Cumulative cost by player
- Affordability tracking (`can_afford`)
- Player distribution by fail count and level
- Cumulative probability of first fail
- Cost as % of balance
- Cost to complete by percentiles

---

## 📈 Outputs & Visualizations

The script generates several charts:
- **Player Distribution by Fails per Level**
- **Final Fail Count Distribution**
- **Level of First Fail Distribution**
- **Cumulative First-Fail Heatmap**
- **Player Difficulty Heatmap**
- **Cost as % of Gems Balance**
- **Affordability Rate by Fails**
- **Completion Cost by Percentile**

---

## 🧪 Tools & Technologies

- Python 3.x
- `pandas`, `numpy`, `matplotlib`, `seaborn`
- `duckdb` for fast in-memory SQL queries
- `gspread` for Google Sheets integration
- Google Colab compatible

---

## 🚀 How to Run

1. **Authorize Google Sheets Access**  
   The simulation connects to a Google Sheet by ID using `gspread`.

2. **Run in Google Colab or Local Notebook**  
   Simply copy the script into a Colab notebook and execute sequentially.

3. **Optional**: Export aggregated data for downstream dashboards or Excel analysis.

---

## 🧠 Insights You Can Generate

- What % of users can afford to keep playing after X fails?
- How much do different gem brackets spend?
- Which levels cause the highest fail dropoff?
- What’s the real cost distribution for completion?

---

## 📂 Folder / Sheet Requirements

Ensure your spreadsheet contains the following worksheets:
- `main config`
- `cooldown settings`
- `gems slope`
- `resource valuation`
- `cost config`
- `segmentation`
- `rewards cost multipliers`

Each sheet must have proper headers in the first row.

---

## ✍️ Developer Notes

This simulation is designed to support monetization tuning, progression balancing, and reward system evaluation for games using "pick until fail" mechanics.

---
