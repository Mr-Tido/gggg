-- ============================================================
--  Интернет-магазин "Кубаночка" — PostgreSQL Database
--  Версия: 1.0
--  Страницы дизайна: Главная, Каталог, Карточка товара,
--  Корзина/Заказы, Личный кабинет, Новости, О компании, Контакты
-- ============================================================

-- ============================================================
--  РАСШИРЕНИЯ
-- ============================================================
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- ============================================================
--  ОЧИСТКА (для повторного запуска)
-- ============================================================
DROP TABLE IF EXISTS
    order_items, orders, cart_items, carts,
    product_images, product_attributes, products,
    subcategories, categories,
    news_tags, news, tags,
    users, branches,
    pages, banners, popular_categories
CASCADE;

-- ============================================================
--  1. ФИЛИАЛЫ
-- ============================================================
CREATE TABLE branches (
    id          SERIAL PRIMARY KEY,
    city        VARCHAR(100)  NOT NULL,
    address     TEXT          NOT NULL,
    phone       VARCHAR(30),
    email       VARCHAR(100),
    is_active   BOOLEAN       DEFAULT TRUE,
    created_at  TIMESTAMPTZ   DEFAULT NOW()
);

INSERT INTO branches (city, address, phone, email) VALUES
('Владивосток', 'ул. Светланская, 45',         '+7 (423) 245-67-89', 'vlad@kubanochka.ru'),
('Владивосток', 'ул. Луговая, 77',             '+7 (423) 311-22-33', 'lugov@kubanochka.ru'),
('Артём',       'пр. Красноармейский, 12',     '+7 (423) 560-11-44', 'artyom@kubanochka.ru'),
('Находка',     'ул. Портовая, 3',             '+7 (4236) 72-15-80', 'nakhodka@kubanochka.ru'),
('Уссурийск',   'ул. Ленина, 28',              '+7 (4234) 32-45-67', 'ussur@kubanochka.ru');

-- ============================================================
--  2. ПОЛЬЗОВАТЕЛИ
-- ============================================================
CREATE TABLE users (
    id              SERIAL PRIMARY KEY,
    email           VARCHAR(150) UNIQUE NOT NULL,
    password_hash   TEXT         NOT NULL,
    first_name      VARCHAR(80),
    last_name       VARCHAR(80),
    phone           VARCHAR(30),
    city            VARCHAR(100),
    address         TEXT,
    role            VARCHAR(20)  NOT NULL DEFAULT 'customer'  -- customer | admin | manager
                    CHECK (role IN ('customer','admin','manager')),
    is_active       BOOLEAN      DEFAULT TRUE,
    created_at      TIMESTAMPTZ  DEFAULT NOW(),
    updated_at      TIMESTAMPTZ  DEFAULT NOW()
);

INSERT INTO users (email, password_hash, first_name, last_name, phone, city, address, role) VALUES
('admin@kubanochka.ru',   gen_salt('bf'),  'Иван',    'Петров',      '+7 (912) 000-00-01', 'Владивосток', 'ул. Светланская, 1-1',   'admin'),
('manager@kubanochka.ru', gen_salt('bf'),  'Мария',   'Сидорова',    '+7 (912) 000-00-02', 'Владивосток', 'ул. Луговая, 5-3',        'manager'),
('ivanov@mail.ru',        gen_salt('bf'),  'Алексей', 'Иванов',      '+7 (924) 111-22-33', 'Владивосток', 'ул. Пушкина, 10-25',      'customer'),
('petrova@yandex.ru',     gen_salt('bf'),  'Ольга',   'Петрова',     '+7 (924) 222-33-44', 'Владивосток', 'пр. Партизанский, 8-12',  'customer'),
('sidorov@gmail.com',     gen_salt('bf'),  'Дмитрий', 'Сидоров',     '+7 (914) 333-44-55', 'Артём',       'ул. Кирова, 3-7',         'customer'),
('kuznecova@mail.ru',     gen_salt('bf'),  'Елена',   'Кузнецова',   '+7 (924) 444-55-66', 'Находка',     'ул. Дзержинского, 15-2',  'customer'),
('morozov@yandex.ru',     gen_salt('bf'),  'Андрей',  'Морозов',     '+7 (924) 555-66-77', 'Уссурийск',   'ул. Ленина, 44-9',        'customer'),
('sokolova@gmail.com',    gen_salt('bf'),  'Наталья', 'Соколова',    '+7 (914) 666-77-88', 'Владивосток', 'ул. Русская, 64-18',      'customer');

-- ============================================================
--  3. КАТЕГОРИИ И ПОДКАТЕГОРИИ
-- ============================================================
CREATE TABLE categories (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(150) NOT NULL,
    slug        VARCHAR(150) UNIQUE NOT NULL,
    icon_url    TEXT,
    sort_order  INT          DEFAULT 0,
    is_popular  BOOLEAN      DEFAULT FALSE,   -- отображается в "Популярных категориях"
    is_active   BOOLEAN      DEFAULT TRUE,
    created_at  TIMESTAMPTZ  DEFAULT NOW()
);

CREATE TABLE subcategories (
    id          SERIAL PRIMARY KEY,
    category_id INT          NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    name        VARCHAR(150) NOT NULL,
    slug        VARCHAR(150) UNIQUE NOT NULL,
    sort_order  INT          DEFAULT 0,
    is_active   BOOLEAN      DEFAULT TRUE,
    created_at  TIMESTAMPTZ  DEFAULT NOW()
);

