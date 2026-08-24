# `events.csv` column codebook

One row = one mid-season managerial change that was either a **sacking** or a **mutual consent** departure, matched to that club’s league fixtures.

Built by `scripts/05_code_and_merge_events.py`. Wikipedia (and Transfermarkt gap-fills) provide the change itself. football-data.co.uk match results provide form, table position, and later results.

The file currently has **665 rows**, seasons **2004/05–2024/25**, five leagues. It is **not** every managerial change: resignations, retirements, end-of-contract moves, preseason changes, and unmatched clubs are kept in `manager_changes_coded.csv` instead.

Missing values on `ppg_next_3` / `ppg_next_5` / `ppg_next_8` mean the club did not have that many league matches left after the new manager was appointed.

---

## Identity of the event

### `league`

Which of the five leagues this change belongs to: `premier_league`, `la_liga`, `serie_a`, `bundesliga`, or `ligue_1`. Taken from the Wikipedia season page (or Transfermarkt, if that season had no Wikipedia table).

### `season`

Season label such as `2008/09`. Wikipedia seasons start in 2004/05. Match results go back further, but this file only includes seasons with a managerial-changes table.

### `team`

Club name as written on Wikipedia (or Transfermarkt). Example: `Borussia Mönchengladbach`.

### `panel_club`

Same club, but using the football-data.co.uk name so it can join to match results. Example: `M'gladbach`. Mapped with aliases and a cleaned name key in `scripts/common.py` (`map_club_name`). Empty would mean no match; those rows are dropped from this file.

### `outgoing_manager`

The manager who left. Copied from the Wikipedia “outgoing manager” column, with footnotes/brackets stripped.

### `incoming_manager`

The manager who took over. Copied from the Wikipedia “incoming manager” / “replaced by” column. Caretaker labels in the name (for example `(caretaker)`) are left in the text; they are also used later to code successor type.

---

## How the manager left

### `manner`

Raw Wikipedia wording for how they left, for example `Sacked` or `Mutual consent`. Not recoded. This is what the next column is based on.

### `departure_type`

A cleaned version of `manner`, used for sample rules.

How it is coded (`_classify_manner`):

| Value | Meaning |
|---|---|
| `sacked` | Words like sacked, dismissed, fired, terminated, released, removed |
| `mutual_consent` | Mutual consent / mutual agreement wording |
| other `exclude_*` codes | Resignation, retirement, end of caretaker spell, signed elsewhere, preseason, same person in and out, etc. |

This file only keeps `sacked` (621 rows) and `mutual_consent` (44 rows). Same person listed as both outgoing and incoming is treated as `exclude_same_person` and never reaches this file.

---

## Dates

### `date_vacancy`

Date the old manager left. Parsed from Wikipedia’s vacancy date (`dayfirst`, mixed formats), then stored as `YYYY-MM-DD`.

### `date_appointment`

Date the new manager was appointed. Parsed the same way. For Transfermarkt gap-fills, appointment is set equal to the leaving date.

### `days_to_appointment`

Gap between those two dates:

`date_appointment − date_vacancy` in calendar days.

`0` means they were appointed the same day. Used as a matching covariate (very short gaps often force a caretaker).

---

## Who replaced them

### `successor_type_auto`

Treatment variable: what kind of appointment this was **at the moment of the vacancy**.

| Code | Meaning |
|---|---|
| `caretaker` | Temporary / interim / acting spell |
| `internal` | Permanent promotion from staff already at the club |
| `external` | Permanent hire from outside |

**Automatic rules** (in order):

1. **Caretaker** if Wikipedia/Transfermarkt flags the incoming name as caretaker/interim/acting/player-manager, the next Wikipedia row is “end of caretaker spell”, Transfermarkt marks the successor as a caretaker, or this person is replaced again within 90 days.
2. **Internal** if this incoming person was the previous incoming caretaker at the same club (promoted after a short spell).
3. **External** if this person left another club in the sample within 30 days before to 180 days after the appointment.
4. Otherwise **external**, but marked for review.

Manual reviews then overwrite the auto-code via `data/coding/successor_type_overrides.csv`. After review, current counts are roughly: external 420, caretaker 206, internal 39.

### `coding_confidence`

How sure the type above is:

- `high` — an automatic rule had a clear flag (caretaker wording, promotion from the previous spell, or arrived from another club in the sample)
- `review` — default external with no clear flag (these get exported to `successor_type_review.csv`)
- `manual` — a human coded it after checking a source, and it overwrote the auto value

