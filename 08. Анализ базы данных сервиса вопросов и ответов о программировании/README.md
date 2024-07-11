# Анализ базы данных сервиса вопросов и ответов о программировании

## Задача

Провести анализ с помощью SQL запросов сервиса вопросов и ответов о программировании.

## Навыки и инструменты

- SQL
- PostgreSQL

## Схема базы данных:

![bd2](https://github.com/DaniilKur/Yandex.Practicum_Data_Analyst/blob/main/Image%20(1).png)

### 1. Найдем количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».

```sql
SELECT COUNT(*)
FROM stackoverflow.posts
WHERE post_type_id=1 AND (favorites_count >=100 OR score> 300) 
```

### 2. Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? 

```sql
WITH 
    an AS (SELECT creation_date::date date,
          COUNT(ID) AS avg
          FROM stackoverflow.posts
          where post_type_id=1 AND creation_date::date BETWEEN '2008-11-01' AND '2008-11-18'
          GROUP BY creation_date::date)
SELECT ROUND(AVG(avg), 0)
FROM an
```

### 3. Сколько пользователей получили значки сразу в день регистрации? 

```sql
SELECT  COUNT(DISTINCT user_id)
FROM stackoverflow.users AS u
JOIN stackoverflow.badges AS b ON u.id=b.user_id
WHERE b.creation_date::date=u.creation_date::date
```

### 4. Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?

```sql
SELECT COUNT(DISTINCT p.id)
FROM stackoverflow.posts p
JOIN stackoverflow.users u ON p.user_id=u.id
RIGHT JOIN stackoverflow.votes v ON p.id=v.post_id
WHERE display_name='Joel Coehoorn' 
```

### 5. Выгрузим все поля таблицы vote_types. Добавим к таблице поле rank, в которое войдут номера записей в обратном порядке

```sql
SELECT *,
ROW_NUMBER() OVER(ORDER BY id DESC) AS rank
FROM stackoverflow.vote_types
ORDER BY rank DESC
```

### 6. Отберем 10 пользователей, которые поставили больше всего голосов типа Close. 

```sql
SELECT DISTINCT user_id,
COUNT(*) c
FROM stackoverflow.votes v
JOIN stackoverflow.vote_types t ON t.id=v.vote_type_id
WHERE name ='Close'
GROUP BY user_id
ORDER BY c DESC, user_id DESC
LIMIT 10
```

### 7. Отберем 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.

```sql
SELECT DISTINCT user_id,
COUNT(id),
DENSE_RANK() OVER (ORDER BY COUNT(id) DESC)
FROM stackoverflow.badges
WHERE creation_date::DATE BETWEEN '2008-11-15' AND '2008-12-15'
GROUP BY user_id
ORDER BY COUNT(id) DESC, user_id 
LIMIT 10
```

### 8. Сколько в среднем очков получает пост каждого пользователя?

```sql
SELECT title, user_id,
score,
ROUND(AVG(score) OVER(PARTITION BY user_id))
FROM stackoverflow.posts
where title IS NOT NULL AND score!=0
```

### 9. Отобразим заголовки постов, которые были написаны пользователями, получившими более 1000 значков

```sql
WITH
    p AS (SELECT DISTINCT user_id
    FROM stackoverflow.badges
    GROUP BY user_id
    HAVING count(*) >1000)
SELECT title
FROM stackoverflow.posts b
JOIN p ON b.user_id=p.user_id
WHERE title IS NOT NULL
```

### 10. Напишим запрос, который выгрузит данные о пользователях из Канады (англ. Canada).

```sql
SELECT id,views,
CASE 
WHEN views >= 350 THEN 1
WHEN views >= 100 AND views < 350 THEN 2
ELSE 3
END AS group
FROM stackoverflow.users
WHERE location LIKE '%Canada%' AND views>0

```

### 11. Дополним предыдущий запрос. Отобразим лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. 

```sql
WITH grp AS (SELECT g.id,
                    g.views,
                    g.group,
                    MAX(g.views) OVER (PARTITION BY g.group) AS max     
             FROM (SELECT id,
                          views,
                          CASE
                             WHEN views >= 350 THEN 1
                             WHEN views < 100 THEN 3
                             ELSE 2
                          END AS group
                   FROM stackoverflow.users
                   WHERE location LIKE '%Canada%' AND views > 0) as g
              )
  
SELECT grp.id, 
       grp.views,  
       grp.group
FROM grp
WHERE grp.views = grp.max
ORDER BY grp.views DESC, grp.id;
```

### 12. Посчитаем ежедневный прирост новых пользователей в ноябре 2008 года. 

```sql
WITH 
    N AS (SELECT EXTRACT (DAY FROM creation_date)  date,
    COUNT(*) as new
    FROM stackoverflow.users
    WHERE creation_date::date BETWEEN '2008-11-01' and '2008-11-30'
    group by date)
SELECT *,
SUM(new) OVER (ORDER BY date)
FROM N
```

### 13. Для каждого пользователя, который написал хотя бы один пост, найдем интервал между регистрацией и временем создания первого поста.

```sql
WITH dt AS (SELECT DISTINCT user_id,
                            MIN(creation_date) OVER (PARTITION BY user_id) AS min_dt      
            FROM stackoverflow.posts)

SELECT dt.user_id,
       (dt.min_dt - u.creation_date) AS diff
FROM stackoverflow.users AS u 
JOIN dt ON  u.id = dt.user_id;
```

### 14. Выведим общую сумму просмотров у постов, опубликованных в каждый месяц 2008 года. 

```sql
SELECT DATE_TRUNC('MONTH', creation_date)::date date,
SUM(views_count)
FROM stackoverflow.posts
WHERE EXTRACT(YEAR FROM DATE_TRUNC('MONTH', creation_date))=2008
GROUP BY date
HAVING SUM(views_count)!=0
ORDER BY SUM(views_count) DESC
```

### 15. Выведим имена самых активных пользователей, которые в первый месяц после регистрации (включая день регистрации) дали больше 100 ответов.

```sql
SELECT display_name,
COUNT(DISTINCT user_id)
FROM stackoverflow.users u
JOIN stackoverflow.posts p ON p.user_id=u.id
WHERE post_type_id = 2 AND  p.creation_date::date BETWEEN u.creation_date::date AND (u.creation_date::date + INTERVAL '1 month') 
GROUP BY u.display_name
HAVING COUNT(p.id) > 100
ORDER BY u.display_name;
```

### 16. Выведим количество постов за 2008 год по месяцам. Отберем посты от пользователей, которые зарегистрировались в сентябре 2008 года и сделали хотя бы один пост в декабре того же года. 

```sql
WITH 
    idm as (SELECT u.id
FROM stackoverflow.users u
JOIN stackoverflow.posts p ON p.user_id=u.id
WHERE EXTRACT(YEAR FROM u.creation_date)=2008 AND EXTRACT(YEAR FROM P.creation_date)=2008 AND EXTRACT(MONTH FROM u.creation_date)=9 AND EXTRACT(MONTH FROM p.creation_date)=12    
)
SELECT DATE_TRUNC('month', creation_date)::date date,
count(*)
FROM stackoverflow.posts
WHERE user_id IN (SELECT id FrOM idm)
GROUP BY date
ORDER BY date DESC
```

### 17. Используя данные о постах, выведим несколько полей:

- идентификатор пользователя, который написал пост;
- дата создания поста;
- количество просмотров у текущего поста;
- сумма просмотров постов автора с накоплением.


```sql
SELECT user_id,
creation_date,
views_count,
SUM(views_count) OVER (PARTITION BY user_id ORDER BY creation_date)
FROM stackoverflow.posts
ORDER BY user_id
FROM i;
```

### 18. Сколько в среднем дней в период с 1 по 7 декабря 2008 года включительно пользователи взаимодействовали с платформой?

```sql
WITH 
    top AS (SELECT  user_id, COUNT(DISTINCT creation_date::date) cout
           FROM stackoverflow.posts
           WHERE creation_date::date BETWEEN '2008-12-01' AND '2008-12-07'
           GROUP BY user_id)
SELECT ROUND(AVG(cout))
FROM top
```

### 19. На сколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года? 

```sql
WITH 
n AS 
(SELECT DISTINCT EXTRACT(MONTH FROM creation_date) date,
COUNT(id) OVER (PARTITION BY EXTRACT(MONTH FROM creation_date)) cout
FROM stackoverflow.posts
WHERE creation_date BETWEEN '2008-09-01' AND '2008-12-31')
SELECT *,
    ROUND(((cout::numeric / LAG(cout) OVER (ORDER BY date)) - 1) * 100, 2) AS user_growth
FROM n

```

### 20. Найдем пользователя, который опубликовал больше всего постов за всё время с момента регистрации. 

```sql
WITH 
top as  (SELECT DISTINCT user_id, COUNT(p.id) AS cout
    FROM stackoverflow.users u
    JOIN stackoverflow.posts p ON p.user_id=u.id
         GROUP BY user_id
    ORDER BY cout DESC
    LIMIT 1),
 top2 AS (SELECT user_id, 
          creation_date,
          extract('week' from creation_date) AS week_number
         FROM stackoverflow.posts 
         WHERE user_id IN (SELECT user_id From top) AND EXTRACT(MONTH FROM creation_date)=10)
SELECT DISTINCT week_number::numeric,
       MAX(creation_date) OVER (PARTITION BY week_number) AS post_dt
FROM top2
ORDER BY week_number;
```

## Вывод:

Проведен анализ с помощью SQL запросов