INSERT INTO categories (name, slug, sort_order, is_popular) VALUES
('Овощи, фрукты, ягоды, зелень, грибы',   'ovoshi-frukty',         1,  TRUE),
('Молоко, сыр, яйцо',                      'molochnoe',             2,  TRUE),
('Мясо, птица, колбасы',                   'myaso',                 3,  TRUE),
('Рыба, икра, морепродукты',               'ryba',                  4,  FALSE),
('Крупы, бобовые',                         'krupy',                 5,  FALSE),
('Макаронные изделия',                     'makarony',              6,  FALSE),
('Мука, всё для выпечки',                  'muka-vypechka',         7,  FALSE),
('Соусы, майонез',                         'sousy',                 8,  FALSE),
('Консервы, варенье, мёд',                 'konservy',              9,  FALSE),
('Кулинария',                              'kulinariya',           10,  FALSE),
('Вода, соки, напитки',                    'napitki',              11,  FALSE),
('Хлеб, хлебцы, выпечка',                 'xleb',                 12,  FALSE),
('Сладости, торты, пирожные',              'sladosti',             13,  FALSE),
('Чай, кофе, какао',                       'chaj-kofe',            14,  FALSE),
('Детские товары',                         'detskie',              15,  FALSE),
('Замороженные продукты, мороженое',        'zamorozhennye',        16,  FALSE),
('Здоровое питание',                       'zdorovoe-pitanie',     17,  FALSE),
('Ягоды',                                  'yagody',               18,  TRUE);

INSERT INTO subcategories (category_id, name, slug, sort_order) VALUES
-- Овощи, фрукты, ягоды
(1,  'Овощи',                     'ovoshi',               1),
(1,  'Фрукты',                    'frukty',               2),
(1,  'Ягоды',                     'yagody-fresh',         3),
(1,  'Зелень',                    'zelen',                4),
(1,  'Грибы',                     'griby',                5),
(1,  'Орехи, сухофрукты, семечки','orehi',                6),
-- Молочная
(2,  'Молоко',                    'moloko',               1),
(2,  'Сыр',                       'syr',                  2),
(2,  'Яйцо',                      'yajco',                3),
(2,  'Йогурт, творог',            'jogurt',               4),
(2,  'Масло сливочное',           'maslo-slivochnoe',     5),
-- Мясная
(3,  'Говядина',                  'govyadina',            1),
(3,  'Свинина',                   'svinina',              2),
(3,  'Птица',                     'ptica',                3),
(3,  'Колбасы, сосиски',          'kolbasy',              4),
(3,  'Полуфабрикаты',             'polufabrikaty',        5),
-- Рыба
(4,  'Рыба свежемороженая',       'ryba-zamorozhennaya',  1),
(4,  'Икра',                      'ikra',                 2),
(4,  'Морепродукты',              'moreprodukty',         3),
-- Крупы
(5,  'Крупы',                     'krupy-sub',            1),
(5,  'Бобовые',                   'bobovye',              2),
-- Соусы
(8,  'Кетчуп, томатная паста',    'ketchup',              1),
(8,  'Майонез',                   'majonez',              2),
(8,  'Соусы',                     'sousy-sub',            3),
(8,  'Заправки для салата',       'zapravki',             4),
(8,  'Горчица, хрен',             'gorchica',             5),
(8,  'Уксус',                     'uksus',                6),
(8,  'Растительное масло',        'maslo-rastitelnoe',    7),
-- Консервы
(9,  'Овощные, грибные',          'konservy-ovoshi',      1),
(9,  'Оливки, маслины',           'olivki',               2),
(9,  'Мясные',                    'konservy-myaso',       3),
(9,  'Рыбные, морепродукты',      'konservy-ryba',        4),
(9,  'Готовые блюда',             'gotovye-blyuda',       5),
(9,  'Мёд',                       'myod',                 6),
(9,  'Варенье, джем',             'varenye',              7),
(9,  'Сиропы',                    'siropy',               8),
(9,  'Сгущёнка',                  'sgushyonka',           9),
(9,  'Фрукты в сиропе',           'frukty-sirop',        10),
-- Мука
(7,  'Мука',                      'muka-sub',             1),
(7,  'Смеси для выпечки',         'smesi',                2),
(7,  'Всё для выпечки',           'dlya-vypechki',        3),
-- Напитки
(11, 'Вода',                      'voda',                 1),
(11, 'Соки',                      'soki',                 2),
(11, 'Напитки',                   'napitki-sub',          3),
-- Чай/кофе
(14, 'Чай',                       'chaj',                 1),
(14, 'Кофе',                      'kofe',                 2),
(14, 'Какао',                     'kakao',                3),
-- Сухие завтраки
(12, 'Сухие завтраки, хлопья',    'suhie-zavtraki',       1),
(12, 'Хлеб',                      'xleb-sub',             2);

