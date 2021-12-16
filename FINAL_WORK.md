# Финальная работа #
Студент: Винников Алексей

Преподаватель: Моргунов Евгений Павлович

Группа: М8О-203М-20

Тема: Книжный магазин

## Описание предметной области ##

В качестве предметной области был выбран "Книжный магазин", по мотивом реального магазина "Библио-Глобус". Покупатель всегда может воспользоваться поиском и узнать на какой полке в каком шкафу и в каком зале лежит интересующая его книга. Автоматизация и контроль положения книг на полках должен быть основной фишкой данной базой данных. В качестве исходных данных имеем данные книжек: авторов, категори, названия, ISBN, рейтинг у читателей, количество страниц, толщину книжки, цену и прочее.

Исходя из замысла данная БД значительно упростит жизнь как сотрудникам "Книжного магазина" так и ее покупателям позволяя с легкостью ориентироваться в местоположении всех книг, например в довольно большом магазине с более чем 100 тысяч книжек на полках.

## Требования к БД ##

База данных должна автоматически контролировать переполнение полок и не позволять сотрудникам магазина, помещать книги куда они заведомо, исходя из хранимых данных, не могут поместиться. Данный контроль необходимо осуществлять на основе данных о толщине книг. 

Кроме того необходимо организовать двухуровнению иерархиию категорий книг. Так, чтобы основная категория могла иметь несколько дочерних, а дочерняя несколько основных. При этом глубина иеерархии не должна быть глубже 2 уровней.

Также необходимо запрерить излишние манипуляцие со шкафами полками, которые нарушают целостность всей схемы и ее актульность (например сломанная полка, по задумке просто освобождается, тип шафа нельзя поменять когда он заполнен и т.д.)
## Требования к данным ##
Книги имеют следующие характеристи:
- ISBN
- Название
- Автор
- Краткое описание
- Категория
- Средняя оценка у покупателей
- Количество оценок у покупателей 
- Год публикации
- Количество страниц
- Толщина
- Цена

Каталог книг должен существовать отдельно от нахождения на полках (если такой книги нет на полке значит ее просто нет в наличии).

Категории могут быть основными и дочерними

У одного автора может быть несколько книг, у книги - несколько авторов

В каждом зале может быть много шкафов 

Шкафы имеют какой-то тип (фиксированая ширина и кол-во полок)

Полки неразрывно связаны со шкафом, нельзя просто убрать или добавить еще одну полку в шкаф. При удалении шкафа удаляются полки, при создании - создаются.

Книги помещаются на полки в зависимости от совободного места на полках, тем не менее экземпляры одной книги должны находиться на одной полке.

## Требования к транзациям ##

- Добавление\Изменение\удаление данных из каталога (кроме изменения толщины книжки)
- Добавление\удаление шкафа (при этом сразу же создатся\удаляются полки)
- При добавлении\изъятия книжки с полки занимаемое место на полке должно  меняется соответсвественно толщины книжки
- Изменение рейгтинга книжки путем обновление голосов с вычитанием среднего значения от 0 до 5
- Просмотр всех книжек по категориям, как основным(включая дочерние) так и дочерним
- Поиск расположения интересующей книги
- Различные фильтры и статистика по каталогу

## Логическая модель ##
<center>
<img src="./images/log_model.jpg " />
<p>Логическая модель БД</p>
</center>

## Физическая модель ##

<center>
<img src="./images/fith_model.jpg " />
<p>Физическая модель БД</p>
</center>

Как видно из схемы все связи в таблицах озозначаются как 1 ко многим. 

Все таблицы находятся в 3НФ. Так как:
1. Ни один неключевой атрибут таблицы не находится в транзитивной функциональной зависимости от потенциального ключа 
2. Каждый неключевой атрибут неприводимо зависит от (каждого) потенциального ключа таблицы
3. Все  атрибуты таблиц атомарны (их нельзя разделить на более простые)

## Реализация ##
**Requiremnts**
- Docker
- Docker-compose

**Запуск сервера БД**

```bash 
$ docker-compose up -d 
```
**Подключение к серверу**
```bash 
$ docker exec -it final-server /usr/bin/psql -U root -d final
```

### Таблицы ###
**Таблица "Категории книг"**

```PGSQL
CREATE TABLE IF NOT EXISTS categories
(
    category VARCHAR(50) PRIMARY KEY NOT NULL,                  -- первичный ключ, категория
    description text,                                           -- описание категории
    CHECK(description IS NULL OR LENGTH(description) > 10)      -- проверяем что описания категории нет или ее длина достаточна
);
```
**Таблица "Иеерархия категорий книг"** 

```PGSQL
CREATE TABLE IF NOT EXISTS categories_hierarchy
(
    id SERIAL PRIMARY KEY NOT NULL,                             -- автоматический индекс
    category VARCHAR(50) NOT NULL,                              -- субкатегория
    dom_category VARCHAR(50) NOT NULL,                          -- основная категория
    UNIQUE(category,dom_category),                              -- пара должна быть уникальной
    FOREIGN KEY ( category )
        REFERENCES categories ( category ),
    FOREIGN KEY ( dom_category )
        REFERENCES categories ( category )
);
```
**Таблица "Книги"**

