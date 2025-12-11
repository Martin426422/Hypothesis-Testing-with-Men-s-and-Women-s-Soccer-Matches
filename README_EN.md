
# Are More Goals Scored in Women's International Soccer Than Men's? — FIFA World Cup (2002–present)

## Overview
As a sports journalist, we investigate whether **women's international soccer matches** feature **more goals** than **men's**. To avoid confounding from qualifiers and older eras, we restrict the analysis to **official FIFA World Cup matches** (men's and women's) **from 2002-01-01 onward**.

We apply a **nonparametric hypothesis test** (Mann–Whitney U, right-tailed) at **α = 0.10**.

- **H0 (null):** The mean number of goals in women's matches equals men's.
- **H1 (alt, one-sided):** The mean number of goals in women's matches is **greater** than men's.

## Dataset
Two CSV files scraped from a reliable source (historical results since the 19th century), then **filtered** to World Cup finals only:
- `men_results.csv`
- `women_results.csv`

**Expected columns**: `date`, `tournament`, `home_score`, `away_score` (+ other metadata such as teams, city).

## Methodology
1. **Filter** matches where `tournament == 'FIFA World Cup'` and `date > '2002-01-01'`.
2. **Compute goals per match** as `home_score + away_score`.
3. **Compare distributions** of goals (women vs men) using **Mann–Whitney U** (no normality assumption; robust to unequal variances).
4. **Right-tailed** test (`alternative='greater'`) because H1 states women > men.
5. **Decision rule** at **α = 0.10**: reject H0 if *p* < 0.10.

## Code (minimal reproducible)
```python
import pandas as pd
from scipy.stats import mannwhitneyu

# Load datasets
men = pd.read_csv("men_results.csv")
women = pd.read_csv("women_results.csv")

# Parse dates
men["date"] = pd.to_datetime(men["date"]) 
women["date"] = pd.to_datetime(women["date"]) 

# Filter: World Cup finals since 2002-01-01
men_wc    = men[(men["date"] > "2002-01-01") & (men["tournament"] == "FIFA World Cup")]
women_wc  = women[(women["date"] > "2002-01-01") & (women["tournament"] == "FIFA World Cup")]

# Goals per match
men_goals    = men_wc["home_score"] + men_wc["away_score"]
women_goals  = women_wc["home_score"] + women_wc["away_score"]

# Mann–Whitney U (right-tailed)
stat, p_val = mannwhitneyu(x=women_goals, y=men_goals, alternative="greater")
result = "reject" if p_val < 0.10 else "fail to reject"
print({"p_val": p_val, "result": result})
```

## Example Output
```text
{"p_val": 0.005106609825443641, "result": "reject"}
```
Interpretation: At **α = 0.10**, we **reject H0** and conclude that women's World Cup matches from 2002 onward tend to have **more goals** than men's, **on average** (distributional shift consistent with higher central tendency).

## Assumptions & Checks
- Independence of match outcomes between groups.
- Same tournament family (World Cup finals only), comparable context.
- Consider **sensitivity** analyses:
  - Split by **cycle/year** (e.g., 2003–2011 vs 2012–present).
  - Remove **extra-time** matches or analyze goals in regulation vs including ET.
  - Explore **Poisson/negative binomial** modeling for count data.

## Visualization (optional)
```python
import matplotlib.pyplot as plt
import seaborn as sns

both = pd.concat([
    women_wc.assign(group="women", goals_scored=women_goals),
    men_wc.assign(group="men",     goals_scored=men_goals)
])

plt.figure(figsize=(8,4))
sns.boxplot(data=both, x="group", y="goals_scored")
plt.title("Goals per match — FIFA World Cup (2002–present)")
plt.show()
```

## How to Run
1. Place `men_results.csv` and `women_results.csv` in `data/`.
2. Install dependencies: `pip install pandas scipy seaborn matplotlib jupyter`.
3. Run notebook or Python script.

## Limitations
- World Cup finals only; **excludes qualifiers**, friendlies, continental cups.
- Changes in rules/format over time may affect goal counts.
- Mann–Whitney tests **medians/ranks**; for mean comparison, also consider **Welch’s t‑test** with normality checks or bootstrap CI.

## License
MIT License © 2025 Martin Karwacki