-- ============================================================
--  4. ТОВАРЫ
-- ============================================================
CREATE TABLE products (
    id              SERIAL PRIMARY KEY,
    category_id     INT           REFERENCES categories(id)    ON DELETE SET NULL,
    subcategory_id  INT           REFERENCES subcategories(id) ON DELETE SET NULL,
    name            VARCHAR(255)  NOT NULL,
    slug            VARCHAR(255)  UNIQUE NOT NULL,
    description     TEXT,
    price           NUMERIC(10,2) NOT NULL CHECK (price >= 0),
    price_old       NUMERIC(10,2) CHECK (price_old >= 0),   -- зачёркнутая цена (акция)
    unit            VARCHAR(30)   NOT NULL DEFAULT 'шт',    -- шт | кг | л | г
    weight_volume   NUMERIC(10,3),                          -- вес/объём одной единицы
    stock_qty       INT           NOT NULL DEFAULT 0 CHECK (stock_qty >= 0),
    sku             VARCHAR(60)   UNIQUE,
    brand           VARCHAR(100),
    country         VARCHAR(80),
    is_active       BOOLEAN       DEFAULT TRUE,
    is_new          BOOLEAN       DEFAULT FALSE,
    is_promo        BOOLEAN       DEFAULT FALSE,
    is_popular      BOOLEAN       DEFAULT FALSE,
    rating          NUMERIC(3,2)  DEFAULT 0 CHECK (rating BETWEEN 0 AND 5),
    reviews_count   INT           DEFAULT 0,
    created_at      TIMESTAMPTZ   DEFAULT NOW(),
    updated_at      TIMESTAMPTZ   DEFAULT NOW()
);

CREATE TABLE product_images (
    id          SERIAL PRIMARY KEY,
    product_id  INT   NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    url         TEXT  NOT NULL,
    alt         VARCHAR(255),
    is_main     BOOLEAN DEFAULT FALSE,
    sort_order  INT     DEFAULT 0
);

CREATE TABLE product_attributes (
    id          SERIAL PRIMARY KEY,
    product_id  INT          NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    attr_name   VARCHAR(100) NOT NULL,
    attr_value  TEXT         NOT NULL
);

-- ---------- Товары ----------
INSERT INTO products
    (category_id, subcategory_id, name, slug, description, price, price_old, unit, weight_volume, stock_qty, sku, brand, country, is_promo, is_popular, rating, reviews_count)
VALUES
-- Соусы / Кетчупы
(8,  21, 'Кетчуп «Томатный "Кубаночка"» 310 г., дойпак 1/20',
    'ketchup-kubanochka-310',
    'Кетчуп из отборных томатов с натуральными специями. Без консервантов и ГМО. Идеален для мяса, пиццы и пасты.',
    42.30, 55.90, 'шт', 0.310, 200, 'SKU-001', 'Кубаночка', 'Россия', TRUE, TRUE, 4.50, 124),

-- Консервы / Огурцы
(9,  27, 'Корнишоны «Кубаночка» (1–3 см) 680 г. 1/8',
    'kornishony-kubanochka-680',
    'Огурчики упругие, довольно плотные, очень хрустящие. Рассол в меру пряный с низким содержанием уксуса. Отлично влияют на пищеварение, повышают аппетит. Богаты калием.',
    89.90, NULL, 'шт', 0.680, 150, 'SKU-002', 'Кубаночка', 'Россия', FALSE, TRUE, 4.70, 87),

-- Ягоды
(18, 3,  'Малина свежемороженая',
    'malina-zamorozhennaya',
    'Малина — очень вкусная и ароматная ягода. Обладает жаропонижающим и противовоспалительным эффектом, богата железом, медью и витаминами А, Е, РР, В2. Технология шоковой заморозки сохраняет вкус, аромат и пользу свежих ягод.',
    250.00, 350.00, 'кг', 1.000, 80, 'SKU-003', NULL, 'Россия', TRUE, TRUE, 4.80, 203),

-- Молочная
(2,  7,  'Молоко «Приморское» пастеризованное 3,2% 1 л',
    'moloko-primorskoe-1l',
    'Пастеризованное коровье молоко жирностью 3,2%. Изготовлено из натурального цельного молока без добавления сухого.',
    69.90, NULL, 'шт', 1.000, 300, 'SKU-004', 'Приморское', 'Россия', FALSE, FALSE, 4.30, 56),

(2,  8,  'Сыр «Российский» 45% 1 кг',
    'syr-rossijskij-45',
    'Классический полутвёрдый сыр с нежным сливочным вкусом и лёгкой кислинкой. Отлично подходит для бутербродов, запекания и салатов.',
    580.00, NULL, 'кг', 1.000, 50, 'SKU-005', 'Белый Медведь', 'Россия', FALSE, FALSE, 4.60, 78),

(2,  9,  'Яйцо куриное С1 10 шт',
    'yajco-kurinnoe-s1-10',
    'Столовые яйца первой категории. Продукт отборных кур-несушек. Богаты белком, витаминами D и B12.',
    99.00, NULL, 'шт', NULL, 400, 'SKU-006', NULL, 'Россия', FALSE, TRUE, 4.40, 110),

-- Мясная
(3,  14, 'Колбаса «Докторская» 400 г',
    'kolbasa-doktorskaya-400',
    'Варёная колбаса по ГОСТ. Нежная текстура, натуральный состав без сои и крахмала.',
    185.00, 220.00, 'шт', 0.400, 120, 'SKU-007', 'Приморский мясокомбинат', 'Россия', TRUE, FALSE, 4.20, 65),

