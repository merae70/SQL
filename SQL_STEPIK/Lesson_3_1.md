(1) Вывести студентов (различных студентов), имеющих максимальные результаты попыток. Информацию отсортировать в алфавитном порядке по фамилии студента.

```
SELECT name_student, result
FROM attempt INNER JOIN student USING(student_id)
WHERE result IN (
    SELECT first_query.result
    FROM (
        SELECT student_id, result
        FROM attempt
        ORDER BY result DESC
    ) first_query
    INNER JOIN (
        SELECT student_id, result
        FROM attempt
        ORDER BY result DESC
        LIMIT 1
    ) second_query
    ON first_query.result = second_query.result
)
ORDER BY name_student;
```

(2) Если студент совершал несколько попыток по одной и той же дисциплине, то вывести разницу в днях между первой и последней попыткой. В результат включить фамилию и имя студента, название дисциплины и вычисляемый столбец Интервал. Информацию вывести по возрастанию разницы. Студентов, сделавших одну попытку по дисциплине, не учитывать. 
```
SELECT name_student, name_subject, DATEDIFF(MAX(date_attempt), MIN(date_attempt)) AS Интервал
FROM
    student
    INNER JOIN attempt USING(student_id)
    INNER JOIN subject USING(subject_id)
    
GROUP BY name_student, name_subject
HAVING Интервал <> 0
ORDER BY Интервал;
```
(3) Вывести вопросы, которые были включены в тест для Семенова Ивана по дисциплине «Основы SQL» 2020-05-17  (значение attempt_id для этой попытки равно 7). Указать, какой ответ дал студент и правильный он или нет (вывести Верно или Неверно). В результат включить вопрос, ответ и вычисляемый столбец  Результат.

```
SELECT name_question, name_answer, IF(is_correct, 'Верно', 'Неверно') AS Результат
FROM 
    question 
    INNER JOIN testing USING(question_id)
    INNER JOIN answer USING(answer_id)
WHERE attempt_id = 7;
```