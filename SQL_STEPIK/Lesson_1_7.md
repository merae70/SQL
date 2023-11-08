(1) Создать таблицу fine
```
CREATE TABLE fine(
    fine_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(30),
    number_plate VARCHAR(6),
    violation VARCHAR(50),
    sum_fine DECIMAL(8, 2),
    date_violation DATE,
    date_payment DATE
)
```

(2) Занести в таблицу fine суммы штрафов, которые должен оплатить водитель, в соответствии с данными из таблицы traffic_violation. При этом суммы заносить только в пустые поля столбца  sum_fine.

Таблица traffic_violation создана и заполнена.

```
UPDATE fine AS f, traffic_violation AS tv
SET f.sum_fine = tv.sum_fine
WHERE f.violation = tv.violation AND f.sum_fine is NULL;

SELECT * FROM fine
```

(3) Вывести фамилию, номер машины и нарушение только для тех водителей, которые на одной машине нарушили одно и то же правило   два и более раз. При этом учитывать все нарушения, независимо от того оплачены они или нет. Информацию отсортировать в алфавитном порядке, сначала по фамилии водителя, потом по номеру машины и, наконец, по нарушению.

```
SELECT name, number_plate, violation
FROM fine
GROUP BY name, number_plate, violation
HAVING COUNT(sum_fine) >= 2
ORDER BY name, number_plate, violation
```

(4) В таблице fine увеличить в два раза сумму неоплаченных штрафов для отобранных на предыдущем шаге записей. 

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

(5) Удалить из таблицы fine информацию о нарушениях, совершенных раньше 1 февраля 2020 года. 
```
DELETE FROM fine
WHERE (MONTH(date_violation) < 2 AND YEAR(date_violation) <= 2020) OR (YEAR(date_violation) < 2020);

SELECT * FROM fine;
```