# assignmnent_viking_analytics


Dustin Lee

Warmup

1.  Get a list of all users in California

SELECT *
FROM users
JOIN states ON (state_id = states.id)
WHERE states.name = 'California'

2.  Get a list of all airports in Minnesota

SELECT *
FROM airports
JOIN states ON (state_id = states.id)
WHERE states.name = 'Minnesota'

3.  Get a list of all payment methods used on itineraries by the user with email address "heidenreich_kara@kunde.net"

SELECT *
FROM itineraries
JOIN users ON (user_id = users.id)
WHERE users.email = 'heidenreich_kara@kunde.net'

4.  Get a list of prices of all flights whose origins are in Kochfurt Probably International Airport.

SELECT price
FROM flights
JOIN airports ON (origin_id = airports.id)
WHERE airports.long_name = 'Kochfurt Probably International Airport'


5.  Find a list of all Airport names and codes which connect to the airport coded LYT.

SELECT origin.long_name AS origin, origin.code AS origin_code, destination.code AS destination_code
FROM flights
JOIN airports origin ON origin.id = flights.origin_id
JOIN airports destination ON destination.id = flights.destination_id
WHERE destination.code = 'LYT'

6.  Get a list of all airports visited by user Dannie D'Amore after January 1, 2012.

SELECT airports.long_name AS airport, CONCAT(users.first_name, ' ', users.last_name) AS user FROM airports
JOIN flights origin ON airports.id = origin.origin_id
JOIN flights destination ON airports.id = destination.destination_id
JOIN tickets ON origin.id = tickets.flight_id
JOIN itineraries ON itineraries.id = tickets.itinerary_id
JOIN users ON users.id = itineraries.user_id
WHERE users.first_name = 'Dannie'
AND users.last_name = 'D''Amore'
AND origin.departure_time > '2012-1-1'

Aggregation

1. Find the top 5 most expensive flights that end in California.

SELECT CONCAT(origin.long_name, ' -> ', destination.long_name) AS flight, flights.price, states.name AS state  FROM flights
JOIN airports origin ON origin.id = flights.origin_id
JOIN airports destination ON destination.id = flights.destination_id
JOIN states ON states.id = destination.state_id
WHERE states.name = 'California'
ORDER BY price DESC

2.  Find the shortest flight that username "ryann_anderson" took.

SELECT users.username AS username,
CONCAT(origin.long_name, ' -> ', destination.long_name) AS flight,
MIN(flights.arrival_time - flights.departure_time) AS flight_time
FROM tickets
JOIN itineraries ON itineraries.id = tickets.itinerary_id
JOIN flights ON flights.id = tickets.flight_id
JOIN users ON users.id = itineraries.user_id
JOIN airports origin ON origin.id = flights.origin_id
JOIN airports destination ON destination.id = flights.destination_id
WHERE username = 'ryann_anderson'
GROUP BY username, flight

3.  Find the average flight distance for every city in Florida

SELECT destination_state.name AS state, cities.name AS city, AVG(flights.distance) AS average_distance FROM flights
JOIN airports origin ON origin.id = flights.origin_id
JOIN airports destination ON destination.id = flights.destination_id
JOIN states origin_state ON origin_state.id = origin.state_id
JOIN states destination_state ON destination_state.id = destination.state_id
JOIN cities ON cities.id = destination.city_id
WHERE destination_state.name = 'Florida'
GROUP BY state, city

4.  Find the 3 users who spent the most money on flights in 2013

SELECT users.username, SUM(flights.price) AS total_money FROM flights
JOIN tickets ON flights.id = tickets.flight_id
JOIN itineraries ON itineraries.id = tickets.itinerary_id
JOIN users ON users.id = itineraries.user_id
WHERE tickets.created_at BETWEEN '2015-1-1' AND '2015-12-31'
GROUP BY users.username
ORDER BY total_money DESC
LIMIT 3

5.  Count all flights to OR from the city of Lake Vivienne that did not land in Florida.

SELECT COUNT(*)
FROM flights
JOIN airports origin ON origin.id = flights.origin_id
JOIN airports destination ON destination.id = flights.destination_id
JOIN states origin_state ON origin_state.id = origin.state_id
JOIN states destination_state ON destination_state.id = destination.state_id
JOIN cities origin_city ON origin_city.id = destination.city_id
JOIN cities destination_city ON destination_city.id = destination.city_id
WHERE destination_state.name != 'Florida'
AND (
  origin_city.name = 'Lake Vivienne'
  OR destination_city.name = 'Lake Vivienne'
)

6.  Return the range of lengths of flights in the system(the maximum, and the minimum).

SELECT
MIN(arrival_time - departure_time) AS min_flight_length,
MAX(arrival_time - departure_time) AS max_flight_length
FROM flights

Advanced

1.  Find the most popular travel destination for users who live in Kansas.

SELECT destination_city.name AS city, COUNT(destination_city) AS count
FROM users
JOIN states user_state ON user_state.id = users.state_id
JOIN itineraries ON users.id = itineraries.user_id
JOIN tickets ON itineraries.id = tickets.itinerary_id
JOIN flights ON flights.id = tickets.flight_id
JOIN airports destination ON destination.id = flights.destination_id
JOIN states destination_state ON destination_state.id = destination.state_id
JOIN cities destination_city ON destination_city.id = destination.city_id
WHERE user_state.name = 'New Hampshire'
GROUP BY city
ORDER BY count
LIMIT 1

2.  How many flights have round trips possible? In other words, we want the count of all airports where there exists a flight FROM that airport and a later flight TO that airport.

SELECT COUNT(*) AS num_round_trip_flights
FROM flights start
JOIN flights finish ON start.origin_id = finish.destination_id
WHERE start.arrival_time < finish.departure_time

3.  Find the cheapest flight that was taken by a user who only had one itinerary.

SELECT * FROM flights
WHERE id = (
  SELECT flights.id FROM itineraries
  JOIN tickets ON itineraries.id = tickets.itinerary_id
  JOIN flights ON flights.id = tickets.flight_id
  GROUP BY flights.id
  HAVING COUNT(itineraries.user_id) = 1
  ORDER BY flights.price
  LIMIT 1
)

4.  Find the average cost of a flight itinerary for users in each state in 2012.

SELECT user_state.name, AVG(flights.price) FROM flights
JOIN tickets ON flights.id = tickets.flight_id
JOIN itineraries ON itineraries.id = tickets.itinerary_id
JOIN users ON users.id = itineraries.user_id
JOIN states user_state ON user_state.id = users.state_id
WHERE flights.arrival_time BETWEEN '2012-1-1' AND '2012-12-31'
GROUP BY user_state.name
ORDER BY user_state.name