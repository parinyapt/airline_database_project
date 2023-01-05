# Airline Database Design

## SQL Query Example
#### 1: 
```sql
INSERT INTO `member`(
    `first_name`,
    `middle_name`,
    `last_name`,
    `gender`,
    `date_of_birth`,
    `tel`,
    `email`,
    `password`,
    `create_at`,
    `update_at`
)
VALUES(
    'Parinya',
    NULL,
    'Termkasipanich',
    'M',
    '2002-11-09',
    '66871189903',
    'parinya.prin@g.swu.ac.th',
    '$2a$07$mZUBnOnbC.mWcSzIRMVPF.YopQuq4GQvR/DaiFEc8B4/x9kwVmmZC',
    CURRENT_TIMESTAMP(),
    CURRENT_TIMESTAMP()
);
```

#### 2: 
```sql
UPDATE
    `employee`
SET
    `last_name` = 'Rukdatabas',
    `update_at` = CURRENT_TIMESTAMP()
WHERE
    `eid` = '650005';
```

#### 3: 
```sql
DELETE FROM member WHERE id = 1;
```

#### 4: 
```sql
SELECT
    employee.id,
    employee.eid,
    employee.idcard,
    employee.passport_no,
    title.short,
    employee.first_name,
    employee.middle_name,
    employee.last_name,
    (CASE
        WHEN employee.gender = "M" THEN 'Male'
        WHEN employee.gender = "F" THEN 'Female'
        ELSE 'Gender Not Found'
    END) AS gender,
    employee.date_of_birth,
    YEAR(FROM_DAYS(DATEDIFF(CURRENT_DATE(),`date_of_birth`))) as age,
    employee_position.title AS position,
    employee_hire_log.start_date 
FROM
    employee
INNER JOIN title ON employee.title_id = title.id
INNER JOIN employee_hire_log ON employee.id = employee_hire_log.employee_id
INNER JOIN employee_position ON employee_hire_log.employee_position_id = employee_position.id
WHERE
    employee_hire_log.end_date IS NULL OR
    (
        employee_hire_log.end_date IS NOT NULL AND
        employee_hire_log.start_date <= CURRENT_DATE() AND
        employee_hire_log.end_date >= CURRENT_DATE()
    )
ORDER BY employee_hire_log.employee_position_id ASC;
```

#### 5: 
```sql
SELECT
    employee.id,
    employee.eid,
    employee.idcard,
    employee.passport_no,
    title.short,
    employee.first_name,
    employee.middle_name,
    employee.last_name,
    SUM(employee_salary_log.amount_baht) AS total_pay
FROM
    employee
INNER JOIN title ON employee.title_id = title.id
INNER JOIN employee_salary_log ON employee.id = employee_salary_log.employee_id
WHERE
	(
        employee_salary_log.salary_log_type_id = 1 AND employee_salary_log.start_date <= LAST_DAY("2020-06-01") AND
        (
            employee_salary_log.end_date IS NULL OR
            (
                employee_salary_log.end_date IS NOT NULL AND
                employee_salary_log.end_date >= LAST_DAY("2020-06-01")
            )
        )
   	) OR
    (
    	(employee_salary_log.salary_log_type_id = 2 OR employee_salary_log.salary_log_type_id = 3) AND 
        employee_salary_log.start_date LIKE '2020-06%' AND
        employee_salary_log.end_date LIKE '2020-06%'
    )
GROUP BY employee.id
ORDER BY total_pay DESC;
```

#### 6: 
```sql
SELECT DISTINCT
    a1.iata_code AS from_airport_iata_code,
    a1.name AS from_airport,
    a1.country AS from_airport_country,
    a2.iata_code AS destination_airport_iata_code,
    a2.name AS destination_airport,
    a2.country AS destination_airport_country
FROM
    schedule
INNER JOIN direction ON schedule.direction_id = direction.id
INNER JOIN airport a1 ON direction.from_airport_id = a1.id
INNER JOIN airport a2 ON direction.destination_airport_id = a2.id;
```

