(1) Query the two cities in STATION with the shortest and longest CITY names, as well as their respective lengths (i.e.: number of characters in the name). If there is more than one smallest or largest city, choose the one that comes first when ordered alphabetically.

```
SELECT CITY, LENGTH(CITY) 
FROM STATION 
WHERE LENGTH(CITY) = (SELECT MIN(LENGTH(CITY))FROM STATION)
ORDER BY CITY LIMIT 1;

SELECT CITY, LENGTH(CITY) 
FROM STATION 
WHERE LENGTH(CITY) = (SELECT MAX(LENGTH(CITY))FROM STATION)
ORDER BY CITY LIMIT 1;
```

(2) You are given a table, BST, containing two columns: N and P, where N represents the value of a node in Binary Tree, and P is the parent of N.

```
SELECT N,
CASE
    WHEN P IS NULL THEN " Root"
    WHEN (N IN (SELECT P FROM BST)) THEN " Inner"
    ELSE " Leaf"
END AS trees
FROM BST
Order By N;
```

(3) A median is defined as a number separating the higher half of a data set from the lower half. Query the median of the Northern Latitudes (LAT_N) from STATION and round your answer to  decimal places.

```
SET @rowindex := -1;

SELECT ROUND(AVG(LAT_N), 4)
FROM (SELECT @rowindex := @rowindex + 1 as row_index, LAT_N
      FROM STATION
      ORDER BY LAT_N) AS t
WHERE row_index IN (FLOOR(@rowindex / 2), CEIL(@rowindex / 2));
```

(4) Ketty gives Eve a task to generate a report containing three columns: Name, Grade and Mark. Ketty doesn't want the NAMES of those students who received a grade lower than 8. The report must be in descending order by grade -- i.e. higher grades are entered first. If there is more than one student with the same grade (8-10) assigned to them, order those particular students by their name alphabetically. Finally, if the grade is lower than 8, use "NULL" as their name and list them by their grades in descending order. If there is more than one student with the same grade (1-7) assigned to them, order those particular students by their marks in ascending order.

```
SELECT
CASE
    WHEN grades.grade > 7 THEN students.name ELSE 'NULL' 
END, grades.grade, students.marks 
FROM students INNER JOIN grades ON students.marks BETWEEN grades.min_mark AND grades.max_mark 
ORDER BY grades.grade DESC, students.name
```

(5) Harry Potter and his friends are at Ollivander's with Ron, finally replacing Charlie's old broken wand.

Hermione decides the best way to choose is by determining the minimum number of gold galleons needed to buy each non-evil wand of high power and age. Write a query to print the id, age, coins_needed, and power of the wands that Ron's interested in, sorted in order of descending power. If more than one wand has same power, sort the result in order of descending age.

```
SELECT w.id, wp.age, w.coins_needed, w.power
FROM wands w JOIN wands_property wp USING(code)
WHERE wp.is_evil = 0 AND w.coins_needed = (SELECT MIN(coins_needed) 
                                           FROM wands 
                                           WHERE wands.code = w.code and wands.power = w.power 
                                           group by code, power)
ORDER BY w.power DESC, wp.age DESC;
```

(6) Julia asked her students to create some coding challenges. Write a query to print the hacker_id, name, and the total number of challenges created by each student. Sort your results by the total number of challenges in descending order. If more than one student created the same number of challenges, then sort the result by hacker_id. If more than one student created the same number of challenges and the count is less than the maximum number of challenges created, then exclude those students from the result.

```
SELECT h.hacker_id, h.name, COUNT(c.challenge_id) 
FROM hackers h JOIN challenges c USING(hacker_id)
GROUP BY h.hacker_id, h.name
HAVING
    COUNT(c.challenge_id) = 
        (SELECT MAX(temp1.cnt)
         FROM (SELECT COUNT(hacker_id) as cnt
               FROM Challenges
               GROUP BY hacker_id
               ORDER BY hacker_id) temp1)
    OR COUNT(c.challenge_id) IN
        (SELECT temp2.cnt
         FROM (SELECT count(*) as cnt 
               FROM challenges
               GROUP BY hacker_id) temp2
         GROUP BY temp2.cnt
         HAVING COUNT(temp2.cnt) = 1)
ORDER BY COUNT(c.challenge_id) DESC, h.hacker_id
```

(7) The total score of a hacker is the sum of their maximum scores for all of the challenges. Write a query to print the hacker_id, name, and total score of the hackers ordered by the descending score. If more than one hacker achieved the same total score, then sort the result by ascending hacker_id. Exclude all hackers with a total score of  from your result.

```
SELECT h.hacker_id, h.name, SUM(c.score) as total_score
FROM hackers h JOIN
(SELECT hacker_id,  MAX(score) AS score 
                    FROM submissions 
                    GROUP BY challenge_id, hacker_id) c
USING (hacker_id)
GROUP BY h.hacker_id, h.name
HAVING total_score > 0
ORDER BY total_score DESC, h.hacker_id;
```