```PGSQL
CREATE TABLE IF NOT EXISTS books
(
    isbn VARCHAR(13) PRIMARY KEY NOT NULL,                      -- 13 значный ISBN книги
    title text NOT NULL,                                        -- Заголовок книги
    category VARCHAR(50) NOT NULL,                              -- Категория книги
    description text,                                           -- Описание книги
    published_year SMALLINT NOT NULL,                           -- Год публикации
    average_rating NUMERIC NOT NULL DEFAULT 0.0,                -- Средняя оценка покупателей
    num_pages  INT NOT NULL,                                    -- Количество страниц
    ratings_count  INT NOT NULL DEFAULT 0.0,                    -- Количество оценок книги покупателями 
    book_thickness  INT NOT NULL,                               -- Толщина книжки
    price NUMERIC NOT NULL,                                     -- Цена книги
    CHECK(isbn ~ $$[0-9]{13}$$),                                -- Проверяем что ISBN состоит из 13 цифр
    CHECK(LENGTH(TRIM(title)) > 0),                             -- Проверяем что заголовок не пустой
    CHECK(description IS NULL OR LENGTH(description) > 0),      -- Проверяем что описания либо нет либо оно не очень короткое
    CHECK(published_year >= 1445 AND                            -- Дата публикации книги от изобретения печатного станка до текущей даты
          published_year <= date_part('year', CURRENT_DATE)),
    CHECK(average_rating >= 0.0 AND average_rating <= 5.0),     -- Рейтинг от 0 до 5 
    CHECK(num_pages > 0),                                       -- Проверка числа страниц
    CHECK(ratings_count >= 0),                                  -- Отзывов может и не быть
    CHECK(book_thickness > 0),                                  -- Толщина книги играет очень вжную роль и не может быть нулевой
    CHECK(price > 0.0),                                         -- Цены не могут быть отрицательными
    FOREIGN KEY ( category )
            REFERENCES categories ( category )
);
```

**Таблица "Авторы книг"**
```PGSQL
CREATE TABLE IF NOT EXISTS books_author
(
    id SERIAL PRIMARY KEY NOT NULL,                             -- Автоматический индекс      
    isbn VARCHAR(13) NOT NULL,                                  -- ISBN книги
    author text NOT NULL,                                       -- Один из ее авторов
    CHECK(LENGTH(TRIM(author)) > 0),                            -- ФИО или псевдоним автора не должно быть меньше 1 символа
    UNIQUE(isbn,author),                                        -- Пара автор-книга должна быть уникальна
    FOREIGN KEY ( isbn )
                REFERENCES books ( isbn )
);
```

**Таблица "Типы книжных шкафов"**
```PGSQL
CREATE TABLE IF NOT EXISTS bookcase_types
(
    type VARCHAR(20) PRIMARY KEY NOT NULL,                      -- Тип шкафа
    width  SMALLINT NOT NULL,                                   -- Ширина полки в мм
    shelf_numbers SMALLINT NOT NULL,                            -- Количество полок в шкафу
    CHECK(width >= 500 AND width <= 10000),                     -- Допустимая ширина полки шкафа от половины до 10 метров
    CHECK(shelf_numbers >= 3 AND shelf_numbers <= 20)           -- Количество полок варируется от 3 до 20
);

```

**Таблица "Книжные шкафы"**
```PGSQL
CREATE TABLE IF NOT EXISTS bookcases
(
    hall SMALLINT NOT NULL,                                     -- Номер зала в котором размещен шкаф
    bookcase SMALLINT NOT NULL,                                 -- Номер шкафа в зале
    bookcase_type VARCHAR(20) NOT NULL,                         -- Тип шкафа
    PRIMARY KEY(hall, bookcase),                                -- Пара зал-шкаф должна быть уникальна
    FOREIGN KEY ( bookcase_type )
        REFERENCES bookcase_types ( type ),
    CHECK( hall > 0 AND hall < 100 ),                           -- Номер зала от 0 до 100
    CHECK( bookcase > 0 AND bookcase < 200 )                    -- Шкафов в зале не может быть больше 200
);

```

**Таблица "Полки в книжных шкафах"**
```PGSQL
CREATE TABLE IF NOT EXISTS shelves
(
    hall SMALLINT NOT NULL,                                     -- Номер зала в котором размещен шкаф
    bookcase SMALLINT NOT NULL,                                 -- Номер шкафа в зале
    shelf SMALLINT NOT NULL,                                    -- Номер полки в шкафу
    occupied_width SMALLINT NOT NULL DEFAULT 0,                 -- Занятое книжками место на полке в мм
    PRIMARY KEY(hall, bookcase, shelf),                         -- Ключ каждой полки также включает в себя шкаф и зал
    FOREIGN KEY (hall, bookcase)                                -- Удаляем шкаф - удалем и полки
        REFERENCES bookcases ( hall, bookcase ) ON DELETE CASCADE,
    CHECK( occupied_width >= 0)                                 -- Занятое книжками место на полке не может быть отрицательным
);
```

**Таблица "Книжки на полках"**
```PGSQL
CREATE TABLE IF NOT EXISTS books_shelf
(
    id SERIAL PRIMARY KEY NOT NULL,       
    isbn VARCHAR(50) NOT NULL,                                  -- ISBN книжки
    hall SMALLINT NOT NULL,                                     -- Номер зала, где лежит книжка
    bookcase SMALLINT NOT NULL,                                 -- Номер шкафа, в котором лежит книжка
    shelf SMALLINT NOT NULL,                                    -- Номер полки, на которой лежит книжка
    numbers SMALLINT NOT NULL,                                  -- Число книг на полке
    UNIQUE(isbn),                                               -- Экземпляры одной книги должны всегда находиться на одной полке 
    FOREIGN KEY ( isbn )
        REFERENCES books ( isbn ),
    FOREIGN KEY ( hall,bookcase,shelf )
        REFERENCES shelves ( hall,bookcase,shelf ),
    CHECK(numbers >= 0)                                         -- Допускаем хранение 0 книжек на полке (такие строки должны удаляться тригерной функцией)
);
```

### Обеспечение целостности через триггерные функции ###

