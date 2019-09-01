# SQLZOO

## Exercises
* [SELECT basics](#select-basics)
* [SELECT from WORLD](#select-from-world)

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
`LIKE` is case-insensitive search/match operator, can be used with wildcards:
	- % representing zero or more characters
	- _ representing a single character
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


