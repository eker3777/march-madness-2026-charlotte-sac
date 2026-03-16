# March Madness GAM - Club Share Data Documentation

This repository publishes club-share datasets used by the notebook workflows.

## Club-share outputs

- Men historical matchups: `data/processed/club_share/m_historical_matchups_2005_2025.csv`
- Women historical matchups: `data/processed/club_share/w_historical_matchups_2014_2025.csv`
- Men all-possible 2026 modeling matchups: `data/processed/club_share/m_modeling_matchups_2026_all_possible.csv`
- Women all-possible 2026 modeling matchups: `data/processed/club_share/w_modeling_matchups_2026_all_possible.csv`
- Men first-round 68-team candidate matchups: `data/processed/club_share/m_first_round_matchups_2026_68team_candidates.csv`
- Women first-round 68-team candidate matchups: `data/processed/club_share/w_first_round_matchups_2026_68team_candidates.csv`
- Men team aggregates: `data/processed/club_share/m_team_aggregates_2005_2026.csv`
- Women team aggregates: `data/processed/club_share/w_team_aggregates_2014_2026.csv`
- 2026 seeding helper workbook: `data/processed/club_share/2026_Seeding.xlsx`

## Scope and row grain

- Historical matchups files: one row per played NCAA tournament game.
- All-possible modeling matchup files: one row per seeded Team1-Team2 pair for 2026.
- First-round candidate files: one row per first-round candidate pairing, including expanded play-in alternate rows.
- Team aggregate files: one row per team-season.

## Orientation conventions

### Historical and all-possible matchup files

- Team orientation uses a modeling-first contract.
- `Team1` is the stronger bracket side by orientation rule (better seed, then tiebreakers).
- `Target_Team1Win` exists in historical matchups and is the matchup-model target.
- Most feature columns are `Diff_*` and are computed as `Team1 - Team2`.
- `Diff_SeedNum` is intentionally flipped to preserve seed-advantage semantics in modeling.

### First-round candidate files

- These files are for first-round visualization and simulation prep.
- Row orientation is explicit:
- `Favorite*` columns represent the lower numeric seed side.
- `Underdog*` columns represent the higher numeric seed side.
- `HasPlayInPath`, `FavoriteIsAlternate`, and `UnderdogIsAlternate` identify play-in expansions.
- `MatchupKey` is the canonical ordered TeamID pair key.
- `InModelingFile` and `FavoriteMatchesModelTeam1` indicate whether and how each row maps to the all-possible matchup table.
- Columns prefixed with `Model_` are copied from the mapped all-possible matchup row to keep the first-round file self-sufficient.

## Metric families

The same advanced metric bases are used across outputs.

- Team aggregate files store team-level metrics directly (for example `Final_Massey`, `sos_adj_net_rating`, `three_factors_composite`).
- Matchup files store the difference form (`Diff_<Metric>`).
- First-round candidate files include both first-round row context and mapped modeling features (`Model_Diff_<Metric>`).

## Team aggregate seed fields (2026)

The team aggregate files include seed metadata needed for downstream bracket logic:

- `Seed`: seed code string when assigned.
- `SeedNum`: seed number used in downstream logic.
- `SeedNumBase`: base numeric seed before play-in handling.
- `SeedRegion`: bracket region code.
- `SeedPlayInSuffix`: play-in suffix marker where present.
- `SeedPlayInFlag`: indicator for play-in seed slot.

These fields are populated for both primary and alternate teams in 11/16 play-in slots.

## How notebooks use these files

- Data prep notebook writes and refreshes all club-share exports.
- Intro EDA notebooks primarily read team aggregates.
- Advanced demo notebook reads:
- historical matchups for model training,
- all-possible 2026 matchups for probability lookup,
- first-round candidate file for first-round visuals and bracket candidate wiring.

## Quick modeling guidance

- Train matchup models on historical matchup files (`Target_Team1Win`).
- Score or simulate broad 2026 possibilities with all-possible matchup files.
- Build first-round specific charts/tables from first-round candidate files.
- Use team aggregate files for season-level analysis and feature inspection.