(3,  13, 'Грудка куриная охлаждённая 1 кг',
    'grudka-kurinaya-1kg',
    'Охлаждённое куриное филе грудки без кожи и костей. Диетический продукт с высоким содержанием белка.',
    210.00, NULL, 'кг', 1.000, 90, 'SKU-008', NULL, 'Россия', FALSE, TRUE, 4.55, 143),

-- Рыба
(4,  17, 'Горбуша стейк замороженная 800 г',
    'gorbusa-stejk-800',
    'Дальневосточная горбуша, выловленная в Японском море. Стейки нарезаны равномерно. Богата омега-3 и витамином D.',
    349.00, 420.00, 'шт', 0.800, 60, 'SKU-009', NULL, 'Россия', TRUE, FALSE, 4.65, 91),

(4,  18, 'Икра горбуши «Экстра» 130 г',
    'ikra-gorbushi-extra-130',
    'Зернистая икра горбуши высшего сорта. Натуральный посол, без красителей и консервантов.',
    590.00, NULL, 'шт', 0.130, 40, 'SKU-010', 'Тихоокеанские деликатесы', 'Россия', FALSE, FALSE, 4.90, 212),

-- Крупы
(5,  19, 'Рис длиннозёрный пропаренный 1 кг',
    'ris-dlinnozernyj-1kg',
    'Длиннозёрный пропаренный рис не слипается при варке. Идеален для гарниров, плова и суши.',
    79.90, NULL, 'кг', 1.000, 250, 'SKU-011', 'Краснодарский', 'Россия', FALSE, FALSE, 4.35, 67),

(5,  19, 'Гречка ядрица 900 г',
    'grechka-yadrica-900',
    'Натуральная гречневая крупа без примесей. Богата железом, магнием, витаминами группы B.',
    95.00, NULL, 'шт', 0.900, 300, 'SKU-012', 'Алтайские Закрома', 'Россия', FALSE, TRUE, 4.70, 189),

-- Соки/напитки
(11, 41, 'Сок яблочный прямого отжима 1 л',
    'sok-yablochnyj-1l',
    'Сок прямого отжима без добавления воды и сахара. 100% натуральный яблочный сок из отборных яблок.',
    139.00, NULL, 'шт', 1.000, 180, 'SKU-013', 'Сады Приморья', 'Россия', FALSE, FALSE, 4.40, 55),

-- Хлеб
(12, 43, 'Хлеб «Бородинский» нарезка 350 г',
    'xleb-borodinskij-350',
    'Классический ржаной хлеб с тмином по традиционному рецепту. Подходит к первым блюдам и бутербродам.',
    49.90, NULL, 'шт', 0.350, 200, 'SKU-014', 'Владхлеб', 'Россия', FALSE, FALSE, 4.50, 98),

-- Замороженные
(16, NULL, 'Пельмени «Сибирские» 800 г',
    'pelmeni-sibirskie-800',
    'Пельмени ручной лепки с говядиной и свининой. Тонкое тесто, сочная начинка, фарш без добавок.',
    259.00, 310.00, 'шт', 0.800, 100, 'SKU-015', 'Сибирская коллекция', 'Россия', TRUE, TRUE, 4.25, 156),

-- Кетчупы
(8,  21, 'Томатная паста «Помидорка» 380 г',
    'tomatnaya-pasta-pomidorka-380',
    'Концентрированная томатная паста без крахмала. Натуральный состав. Содержание томатов не менее 25%.',
    55.00, NULL, 'шт', 0.380, 220, 'SKU-016', 'Помидорка', 'Россия', FALSE, FALSE, 4.10, 43),

-- Сыр дополнительный
(2,  8,  'Сыр «Пармезан» 40% 200 г',
    'syr-parmezan-200',
    'Итальянский выдержанный сыр с насыщенным ореховым вкусом. Идеален для пасты, ризотто и салатов.',
    320.00, NULL, 'шт', 0.200, 35, 'SKU-017', 'Grana Padano', 'Италия', FALSE, FALSE, 4.80, 72),

-- Йогурт
(2,  10, 'Йогурт «Активиа» натуральный 2,9% 260 г',
    'jogurt-aktivia-260',
    'Натуральный питьевой йогурт с живыми бифидобактериями. Без консервантов и красителей.',
    85.00, NULL, 'шт', 0.260, 160, 'SKU-018', 'Activia', 'Россия', FALSE, FALSE, 4.45, 88),

-- Масло
(2,  11, 'Масло сливочное «Простоквашино» 72,5% 180 г',
    'maslo-prostokvashino-180',
    'Классическое сладкосливочное масло из натуральных сливок. Без растительных жиров. ГОСТ 32261.',
    149.00, 175.00, 'шт', 0.180, 140, 'SKU-019', 'Простоквашино', 'Россия', TRUE, TRUE, 4.60, 201),

-- Майонез
(8,  22, 'Майонез «Провансаль» классический 400 г',
    'majonez-provansale-400',
    'Классический майонез на основе яичного желтка. Нежный вкус, густая консистенция.',
    99.00, NULL, 'шт', 0.400, 175, 'SKU-020', 'Слобода', 'Россия', FALSE, FALSE, 4.20, 134);

