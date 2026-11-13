# IMDB Movies Data Pipeline (Talend Open Studio)

This is a data pipeline I built in Talend Open Studio while learning the tool. It takes a messy movies dataset, finds the problems in it, cleans it up, and then produces a few summary tables that are ready for reporting.

## Why I built it

I wanted to practise building a real pipeline from start to finish, not just load a file and print it. The dataset I used looked simple at first, but once I profiled it I found it had a lot of quality problems. So the project became about finding those problems, fixing them in a repeatable way, and showing how much they changed the results.

## The data

The input file is `movies_data.csv`. It has 326 rows and 6 columns:

`movie_id, year_of_release, Genre, Director, IMDB_ratings, country`

Note: this looks like a practice dataset. Some IDs don't match the real films, so I didn't treat the results as real-world facts about these directors.

## What was wrong with the data

When I profiled the file in Talend I found three problems:

1. **Duplicates.** The 326 rows are really only 48 movies. Each movie appears between 4 and 12 times.
2. **Wrong ratings.** 15 of the movies have conflicting ratings between their copies. Usually one copy has a much lower score than the rest (for example 5.0 instead of 9.0). In total there were 17 bad rows.
3. **Unreliable country.** The same movie is tagged with different countries (USA, UK, IN, CN) in different rows, so this column can't be trusted.

## How the pipeline works

The pipeline is split into separate jobs, and one master job runs them in order.

| Job | What it does |
|---|---|
| `J00_Master` | Runs the other jobs one after another. If one fails, it stops. |
| `J01_Read_Raw` | Profiles the raw data. Counts unique movies vs duplicates and finds movies with conflicting ratings. |
| `J03_Clean_Ratings` | Works out the correct rating for each movie and splits the data into clean rows and rejected rows. |
| `J04_Analysis` | Builds the summary tables from the clean data. |

**How the correct rating is chosen:** for each movie, I take the rating that appears most often across its copies. For example, if a movie appears 6 times and 5 copies say 8.8 and 1 says 3.8, the correct rating is 8.8. Any row that doesn't match the correct rating goes to the rejects file. Then duplicates are removed so each movie appears once.

Main Talend components used: tFileInputDelimited, tUniqRow, tAggregateRow, tSortRow, tMap (joins and calculations), tFilterRow, tFileOutputDelimited and tRunJob.

## Outputs

All outputs are in the `talend_output` folder.

| File | Rows | What's in it |
|---|---|---|
| `movies_clean.csv` | 48 | One row per movie with the correct rating |
| `movies_rejects.csv` | 17 | The bad rows that were removed, with the correct rating next to them |
| `genre_summary.csv` | 8 | Number of movies and average rating per genre |
| `director_summary.csv` | 11 | Average rating per director (only directors with 2 or more films) |
| `decade_summary.csv` | 5 | Number of movies and average rating per decade |
| `before_after_directors.csv` | 22 | Each director's average on the raw data vs the clean data |

Row check: 309 good rows + 17 rejected rows = 326, so nothing is lost along the way.

## What I found

- **Genre:** Crime (9.1) and Drama (8.51) have the highest average ratings. Adventure is the lowest (7.86).
- **Director:** Francis Ford Coppola (9.13) and Frank Darabont (9.1) come out on top.
- **Decade:** The 1970s have the highest average (9.0) and the 2000s the lowest (8.01). The 1970s only have 3 movies though, so I wouldn't read too much into it.
- **Before vs after cleaning:** this was the most interesting part. The bad data made 10 of the 22 directors look worse than they are. The Wachowskis went from 7.95 on the raw data to 8.7 after cleaning, and Christopher Nolan went from 8.26 to 8.7.
- **Duplicates matter too, not just wrong values.** Chris Columbus had no wrong ratings at all, but his average still changed slightly (7.53 to 7.5). That's because one of his films was copied 12 times in the raw data and the other only 6 times, so the raw average was weighted towards one film.

## What's in this repo

```
IMDB_datapipeline_project/
├── movies_data.csv        the raw input file (326 rows)
├── talend_output/         the 6 output files the pipeline creates
├── process/               the Talend jobs (J00, J01, J04, and J03 inside MOVIES_PIPELINE/)
├── metadata/              the file definitions for movies_data and movies_clean
├── talend.project         the Talend project file
└── .settings/             Talend project settings
```

The whole folder is a Talend project export, so it can be imported straight into Talend Open Studio.

## How to run it

1. Open Talend Open Studio (I used version 8.0.1).
2. Go to **File > Import Items**, select this project folder (or the zip), tick everything and click Finish.
3. Update the file paths. The jobs point to folders on my computer (`C:/Users/sheld/...`), so you'll need to change:
   - the file path in the two metadata items (`movies_data` and `movies_clean`)
   - the file paths in the tFileOutputDelimited components in J03 and J04
4. Open `J00_Master` and press F6 to run the whole pipeline.

## Limitations and things I'd improve

- **File paths are hard-coded.** The next thing I'd do is move them into context variables so the folder only needs setting once.
- **Country isn't fixed.** Unlike ratings, there's no clear "correct" value to pick for country, so I left it as it is. The value in the clean file is just the first one that came through and shouldn't be relied on.
- **The clean file has two rating columns** (`IMDB_ratings` and `true_rating`). After cleaning they're always the same, so one of them could be dropped.
- The outputs are CSV files for now. The next step is to load them into Power BI for reporting, which is my next module.

## Tools

Talend Open Studio for Data Integration 8.0.1, CSV files, Excel for checking outputs.
