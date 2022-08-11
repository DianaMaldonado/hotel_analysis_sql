Hello!

This is a hotel data analysis with SQL. It is inspired in this tutorial: https://www.youtube.com/watch?v=S2zBHmkRbhY but added more scopes of analyisis.
The questions that the analysis seeks to answer are the following:
1. Is the hotel revenue grwoing by year?
2. Should the hotel invest in increasing the parking spaces number?
3. Which months, weeks, and days of month have more guests?
4. Which months have more week and weekends stays?
5. Which month have more guests classified for guest type (adults, children, and babies)?
6. How much revenue in total and average is generated according to guests?
7. Which year and month had the biggest average daily rate and average number of guests?
8. What meal type is more consumed and how much is the total and average cost of each one?
9. From which country come the guests?
10. From how many countries do our guests come?
11. Which market segment provides more guests and revenue?
12. Which distribution channel provides more guests and more revenue?
13. What is the number, nights stayed, revenue, and average revenue for recurrent and non recurrent guests?
14. Are all the guests ssigned to the same room type that they reserved?
15. Which room type is the most demanded?

To approach this analysis I worked with MySQL, importing each table from the file and then union and joining them as required.
![Captura de Pantalla 2022-08-11 a la(s) 12 36 06](https://user-images.githubusercontent.com/74566268/184197448-718ac0c5-dac5-48eb-93e9-bd8b84563517.png)


Below the code :)

/* We create a new table from the union of the 3 tables of years */
CREATE TABLE unions
SELECT *
FROM data_2018
UNION
SELECT *
FROM data_2019
UNION
SELECT *
FROM data_2020;

/* We join the tables unions, market_segment, and meal_cost to perform further analysis */
CREATE TABLE hotels
SELECT
	u.*,
	m.Discount,
    me.Cost
FROM unions AS u
	INNER JOIN market_segment AS m
			ON u.market_segment = m.market_segment
	INNER JOIN meal_cost AS me
			ON u.meal = me.meal;

/* To know the revenue by year */
SELECT 
	arrival_date_year,
	ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue -- here we first sum the stays in weekdays and weekends, and then multiply them for the adr which is the fee per night
FROM hotels
GROUP BY arrival_date_year
ORDER BY revenue DESC;

/* In the last query we see that 2019 is the year with more revenue. It is important to confirm if all years have the same months. To know if all years have the same months */
SELECT 
	arrival_date_year,
    arrival_date_month
FROM hotels
GROUP BY 
	arrival_date_year,
    arrival_date_month
ORDER BY
	arrival_date_month;
    
/* Since not all years have the same months we cannot compare the aggregate of year. However, we can compare the month all years have in common*/
SELECT
	arrival_date_year,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue
FROM hotels
WHERE arrival_date_month = 'August'
GROUP BY arrival_date_year
ORDER BY revenue DESC;

/* Also we can compare 2019 to 2018 in their second semester and 2019 and 2020 from January to August*/
SELECT -- This query is for comparing 2018 and 2019 in their second semester
	arrival_date_year,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue
FROM hotels
WHERE arrival_date_month IN ('July', 'August', 'September', 'October', 'November', 'December') AND arrival_date_year IN (2018, 2019)
GROUP BY 
	arrival_date_year
ORDER BY 
    revenue DESC;
    
SELECT -- This query is for comparing 2019 AND 2020 from January to August
	arrival_date_year,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue
FROM hotels
WHERE arrival_date_month IN ('January', 'February', 'March', 'April', 'May', 'June', 'July', 'August') AND arrival_date_year IN (2019, 2020)
GROUP BY 
	arrival_date_year
ORDER BY 
	revenue DESC;

/* Now it is important to segment the results of the previous 2 queries by hotel */
SELECT -- First, we check which hotel has more demand
	hotel, 
    COUNT(hotel) AS records
FROM hotels
GROUP BY hotel
ORDER BY records DESC;

SELECT -- Second, we see which hotel generated more revenue each year (comparing the same months for 2018 and 2019)
	arrival_date_year,
    hotel,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue
FROM hotels
WHERE arrival_date_month IN ('July', 'August', 'September', 'October', 'November', 'December') AND arrival_date_year IN (2018, 2019)
GROUP BY 
	arrival_date_year,
    hotel
ORDER BY 
    revenue DESC;

SELECT -- Finally, we compare which hotel generated more revenue each year (comparing the same months for 2019 and 2020)
	arrival_date_year,
    hotel,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue
FROM hotels
WHERE arrival_date_month IN ('January', 'February', 'March', 'April', 'May', 'June', 'July', 'August') AND arrival_date_year IN (2019, 2020)
GROUP BY 
	arrival_date_year,
    hotel
