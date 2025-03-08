# TABLE OF CONTENTS
**1.ACKNOWLEDGEMENT**

**2.PROJECT OVERVIEW**

**3.DATA SOURCES**

**4.DATA ANALYSIS**

**5.CONCLUSION**


# ACKNOWLEDGEMENT
This project is prepared with the assistance of the course **[SQL for Data Analysis: Advanced SQL Querying Techniques](https://www.udemy.com/course/sql-advanced-queries)** on Udemy.

# PROJECT OVERVIEW
Working with a comprehensive historical dataset of Major League Baseball (MLB) players, including statistics such as schools attended, salaries, teams played for, height, weight, and more. The project focuses on leveraging advanced SQL querying techniques to analyze how player statistics have evolved over time and across different teams in the league. 
# 🎯 **PROJECT OBJECTIVES**  

This project aims to explore historical MLB player data using advanced SQL queries to answer key questions:  

1. 🏫 **Player Education:** Identify the schools attended by MLB players.  
2. 💰 **Team Expenditures:** Analyze how much teams spend on player salaries over the years.  
3. 🏆 **Career Progression:** Examine the career timelines of players, from debut to final game.  
4. ⚖️ **Player Comparison:** Compare player attributes such as height, weight, batting, and throwing preferences.  

# DATA SOURCES

There are 4 tables :- players , salaries , school_details ,schools 
- **`players`**: Player biographical and career details.  
  - **PK:** `playerID`  
  - **Key Fields:** Birth/death info, name, weight, height, batting/throwing hand, debut/final game dates, external IDs.  

- **`salaries`**: Annual salary data for players.  
  - **PK:** (`yearID`, `teamID`, `playerID`)  
  - **Key Fields:** Year, team, league, salary (USD).  

- **`school_details`**: School information.  
  - **PK:** `schoolID`  
  - **Key Fields:** Full name, city, state, country.  

- **`schools`**: Player-school associations by year.  
  - **PK:** (`playerID`, `schoolID`, `yearID`)  
  - **Key Fields:** Player ID, school ID, year.
 
### 📊 DATA ANALYSIS
This project involves analyzing historical MLB player data using advanced SQL queries, structured into four main parts: School Analysis, Salary Analysis, Player Career Analysis, and Player Comparison Analysis. Each section addresses specific questions to derive meaningful insights.

##  PART I: SCHOOL ANALYSIS
Focuses on understanding the educational backgrounds of MLB players.

**🔎 Steps:**
**1.Explore School Data: Review the schools and school_details tables.**
 ```sql 
 SELECT * FROM schools;
SELECT * FROM school_details;
```
**2.Decade-wise School Count: Determine how many schools produced players in each decade.**
 ```sql
SELECT 	FLOOR(yearID / 10) * 10 AS decade, COUNT(DISTINCT schoolID) AS num_schools
FROM	schools
GROUP BY decade
ORDER BY decade;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution1-2.JPG)

**3.Top Schools Overall: Identify the top 5 schools that produced the most players.**
 ```sql
SELECT	 sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
FROM	 schools s LEFT JOIN school_details sd
		 ON s.schoolID = sd.schoolID
GROUP BY s.schoolID
ORDER BY num_players DESC
LIMIT 	 5;
```

Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution1-3.JPG)

**4.Top Schools by Decade: For each decade, list the top 3 schools with the highest number of player alumni.**
 ```sql
WITH ds AS (SELECT	 FLOOR(s.yearID / 10) * 10 AS decade, sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
			FROM	 schools s LEFT JOIN school_details sd
					 ON s.schoolID = sd.schoolID
			GROUP BY decade, s.schoolID),
            
	 rn AS (SELECT	decade, name_full, num_players,
					ROW_NUMBER() OVER (PARTITION BY decade ORDER BY num_players DESC) AS row_num
                    /* ALTERNATIVE SOLUTION UPDATE: ROW_NUMBER will return exactly 3 schools for each decade. To account for ties,
                       use DENSE_RANK instead to return the top 3 player counts, which could potentially include more than 3 schools */
			FROM	ds)
            
SELECT	decade, name_full, num_players
FROM	rn
WHERE	row_num <= 3
ORDER BY decade DESC, row_num;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution1-4.JPG)

## PART II: SALARY ANALYSIS

**🔎 Steps:**

**1. View the salaries table.**
 ```sql
SELECT * FROM salaries;
```
**2. Return the top 20% of teams in terms of average annual spending.**
```sql
WITH ts AS (SELECT 	teamID, yearID, SUM(salary) AS total_spend
			FROM	salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID), -- ORDER BY in CTE is not needed and can be omitted
            
	 sp AS (SELECT	teamID, AVG(total_spend) AS avg_spend,
					NTILE(5) OVER (ORDER BY AVG(total_spend) DESC) AS spend_pct
			FROM	ts
			GROUP BY teamID)
            
SELECT	teamID, ROUND(avg_spend / 1000000, 1) AS avg_spend_millions
FROM	sp
WHERE	spend_pct = 1;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution2-2.JPG)

**3. For each team, show the cumulative sum of spending over the years.**
```sql
WITH ts AS (SELECT	 teamID, yearID, SUM(salary) AS total_spend
			FROM	 salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID) -- ORDER BY in CTE is not needed and can be omitted
            
SELECT	teamID, yearID,
		ROUND(SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID) / 1000000, 1)
			AS cumulative_sum_millions
FROM	ts;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution2-3.JPG)