**Корректность иерархии категории**

Начнем с иерархии категорий, как уже было упомянуто основная категория может иметь подкатегорию, при этом подкатегория не может иметь дочерних подкатегорий, но может иметь несколько основных. Реализуем триггерную функцию которая будет проверять этот аспект при добавлении нового отношения в иерархию.

```PGSQL
CREATE OR REPLACE FUNCTION categories_hierarchy_check() RETURNS trigger AS $$
DECLARE
 i1 SMALLINT:=0;
 i2 SMALLINT:=0;
BEGIN
-- Получим количество строк где основная категория добавляемого отношения является дочерней
SELECT count(*) INTO i1 FROM categories_hierarchy WHERE category = NEW.dom_category;
-- Получим количество строк где дочерняя категория добавляемого отношения является основной
SELECT count(*) INTO i2 FROM categories_hierarchy WHERE dom_category = NEW.category;

IF( i1 <> 0  OR i2 <> 0) THEN  -- Если есть хоть 1 такое отношение - некорректная иерархия
  RAISE NOTICE '% % % %', i1, i2, NEW.dom_category, NEW.category;
  RAISE EXCEPTION 'More then 2 hierarchy levels ';
 END IF;
 
 -- При добавлении/обновлении возвращаем новую строку, в остальном NULL
 IF ( TG_OP = 'UPDATE' OR TG_OP = 'INSERT' ) THEN
   RETURN NEW;
 END IF;
 RETURN NULL;
END; $$
LANGUAGE plpgsql;
```
Будем запускать данную функцию при каждом добавлении или изменении таблицы "Иерархия категорий"

```PGSQL
DROP TRIGGER IF EXISTS categories_hierarchy_check ON categories_hierarchy;
CREATE TRIGGER categories_hierarchy_check AFTER INSERT OR UPDATE ON categories_hierarchy
    FOR EACH ROW EXECUTE PROCEDURE categories_hierarchy_check();
```

**Согласованность и корректность информации о полоках в книжных шкафах**

Поскольку полки в шкафах должны создавать вместе с созданием шкафа необходимо заблокировать ручные манипуляции с этой таблицей специальной тригеерной функцией. При этом надо учесть, что обновление данных о занимаем книгами месте на полке не столь критично, но будет происходить очень часто, в отличии от добавления или удаления целого шкафа(это можно сделать например ночью и блокирова всей таблицы на несколько милисекунд никому не помешает). Необходимо пропускать такие запросы.

```PGSQL
CREATE OR REPLACE FUNCTION disable_operations_with_shelves() RETURNS trigger AS $$
BEGIN
    IF (TG_OP = 'UPDATE' AND 
        NEW.hall = OLD.hall AND
        NEW.bookcase = OLD.bookcase AND 
        OLD.shelf = NEW.shelf ) THEN
        -- Запрос на обновление занимаемого книгами места на полке
        IF (NEW.occupied_width <=  (SELECT DISTINCT bookcase_types.width FROM bookcases, bookcase_types 
                    WHERE bookcases.hall=OLD.hall AND bookcases.bookcase = OLD.bookcase  AND bookcases.bookcase_type=bookcase_types.type)) 
        THEN 
        -- При обновлении полка не переполнилась
            return NEW;
        END IF;
    END IF;

    -- Переполнение полки или попытка ручного обновления
    RAISE EXCEPTION 'shelves can be changed only automatic';
    RETURN NULL;
END; $$
LANGUAGE plpgsql;
```
Эта функция будет срабатывать при каждой папытки изменить какую либо строку в таблице "Полки в книжных шкафах"
```PGSQL
DROP TRIGGER IF EXISTS disable_operations_with_shelves ON shelves;
CREATE TRIGGER disable_operations_with_shelves AFTER INSERT OR UPDATE OR DELETE ON shelves 
FOR EACH ROW EXECUTE PROCEDURE disable_operations_with_shelves();
```

Создадим также функции для отключения и включения данной функции (они понадобяться дальше для момента добавления и удаления шкафа)

```PGSQL
CREATE OR REPLACE FUNCTION disable_shelves_blocks() RETURNS trigger AS $$   -- Отключение блокировки обновления таблицы "Полки в книжных шкафах"
BEGIN
    ALTER TABLE shelves DISABLE TRIGGER disable_operations_with_shelves;
    RETURN OLD;
END; $$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION enable_shelves_blocks() RETURNS trigger AS $$    -- Включение блокировки обновления таблицы "Полки в книжных шкафах"
BEGIN
    ALTER TABLE shelves ENABLE TRIGGER disable_operations_with_shelves;
    RETURN OLD;
END; $$
LANGUAGE plpgsql;
```

Реализуем функцию создания полок при добавлении книжного шкафа:
```PGSQL
CREATE OR REPLACE FUNCTION create_shelves_after_creating_bookcase() RETURNS trigger AS $$
DECLARE
    N SMALLINT; -- Число полок в шкафу
BEGIN
    SELECT shelf_numbers INTO N FROM bookcase_types WHERE type = NEW.bookcase_type; -- Найдем число полок исходя из типа добавлямого шкафа  
    
    ALTER TABLE shelves DISABLE TRIGGER disable_operations_with_shelves; -- Отключим блокировку таблицы "Полки в книжных шкафах"
    FOR i IN 1..N LOOP -- добавим N полок
        INSERT INTO shelves(hall,bookcase,shelf) VALUES(NEW.hall, NEW.bookcase,i);
    END LOOP;
    ALTER TABLE shelves ENABLE TRIGGER disable_operations_with_shelves; -- Включим блокировку таблицы "Полки в книжных шкафах"
    RETURN NEW;
END; $$
LANGUAGE plpgsql;

-- Эта функция будет срабатывать каждый раз при добавлении нового шкафа
DROP TRIGGER IF EXISTS create_shelves_after_creating_bookcase ON bookcases;
CREATE TRIGGER create_shelves_after_creating_bookcase AFTER INSERT ON bookcases FOR EACH ROW EXECUTE FUNCTION create_shelves_after_creating_bookcase();
```