#### 7: 
```sql
SELECT
    airplane_manufacturer.name,
    airplane_data.model,
    airplane_data.icao_code,
    COUNT(airplane_data.model) AS plane_count
FROM
    airplane
INNER JOIN airplane_data ON airplane.airplane_data_id = airplane_data.id
INNER JOIN airplane_manufacturer ON airplane_data.airplane_manufacturer_id = airplane_manufacturer.id
WHERE airplane.airplane_status_id = (SELECT id FROM airplane_status WHERE airplane_status.title = "active")
GROUP BY airplane_data.model;
```

#### 8: 
```sql
SELECT
    `schedule`.`flight_code`,
    a1.iata_code AS from_airport_iata_code,
    a1.name AS from_airport,
    a1.city AS from_airport_city,
    a1.country AS from_airport_country,
    a2.iata_code AS destination_airport_iata_code,
    a2.name AS destination_airport,
    a2.city AS destination_airport_city,
    a2.country AS destination_airport_country,
    `schedule`.`departure_datetime_gmt`
FROM
    `schedule`
INNER JOIN direction ON `schedule`.direction_id = direction.id
INNER JOIN airport a1 ON direction.from_airport_id = a1.id
INNER JOIN airport a2 ON direction.destination_airport_id = a2.id
WHERE
    direction.from_airport_id IN(
    SELECT
        airport.id
    FROM
        airport
    WHERE
        airport.city = "Bangkok"
) AND direction.destination_airport_id IN(
    SELECT
        airport.id
    FROM
        airport
    WHERE
        airport.city = "Shanghai"
) AND `schedule`.departure_datetime_gmt >= '2022-09-01 00:00:00' 
AND `schedule`.departure_datetime_gmt <= '2022-09-10 23:59:59' 
AND `schedule`.departure_datetime_gmt NOT LIKE '2022-09-04%';
```

#### 9: 
```sql
SELECT
    employee.id,
    employee.eid,
    employee.first_name,
    employee.middle_name,
    employee.last_name,
    SUM(`schedule`.confirm_duration) AS flightMinute
FROM
    `schedule_employee_plan`
INNER JOIN employee ON schedule_employee_plan.employee_id = employee.id
INNER JOIN `schedule` ON schedule_employee_plan.schedule_id = `schedule`.id
WHERE employee.id IN (SELECT
    employee.id
FROM
    employee
INNER JOIN employee_hire_log ON employee.id = employee_hire_log.employee_id
INNER JOIN employee_position ON employee_hire_log.employee_position_id = employee_position.id
WHERE employee_position.title = "Pilot" AND (employee_hire_log.end_date IS NULL OR
    (
        employee_hire_log.end_date IS NOT NULL AND
        employee_hire_log.start_date <= CURRENT_DATE() AND
        employee_hire_log.end_date >= CURRENT_DATE()
    )
)) AND schedule_employee_plan.schedule_employee_plan_status_id =(
    SELECT
        id
    FROM
        schedule_employee_plan_status
    WHERE
        title = "confirm"
)
GROUP BY employee.id
HAVING flightMinute > 300;
```

#### 10: 
```sql
SELECT
    `schedule`.`flight_code`,
    airplane_seat.code,
    airplane_seat_class.title,
    direction.distance_miles,
    airplane_seat_class.price_rate,
    (direction.distance_miles * airplane_seat_class.price_rate) AS ticket_price,
    (CASE
        WHEN discount.id IS NOT NULL THEN (discount.discount_percent)
        ELSE 0
    END) AS discount_percent,
    (CASE
        WHEN discount.id IS NOT NULL THEN ((direction.distance_miles * airplane_seat_class.price_rate) * discount.discount_percent/100)
        ELSE 0
    END) AS ticket_discount,
    (CASE
        WHEN discount.id IS NOT NULL THEN ((direction.distance_miles * airplane_seat_class.price_rate) - ((direction.distance_miles * airplane_seat_class.price_rate) * discount.discount_percent/100))
        ELSE (direction.distance_miles * airplane_seat_class.price_rate)
    END) AS total_ticket_price
FROM
    `schedule`
INNER JOIN direction ON `schedule`.direction_id = direction.id
INNER JOIN airplane_seat ON `schedule`.`airplane_id` = airplane_seat.airplane_id
INNER JOIN airplane_seat_class ON airplane_seat.airplane_seat_class_id = airplane_seat_class.id
LEFT JOIN discount ON `schedule`.id = discount.schedule_id AND airplane_seat_class.id = discount.airplane_seat_class_id AND '2022-11-25 11:00:00' >= DATE_SUB(`schedule`.`departure_datetime_gmt`, INTERVAL discount.before_hour HOUR) AND '2022-11-25 11:00:00' < `schedule`.`departure_datetime_gmt`
WHERE `schedule`.`flight_code` = "PT004" AND
`schedule`.departure_datetime_gmt LIKE '2022-11-26%' AND 
airplane_seat.code = "A1";
```

