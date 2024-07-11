# Анализ рынка инвестиций

## Задание

Провести анализ  с помощью SQL запросов базы данных, которая хранит информацию о венчурных фондах и инвестициях в компании-стартапы.

## Навыки и инструменты

- SQL
- PostgreSQL

## Схема базы данных:

![bd](https://github.com/DaniilKur/Yandex.Practicum_Data_Analyst/blob/main/Image.png)

### 1. Отобразим все записи из таблицы company по компаниям, которые закрылись

```sql
SELECT * /n
FROM company
WHERE status='closed'
```

### 2. Отобразим количество привлечённых средств для новостных компаний США

```sql
SELECT funding_total
FROM company
WHERE category_code='news' and country_code='USA'
ORDER BY funding_total DESC
```

### 3. Найдем общую сумму сделок по покупке одних компаний другими в долларах. Отберкм сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно

```sql
SELECT SUM(price_amount)
FROM acquisition
WHERE term_code='cash' AND EXTRACT(YEAR FROM CAST(acquired_at AS date)) IN (2011, 2012, 2013)
```

### 4. Отобразим имя, фамилию и названия аккаунтов людей в поле network_username, у которых названия аккаунтов начинаются на 'Silver'

```sql
SELECT first_name, 
last_name, 
network_username
FROM people
WHERE network_username LIKE ('Silver%')
```

### 5. Выведим на экран всю информацию о людях, у которых названия аккаунтов в поле network_username содержат подстроку 'money', а фамилия начинается на 'K'

```sql
SELECT *
FROM people
WHERE network_username LIKE ('%money%') AND last_name LIKE ('K%')
```

### 6. Для каждой страны отобразим общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране.

```sql
SELECT country_code,
SUM(funding_total)
FROM company
GROUP BY country_code
ORDER BY SUM(funding_total) DESC
```

### 7. Составим таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату. Оставим в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.

```sql
SELECT funded_at,
MIN(raised_amount),
MAX(raised_amount)
FROM funding_round
GROUP BY funded_at
HAVING MIN(raised_amount)!=0 and MIN(raised_amount)!=MAX(raised_amount)
```

### 8 .Создим поле с категориями: 
- Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
- Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
- Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
  
Отобразим все поля таблицы fund и новое поле с категориями.

```sql
SELECT *,
CASE
WHEN invested_companies >= 100 THEN 'high_activity'
WHEN invested_companies < 100 AND invested_companies >= 20 THEN 'middle_activity'
WHEN invested_companies < 20 THEN 'low_activity'
END
FROM fund
```

### 9. Для каждой из категорий, назначенных в предыдущем задании, посчитаем округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие

```sql
SELECT 
       CASE
           WHEN invested_companies>=100 THEN 'high_activity'
           WHEN invested_companies>=20 THEN 'middle_activity'
           ELSE 'low_activity'
       END AS activity,
       ROUND(AVG(investment_rounds))
FROM fund
GROUP BY activity
ORDER BY ROUND(AVG(investment_rounds)) ;
```

### 10. Выгрузим десять самых активных стран-инвесторов. Затем добавим сортировку по коду страны в лексикографическом порядке.

```sql
SELECT country_code,
       MIN(invested_companies),
       MAX(invested_companies),
       AVG(invested_companies)
FROM fund
WHERE EXTRACT(YEAR FROM founded_at) IN (2010, 2011, 2012)
GROUP BY country_code
HAVING MIN(invested_companies)!=0
ORDER BY AVG(invested_companies) DESC, country_code
LIMIT 10

```

### 11. Отобразим имя и фамилию всех сотрудников стартапов. Добавим поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

```sql
SELECT first_name,
       last_name,
       instituition
FROM people AS p
LEFT JOIN education AS e ON p.id=e.person_id
```

### 12. Для каждой компании найдем количество учебных заведений, которые окончили её сотрудники. Составим топ-5 компаний по количеству университетов.

```sql
SELECT c.name,
       COUNT(DISTINCT e.instituition)
FROM people AS p
JOIN company AS c ON c.id=p.company_id
JOIN education AS e ON p.id=e.person_id
GROUP BY c.name
ORDER BY COUNT(DISTINCT e.instituition) DESC
LIMIT 5
```

### 13. Составим список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

```sql
SELECT DISTINCT(c.name)
FROM company AS c
WHERE status = 'closed' AND id IN (SELECT company_id
                                   FROM funding_round
                                   WHERE is_first_round = 1 AND is_last_round = 1);
```

### 14. Составим список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

```sql
SELECT DISTINCT id
FROM people
WHERE company_ID in
(SELECT DISTINCT(id)
FROM company AS c
WHERE status = 'closed' AND id IN (SELECT company_id
                                   FROM funding_round
                                   WHERE is_first_round = 1 AND is_last_round = 1))
```

### 15. Составим таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.

```sql
SELECT DISTINCT(p.id), instituition
FROM education AS e
JOIN people AS p ON e.person_id=p.id 
WHERE person_id IN 
(SELECT DISTINCT id
FROM people
WHERE company_ID in
(SELECT DISTINCT(id)
FROM company AS c
WHERE status = 'closed' AND id IN (SELECT company_id
                                   FROM funding_round
                                   WHERE is_first_round = 1 AND is_last_round = 1)))
```

### 16. Посчитаем количество учебных заведений для каждого сотрудника из предыдущего задания

```sql
SELECT DISTINCT(p.id), COUNT(e.instituition)
FROM people AS p
LEFT JOIN education AS e ON p.id=e.person_id
WHERE company_id IN (SELECT id
                     FROM company AS c
                     WHERE status = 'closed' 
                     AND id IN (SELECT company_id
                                FROM funding_round
                                WHERE is_first_round = 1 
                                AND is_last_round = 1))
                  AND e.instituition IS NOT NULL
GROUP BY p.id;
```

### 17. Дополним предыдущий запрос и выведим среднее число учебных заведений, которые окончили сотрудники разных компаний

```sql
WITH
i AS (SELECT COUNT(e.instituition) AS count
FROM people AS p
LEFT JOIN education AS e ON p.id=e.person_id
WHERE company_id IN (SELECT id
                     FROM company AS c
                     WHERE status = 'closed' 
                     AND id IN (SELECT company_id
                                FROM funding_round
                                WHERE is_first_round = 1 
                                AND is_last_round = 1))
                  AND e.instituition IS NOT NULL
GROUP BY p.id)
SELECT AVG(count)
FROM i;
```

### 18. Напишим похожий запрос: выведим среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Socialnet.

```sql
WITH
i AS (SELECT COUNT(e.instituition) AS count
FROM people AS p
LEFT JOIN education AS e ON p.id=e.person_id
WHERE company_id IN (SELECT id
                     FROM company AS c
                     WHERE name='Socialnet' 
                     AND id IN (SELECT company_id
                                FROM funding_round
))
                  AND e.instituition IS NOT NULL
GROUP BY p.id)
SELECT AVG(count)
FROM i;
```

### 19. Составим таблицу из полей:
- name_of_fund — название фонда;
- name_of_company — название компании;
- amount — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.

```sql
SELECT c.name AS name_of_company,
       f.name AS name_of_fund,
       fr.raised_amount AS amount
FROM fund AS f
JOIN investment AS i ON f.id=i.fund_id
JOIN company AS c ON i.company_id=c.id
JOIN funding_round AS fr ON fr.id=i.funding_round_id
WHERE c.milestones > 6 AND EXTRACT(YEAR FROM fr.funded_at) IN (2012, 2013)
```

### 20. Выгрузим таблицу, в которой будут поля:
- название компании-покупателя;
- сумма сделки;
- название компании, которую купили;
- сумма инвестиций, вложенных в купленную компанию;
- доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.

```sql
SELECT c.name AS acquiring_name,
       a.price_amount,
       c2.name AS acquired_name,
       c2.funding_total,
       ROUND((a.price_amount / c2.funding_total))

FROM acquisition AS a
LEFT JOIN company AS c ON a.acquiring_company_id=c.id
LEFT JOIN company AS c2 ON a.acquired_company_id=c2.id
WHERE a.price_amount != 0 AND c2.funding_total != 0
ORDER BY  a.price_amount DESC, c2.name
LIMIT 10;
```

### 21. Выгрузим таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно.

```sql
WITH
s AS (SELECT id, 
     name
     FROM company
     WHERE category_code ='social'),
f AS (SELECT company_id, EXTRACT(MONTH FROM funded_at) AS month
     FROM funding_round 
     WHERE EXTRACT(YEAR FROM funded_at) IN (2010, 2011, 2012, 2013) AND 
     raised_amount !=0)
SELECT name, month
FROM s
JOIN f ON s.id=f.company_id
```

### 22. Отберим данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируем данные по номеру месяца и получим таблицу, в которой будут поля:
- номер месяца, в котором проходили раунды;
- количество уникальных названий фондов из США, которые инвестировали в этом месяце;
- количество компаний, купленных за этот месяц;
- общая сумма сделок по покупкам в этом месяце.

```sql
WITH

a AS (SELECT EXTRACT(MONTH FROM acquired_at) AS month,
          COUNT(acquired_company_id) as acq,
          SUM(price_amount) as total
      FROM acquisition
      WHERE EXTRACT(YEAR FROM acquired_at) BETWEEN 2010 AND 2013
      GROUP BY month),
      
fd AS (SELECT EXTRACT(MONTH FROM funded_at) AS month, id
       FROM funding_round
       WHERE EXTRACT(YEAR FROM funded_at) BETWEEN 2010 AND 2013),
       
i AS (SELECT funding_round_id, fund_id
      FROM investment
      WHERE funding_round_id IN (SELECT id
                                 FROM funding_round
       WHERE EXTRACT(YEAR FROM funded_at) BETWEEN 2010 AND 2013)
       AND fund_id IN (SELECT DISTINCT(id)
                       FROM fund
                       WHERE country_code = 'USA')),
                       
c1 AS (SELECT month, COUNT(distinct(fund_id)) as fname
       FROM fd
       FULL OUTER JOIN i ON fd.id=i.funding_round_id
       GROUP BY month)                       
      
SELECT c1.month, fname, acq, total
FROM c1
LEFT JOIN a ON c1.month=a.month;
```

### 23. Составим сводную таблицу и выведим среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах

```sql
WITH
     inv_2011 AS (SELECT country_code,
                 AVG(funding_total) total_inv_2011
                 FROM company
                 WHERE EXTRACT(YEAR FROM founded_at)= 2011
                 GROUP BY country_code),
     inv_2012 AS (SELECT country_code,
                 AVG(funding_total) total_inv_2012
                 FROM company
                 WHERE EXTRACT(YEAR FROM founded_at)= 2012
                 GROUP BY country_code),  
     inv_2013 AS (SELECT country_code,
                 AVG(funding_total) total_inv_2013
                 FROM company
                 WHERE EXTRACT(YEAR FROM founded_at)= 2013
                 GROUP BY country_code)
SELECT inv_2011.country_code,
        total_inv_2011,
        total_inv_2012,
        total_inv_2013
FROM inv_2011
INNER JOIN inv_2012 ON inv_2011.country_code=inv_2012.country_code
INNER JOIN inv_2013 ON inv_2011.country_code=inv_2013.country_code
ORDER BY total_inv_2011 DESC
```

## Вывод:

Проведен анализ с помощью SQL запросов
