<h3 align="center"> All of my answers in the PostgreSQL Exercises <hr>

Click [Here](https://pgexercises.com/), If you want to improve your PostgreSQL skills and acquire more experience with difficult queries  <hr>


# Simple SQL Queries

Exercise 1 

1. How can you retrieve all the information from the cd.facilities table?
```sql
SELECT * FROM cd.facilities;
```
2. You want to print out a list of all of the facilities and their cost to members. How would you retrieve a list of only facility names and costs?
```sql
SELECT name, membercost FROM cd.facilities;
```
3. How can you produce a list of facilities that charge a fee to members?
```sql
SELECT * FROM cd.facilities WHERE membercost != 0 ;
```
4. How can you produce a list of facilities that charge a fee to members, and that fee is less than 1/50th of the monthly maintenance cost? Return the facid, facility name, member cost, and monthly maintenance of the facilities in question.
```sql
SELECT facid,name,membercost,monthlymaintenance FROM cd.facilities 
WHERE membercost != 0 AND membercost <= monthlymaintenance /50.0;
```
5. How can you produce a list of all facilities with the word 'Tennis' in their name?
```sql
SELECT * FROM cd.facilities WHERE name LIKE  '%Tennis%';
```
6. How can you retrieve the details of facilities with ID 1 and 5? Try to do it without using the OR operator.
```sql
SELECT * FROM cd.facilities 
WHERE name IN ('Tennis Court 2', 'Massage Room 2');
```
OR 
```sql
SELECT * FROM cd.facilities 
  WHERE facid IN (1,5);                                                                                                                  
```
7. How can you produce a list of facilities, with each labelled as 'cheap' or 'expensive' depending on if their monthly maintenance cost is more than $100? 
   
   Return the name and monthly maintenance of the facilities in question.
```sql
SELECT name, 
 CASE WHEN monthlymaintenance > 100.0 THEN 'expensive'
	ELSE 'cheap' END AS cost
FROM cd.facilities;               
```

8. How can you produce a list of members who joined after the start of September 2012? 
   Return the memid, surname, firstname, and joindate of the members in question.
```sql
SELECT memid, surname, firstname, joindate FROM cd.members
WHERE joindate >= '2012-09-01';  
```                                                         
9. How can you produce an ordered list of the first 10 surnames in the members table? The list must not contain duplicates.
```sql                                                         
SELECT DISTINCT surname FROM cd.members 
ORDER BY surname
LIMIT 10;
```                                                         
10. You, for some reason, want a combined list of all surnames and all facility names. Yes, this is a contrived example :-). Produce that list!                                   ```sql
SELECT surname FROM cd.members 
UNION 
SELECT name FROM cd.facilities;
```                      
11. You'd like to get the signup date of your last member. How can you retrieve this information?
```sql
SELECT joindate AS latest FROM cd.members 
ORDER BY joindate DESC
LIMIT 1 ;
```
OR 
```sql
SELECT MAX(joindate) AS latest FROM cd.members;
```  
12. You'd like to get the first and last name of the last member(s) who signed up - not just the date. How can you do that?
```sql
SELECT firstname, surname, joindate FROM cd.members
ORDER BY joindate DESC
LIMIT 1 ;
```    
OR 
```sql
SELECT firstname, surname, joindate FROM cd.members
WHERE joindate = ( SELECT MAX(joindate) FROM cd.members);
```   
  
  
# Joins and Subqueries

Exercise 2 

1. How can you produce a list of the start times for bookings by members named 'David Farrell'?
```sql
SELECT starttime FROM cd.bookings
INNER JOIN cd.members ON 
cd.members.memid = cd.bookings.memid 
WHERE firstname = 'David' AND surname = 'Farrell';
```
OR 
```sql
SELECT bks.starttime FROM cd.bookings bks
INNER JOIN cd.members mems ON
mems.memid = bks.memid
WHERE mems.firstname = 'David' AND mems.surname = 'Farrell';
```
2. How can you produce a list of the start times for bookings for tennis courts, for the date '2012-09-21'? 
	Return a list of start time and facility name pairings, ordered by the time.
```sql
SELECT bks.starttime As start, fcts.name AS name 
FROM cd.facilities fcts
INNER JOIN cd.bookings bks ON 
fcts.facid = bks.facid
WHERE fcts.name IN ('Tennis Court 1','Tennis Court 2') AND
bks.starttime >= '2012-09-21' AND bks.starttime < '2012-09-22'
ORDER BY bks.starttime;
```
OR
```sql
SELECT bks.starttime as start, fcts.name as name
FROM cd.facilities fcts
INNER JOIN cd.bookings bks ON 
fcts.facid = bks.facid
WHERE fcts.facid in (0,1) and
	  bks.starttime >= '2012-09-21' AND
	  bks.starttime < '2012-09-22'
ORDER BY bks.starttime;   
```						       
3. How can you output a list of all members who have recommended another member? 
Ensure that there are no duplicates in the list, and that results are ordered by (surname, firstname).
```sql
SELECT DISTINCT rec.firstname AS firstname, rec.surname AS surname 
FROM cd.members mems 
INNER JOIN cd.members rec ON
rec.memid = mems.recommendedby
ORDER BY surname ,firstname;```	       
4. How can you output a list of all members, including the individual who recommended them (if any)? Ensure that results are ordered by (surname, firstname).
```sql
SELECT Title, Year FROM movies LIMIT 5;
```
5. 
```sql
SELECT Title, Year FROM movies LIMIT 5;
```
6. 
```sql
SELECT Title, Year FROM movies LIMIT 5;
```
7. 
```sql
SELECT Title, Year FROM movies LIMIT 5;
```
8. 
```sql
SELECT Title, Year FROM movies LIMIT 5;
```

  