### `coding_reason`

Short note on why the type was chosen. Automatic examples: `caretaker_flag_or_short_spell`, `promoted_from_prior_spell_at_club`, `arrived_from_other_club_in_sample`, `default_external_needs_review`. Manual rows have a one-line evidence summary (for example “assistant appointed interim head coach”).

### `source`

Where the change row came from. Currently all `wikipedia`. The pipeline can also insert `transfermarkt_gap` rows for in-season Transfermarkt changes in seasons Wikipedia has no table for.

### `wiki_url`

Link to the Wikipedia season page (or Transfermarkt URL for gap-fills). Used to check the original table.

---

## Situation at dismissal (from the match panel)

These are taken from the club’s **last league match on or before** `date_vacancy`. Match data is football-data.co.uk, exploded to one row per club per match in `scripts/02_build_match_panel.py`.

### `matchweek_at_dismissal`

How many league games that club had already played that season, including the last match before the vacancy. It is a running count of that club’s fixtures (`1, 2, 3, …`), not the official round number if fixtures were postponed.

### `league_position_at_dismissal`

Table rank after that last match. Clubs are ranked by cumulative points, then goal difference, then goals scored. Recalculated after every match date in that league-season.

### `trailing_6_ppg`

Average points per game over the last **up to 6** league matches before (and including) that last match. Win = 3, draw = 1, loss = 0. If the club had played fewer than 6 games, it uses whatever they had (`min_periods=1`). So early-season sackings are a shorter window.

### `trailing_6_gd`

Total goal difference (goals scored minus goals conceded) over the same last-up-to-6 matches. A sum, not an average. `-9` means they were nine goals worse than opponents over that window.

### `trailing_matches`

How many matches actually went into those two trailing stats. Usually `6`. Smaller numbers mean the sacking happened before the club had played six league games.

---

## What happened after the new manager arrived

The “clock” for post-appointment matches is `date_appointment` if it exists, otherwise `date_vacancy`. Only **league** matches after that date count.

### `n_remaining_after_appointment`

How many league matches that club still had that season after the new manager’s appointment date.

### `ppg_next_3`

Points per game over the **next 3** league matches after appointment. `sum of those 3 results / 3`. Blank if fewer than 3 matches remained.

### `ppg_next_5`

Same for the next **5** matches. This is the main outcome in the project plan (the short-run “bounce”). Blank if fewer than 5 matches remained.

### `ppg_next_8`

Same for the next **8** matches. Robustness window. Blank if fewer than 8 matches remained.

These windows do **not** check whether a second manager took over mid-window. That is why caretakers are kept as their own category: a long window can start measuring someone else.

---

## Sample flags

### `in_primary_sample`

`True` if this row is in the main analysis sample. All of these must hold:

1. `departure_type` is `sacked` (not mutual consent)
2. The change merged onto the match panel (`merge_ok`)
3. At least **5** league matches remained after appointment (so `ppg_next_5` exists)
4. The last match before the vacancy is **not** the club’s first or last match of the season (`mid_season`)

Currently **585** of 665 rows.

### `include_robustness_mutual`

`True` if this is a mid-season **mutual consent** change that matched a club and fell inside the season’s first–last match dates. These 44 rows are extra events for a robustness check, not the primary sample.

How a row gets into this file at all (before the primary-sample cut):

- `departure_type` is `sacked` **or** `mutual_consent`
- vacancy date is on/between that club’s first and last league match of the season
- not flagged as preseason
- club name matched to the panel

### `merge_ok`

`True` if the event found a matching club-season in the match panel and at least one league match on or before the vacancy date. All 665 rows here are `True`; failed merges never get the form/outcome columns filled in.

---

## How a row is built (short version)

1. Parse Wikipedia “Managerial changes” tables (`scripts/03_parse_wikipedia_changes.py`).
2. Optionally add Transfermarkt in-season rows for seasons Wikipedia missed.
3. Classify `departure_type` from `manner`; auto-code `successor_type_auto`; apply manual overrides.
4. Map `team` → `panel_club`.
5. Keep only in-season sackings and mutual consents that matched a club.
6. Attach the last match on/before the vacancy (form and table) and later matches (PPG windows).
7. Set `in_primary_sample` using the sacking + 5 remaining matches + mid-season rule.
