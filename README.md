# Sql-Data-Analysis-Music-Project
Mastered SQL complexities: Window Functions, CASE Statements, CTEs, DDL/DML commands. Executed diverse queries for robust data analysis and manipulation.
# Question Set 1-Easy
set Sql_SAFE_UPDATES=0;
# Who is the Most Senior Employee based on the job Title?
select * FROM eMPLOYEE;
select employee_id,concat(First_name," ",Last_Name) as Full_Name,Title,levels 
from employee
ORDER BY LEVELS desc limit 1 ;

# which country has the most invoice?
select* from invoice;
select billing_country,count(billing_country) as No_Of_Invoices from invoice
group by billing_country order by NO_OF_iNVOICES desc;

# What are top 3 values of Total Invoices?
select* from invoice;
select invoice_id,customer_id, total as Total_Invoice
from invoice
order by total desc limit 3;


/*
Which city has the best customers? We would like to throw a promotional Music
Festival in the city we made the most money. Write a query that returns one city that
has the highest sum of invoice totals. Return both the city name & sum of all invoice
totals
*/
select * from invoice;
select billing_country as Country,billing_city as City 
,round(max(total),2) as Highest_Invoice from invoice
group by billing_City,billing_country 
order by Highest_Invoice desc limit 1;

/*
Who is the best customer? The customer who has spent the most money will be
declared the best customer. Write a query that returns the person who has spent the
most money
*/
select * from customer;
select * from invoice;

select c.customer_id as Customer_ID,concat(c.first_name," ",c.last_name) as Full_Name,
c.phone as Phone_Number ,c.email as Email ,round(max(i.total),2) as Invoice 
from customer c
inner join invoice i on c.customer_id=i.customer_id
group by c.customer_id,c.first_name,c.last_name,c.phone,c.email 
order by max(i.total) desc limit 1;

# Question Set 2-Moderate

/*Write query to return the email, first name, last name, & Genre of all Rock Music 
listeners. Return your list ordered alphabetically by email starting with A */
select * from customer;
select * from genre;


select distinct c.first_name,c.last_name,c.email, g.name
from customer c
inner join invoice i on c.customer_id=i.customer_id
inner join invoice_line il on i.invoice_id=il.invoice_id
inner join track t on il.track_id=t.track_id
inner join genre g on t.genre_id=g.genre_id 
where g.name="Rock" order by c.email;

/*Let's invite the artists who have written the most rock music in our dataset. Write a 
query that returns the Artist name and total track count of the top 10 rock bands
*/

select * from artist;
select * from album;
select * from track;
select * from genre;



select 
ar.artist_id,
count( ar.artist_id) as Number_of_Songs,
ar.name as Artist_Name,
ge.name as Genre
from artist ar

inner join album al on al.artist_id=ar.artist_id
inner join track tr on al.album_id=tr.album_id
inner join  genre ge on tr.genre_id=ge.genre_id

group by 
ge.name,
ar.name,
ar.artist_id

having ge.name="Rock" 
order by Number_of_Songs desc 
limit 10;


/*
Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the 
longest songs listed first
*/
select * from track;


select track_id,name,milliseconds
from track 
where milliseconds> (select avg(milliseconds) from track)
order by milliseconds desc;


# Question Set 3-Advance

/*
Find how much amount spent by each customer on artists? Write a query to return
customer name, artist name and total spent
*/
select * from customer;
select * from invoice;
select * from invoice_line;
select * from track;


select distinct concat(c.First_Name," ",c.last_name) as Customer_Name,t.composer as Artist ,round(sum(i.total),2) as Amount_Spent
from customer c
join invoice i on c.customer_id=i.customer_id
join invoice_line il on  i.invoice_id=il.invoice_id
join track t on il.track_id=t.track_id
group by Customer_Name,Artist order by customer_name asc;

/*
We want to find out the most popular music Genre for each country. We determine the 
most popular genre as the genre with the highest amount of purchases. Write a query 
that returns each country along with the top Genre. For countries where the maximum 
number of purchases is shared return all Genres
*/
select * from customer;
select * from invoice;
select * from invoice_line;
select * from track;
select * from genre;

WITH RankedGenres as (
	select  i.billing_country as Country , G.name as Genre,round(sum(i.total),2) as Amount_Spend,
	ROW_NUMBER() OVER (PARTITION BY i.billing_country ORDER BY SUM(i.total) DESC) AS Number
    from customer c
	join invoice i on c.customer_id=i.customer_id
	join invoice_line il on i.invoice_id=il.invoice_id
	join track t on il.track_id=t.track_id
	join genre g on t.genre_id=g.genre_id
	group by i.billing_country,G.name
)
select  Country,Genre,Amount_Spend, number
from RankedGenres
where  Number=1
ORDER BY Amount_Spend DESC;

/*
Write a query that determines the customer that has spent the most on music for each 
country. Write a query that returns the country along with the top customer and how
much they spent. For countries where the top amount spent is shared, provide all 
customers who spent this amount
*/

select * from customer;
select * from invoice;


with RankedCustomer as (
	select concat(c.First_Name," ",c.Last_Name) as Customer_Name,c.COUNTRY AS State,
    C.Email as Customer_Email,round(Sum(i.Total),2) as Amount_Spend,
    ROW_NUMBER() OVER (PARTITION BY c.COUNTRY ORDER BY SUM(i.total) DESC) AS Number
    from customer c
	inner join invoice i on c.customer_id=i.customer_id
    group by Customer_Name,c.COUNTRY,C.Email

)
select  Customer_Name,State,Amount_Spend
from RankedCustomer
where  Number=1
ORDER BY State ;