#### 11: 
```sql
SELECT
    booking_passenger_detail.passport_no,
    title.short,
    booking_passenger_detail.first_name,
    booking_passenger_detail.middle_name,
    booking_passenger_detail.last_name
FROM
    booking
INNER JOIN booking_passenger_detail ON booking.booking_passenger_detail_id = booking_passenger_detail.id
INNER JOIN title ON booking_passenger_detail.title_id = title.id
WHERE booking.reference_code = 'A2y34LlpCp';
```

#### 12: 
```sql
SELECT DISTINCT
    booking_passenger_detail.passport_no,
    title.short,
    booking_passenger_detail.first_name,
    booking_passenger_detail.middle_name,
    booking_passenger_detail.last_name,
    airplane_seat.code
FROM
    booking
INNER JOIN booking_passenger_detail ON booking.booking_passenger_detail_id = booking_passenger_detail.id
INNER JOIN `schedule` ON booking.schedule_id = `schedule`.`id`
INNER JOIN payment ON booking.reference_code = payment.booking_reference_code
INNER JOIN airplane_seat ON booking.airplane_seat_id = airplane_seat.id
INNER JOIN title ON booking_passenger_detail.title_id = title.id
WHERE `schedule`.`flight_code` = "PT001" AND `schedule`.`departure_datetime_gmt` = '2022-09-01 05:00:00'
ORDER BY airplane_seat.airplane_seat_class_id ASC, airplane_seat.code ASC;
```

#### 13: 
```sql
SELECT
    employee.id,
    employee.eid,
    employee.idcard,
    employee.passport_no,
    title.short,
    employee.first_name,
    employee.middle_name,
    employee.last_name,
    COUNT(schedule_employee_plan_status.title) AS absence_count
FROM
    schedule_employee_plan
INNER JOIN schedule_employee_plan_status ON schedule_employee_plan.schedule_employee_plan_status_id = schedule_employee_plan_status.id
INNER JOIN employee ON schedule_employee_plan.employee_id = employee.id
INNER JOIN title ON employee.title_id = title.id
WHERE schedule_employee_plan_status.title = 'absence'
GROUP BY employee.id;
```

#### 14: 
```sql
SELECT DISTINCT
    airport.name
FROM
    airport
WHERE airport.country = "Thailand"
ORDER BY airport.name DESC;
```

#### 15: 
```sql
SELECT DISTINCT
    `schedule`.`flight_code`,
    `schedule`.`departure_datetime_gmt`,
    airplane_seat_class.title AS class,
    discount.before_hour,
    DATE_SUB(`schedule`.`departure_datetime_gmt`, INTERVAL discount.before_hour HOUR) AS datetime_get_discount,
    discount.discount_percent
FROM
    discount
INNER JOIN `schedule` ON discount.schedule_id = `schedule`.`id`
INNER JOIN airplane_seat_class ON discount.airplane_seat_class_id = airplane_seat_class.id
ORDER BY `schedule`.`flight_code`,`schedule`.`departure_datetime_gmt`,airplane_seat_class.id ASC;
```

> By Parinya Termkasipanich