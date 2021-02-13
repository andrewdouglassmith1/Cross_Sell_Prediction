# Part 3: Soccer Data

*Introductory - Intermediate level SQL*

---

## Setup

Download the [SQLite database](https://www.kaggle.com/hugomathien/soccer/download). *Note: You may be asked to log in, or "continue and download".* Unpack the ZIP file into your working directory (i.e., wherever you'd like to complete this challenge set). There should be a *database.sqlite* file.

As with Part II, you can check the schema:

```python
import pandas as pd
import sqlite3

conn = sqlite3.connect('database.sqlite')

query = "SELECT * FROM sqlite_master"

df_schema = pd.read_sql_query(query, conn)

df_schema.tbl_name.unique()
```

---

Please complete this exercise using sqlite3 (the soccer data, above) and your Jupyter notebook.

1. Which team scored the most points when playing at home?  

   Real Madrid with 505 goals.

   ```sqlite
   home_goals = """
   SELECT team_long_name, SUM(home_team_goal)
   FROM Match
       LEFT JOIN team
       ON team.team_api_id =  Match.home_team_api_id
   GROUP BY home_team_api_id
   ORDER BY SUM(home_team_goal) DESC
   LIMIT 5
   """
   ```

2. Did this team also score the most points when playing away?  

   No FC Barcelona scored the most points when playing away.

   ```sqlite
   away_goals = """
   SELECT team_long_name, SUM(away_team_goal)
   FROM Match
       LEFT JOIN team
       ON team.team_api_id =  Match.away_team_api_id
   GROUP BY away_team_api_id
   ORDER BY SUM(away_team_goal) DESC
   LIMIT 5
   """
   ```

3. How many matches resulted in a tie?

   There were 6,596 tie games

   ```sqlite
   tie_games = """
   SELECT COUNT(id)
   FROM Match
   WHERE away_team_goal = home_team_goal
   """
   ```

4. 15 players have Smith for their last name and 18 players have smith anywhere in their name

   ```sqlite
   smith_query = """
   WITH FirstLast AS ( 
   SELECT substr(player_name, 1, pos) AS first_name,
          substr(player_name, pos+1) AS last_name
   FROM
     (SELECT *,
             instr(player_name,' ') AS pos
      FROM player))
   Select COUNT(last_name)
   FROM FirstLast
   WHERE last_name = "Smith"
   """
   ```

   ```sqlite
   smith_query2 = """
   Select COUNT(player_name)
   FROM player
   WHERE player_name LIKE '%smith%'
   """
   ```

5. What was the median tie score? Use the value determined in the previous question for the number of tie games. *Hint:* PostgreSQL does not have a median function. Instead, think about the steps required to calculate a median and use the [`WITH`](https://www.postgresql.org/docs/8.4/static/queries-with.html) command to store stepwise results as a table and then operate on these results. 

   The median goals scored was 1 goal

   ```sqlite
   tie_games = """
   WITH tie_games AS 
   (SELECT row_number() over (order by away_team_goal) as row_id, away_team_goal, home_team_goal,  (select count(1) from match) as ct 
   FROM Match
   WHERE away_team_goal = home_team_goal)
   SELECT AVG(away_team_goal) AS median
   FROM tie_games
   WHERE row_id between 3928 and 3929
   """
   ```

   

6. What percentage of players prefer their left or right foot? *Hint:* Calculate either the right or left foot, whichever is easier based on how you setup the problem.

   75% of players prefer their right foot.

```sqlite
foot_query = """
select preferred_foot,
       (count(*) * 1.0 / sum(count(*)) over ()) as ratio
from Player_Attributes
group by preferred_foot
"""
```

