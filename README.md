# Лабораторные работы по PostgreSQL #
Студент: Винников Алексей

Преподаватель: Моргунов Евгений Павлович

Группа: М8О-203М-20


## Подготовка окружения к ДЗ №2-9 ##

Задания в этом блоке выполнялись по контрольным вопросам из [книги](https://edu.postgrespro.ru/sql_primer.pdf) 

**Requirements**

- Docker
- Docker-compose

**Запуск сервера БД**

```bash 
$ docker-compose up -d 
```
**Получение и распаковка учебной БД**

```BASH 
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin wget https://edu.postgrespro.ru/demo-small-20161013.zip -P /var/sqldump/ # скачаем файл
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin unzip /var/sqldump/demo-small-20161013.zip -d /var/sqldump/task2-9/ # распакуем файлы 
$ docker exec -it task2-9-server /usr/bin/psql -f /var/sqldump/task2-9/demo_small.sql -U root -d demo # выполним экспорт
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin sh -c "rm -rf /var/sqldump/demo-small-20161013.zip /var/sqldump/task2-9/*" # эти файлы больше не нужны
```

**Подключение к БД по psql**

```bash 
$ docker exec -it task2-9-server /usr/bin/psql -U root -d demo
```
Так как в учебной БД таблицы создаются в schema bookings то будем переключаться на нее по мерее необходимости
```PGSQL
SET search_path TO bookings
```

**Графические средства для работы с БД**

Для удобства используем графическое расширение для работы с БД прямо из IDE (в моем случае расширение PostgreSQL в VS code) 

<center>
<img src="./images/vscode.png " />
<p>Работа с БД с помощью графического интерфеса расширения PostgreSQL в VS code</p>
</center>

Также можно использовать pgAdmin, он уже запущен как docker-контейнер перейдем в браузер и передем на `http://localhost:8080`. После чего нам станет доступен интерфейс pgAdmin. Войдем с логином `root` и паролем `1234`. После чего создадим обычное соединение с базой данных, указав в качестве хоста `postgres` (это важно!) так-как именно такое именно имя используется во внутреней сети docker. Скриншот с работой с нашей БД из под PgAdmin представлен ниже.

<center>
<img src="./images/pgadmin.png " />
<p>Работа с БД с помощью графического интерфеса расширения PgAdmin</p>
</center>
<p></p>

## ДЗ №2. Ответы на контрольные вопросы и задания к главе №3 ##

**Ответ на вопрос №1**

Следующая операция  
```PGSQL 
INSERT INTO aircrafts
VALUES ( 'SU9', 'Sukhoi SuperJet-100', 3000 );
```
не выполниться, так как таблица aircrafts содержит атрибут aircraft_code который является первичным ключем и должен быть уникальным. А строка с индексом 'SU9' уже содержиться в таблице

**Ответ на вопрос №2**

Команда для выборки всех строк из таблицы `aircraft` в обратном порядке следующая:
```PGSQL 
SELECT * FROM aircrafts ORDER BY aircraft_code DESC;
```
В результате получим

```
 aircraft_code |        model        | range 
---------------+---------------------+-------
 SU9           | Sukhoi SuperJet-100 |  3500
 CR2           | Bombardier CRJ-200  |  2700
 CN1           | Cessna 208 Caravan  |  1200
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 733           | Boeing 737-300      |  4200
 321           | Airbus A321-200     |  5600
 320           | Airbus A320-200     |  5700
 319           | Airbus A319-100     |  6700
(9 rows)
```

**Ответ на вопрос №3**

Команда для увеличения значения `range` в два раза у модели `Sukhoi SuperJet-100` следующая:
```PGSQL 
UPDATE aircrafts SET range = range * 2
WHERE model = 'Sukhoi SuperJet-100';
```

**Ответ на вопрос №4**

Пример SQL запроса на данной БД который не удалит не одной строки в таблице:
```PGSQL 
DELETE FROM aircrafts WHERE range < 0;
```

## ДЗ №3. Ответы на контрольные вопросы и задания к главе №4 ##

**Ответ на вопрос №2**

Для хранения в одном столбце плавающие значения с различной точностью используем тип numeric без указанаия точности и масштаба. Создадим таблицу и заполним ее:
```PGSQL 
CREATE TABLE test_numeric
( measurement numeric,
description text
);

INSERT INTO test_numeric
VALUES ( 1234567890.0987654321,
'Точность 20 знаков, масштаб 10 знаков' );

INSERT INTO test_numeric
VALUES ( 1.5,
'Точность 2 знака, масштаб 1 знак' );

INSERT INTO test_numeric
VALUES ( 0.12345678901234567890,
'Точность 21 знак, масштаб 20 знаков' );

INSERT INTO test_numeric
VALUES ( 1234567890,
'Точность 10 знаков, масштаб 0 знаков (целое число)' );
```

Теперь при выборке из данной таблице получим следующий результат
```
      measurement       |                    description                     
------------------------+----------------------------------------------------
  1234567890.0987654321 | Точность 20 знаков, масштаб 10 знаков
                    1.5 | Точность 2 знака, масштаб 1 знак
 0.12345678901234567890 | Точность 21 знак, масштаб 20 знаков
             1234567890 | Точность 10 знаков, масштаб 0 знаков (целое число)
(4 rows)

```

**Ответ на вопрос №4**

Посмотрим поведение PostgreSQL на верхних границах допустимых значений типов real и double precision

```PGSQL
/* Границы типа double precision 1E-307 до 1E+308 с точностью 15. То-есть для маленьких значений фактически допускается числа вплоть до 1E-323 разряды младше будут теряться. Для очень больших (на границе) принимается в расчет только первые 16 старших десятичных разрядов */

SELECT '1e+308'::double precision + '1e+292'::double precision = '1e+308'::double precision + '0.0'::double precision; -- Здесь 17 старший разряд обрязается так что числа становяться равными
/*
 ?column? 
----------
 t
(1 row)
*/

demo=#SELECT '1e+308'::double precision + '1e+291'::double precision = '1e+308'::double precision + '0.0'::double precision; -- Здесь всего 16 старших десятичных разрядов, так что различе в последнем старшем разряде учитывается
/*
 ?column? 
----------
 f
(1 row)
*/

/* У типа real границы следующие 1E-37 до 1E+37, а точность 6 на них поведение идентично типу double precision */

SELECT '1e+38'::real + '1e+31'::real = '1e+38'::real + '0.0'::real; -- Здесь 7 старший десятичный разряд будет 1 в первом случае. он учитывается следовательно числа не равны
/*
 ?column? 
----------
 f
(1 row)
*/

SELECT '1e+38'::real + '1e+30'::real = '1e+38'::real + '0.0'::real; -- Здесь вторая единичка уже на 8 разряде следовательно она не учитывается
/*
 ?column? 
----------
 t
(1 row)
*/
```

**Ответ на вопрос №8**

Задание на понимание принципа работы автоматически инкрементно обновляемого индекса.
```PGSQL 
CREATE TABLE test_serial
( id serial PRIMARY KEY,
name text
);
/* Содздали таблица при вставке без конкретного указазания значения id будет присвоено значение 1. 
Выполним запросы на вставку строк в таблицу. */

INSERT INTO test_serial ( name ) VALUES ( 'Вишневая' ); -- Для данной записи будет присвоено id 1, для следующего будет значение 2
INSERT INTO test_serial ( id, name ) VALUES ( 2, 'Прохладная' ); -- Тут мы явно указываем id (обновление последовательности для id не происходит), значит для следующего будет по прежденему 2
INSERT INTO test_serial ( name ) VALUES ( 'Грушевая' ); -- Ошибка, так как автоматически в последовательности проставляется значение 2 и происходит обновляет, для следующего значения уже будет 3, однако на данной момент это 2 и строка с таким id уже существует.
INSERT INTO test_serial ( name ) VALUES ( 'Грушевая' ); -- Успешно, так как последовательность обновилось несмотря на ошибку при прошлом запросе и теперь уже ошибки не происходит. Текущее значение последовательность id 3, для следующего запроса - 4
INSERT INTO test_serial ( name ) VALUES ( 'Зеленая' ); --Текущий id - 4, для следующего будет 5
DELETE FROM test_serial WHERE id = 4; --Удаляем строку, однако значение последовательности при этом не меняется (для следубющей вставки id будет 5)
INSERT INTO test_serial ( name ) VALUES ( 'Луговая' ); -- Здесь как и ожидалось будет вставка с id 5, для следующей вставки будет 7

SELECT * FROM test_serial; 
-- Вывод имперически подверждает вышеприведенные рассуждения. Таблица будет следующая:
/*
 id |    name    
----+------------
  1 | Вишневая
  2 | Прохладная
  3 | Грушевая
  5 | Луговая
(4 rows) */
```

**Ответ на вопрос №12**

Переведем datastyle в значение по умолчанию (в данной версии это ISO, MDY)
```PGSQL 
SET datestyle TO DEFAULT;
SHOW datestyle;
-- Получим
/* DateStyle 
-----------
 ISO, MDY
(1 row) */
```
Поэксперементируем с форматами даты SQL (традиционный стиль) и German (региональный стиль):

```PGSQL 
SET datestyle TO 'German, DMY';
SELECT '17.12.1997'::date;
/*  date    
------------
 17.12.1997
(1 row) */

SELECT '12.17.1997'::date;
/* Ошибка так как вторым значением по формату даты DMY является месяц

ERROR:  date/time field value out of range: "12.17.1997"
LINE 1: SELECT '12.17.1997'::date;
               ^
HINT:  Perhaps you need a different "datestyle" setting.

Поменяем формат даты на 'German, MDY' и теперь данный запрос успешно выполниться
*/
SET datestyle TO 'German, MDY';
SELECT '12.17.1997'::date;
-- Получим
/*  date    
------------
 17.12.1997
(1 row) */

/* В качестве эксперемента повторим тоже самое с форматом даты SQL*/

SET datestyle TO 'SQL, DMY';
SELECT '17/12/1997'::date;
-- Получим
/*  date    
------------
 17/12/1997
(1 row) */

SELECT '12/17/1997'::date; -- Ошибка
SET datestyle TO 'SQL, MDY';
SELECT '12/17/1997'::date; -- Все нормально
```

**Ответ на вопрос №15**

Работа с форматированием метки времени в строку с помощью функции to_char:

```PGSQL 
SELECT to_char( current_timestamp, 'mi:ss' ); -- Вывод в формате 'минута:секунда' (например 26:25 - 12 минут 0 секуд)
SELECT to_char( current_timestamp, 'dd' ); -- Вывод в формате 'номер дня в месяце' например (15 - 15 число текущего месяца) 
SELECT to_char( current_timestamp, 'yyyy-mm-dd' ); -- Вывод текущей даты в численном формате 'год-месяц-день' (2021-11-15)
SELECT to_char( current_timestamp, 'yyyy-mm-dd:SSSS' ); -- Вывод текущей даты в численном формате 'год-месяц-день:число секунд с начала суток' (2021-11-15:63030)
SELECT to_char( current_timestamp, 'yyyy MONTHdd' ); -- Вывод текущей даты в численном формате 'год месяц(текстом) день' (2021 NOVEMBER 15)
```

**Ответ на вопрос №21**

При добавление интервала PostgreSQL учитывает различное число дней в месяцах, так например при добавление к дате (конец какого либо месяца), просматривается число дней в следующем месяце и если оно меньше, то проставляется последнее число следующего месяца. Например:
```PGSQL
SELECT ( '2016-01-31'::date + '1 mon'::interval ) AS new_date;
-- Следующий месяц содержит 29 дней так, что ставиться последнее число след месяца
/*      new_date       
---------------------
 2016-02-29 00:00:00
(1 row) */

SELECT ( '2016-02-29'::date + '1 mon'::interval ) AS new_date;
-- Здесь все нормально 
/*      new_date       
---------------------
 2016-03-29 00:00:00
(1 row) */
```

**Ответ на вопрос №30**

Работа с булевым типом

```PGSQL
CREATE TABLE test_bool
( a boolean,
b text
);

/*  Допустимые boolean значения: 
      TRUE, true, 't', 'true', 'y', 'yes', 'on', '1'
      FALSE, false, 'f', 'false', 'n', 'no', 'off', '0'
*/

INSERT INTO test_bool VALUES ( TRUE, 'yes' ); -- Данный запрос корректен
INSERT INTO test_bool VALUES ( yes, 'yes' ); -- Ошибка токен yes не зарезервирован
INSERT INTO test_bool VALUES ( 'yes', true ); -- Запрос корректен второй аргумент преобразуется в строчку 
INSERT INTO test_bool VALUES ( 'yes', TRUE ); -- Корректно
INSERT INTO test_bool VALUES ( '1', 'true' ); -- Корректно
INSERT INTO test_bool VALUES ( 1, 'true' ); -- Некорректно, токен 1 не зарезервирован под boolean
INSERT INTO test_bool VALUES ( 't', 'true'); -- Корректно
INSERT INTO test_bool VALUES ( 't', truth ); -- Ошибка токен truth не зарезервирован
INSERT INTO test_bool VALUES ( true, true); -- Корректно
INSERT INTO test_bool VALUES ( 1::boolean, 'true' ); -- Конвертация любого числа, кроме 0, в boolean дает TRUE так что корректно 
INSERT INTO test_bool VALUES ( 111::boolean, 'true'); -- Корректно

-- Выполнив данные запросы подтвердим вышенаписанное:
/*
INSERT 0 1
ERROR:  column "yes" does not exist
LINE 1: INSERT INTO test_bool VALUES ( yes, 'yes' );
                                       ^
INSERT 0 1
INSERT 0 1
INSERT 0 1
ERROR:  column "a" is of type boolean but expression is of type integer
LINE 1: INSERT INTO test_bool VALUES ( 1, 'true' );
                                       ^
HINT:  You will need to rewrite or cast the expression.
INSERT 0 1
ERROR:  column "truth" does not exist
LINE 1: INSERT INTO test_bool VALUES ( 't', truth );
                                            ^
INSERT 0 1
INSERT 0 1
INSERT 0 1
*/
```

**Ответ на вопрос №33**

Создадим таблицу "Пилоты" с полем meal(обеды) в виде двумерного строкового массива:

```PGSQL 
CREATE TABLE pilots                                
( pilot_name text,
schedule integer[],
meal text[][]
);
```

Добавим различные строки и сделаем различные выборки:
```PGSQL
INSERT INTO pilots
VALUES ( 'Ivan', '{ 1, 3, 5, 6, 7 }'::integer[],
'{ 
   { "сосиска", "макароны", "кофе" }, 
   { "куриное филе", "пюре", "какао" }, 
   { "рагу", "сэндвич с семгой", "морс ягодный" }, 
   { "шарлотка яблочная", "гречка", "компот вишевый" }, 
   { "омлет с овощами", "бекон", "кофе" } 
 }'::text[][]),
( 'Petr', '{ 1, 2, 5, 7 }'::integer[],
'{ 
   { "котлета", "каша", "кофе" },
   { "куринная отбивная", "рис", "компот" },
   { "манная каша", "билины с мясом", "компот" },
   { "мясо запеченное", "пюре", "какао" } 
 }'::text[][]),
( 'Pavel', '{ 2, 5 }'::integer[],
'{ 
   { "сосиска", "каша", "кофе" },
   { "мясо запеченное", "пюре", "какао" }
 }'::text[][]),
( 'Boris', '{ 3, 5, 6 }'::integer[],
'{ 
   { "котлета", "каша", "чай" },
   { "куринная отбивная", "рис", "компот" },
   { "сосиска", "макароны", "кофе" }
  }'::text[][]);

-- Теперь имеем следующую таблицу:
SELECT * FROM pilots;
-- Получим
/* 
 pilot_name |  schedule   |                                                                                    meal                                                                                     
------------+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Ivan       | {1,3,5,6,7} | {{сосиска,макароны,кофе},{"куриное филе",пюре,какао},{рагу,"сэндвич с семгой","морс ягодный"},{"шарлотка яблочная",гречка,"компот вишевый"},{"омлет с овощами",бекон,кофе}}
 Petr       | {1,2,5,7}   | {{котлета,каша,кофе},{"куринная отбивная",рис,компот},{"манная каша","билины с мясом",компот},{"мясо запеченное",пюре,какао}}
 Pavel      | {2,5}       | {{сосиска,каша,кофе},{"мясо запеченное",пюре,какао}}
 Boris      | {3,5,6}     | {{котлета,каша,чай},{"куринная отбивная",рис,компот},{сосиска,макароны,кофе}}
(4 rows)
*/

/* Выведем имена пилотов который в первый день их работы едят котлету или кашу */

SELECT pilot_name FROM pilots WHERE  meal[1][1] IN('котлета','каша') OR meal[1][2] IN ('котлета','каша') OR meal[1][3] IN('котлета','каша');
-- Получим
/*
 pilot_name 
------------
 Petr
 Pavel
 Boris
(3 rows)

```

**Ответ на вопрос №35**

Продемонстрируем некотрые функции для работы с JSON в PostreSQL из [документации](https://postgrespro.ru/docs/postgrespro/9.6/functions-json)

```PGSQL
SELECT to_json('Fred said "Hi."'::text); --функция перевода внутрених значений PostgreSQL в строку json
-- Получим
/* 
       to_json       
---------------------
 "Fred said \"Hi.\""
(1 row)*/

SELECT json_build_object('foo',1,'bar',2); --построение json строки из списка по соглашения чередования ключ-строка
-- Получим
/* 
   json_build_object    
------------------------
 {"foo" : 1, "bar" : 2}
(1 row)
*/

SELECT json_extract_path('{"f2":{"f3":1},"f4":{"f5":99,"f6":"foo"}}','f4'); -- получение строки JSON по ключу из строки JSON
-- Получим
/* 
  json_extract_path   
----------------------
 {"f5":99,"f6":"foo"}
(1 row)
*/

SELECT json_object_keys('{"f1":"abc","f2":{"f3":"a", "f4":"b"}}'); -- получение ключей JSON строки
-- Получим
/* 
 json_object_keys 
------------------
 f1
 f2
(2 rows)
*/

SELECT jsonb_pretty(jsonb_insert('{"a": [0,1,2]}', '{a, 1}', '"new_value"')); -- Вставка значениес форматированным красивым выводом JSON строки
-- Получим
/*
     jsonb_pretty     
----------------------
 {                   +
     "a": [          +
         0,          +
         "new_value",+
         1,          +
         2           +
     ]               +
 }
(1 row) */
```
 Здесь далеко не все функции по необходимости их можно посмотреть в [документации](https://postgrespro.ru/docs/postgrespro/9.6/functions-json). Стоит отметить что среди этих функции в Postgres присутвуют функции для редактирования, удаления значения по ключу, а также функции для различных преобразований ориентации json объектов и многочисленные функции построения JSON объектов из различных форматов 

## ДЗ №4. Ответы на контрольные вопросы и задания к главе №5 ##

**Ответ на вопрос №2**

Произведем манипуляции с созданной в главе базой данных

```PGSQL
ALTER TABLE progress ADD COLUMN test_form text; -- Добавим в таблицу progress колонку test_form
ALTER TABLE progress  -- А также дополнительное условие                         
ADD CHECK (
( test_form = 'экзамен' AND mark IN ( 3, 4, 5 ) )
OR
( test_form = 'зачет' AND mark IN ( 0, 1 ) )
);

/* Добавим данные */
INSERT INTO students VALUES (11111, 'Vinnikov A.V', 1111, 12345);
-- INSERT 0 1
INSERT INTO progress VALUES(11111,'SQL lang','2021-2022',2,5,'экзамен'); -- Запись с экзаменом - корректно добавляется
-- INSERT 0 1
INSERT INTO progress VALUES(11111,'ML','2021-2022',2, 1,'зачет'); -- Запись с зачетом, тут ошибка, так как check срабатывают по мере их добавления

-- Получим
/* ERROR:  new row for relation "progress" violates check constraint "progress_mark_check"
DETAIL:  Failing row contains (11111, ML, 2021-2022, 2, 1, зачет).
*/

--Удалим проверку mark, так как ее полностью покрывает проверка test_form
INSERT INTO progress VALUES(11111,'ML','2021-2022',2, 1,'зачет'); 
-- Теперь все корректно 
/* INSERT 0 1 */

/*Добавим проверку учебного года */

ALTER TABLE progress ADD CHECK (acad_year ~ $$^20[0-9]{2}\-20[0-9]{2}$$);
-- ALTER TABLE
/* Теперь в корлонку acad_year можно вставлять только два года через дифис начиная с 2000 заканчивая 2099 */
-- Проверим
INSERT INTO progress VALUES(11111,'ML','2021-2022',2, 1,'зачет');
-- INSERT 0 1
INSERT INTO progress VALUES(11111,'ML','20212022',2, 1,'зачет');
/*ERROR:  new row for relation "progress" violates check constraint "progress_acad_year_check"
DETAIL:  Failing row contains (11111, ML, 20212022, 2, 1, зачет).
*/
```

**Ответ на вопрос №9**

```PGSQL
-- При добавлении пустых строчек в колонках типа text NOT NULL никаких ошибок не возникает */
INSERT INTO students VALUES (11112, ' ', 1111, 12345); -- Этот запрос успешно выполниться 
DELETE FROM students WHERE TRIM(name) = ''; -- Сначала удалим все такие строчки
ALTER TABLE students ADD CHECK (TRIM(name) <> ''); -- Добавим саму проверку 
INSERT INTO students VALUES (11112, ' ', 1111, 12345); -- Теперь такая вставка невозможна
```

**Ответ на вопрос №17**

```PGSQL
-- Создадим представление всех вылетов из Москвы
CREATE VIEW flights_from_Moscow AS 
SELECT 
  f.flight_id, 
  f.airport as depature_airport, 
  airports.airport_name as arrival_airport, 
  f.flight_no, 
  f.scheduled_departure, 
  f.scheduled_arrival, 
  f.status, 
  f.aircraft_code, 
  f.actual_departure, 
  f.actual_arrival 
FROM 
  (
    SELECT 
      airports.airport_name as airport, 
      flights.flight_id, 
      flights.flight_no, 
      flights.scheduled_departure, 
      flights.scheduled_arrival, 
      flights.arrival_airport, 
      flights.status, 
      flights.aircraft_code, 
      flights.actual_departure, 
      flights.actual_arrival 
    FROM 
      flights, 
      airports 
    WHERE 
      airports.city = 'Москва' 
      AND flights.departure_airport = airports.airport_code
  ) as f, 
  airports 
WHERE 
  f.arrival_airport = airports.airport_code;


-- Посмотрим первые 10 строк
SELECT * FROM flights_from_Moscow WHERE 1 = 1 LIMIT 10;
-- Получим
/*
 flight_id | depature_airport | arrival_airport | flight_no |  scheduled_departure   |   scheduled_arrival    |  status   | aircraft_code |    actual_departure    |     actual_arrival     
-----------+------------------+-----------------+-----------+------------------------+------------------------+-----------+---------------+------------------------+------------------------
         1 | Домодедово       | Пулково         | PG0405    | 2016-09-13 05:35:00+00 | 2016-09-13 06:30:00+00 | Arrived   | 321           | 2016-09-13 05:44:00+00 | 2016-09-13 06:39:00+00
         2 | Домодедово       | Пулково         | PG0404    | 2016-10-03 15:05:00+00 | 2016-10-03 16:00:00+00 | Arrived   | 321           | 2016-10-03 15:06:00+00 | 2016-10-03 16:01:00+00
         3 | Домодедово       | Пулково         | PG0405    | 2016-10-03 05:35:00+00 | 2016-10-03 06:30:00+00 | Arrived   | 321           | 2016-10-03 05:39:00+00 | 2016-10-03 06:34:00+00
         4 | Домодедово       | Пулково         | PG0402    | 2016-11-07 08:25:00+00 | 2016-11-07 09:20:00+00 | Scheduled | 321           |                        | 
         5 | Домодедово       | Пулково         | PG0405    | 2016-10-14 05:35:00+00 | 2016-10-14 06:30:00+00 | On Time   | 321           |                        | 
         6 | Домодедово       | Пулково         | PG0404    | 2016-10-14 15:05:00+00 | 2016-10-14 16:00:00+00 | Scheduled | 321           |                        | 
         7 | Домодедово       | Пулково         | PG0403    | 2016-10-14 07:25:00+00 | 2016-10-14 08:20:00+00 | Delayed   | 321           |                        | 
         8 | Домодедово       | Пулково         | PG0402    | 2016-10-14 08:25:00+00 | 2016-10-14 09:20:00+00 | On Time   | 321           |                        | 
         9 | Домодедово       | Пулково         | PG0405    | 2016-10-23 05:35:00+00 | 2016-10-23 06:30:00+00 | Scheduled | 321           |                        | 
        10 | Домодедово       | Пулково         | PG0402    | 2016-10-21 08:25:00+00 | 2016-10-21 09:20:00+00 | Scheduled | 321           |                        | 
(10 rows)
*/

-- Посчитаем количество вылетов из каждого московского аэропорта

SELECT count(flight_id), depature_airport FROM flights_from_Moscow GROUP BY depature_airport;
-- Получим
/*
 count | depature_airport 
-------+------------------
  1719 | Внуково
  2981 | Шереметьево
  3217 | Домодедово
(3 rows) */
```
**Ответ на вопрос №18**

Добавим в таблицу "Аэропорты" столбец "Инфрастуктура" (залы ожидания, кафе, кол-во входов на посадку и т.д.)
```PGSQL
ALTER TABLE airports ADD COLUMN infrastructure jsonb;

UPDATE airports
SET infrastructure =
'{ "extra": {"police": "1 floor", "medical_center":"1 floor"},
   "toilets": {"1 floor": 3, "2 floor": 9, "3 floor": 6},
   "food": { "cafe": 16,"restaurant": 9 }
}'::jsonb
WHERE airport_name = 'Домодедово';

-- Посмотрим вывод
/* SELECT * FROM airports WHERE airport_name = 'Домодедово';
 airport_code | airport_name |  city  | longitude | latitude  |   timezone    |                                                                        infrastructure                                                                        
--------------+--------------+--------+-----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------
 DME          | Домодедово   | Москва | 37.906111 | 55.408611 | Europe/Moscow | {"food": {"cafe": 16, "restaurant": 9}, "extra": {"police": "1 floor", "medical_center": "1 floor"}, "toilets": {"1 floor": 3, "2 floor": 9, "3 floor": 6}}
(1 row)
*/
```

## ДЗ №5. Ответы на контрольные вопросы и задания к главе №6 ##

**Ответ на вопрос №2**

Выедем всех пасажиров у которых имена состоят из 5 букв
```PGSQL
SELECT passenger_name FROM tickets WHERE passenger_name LIKE '_____ %';
```

**Ответ на вопрос №7**

Избавимся от дубликатов следующим образом 

```PGSQL
WITH f1 AS (
  SELECT 
    DISTINCT departure_city, 
    arrival_city 
  FROM 
    routes r 
    JOIN aircrafts a ON r.aircraft_code = a.aircraft_code 
  WHERE 
    a.model = 'Boeing 777-300' 
  ORDER BY 
    1
), 
f2 AS (
  SELECT 
    CASE WHEN f1.departure_city < f1.arrival_city THEN f1.departure_city ELSE f1.arrival_city END AS arrival_city, 
    CASE WHEN f1.departure_city >= f1.arrival_city THEN f1.departure_city ELSE f1.arrival_city END AS departure_city
  FROM 
    f1
) 
SELECT 
  DISTINCT * 
FROM 
  f2;

-- Получим
/*
 arrival_city | departure_city 
--------------+--------------
 Москва       | Новосибирск
 Екатеринбург | Москва
 Москва       | Пермь
 Москва       | Сочи
(4 rows) */
```

**Ответ на вопрос №9**

```PGSQL 
SELECT 
  departure_city, 
  arrival_city, 
  count(*) 
FROM 
  routes 
GROUP BY 
  departure_city, 
  arrival_city 
HAVING 
  departure_city = 'Москва' 
  AND arrival_city = 'Санкт-Петербург';

-- Получим

/*
 departure_city |  arrival_city   | count 
----------------+-----------------+-------
 Москва         | Санкт-Петербург |    12 
 */
```

**Ответ на вопрос №13**

Для избежания потерь маршрутов с нулем проданных билетов достаточно поменять объединение на LEFT OUTER JOIN
```PGSQL
SELECT 
  f.departure_city, 
  f.arrival_city, 
  max(tf.amount), 
  min(tf.amount) 
FROM 
  flights_v f 
  LEFT OUTER JOIN ticket_flights tf ON f.flight_id = tf.flight_id 
GROUP BY 1, 2 
ORDER BY 1, 2;

-- Получим

/*
      departure_city      |       arrival_city       |    max    |   min    
--------------------------+--------------------------+-----------+----------
 Абакан                   | Архангельск              |           |         
 Абакан                   | Грозный                  |           |         
 Абакан                   | Кызыл                    |           |         
 Абакан                   | Москва                   | 101000.00 | 33700.00
 Абакан                   | Новосибирск              |   5800.00 |  5800.00
 Абакан                   | Томск                    |   4900.00 |  4900.00
 Анадырь                  | Москва                   | 185300.00 | 61800.00
 Анадырь                  | Хабаровск                |  92200.00 | 30700.00
 Анапа                    | Белгород                 |  18900.00 |  6300.00
 Анапа                    | Москва                   |  36600.00 | 12200.00
 Анапа                    | Новокузнецк              |           |         
 Архангельск              | Абакан                   |           |         
 Архангельск              | Иркутск                  |           |         
 Архангельск              | Москва                   |  11100.00 | 10100.00
 Архангельск              | Нарьян-Мар               |   7300.00 |  6600.00
 Архангельск              | Пермь                    |  11000.00 | 11000.00
 Архангельск              | Томск                    |  28100.00 | 25500.00
 Архангельск              | Тюмень                   |  17100.00 | 15500.00
 Архангельск              | Ханты-Мансийск           |  16400.00 | 14900.00
 Астрахань                | Барнаул                  |  29000.00 | 26400.00
 Астрахань                | Москва                   |  14300.00 | 12400.00

 ...
*/
```

**Ответ на вопрос №19**

Добавим номер итерации в каждый запрос 
```PGSQL 
WITH RECURSIVE ranges ( min_sum, max_sum, iteration)
AS (
VALUES  ( 0, 100000, 0),
        ( 100000, 200000, 0 ),
        ( 200000, 300000, 0 )
UNION ALL
SELECT min_sum + 100000, max_sum + 100000, iteration + 1
FROM ranges
WHERE max_sum < ( SELECT max( total_amount ) FROM bookings )
)
SELECT * FROM ranges;

-- Получим
/*
 min_sum | max_sum | iteration 
---------+---------+-----------
       0 |  100000 |         0
  100000 |  200000 |         0
  200000 |  300000 |         0
  100000 |  200000 |         1
  200000 |  300000 |         1
  300000 |  400000 |         1
  200000 |  300000 |         2
  300000 |  400000 |         2
  400000 |  500000 |         2
  300000 |  400000 |         3
  400000 |  500000 |         3
  500000 |  600000 |         3
  400000 |  500000 |         4
  500000 |  600000 |         4
  600000 |  700000 |         4
  500000 |  600000 |         5
  600000 |  700000 |         5
  700000 |  800000 |         5
  600000 |  700000 |         6
  700000 |  800000 |         6
  800000 |  900000 |         6
...
*/
```
При использовании UNION вместо UNION ALL дубликаты не удаляться, так как появляются в разных итерациях

```PGSQL 
WITH RECURSIVE ranges ( min_sum, max_sum, iteration)
AS (
VALUES  ( 0, 100000, 0),
        ( 100000, 200000, 0 ),
        ( 200000, 300000, 0 )
UNION
SELECT min_sum + 100000, max_sum + 100000, iteration + 1
FROM ranges
WHERE max_sum < ( SELECT max( total_amount ) FROM bookings )
)
SELECT * FROM ranges;

-- Получим
/*
 min_sum | max_sum | iteration 
---------+---------+-----------
       0 |  100000 |         0
  100000 |  200000 |         0
  200000 |  300000 |         0
  100000 |  200000 |         1
  200000 |  300000 |         1
  300000 |  400000 |         1
  200000 |  300000 |         2
  300000 |  400000 |         2
  400000 |  500000 |         2
  300000 |  400000 |         3
  400000 |  500000 |         3
  500000 |  600000 |         3
  400000 |  500000 |         4
  500000 |  600000 |         4
  600000 |  700000 |         4
  500000 |  600000 |         5
  600000 |  700000 |         5
  700000 |  800000 |         5
  600000 |  700000 |         6
  700000 |  800000 |         6
  800000 |  900000 |         6
...
*/
```

**Ответ на вопрос №21**

Первым запросом получаем все города кроме москвы, вторым - города в которые есть рейсы из МОСКВЫ. Следовательно необходимо из первого множества вычесть второе. Для этого используем операцию `EXCEPT`
```PGSQL
SELECT city
FROM airports
WHERE city <> 'Москва'
EXCEPT     
SELECT arrival_city
FROM routes   
WHERE departure_city = 'Москва'
ORDER BY city;

-- Получим

/*       city         
----------------------
 Благовещенск
 Иваново
 Иркутск
 Калуга
 Когалым
 Комсомольск-на-Амуре
 Кызыл
 Магадан
 Нижнекамск
 Новокузнецк
 Стрежевой
 Сургут
 Удачный
 Усть-Илимск
 Усть-Кут
 Ухта
 Череповец
 Чита
 Якутск
 Ярославль
(20 rows) */
```

**Ответ на вопрос №23**

Перепишем запрос с общим табличным выражением
```PGSQL
WITH f AS (SELECT count( * )
  FROM ( SELECT DISTINCT city FROM airports ) AS a1
  JOIN ( SELECT DISTINCT city FROM airports ) AS a2
    ON a1.city <> a2.city)
SELECT * FROM f;

-- Получим

/* 
count 
-------
 10100
(1 row)
*/
```

## ДЗ №6. Ответы на контрольные вопросы и задания к главе №7 ##

**Ответ на вопрос №1**

Добавим значение по умолчанию в поле `when_add` в таблице `aircrafts_log` и изменим запросы из главы связанные с этой таблицей.

```PGSQL 
ALTER TABLE aircrafts_log ALTER COLUMN when_add SET DEFAULT now();

-- Запрос 1
WITH add_row AS (
  INSERT INTO aircrafts_tmp 
  SELECT * 
  FROM 
    aircrafts RETURNING *
) 
INSERT INTO aircrafts_log ( aircraft_code, model, range, operation) 
SELECT add_row.aircraft_code, add_row.model, add_row.range, 'INSERT' 
FROM add_row;

-- Запрос 2
WITH add_row AS
( INSERT INTO aircrafts_tmp
  VALUES ( 'SU9', 'Sukhoi SuperJet-100', 3000 )
  ON CONFLICT DO NOTHING
  RETURNING *
)
INSERT INTO aircrafts_log ( aircraft_code, model, range, operation) 
SELECT add_row.aircraft_code, add_row.model, add_row.range, 'INSERT' 
FROM add_row;

-- Запрос 3
WITH update_row AS
( UPDATE aircrafts_tmp
  SET range = range * 1.2
  WHERE model ~ '^Bom'
  RETURNING *
)
INSERT INTO aircrafts_log ( aircraft_code, model, range, operation) 
SELECT ur.aircraft_code, ur.model, ur.range, 'UPDATE' 
FROM update_row ur;

-- Запрос 4
WITH delete_row AS
( DELETE FROM aircrafts_tmp
  WHERE model ~ '^Bom'
  RETURNING *
)
INSERT INTO aircrafts_log ( aircraft_code, model, range, operation) 
SELECT dr.aircraft_code, dr.model, dr.range, 'DELETE' 
FROM delete_row dr;
```

**Ответ на вопрос №2**

В данном случае можно сделать выборку по всем столбцам уже форматированого запроса

```PGSQL
WITH add_row AS
( INSERT INTO aircrafts_tmp
  SELECT * FROM aircrafts
  RETURNING aircraft_code, model, range, current_timestamp, 'INSERT'
)
INSERT INTO aircrafts_log SELECT * FROM add_row;

SELECT * FROM aircrafts_log; -- Получим 

/*
 aircraft_code |        model        | range |          when_add          | operation 
---------------+---------------------+-------+----------------------------+-----------
 773           | Boeing 777-300      | 11100 | 2021-12-01 03:13:20.675179 | INSERT
 763           | Boeing 767-300      |  7900 | 2021-12-01 03:13:20.675179 | INSERT
 SU9           | Sukhoi SuperJet-100 |  3000 | 2021-12-01 03:13:20.675179 | INSERT
 320           | Airbus A320-200     |  5700 | 2021-12-01 03:13:20.675179 | INSERT
 321           | Airbus A321-200     |  5600 | 2021-12-01 03:13:20.675179 | INSERT
 319           | Airbus A319-100     |  6700 | 2021-12-01 03:13:20.675179 | INSERT
 733           | Boeing 737-300      |  4200 | 2021-12-01 03:13:20.675179 | INSERT
 CN1           | Cessna 208 Caravan  |  1200 | 2021-12-01 03:13:20.675179 | INSERT
 CR2           | Bombardier CRJ-200  |  2700 | 2021-12-01 03:13:20.675179 | INSERT
(9 rows)
*/

```
**Ответ на вопрос №4**

Предупредим конфликты при добавлении в таблицу `Seats` строк с одинаковыми первичными ключами

```PGSQL
-- Создадим копию таблицы Seats
CREATE TEMP TABLE Seats_tmp ( LIKE Seats INCLUDING CONSTRAINTS INCLUDING INDEXES );
INSERT INTO Seats_tmp SELECT * FROM Seats;

INSERT INTO Seats_tmp VALUES( 319,'2A', 'Business'); -- Ошибка
/* ERROR:  duplicate key value violates unique constraint "seats_tmp_pkey"
DETAIL:  Key (aircraft_code, seat_no)=(319, 2A) already exists. */

-- Решение через имена столбцов
INSERT INTO Seats_tmp VALUES( 319,'2A', 'Business') 
  ON CONFLICT (aircraft_code, seat_no) 
  DO UPDATE SET fare_conditions = excluded.fare_conditions 
  RETURNING *;
-- Получим
/*
 aircraft_code | seat_no | fare_conditions 
---------------+---------+-----------------
 319           | 2A      | Business  */

-- Решение через ON CONSTRAINT
INSERT INTO Seats_tmp VALUES( 319,'2A', 'Business') 
  ON CONFLICT ON CONSTRAINT seats_tmp_pkey
  DO UPDATE SET fare_conditions = excluded.fare_conditions 
  RETURNING *;
-- Получим
/*
 aircraft_code | seat_no | fare_conditions 
---------------+---------+-----------------
 319           | 2A      | Business  */
```

## ДЗ №7. Ответы на контрольные вопросы и задания к главе №8 ##

**Ответ на вопрос №1**
Вставка одинаковых строк с NULL в уникальном индексе выполниться, так как над нул нельзя выполнить некоторые операции в том числе сравнивать их. Проверим
```PGSQL
CREATE TABLE temp1 (column1 VARCHAR, column2 VARCHAR);
CREATE UNIQUE INDEX ON temp1 (column1, column2);

INSERT INTO temp1 VALUES ('asfds', 'dsf'); -- успешно
INSERT INTO temp1 VALUES ('asfds', 'dsf'); -- ошибка
/*
ERROR:  duplicate key value violates unique constraint "temp1_column1_column2_idx"
DETAIL:  Key (column1, column2)=(asfds, dsf) already exists. */
INSERT INTO temp1 VALUES ('asfds', NULL); -- успешно
-- INSERT 0 1
INSERT INTO temp1 VALUES ('asfds', NULL); -- успешно
-- INSERT 0 1
```
**Ответ на вопрос №3**

Время без индексов

```PGSQL
SELECT count( * ) FROM ticket_flights WHERE fare_conditions = 'Comfort';
-- Time: 29.666 ms

/* count 
-------
 17291
(1 row) */

SELECT count( * ) FROM ticket_flights WHERE fare_conditions = 'Business';
-- Time: 31.976 ms

/* count 
-------
 107642
(1 row) */

SELECT count( * ) FROM ticket_flights WHERE fare_conditions = 'Economy';
-- Time: 38.417 ms

/* count 
-------
 920793
(1 row) */

SELECT count( * ) FROM ticket_flights WHERE flight_id < 100;
-- Time: 26.963 ms

/* count 
-------
  3938
(1 row) */ 

SELECT count( * ) FROM ticket_flights WHERE flight_id < 1000;
-- Time: 27.451 ms

/* count 
-------
 51654
(1 row) */

SELECT count( * ) FROM ticket_flights WHERE flight_id < 10000;
-- Time: 31.618 ms

/* count  
--------
 453988
(1 row) */

SELECT count( * ) FROM ticket_flights WHERE amount < 10000;
-- Time: 42.735 ms

/* count  
--------
 359344
(1 row) */

SELECT count( * ) FROM ticket_flights WHERE amount < 50000;
-- Time: 49.510 ms

/* count  
--------
 972079
(1 row) */

SELECT count( * ) FROM ticket_flights WHERE amount < 100000;
-- Time: 48.173 ms
/*  count  
---------
 1031981
(1 row) */



```

Создадим индексы и посмотрим на результаты

```PGSQL
CREATE INDEX
ON ticket_flights (fare_conditions);

SELECT count( * ) FROM ticket_flights WHERE fare_conditions = 'Comfort';
-- Time: 4.428 ms

SELECT count( * ) FROM ticket_flights WHERE fare_conditions = 'Business';
-- Time: 5.578 ms

SELECT count( * ) FROM ticket_flights WHERE fare_conditions = 'Economy';
-- Time: 18.663 ms

CREATE INDEX
ON ticket_flights (flight_id);

SELECT count( * ) FROM ticket_flights WHERE flight_id < 100;
-- Time: 2.165 ms

SELECT count( * ) FROM ticket_flights WHERE flight_id < 1000;
-- Time: 4.383 ms

SELECT count( * ) FROM ticket_flights WHERE flight_id < 10000;
-- Time: 10.192 ms


CREATE INDEX
ON ticket_flights (amount);

SELECT count( * ) FROM ticket_flights WHERE amount < 10000;
-- Time: 17.611 ms

SELECT count( * ) FROM ticket_flights WHERE amount < 50000;
-- Time: 43.259 ms

SELECT count( * ) FROM ticket_flights WHERE amount < 100000;
-- Time: 43.287 ms

```

Видно, что чем больше процент выборки от всей таблице тем меньше пользы приносят индексы. Для класса конмфорт ускорение почти в 6 раз, а для эконом класса лишь - в два раза. Аналогичные результаты для столбцов `flight_id` и `amount`.

## ДЗ №8. Ответы на контрольные вопросы и задания к главе №9 ##

**Ответ на вопрос №2**

Попробуем отменить транзакцию на пером клиенте и посмотрим что получиться 

```PGSQL
-- Создадим коппию таблицы

CREATE TABLE aircrafts_tmp ( LIKE aircrafts INCLUDING CONSTRAINTS INCLUDING INDEXES );
INSERT INTO aircrafts_tmp SELECT * FROM aircrafts RETURNING *;

-- Запустим транзакцию в пером сеансе

BEGIN;

SELECT * FROM aircrafts_tmp WHERE range < 2000; -- 1 клиент

-- Получим вывод
/*
 aircraft_code |       model        | range 
---------------+--------------------+-------
 CN1           | Cessna 208 Caravan |  1200 */

-- Обновим некотрые поля
UPDATE aircrafts_tmp SET range = 2100 WHERE aircraft_code = 'CN1'; -- 1 клиент
UPDATE aircrafts_tmp SET range = 1900 WHERE aircraft_code = 'CR2'; -- 1 клиент

-- Запустим транзакцию на втором клиенте

BEGIN; -- 2 клиент 

SELECT * FROM aircrafts_tmp WHERE range < 2000; -- 2 клиент 
-- Получим вывод
/*
 aircraft_code |       model        | range 
---------------+--------------------+-------
 CN1           | Cessna 208 Caravan |  1200 */


-- Попытаемся удалить самолеты у которых длина полета меньше 2000

DELETE FROM aircrafts_tmp WHERE range < 2000; -- Встаем режим ожидания (2 клиент) 

-- Выполним отмену действий в транзакии у первого клиента
ROLLBAK; -- 1 клиент

-- Посмотрим что получилось на второй транзакции 
SELECT * FROM aircrafts_tmp;
/*
 aircraft_code |        model        | range 
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 SU9           | Sukhoi SuperJet-100 |  3000
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 319           | Airbus A319-100     |  6700
 733           | Boeing 737-300      |  4200
 CR2           | Bombardier CRJ-200  |  2700
(8 rows) */

-- Видно что была удалена строка с самолетом Cessna 208 Caravan

COMMIT; -- Завершим транзакцию (2 клиент)

```
Результат, впринципе, строка с самолетом `Cessna 208 Caravan` была заблокирована первой транзакцией и одновременно выбрана и ожидаема у второй. После отмены первой транзакции, она прошла по условиям запроса второй транзакции и была удалена. Строки обновленные а затем отменненные в первой транзации не были видны и отмеченные второй, следовательно, они и не могли обновиться. 

**Ответ на вопрос №3**

Обновленние может потеряться, например, если считать всю транзакцию атомарной. Например какая-то транзакция началась раньше, но из-за каких либо проблем строку заблокировало позже. Следовательно, более актуальным (по факту) будет обновление из позже начавшейся транзакции, однако, так, как, в данном случае, она раньше заблокирует нужную строку, а ранее начавшаяся транзакция ее перезатрет. Приведем пример

```PGSQL
BEGIN; -- Первая транзакция 

BEGIN; -- Вторая транзакция

UPDATE aircrafts_tmp SET range = 2100 WHERE aircraft_code = 'CR2'; -- Вторая транзакция, блокируем строку раньше первой

UPDATE aircrafts_tmp SET range = 2500 WHERE aircraft_code = 'CR2'; -- Первая транзакция, встаем на паузу

COMMIT; -- Вторая транзакция, записали самое актуальное значение исходя из свойств атомарности
COMMIT; -- Первая транзакция, перезаписали. Учитывая свойства атомарности, потерялось обновление данных.
```

## ДЗ №9. Ответы на контрольные вопросы и задания к главе №10 ##

**Ответ на вопрос №3**

Вспомним задачу из ДЗ 5 задания 7: было необходимо узнать между какими городами летает определенного типа самолет. Для наглядности добавим в вывод еще пару `('Сант-Петербург','Москва')` и затем присоеденим исходную выборку, проведем анализ плана запроса.

```PGSQL
EXPLAIN WITH f1 AS (
  SELECT 
    DISTINCT departure_city, 
    arrival_city 
  FROM 
    routes r 
    JOIN aircrafts a ON r.aircraft_code = a.aircraft_code 
  WHERE 
    a.model = 'Boeing 777-300' 
  ORDER BY 
    1
), 
f2 AS (
  SELECT 
    CASE WHEN f1.departure_city < f1.arrival_city THEN f1.departure_city ELSE f1.arrival_city END AS arrival_city, 
    CASE WHEN f1.departure_city >= f1.arrival_city THEN f1.departure_city ELSE f1.arrival_city END AS departure_city
  FROM 
    f1
) 
SELECT 
  DISTINCT * 
FROM 
  f2
UNION ALL
VALUES ('Сант-Петербург','Москва')
UNION ALL
SELECT DISTINCT * FROM f2;

-- Получим следующий план 

/*

                                           QUERY PLAN                                            
-------------------------------------------------------------------------------------------------
 Append  (cost=34.21..40.15 rows=159 width=64)
   CTE f2
     ->  Subquery Scan on f1  (cost=30.46..32.24 rows=79 width=64)
           ->  Unique  (cost=30.46..31.05 rows=79 width=34)
                 ->  Sort  (cost=30.46..30.66 rows=79 width=34)
                       Sort Key: r.departure_city, r.arrival_city
                       ->  Hash Join  (cost=1.12..27.97 rows=79 width=34)
                             Hash Cond: (r.aircraft_code = a.aircraft_code)
                             ->  Seq Scan on routes r  (cost=0.00..24.10 rows=710 width=38)
                             ->  Hash  (cost=1.11..1.11 rows=1 width=4)
                                   ->  Seq Scan on aircrafts a  (cost=0.00..1.11 rows=1 width=4)
                                         Filter: (model = 'Boeing 777-300'::text)
   ->  HashAggregate  (cost=1.98..2.77 rows=79 width=64)
         Group Key: f2.arrival_city, f2.departure_city
         ->  CTE Scan on f2  (cost=0.00..1.58 rows=79 width=64)
   ->  Result  (cost=0.00..0.01 rows=1 width=64)
   ->  HashAggregate  (cost=1.98..2.77 rows=79 width=64)
         Group Key: f2_1.arrival_city, f2_1.departure_city
         ->  CTE Scan on f2 f2_1  (cost=0.00..1.58 rows=79 width=64)
(19 rows) */
```
Изначально выполняется CTE f2 - отсортированные по столбцам в алфавитном порядке пары городов: сначала выполняется запрос на создание f1 (неуникальных пар по столбцам) а лишь затем f2. Можем видеть что эьтот CTE формируется в пределах от 30.46 до 32.24 временных единиц. 

После чего имеем три Append дочернего узла
1. Проводиться агрегация над прочитаными через CTE scan строками. Нижний предел выборки запроса 0.00 , это значит что запрос уже не выполяется а уже сохранен в памяти.
2. Просто пара значений
3. Почти идентичная первому дочернему узлу Append

Если сложить все нижние границы 30.46 + 1.98 + 0.00 + 1.98, то получим 34.42, это еще не учитывая времени на операцию объединения. Это можно объяснить тем, что 1 и 3 дочерние Append узлы можно выполнять параллельно.

**Ответ на вопрос №6**

Проанализируем пример с оконной функции из главы №6: необходимо проранжировать аэропорты по близости к северу в разных временных зонах отдельно.

```PGSQL
EXPLAIN SELECT 
  airport_name, 
  city, 
  round(latitude :: numeric, 2) AS ltd, 
  timezone, 
  rank() OVER (
    PARTITION BY timezone 
    ORDER BY 
      latitude DESC
  ) 
FROM 
  airports 
WHERE 
  timezone IN (
    'Asia/Irkutsk', 'Asia/Krasnoyarsk'
  ) 
ORDER BY 
  timezone, 
  rank;

-- Получим следующий план 

/*
                                        QUERY PLAN                                        
------------------------------------------------------------------------------------------
 Sort  (cost=8.11..8.14 rows=13 width=97)
   Sort Key: timezone, (rank() OVER (?))
   ->  WindowAgg  (cost=7.54..7.87 rows=13 width=97)
         ->  Sort  (cost=7.54..7.57 rows=13 width=57)
               Sort Key: timezone, latitude DESC
               ->  Seq Scan on airports  (cost=0.00..7.30 rows=13 width=57)
                     Filter: (timezone = ANY ('{Asia/Irkutsk,Asia/Krasnoyarsk}'::text[]))
(7 rows)

*/

```
Здесь имеем дочерний узел WindowAgg который задает оконную функцию. Он занимает имеено такое место потому как вышестоящий узел задает функцию rank через Over, а дочерние узлы уже отсортировали выборку в нужном для ранжирования пордке. Другими словами на этом месте необходимо лишь пройтись линейно и проставить rank что по прогнозу займет совсем не много времени.

**Ответ на вопрос №8**

Сравним два запроса:

```PGSQL
EXPLAIN ANALYZE 
SELECT 
  a.aircraft_code AS a_code, 
  a.model, 
  (
    SELECT 
      count(r.aircraft_code) 
    FROM 
      routes r 
    WHERE 
      r.aircraft_code = a.aircraft_code
  ) AS num_routes 
FROM 
  aircrafts a 
GROUP BY 
  1, 2 
ORDER BY  3 DESC;

-- Здесь получим следующий план и данные исполнения

/*
                                                       QUERY PLAN                                                        
-------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=236.31..236.34 rows=9 width=28) (actual time=3.311..3.315 rows=9 loops=1)
   Sort Key: ((SubPlan 1)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  HashAggregate  (cost=1.11..236.17 rows=9 width=28) (actual time=0.446..3.294 rows=9 loops=1)
         Group Key: a.aircraft_code
         Batches: 1  Memory Usage: 24kB
         ->  Seq Scan on aircrafts a  (cost=0.00..1.09 rows=9 width=20) (actual time=0.010..0.014 rows=9 loops=1)
         SubPlan 1
           ->  Aggregate  (cost=26.10..26.11 rows=1 width=8) (actual time=0.358..0.358 rows=1 loops=9)
                 ->  Seq Scan on routes r  (cost=0.00..25.88 rows=89 width=4) (actual time=0.075..0.339 rows=79 loops=9)
                       Filter: (aircraft_code = a.aircraft_code)
                       Rows Removed by Filter: 631
 Planning Time: 0.265 ms
 Execution Time: 3.392 ms
(14 rows)

*/

EXPLAIN ANALYZE 
SELECT 
  a.aircraft_code AS a_code, 
  a.model, 
  count(r.aircraft_code) AS num_routes 
FROM 
  aircrafts a 
  LEFT OUTER JOIN routes r ON r.aircraft_code = a.aircraft_code 
GROUP BY 
  1, 
  2 
ORDER BY 
  3 DESC;


-- Здесь получим следующий план и данные исполнения

/*
                                                          QUERY PLAN                                                          
------------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=31.83..31.85 rows=9 width=28) (actual time=1.485..1.491 rows=9 loops=1)
   Sort Key: (count(r.aircraft_code)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  HashAggregate  (cost=31.60..31.69 rows=9 width=28) (actual time=1.462..1.472 rows=9 loops=1)
         Group Key: a.aircraft_code
         Batches: 1  Memory Usage: 24kB
         ->  Hash Right Join  (cost=1.20..28.05 rows=710 width=24) (actual time=0.048..0.954 rows=711 loops=1)
               Hash Cond: (r.aircraft_code = a.aircraft_code)
               ->  Seq Scan on routes r  (cost=0.00..24.10 rows=710 width=4) (actual time=0.006..0.212 rows=710 loops=1)
               ->  Hash  (cost=1.09..1.09 rows=9 width=20) (actual time=0.029..0.031 rows=9 loops=1)
                     Buckets: 1024  Batches: 1  Memory Usage: 9kB
                     ->  Seq Scan on aircrafts a  (cost=0.00..1.09 rows=9 width=20) (actual time=0.014..0.019 rows=9 loops=1)
 Planning Time: 0.348 ms
 Execution Time: 1.574 ms
(14 rows)

*/

```

Несложно заменить что в первом случае для каждой модели самолета выполняется отдельные sql запрос для посчета числа маршрутов. У нас всего 9 самолетов и 9 циклов(или 9 выборок) для дочернего узла `Seq Scan on routes r`. На основе результатов трех запусков, это запрос в среднем выполняется за 3.3 секунды 

В втором случае создается хэш таблица с ключами-аэропортами и происходит сляние по хэшу с таблицей "маршруты" за 1 проход по данной таблице. В средгнем данный запрос выполняется за полторы минуты что более чем в два раза быстрее первого запроса.

Поэксперементируем с более объемной таблице чем "маршруты". Разница в быстродействии должна возрасти. Для эксперемента возмем таблицу `flights`:

```PGSQL 
EXPLAIN ANALYZE
SELECT 
  a.aircraft_code AS a_code, 
  a.model, 
  (
    SELECT 
      count(f.aircraft_code) 
    FROM 
      flights f 
    WHERE 
      f.aircraft_code = a.aircraft_code
  ) AS num_flights 
FROM 
  aircrafts a 
GROUP BY 
  1, 2 
ORDER BY 
  3 DESC;

-- Здесь получим следующий план и данные исполнения

/*
                                                                QUERY PLAN                                               
                 
-------------------------------------------------------------------------------------------------------------------------
-----------------
 Sort  (cost=7988.24..7988.26 rows=9 width=28) (actual time=34.173..34.176 rows=9 loops=1)
   Sort Key: ((SubPlan 1)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  Group  (cost=0.14..7988.09 rows=9 width=28) (actual time=5.593..34.160 rows=9 loops=1)
         Group Key: a.aircraft_code
         ->  Index Scan using aircrafts_pkey on aircrafts a  (cost=0.14..12.27 rows=9 width=20) (actual time=0.012..0.020
 rows=9 loops=1)
         SubPlan 1
           ->  Aggregate  (cost=886.19..886.20 rows=1 width=8) (actual time=3.790..3.790 rows=1 loops=9)
                 ->  Seq Scan on flights f  (cost=0.00..875.06 rows=4451 width=4) (actual time=0.445..3.621 rows=3680 loo
ps=9)
                       Filter: (aircraft_code = a.aircraft_code)
                       Rows Removed by Filter: 29441
 Planning Time: 0.282 ms
 Execution Time: 34.239 ms
(13 rows)
*/

EXPLAIN ANALYZE 
SELECT 
  a.aircraft_code AS a_code, 
  a.model, 
  count(f.aircraft_code) AS num_flights 
FROM 
  aircrafts a 
  LEFT OUTER JOIN flights f ON f.aircraft_code = a.aircraft_code 
GROUP BY 
  1, 2 
ORDER BY 
  3 DESC;

/*
                                                                QUERY PLAN                                               
                 
-------------------------------------------------------------------------------------------------------------------------
-----------------
 Sort  (cost=1102.98..1103.01 rows=9 width=28) (actual time=12.505..12.507 rows=9 loops=1)
   Sort Key: (count(f.aircraft_code)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  HashAggregate  (cost=1102.75..1102.84 rows=9 width=28) (actual time=12.499..12.501 rows=9 loops=1)
         Group Key: a.aircraft_code
         Batches: 1  Memory Usage: 24kB
         ->  Hash Right Join  (cost=1.20..924.73 rows=35605 width=24) (actual time=0.011..7.990 rows=33122 loops=1)
               Hash Cond: (f.aircraft_code = a.aircraft_code)
               ->  Seq Scan on flights f  (cost=0.00..786.05 rows=35605 width=4) (actual time=0.001..1.601 rows=33121 loo
ps=1)
               ->  Hash  (cost=1.09..1.09 rows=9 width=20) (actual time=0.007..0.007 rows=9 loops=1)
                     Buckets: 1024  Batches: 1  Memory Usage: 9kB
                     ->  Seq Scan on aircrafts a  (cost=0.00..1.09 rows=9 width=20) (actual time=0.003..0.004 rows=9 loop
s=1)
 Planning Time: 0.099 ms
 Execution Time: 12.528 ms
(14 rows) */
```

Наблюдаем похожую ситуацию, но разница быстродействия уже в три раза.


## ДЗ №10. Программирование на стороне сервера. ##

Задания из главы 4 следующего [руководства](http://www.morgunov.org/docs/inf_sys_admin.pdf)

**Подготовка БД**

```BASH 
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin wget http://www.morgunov.org/docs/inf_sys_admin_prg.tgz -P /var/sqldump/ # скачаем файл
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin tar -xvzf /var/sqldump/inf_sys_admin_prg.tgz -C /var/sqldump/task10/ # распакуем файлы 
$ docker exec -it task10-server /usr/bin/psql -f /var/sqldump/task10/UTF-8/adj_list.sql -U root -d ais # выполним экспорт
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin sh -c "rm -rf /var/sqldump/inf_sys_admin_prg.tgz /var/sqldump/task10/*" # эти файлы больше не нужны
```

**Подключение к базе данных**
```BASH
$ docker exec -it task10-server psql -U root -d ais
```
**Ответ на вопрос №12**

Проведем выборки из таблиц "Персонал" и "Организационная структура" а также из представления для реконструирования организационной структуры и из представления для построения всех путей сверху вниз.

```BASH
$ docker exec -it task10-server /usr/bin/psql -U root -d ais -c "SELECT * FROM Personnel"

# Получим вывод 
'emp_nbr | emp_name |         address         | birth_date 
---------+----------+-------------------------+------------
       0 | вакансия |                         | 2014-05-19
       1 | Иван     | ул. Любителей языка C   | 1962-12-01
       2 | Петр     | ул. UNIX гуру           | 1965-10-21
       3 | Антон    | ул. Ассемблерная        | 1964-04-17
       4 | Захар    | ул. им. СУБД PostgreSQL | 1963-09-27
       5 | Ирина    | просп. Программистов    | 1968-05-12
       6 | Анна     | пер. Перловый           | 1969-03-20
       7 | Андрей   | пл. Баз данных          | 1945-11-07
       8 | Николай  | наб. ОС Linux           | 1944-12-01
(9 rows) '

$ docker exec -it task10-server /usr/bin/psql -U root -d ais -c "SELECT * FROM Org_chart"

# Получим вывод 
'     job_title      | emp_nbr | boss_emp_nbr |  salary   
---------------------+---------+--------------+-----------
 Президент           |       1 |              | 1000.0000
 Вице-президент 1    |       2 |            1 |  900.0000
 Вице-президент 2    |       3 |            1 |  800.0000
 Архитектор          |       4 |            3 |  700.0000
 Ведущий программист |       5 |            3 |  600.0000
 Программист C       |       6 |            3 |  500.0000
 Программист Perl    |       7 |            5 |  450.0000
 Оператор            |       8 |            5 |  400.0000
(8 rows)'

$ docker exec -it task10-server /usr/bin/psql -U root -d ais -c "SELECT * FROM Personnel_org_chart"

# Получим вывод 
'emp_nbr |   emp   | boss_emp_nbr | boss  
---------+---------+--------------+-------
       1 | Иван    |              | 
       2 | Петр    |            1 | Иван
       3 | Антон   |            1 | Иван
       4 | Захар   |            3 | Антон
       5 | Ирина   |            3 | Антон
       6 | Анна    |            3 | Антон
       7 | Андрей  |            5 | Ирина
       8 | Николай |            5 | Ирина
(8 rows)'

$ docker exec -it task10-server /usr/bin/psql -U root -d ais -c "SELECT * FROM Create_paths" 

# Получим вывод       
'level1 | level2 | level3 | level4  
--------+--------+--------+---------
 Иван   | Антон  | Ирина  | Андрей
 Иван   | Антон  | Ирина  | Николай
 Иван   | Петр   |        | 
 Иван   | Антон  | Захар  | 
 Иван   | Антон  | Анна   | 
(5 rows)'
```


**Ответ на вопрос №13**

Поэксперементируем с циклами в иэрархии

```PGSQL
SELECT * FROM tree_test(); -- проверим наличие циклов

/* tree_test 
-----------
 Tree
(1 row) */

-- Tree  -> циклов нет

-- Создадим короткий цикл
BEGIN;
UPDATE Org_chart SET boss_emp_nbr = 8 WHERE emp_nbr = 6;
UPDATE Org_chart SET boss_emp_nbr = 6 WHERE emp_nbr = 5;
SELECT * FROM tree_test();
/* tree_test 
-----------
 Cycles
(1 row) */ -- Найден цикл
ROLLBACK;
SELECT * FROM tree_test(); -- Циклов нет

-- Создадим длинный цикл

BEGIN;
UPDATE Org_chart SET boss_emp_nbr = 2 WHERE emp_nbr = 3;
UPDATE Org_chart SET boss_emp_nbr = 8 WHERE emp_nbr = 2;
SELECT * FROM tree_test();
/* tree_test 
-----------
 Cycles
(1 row) */ -- Найден цикл
ROLLBACK;
```

**Ответ на вопрос №14**

Поэксперемнтируем с обходом организационной структуры с разыми функциями:

```PGSQL
SELECT * FROM up_tree_traversal( 6 );

-- Получим вывод
/*
 emp_nbr | boss_emp_nbr 
---------+--------------
       6 |            3
       3 |            1
       1 |             
(3 rows)*/

 SELECT * FROM up_tree_traversal( 8 );

 -- Получим вывод
/*
 emp_nbr | boss_emp_nbr 
---------+--------------
       8 |            5
       5 |            3
       3 |            1
       1 |             
(4 rows)*/


SELECT * FROM up_tree_traversal2( 6 ) AS (emp int, boss int);

-- Получим вывод
/*
 emp_nbr | boss_emp_nbr 
---------+--------------
       6 |            3
       3 |            1
       1 |             
(3 rows)*/

-- Сделаем обход по имени а не по номеру 

SELECT * FROM up_tree_traversal( ( SELECT emp_nbr FROM Personnel
WHERE emp_name = 'Андрей') );

-- Получим вывод
/*
 emp_nbr | boss_emp_nbr 
---------+--------------
       7 |            5
       5 |            3
       3 |            1
       1 |             
(4 rows) */

SELECT * FROM up_tree_traversal( ( SELECT emp_nbr FROM Personnel WHERE emp_name = 'Иван') );

-- Получим вывод
/*
 emp_nbr | boss_emp_nbr 
---------+--------------
       1 |             
(1 row)*/
```

**Ответ на вопрос №15**

Поэксперементируем с удалением поддеревьев организационной структуры:

```PGSQL
BEGIN;

-- Удалим поддерево с корнем в рабонике с номером 6
SELECT * FROM delete_subtree( 6 );

-- Посмотрим на актуальную иерархию
SELECT * FROM Personnel_org_chart;
/*
 emp_nbr |   emp   | boss_emp_nbr | boss  
---------+---------+--------------+-------
       1 | Иван    |              | 
       2 | Петр    |            1 | Иван
       3 | Антон   |            1 | Иван
       4 | Захар   |            3 | Антон
       7 | Андрей  |            5 | Ирина
       8 | Николай |            5 | Ирина
       5 | Ирина   |            3 | Антон
(7 rows) */

SELECT * FROM Create_paths;
/*
 level1 | level2 | level3 | level4  
--------+--------+--------+---------
 Иван   | Антон  | Ирина  | Андрей
 Иван   | Антон  | Ирина  | Николай
 Иван   | Петр   |        | 
 Иван   | Антон  | Захар  | 
(4 rows)*/

-- Удалим поддерево с корнем в рабонике с именем Антон

SELECT * FROM delete_subtree( ( SELECT emp_nbr FROM Personnel WHERE emp_name = 'Антон') );

-- Посмотрим на актуальную иерархию
SELECT * FROM Personnel_org_chart;
/*
 emp_nbr | emp  | boss_emp_nbr | boss 
---------+------+--------------+------
       1 | Иван |              | 
       2 | Петр |            1 | Иван
(2 rows)*/

SELECT * FROM Create_paths;
/*
 level1 | level2 | level3 | level4 
--------+--------+--------+--------
 Иван   | Петр   |        | 
(1 row)*/

ROLLBACK;
```

**Ответ на вопрос №16**

Поэсперементируем с функции удаления элемента и продвижения его дочерних элементов вверх:

```PGSQL
BEGIN;

SELECT * FROM delete_and_promote_subtree( 5 ); -- Удлим узел с рабоником номер 5

SELECT * FROM Personnel_org_chart;
/* emp_nbr |   emp   | boss_emp_nbr | boss  
---------+---------+--------------+-------
       1 | Иван    |              | 
       2 | Петр    |            1 | Иван
       3 | Антон   |            1 | Иван
       4 | Захар   |            3 | Антон
       6 | Анна    |            3 | Антон
       7 | Андрей  |            3 | Антон
       8 | Николай |            3 | Антон
(7 rows) */

SELECT * FROM Create_paths;
/*
 level1 | level2 | level3  | level4 
--------+--------+---------+--------
 Иван   | Петр   |         | 
 Иван   | Антон  | Захар   | 
 Иван   | Антон  | Николай | 
 Иван   | Антон  | Анна    | 
 Иван   | Антон  | Андрей  | 
(5 rows) */

-- Видно что узел с номер 5 исчез а узлы 7 и 8 стали дочерними узлу 3

-- Удалим узел с рабоником "Антон" и продвинем его дочерние узлы
SELECT * FROM delete_and_promote_subtree( ( SELECT emp_nbr FROM Personnel WHERE emp_name = 'Антон') ); 

-- Посмотрим что получилось 

SELECT * FROM Personnel_org_chart;
/*
 emp_nbr |   emp   | boss_emp_nbr | boss 
---------+---------+--------------+------
       1 | Иван    |              | 
       2 | Петр    |            1 | Иван
       4 | Захар   |            1 | Иван
       6 | Анна    |            1 | Иван
       7 | Андрей  |            1 | Иван
       8 | Николай |            1 | Иван
(6 rows) */

SELECT * FROM Create_paths;
/*
 level1 | level2  | level3 | level4 
--------+---------+--------+--------
 Иван   | Андрей  |        | 
 Иван   | Анна    |        | 
 Иван   | Николай |        | 
 Иван   | Петр    |        | 
 Иван   | Захар   |        | 
(5 rows)*/

-- как и ожидалось все оставшиеся узлы стали дочерними узлу 1

ROLLBACK;
```

**Ответ на вопрос №17**

Изменим функцию `Create_paths` для работы с 5 уровнями иерархии

```PGSQL
CREATE OR REPLACE VIEW Create_paths ( level1, level2, level3, level4, level5 ) AS
  SELECT O1.emp AS e1, O2.emp AS e2, O3.emp AS e3, O4.emp AS e4, O5.emp AS e5
  FROM Personnel_org_chart AS O1
  LEFT OUTER JOIN Personnel_org_chart AS O2 ON O1.emp = O2.boss
  LEFT OUTER JOIN Personnel_org_chart AS O3 ON O2.emp = O3.boss 
  LEFT OUTER JOIN Personnel_org_chart AS O4 ON O3.emp = O4.boss
  LEFT OUTER JOIN Personnel_org_chart AS O5 ON O4.emp = O5.boss
  WHERE O1.emp = 'Иван';

-- Посмотрим актуальную иерархию
SELECT * FROM Create_paths;
/*
 level1 | level2 | level3 | level4  | level5 
--------+--------+--------+---------+--------
 Иван   | Антон  | Анна   |         | 
 Иван   | Антон  | Захар  |         | 
 Иван   | Петр   |        |         | 
 Иван   | Антон  | Ирина  | Николай | 
 Иван   | Антон  | Ирина  | Андрей  | 
(5 rows) */

-- Переместим Анну на 5 уровень иерархии
UPDATE Org_chart SET boss_emp_nbr = 8 WHERE emp_nbr = 6;

SELECT * FROM Create_paths;
/*
level1 | level2 | level3 | level4  | level5 
--------+--------+--------+---------+--------
 Иван   | Антон  | Ирина  | Николай | Анна
 Иван   | Антон  | Захар  |         | 
 Иван   | Петр   |        |         | 
 Иван   | Антон  | Ирина  | Андрей  | 
(4 rows)*/
```

**Ответ на вопрос №18**

Создадим простую курсорную функцию с помощью которой выведем имена всех сотрудников в строчку:

```PGSQL
CREATE OR REPLACE FUNCTION curs_fn()
    RETURNS text
AS $$
DECLARE 
          cr CURSOR FOR SELECT emp_name FROM Personnel;
          result text:='';
          temp VARCHAR(10);
BEGIN
          OPEN cr;
          LOOP
             FETCH cr INTO temp;
             IF NOT FOUND THEN EXIT;END IF;
          result:= result || ' ' || temp;
          END LOOP;
          CLOSE cr;
          RETURN result;
END;
$$
LANGUAGE 'plpgsql';

select * from  curs_fn();
/*
                          curs_fn                          
-----------------------------------------------------------
  вакансия Иван Петр Антон Захар Ирина Анна Андрей Николай
(1 row)*/
```

## ДЗ №11. Полнотекстовый поиск ##

Добавим в полнотекстовый поиск ранжирование и выделение слов по котором были найдены строки:

**Скачем и распакуем файлы с каталогом книг**

```BASH 
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin wget https://edu.postgrespro.ru/sqlprimer/sqlprimer-2019-msu-10.tgz -P /var/sqldump/ # скачаем файл
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin tar -xvzf /var/sqldump/sqlprimer-2019-msu-10.tgz -C /var/sqldump/task11/ # распакуем файлы 
```

**Подключение к базе данных**
```BASH
$ docker exec -it task11-server psql -U root -d books
```

**Непосредственно реализация**
```PGSQL
-- Создаем таблицу в котрой будем хранить все данные

CREATE TABLE books
(   book_id integer PRIMARY KEY,
    book_description text
);

-- Заполняем таблицу 
COPY books FROM '/var/sqldump/task11/books3.txt';

-- Создадим таблицу для хранения представления tsvector 
ALTER TABLE books ADD COLUMN ts_description tsvector;

-- Создадим нормализированное описание для каждой строчки и сохраним в вышесозданный столбец 
UPDATE books
SET ts_description = to_tsvector( 'russian', book_description );

-- Создадим индекс GIN по нормализированному описанию
CREATE INDEX books_idx ON books USING GIN ( ts_description );

-- Проверим работоспособность 
SELECT book_description FROM books WHERE ts_description @@ to_tsquery('russian', 'Python' );
/*                                                                                      book_description                                                                                       
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Любанович, Билл. Простой Python [Текст] : современный стиль программирования / Билл Любанович. - Санкт-Петербург ; Москва ; Екатеринбург : Питер, 2018. - 476 с.
 Силен, Дэви. Основы Data Science и Big Data. Python и наука о данных [Текст] / Дэви Силен, Арно Мейсман, Мохамед Али ; [пер. с англ. Е. Матвеева]. - Санкт-Петербург : Питер, 2017. - 334 с.
 Доусон, Майкл. Программируем на Python [Текст] / Майкл Доусон ; [пер. с англ.: В. Порицкий]. - Москва [и др.] : Питер, 2016. - 414 с.
 Лутц, Марк. Изучаем Python [Текст] / Марк Лутц ; [пер. с англ. А. Киселева]. - Санкт-Петербург ; Москва : Символ : Символ-Плюс, 2011. - 1272 с.
(4 rows)
*/


-- Создадим функцию для поиска книг с ранжирование и выделением совпадающих слов

CREATE 
OR REPLACE FUNCTION find_book(
  IN user_query text, IN numbers INTEGER
) RETURNS TABLE(
  book_id INTEGER, book_description text, 
  rank float4
) AS $$ BEGIN RETURN QUERY 
SELECT 
  books.book_id, 
  ts_headline(
    'russian', books.book_description, 
    query, 'MaxFragments=10, MaxWords=7, MinWords=3, StartSel=<<, StopSel=>>'
  ), 
  ts_rank_cd(books.ts_description, query) AS rank 
FROM 
  books, 
  to_tsquery(
    'russian', user_query
  ) query 
WHERE 
  query @@ ts_description 
ORDER BY 
  rank DESC 
LIMIT 
  numbers;
END;
$$ LANGUAGE plpgsql;


-- Проверим работспособность 

SELECT * FROM find_book('приложения & PHP', 3);
/*
 book_id |                                                      book_description                                                       |    rank     
---------+-----------------------------------------------------------------------------------------------------------------------------+-------------
     775 | веб-<<приложений>> с помощью <<PHP>> и MySQL                                                                                | 0.033333335
     682 | <<PHP>> и MySQL. Разработка Web-<<приложений>> [Текст                                                                       | 0.016666668
     759 | Колисниченко, Денис Николаевич. <<PHP>> 5/6 и MySQL ... Разработка Web-<<приложений>> [Текст] / Денис Колисниченко. - Санкт |      0.0125
(3 rows)
*/

SELECT * FROM find_book('базы | данных | SQL', 5);
/*
 book_id |                                                                               book_description                                                                                | rank 
---------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------
     690 | Агальцов, Виктор Петрович. <<Базы>> <<данных>> [Текст] : [учебник ... Кн. 1 : Локальные <<базы>> <<данных>>                                                                   |  0.4
     462 | обеспечения систем обработки <<данных>> Сигма [Препринт]. - Красноярск ... пользователя. Программы обработки <<баз>> <<данных>>. Программы построения ... выходных <<данных>> |  0.4
     463 | обеспечения систем обработки <<данных>> Сигма [Препринт]. - Красноярск ... Программы структурной реорганизации <<баз>> <<данных>>                                             |  0.3
     460 | обеспечения систем обработки <<данных>> Сигма [Препринт]. - Красноярск ... пользователя. Программы обработки <<баз>> <<данных>>. Функции                                      |  0.3
     459 | обеспечения систем обработки <<данных>> Сигма [Препринт]. - Красноярск ... пользователя. Программы обработки <<баз>> <<данных>>. Форматная выдача                             |  0.3

(5 rows)
 */

SELECT * FROM find_book('Oracle', 3);
/*
 book_id |                       book_description                       | rank 
---------+--------------------------------------------------------------+------
      72 | Мак-Локлин, Майкл. <<Oracle>> Database 11g. Программирование |  0.1
(1 row) */
```



**Удалим лишние файлы**
```BASH 
$ docker exec -it --user "$(id -u):$(id -g)" pgadmin sh -c "rm -rf /var/sqldump/sqlprimer-2019-msu-10.tgz /var/sqldump/task11/*" # эти файлы больше не нужны
```

## Финальная работа ##

Отчет располается в файле [FINAL_WORK.md](https://github.com/alex-12345/posgreSQL/blob/main/FINAL_WORK.md)