-- Фото товаров (заглушки с unsplash)
INSERT INTO product_images (product_id, url, alt, is_main, sort_order) VALUES
(1,  '/images/products/ketchup-kubanochka-main.jpg',    'Кетчуп Кубаночка',           TRUE,  1),
(2,  '/images/products/kornishony-kubanochka-main.jpg', 'Корнишоны Кубаночка',        TRUE,  1),
(3,  '/images/products/malina-main.jpg',                'Малина свежемороженая',       TRUE,  1),
(4,  '/images/products/moloko-primorskoe-main.jpg',     'Молоко Приморское',          TRUE,  1),
(5,  '/images/products/syr-rossijskij-main.jpg',        'Сыр Российский',             TRUE,  1),
(6,  '/images/products/yajco-main.jpg',                 'Яйцо куриное С1',            TRUE,  1),
(7,  '/images/products/kolbasa-doktorskaya-main.jpg',   'Колбаса Докторская',         TRUE,  1),
(8,  '/images/products/grudka-main.jpg',                'Грудка куриная',             TRUE,  1),
(9,  '/images/products/gorbusa-main.jpg',               'Горбуша стейк',              TRUE,  1),
(10, '/images/products/ikra-main.jpg',                  'Икра горбуши',               TRUE,  1),
(11, '/images/products/ris-main.jpg',                   'Рис длиннозёрный',           TRUE,  1),
(12, '/images/products/grechka-main.jpg',               'Гречка ядрица',              TRUE,  1),
(13, '/images/products/sok-yablochnyj-main.jpg',        'Сок яблочный',               TRUE,  1),
(14, '/images/products/xleb-borodinskij-main.jpg',      'Хлеб Бородинский',           TRUE,  1),
(15, '/images/products/pelmeni-main.jpg',               'Пельмени Сибирские',         TRUE,  1),
(16, '/images/products/tomatpasta-main.jpg',            'Томатная паста Помидорка',   TRUE,  1),
(17, '/images/products/parmezan-main.jpg',              'Сыр Пармезан',               TRUE,  1),
(18, '/images/products/jogurt-main.jpg',                'Йогурт Активиа',             TRUE,  1),
(19, '/images/products/maslo-main.jpg',                 'Масло Простоквашино',        TRUE,  1),
(20, '/images/products/majonez-main.jpg',               'Майонез Провансаль',         TRUE,  1);

-- Атрибуты товаров
INSERT INTO product_attributes (product_id, attr_name, attr_value) VALUES
(1,  'Масса нетто',   '310 г'),
(1,  'Упаковка',      'Дойпак'),
(1,  'Количество в ящике', '20 шт'),
(1,  'Состав',        'Томатная паста, сахар, уксус, соль, специи'),
(2,  'Масса нетто',   '680 г'),
(2,  'Размер',        '1–3 см'),
(2,  'Количество в ящике', '8 шт'),
(3,  'Вид заморозки', 'Шоковая'),
(3,  'Фасовка',       '500 г / 1 кг'),
(5,  'Жирность',      '45%'),
(5,  'Тип',           'Полутвёрдый'),
(6,  'Категория',     'С1'),
(6,  'Количество',    '10 шт'),
(9,  'Вид нарезки',   'Стейк'),
(9,  'Вид обработки', 'Замороженная'),
(17, 'Выдержка',      '12 месяцев'),
(17, 'Жирность',      '40%'),
(19, 'Жирность',      '72,5%'),
(19, 'Вид',           'Сладкосливочное'),
(10, 'Посол',         'Натуральный'),
(10, 'Сорт',          'Высший (Экстра)');

-- ============================================================
--  5. КОРЗИНЫ И ЗАКАЗЫ
-- ============================================================
CREATE TABLE carts (
    id          SERIAL PRIMARY KEY,
    user_id     INT          REFERENCES users(id) ON DELETE CASCADE,
    session_id  VARCHAR(100),                   -- для незалогиненных
    branch_id   INT          REFERENCES branches(id),
    created_at  TIMESTAMPTZ  DEFAULT NOW(),
    updated_at  TIMESTAMPTZ  DEFAULT NOW(),
    CONSTRAINT cart_owner CHECK (user_id IS NOT NULL OR session_id IS NOT NULL)
);

CREATE TABLE cart_items (
    id          SERIAL PRIMARY KEY,
    cart_id     INT           NOT NULL REFERENCES carts(id) ON DELETE CASCADE,
    product_id  INT           NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    qty         INT           NOT NULL DEFAULT 1 CHECK (qty > 0),
    price_snap  NUMERIC(10,2) NOT NULL,   -- цена на момент добавления
    UNIQUE (cart_id, product_id)
);

-- Корзины
INSERT INTO carts (user_id, branch_id) VALUES
(3, 1),
(4, 1),
(5, 3),
(6, 4);

INSERT INTO cart_items (cart_id, product_id, qty, price_snap) VALUES
(1, 1,  3, 42.30),
(1, 2,  1, 89.90),
(1, 19, 2, 149.00),
(2, 3,  2, 250.00),
(2, 12, 1, 95.00),
(3, 7,  1, 185.00),
(3, 8,  2, 210.00),
(4, 10, 1, 590.00),
(4, 9,  1, 349.00);

CREATE TABLE orders (
    id              SERIAL PRIMARY KEY,
    user_id         INT           NOT NULL REFERENCES users(id),
    branch_id       INT           REFERENCES branches(id),
    status          VARCHAR(30)   NOT NULL DEFAULT 'new'
                    CHECK (status IN ('new','confirmed','processing','shipped','delivered','cancelled')),
    delivery_address TEXT,
    delivery_date   DATE,
    comment         TEXT,
    total_amount    NUMERIC(12,2) NOT NULL CHECK (total_amount >= 0),
    discount_amount NUMERIC(12,2) DEFAULT 0,
    created_at      TIMESTAMPTZ   DEFAULT NOW(),
    updated_at      TIMESTAMPTZ   DEFAULT NOW()
);

