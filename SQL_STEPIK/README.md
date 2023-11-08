(1) Вывести жанр (или жанры), в котором было заказано больше всего экземпляров книг, указать это количество. Последний столбец назвать Количество.

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

(2) Сравнить ежемесячную выручку от продажи книг за текущий и предыдущий годы. Для этого вывести год, месяц, сумму выручки в отсортированном сначала по возрастанию месяцев, затем по возрастанию лет виде. Название столбцов: Год, Месяц, Сумма.

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

(3) Для каждой отдельной книги необходимо вывести информацию о количестве проданных экземпляров и их стоимости за 2020 и 2019 год . Вычисляемые столбцы назвать Количество и Сумма. Информацию отсортировать по убыванию стоимости.

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

(4) Вывести фамилию, номер машины и нарушение только для тех водителей, которые на одной машине нарушили одно и то же правило   два и более раз. При этом учитывать все нарушения, независимо от того оплачены они или нет. Информацию отсортировать в алфавитном порядке, сначала по фамилии водителя, потом по номеру машины и, наконец, по нарушению.

```
SELECT name, number_plate, violation
FROM fine
GROUP BY name, number_plate, violation
HAVING COUNT(sum_fine) >= 2
ORDER BY name, number_plate, violation
```

(5) В таблице fine увеличить в два раза сумму неоплаченных штрафов для отобранных на предыдущем шаге записей. 

```
UPDATE fine, 
    (
    SELECT name, number_plate, violation
    FROM fine
    GROUP BY name, number_plate, violation
    HAVING COUNT(sum_fine) >= 2
    ) query_in
SET sum_fine = 2 * sum_fine
WHERE fine.name = query_in.name and fine.number_plate = query_in.number_plate and fine.date_payment is null;

SELECT * from fine;
```