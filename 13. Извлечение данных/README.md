# 13. Извлечение данных

## Описание проекта
Вы аналитик российской авиакомпании F9, выполняющей внутренние пассажирские перевозки. Важно понять предпочтения пользователей, покупающих билеты на разные направления.  
Вам предстоит изучить базу данных и проанализировать спрос пассажиров на рейсы в города, где проходят крупнейшие культурные фестивали.  

### Инструкция по выполнению
1. Проведите исследовательский анализ данных средствами SQL.
2. Соберите данные для анализа из базы.
3. Проанализируйте данные средствами Python.

### Используемый стек инструментов
- python
- pandas
- numpy
- sql
- matplotlib
- seaborn

### Описание данных
В вашем распоряжении база данных об авиаперевозках.  
Таблица airports — информация об аэропортах:
- airport_code — трёхбуквенный код аэропорта
- airport_name — название аэропорта
- city — город
- timezone — временная зона

Таблица aircrafts — информация о самолётах:
- aircraft_code — код модели самолёта
- model — модель самолёта
- range — дальность полёта

Таблица tickets — информация о билетах:
- ticket_no — уникальный номер билета
- passenger_id — персональный идентификатор пассажира
- passenger_name — имя и фамилия пассажира

Таблица flights — информация о рейсах:
- flight_id — уникальный идентификатор рейса
- departure_airport — аэропорт вылета
- departure_time — дата и время вылета
- arrival_airport — аэропорт прилёта
- arrival_time — дата и время прилёта
- aircraft_code — id самолёта

Таблица ticket_flights — стыковая таблица «рейсы-билеты»
- ticket_no — номер билета
- flight_id — идентификатор рейса

Таблица festivals — информация о фестивалях
- festival_id — уникальный номер фестиваля
- festival_date — дата проведения фестиваля
- festival_city — город проведения фестиваля
- festival_name — название фестиваля

Схема таблиц:  
![Структура БД](https://b.radikal.ru/b38/2105/38/13dad104d90b.jpg "Орк")

Пояснение: В базе данных нет прямой связи между таблицами airports и festivals, а также festivals и flights. Но вы можете писать JOIN и связывать эти таблицы по городу проведения фестиваля (festival_city) и городу аэропорта (city). Потребуется некоторое преобразование дат в flights, и тогда эту таблицу также можно будет связать по дате проведения фестиваля (festival_date) в запросах с JOIN.

SQL запросы:  
```sql
SELECT 
    aircrafts.model,
    COUNT(flights.flight_id) AS flights_amount
FROM 
    aircrafts
    INNER JOIN flights ON flights.aircraft_code = aircrafts.aircraft_code
WHERE
    CAST(flights.departure_time AS date) >= '2018-09-01' AND CAST(flights.departure_time AS date) < '2018-10-01'
GROUP BY
    aircrafts.model


SELECT 
    --aircrafts.model,
    CASE WHEN aircrafts.model LIKE '%Boeing%' THEN 'Boeing'
    WHEN aircrafts.model LIKE '%Airbus%' THEN 'Airbus'
    ELSE 'other'
    END AS model,
    COUNT(*) AS flights_amount
FROM 
    aircrafts
    INNER JOIN flights ON flights.aircraft_code = aircrafts.aircraft_code
WHERE
    CAST(flights.departure_time AS date) >= '2018-09-01' AND CAST(flights.departure_time AS date) < '2018-10-01'
GROUP BY
    CASE WHEN aircrafts.model LIKE '%Boeing%' THEN 'Boeing'
    WHEN aircrafts.model LIKE '%Airbus%' THEN 'Airbus'
    ELSE 'other'
    END


SELECT
    subq.city AS city,
    AVG(subq.sum_flights) AS average_flights
FROM
(SELECT 
    airports.city AS city,
    COUNT(flights.flight_id) AS sum_flights,
    EXTRACT (day from CAST(flights.arrival_time AS date)) AS day
FROM
    airports
    INNER JOIN flights ON flights.arrival_airport = airports.airport_code
WHERE
    CAST(flights.arrival_time AS date) BETWEEN '2018-08-01' AND '2018-08-31'
GROUP BY
    airports.city,
    day) AS subq
GROUP BY 
    city


SELECT
    festival_name, 
    EXTRACT(WEEK FROM CAST(festival_date AS date)) AS festival_week
FROM
    festivals
WHERE
    festival_date BETWEEN '2018-07-23' AND '2018-09-30' AND festival_city = 'Москва'


SELECT
    EXTRACT(week FROM CAST(flights.arrival_time AS date)) AS week_number,
    COUNT(ticket_flights.ticket_no) AS ticket_amount,
    sub.festival_week AS festival_week,
    sub.festival_name AS festival_name
    
FROM tickets
    LEFT JOIN ticket_flights ON ticket_flights.ticket_no  = tickets.ticket_no
    LEFT JOIN flights ON flights.flight_id  = ticket_flights.flight_id
    LEFT JOIN airports ON airports.airport_code = flights.arrival_airport
    LEFT JOIN
        (SELECT
             festival_name,
             EXTRACT(WEEK FROM CAST(festival_date AS date)) AS festival_week
         FROM
             festivals
         WHERE
             festival_date BETWEEN '2018-07-23' AND '2018-09-30' AND festival_city = 'Москва')
             AS sub ON sub.festival_week = EXTRACT(week FROM CAST(flights.arrival_time AS date))
WHERE
    CAST(flights.arrival_time AS date) BETWEEN '2018-07-23' AND '2018-09-30' AND airports.city = 'Москва'
GROUP BY
    EXTRACT(week FROM CAST(flights.arrival_time AS date)), 
    festival_week, 
    festival_name
``` 