<h3 align="center"> All of my answers in the PostgreSQL Exercises <hr>

Click [Here](https://pgexercises.com/), If you want to improve your PostgreSQL skills and acquire more experience with difficult queries  <hr>

We'll look at the following chapters

:heavy_check_mark: Simple SQL Queries\
:heavy_check_mark: Joins and Subqueries\
:heavy_check_mark: Modifying data\
:heavy_check_mark: Aggregation \
:heavy_check_mark: Working with Timestamps \
:heavy_check_mark: String Operations \
:heavy_check_mark: Recursive Queries 
	
<br>
	
# Chapter 1:   Simple SQL Queries
 

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
SELECT * FROM cd.facilities WHERE facid IN (1,5);
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
	
10. You, for some reason, want a combined list of all surnames and all facility names. \
Yes, this is a contrived example :-).
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
	
# Chapter 2: Joins and Subqueries

	
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
WHERE fcts.facid in (0,1) AND
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
			       
4. How can you output a list of all members, including the individual who recommended them (if any)? \
Ensure that results are ordered by (surname, firstname).
```sql
SELECT mems.firstname AS memfname, mems.surname AS memsname, 
rec.firstname AS recfname, rec.surname AS recsname
FROM cd.members mems
LEFT OUTER JOIN cd.members rec ON 
rec.memid = mems.recommendedby
ORDER BY memsname, memfname;
```
			       
5. How can you produce a list of all members who have used a tennis court? \
Include in your output the name of the court, and the name of the member formatted as a single column.\
Ensure no duplicate data, and order by the member name followed by the facility name. 
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
	
	(
	SELECT recs.firstname || ' ' || recs.surname as recommender 
		
	 FROM cd.members recs 
		WHERE recs.memid = mems.recommendedby
	)
	FROM 
		cd.members mems

ORDER BY member; 
	
```
8. The Produce a list of costly bookings exercise contained some messy logic: we had to calculate the booking cost in both the WHERE clause and the CASE statement. \
Try to simplify this calculation using subqueries. For reference, the question was:\
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

# Chapter 3: Modifying data 

	
1. The club is adding a new facility - a spa. We need to add it into the facilities table. \
Use the following values:\
```facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800```
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
	
2. In the previous exercise, you learned how to add a facility. Now you're going to add multiple facilities in one command. Use the following values:

```facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800```\
```facid: 10, Name: 'Squash Court 2', membercost: 3.5, guestcost: 17.5, initialoutlay: 5000, monthlymaintenance: 80```
	
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

```Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800```
	
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
UPDATE cd.facilities
    SET
        membercost = 6,
        guestcost = 30
    WHERE facid IN (0,1);
```

6.We want to alter the price of the second tennis court so that it costs 10% more than the first one. \
Try to do this without using constant values for the prices, so that we can reuse the statement if we want to.	
```sql
UPDATE cd.facilities fct
SET 
    membercost = (SELECT membercost * 1.1 from cd.facilities where facid = 0),
    guestcost = (SELECT guestcost * 1.1 from cd.facilities where facid = 0)
WHERE fct.facid = 1;
```
	
7.As part of a clearout of our database, we want to delete all bookings from the cd.bookings table. \
How can we accomplish this? 
```sql
DELETE FROM cd.bookings;
```
	
OR 
	
```sql
TRUNCATE cd.bookings;
```
	
8. We want to remove member 37, who has never made a booking, from our database. How can we achieve that?
```sql
DELETE FROM cd.members WHERE memid = 37;	
```

9. In our previous exercises, we deleted a specific member who had never made a booking. \
How can we make that more general, to delete all members who have never made a booking?
```sql
DELETE FROM cd.members 
WHERE memid NOT IN (SELECT memid FROM cd.bookings);
```

<br>
	
# Chapter 4: Aggregation
	
1. For our first foray into aggregates, we're going to stick to something simple. \
We want to know how many facilities exist - simply produce a total count.
```sql
SELECT COUNT(*) AS count FROM cd.facilities;
```
	