# SQL Lesson 3: Queries with constraints (Pt. 2)

Exercise 3 — Tasks

1. Find all the Toy Story movies
```sql
SELECT * FROM movies WHERE Title LIKE '%Toy Story%';
```
2. Find all the movies directed by John Lasseter
```sql
SELECT * FROM movies WHERE Director = 'John Lasseter';
```
3. Find all the movies (and director) not directed by John Lasseter
```sql
SELECT Title, Director FROM movies WHERE Director != 'John Lasseter';
```
4. Find all the WALL-* movies
```sql
SELECT Title FROM movies WHERE Title LIKE '%WALL%';
```


# SQL Lesson 4: Filtering and sorting Query results

Exercise 4 — Tasks

1. List all directors of Pixar movies (alphabetically), without duplicates
```sql
SELECT DISTINCT Director FROM movies ORDER BY Director;
```
2. List the last four Pixar movies released (ordered from most recent to least)
```sql
SELECT Title FROM movies ORDER BY Year DESC LIMIT 4;
```
3. List the first five Pixar movies sorted alphabetically
```sql
SELECT Title FROM movies ORDER BY Title LIMIT 5;
```
4. List the next five Pixar movies sorted alphabetically
```sql
SELECT Title FROM movies ORDER BY Title LIMIT 5 OFFSET 5;
```


# SQL Review: Simple SELECT Queries

Review 1 — Tasks

1. List all the Canadian cities and their populations
```sql
SELECT City, Population FROM north_american_cities
WHERE Country = 'Canada';
```
2. Order all the cities in the United States by their latitude from north to south
```sql
SELECT City FROM north_american_cities
WHERE Country = 'United States'
ORDER BY Latitude DESC;
```
3. List all the cities west of Chicago, ordered from west to east
```sql
SELECT City, Longitude FROM north_american_cities
WHERE Longitude < -87.629798
ORDER BY Longitude ASC;
```
4.List the two largest cities in Mexico (by population)
```sql
SELECT City, Population FROM north_american_cities
WHERE Country = 'Mexico'
ORDER BY Population DESC
LIMIT 2;
```
5. List the third and fourth largest cities (by population) in the United States and their population
```sql
SELECT City, Population FROM north_american_cities
WHERE Country LIKE 'United States'
ORDER BY Population 
LIMIT 2 OFFSET 2;
```


# SQL Lesson 6: Multi-table queries with JOINs

Exercise 6 — Tasks

1. Find the domestic and international sales for each movie
```sql
SELECT Title , Domestic_sales, International_sales FROM Movies 
INNER JOIN Boxoffice ON
Boxoffice.Movie_id = Movies.Id;
```
2. Show the sales numbers for each movie that did better internationally rather than domestically
```sql
SELECT Title, Domestic_sales,International_sales FROM Movies
INNER JOIN Boxoffice ON 
Movies.Id = Boxoffice.Movie_id
WHERE International_sales > Domestic_sales;
```
3. List all the movies by their ratings in descending order
```sql
SELECT Title, Rating FROM Movies
INNER JOIN Boxoffice ON
Movies.Id = Boxoffice.Movie_id
ORDER BY Rating DESC;
```


# SQL Lesson 7: OUTER JOINs

Exercise 7 — Tasks

1. Find the list of all buildings that have employees
```sql
SELECT DISTINCT Building FROM Employees;
```
2. Find the list of all buildings and their capacity
```sql
SELECT * FROM Buildings;
```
3. List all buildings and the distinct employee roles in each building (including empty buildings)
```sql
SELECT DISTINCT Building_name, Role FROM Buildings 
  LEFT JOIN Employees ON Building_name = Building;
```


# SQL Lesson 8: A short note on NULLs

Exercise 8 — Tasks

1. Find the name and role of all employees who have not been assigned to a building
```sql
SELECT Name,Role FROM employees WHERE Building IS NULL;
```
2. Find the names of the buildings that hold no employees
```sql
SELECT DISTINCT Building_name FROM Buildings 
  LEFT JOIN Employees ON Building_name = Building
    WHERE Role IS NULL;
```


# SQL Lesson 9: Queries with expressions

Exercise 9 — Tasks

