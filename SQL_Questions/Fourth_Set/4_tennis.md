# Part 4: Tennis Data

*Intermediate - Advanced level SQL*

---

## Setup

We'll be using tennis data from [here](https://archive.ics.uci.edu/ml/datasets/Tennis+Major+Tournament+Match+Statistics).

Navigate to your preferred working directory, and download the data.

```bash
curl -L -o tennis.zip http://archive.ics.uci.edu/ml/machine-learning-databases/00300/Tennis-Major-Tournaments-Match-Statistics.zip
unzip tennis.zip -d tennis
```

Make sure you have Postgres installed and initialized

```bash
brew install postgresql
brew services start postgres
```

Install SQLAlchemy if you haven't already

```
conda install -c anaconda sqlalchemy
```

Start Postgres in your terminal with the command `psql`. Then, create a `tennis` database using the `CREATE DATABASE` command.

```
psql

<you_user_name>=# CREATE DATABASE TENNIS;
CREATE DATABASE
<you_user_name>=# \q
```

Pick a table from the *tennis* folder, and upload it to the database using SQLAlchemy and Pandas.

```python
from sqlalchemy import create_engine
import pandas as pd


engine = create_engine('postgresql://<your_user_name>:localhost@localhost:5432/tennis')

aus_men = pd.read_csv('./tennis/AusOpen-men-2013.csv')

# I'm choosing to name this table "aus_men"
aus_men.to_sql('aus_men', engine, index=False)
```

*Note: In the place of `<your_user_name>` you should have your computer user name ...*

Check that you can access the table

```python
query = 'SELECT * FROM aus_men;'
df = pd.read_sql(query, engine)

df.head()
```

Do the same for the other CSV files in the *tennis* directory.

---

## The challenges!

This challenge uses only SQL queries. Please submit answers in a markdown file.

Create initial table

```sqlite
query = """
WITH all_tourneys2 AS( 

SELECT *, 'm' AS gender, 'aus' AS tournament 
    FROM aus_men
    UNION ALL
SELECT *,'w' AS gender, 'aus' AS tournament 
    FROM aus_women
    UNION ALL
SELECT *, 'm' AS gender, 'french' AS tournament  
    FROM french_men
    UNION ALL
SELECT *,'w' AS gender, 'french' AS tournament
    FROM french_women   
    UNION ALL
SELECT * , 'm' AS gender, 'us' AS tournament
    FROM usopen_men   
    UNION ALL
SELECT *, 'w' AS gender, 'us' AS tournament 
    FROM usopen_women  
    UNION ALL
SELECT *, 'm' AS gender, 'wim' AS tournament 
    FROM wim_men
    UNION ALL
SELECT *, 'w' AS gender, 'wim' AS tournament 
    FROM wim_women)

SELECT *
FROM all_tourneys2
  """
df.to_sql('all_tourneys2', engine)
```

1. Using the same tennis data, find the number of matches played by
   each player in each tournament. (Remember that a player can be
   present as both player1 or player2).

   Rafeal Nadal had the most matches
   
   ```sqlite
   query = """
   SELECT player, tournament,  count(*) Total
   FROM 
   (
     SELECT tournament, "Player1" AS player
     FROM all_tourneys2
     UNION ALL
     SELECT tournament, "Player2"
     FROM all_tourneys2
   ) d
   GROUP BY player, tournament
   ORDER BY player;
   """
   ```
   
   
   
2. Who has played the most matches total in all of US Open, AUST Open, 
   French Open? Answer this both for men and women.

   Rafael Nadal and Agnieszka Radwanska played the most matches.

   ```
   query = """
   SELECT gender,player, count(*) Total
   FROM 
   (
     SELECT gender, "Player1" AS player
     FROM all_tourneys2
     WHERE tournament IN ('aus','us','french')
     UNION ALL
     SELECT gender, "Player2"
     FROM all_tourneys2
     WHERE tournament IN ('aus','us','french')
   ) d
   GROUP BY gender,player
   ORDER BY total DESC
   """
   ```

   