ORDER BY 
	revenue DESC;

/* To know if the hotels should increase their parking lots*/
SELECT -- This query shows how many parking lots were required by year and hotel (comparing same months for 2018 and 2019)
	arrival_date_year,
    hotel,
    SUM(required_car_parking_spaces) AS parking_spaces
FROM hotels
WHERE arrival_date_month IN ('July', 'August', 'September', 'October', 'November', 'December') AND arrival_date_year IN (2018, 2019)
GROUP BY
	arrival_date_year,
    hotel;
  
SELECT -- This query shows how many parking lots were required by year and hotel (comparing same months for 2019 and 2020)
	arrival_date_year,
    hotel,
    SUM(required_car_parking_spaces) AS parking_spaces
FROM hotels
WHERE arrival_date_month IN ('January', 'February', 'March', 'April', 'May', 'June', 'July', 'August') AND arrival_date_year IN (2019, 2020)
GROUP BY
	arrival_date_year,
    hotel;

/*The following queries show the months, week number, and day of month with more guests*/
SELECT -- Months with more guests
	arrival_date_month,
    SUM(adults + children + babies) AS total_guests
FROM hotels
GROUP BY arrival_date_month
ORDER BY total_guests DESC;

SELECT -- Week number of the year with more guests
	arrival_date_week_number,
    SUM(adults + children + babies) AS total_guests
FROM hotels
GROUP BY arrival_date_week_number
ORDER BY total_guests DESC
LIMIT 10;

SELECT -- Day of the month with more guests
	arrival_date_day_of_month,
    SUM(adults + children + babies) AS total_guests
FROM hotels
GROUP BY arrival_date_day_of_month
ORDER BY total_guests DESC;

/* Months with more weekend and week stays */
SELECT
CASE WHEN arrival_date_month = 'January' THEN 1
	WHEN arrival_date_month = 'February' THEN 2
    WHEN arrival_date_month = 'March' THEN 3
    WHEN arrival_date_month = 'April' THEN 4
    WHEN arrival_date_month = 'May' THEN 5
    WHEN arrival_date_month = 'June' THEN 6
    WHEN arrival_date_month = 'July' THEN 7
    WHEN arrival_date_month = 'August' THEN 8
    WHEN arrival_date_month = 'September' THEN 9
    WHEN arrival_date_month = 'October' THEN 10
	WHEN arrival_date_month = 'November' THEN 11
    ELSE 12 END AS arrival_date_month_number,
	SUM(stays_in_weekend_nights) AS weekend_stays,
    SUM(stays_in_week_nights) AS week_stays
FROM hotels
GROUP BY arrival_date_month
ORDER BY arrival_date_month_number ASC;

/*Months with more guests classified by adults, children, and babies*/
SELECT
	arrival_date_month,
    SUM(adults) AS adults_guests,
    SUM(children) AS children_guests,
    SUM(babies) AS babies_guests
FROM hotels
GROUP BY arrival_date_month
ORDER BY adults_guests DESC;