2. Produce a count of the number of facilities that have a cost to guests of 10 or more.	
```sql
SELECT COUNT(*) FROM cd.facilities WHERE guestcost >= 10;
```
	
3. Produce a count of the number of recommendations each member has made. Order by member ID.	
```sql
SELECT recommendedby, COUNT(*) FROM cd.members
WHERE recommendedby IS NOT NULL
GROUP BY recommendedby
ORDER BY recommendedby;
```
	
4. Produce a list of the total number of slots booked per facility. \
For now, just produce an output table consisting of facility id and slots, sorted by facility id.	
```sql
SELECT facid, SUM(slots) AS "Total Slots" FROM cd.bookings
GROUP BY facid
ORDER BY facid;
```
	
5. Produce a list of the total number of slots booked per facility in the month of September 2012.\
Produce an output table consisting of facility id and slots, sorted by the number of slots.	
```sql
SELECT facid, SUM(slots) AS "Total Slots" FROM cd.bookings
WHERE starttime >= '2012-09-01' AND  starttime < '2012-10-1'
GROUP BY facid
ORDER BY SUM(slots);
```
6. Produce a list of the total number of slots booked per facility per month in the year of 2012. \ 
Produce an output table consisting of facility id and slots, sorted by the id and month.	
```sql
SELECT facid, EXTRACT (month from starttime) AS month, SUM(slots) AS "Total Slots" FROM cd.bookings
WHERE EXTRACT (year from starttime ) = 2012
GROUP BY facid, month
ORDER BY facid, month;
```
```						      
EXTRACT allows you to get individual components of a timestamp, like day, month, year, etc.\ 
We group by the output of this function to provide per-month values.		
```
						      
7. Find the total number of members (including guests) who have made at least one booking.	
```sql
SELECT COUNT (DISTINCT memid) FROM cd.bookings
WHERE slots >= 1;
```

8. Produce a list of facilities with more than 1000 slots booked. \
Produce an output table consisting of facility id and slots, sorted by facility id.
```sql
SELECT facid, SUM(slots) AS "Total Slots" FROM cd.bookings
GROUP BY facid
HAVING SUM(slots) > 1000
ORDER BY facid;
```
	
```
The behaviour of HAVING is easily confused with that of WHERE.\
The best way to think about it is that in the context of a query with an aggregate function, WHERE is used to filter what data gets input into the aggregate function,\
while HAVING is used to filter the data once it is output from the function.	
```
	
9.Produce a list of facilities along with their total revenue. \ 
The output table should consist of facility name and revenue, sorted by revenue. \ 
Remember that there's a different cost for guests and members!
```sql
SELECT fct.name, SUM ( slots * CASE 
			WHEN memid = 0 THEN fct.guestcost
			ELSE fct.membercost 
			END) AS revenue 
		
		FROM cd.bookings bks
		INNER JOIN cd.facilities fct ON 
		bks.facid = fct.facid
		
		GROUP BY fct.name
		ORDER BY revenue;
```
	
10.Produce a list of facilities with a total revenue less than 1000.\
Produce an output table consisting of facility name and revenue, sorted by revenue.\
Remember that there's a different cost for guests and members!
```sql
SELECT fct.name , SUM( slots * CASE 
			WHEN memid = 0 THEN fct.guestcost
			ELSE fct.membercost 
			END) AS revenue
			
			FROM cd.bookings bks 
			INNER JOIN cd.facilities fct ON
			bks.facid = fct.facid
			GROUP BY fct.name
			HAVING SUM( slots * CASE 
			WHEN memid = 0 THEN fct.guestcost
			ELSE fct.membercost 
			END) < 1000
			ORDER BY revenue;
```
	
11. Output the facility id that has the highest number of slots booked.\
For bonus points, try a version without a LIMIT clause. This version will probably look messy!	
```sql
SELECT facid, SUM(slots) AS "Total Slots" FROM cd.bookings
GROUP BY facid
ORDER BY SUM(slots) DESC
LIMIT 1;
```
				   