3. Who has the highest first serve percentage? (Just the maximum value
   in a single match.)

   S. Errani has the highest first serve percentage.

   ```sqlite
   query = """
   SELECT player, MAX(fsp)
   FROM 
   (
     SELECT "FSP.1" AS fsp, "Player1" AS player
     FROM all_tourneys2
     UNION ALL
     SELECT "FSP.2", "Player2"
     FROM all_tourneys2
   ) d
   GROUP BY player
   ORDER BY MAX(fsp) DESC
   """
   ```

   

4. What are the unforced error percentages of the top three players
   with the most wins? (Unforced error percentage is % of points lost
   due to unforced errors. In a match, you have fields for number of
   points won by each player, and number of unforced errors for each
   field.)


*Hint:* `SUM(double_faults)` sums the contents of an entire column. For each row, to add the field values from two columns, the syntax `SELECT name, double_faults + unforced_errors` can be used.

*Special bonus hint:* To be careful about handling possible ties, consider using [rank functions](http://www.sql-tutorial.ru/en/book_rank_dense_rank_functions.html).

Unforced error percentages calculated as (unforced errors + double faults) / (points lost) for all matches.

- Rafael Nadal	0.226436
- Novak Djokovic	0.242308
- Stanislas Wawrinka	0.258952

```sqlite
# needed a new query to combine all players and get wins/relevant stats
combine_query = """
WITH test AS( 
SELECT "Player1" as player, 'm' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe, "TPW.2" as points_lost
    FROM aus_men
    UNION ALL
SELECT "Player2" as player, 'm' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM aus_men
    UNION ALL
SELECT "Player1" as player, 'f' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe, "TPW.2" as points_lost
    FROM aus_women
    UNION ALL
SELECT "Player2" as player, 'f' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM aus_women
    UNION ALL
SELECT "Player1" as player, 'm' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe,"TPW.2" as points_lost
    FROM french_men
    UNION ALL
SELECT "Player2" as player, 'm' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM french_men
    UNION ALL
SELECT "Player1" as player, 'f' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe,"TPW.2" as points_lost
    FROM french_women
    UNION ALL
SELECT "Player2" as player, 'f' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM french_women
    UNION ALL
SELECT "Player1" as player, 'm' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe,"TPW.2" as points_lost
    FROM usopen_men
    UNION ALL
SELECT "Player2" as player, 'm' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM usopen_men
    UNION ALL
SELECT "Player 1" as player, 'f' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe,"TPW.2" as points_lost
    FROM usopen_women
    UNION ALL
SELECT "Player 2" as player, 'f' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM usopen_women
    UNION ALL
SELECT "Player1" as player, 'm' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe,"TPW.2" as points_lost
    FROM wim_men
    UNION ALL
SELECT "Player2" as player, 'm' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM wim_men
    UNION ALL
SELECT "Player1" as player, 'f' AS gender, 'aus' AS tournament, "Result" AS win, "FSP.1" AS fsp, "DBF.1" AS dbf, "UFE.1" AS ufe,"TPW.2" as points_lost
    FROM wim_women
    UNION ALL
SELECT "Player2" as player, 'f' AS gender, 'aus' AS tournament, 1-"Result" AS win, "FSP.2" AS fsp, "DBF.2" AS dbf, "UFE.2" AS ufe,"TPW.1" as points_lost
    FROM wim_women
)

SELECT *
FROM test
  """
  
df = pd_sql.read_sql(combine_query, connection)
df.to_sql('aggregated_final2', engine)


query = """
WITH top_wins AS (
SELECT "player", SUM("win") as total
FROM aggregated_final2
GROUP BY "player"
ORDER BY total DESC
LIMIT 3)

SELECT *
FROM top_wins
"""

df.to_sql('top_wins', engine)

query = """
SELECT player,SUM(dbf+ufe)/SUM(points_lost) as ufe_pct
FROM aggregated_final2
WHERE player IN (SELECT player from top_wins)
GROUP BY player
ORDER BY ufe_pct
"""
```

