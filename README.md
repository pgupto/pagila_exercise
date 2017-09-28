# pagila_exercise
# Pagila Exercises 

### Displaying Names
- Display list of all the actors’  first name and last name
- Display the first and last name of each actor in a single column in upper case letters. N
```
-- 1a and 1b. Using CONCAT_WS function to get the full names of customers in one column:
SELECT
	concat_ws (
	', ',
	UPPER ( last_name ),
	UPPER ( first_name )) AS actor_name 
FROM
	actor 
ORDER BY
	last_name;
SELECT
	first_name,
	last_name 
FROM
	actor 
ORDER BY
	first_name;
```

### Lookup for Names
```
-- 2a. Finding Joe in the actor list
SELECT
	actor_id,
	first_name,
	last_name 
FROM
	actor 
WHERE
	first_name LIKE '%JOE%';
	
-- 2b. Finding actors with GEN in their last names
SELECT
	actor_id,
	concat (
	LOWER ( last_name )) "GEN in last name of the actor" 
FROM
	actor 
WHERE
	last_name LIKE '%GEN%';
	
-- 2c. Finding actors with LI in their last names (using iLIKE function to make it case insensitive)
SELECT
	actor_id,
	concat (
	LOWER ( last_name )) "LI in last name of the actor",
	first_name 
FROM
	actor 
WHERE
	last_name ILIKE '%LI%';
	
-- 2d. Displaying countries 

SELECT
	country_id,
	country 
FROM
	country 
WHERE
	country IN ( 'China', 'Bangladesh', 'Afghanistan' ) 
ORDER BY
	country;
```
### Editing Name
```
-- Adding middle_name column 
SELECT * FROM actor;
ALTER TABLE actor
ADD COLUMN middle_name CHARACTER VARYING (20);

-- Removing/Deleting the column middle_name that we just created  
ALTER TABLE actor
DROP COLUMN middle_name;
```
### Creating List
```
-- 4a actors have that last name	
SELECT
	last_name 
FROM
	actor 
ORDER BY
	last_name;
	
-- 4b names that are shared by at least two actors
SELECT COUNT
	( last_name ),
	last_name 
FROM
	actor 
GROUP BY
	last_name 
HAVING
	( COUNT ( last_name ) = 2 );
	
-- 4c Changing the name to GROUCHO WILLIAMS
SELECT
	first_name, last_name, actor_id 
FROM
	actor 
	ORDER BY
	last_name;
	
UPDATE actor
SET first_name = 'GROUCHO', last_name = 'WILLIAMS'
WHERE actor_id= 172;
```
### Joins
```
-- 6a. The first and last names, as well as the address, of each staff member.
SELECT
	st.first_name,
	st.last_name,
	ad.address 
FROM
	staff st
	LEFT JOIN address ad ON st.address_id = ad.address_id;
	
-- 6b. Displaying the total amount rung up by each staff member in January of 2007.
SELECT
	st.staff_id,
	st.first_name,
	st.last_name,
	SUM ( payment.amount ) 
FROM
	staff st
	LEFT JOIN payment ON st.staff_id = payment.staff_id 
WHERE
	CAST ( payment_date AS DATE ) < '2007-02-01' 
GROUP BY
	st.staff_id;
	
-- 6c. List each film and the number of actors who are listed for that film. Use tables film_actor and film. Use inner join.
SELECT
	f.film_id,
	f.title,
	COUNT ( af.actor_id ) AS actor_count 
FROM
	film f
	LEFT JOIN film_actor af ON f.film_id = af.film_id 
GROUP BY
	f.film_id;
	
-- 6d. Copies of the film Hunchback Impossible existing in the inventory system
SELECT
	i.film_id,
	f.title,
	COUNT ( i.inventory_id ) AS movie_copies_in_stock
	
FROM
	inventory i
	LEFT JOIN film f ON f.title= 'HUNCHBACK IMPOSSIBLE'
	WHERE i.film_id = f.film_id
	GROUP BY f.film_id, i.film_id;

-- 6e.list the total amount paid by each customer. List the customers alphabetically by last name

SELECT
	cs.last_name,
	cs.first_name,
	SUM ( payment.amount ) AS total_amount_paid
FROM
	customer cs
	
LEFT JOIN payment ON cs.customer_id = payment.customer_id

GROUP BY
	cs.last_name, cs.first_name 
ORDER BY cs.last_name;
```
### Subqueries
```
-- 7a. Display the titles of movies starting with the letters K and Q whose language is English.
SELECT
	film.film_id,
	film.title,
	LANGUAGE.NAME 
FROM
	film
	LEFT JOIN LANGUAGE ON film.language_id = LANGUAGE.language_id 
WHERE
	LANGUAGE.NAME = 'English' 
	AND (
	film.title ILIKE 'k%' 
	OR film.title ILIKE 'q%' 
	);
	
-- 7b. Actors who appear in the film Alone Trip
SELECT
	actor.first_name,
	actor.last_name 
FROM
	actor 
WHERE
	actor_id IN (
SELECT
	film_actor.actor_id 
FROM
	film_actor
	INNER JOIN film ON film_actor.film_id = film.film_id 
WHERE
	film.title ILIKE 'ALONE TRIP' 
	) 
ORDER BY
	actor.first_name;

-- 7c. Names and email addresses of all Canadian customers
Select first_name, last_name,email,address
FROM
( 
	SELECT cu.first_name, cu.last_name, cu.email, a.address
	FROM customer cu
	LEFT JOIN country c
	ON c.country = 'Canada' 
	LEFT JOIN address a
	ON a.address_id = cu.address_id
	LEFT JOIN city ci
	ON a.city_id = ci.city_id
	WHERE
	ci.country_id = c.country_id
) cd
;

-- 7d. Identify all movies categorized as a family film
SELECT name as category, title
FROM
( 
	SELECT c.name, f.title, f.film_id
	FROM film f
	LEFT JOIN category c
	ON c.name ilike 'Family%'
	LEFT JOIN film_category fc
	ON fc.category_id = c.category_id
	WHERE
	fc.film_id = f.film_id
) fm
GROUP BY fm.name, fm.title, fm.film_id;

-- 7e. Most frequently rented movies in descending order

SELECT title, count(rental_id)
FROM
( 
	SELECT f.title, r.rental_id
	FROM film f
	LEFT JOIN inventory i
	ON i.film_id = f.film_id
	LEFT JOIN rental r
	ON r.inventory_id = i.inventory_id
) frm
GROUP BY frm.title
ORDER BY
frm.count DESC;

-- 7f. Business per store
Select store_id, cast(sum(amount) as money)
FROM
( 
	SELECT s.store_id, p.amount
	FROM store s
	LEFT JOIN customer c
	ON c.store_id = s.store_id
	LEFT JOIN payment p
	ON p.customer_id = c.customer_id
) st
GROUP BY st.store_id;
```

### Views

```
-- 8a,b,c. Creating view, displaying and deleting it
CREATE VIEW top_five_categories AS 
SELECT category.NAME,
SUM ( payment.amount ) 
FROM
	rental
	INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id
	INNER JOIN payment ON rental.rental_id = payment.rental_id
	INNER JOIN film ON inventory.film_id = film.film_id
	INNER JOIN film_category ON film.film_id = film_category.film_id
	INNER JOIN category ON film_category.category_id = category.category_id 
GROUP BY
	category.NAME 
ORDER BY
	SUM ( payment.amount ) DESC 
	LIMIT 5;
SELECT
	* 
FROM
	top_five_categories;
DROP VIEW top_five_categories;
```