CREATE TABLE order_items (
    id          SERIAL PRIMARY KEY,
    order_id    INT           NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id  INT           NOT NULL REFERENCES products(id),
    product_name VARCHAR(255) NOT NULL,   -- snapshot
    qty         INT           NOT NULL CHECK (qty > 0),
    unit        VARCHAR(30)   NOT NULL,
    price       NUMERIC(10,2) NOT NULL,
    subtotal    NUMERIC(12,2) GENERATED ALWAYS AS (qty * price) STORED
);

-- Заказы
INSERT INTO orders (user_id, branch_id, status, delivery_address, delivery_date, total_amount, discount_amount) VALUES
(3, 1, 'delivered', 'г. Владивосток, ул. Пушкина, 10-25',  '2026-05-01', 1560.00, 0),
(3, 1, 'delivered', 'г. Владивосток, ул. Пушкина, 10-25',  '2026-05-12', 879.50,  50.00),
(4, 1, 'shipped',   'г. Владивосток, пр. Партизанский, 8-12', '2026-05-29', 2350.00, 0),
(5, 3, 'confirmed', 'г. Артём, ул. Кирова, 3-7',           '2026-06-01', 615.80,  0),
(6, 4, 'new',       'г. Находка, ул. Дзержинского, 15-2',  '2026-06-02', 1080.00, 100.00),
(7, 5, 'cancelled', 'г. Уссурийск, ул. Ленина, 44-9',      '2026-05-20', 490.00,  0),
(8, 1, 'processing','г. Владивосток, ул. Русская, 64-18',  '2026-05-31', 3200.00, 200.00);

INSERT INTO order_items (order_id, product_id, product_name, qty, unit, price) VALUES
-- Заказ 1
(1, 1,  'Кетчуп «Томатный "Кубаночка"» 310 г', 10, 'шт', 42.30),
(1, 2,  'Корнишоны «Кубаночка» 680 г',          5,  'шт', 89.90),
(1, 16, 'Томатная паста «Помидорка» 380 г',     8,  'шт', 55.00),
-- Заказ 2
(2, 3,  'Малина свежемороженая',                2,  'кг', 250.00),
(2, 12, 'Гречка ядрица 900 г',                  3,  'шт', 95.00),
(2, 14, 'Хлеб «Бородинский» 350 г',             2,  'шт', 49.90),
-- Заказ 3
(3, 10, 'Икра горбуши «Экстра» 130 г',          2,  'шт', 590.00),
(3, 9,  'Горбуша стейк замороженная 800 г',     3,  'шт', 349.00),
-- Заказ 4
(4, 6,  'Яйцо куриное С1 10 шт',               2,  'шт', 99.00),
(4, 4,  'Молоко «Приморское» 1 л',             6,  'шт', 69.90),
(4, 18, 'Йогурт «Активиа» 260 г',              3,  'шт', 85.00),
-- Заказ 5
(5, 8,  'Грудка куриная охлаждённая 1 кг',     3,  'кг', 210.00),
(5, 7,  'Колбаса «Докторская» 400 г',          2,  'шт', 185.00),
-- Заказ 6
(6, 11, 'Рис длиннозёрный 1 кг',               3,  'кг', 79.90),
(6, 15, 'Пельмени «Сибирские» 800 г',          2,  'шт', 259.00),
-- Заказ 7
(7, 5,  'Сыр «Российский» 45% 1 кг',          2,  'кг', 580.00),
(7, 17, 'Сыр «Пармезан» 200 г',               2,  'шт', 320.00),
(7, 19, 'Масло сливочное «Простоквашино» 180 г', 6, 'шт', 149.00),
(7, 20, 'Майонез «Провансаль» 400 г',          4,  'шт', 99.00);

-- ============================================================
--  6. НОВОСТИ
-- ============================================================
CREATE TABLE tags (
    id      SERIAL PRIMARY KEY,
    name    VARCHAR(80) UNIQUE NOT NULL,
    slug    VARCHAR(80) UNIQUE NOT NULL
);

CREATE TABLE news (
    id          SERIAL PRIMARY KEY,
    author_id   INT          REFERENCES users(id) ON DELETE SET NULL,
    title       VARCHAR(255) NOT NULL,
    slug        VARCHAR(255) UNIQUE NOT NULL,
    preview     TEXT,
    content     TEXT,
    cover_url   TEXT,
    is_published BOOLEAN     DEFAULT FALSE,
    published_at TIMESTAMPTZ,
    created_at  TIMESTAMPTZ  DEFAULT NOW(),
    updated_at  TIMESTAMPTZ  DEFAULT NOW()
);

CREATE TABLE news_tags (
    news_id INT NOT NULL REFERENCES news(id)  ON DELETE CASCADE,
    tag_id  INT NOT NULL REFERENCES tags(id)  ON DELETE CASCADE,
    PRIMARY KEY (news_id, tag_id)
);

INSERT INTO tags (name, slug) VALUES
('Новинки',    'novinki'),
('Акции',      'aktsii'),
('Рецепты',    'recepti'),
('Компания',   'kompaniya'),
('Рекорды',    'rekordy');

