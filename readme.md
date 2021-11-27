<h3 align="center"> All of my answers in the PostgreSQL Exercises <hr>

Click [Here](https://pgexercises.com/), If you want to improve your PostgreSQL skills and acquire more experience with difficult queries  <hr>


# Simple SQL Queries

Exercise 1 

1. How can you retrieve all the information from the cd.facilities table?
```sql
SELECT * FROM cd.facilities;
```
	
2. You want to print out a list of all of the facilities and their cost to members.\
How would you retrieve a list of only facility names and costs?
```sql
SELECT name, membercost FROM cd.facilities;
```
	
3. How can you produce a list of facilities that charge a fee to members?
```sql
SELECT * FROM cd.facilities WHERE membercost != 0 ;
```
	
4. How can you produce a list of facilities that charge a fee to members, and that fee is less than 1/50th of the monthly maintenance cost? \
Return the facid, facility name, member cost, and monthly maintenance of the facilities in question.
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
							 
7. How can you produce a list of facilities, with each labelled as 'cheap' or 'expensive' depending on if their monthly maintenance cost is more than $100? \
   Return the name and monthly maintenance of the facilities in question.
```sql
SELECT name, 
 CASE WHEN monthlymaintenance > 100.0 THEN 'expensive'
	ELSE 'cheap' END AS cost
FROM cd.facilities;               
```

8. How can you produce a list of members who joined after the start of September 2012? \
   Return the memid, surname, firstname, and joindate of the members in question.
```sql
SELECT memid, surname, firstname, joindate FROM cd.members
WHERE joindate >= '2012-09-01';  
```
	
9. How can you produce an ordered list of the first 10 surnames in the members table? \
The list must not contain duplicates.
```sql                                                         
SELECT DISTINCT surname FROM cd.members 
ORDER BY surname
LIMIT 10;
```  
	
10. You, for some reason, want a combined list of all surnames and all facility names. Yes, this is a contrived example :-).\ 
Produce that list!                                   
```sql
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
  
<br>
	
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
2. How can you produce a list of the start times for bookings for tennis courts, for the date '2012-09-21'? \
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
3. How can you output a list of all members who have recommended another member? \
Ensure that there are no duplicates in the list, and that results are ordered by (surname, firstname).
```sql
SELECT DISTINCT rec.firstname AS firstname, rec.surname AS surname 
FROM cd.members mems 
INNER JOIN cd.members rec ON
rec.memid = mems.recommendedby
ORDER BY surname ,firstname;
```	       
4. How can you output a list of all members, including the individual who recommended them (if any)? Ensure that results are ordered by (surname, firstname).
```sql
SELECT mems.firstname AS memfname, mems.surname AS memsname, 
rec.firstname AS recfname, rec.surname AS recsname
FROM cd.members mems
LEFT OUTER JOIN cd.members rec ON 
rec.memid = mems.recommendedby
ORDER BY memsname, memfname;
```
5. How can you produce a list of all members who have used a tennis court? \
Include in your output the name of the court, and the name of the member formatted as a single column. Ensure no duplicate data, and order by the member name followed by the facility name. 
```sql
SELECT DISTINCT mems.firstname || ' ' || mems.surname AS member, fct.name AS facility 
FROM cd.members mems 
INNER JOIN cd.bookings bks ON 
mems.memid = bks.memid 
INNER JOIN cd.facilities fct ON 
bks.facid = fct.facid
WHERE fct.name IN ('Tennis Court 1','Tennis Court 2')
ORDER BY member, facility;
```
6. How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? \
Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. \
Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. \
Order by descending cost, and do not use any subqueries.
```sql
SELECT mems.firstname || ' ' || mems.surname AS member, fct.name AS facility,

CASE 
	
	WHEN mems.memid = 0 THEN 
		bks.slots * fct.guestcost
	ELSE 
		bks.slots * fct.membercost
END AS cost
	
FROM cd.members mems 
	
	INNER JOIN cd.bookings bks ON
	mems.memid = bks.memid
	
	INNER JOIN cd.facilities fct ON 
	bks.facid = fct.facid
	
WHERE bks.starttime >= '2012-09-14' AND bks.starttime < '2012-09-15' AND 
	(
	(mems.memid = 0 AND bks.slots * fct.guestcost > 30 ) OR 
	(mems.memid != 0 AND bks.slots * fct.membercost > 30 ) 
	)
	
ORDER BY cost DESC;			       
```
7. How can you output a list of all members, including the individual who recommended them (if any), without using any joins? \
Ensure that there are no duplicates in the list, and that each firstname + surname pairing is formatted as a column and ordered.
```sql
SELECT DISTINCT mems.firstname || ' ' ||  mems.surname as member,
	
	(SELECT recs.firstname || ' ' || recs.surname as recommender 
		
	 FROM cd.members recs 
		WHERE recs.memid = mems.recommendedby
	)
	FROM 
		cd.members mems

ORDER BY member; 
	
```
8. The Produce a list of costly bookings exercise contained some messy logic: we had to calculate the booking cost in both the WHERE clause and the CASE statement. \
Try to simplify this calculation using subqueries. For reference, the question was:
	
How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30?\
Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. \
Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost.
```sql
SELECT member, facility, cost FROM (
	SELECT 
		mems.firstname || ' ' || mems.surname AS member,
		facs.name AS facility,
		CASE
			WHEN mems.memid = 0 then
				bks.slots*facs.guestcost
			ELSE
				bks.slots*facs.membercost
		END AS cost
		FROM
			cd.members mems
			INNER JOIN cd.bookings bks
				ON mems.memid = bks.memid
			INNER JOIN cd.facilities facs
				ON bks.facid = facs.facid
		WHERE
			bks.starttime >= '2012-09-14' AND
			bks.starttime < '2012-09-15'
	) AS bookings
	WHERE cost > 30
ORDER BY cost desc;  
```

<br>

# Modifying data 

Exercise 3
	
1. The club is adding a new facility - a spa. We need to add it into the facilities table. \
Use the following values:\
facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.
```sql
INSERT INTO cd.facilities
(facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
VALUES
(9, 'Spa', 20, 30, 100000, 800);
```
OR 
	
```sql
INSERT INTO cd.facilities VALUES (9, 'Spa', 20, 30, 100000, 800);
```
	
2. In the previous exercise, you learned how to add a facility. Now you're going to add multiple facilities in one command. Use the following values:\
facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.\
facid: 10, Name: 'Squash Court 2', membercost: 3.5, guestcost: 17.5, initialoutlay: 5000, monthlymaintenance: 80.
```sql
INSERT INTO cd.facilities 
(facid, Name, membercost, guestcost, initialoutlay, monthlymaintenance)
VALUES
(9, 'Spa', 20, 30, 100000, 800),
(10, 'Squash Court 2', 3.5, 17.5, 5000, 80); 
```

3. Let's try adding the spa to the facilities table again. \
This time, though, we want to automatically generate the value for the next facid, rather than specifying it as a constant.\ 
Use the following values for everything else: \
Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.	
```sql
INSERT INTO cd.facilities 
(facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
SELECT
(SELECT MAX(facid) FROM cd.facilities) + 1,'Spa', 20, 30, 100000, 800; 
```

4. We made a mistake when entering the data for the second tennis court.\ 
The initial outlay was 10000 rather than 8000: you need to alter the data to fix the error.	
```sql
UPDATE cd.facilities
SET initialoutlay = 10000
WHERE facid = 1;
```

5. We want to increase the price of the tennis courts for both members and guests. \
Update the costs to be 6 for members, and 30 for guests.
```sql

```
```sql

```

```sql

```
	
