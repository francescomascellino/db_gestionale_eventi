Selezionare tutti gli utenti con il ruolo di Admin nella tabella roles
```sql
SELECT
    users.* (SELECTS ALL COLUMNS FROM USERS TABLE), roles.name AS role_name
FROM
    users
JOIN roles ON users.role_id = roles.id
WHERE
    roles.name = 'Admin';
```

Selezionare tutti gli eventi gratis, cioè con prezzo nullo
```sql
SELECT
    *
FROM EVENTS
WHERE EVENTS
    .price IS NULL
```

Selezionare tutte le location in ordine alfabetico
```sql
SELECT
    *
FROM
    locations
ORDER BY
    locations.name ASC;
```

Selezionare tutti gli eventi che costano meno di 20 euro e durano meno di 3 ore
```sql
TIME_TO_SEC: Convert a time value into seconds:

SELECT TIME_TO_SEC("00:00:05");

SELECT
    *
FROM EVENTS
WHERE
    price < 20 AND TIME_TO_SEC(EVENTS.duration) < TIME_TO_SEC('03:00:00');
```

Selezionare tutti gli eventi di dicembre 2023
```sql
SELECT
    *
FROM EVENTS
WHERE
    MONTH(EVENTS.start) = 12 AND YEAR(EVENTS.start) = 2023;
```

Selezionare tutti gli eventi con una durata maggiore alle 2 ore
```sql
SELECT
    *
FROM EVENTS
WHERE
    EVENTS.duration > '02:00:00';
```

Selezionare tutti gli eventi, mostrando nome, data di inizio, ora di inizio, ora di fine e
durata totale
```sql
SELECT NAME AS
    nome,
    DATE(START) AS DATA,
    DATE_FORMAT(START, '%H:%i') AS 'hour',
    TIME(START) AS ora,
    DATE_FORMAT(duration, '%H:%i') AS duration,
    DATE_FORMAT(
        DATE_ADD(
            START,
            INTERVAL duration HOUR_SECOND
        ),
        '%H:%i'
    ) AS end_time
FROM EVENTS;
```

Selezionare tutti gli eventi aggiunti da “Fabiano Lombardo”
```sql
SELECT EVENTS
    .*,
    users.first_name,
    users.last_name
FROM EVENTS
JOIN users ON EVENTS
    .user_id = users.id
WHERE
    users.first_name = 'Fabiano' AND users.last_name = 'Lombardo'
```

Selezionare tutti gli utenti admin ed editor
```sql
SELECT
    users.*, roles.name AS role_name
FROM
    users
JOIN roles ON users.role_id = roles.id
WHERE
    roles.name = 'Admin' OR roles.name = 'Editor';
```

Selezionare tutti i concerti (eventi con il tag “concerti”)
```sql
SELECT EVENTS
    .*, tags.name AS 'tag'
FROM EVENTS
JOIN event_tag ON EVENTS
    .id = event_tag.event_id
JOIN tags ON tags.id = event_tag.tag_id
WHERE
    tags.name = 'concerti';
```

Selezionare tutti i tag e il prezzo medio degli eventi a loro collegati
```sql
SELECT
    tags.name AS tag,
    AVG(events.price) AS avg_price
FROM
    tags
JOIN
    event_tag ON tags.id = event_tag.tag_id
JOIN
    events ON events.id = event_tag.event_id
GROUP BY
    tags.name;
```

Selezionare tutti i partecipanti per l’evento “Concerto Classico Serale” (slug:
concerto-classico-serale)
```sql
SELECT
    users.*,
    EVENTS.name AS 'name'
FROM
    users
JOIN bookings ON users.id = bookings.user_id
JOIN EVENTS ON EVENTS
    .id = bookings.event_id
WHERE EVENTS
    .slug = 'concerto-classico-serale'
```

Selezionare tutti i partecipanti all’evento “Festival Jazz Estivo” (slug:
festival-jazz-estivo) specificando nome e cognome
```sql
SELECT
    users.first_name,
    users.last_name,
    EVENTS.name AS 'name'
FROM
    users
JOIN bookings ON users.id = bookings.user_id
JOIN EVENTS ON EVENTS
    .id = bookings.event_id
WHERE EVENTS
    .slug = 'festival-jazz-estivo';
```

Selezionare tutti gli eventi sold out (dove il totale delle prenotazioni è uguale ai
biglietti totali per l’evento)
```sql
SELECT EVENTS
    .name,
    EVENTS.total_tickets,
    total_bookings
FROM EVENTS
JOIN(
    SELECT event_id,
        COUNT(*) AS total_bookings
    FROM
        bookings
    GROUP BY
        event_id
) AS bookings_count
ON EVENTS
    .id = bookings_count.event_id
WHERE EVENTS
    .total_tickets = bookings_count.total_bookings;
```

Oppure:
```sql
SELECT EVENTS
    .*,
    COUNT(*) AS `sold_out`
FROM
    `events`
JOIN `bookings` ON `bookings`.`event_id` = `events`.`id`
GROUP BY
    `bookings`.`event_id`
HAVING
    COUNT(*) = MAX(`events`.`total_tickets`);
```

Selezionare tutte le location in ordine per chi ha ospitato più eventi
```sql
SELECT
    locations.id,
    locations.name,
    COUNT(EVENTS.id) AS total_events
FROM
    locations
JOIN EVENTS ON locations.id = EVENTS.location_id
GROUP BY
    locations.id,
    locations.name
ORDER BY
    total_events
DESC;
```

Selezionare tutti gli utenti che si sono prenotati a più di 70 eventi
```sql
SELECT
    users.id,
    users.username,
    COUNT(bookings.id) AS total_bookings
FROM
    users
JOIN
    bookings ON users.id = bookings.user_id
GROUP BY
    users.id,
    users.username
HAVING
    COUNT(bookings.id) > 70;
```

Selezionare tutti gli eventi, mostrando il nome dell’evento, il nome della location, il
numero di prenotazioni e il totale di biglietti ancora disponibili per l’evento
```sql
SELECT
    events.name AS event_name,
    locations.name AS location_name,
    COUNT(bookings.id) AS total_bookings,
    events.total_tickets - COUNT(bookings.id) AS tickets_available
FROM
    events
JOIN
    locations ON events.location_id = locations.id
LEFT JOIN
    bookings ON events.id = bookings.event_id
GROUP BY
    events.id,
    events.name,
    locations.id,
    locations.name,
    events.total_tickets;
```