# SQLZOO

## Exercises
* [SELECT basics](#select-basics)
* [SELECT from WORLD](#select-from-world)
* [SELECT from NOBEL](#select-from-nobel)
* [SELECT in SELECT](#select-in-select)
* [Aggregate functions](#aggregate-functions)
* [JOIN operation](#join-operation)
* [More JOIN](#more-join)

## table used
[`world`](https://sqlzoo.net/wiki/Read_the_notes_about_this_table.) </br>
world(name, continent, area, population, gdp) </br>
nobel(yr, subject, winner)


### SELECT basics


1. ##### Introducing the `world`
```sql
SELECT population FROM world
WHERE name = 'Germany';
```

2. ##### Scandinavia
`IN` operator allows you to specify multiple values in a WHERE clause
```sql
SELECT name, population FROM world
WHERE name IN ('Sweden', 'Norway', 'Denmark');
```

3. ##### Just the right size
`BETWEEN` operator selects values within a given range. The values can be numbers, text, or dates
```sql
SELECT name, area FROM world
WHERE area BETWEEN 200000 AND 250000;
```


### SELECT from WORLD


1. ##### Introduction
```sql
SELECT name, continent, population FROM world;
```
2. ##### Large Countries
```sql
SELECT name FROM world
WHERE population >= 200000000;
```
3. ##### Per capita GDP
```sql
SELECT name, gdp/population
FROM world
WHERE population >= 200000000;
```
4. ##### South America In millions
`LIKE` is case-insensitive search/match operator, can be used with wildcards:% representing zero or more characters, _ representing a single character
```sql
SELECT name, population/1000000 AS population_milion
FROM world
WHERE continent LIKE 'south america';
```
5. ##### France, Germany, Italy
```sql
SELECT name, population
FROM world
WHERE name IN('France', 'Germany', 'Italy');
```
6. ##### United
```sql
SELECT name
FROM world
WHERE name LIKE '%united%';
```
7. ##### Two ways to be big
```sql
SELECT name, population, area
FROM world
WHERE area > 3000000 OR population > 250000000;
```
8. ##### One or the other (but not both)
```sql
SELECT name, population, area
FROM world
WHERE area > 3000000 XOR population > 250000000;
```
9. ##### Rounding
`ROUND(f,p)` returns f rounded to p decimals
```sql
SELECT name, ROUND(population/1000000, 2), ROUND(gdp/1000000000, 2)
FROM world
WHERE continent = 'South America';
```
10. ##### Trillion dollar economies
```sql
SELECT name, ROUND(gdp/population, -3)
FROM world
WHERE gdp >= 1000000000000;
```
11. ##### Name and capital have the same length
`LENGTH(n)` function finds the number of characters in string
```sql
SELECT name, capital
FROM world
WHERE LENGTH(name) = LENGTH(capital);
```
12. ##### Matching name and capital
`LEFT(s,n)` function extracts n characters from the start of the string
```sql
SELECT name, capital
FROM world
WHERE LEFT(name,1)=LEFT(capital,1) AND name <> capital;
```
13. ##### All the vowels
```sql
SELECT name
FROM world
WHERE name LIKE '%a%' AND name LIKE '%e%' AND name LIKE '%i%' AND name LIKE '%o%' AND name LIKE '%u%'
AND name NOT LIKE '% %';
```


### SELECT from NOBEL


1. ##### Winners from 1950
```sql
SELECT yr, subject, winner
FROM nobel
WHERE yr = 1950;
```
2. ##### 1962 Literature
```sql
SELECT winner
FROM nobel
WHERE yr = 1962
AND subject = 'Literature';
```
3. ##### Albert Einstein
```sql
SELECT yr, subject
FROM nobel
WHERE winner = 'Albert Einstein';
```
4. ##### Recent Peace Prizes
```sql
SELECT winner
FROM nobel
WHERE subject = 'Peace' AND yr >= 2000;
```
5. ##### Literature in the 1980's
```sql
SELECT yr, subject, winner
FROM nobel
WHERE subject = 'Literature' AND yr BETWEEN 1980 AND 1989;
```
6. ##### Only Presidents
```sql
SELECT * FROM nobel
WHERE winner IN('Theodore Roosevelt', 
                'Woodrow Wilson', 
                'Jimmy Carter', 
                'Barack Obama');
```
7. ##### John
```sql
SELECT winner FROM nobel
WHERE winner LIKE 'John%';
```
8. ##### Chemistry and Physics from different years
```sql
SELECT yr, subject, winner 
FROM nobel
WHERE subject = 'Physics' AND yr = 1980 
OR subject = 'Chemistry' AND yr = 1984;
```
9. ##### Exclude Chemists and Medics
```sql
SELECT yr, subject, winner FROM nobel
WHERE yr = 1980 AND subject NOT IN('Chemistry', 'Medicine');
```
10. ##### Early Medicine, Late Literature
```sql
SELECT yr, subject, winner
FROM nobel
WHERE subject = 'Medicine' AND yr < 1910
OR subject = 'Literature' AND yr >= 2004;
```
11. ##### Umlaut
```sql
SELECT * FROM nobel
WHERE winner LIKE 'Peter Gr_nberg';
```
12. ##### Apostrophe
Apostrophe is escaped by typing it double `''`
```sql
SELECT * FROM nobel
WHERE winner LIKE 'Eugene O''neill';
```
13. ##### Knights of the realm
```sql
SELECT winner, yr, subject FROM nobel
WHERE winner LIKE 'Sir%' ORDER BY yr DESC;
```
14. ##### Chemistry and Physics last 
`IN` in `order` is returned as boolean, so if it's true it will return 1, and place it afther all zeroes
```sql
SELECT winner, subject FROM nobel
WHERE yr = 1984
ORDER BY subject IN('Chemistry','Physics'), subject, winner;
```
Solution above is not standard sql and may not work in all databeses, so better use `CASE`
```sql
SELECT winner, subject FROM nobel
WHERE yr = 1984
ORDER BY
CASE WHEN subject IN ('Physics', 'Chemistry') THEN 1 ELSE 0 end,
subject, winner;
```


### SELECT in SELECT


The result of a SELECT statement may be used as a value in another statement. (some versions insist on using alias on subquery `AS somone`)
The subquery may return more than one result so it is safer to use `IN` to cope with this possibility.
example:
```
SELECT name, continent FROM world
WHERE continent IN
(SELECT continent FROM world WHERE name='Brazil'
                                 OR name='Mexico')
```

You can use the words ALL or ANY where the right side of the operator might have multiple values.


1. ##### Bigger than Russia
```sql
SELECT name FROM world
WHERE population > 
(SELECT population FROM world WHERE name = 'Russia');
```
2. ##### Richer than UK
```sql
SELECT name
FROM world
WHERE continent='Europe' AND
gdp/population>(
  SELECT gdp/population FROM world 
  WHERE name='United Kingdom'
);
```
3. ##### Neighbours of Argentina and Australia
```sql
SELECT name, continent FROM world
WHERE continent IN (
  SELECT continent FROM world 
  WHERE name='Argentina' OR name='Australia'
) ORDER by name;
```
4. ##### Between Canada and Poland
```sql
SELECT name, population FROM world
WHERE population >(
  SELECT population FROM world WHERE name='Canada')
and population <(
  select population FROM world WHERE name='Poland'
);
```
5. ##### Percentages of Germany
use `CONCAT` to add '%' char 
```sql
SELECT name, 
  CONCAT(ROUND(100*population/(SELECT population FROM world WHERE name='Germany')), '%')
FROM world
WHERE continent='Europe';
```
6. ##### Bigger than every country in Europe
Some values may have NULL so use IS NOT NULL or > 0;
```sql
SELECT name FROM world
WHERE gdp > ALL(SELECT gdp FROM world WHERE continent='Europe' AND gdp IS NOT NULL);
```
7. ##### Largest in each continent
```sql
SELECT continent, name, area FROM world x
WHERE area >= ALL(SELECT area FROM world y
                  WHERE y.continent=x.continent
                  AND area>0);
```
8. ##### First country of each continent (alphabetically)
```sql
SELECT continent, name FROM world x
WHERE name <= ALL (SELECT name FROM world y
                   WHERE y.continent=x.continent)
```
9. ##### 
```sql
SELECT name, continent, population FROM world x
WHERE 25000000 >= ALL(SELECT population FROM world y
                      WHERE y.continent = x.continent AND y.population>0);
```
10. ##### 
```sql
SELECT name, continent FROM world x
WHERE population >= ALL(SELECT population*3 FROM world y
                        WHERE x.continent=y.continent AND y.name!=x.name);
```

#### NOTE ON SUBQUERIES
   * [stackoverflow](https://stackoverflow.com/questions/46927348/why-does-this-correlated-subquery-work-sqlzoo-select-within-select-7)


### Aggregate functions


* An aggregate function takes many values and delivers just one value. These functions are even more useful when used with the GROUP BY clause.
* `DISTINCT`-By default the result of a SELECT may contain duplicate rows. We can remove these duplicates using the DISTINCT key word.
* `ORDER BY`-ORDER BY permits us to see the result of a SELECT in any particular order. We may indicate ASC or DESC for ascending (smallest first, largest last) or descending order.
* By including a GROUP BY clause functions such as SUM and COUNT are applied to groups of items sharing values. When you specify GROUP BY continent the result is that you get only one 
row for each different value of continent. All the other columns must be "aggregated" by one of SUM, COUNT
* The HAVING clause allows use to filter the groups which are displayed. The WHERE clause filters rows before the aggregation, the HAVING clause filters after the aggregation
if a ORDER BY clause is included we can refer to columns by their position.


1. ##### Total world population
```sql
SELECT SUM(population)
FROM world;
```
2. ##### List of continents
```sql
SELECT DISTINCT continent FROM world;
```
3. ##### GDP of Africa
```sql
SELECT SUM(gdp) FROM world
WHERE continent='Africa';
```
4. ##### Count the big countries
```sql
SELECT COUNT(name) FROM world
WHERE area >= 1000000;
```
5. ##### Baltic states population
```sql
SELECT SUM(population) FROM world
WHERE name IN('Estonia','Latvia','Lithuania');
```
6. ##### Counting the countries of each continent
```sql
SELECT continent, COUNT(name) FROM world
GROUP BY continent;
```
7. ##### Counting big countries in each continent
```sql
SELECT continent, COUNT(name) FROM world
WHERE population > 10000000
GROUP BY continent;
```
8. ##### Counting big continents
```sql
SELECT continent FROM world
GROUP BY continent
HAVING SUM(population) > 100000000;
```


### JOIN operation


1. 
```sql
SELECT matchid, player FROM goal 
WHERE teamid = 'GER';
```
2. 
```sql
SELECT id,stadium,team1,team2
 FROM game
WHERE id = 1012;
```
3. 
```sql
SELECT player,teamid,stadium,mdate
FROM goal JOIN game 
ON game.id=goal.matchid
WHERE teamid = 'GER';
```
4. 
```sql
SELECT team1,team2,player
FROM goal JOIN game
ON game.id=goal.matchid
WHERE player LIKE 'mario%';
```
5. 
```sql
SELECT player, teamid, coach, gtime
FROM goal JOIN eteam
ON goal.teamid=eteam.id 
WHERE gtime<=10;
```
6. 
```sql
SELECT mdate, teamname
FROM game JOIN eteam
ON game.team1 = eteam.id
WHERE coach='Fernando Santos';
```
7. 
```sql
SELECT player 
FROM  goal JOIN game
ON game.id=goal.matchid
WHERE stadium='National Stadium, Warsaw';
```
8. 
```sql
SELECT DISTINCT player
FROM goal JOIN game
ON game.id=goal.matchid
WHERE goal.teamid<>'GER' AND (team1='GER' OR team2='GER');
```
9. 
```sql
SELECT teamname, COUNT(*)
FROM eteam JOIN goal
ON id=teamid
GROUP BY teamname;
```
10. 
```sql
SELECT stadium, COUNT(*)
FROM game JOIN goal
ON game.id=goal.matchid
GROUP BY stadium;
```
11. 
```sql
SELECT matchid, mdate, COUNT(player)
FROM game JOIN goal
ON (matchid=id AND (team1 = 'POL' OR team2 = 'POL'))
GROUP BY matchid, mdate
```
12. 
```sql
SELECT matchid, mdate, COUNT(*) AS goals
FROM game JOIN goal
ON id=matchid
WHERE goal.teamid='GER'
GROUP BY matchid,mdate;
```
13. 
```sql
SELECT mdate,
       team1,
       SUM(CASE WHEN teamid = team1 THEN 1 ELSE 0 END) AS score1,
       team2,
       SUM(CASE WHEN teamid = team2 THEN 1 ELSE 0 END) AS score2 FROM
    game LEFT JOIN goal ON (id = matchid)
    GROUP BY mdate,team1,team2
    ORDER BY mdate, matchid, team1, team2;
```

### More JOIN


1. 
```sql
SELECT id, title
FROM movie
WHERE yr=1962
```
2. 
```sql
SELECT yr FROM movie
WHERE title='Citizen Kane';
```
3. 
```sql
SELECT id, title, yr FROM movie
WHERE title LIKE '%star trek%'
ORDER BY yr;
```
4. 
```sql
SELECT id FROM actor
WHERE name='Glenn Close';
```
5. 
```sql
SELECT id FROM movie
WHERE title='Casablanca';
```
6. 
```sql
select name FROM actor 
JOIN casting ON actorid=actor.id
JOIN movie ON movieid=movie.id
WHERE movie.title='Casablanca';
```
7. 
```sql
SELECT name FROM actor 
JOIN casting ON actor.id=casting.actorid
JOIN movie ON casting.movieid=movie.id
WHERE movie.title='Alien';
```
8. 
```sql
SELECT title 
FROM movie JOIN casting
ON id=movieid
JOIN actor
ON actorid=actor.id
WHERE actor.name='Harrison Ford';
```
9. 
```sql
SELECT title 
FROM movie JOIN casting
ON id=movieid
JOIN actor
ON actorid=actor.id
WHERE actor.name='Harrison Ford' AND ord<>1;
```
10. 
```sql
SELECT title, name
FROM movie JOIN casting
ON id=movieid
JOIN actor
ON actorid=actor.id
WHERE yr=1962 AND ord=1;
```
11. 
```sql
SELECT yr,COUNT(title) FROM
  movie JOIN casting ON movie.id=movieid
        JOIN actor   ON actorid=actor.id
WHERE name='Rock Hudson'
GROUP BY yr
HAVING COUNT(title) > 2
```
12. 
```sql
SELECT title, name
FROM movie JOIN casting ON (movieid=movie.id AND ord=1)
JOIN actor ON actorid=actor.id
WHERE movie.id IN(
                 SELECT movieid FROM casting 
                 WHERE actorid IN(
                                  SELECT id FROM actor 
                                  WHERE name='Julie Andrews' ));
```
13. 
```sql
SELECT DISTINCT name FROM casting
JOIN actor ON actor.id = actorid
WHERE actorid IN (
	  SELECT actorid FROM casting
	  WHERE ord = 1
	  GROUP BY actorid
	  HAVING COUNT(actorid) >= 30)
```
14. 
```sql
SELECT title, COUNT(actorid) FROM casting
  JOIN movie ON movieid = movie.id
  WHERE yr = 1978
  GROUP BY movieid, title
  ORDER BY COUNT(actorid) DESC
```
15. 
```sql
SELECT DISTINCT name FROM actor JOIN casting
ON actor.id=actorid
WHERE name<>'Art Garfunkel' AND movieid IN (
  SELECT movieid FROM casting 
  JOIN actor ON (actorid=id AND name='Art Garfunkel'))
```