**4. Return the first year that each team's cumulative spending surpassed 1 billion.**
```sql
WITH ts AS (SELECT	 teamID, yearID, SUM(salary) AS total_spend
			FROM	 salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID), -- ORDER BY in CTE is not needed and can be omitted
            
	 cs AS (SELECT	teamID, yearID,
					SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID)
						AS cumulative_sum
			FROM	ts),
            
	 rn AS (SELECT	teamID, yearID, cumulative_sum,
					ROW_NUMBER() OVER (PARTITION BY teamID ORDER BY cumulative_sum) AS rn
			FROM	cs
			WHERE	cumulative_sum > 1000000000)
            
SELECT teamID, yearID, ROUND(cumulative_sum / 1000000000, 2) AS cumulative_sum_billions
FROM	rn
WHERE	rn = 1;
```

Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution2-4.JPG)

## PART III: PLAYER CAREER ANALYSIS

**1. View the players table and find the number of players in the table**
```sql
SELECT * FROM players;
SELECT COUNT(*) FROM players;
```

**2. For each player, calculate their age at their first game, their last game, and their career length (all in years). Sort from longest career to shortest career.**

```sql
SELECT 	nameGiven,
        TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE), debut)
			AS starting_age,
		TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE), finalGame)
			AS ending_age,
		TIMESTAMPDIFF(YEAR, debut, finalGame) AS career_length
FROM	players
ORDER BY career_length DESC;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution3-2.JPG)

**3. What team did each player play on for their starting and ending years?**
```sql
SELECT 	p.nameGiven,
		s.yearID AS starting_year, s.teamID AS starting_team,
        e.yearID AS ending_year, e.teamID AS ending_team
FROM	players p INNER JOIN salaries s
							ON p.playerID = s.playerID
							AND YEAR(p.debut) = s.yearID
				  INNER JOIN salaries e
							ON p.playerID = e.playerID
							AND YEAR(p.finalGame) = e.yearID;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution3-3.JPG)

**4. How many players started and ended on the same team and also played for over a decade?**

```sql
SELECT 	p.nameGiven,
		s.yearID AS starting_year, s.teamID AS starting_team,
        e.yearID AS ending_year, e.teamID AS ending_team
FROM	players p INNER JOIN salaries s
							ON p.playerID = s.playerID
							AND YEAR(p.debut) = s.yearID
				  INNER JOIN salaries e
							ON p.playerID = e.playerID
							AND YEAR(p.finalGame) = e.yearID
WHERE	s.teamID = e.teamID AND e.yearID - s.yearID > 10;
```

Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution3-4.JPG)

## PART IV: PLAYER COMPARISON ANALYSIS

**1. View the players table.**
```sql
SELECT * FROM players;
```

**2. Which players have the same birthday?**

```sql
WITH bn AS (SELECT	CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE) AS birthdate,
					nameGiven
			FROM	players)
            
SELECT	birthdate, GROUP_CONCAT(nameGiven SEPARATOR ', ') AS players
FROM	bn
WHERE	YEAR(birthdate) BETWEEN 1980 AND 1990
GROUP BY birthdate
ORDER BY birthdate;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution4-2.JPG)

**3. Create a summary table that shows for each team, what percent of players bat right, left and both.**
```sql
WITH up AS (SELECT DISTINCT s.teamID, s.playerID, p.bats
           FROM salaries s LEFT JOIN players p
           ON s.playerID = p.playerID) -- unique players CTE

SELECT teamID,
		ROUND(SUM(CASE WHEN bats = 'R' THEN 1 ELSE 0 END) / COUNT(playerID) * 100, 1) AS bats_right,
        ROUND(SUM(CASE WHEN bats = 'L' THEN 1 ELSE 0 END) / COUNT(playerID) * 100, 1) AS bats_left,
        ROUND(SUM(CASE WHEN bats = 'B' THEN 1 ELSE 0 END) / COUNT(playerID) * 100, 1) AS bats_both
FROM up
GROUP BY teamID;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution4-3.JPG)


**4. How have average height and weight at debut game changed over the years, and what's the decade-over-decade difference?**

```sql
WITH hw AS (SELECT	FLOOR(YEAR(debut) / 10) * 10 AS decade,
					AVG(height) AS avg_height, AVG(weight) AS avg_weight
			FROM	players
			GROUP BY decade)
            
SELECT	decade,
		avg_height - LAG(avg_height) OVER(ORDER BY decade) AS height_diff,
        avg_weight - LAG(avg_weight) OVER(ORDER BY decade) AS weight_diff
FROM	hw
WHERE	decade IS NOT NULL;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/Sports_Data_Analytics_MLB_Project1/blob/main/solution4-4.JPG)

## CONCLUSION 

**School analysis focuses on the educational background of MLB players, providing insights into the schools contributing to player development. This data can be used for the following insights:-**

1.Trends in player development pipelines from schools to MLB.

2.Shifts in the geographic distribution of player-producing schools.

3.Long-term influence of top-performing schools on the league.

**Salary analysis provides key insights into team salary spending patterns in Major League Baseball (MLB). This data can be used for the following insights:-**

1.Financial strategies and spending behavior of MLB teams.

2.Comparison between high-spending and low-spending teams.

3.Trends in salary growth and major investment periods across the league.

4.These insights can help correlate team spending with performance, player acquisition strategies, and long-term financial planning.

**Player Career analysis examines the career trajectories of MLB players, focusing on career length, team associations, and player loyalty. This data can be used for the following insights:-**

1.Trends in career longevity among MLB players.

2.The extent of player loyalty to specific teams.

3.Patterns in player movement between teams over the course of their careers.

4.Identification of notable players with long and stable careers.

**Player Comparison analysis focuses on comparing player characteristics across MLB teams, including physical attributes, batting preferences, and unique player coincidences. This data can be used for the following insights:-**

1.Demographic patterns and unique coincidences among players.

2.Team strategies based on batting styles.

3.Evolution of the ideal player physique in MLB over the decades.

4.Insights into how player profiles have shifted in response to changes in the game.
