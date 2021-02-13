# Part 2: Baseball Data

*Introductory - Intermediate level SQL*

---

## Setup

`cd` into the directory you'd like to use for this challenge. Then, download the Lahman SQL Lite dataset

```
curl -L -o lahman.sqlite https://github.com/WebucatorTraining/lahman-baseball-mysql/raw/master/lahmansbaseballdb.sqlite
```

*The `-L` follows redirects, and the `-o` uses the filename instead of outputting to the terminal.*

Make sure sqlite3 is installed

```
conda install -c anaconda sqlite
```

In your notebook, check out the schema

```python
import pandas as pd
import sqlite3

conn = sqlite3.connect('lahman.sqlite')

query = "SELECT * FROM sqlite_master;"

df_schema = pd.read_sql_query(query, conn)

df_schema.tbl_name.unique()
```

---

Please complete this exercise using SQL Lite (i.e., the Lahman baseball data, above) and your Jupyter notebook.

1. What was the total spent on salaries by each team, each year?

   ```sqlite
   query_salaries = """
   SELECT SUM(salary), TEAMID, yearID FROM salaries GROUP BY teamID,yearID ORDER BY yearID
   """
   ```

   

2. What is the first and last year played for each player? *Hint:* Create a new table from 'Fielding.csv'.

   ```sqlite
   fielding_query = """
   SELECT playerID, ID, MIN(yearID),MAX(yearID) FROM fielding GROUP BY playerID
   """
   ```

   

3. Who has played the most all star games?

   Hank Aaron, Willie Mays, and Stan Musual are tied for the most all-star appearances at 24 games. 

   ```sqlite
   allstar_query = """
   SELECT playerID, COUNT(yearID) FROM allstarfull GROUP BY PlayerID ORDER BY COUNT(yearID) DESC LIMIT 10
   """
   ```

   

4. Which school has generated the most distinct players? *Hint:* Create new table from 'CollegePlaying.csv'.

   Texas has generate the highest number of distinct players with 265.

   ```sqlite
   collegeplaying_query = """
   SELECT schoolID, COUNT(playerID) FROM collegeplaying GROUP BY schoolID ORDER BY COUNT(playerID) DESC LIMIT 5
   """
   ```

5. Which players have the longest career? Assume that the `debut` and `finalGame` columns comprise the start and end, respectively, of a player's career. *Hint:* Create a new table from 'Master.csv'. Also note that strings can be converted to dates using the [`DATE`](https://wiki.postgresql.org/wiki/Working_with_Dates_and_Times_in_PostgreSQL#WORKING_with_DATETIME.2C_DATE.2C_and_INTERVAL_VALUES) function and can then be subtracted from each other yielding their difference in days.

   Nick Altrock had the longest playing career

   ```sqlite
   query = """
   Select playerID, finalgame_date, debut_date, Cast ((
       JulianDay(finalgame_date) - JulianDay(debut_date)
   ) As Integer) AS TotalGames 
   FROM people
   ORDER BY TotalGames DESC;
   """
   ```

6. What is the distribution of debut months? *Hint:* Look at the `DATE` and [`EXTRACT`](https://www.postgresql.org/docs/current/static/functions-datetime.html#FUNCTIONS-DATETIME-EXTRACT) functions.

   ```sqlite
   distribution_query = """
   SELECT strftime('%m',debut_date) as month, COUNT(strftime('%m',debut_date))
   FROM people
   GROUP BY month
   """
   ```

   

7. What is the effect of table join order on mean salary for the players listed in the main (master) table? *Hint:* Perform two different queries, one that joins on playerID in the salary table and other that joins on the same column in the master table. You will have to use left joins for each since right joins are not currently supported with SQLalchemy.

Given there are more peopleID's in the people dataset than in the salaries dataset, there are salaries for all playerIDs when the people dataset is left joined and "none" salaries for some players when the salaries dataset is left joined.

```sqlite
query1 = """
SELECT Salaries.playerid, AVG(salary)
    FROM Salaries 
    LEFT JOIN people
    ON Salaries.playerid = People.playerid
    GROUP BY Salaries.playerID
    ORDER BY AVG(salary) ASC
    LIMIT 10
"""
```

```sqlite
query2 ="""
SELECT people.playerid, AVG(salary)
    FROM people LEFT JOIN Salaries
    ON people.playerid = Salaries.playerid
    GROUP BY people.playerid
    ORDER BY AVG(salary) ASC
    LIMIT 10
"""
```