/*In the following queries it is compared how much revenue as total and average it is per only adults, adults with children, and adults with children and babies*/
CREATE TABLE only_adults_revenue -- This query creates a table that identifies revenue for only adults
SELECT
	arrival_date_month,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS adults_total_revenue,
    ROUND(AVG((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS adults_avg_revenue
FROM hotels
WHERE adults >= 1 AND children = 0 AND babies = 0
GROUP BY 
	arrival_date_month;

CREATE TABLE child_revenue -- This query creates a table that identifies revenue for adults with children (no babies)
SELECT
	arrival_date_month,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS child_total_revenue,
    ROUND(AVG((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS child_avg_revenue
FROM hotels
WHERE adults >= 1 AND children >=1 AND babies = 0
GROUP BY 
	arrival_date_month;
    
CREATE TABLE babies_only_revenue -- This query creates a table that identifies revenue for adults with babies (no children)
SELECT
	arrival_date_month,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS babies_total_revenue,
    ROUND(AVG((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS babies_avg_revenue
FROM hotels
WHERE adults >= 1 AND children =0 AND babies >= 1
GROUP BY 
	arrival_date_month;

CREATE TABLE bab_revenue -- This query ceates a table that identifies revenue for adults with children and babies
SELECT
	arrival_date_month,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS bab_total_revenue,
    ROUND(AVG((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS bab_avg_revenue
FROM hotels
WHERE adults >= 1 AND children >=1 AND babies >=1
GROUP BY 
	arrival_date_month;

CREATE TABLE revenue_family -- This query creates a table that merges the previous four tables to compare total and average revenues according to family members
SELECT 
	a.*,
	ch.child_total_revenue,
    ch.child_avg_revenue,
    ba.babies_total_revenue,
    ba.babies_avg_revenue,
    b.bab_total_revenue,
    b.bab_avg_revenue
FROM only_adults_revenue as a
	LEFT JOIN child_revenue as ch
		ON a.arrival_date_month = ch.arrival_date_month
	LEFT JOIN babies_only_revenue as ba
		ON a.arrival_date_month = ba.arrival_date_month
	LEFT JOIN bab_revenue as b
		ON a.arrival_date_month = b.arrival_date_month;

SELECT -- This query identifies which type of family generates the most in total and average revenue
	SUM(adults_total_revenue) AS only_adults,
    SUM(child_total_revenue) AS adults_child,
    SUM(babies_total_revenue) AS adults_babies,
    SUM(bab_total_revenue) AS adults_child_babies,
    AVG(adults_avg_revenue) AS avg_only_adults,
    AVG(child_avg_revenue) AS avg_adults_child,
    AVG(babies_avg_revenue) AS avg_adults_babies,
    AVG(bab_avg_revenue) AS avg_adults_child_babies
FROM revenue_family;

/*The following query identifies the year and month with the biggest average daily rate and average number of guests*/
SELECT
	arrival_date_year,
    arrival_date_month,
    AVG(adr) AS avg_daily_rate,
    AVG(adults + children + babies) AS avg_guests
FROM hotels
GROUP BY 
	arrival_date_month,
    arrival_date_year
ORDER BY avg_guests DESC;

/*The following query identifies the frecuency of meal types*/
SELECT
	meal,
    COUNT(meal) as meal_counts,
    COUNT(meal) * 100.0 / (SELECT count(meal) FROM hotels) AS meal_perc,
    ROUND(SUM(Cost),2) AS meal_cost,
    ROUND(AVG(Cost),2) AS avg_meal_cost
FROM hotels
GROUP BY meal
ORDER BY meal_counts DESC;

/*Analysis by country of origin*/
SELECT -- This query identifies from which country most of the guests come from
	country,
    COUNT(country) AS country_guests,
    COUNT(country) * 100.0 / (SELECT count(country) FROM hotels) AS country_perc
FROM hotels
GROUP BY country
ORDER BY country_guests DESC
LIMIT 10;

SELECT -- This query counts how many distinct countries are the origin of the guests
    COUNT(DISTINCT country)
FROM hotels;

/* This query identifies from which market segment provides more guests and revenue */
SELECT
	market_segment,
    COUNT(market_segment) AS guests,
    COUNT(market_segment) * 100.0 / (SELECT count(market_segment) FROM hotels) AS percentage,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue
FROM hotels
GROUP BY market_segment
ORDER BY revenue DESC;

/* This query identifies which distribution channel provides more guests and more revenue*/
SELECT
	distribution_channel,
    COUNT(distribution_channel) AS guests,
    COUNT(distribution_channel) * 100.0 / (SELECT COUNT(distribution_channel) FROM hotels) AS percentage,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue
FROM hotels
GROUP BY distribution_channel
ORDER BY revenue DESC;

/* This query shows the number of guests, nights stayed, revenue, and average revenue for recurrent and non recurrent guests*/
SELECT
    is_repeated_guest,
    COUNT(is_repeated_guest) AS guests,
    SUM(stays_in_week_nights + stays_in_weekend_nights) AS nights,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue,
    ROUND(AVG((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS avg_revenue
FROM hotels
GROUP BY is_repeated_guest
ORDER BY guests DESC;

/* The following queries show the differences of the reserved and assigned room types*/
SELECT -- This query identifies the number of guests that reservead each room type
	reserved_room_type,
    COUNT(reserved_room_type) AS guests_1
FROM hotels
GROUP BY
	reserved_room_type
ORDER BY reserved_room_type;
    
SELECT -- This query identifies the number of guests that were assigned to each room type
	assigned_room_type,
    COUNT(assigned_room_type) AS guests_2
FROM hotels
GROUP BY
	assigned_room_type
ORDER BY assigned_room_type;

SELECT -- This query tells us how many of the guests that reserved each room type actually got the same when they were assigned. Also it shows the average daily rate, revenue, and average revenue for each reserved room type
	reserved_room_type,
    COUNT(reserved_room_type) as reserved,
	SUM(CASE WHEN reserved_room_type = assigned_room_type THEN 1
	ELSE 0 END) AS assigned_as_reserved,
    ROUND(AVG(adr),2) as avg_adr,
    ROUND(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS revenue,
    ROUND(AVG((stays_in_week_nights + stays_in_weekend_nights) * adr),2) AS avg_revenue
FROM hotels
GROUP BY reserved_room_type
ORDER BY reserved_room_type;