1. List all movies and their combined sales in millions of dollars
```sql
SELECT DISTINCT Title, (Domestic_sales + International_sales) / 1000000 AS Sales FROM Movies
INNER JOIN Boxoffice ON 
Movies.Id = Boxoffice.Movie_id;
```
2. List all movies and their ratings in percent
```sql
SELECT DISTINCT Title, (Rating * 10) AS Rating_Percent FROM Movies 
INNER JOIN Boxoffice  ON 
Movies.Id = Boxoffice.Movie_id;

```
3. List all movies that were released on even number years
```sql
SELECT Title FROM Movies WHERE Year % 2 = 0;
```


# SQL Lesson 10: Queries with aggregates (Pt. 1)

Exercise 10 — Tasks

1. Find the longest time that an employee has been at the studio 
```sql
SELECT MAX(Years_employed) FROM employees;
```
2. For each role, find the average number of years employed by employees in that role
```sql
SELECT Role, AVG(Years_employed) AS Average_Years FROM Employees GROUP BY Role;
```
3. Find the total number of employee years worked in each building
```sql
SELECT DISTINCT Building, SUM(Years_employed) FROM Employees
GROUP BY Building;
```


# SQL Lesson 11: Queries with aggregates (Pt. 2)

Exercise 11 — Tasks

1. Find the number of Artists in the studio (without a HAVING clause)
```sql
SELECT COUNT(*) FROM Employees 
WHERE Role LIKE 'Artist';
```
2. Find the number of Employees of each role in the studio
```sql
SELECT DISTINCT Role, COUNT(*) FROM Employees 
GROUP BY Role;
```
3. Find the total number of years employed by all Engineers. 
```sql
SELECT Role, SUM(Years_employed) FROM Employees
GROUP BY Role
HAVING Role = 'Engineer';
```


# SQL Lesson 12: Order of execution of a Query

Exercise 12 — Tasks

1. Find the number of movies each director has directed
```sql
SELECT Director , COUNT(Title) AS Amount_of_Movies FROM Movies 
GROUP BY Director;
```
2. Find the total domestic and international sales that can be attributed to each director
```sql
SELECT Director, SUM(Domestic_sales + International_sales) AS Sales FROM Movies 
INNER JOIN Boxoffice  ON
Movies.Id = Boxoffice.Movie_id
GROUP BY Director;
```


# SQL Lesson 13: Inserting rows

Exercise 13 — Tasks

1. Add the studio's new production, Toy Story 4 to the list of movies (you can use any director)
```sql
INSERT INTO Movies VALUES (4, 'Toy Story 4', 'John Lasseter', 2013,100);
```
2.Toy Story 4 has been released to critical acclaim! It had a rating of 8.7, and made 340 million domestically and 270 million internationally. Add the record to the BoxOffice table.
```sql
INSERT INTO Boxoffice VALUES(4,8.7,340000000,270000000);
```


# SQL Lesson 14: Updating rows

Exercise 14 — Tasks

1. The director for A Bug's Life is incorrect, it was actually directed by John Lasseter
```sql
UPDATE Movies
SET Director = 'John Lasseter'
WHERE Id = 2;
```

OR 

```sql
UPDATE Movies
SET Director = 'John Lasseter'
WHERE Title = "A Bug's Life";
```

2. The year that Toy Story 2 was released is incorrect, it was actually released in 1999
```sql
UPDATE Movies
SET Year = '1999'
WHERE Title = 'Toy Story 2'
```

3. Both the title and director for Toy Story 8 is incorrect! The title should be "Toy Story 3" and it was directed by Lee Unkrich
```sql
UPDATE Movies
SET Title = 'Toy Story 3', Director = 'Lee Unkrich'
WHERE Movies.Id = 11;
```


# SQL Lesson 15: Deleting rows

Exercise 15 — Tasks

1. This database is getting too big, lets remove all movies that were released before 2005.
```sql
DELETE FROM Movies WHERE Year < 2005;
```
2.Andrew Stanton has also left the studio, so please remove all movies directed by him.
```sql
DELETE FROM Movies WHERE Director = 'Andrew Stanton';
```


# SQL Lesson 16: Creating tables

Exercise 16 — Tasks

1. Create a new table named Database with the following columns:
  – Name A string (text) describing the name of the database
  – Version A number (floating point) of the latest version of this database
  – Download_count An integer count of the number of times this database was downloaded

This table has no constraints.

```sql
CREATE TABLE Database
(Name TEXT,
Version REAL,
Download_count INTEGER);
```


# SQL Lesson 17: Altering tables

Exercise 17 — Tasks

1. Add a column named Aspect_ratio with a FLOAT data type to store the aspect-ratio each movie was released in.
```sql
ALTER TABLE  Movies 
  ADD Aspect_ratio FLOAT;
```

2. Add another column named Language with a TEXT data type to store the language that the movie was released in. Ensure that the default for this language is English.
```sql
ALTER TABLE Movies
  ADD COLUMN Language TEXT DEFAULT 'English';
```


# SQL Lesson 18: Dropping tables

Exercise 18 — Tasks

1. We've sadly reached the end of our lessons, lets clean up by removing the Movies table
```sql
DROP TABLE IF EXISTS Movies ;
```

2. And drop the BoxOffice table as well
```sql
DROP TABLE IF EXISTS BoxOffice  ;
```


<h3 align="center">  THE END   <hr></h3>