INSERT INTO news (author_id, title, slug, preview, content, cover_url, is_published, published_at) VALUES
(1,
 '«Самое ценное в нашем мире — овощи»',
 'samoe-tsennoe-ovoshhi',
 'Наш шеф-технолог рассказывает о правильном выборе и хранении свежих овощей для вашего стола.',
 '<p>Свежие овощи — основа здорового рациона. В этой статье наш шеф-технолог делится секретами выбора лучших продуктов и рассказывает о том, как мы работаем с фермерами напрямую, чтобы обеспечить максимальную свежесть.</p><p>Мы убеждены: качество начинается с поля, а не с полки магазина.</p>',
 '/images/news/ovoshi-cover.jpg',
 TRUE, '2021-12-15 10:00:00+10'),

(2,
 'Чувство вкуса',
 'chuvstvo-vkusa',
 'Как правильно сочетать продукты, чтобы раскрыть их лучшие вкусовые качества? Отвечают наши эксперты.',
 '<p>Гастрономия — это искусство. Наш нутрициолог рассказывает о принципах вкусовых сочетаний и о том, почему некоторые продукты "дружат" друг с другом, а некоторые — нет.</p>',
 '/images/news/vkus-cover.jpg',
 TRUE, '2021-12-15 12:00:00+10'),

(1,
 '"TDA" установила новый мировой рекорд Гиннесса',
 'tda-rekord-ginnessa',
 'Наша компания совместно с партнёрами установила рекорд по наибольшему количеству участников дегустации одновременно.',
 '<p>25 ноября 2021 года в рамках городского фестиваля еды "TDA" и её партнёры провели масштабную дегустацию, в которой приняли участие более 5 000 человек одновременно. Рекорд официально зарегистрирован Книгой рекордов Гиннесса.</p>',
 '/images/news/rekord-cover.jpg',
 TRUE, '2021-12-15 14:00:00+10'),

(2,
 'Новинка сезона: линейка замороженных ягод',
 'novinki-zamorozhennye-yagody',
 'Расширяем ассортимент! Теперь в наличии клубника, малина, черника и смородина методом шоковой заморозки.',
 '<p>Мы рады сообщить о запуске новой линейки замороженных ягод. Шоковая заморозка сохраняет до 95% витаминов и позволяет наслаждаться вкусом лета круглый год.</p>',
 '/images/news/yagody-new-cover.jpg',
 TRUE, '2026-04-10 09:00:00+10'),

(1,
 'Весенние акции на молочную продукцию',
 'vesennie-aktsii-molochka',
 'С 1 по 31 мая скидки до 30% на весь ассортимент молочной продукции. Не упустите!',
 '<p>В мае мы дарим нашим покупателям особые условия на молочную продукцию. Скидки распространяются на молоко, сыр, масло, творог и йогурты ведущих российских производителей.</p>',
 '/images/news/akcii-cover.jpg',
 TRUE, '2026-05-01 08:00:00+10');

INSERT INTO news_tags (news_id, tag_id) VALUES
(1, 3), (1, 4),
(2, 3),
(3, 4), (3, 5),
(4, 1),
(5, 2);

-- ============================================================
--  7. БАННЕРЫ (главная страница, «Популярные категории»)
-- ============================================================
CREATE TABLE banners (
    id          SERIAL PRIMARY KEY,
    title       VARCHAR(255),
    subtitle    TEXT,
    image_url   TEXT         NOT NULL,
    link_url    TEXT,
    position    VARCHAR(30)  DEFAULT 'main'   -- main | promo | sidebar
                CHECK (position IN ('main','promo','sidebar')),
    sort_order  INT          DEFAULT 0,
    is_active   BOOLEAN      DEFAULT TRUE,
    created_at  TIMESTAMPTZ  DEFAULT NOW()
);

INSERT INTO banners (title, subtitle, image_url, link_url, position, sort_order) VALUES
('Свежие ягоды к вашему столу',      'Малина, клубника, черника с доставкой', '/images/banners/banner-yagody.jpg',    '/catalog/yagody',      'main',  1),
('Молочная продукция от фермеров',   'Натуральные продукты без добавок',       '/images/banners/banner-moloko.jpg',    '/catalog/molochnoe',   'main',  2),
('Рыба и морепродукты Дальнего Востока', 'Прямо из Японского моря',            '/images/banners/banner-ryba.jpg',      '/catalog/ryba',        'main',  3),
('Акция: кетчупы -25%',              'Только до конца месяца',                 '/images/banners/promo-ketchup.jpg',    '/catalog/sousy',       'promo', 1),
('Пельмени «Сибирские» -15%',        'При покупке от 2 упаковок',              '/images/banners/promo-pelmeni.jpg',    '/catalog/zamorozhennye','promo',2);

-- Популярные категории (блок на главной)
CREATE TABLE popular_categories (
    id          SERIAL PRIMARY KEY,
    category_id INT NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    image_url   TEXT,
    sort_order  INT DEFAULT 0
);

INSERT INTO popular_categories (category_id, image_url, sort_order) VALUES
(2,  '/images/cats/molochnoe.jpg',  1),
(3,  '/images/cats/myaso.jpg',      2),
(1,  '/images/cats/ovoshi.jpg',     3),
(18, '/images/cats/yagody.jpg',     4);

