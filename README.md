# March Madness Club Share Guide (Intro EDA Focus)

This project is designed so students can start with the Intro EDA notebooks, understand the shared data tables, and then optionally move into advanced modeling.

## Start here: notebook flow

1. `notebooks/Intro_EDA_Demo.ipynb`
	 - Student-facing walkthrough with TODO prompts.
	 - Uses the club_share CSVs directly.
	 - Supports both men and women via `DATA_PREFIX`.

2. `notebooks/Intro_EDA_Answer_Key.ipynb`
	 - Completed instructor/reference version of the same flow.
	 - Keeps the same file contracts and challenge structure as Demo.

3. `notebooks/Club_Demo_Advanced_Modeling.ipynb` (optional next step)
	 - Adds richer visuals and simulation/modeling workflows.
	 - Still built on the same club_share exports so students can connect concepts.

---

## Club share files used by Intro EDA

Men
- `data/processed/club_share/m_historical_matchups_2005_2025.csv`
- `data/processed/club_share/m_team_aggregates_2005_2026.csv`
- `data/processed/club_share/m_modeling_matchups_2026_all_possible.csv`
- `data/processed/club_share/m_first_round_matchups_2026_68team_candidates.csv`

Women
- `data/processed/club_share/w_historical_matchups_2014_2025.csv`
- `data/processed/club_share/w_team_aggregates_2014_2026.csv`
- `data/processed/club_share/w_modeling_matchups_2026_all_possible.csv`
- `data/processed/club_share/w_first_round_matchups_2026_68team_candidates.csv`

Also present:
- `data/processed/club_share/2026_Seeding.xlsx` (seed source used for field construction and alternates)

---

## Orientation and modeling contract

- Team orientation in matchup files:
	- Team1 is the favorite orientation used for modeling.
	- `Target_Team1Win = 1` means Team1 won.
- Feature engineering convention:
	- Matchup files use `Diff_*` columns (Team1 minus Team2), except seed-gap fields where sign is defined to preserve interpretation.
	- Team aggregate files use team-level base fields without `Diff_` prefix.

This consistent contract is what lets Intro EDA and Advanced notebook examples share the same data safely.

---

## Data dictionary (cleaned)

### 1) historical_matchups (men/women)

Row grain: one played tournament game.

Core columns

| Column | Meaning |
|---|---|
| Season | Tournament season year |
| DayNum | Kaggle season day index |
| Team1ID, Team2ID | Kaggle team IDs in model orientation |
| Team1_TeamName, Team2_TeamName | Team names |
| Team1_Seed, Team2_Seed | Seed strings (examples: `X03`, `Z16a`) |
| Diff_SeedNum | Seed advantage signal in matchup orientation |
| Target_Team1Win | Binary target: 1 if Team1 wins, else 0 |
| Team1_Color, Team1_AlternateColor, Team1_Logo | Team1 branding metadata |
| Team2_Color, Team2_AlternateColor, Team2_Logo | Team2 branding metadata |

Feature columns
- Includes many `Diff_*` features spanning ratings, momentum, efficiency, possession profile, and derived composites.
- These columns are aligned with the same feature bases in `team_aggregates` and `modeling_matchups_2026_all_possible`.

---

### 2) team_aggregates (men/women)

Row grain: one team-season.

Core columns

| Column | Meaning |
|---|---|
| Season | Season year |
| TeamID | Kaggle team ID |
| TeamName | Team name |
| Seed | Seed string when available |
| TeamColor, TeamAlternateColor, TeamLogo | Branding metadata |
| SeedRegion, SeedPlayInSuffix | Seed-slot metadata used in bracket logic |

Feature columns
- Same metric bases used in matchup files, but as team-level values (no `Diff_` prefix).
- Includes ratings, efficiency factors, momentum features, and composite signals used downstream.

---

### 3) modeling_matchups_2026_all_possible (men/women)

Row grain: one possible 2026 seeded matchup (favorite orientation).

Core columns