Как только мы удаляем шкаф удаляем и полки. Для этого нам надо отключать блокировку таблицы "полки в книжном шкафу" перед попыткой удаления шкафа и включать обратно после
```PGSQL 
DROP TRIGGER IF EXISTS disable_shelves_blocks ON bookcases;
DROP TRIGGER IF EXISTS disable_shelves_blocks ON bookcases;
CREATE TRIGGER disable_shelves_blocks BEFORE DELETE ON bookcases FOR EACH ROW EXECUTE PROCEDURE disable_shelves_blocks();
CREATE TRIGGER disable_shelves_blocks AFTER  DELETE ON bookcases FOR EACH ROW EXECUTE PROCEDURE enable_shelves_blocks();

```
**Согласованность и корректность информации о книжных шкафах и их типах**

Необходимо запретить изменение шкафа (например его типа, тогда нарушмиться все количество полок, размеры свободного место и т.д.). Шкаф можно только удалить или добавить вместе с полками. Аналогично с типом шкафов (его характеристики нельзя менять, только создать/удалить):

```PGSQL
CREATE OR REPLACE FUNCTION disable_update() RETURNS trigger AS $$
BEGIN
   RAISE EXCEPTION '% table cannot be update - only insert/remove', TG_ARGV[0]::VARCHAR;
END; $$
LANGUAGE plpgsql;

-- Блокируем обновления в шкафах
DROP TRIGGER IF EXISTS disable_update_bookcases ON bookcases;
CREATE TRIGGER disable_update_bookcases AFTER UPDATE ON bookcases FOR EACH ROW EXECUTE PROCEDURE disable_update('bookcase');
 
-- Блокируем обновления в типах шкафов
DROP TRIGGER IF EXISTS disable_update_bookcases_types ON bookcase_types;
CREATE TRIGGER disable_update_bookcases_types AFTER UPDATE ON bookcase_types FOR EACH ROW EXECUTE PROCEDURE disable_update('bookcase_types');

```
**Изменение места на полке при изменении числа экземпляров определенной книги**

После того как экзепляры книги разместили на полке/купили/добавили еще необходимо мониторить наличие свободного места и обновлять занятое на полке место. Если его будет не хватать - выбрасывать ошибку

```PGSQL
CREATE OR REPLACE FUNCTION books_shelf_manipulate() RETURNS trigger AS $$
DECLARE
    b_isbn VARCHAR(50);         -- ISBN книжки
    s_hall SMALLINT;            -- Номер зала
    s_bookcase SMALLINT;        -- Номер шкафа
    s_shelf SMALLINT;           -- Номер полки
    numbers SMALLINT;           -- Число добавляемых(>0)/изымаемых(<0) экземпляров книг  
    s_occupied_width SMALLINT;  -- Занимаемое место книгами на полке
    bookcase_width SMALLINT;    
    shalf_width SMALLINT;
BEGIN

    IF (TG_OP = 'INSERT' ) THEN     -- Впервые добавляем экземпляры на полку
        b_isbn:= NEW.isbn;
        s_hall:= NEW.hall;
        s_bookcase:= NEW.bookcase;
        s_shelf:= NEW.shelf;
        numbers:=NEW.numbers;

        IF (NEW.numbers = 0) THEN   -- Нет смысла добавлять 0
            RAISE 'numbers must be greater than 0';
        END IF;

    ELSIF (TG_OP = 'UPDATE') THEN   -- Обновляем число экземпляров например при покупке
        b_isbn:= NEW.isbn;
        s_hall:= NEW.hall;
        s_bookcase:= NEW.bookcase;
        s_shelf:= NEW.shelf;
        numbers:=NEW.numbers - OLD.numbers;

    ELSE                            -- Удаляем
        IF (OLD.numbers = 0) THEN   -- Итак пустой - просто удаляем
            RETURN OLD;
        END IF;
        -- Тут еще надо будет поменять количество места на полке

        b_isbn:= OLD.isbn;
        s_hall:= OLD.hall;
        s_bookcase:= OLD.bookcase;
        s_shelf:= OLD.shelf;
        numbers:=0 - OLD.numbers;

    END IF;

    -- Получаем место занимаемое(>0)/освобождаемое(<0) после обновления информации
    SELECT book_thickness INTO s_occupied_width FROM books WHERE isbn = b_isbn; 
    s_occupied_width := s_occupied_width* numbers;

    IF( s_occupied_width > 0) THEN -- Значит добавляем книги
        
        -- Получим ширину книжной полки в данном шкафу
        SELECT DISTINCT bookcase_types.width INTO bookcase_width FROM bookcases, bookcase_types 
                WHERE bookcases.hall=s_hall AND bookcases.bookcase = s_bookcase AND bookcases.bookcase_type=bookcase_types.type;
        -- Получим текущее занимаемое на этой полке пространство        
        SELECT DISTINCT occupied_width INTO shalf_width FROM shelves WHERE hall = s_hall AND bookcase = s_bookcase AND shelf = s_shelf;
        
        -- Если после добавления суммарное пространство полки будет перполнено того выбрасываем ошибку
        IF (bookcase_width - shalf_width < s_occupied_width ) THEN 
            RAISE 'Not enough shelf space';
        END IF;

        -- Все хорошо. Добавляем экземпляры на полку
        UPDATE shelves SET occupied_width = occupied_width + s_occupied_width WHERE hall = s_hall AND bookcase = s_bookcase AND shelf = s_shelf;

    ELSIF (s_occupied_width < 0) THEN -- Здесь понимаем что мы убираем с полки книги
        -- Пространство полки здесь не переполнить так что, просто вычитаем занимаемое место 
        UPDATE shelves SET occupied_width = occupied_width + s_occupied_width WHERE hall = s_hall AND bookcase = s_bookcase AND shelf = s_shelf;

    ELSE
        -- Здесь число экземпляров после обновления совпадает -> Ничего не деланем
        RETURN NULL;

    END IF;

    RETURN NEW;
END; $$
LANGUAGE plpgsql;
```
Будем запускать эту функцию при каждой манипуляцией с таблицей "Книги на полках" 
```PGSQL
DROP TRIGGER IF EXISTS books_shelf_manipulate ON books_shelf;
CREATE TRIGGER books_shelf_manipulate AFTER DELETE OR INSERT OR UPDATE ON books_shelf FOR EACH ROW EXECUTE PROCEDURE books_shelf_manipulate();
```
Также нам необходимо удалять из таблицы "Книги на полках" экземпляры представленные в количестве 0 штук:

