# Домашняя работа: Ресторан.
## Предисловие
Чтож, очередное домашние задание? Да! В этот постараюсь как можно меньше использовать интернета и нейросетей. Всё ж, теории достаточно, а вот практики мало. Ну и в этот раз задание я оставлю не только в самом репозитории, но и пропишу прям здесь. Ну-с, приятного промиотра!

## Задание
### РЕСТОРАН.  
Создаваемая база данных предназначена для учета деятельности ресторана, а именно: движение и учет блюд, ингредиентов, напитков, денег, заказов и пр. 
Блюда ресторана готовятся из ингредиентов.

Например, для приготовления омлета требуется:
    • 1 яйцо 
    • 20 г сливочного масла 
    • 50 г лука 
    • 10 г соли 

Имеется справочник блюд:
    • Наименование блюда
    • Состав (ингредиент, количество, единица измерения)
    • Стоимость

Заказ включает сведения:
    • Дата
    • Клиент
    • Официант
    • Перечень блюд, их количество, цена
    • Общая стоимость заказа

Спроектировать базу данных для учета работы ресторана. 

## Немного слов об задании
Какой кошмар и ужас. Ладно, переживём. Ну-с, тут конечно не так уж много и страшно, но поныть-то хочется ¡-¡. Самое ужасное, что я до сих пор не выучил команды git ._. 

# Реализация задания
## Подготовка
И так, немного подумав, я понял, что тут очень много разных БД придётся делать... За что ¡-¡ ? На этом моменте нужно определиться с тем, какие БД должны быть, а именно: Бухгалтерия, склад, заказы, рецепты, персонал, столы(брони, поломки и т.д.), поставки и календарь(мероприятия, выходные, смены и т.д.). Немного, конечно, накинул много БД, но зато так интереснее. Ну-с, время запускать pgAdmin и стартовать! Вот, к слову, момент с тем, что изначально я ставил PosgreSQL на Debian, но работаю на Windows. Дело в том, что мне влом возиться с неттопом, который лежит в дальнем углу ящика и вся гарнитура к нему раскидана в разных уголках квартиры. Ставить эмулятор я не вижу особо смысла, ведь, а какая разница между работой на разных операционках? Только в приложении, на котором я буду писать.

## SQL запросы. Создание БД.
Начнём с создание таблицы "Персонал", ну и сразу вместе добавил "Стол":
```sql
CREATE TABLE staff {
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL, --ну, я надеюсь, что мы не повтречаем какого-нибудь, кто по имени зайдёт за лимиты. Всё ж, оптимизацию никто не отменял
    last_name VARCHAR(50) NOT NULL,
    job_title VARCHAR(50) NOT NULL,
    email TEXT, --ну, тут уж без лимитов, кто ж знает, какой у человека email
    phone SMALLINT
};

CREATE TABLE tables_of_restrourant { --tables зарезервированное слово
    id SERIAL PRIMARY KEY,
    capacity SMALLINT, --полезно для брони
    status_of_table VARCHAR(50) NOT NULL DEFAULT 'free', --увы, но status зарезервированное слово самой PosgreSQL
};
```

И как вы думаете, что произошло? Ошибка! Потому, что я использовал фигурные скобки, вместо обычных. Ну и лищняя запятая в конце, на 13 строчке

```sql
CREATE TABLE staff (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL, --ну, я надеюсь, что мы не повтречаем какого-нибудь, кто по имени зайдёт за лимиты. Всё ж, оптимизацию никто не отменял
    last_name VARCHAR(50) NOT NULL,
    job_title VARCHAR(50) NOT NULL,
    email TEXT, --ну, тут уж без лимитов, кто ж знает, какой у человека email
    phone SMALLINT
);

CREATE TABLE tables_of_restrourant ( --tables зарезервированное слово
    id SERIAL PRIMARY KEY, --тут было бы хорошо использовать SMALLSERIAL, но оставим по стандарту
    capacity SMALLINT, --полезно для брони
    status_of_table VARCHAR(50) NOT NULL --увы, но status зарезервированное слово самой PosgreSQL
);
```

Теперь-то всё работает. Ну, тогда продолжаем. Теперь таблицы "Блюда", "Ингридиенты" и "Рецепты":

```sql
CREATE TABLE dishes (
    id SERIAL PRIMARY KEY,
    name_of_dishes TEXT NOT NULL,
    price MONEY
);

CREATE TABLE ingredients (
    id SERIAL PRIMARY KEY,
    name_of_ingridients TEXT NOT NULL,
    unit VARCHAR(20) DEFAULT 'grams',
    stock_quantity SMALLINT DEFAULT 0
);

CREATE TABLE recipes (
    dish_id INT REFERENCES dishes(id),
    ingridient_id INT REFERENCES ingridients(id),
    quantity TEXT --cколько чего и на что
);
```

Ну и продолжим с "Бронь", "Заказы" и "Содержимое заказа":

```sql
CREATE TABLE bookings (
    id SERIAL PRIMARY KEY,
    table_id INT REFERENCES tables_of_restrourant(id),
    client_name VARCHAR(50),
    booking_time TIMESTAMP
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    table_id INT REFERENCES tables_of_restrourant(id),
    waiter_id INT REFERENCES staff(id),
    created_at TIMESTAMP DEFAULT NOW(),
    additional_info TEXT, --на случай, если есть какие-то предпочтения
    finale_price MONEY
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id),
    dish_id INT REFERENCES dishes(id),
    quantity SMALLINT, --количество порций
    price_at_sale MONEY
);
```