-- ============================================================
--  8. СТРАНИЦЫ (О компании, Контакты, Текстовые страницы)
-- ============================================================
CREATE TABLE pages (
    id          SERIAL PRIMARY KEY,
    slug        VARCHAR(100) UNIQUE NOT NULL,
    title       VARCHAR(255) NOT NULL,
    content     TEXT,
    is_published BOOLEAN DEFAULT TRUE,
    updated_at  TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO pages (slug, title, content) VALUES
('o-kompanii', 'О компании',
 '<h2>15 лет на рынке</h2>
<p>Компания «TDA» работает на рынке продовольственных товаров с 2009 года. За это время мы выросли из небольшого оптового склада во Владивостоке в крупную региональную торговую сеть с пятью филиалами по Приморскому краю.</p>
<p>Мы работаем напрямую с производителями и фермерскими хозяйствами, что позволяет нам предлагать продукцию по оптовым ценам при розничном качестве обслуживания.</p>
<h2>Наши принципы</h2>
<ul>
  <li>Качество — превыше всего</li>
  <li>Прозрачные цены без скрытых наценок</li>
  <li>Доставка по городу и краю</li>
  <li>Работа с фермерами напрямую</li>
</ul>'),

('dostavka', 'Доставка',
 '<h2>Доставка по городу и краю</h2>
<p>Мы осуществляем доставку товаров во Владивостоке, Артёме, Находке и Уссурийске. Минимальная сумма заказа для бесплатной доставки — 2 000 ₽.</p>
<p>Срок доставки: 1–2 рабочих дня. Самовывоз из любого нашего филиала — бесплатно.</p>'),

('kontakty', 'Контакты',
 '<h2>Свяжитесь с нами</h2>
<p>Телефон: +7 (123) 456-78-90<br>Email: info@kubanochka.ru<br>Режим работы: Пн–Пт 8:00–18:00</p>
<p>Головной офис: г. Владивосток, ул. Светланская, 45</p>'),

('politika-konfidentsialnosti', 'Политика конфиденциальности',
 '<p>Настоящая политика конфиденциальности описывает, как мы собираем, используем и защищаем ваши персональные данные при использовании нашего сайта.</p>');

-- ============================================================
--  9. ИНДЕКСЫ
-- ============================================================
CREATE INDEX idx_products_category      ON products(category_id);
CREATE INDEX idx_products_subcategory   ON products(subcategory_id);
CREATE INDEX idx_products_is_promo      ON products(is_promo) WHERE is_promo = TRUE;
CREATE INDEX idx_products_is_active     ON products(is_active) WHERE is_active = TRUE;
CREATE INDEX idx_products_slug          ON products(slug);
CREATE INDEX idx_orders_user            ON orders(user_id);
CREATE INDEX idx_orders_status          ON orders(status);
CREATE INDEX idx_order_items_order      ON order_items(order_id);
CREATE INDEX idx_cart_items_cart        ON cart_items(cart_id);
CREATE INDEX idx_news_published         ON news(published_at DESC) WHERE is_published = TRUE;
CREATE INDEX idx_subcategories_category ON subcategories(category_id);

-- ============================================================
--  10. ПОЛЕЗНЫЕ ПРЕДСТАВЛЕНИЯ (VIEWS)
-- ============================================================

-- Акционные товары
CREATE OR REPLACE VIEW v_promo_products AS
SELECT
    p.id, p.name, p.slug, p.price, p.price_old,
    ROUND((1 - p.price / p.price_old) * 100) AS discount_pct,
    p.unit, p.stock_qty, p.rating, p.reviews_count,
    pi.url AS image_url,
    c.name  AS category_name
FROM products p
LEFT JOIN product_images pi ON pi.product_id = p.id AND pi.is_main = TRUE
LEFT JOIN categories c      ON c.id = p.category_id
WHERE p.is_promo = TRUE AND p.is_active = TRUE AND p.price_old IS NOT NULL;

-- Содержимое корзины с подробностями
CREATE OR REPLACE VIEW v_cart_details AS
SELECT
    ci.cart_id,
    u.email         AS user_email,
    CONCAT(u.first_name, ' ', u.last_name) AS user_name,
    p.id            AS product_id,
    p.name          AS product_name,
    p.unit,
    ci.qty,
    ci.price_snap,
    (ci.qty * ci.price_snap) AS line_total
FROM cart_items ci
JOIN carts     c ON c.id = ci.cart_id
JOIN products  p ON p.id = ci.product_id
LEFT JOIN users u ON u.id = c.user_id;

-- Статистика заказов по пользователям
CREATE OR REPLACE VIEW v_user_order_stats AS
SELECT
    u.id,
    CONCAT(u.first_name, ' ', u.last_name) AS full_name,
    u.email,
    COUNT(o.id)          AS orders_total,
    SUM(o.total_amount)  AS total_spent,
    MAX(o.created_at)    AS last_order_at
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.role = 'customer'
GROUP BY u.id;

-- ============================================================
--  ГОТОВО
-- ============================================================
-- Таблицы: branches, users, categories, subcategories,
--          products, product_images, product_attributes,
--          carts, cart_items, orders, order_items,
--          tags, news, news_tags,
--          banners, popular_categories, pages
-- Представления: v_promo_products, v_cart_details, v_user_order_stats
-- Записей: ~200+
-- ============================================================