```PGSQL 
CREATE OR REPLACE FUNCTION books_shelf_check_update_zero() RETURNS trigger AS $$
BEGIN
    -- Перед каждым обновлением на ноль заменим операцию на удаление
    IF (NEW.numbers = 0) THEN 
        DELETE FROM books_shelf WHERE isbn = NEW.isbn;
        RETURN NULL;
    END IF;

RETURN NEW;

END; $$
LANGUAGE plpgsql;


DROP TRIGGER IF EXISTS books_shelf_check_update_zero ON books_shelf;
CREATE TRIGGER books_shelf_check_update_zero BEFORE UPDATE ON books_shelf FOR EACH ROW EXECUTE PROCEDURE books_shelf_check_update_zero();

```

## Данные для базы данных ##

Заполним БД реальными данными после чего повыполняем типичные операции. Для этого специально был немного переработан датасет с [Kaggle](https://www.kaggle.com/dylanjcastillo/7k-books-with-metadata) с добавлением полей: "цена", "толщина книги" (основна на типчной толщине страниц и обложки умноженная на количество страниц). Переработаны данные о категории с формированием иеерархии, также переработаны данные об авторах для нормализации данных в 3НФ. Также были сгенерированны таблицы "типы шкафов", "Кнжные шкафы" и "размещение книжек на полках". Таблица полки, где собственно и происходит рассчет свободного места не генерировалась, так как она должна сгенрироваться при добавлении таблицы "Книжные шкафы", а место рассчитаться на основе данных о "книгах на полках". Все преобразования с датасетом можно посмотреть в файле [tables_generate.ipynb](https://github.com/alex-12345/posgreSQL/blob/main/tables_generate.ipynb). Итак скопируем данные из CSV файлов в наши таблицы.

```PGSQL
-- На всякий случай сначала отчистим таблицы
truncate bookcase_types,bookcases,books,books_author,books_shelf,categories,categories_hierarchy,shelves; 
-- Загрузим данные 
COPY categories FROM '/var/sqldump/categories.csv' WITH (FORMAT csv);
COPY categories_hierarchy(category, dom_category) FROM '/var/sqldump/categories_hierarchy.csv' WITH (FORMAT csv, HEADER true);
COPY books(isbn,title,category,description,published_year,average_rating,num_pages,ratings_count,book_thickness,price) FROM '/var/sqldump/books.csv' WITH (FORMAT csv, HEADER true);
COPY books_author(isbn,author) FROM '/var/sqldump/book_author.csv' WITH (FORMAT csv, HEADER true);
COPY bookcase_types(type,width,shelf_numbers) FROM '/var/sqldump/bookcase_types.csv' WITH (FORMAT csv, HEADER true);
COPY bookcases(hall,bookcase,bookcase_type) FROM '/var/sqldump/bookcases.csv' WITH (FORMAT csv, HEADER true);
COPY books_shelf(isbn,hall,bookcase,shelf,numbers) FROM '/var/sqldump/books_shelf.csv' WITH (FORMAT csv, HEADER true);
```

## Несколько сложных, но довольно типичных запросов к БД ##

**Получим 15 самых свободных шкафов**

```PGSQL
WITH shelves_data AS (
  SELECT 
    b.hall, 
    b.bookcase, 
    s.shelf, 
    s.occupied_width, 
    b.width 
  FROM 
    shelves s 
    INNER JOIN (
      SELECT 
        b2.hall, 
        b2.bookcase, 
        bt2.width 
      FROM 
        bookcases b2 
        INNER JOIN bookcase_types bt2 ON b2.bookcase_type = bt2.type
    ) as b ON s.hall = b.hall 
    AND s.bookcase = b.bookcase
) 
SELECT 
  hall, 
  bookcase, 
  SUM(width - occupied_width) as free_width 
FROM 
  shelves_data 
GROUP BY 
  hall, 
  bookcase 
ORDER BY 
  free_width DESC, 
  hall DESC, 
  bookcase DESC 
LIMIT 
  15;

-- Получим следующее
/*
 hall | bookcase | free_width 
------+----------+------------
    5 |        8 |      14400
    5 |        6 |      14400
    5 |       12 |      12600
    5 |       11 |      12600
    5 |       10 |      12600
    5 |        9 |      12600
    5 |        7 |       9600
    5 |        5 |       5082
    2 |       10 |       1445
    5 |        3 |       1408
    3 |        5 |       1239
    4 |        3 |       1160
    4 |        8 |       1121
    4 |        9 |       1105
    2 |       11 |       1098
(15 rows) */

```
Как можно заменить при добавлении все успешно расчиталось. Так как при генерации шкафы заполнялись подряд и если пачка экземпляров одной книги не влезала на полку переходили к следующей. Из реузльтатов видно что шкафы с 7 по 12 в последнем 5 зале полностью пустуют (разница в свободном месте объясняется различными типами шкафов). Ну а распределение сработало так что где -то 1 метр сумаррно на всех полках шкафа остается не заполнен.

**Выведем 20 самых популярных авторов, основываясь на количестве отзывов покупателей**

```PGSQL
SELECT 
  author, 
  SUM(ratings_count) as rating_count, 
  AVG(average_rating) rating_avg, 
  COUNT(*) as book_count 
FROM 
  books_author ba 
  INNER JOIN books b ON ba.isbn = b.isbn 
GROUP BY 
  author 
ORDER BY 
  rating_count DESC 
LIMIT 
  20;

-- Получим 
/* 
          author           | rating_count |     rating_avg     | book_count 
---------------------------+--------------+--------------------+------------
 Rowling, J.K.             |     13835911 | 4.4960000000000000 |          5
 Stephenie Meyer           |      4367341 | 3.5900000000000000 |          1
 William Shakespeare       |      4101324 | 3.9711363636363636 |         44
 Dan Brown                 |      3890172 | 3.8080000000000000 |          5
 John Ronald Reuel Tolkien |      3051703 | 4.1620000000000000 |         30
 Stephen King              |      2795366 | 4.0513953488372093 |         43
 John Grisham              |      2203924 | 3.8080000000000000 |         15
 Lois Lowry                |      2082321 | 3.8966666666666667 |          9
 J. R. R. Tolkien          |      2015047 | 4.3883333333333333 |          6
 Paulo Coelho              |      1989300 | 3.6416666666666667 |         12
 William Golding           |      1870343 | 3.6333333333333333 |          6
 Neil Gaiman               |      1689745 | 4.1290000000000000 |         20
 Jodi Picoult              |      1451846 | 3.7933333333333333 |         12
 Charles Dickens           |      1389679 | 3.9752631578947368 |         19
 Aldous Huxley             |      1369059 | 4.0500000000000000 |         11
 Charlotte Brontë          |      1332815 | 4.0500000000000000 |          2
 Elizabeth Gilbert         |      1321470 | 3.6100000000000000 |          4
 E. B. White               |      1291473 | 4.1375000000000000 |          4
 Diana Gabaldon            |      1260619 | 4.2812500000000000 |          8
 Margaret Atwood           |      1205565 | 3.8400000000000000 |          9
(20 rows)*/
```
Самым популярным автором оказалась Роулинг с ее серей книг по Гарри Поттера. Так же можно видить таких авторов как Стивен Книг, Толкиен, Вильям Голдинг, Шекспир и других.

**Мы покпатели в магазине и хотим купить книгу Голдинга "Повелитель мух" и вбиваем 'Lord of the'**
Найдем полки где находять такие книги и отранжируем их по количеству отзывов с лимитом в 10 результатов.
```PGSQL
SELECT 
  b.isbn, 
  substring(
    b.title 
    from 
      0 for 80
  ) as title, 
  bs.hall, 
  bs.bookcase, 
  bs.shelf, 
  bs.numbers as in_stock, 
  b.average_rating, 
  b.ratings_count, 
  b.num_pages, 
  b.price 
FROM 
  books b 
  INNER JOIN books_shelf bs ON b.isbn = bs.isbn 
  AND b.title LIKE '%Lord of the %' 
ORDER BY 
  b.ratings_count DESC 
LIMIT 
  10;

-- Получим следующий вывод:
/*
     isbn      |                                   title                                   | hall | bookcase | shelf | in_stock | average_rating | ratings_count | num_pages | price 
---------------+---------------------------------------------------------------------------+------+----------+-------+----------+----------------+---------------+-----------+-------
 9780618346257 | The Fellowship of the Ring. Being the First Part of the Lord of the Rings |    3 |        8 |     4 |        2 |           4.35 |       2009749 |       398 |   6.0
 9780140283334 | Lord of the Flies                                                         |    1 |        7 |     1 |        3 |           3.67 |       1861140 |       182 |   2.0
 9780345339737 | The Fellowship of the Ring. Being the First Part of the Lord of the Rings |    2 |        3 |     5 |        2 |           4.52 |        532629 |       490 | 21.22
 9780618640140 | The Lord of the Rings Sketchbook                                          |    3 |        8 |     6 |        3 |           4.27 |          9593 |       192 |  8.45
 9780618260225 | The Lord of the Rings. The Making of the Movie Trilogy                    |    3 |        8 |     4 |        1 |           4.46 |          7624 |       192 | 10.27
 9780812695458 | The Lord of the Rings and Philosophy. One Book to Rule Them All           |    4 |        7 |     1 |        4 |           4.28 |          4828 |       240 | 10.75
 9780618258024 | The Lord of the Rings. The Two Towers : Visual Companion                  |    3 |        8 |     4 |        4 |           4.51 |          4501 |        72 |  4.45
 9780571084838 | Lord of the Flies                                                         |    3 |        7 |     3 |        5 |           3.67 |          4498 |       223 | 12.11
 9780451452610 | Bored of the Rings. A Parody of J.R.R. Tolkien's The Lord of the Rings    |    3 |        3 |     1 |        5 |           3.12 |          4325 |       149 | 11.66
 9780618083572 | The Return of the Shadow. The History of The Lord of the Rings, Part One  |    3 |        8 |     3 |        2 |           4.02 |          2263 |       512 | 28.78
(10 rows) */
```
Как можно заметить книга Толкиена более популярна, тем не менее мы нашли, что в наличии есть несколько изданий данной книги и самая популярная и дешевая находиться в первом зале, в седьмом шакфу, на первой полке в количестве 3 штук.

**Получение 5 самых популярных книг в каждой категории**

На сайте магазина очень часто нужно показывать покупателям уже хорошо зарекомендовавшие себя книжки. Сделаем данную выборку отсортировывая также и сами категории по их популярности у покупателей

```PGSQL
SELECT 
  f.category, 
  f.r as rank, 
  f.isbn, 
  substring(
    f.title 
    from 
      0 for 80
  ) as title, 
  f.ratings_count, 
  f.category_total_raiting_count 
FROM 
  (
    SELECT 
      *, 
      ROW_NUMBER() OVER (
        PARTITION BY d.category 
        ORDER BY 
          ratings_count DESC
      ) as r, 
      SUM(ratings_count) OVER (PARTITION BY category) AS category_total_raiting_count 
    FROM 
      (
        SELECT 
          b.isbn, 
          CASE WHEN ch.dom_category IS NULL THEN b.category ELSE ch.dom_category END as category, 
          b.title, 
          b.ratings_count, 
          b.average_rating 
        FROM 
          books b 
          LEFT JOIN categories_hierarchy ch ON b.category = ch.category
      ) as d
  ) as f 
WHERE 
  f.r < 6 
ORDER BY 
  f.category_total_raiting_count DESC, 
  rank;

-- Получим вывод
/*
           category            | rank |     isbn      |                                      title                                      | ratings_count | category_total_raiting_count 
-------------------------------+------+---------------+---------------------------------------------------------------------------------+---------------+------------------------------
 Fiction                       |    1 | 9780439554930 | Harry Potter and the Sorcerer's Stone (Book 1)                                  |       5629932 |                    106399227
 Fiction                       |    2 | 9780316015844 | Twilight                                                                        |       4367341 |                    106399227
 Fiction                       |    3 | 9780618260300 | The Hobbit, Or, There and Back Again                                            |       2364968 |                    106399227
 Fiction                       |    4 | 9781416524793 | Angels & Demons                                                                 |       2279854 |                    106399227
 Fiction                       |    5 | 9780439655484 | Harry Potter and the Prisoner of Azkaban (Book 3)                               |       2149872 |                    106399227
 Biography & Autobiography     |    1 | 9780143038412 | Eat, Pray, Love. One Woman's Search for Everything Across Italy, India, and Ind |       1309623 |                      6981716
 Biography & Autobiography     |    2 | 9780743247542 | The Glass Castle. A Memoir                                                      |        752358 |                      6981716
 Biography & Autobiography     |    3 | 9780385494786 | Into Thin Air. A Personal Account of the Mount Everest Disaster                 |        327791 |                      6981716
 Biography & Autobiography     |    4 | 9780553279375 | I Know why the Caged Bird Sings                                                 |        326424 |                      6981716
 Biography & Autobiography     |    5 | 9780743223133 | John Adams                                                                      |        264047 |                      6981716
 Drama                         |    1 | 9780743477116 | Romeo and Juliet                                                                |       1811259 |                      4023201
 Drama                         |    2 | 9780743477109 | Macbeth                                                                         |        560816 |                      4023201
 Drama                         |    3 | 9780743477543 | A Midsummer Night's Dream                                                       |        371813 |                      4023201
 Drama                         |    4 | 9780743477550 | Othello                                                                         |        261813 |                      4023201
 Drama                         |    5 | 9780822210894 | A Streetcar Named Desire. Play in Three Acts                                    |        229601 |                      4023201
 Art                           |    1 | 9780307277671 | Robert Langdon Novels. The Da Vinci Code : a Novel. 02                          |       1588890 |                      1959795
 Art                           |    2 | 9780140135152 | Ways of Seeing. A Book                                                          |        177890 |                      1959795
 Art                           |    3 | 9780812971422 | Color. A Natural History of the Palette                                         |         59145 |                      1959795
 Art                           |    4 | 9780395530078 | The Natural Way to Draw. A Working Plan for Art Study                           |         36530 |                      1959795
 Art                           |    5 | 9780156717205 | The Philosophy of Andy Warhol. From A to B and Back Again                       |         32180 |                      1959795
 Juvenile Nonfiction           |    1 | 9780521618748 | Hamlet                                                                          |        577429 |                      1346606
 Juvenile Nonfiction           |    2 | 9780590032490 | The witches                                                                     |        254867 |                      1346606
 Juvenile Nonfiction           |    3 | 9780198320272 | Julius Caesar                                                                   |        133575 |                      1346606
 Juvenile Nonfiction           |    4 | 9780060513092 | Falling Up. Poems and Drawings                                                  |        114058 |                      1346606
 Juvenile Nonfiction           |    5 | 9781857232097 | The Fires of Heaven                                                             |        106302 |                      1346606
 Science                       |    1 | 9780767908184 | A Short History of Nearly Everything                                            |        228522 |                      1235099
 Science                       |    2 | 9780553380163 | A Brief History of Time                                                         |        214520 

 ...

*/
```

Как можем видеть самой популярной категорией является "Художественная литература", следом "Биография" и "Драмма". Понятно, что в топе будут такие известные произведения, как "Гарри Поттер", "Властелин Колец", драммы Шекспира и другие.

**Пример запроса со стороны сотрудника магазина**

Попробуем переполнить какую то полку или поменять свойства шкафа или сделать что то подобное

```PGSQL
-- Посмотрим толщину книжки про Гарри Поттера
SELECT title, book_thickness FROM books WHERE isbn='9780439554930';
/*                   title                      | book_thickness 
------------------------------------------------+----------------
 Harry Potter and the Sorcerer's Stone (Book 1) |             36
(1 row)*/

-- Ширина книжки 36 мм

-- Посмотрим на какой полке и сколь экзмепляров есть в наличии
SELECT hall, bookcase, shelf, isbn, numbers FROM books_shelf WHERE isbn='9780439554930';
/*
 hall | bookcase | shelf |     isbn      | numbers 
------+----------+-------+---------------+---------
    2 |       12 |     1 | 9780439554930 |       1
(1 row)*/
-- Есть 1 штука в 2 зале в 12 шкафу на 1 полке

-- Посмотрим загруженость полки

SELECT s.hall, s,bookcase,s.shelf,s.occupied_width,bt.width FROM (SELECT  sh.hall, sh.bookcase, sh.shelf, sh.occupied_width, b.bookcase_type FROM shelves sh INNER JOIN bookcases b ON sh.hall=b.hall AND sh.bookcase=b.bookcase AND sh.hall=2 AND sh.bookcase=12 AND sh.shelf=1) as s INNER JOIN bookcase_types bt ON s.bookcase_type = bt.type;
/*
 hall |          s           | bookcase | shelf | occupied_width | width 
------+----------------------+----------+-------+----------------+-------
    2 | (2,12,1,1782,type_2) |       12 |     1 |           1782 |  1800
(1 row) */

-- Полка почти заполнена, попытаемся закинуть на нее еще 1 штуку, должна появиться ошибка 
UPDATE books_shelf SET numbers=numbers + 1 WHERE isbn='9780439554930';
-- Получили ошибку, что не хватает место
/* ERROR:  Not enough shelf space
CONTEXT:  PL/pgSQL function books_shelf_manipulate() line 53 at RAISE */

-- Проведем покупку 1 штуки или просто уберем ее с полки на склад
BEGIN 
UPDATE books_shelf SET numbers=numbers - 1 WHERE isbn='9780439554930';
/* Получим
UPDATE 0*/
COMMIT

/* Так вышло, потому, что запрос на обновления был перехвачен так как в результате оставалась одна шутука*/

-- Проверим 
SELECT hall, bookcase, shelf, isbn, numbers FROM books_shelf WHERE isbn='9780439554930';
/* Все ее больше нет
 hall | bookcase | shelf | isbn | numbers 
------+----------+-------+------+---------
(0 rows) */
-- Посмотрим что там на полке

SELECT s.hall, s,bookcase,s.shelf,s.occupied_width,bt.width FROM (SELECT  sh.hall, sh.bookcase, sh.shelf, sh.occupied_width, b.bookcase_type FROM shelves sh INNER JOIN bookcases b ON sh.hall=b.hall AND sh.bookcase=b.bookcase AND sh.hall=2 AND sh.bookcase=12 AND sh.shelf=1) as s INNER JOIN bookcase_types bt ON s.bookcase_type = bt.type;
/* Как и ожидалось занятого места на полке стало меньше на 36 мм
 hall |          s           | bookcase | shelf | occupied_width | width 
------+----------------------+----------+-------+----------------+-------
    2 | (2,12,1,1746,type_2) |       12 |     1 |           1746 |  1800
(1 row) */

-- Добавим 12 экземпляров Гарри Поттера на самый пустеющий шкаф

INSERT INTO books_shelf(hall, bookcase, shelf, isbn, numbers ) VALUES (5,8,1,'9780439554930', 10);
-- Как мы помним все полки этого шкафа были пустые (см первый пример)
-- Посмотрим занятость этой полки

SELECT s.hall, s,bookcase,s.shelf,s.occupied_width,bt.width FROM (SELECT  sh.hall, sh.bookcase, sh.shelf, sh.occupied_width, b.bookcase_type FROM shelves sh INNER JOIN bookcases b ON sh.hall=b.hall AND sh.bookcase=b.bookcase AND sh.hall=5 AND sh.bookcase=8 AND sh.shelf=1) as s INNER JOIN bookcase_types bt ON s.bookcase_type = bt.type;
/* Вот и наши 10 экземпляров заняли 360 мм
 hall |         s          | bookcase | shelf | occupied_width | width 
------+--------------------+----------+-------+----------------+-------
    5 | (5,8,1,360,type_3) |        8 |     1 |            360 |  2400
(1 row)
*/

```
## Заключение ##
В результате работы было создано, практически полностью самобытнапя база данных для книжного магазина по образцу существующего магазина "Библио-Глобус". В согласовановсти на мой взгляд есть небольшая недоработка в том, что мы не запрешаем ручное обновление загружености полок, тем не менее если этого не делать, то база данных всегда будет находиться в корректном и актуальном состоянии.

В работе не была продемострирована оработа многих тригеррных функций таких как блокировки ручного обновления (тем не менее спешу заверить, что все добавленные триггерные функции были отлажены). Однако отчет и так получился довольно большой. Кроме того в данную базу данных не мешало бы добавить полнотекстовый поиск по названию книжки, категории и ее автору. Тем не менее имеем, что имеем. 

Также был упущена важная блокировка на изменение толщины книжки, если ее экземмпляры присутсвуют на полках.

## Прочее ##
**Отчистка всех таблиц**
```PGSQL
truncate bookcase_types,bookcases,books,books_author,books_shelf,categories,categories_hierarchy,shelves;
```

Дамп базы данных находиться в файле [sqldump/final.dump.sql](https://github.com/alex-12345/posgreSQL/blob/main/sqldump/final.dump.sql)
