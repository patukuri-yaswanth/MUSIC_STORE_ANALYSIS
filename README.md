# SQL_MUSIC_STORE_ANALYSIS
__1Q- Who is the senior most employee based on job title?__

select * from employee order by levels desc limit 1;
   
# 2Q- Which country have the most invoices?

  select billing_country,count(*) as c 
  from invoice 
  group by billing_country 
  order by c desc
  Limit 1

# 3Q- What are top 3 Values of total invoice?
	
select customer_id,total from invoice 
order by total desc 
limit 3

__4Q Which city has the best cutomers? We would like to throw a promotional Music Festival in the city 
   we made the most money. Write a query  that returns one city that has the highest sum of invoice 
   totals.Return both the city name & sum of all invoice totals?__

select billing_city,sum(total) as invoice_total 
from invoice
group by billing_city
order by invoice_total desc

__5Q-Who is the best customer?The customer who has spent the most money will be declared the best customer.
   Write a query that returns the person who has spent the most money?__

select customer.customer_id,customer.first_name,customer.last_name,sum(invoice.total) as total
from customer 
inner join invoice on customer.customer_id=invoice.customer_id 
group by customer.customer_id
order by total Desc
limit 1


__6Q- Write query to return the email,first name,last name,and genre of all rock music listners.
	return your list ordered alphabetically by email staring with A?__

select Distinct email,first_name,last_name
from customer
join invoice on customer.customer_id = invoice.customer_id
join invoice_line on invoice.invoice_id=invoice_line.invoice_id
where track_id in(
	Select track_id from track
	join genre on track.genre_id=genre.genre_id
	where genre.name Like 'Rock'
)
order by email;	

__7Q- Lets invite the artists who have written the most rock music in our dataset.Write a query that 
	return the Artist name and total track count of the top 10 rock bands__

select artist.artist_id,artist.name,Count(artist.artist_id) As number_of_songs
from track
join album on album.album_id=track.album_id
join artist on artist.artist_id=album.artist_id
join genre on genre.genre_id = track.genre_id
where genre.name like 'Rock'
group by artist.artist_id
order by number_of_songs Desc
Limit 10

__8Q- Return all the track names that have a song length longer than average song length.
	return the name and milliseconds for each track.Order by the song length with the 
    longest songs listed first.__
	
select name,milliseconds
from track
where milliseconds > (select avg(milliseconds) as length_of_the_track
	from track)
order by milliseconds Desc;

__9Q- Find how much amount spent by each customer on artists? write a query to return customer name,
    artist name and total spent?__

with best_selling_artist As(
	select artist.artist_id as artist_id,artist.name as artist_name,sum(invoice_line.unit_price+invoice_line.quantity) as rowno
	from invoice_line
	join track on track.track_id = invoice_line.track_id
	join album on album.album_id = track.album_id
	join artist on artist.artist_id = album.artist_id
	group by 1
	order by 3 Desc
	limit 1
)
select c.customer_id,c.first_name,c.last_name,bsa.artist_name,sum(il.unit_price*il.quantity) as amount_spent
from invoice i
join customer c on c.customer_id = i.customer_id
join invoice_line il on il.invoice_id = i.invoice_id
join track t on t.track_id =il.track_id
join album alb on alb.album_id = t.album_id
join best_selling_artist bsa on bsa.artist_id = alb.artist_id
group by 1,2,3,4
order by 5 desc;
	
__10Q- We want to find out the most popular music Genre for each country.We determine the most popular
     as the genre with the highest amount of purchases.Write a query that returns each country along
     with the top Genre.For countries where the maximum number of purchases is shared return all Genres?__

with popular_genre as
(
	select count(invoice_line.quantity) as purchases,customer.country,genre.name,genre.genre_id,
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity)Desc) as rowno
	from invoice_line
	join invoice on invoice.invoice_id = invoice_line.invoice_id
	join customer on customer.customer_id = invoice.customer_id
	join track on track.track_id = invoice_line.track_id
	join genre on genre.genre_id = track.genre_id
	Group By 2,3,4
	Order By 2 ASC,1 desc
)
SELECT * from popular_genre where Rowno <=1

__11Q- Write a query that determines the customer that has spent the most on music for each country.
	 Write a query that returns the country along with the top customer and how much they spent.For
     Countries where the top amount spent is shared,provide all custsomers who spent this amount?__
with customer_with_country As(
	SELECT customer.customer_id,first_name,last_name,billing_country,sum(total) as total_spending,
	row_number() over(partition by billing_country Order by sum(total) DESC) As RowNo
	   From invoice
	   join customer on customer.customer_id = invoice.customer_id
	   group by 1,2,3,4
	   Order by 4 asc,5 desc)
select * from customer_with_country where RowNo <=1







