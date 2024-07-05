# Проект по SQL

## Схема базы данных:

![bd2]([https://github.com/DaniilKur/Yandex.Practicum_Data_Analyst/blob/main/Image.png](https://github.com/DaniilKur/Yandex.Practicum_Data_Analyst/blob/main/Image%20(2).png)

## Описание данных

Таблица books содержит данные о книгах:
- book_id — идентификатор книги;
- author_id — идентификатор автора;
- title — название книги;
- num_pages — количество страниц;
- publication_date — дата публикации книги;
- publisher_id — идентификатор издателя.
  
Таблица authors содержит данные об авторах:
- author_id — идентификатор автора;
- author — имя автора.
  
Таблица publishers содержит данные об издательствах:
- publisher_id — идентификатор издательства;
- publisher — название издательства.
  
Таблица ratings содержит данные о пользовательских оценках книг:
- rating_id — идентификатор оценки;
- book_id — идентификатор книги;
- username — имя пользователя, оставившего оценку;
- rating — оценка книги.

Таблица reviews содержит данные о пользовательских обзорах:
- review_id — идентификатор обзора;
- book_id — идентификатор книги;
- username — имя автора обзора;
- text — текст обзора.