| Column | Meaning |
|---|---|
| Season | 2026 |
| Team1ID, Team2ID | Team IDs in model orientation |
| Team1_TeamName, Team2_TeamName | Team names |
| Team1_Seed, Team2_Seed | Seed strings |
| Team1_Field68Role, Team2_Field68Role | Primary/alternate slot role |
| Team1_SeedRegion, Team2_SeedRegion | Region tags |

Feature columns
- Full `Diff_*` feature set aligned to historical matchups, ready for scoring and simulation workflows.

---

### 4) first_round_matchups_2026_68team_candidates (men/women)

Row grain: first-round candidate matchup rows from 68-team field construction (includes play-in alternates).

Core columns

| Column | Meaning |
|---|---|
| DataPrefix | `m` or `w` |
| Season | 2026 |
| Region | Bracket region |
| FavoriteSeed, UnderdogSeed | Numeric seeds |
| FavoriteSeedCode, UnderdogSeedCode | Seed-slot codes |
| FavoriteTeamID, UnderdogTeamID | Team IDs |
| FavoriteTeam, UnderdogTeam | Team names |
| Diff_Final_Massey | Favorite minus underdog final Massey |
| FavoriteIsAlternate, UnderdogIsAlternate | Play-in alternate flags |
| HasPlayInPath | True when matchup path involves play-in slot(s) |
| InModelingFile | True when orientation appears in all-possible modeling table |
| MatchupLabel, MatchupKey | Display and unique key fields |

Model-prefixed columns
- `Model_*` fields carry joined context from the all-possible matchup file for direct modeling handoff.

---

## Core metric definitions used across files

These appear as `Diff_<Metric>` in matchup files and `<Metric>` in team aggregates.

| Metric base | Definition |
|---|---|
| Final_Massey | End-of-season Massey rating |
| Adj_Massey | Seed-adjusted Massey residual |
| Adj_Seed | Rating-implied adjusted seed |
| Massey_Momentum | Late-season Massey momentum signal |
| Relative_Massey_Momentum | Scaled/relative Massey momentum |
| Final_Colley | End-of-season Colley rating |
| Colley_Momentum | Late-season Colley momentum signal |
| Relative_Colley_Momentum | Scaled/relative Colley momentum |
| TournamentPrestige_5Y_Decay | Decay-weighted prior tournament success |
| avg_off_rating | Offensive points per 100 possessions |
| avg_def_rating | Defensive points allowed per 100 possessions |
| avg_net_rating_game | `avg_off_rating - avg_def_rating` |
| adj_net_rating | Home/away adjusted net rating |
| sos_adj_net_rating | Schedule-adjusted net rating |
| avg_efg_pct | Effective FG percentage |
| avg_ts_pct | True shooting percentage |
| avg_three_par | 3PA/FGA rate |
| avg_three_pct | 3-point percentage |
| avg_three_score | Three-point scoring efficiency proxy |
| avg_ft_par | FTA/FGA rate |
| avg_ft_pct | Free-throw percentage |
| avg_ft_score | Free-throw scoring efficiency proxy |
| avg_tov_pct | Turnover percentage |
| avg_orb_pct, avg_drb_pct, avg_trb_pct | Rebounding rates |
| net_efg, net_tov, net_reb | Net four-factor style signals |
| net_ast_rate, net_stl_pct, net_blk_pct | Net playmaking/defense components |
| playmaking_defense_composite | Composite from assist/steal/block differentials |
| three_factors_composite | Composite from shooting/turnover/rebounding profile |
| pyth_win_pct | Pythagorean expected win rate proxy |

---

## Quick usage notes for Intro EDA

- Use `DATA_PREFIX = "m"` for men or `"w"` for women.
- Intro notebooks load:
	- one team aggregate file,
	- one all-possible 2026 matchup file,
	- one first-round 68-team candidate file for the challenge section.
- If the first-round candidate file is missing, run the export preprocessing workflow that writes club_share outputs first.

This keeps classroom workflows simple while still matching the data contracts used by the advanced notebook.