12.Produce a list of the total number of slots booked per facility per month in the year of 2012. \
In this version, include output rows containing totals for all months per facility, and a total for all months for all facilities. \
The output table should consist of facility id, month and slots, sorted by the id and month. \
When calculating the aggregated values for all months and all facids, return null values in the month and facid columns.				   
```sql
SELECT facid, EXTRACT (month FROM starttime) AS month, SUM(slots) AS "slots" FROM cd.bookings
WHERE ( starttime >= '2012-01-01' AND starttime < '2013-01-01' )
GROUP BY ROLLUP(facid, month)
ORDER BY facid, month;
```

13.Produce a list of the total number of hours booked per facility, remembering that a slot lasts half an hour.\
The output table should consist of the facility id, name, and hours booked, sorted by facility id. \
Try formatting the hours to two decimal places.		

```One of the taughest question so far```						
```sql
SELECT fct.facid, fct.name, trim(to_char(sum(bks.slots)/2.0, '9999999999999999D99')) as "Total Hours"  FROM cd.bookings bks
INNER JOIN cd.facilities fct ON 
fct.facid = bks.facid
GROUP BY fct.facid, fct.name
ORDER BY fct.facid;
```
14.Produce a list of each member name, id, and their first booking after September 1st 2012. Order by member ID.	
```sql
SELECT mems.surname, mems.firstname,mems.memid, MIN(starttime) FROM cd.members mems
INNER JOIN cd.bookings bks ON 
mems.memid = bks.memid 
WHERE starttime > '2012-09-01'
GROUP BY mems.surname, mems.firstname,mems.memid
ORDER BY memid;
```

15. Produce a list of member names, with each row containing the total member count. Order by join date, and include guest members.
```sql
SELECT (SELECT COUNT(*) FROM cd.members), firstname, surname FROM cd.members mems
GROUP BY mems.firstname, mems.surname, mems.joindate
ORDER BY joindate;
```
	
16.Produce a monotonically increasing numbered list of members (including guests), ordered by their date of joining.\
Remember that member IDs are not guaranteed to be sequential.
```sql
SELECT row_number() OVER (ORDER BY joindate), firstname, surname
FROM cd.members ORDER BY joindate;	
```
OR
```sql
SELECT COUNT(*) OVER (ORDER BY joindate) AS row_number, firstname, surname FROM cd.members;
```
	
17.Output the facility id that has the highest number of slots booked.\
Ensure that in the event of a tie, all tieing results get output.	
```sql
SELECT facid, total FROM (
  
  SELECT facid, SUM(slots) AS total,
         rank() OVER (ORDER BY SUM(slots) DESC) AS rank
  
  FROM cd.bookings
  GROUP BY facid
) AS ranked WHERE rank = 1	
```

18. Produce a list of members (including guests), along with the number of hours they've booked in facilities, rounded to the nearest ten hours.\
Rank them by this rounded figure, producing output of first name, surname, rounded hours, rank.\
Sort by rank, surname, and first name.	
```sql
SELECT firstname, surname, hours, rank () OVER (ORDER BY hours DESC) FROM 

(SELECT firstname, surname,
		( ( SUM(bks.slots) + 10 ) / 20 ) * 10 AS hours

		FROM cd.bookings bks
		
 			INNER JOIN cd.members mems
			ON bks.memid = mems.memid
		GROUP BY mems.memid
	) AS subq
ORDER BY rank, surname, firstname;	
```

19.Produce a list of the top three revenue generating facilities (including ties).\
Output facility name and rank, sorted by rank and facility name.	
```sql
SELECT name, rank FROM (
  
  SELECT fct.name AS name, RANK () OVER (ORDER BY SUM( CASE 
		
		WHEN memid = 0 THEN slots * fct.guestcost
  		ELSE slots * fct.membercost 
		END ) 
		DESC ) AS rank 

FROM cd.bookings bks 

INNER JOIN cd.facilities fct ON 
bks.facid = fct.facid 

GROUP BY fct.name) AS subq
WHERE rank <= 3
ORDER BY rank;					
```

20.Classify facilities into equally sized groups of high, average, and low based on their revenue.\
Order by classification and facility name.	
```sql
	
```
