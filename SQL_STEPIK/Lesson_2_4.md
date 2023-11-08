

(1) Вывести города, в которых живут клиенты, оформлявшие заказы в интернет-магазине. Указать количество заказов в каждый город, этот столбец назвать Количество. Информацию вывести по убыванию количества заказов, а затем в алфавитном порядке по названию городов.

```
SELECT name_city, COUNT(buy.client_id) AS Количество
FROM
    city
    INNER JOIN client ON city.city_id = client.city_id
    LEFT JOIN buy ON buy.client_id = client.client_id
 
GROUP BY name_city
ORDER BY Количество DESC, name_city;
```

(2) Вывести номера заказов (buy_id) и названия этапов,  на которых они в данный момент находятся. Если заказ доставлен –  информацию о нем не выводить. Информацию отсортировать по возрастанию buy_id.

```
SELECT DISTINCT buy_id, name_step
FROM
    buy_step
    INNER JOIN step ON buy_step.step_id = step.step_id
WHERE (date_step_beg IS NOT NULL) AND (date_step_end IS NULL)
ORDER BY buy_id;
```

(3) Вывести жанр (или жанры), в котором было заказано больше всего экземпляров книг, указать это количество. Последний столбец назвать Количество.

```
SELECT name_genre, SUM(buy_book.amount) AS Количество
FROM
    genre
    JOIN book USING(genre_id)
    JOIN buy_book USING(book_id)
WHERE name_genre IN (
    SELECT first_query.name_genre
    FROM (
        SELECT name_genre, SUM(buy_book.amount) AS Количество
        FROM genre
            JOIN book USING(genre_id)
            JOIN buy_book USING(book_id)
        GROUP BY name_genre
     ) first_query
    INNER JOIN (
        SELECT name_genre, SUM(buy_book.amount) AS Количество
        FROM genre
            JOIN book USING(genre_id)
            JOIN buy_book USING(book_id)
        GROUP BY name_genre
        ORDER BY Количество DESC
        LIMIT 1
    ) second_query
    ON first_query.Количество = second_query.Количество
)
GROUP BY name_genre
ORDER BY Количество DESC;
```

(4) Сравнить ежемесячную выручку от продажи книг за текущий и предыдущий годы. Для этого вывести год, месяц, сумму выручки в отсортированном сначала по возрастанию месяцев, затем по возрастанию лет виде. Название столбцов: Год, Месяц, Сумма.

```
SELECT YEAR(date_payment) AS Год, MONTHNAME(date_payment) AS Месяц, SUM(amount * price) AS Сумма
FROM buy_archive
GROUP BY Год, Месяц
UNION
SELECT YEAR(date_step_end) AS Год, MONTHNAME(date_step_end) AS Месяц, 
       SUM(buy_book.amount * book.price) AS Сумма
FROM 
    book
    INNER JOIN buy_book USING(book_id)
    INNER JOIN buy USING(buy_id)
    INNER JOIN buy_step USING(buy_id)
    INNER JOIN step USING(step_id)
WHERE name_step = 'Оплата' AND date_step_end IS NOT NULL
GROUP BY Год, Месяц
ORDER BY Месяц, Год;
```

(5) Для каждой отдельной книги необходимо вывести информацию о количестве проданных экземпляров и их стоимости за 2020 и 2019 год . Вычисляемые столбцы назвать Количество и Сумма. Информацию отсортировать по убыванию стоимости.

```
SELECT title, SUM(Количество) AS Количество, SUM(Сумма) AS Сумма
FROM (
    SELECT title, SUM(buy_book.amount) AS Количество, SUM(buy_book.amount * book.price) AS Сумма
    FROM 
        book
        INNER JOIN buy_book USING(book_id)
        INNER JOIN buy USING(buy_id)
        INNER JOIN buy_step USING(buy_id)
        INNER JOIN step USING(step_id)
    WHERE name_step = 'Оплата' AND date_step_end IS NOT NULL
    GROUP BY title
    UNION ALL
    SELECT title, SUM(buy_archive.amount) AS Количество, SUM(buy_archive.amount * buy_archive.price) AS Сумма
    FROM
        buy_archive
        INNER JOIN book USING(book_id)
    GROUP BY title
) AS derived_table
GROUP BY title
ORDER BY Сумма DESC;
```