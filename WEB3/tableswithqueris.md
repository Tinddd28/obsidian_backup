## Networks
id: bigint, primary key, serial, not null
name: varchar(255), not null
code, varchar(255)
wallets - в питоне это 
```
wallets: Mapped[list["Wallets"]] = relationship(

"Wallets", back_populates="network_standard"
```
видимо, в таблице wallets будет foreign key на эту таблицу

тут еще можно добавить колонку с количеством токенов или чего-то такого и как-то подумать над операциями  сети (не точно)

### Запросы для нее:
Создать с заполнением полей - insert;
CREATE TABLE networks (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  -- Идентификатор
    name VARCHAR(255) NOT NULL,                          -- Название сети
    code VARCHAR(255)                                    -- Код сети 
);

ALTER TABLE wallets
ADD COLUMN network_id BIGINT,
ADD CONSTRAINT fk_network
FOREIGN KEY (network_id) REFERENCES networks(id)
ON DELETE CASCADE
ON UPDATE CASCADE;

INSERT INTO networks (name, code)
VALUES ('network_name', 'network_code');

получить все - select;
SELECT * FROM networks;

удалить по id;
DELETE FROM networks
WHERE id = id_value;



## Operations (лучше будет заменить на transactions)
id: bigint, primary key, serial, not null
ticket_project: bigint (? хз для чего) - сюда индекс
user_id: bigint, foreign key(users.id), on delete cascade, on update cascade
project_id: bigint,  foreign key(project.id), on delete cascade, on update cascade
quantity: integer
total_amount: float
tx_hash: varchar(255) unique - это хэш транзакции - сюда index
transaction_date: datetime - тут можно сделать default time.now() как-то так

### Запросы: 
тут просто запрос на создание таблицы, нужно еще подумать
CREATE TABLE transactions (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  -- Идентификатор транзакции
    ticket_project BIGINT,                               -- Индекс проекта
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE ON UPDATE CASCADE,  -- Внешний ключ на пользователя
    project_id BIGINT REFERENCES projects(id) ON DELETE CASCADE ON UPDATE CASCADE,  -- Внешний ключ на проект
    quantity INTEGER,                                    -- Количество токенов в транзакции
    total_amount FLOAT,                                  -- Сумма транзакции
    tx_hash VARCHAR(255) UNIQUE,                         -- Хэш транзакции
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP  -- Дата и время транзакции
);

Добавление индекса на колонку ticket_project
CREATE INDEX idx_ticket_project ON transactions(ticket_project);

посмотреть все транзакции пользователя: выборка с джоином, но это скорее нужно будет из под таблицы с пользователем)
...

совершение операции: update - уменьшение кол-ва токенов у проекта, увеличение у пользователя

Уменьшение количества токенов у проекта
UPDATE projects
SET token_balance = token_balance - quantity_value
WHERE id = project_id_value;

Увеличение количества токенов у пользователя
UPDATE users
SET token_balance = token_balance + quantity_value
WHERE id = user_id_value;

Вставка новой транзакции в таблицу transactions
INSERT INTO transactions (ticket_project, user_id, project_id, quantity, total_amount, tx_hash)
VALUES (ticket_project_value, user_id_value, project_id_value, quantity_value, total_amount_value, 'transaction_hash');




## Projects
id: bigint, primary key, serial, not null
title: varchar(255), unique
desctiption: varchar(255)
image: varchar(255), unique
amount: int - наверное, тоже лучше сделать float, чтобы не было кривизны
cost_per_token: float - сюда index
currency: varchar(3) - подумать, чтобы можно было проектам использовать разную валюту
(как-то еще нужно получать текущий курс каждой валюты - решение: ?парсер? (либо готовый софт для этого, может быть есть))

### Запросы

retrieve: select ?не знаю, что это за запрос, в роутам на питоне он отображается
...

наверное, сюда стоит добавить количество токенов, которые на разлоке, чтобы можно было менять через панель, для вывода средств

создание: create
CREATE TABLE projects (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,   -- Идентификатор проекта
    title VARCHAR(255) UNIQUE,                            -- Название проекта
    description VARCHAR(255),                             -- Описание проекта
    image VARCHAR(255) UNIQUE,                            -- Ссылка на изображение проекта
    amount FLOA    "updated_at" timestamptz not null default(now())T,                                         -- Сумма, привлекаемая проектом
    cost_per_token FLOAT,                                 -- Стоимость одного токена, индексируемая
    currency VARCHAR(3),                                  -- Валюта, используемая проектом (что то типа USD, EUR, BTC)
    token_balance FLOAT                                   -- Количество токенов
);

-- Добавление индекса на колонку cost_per_token
CREATE INDEX idx_cost_per_token ON projects(cost_per_token);


INSERT INTO projects (title, description, image, amount, cost_per_token, currency, token_balance)
VALUES ('Project Title', 'Project Description', 'http://example.com/image.png', 1000000.0, 10.0, 'USD', 100000);

просмотр всех: select 
SELECT * FROM projects;

обновление: update по id - обязательные поля: amount, cost_per_token
SELECT * FROM projects WHERE id = project_id_value;
UPDATE projects
SET amount = new_amount_value,
    cost_per_token = new_cost_per_token_value,
    token_balance = new_token_balance_value
WHERE id = project_id_value;

удаление по id
DELETE FROM projects WHERE id = project_id_value;

## Users
id: bigint, primary key, serial, not null
first_name: varchar
last_name: varchar
email: varchar, unique сюда индекс
country: varchar
is_active: bool
is_superuser: bool
is_verifed: bool

wallets: связать с таблицей wallets, чтобы можно было смотреть все кошельки пользователя
operations: аналогично

### Запросы
создание: create - пароль сделать default NULL, в коде генерерируется

CREATE TABLE users (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  -- Идентификатор пользователя
    first_name VARCHAR(255),                              -- Имя пользователя
    last_name VARCHAR(255),                               -- Фамилия пользователя
    email VARCHAR(255) UNIQUE,                            -- Почта пользователя
    password VARCHAR(255) DEFAULT NULL,                   -- Пароль пользователя
    country VARCHAR(255),                                 -- Страна пользователя
    is_active BOOLEAN DEFAULT TRUE,                       -- Флаг активности пользователя
    is_superuser BOOLEAN DEFAULT FALSE,                   -- Флаг суперпользователя
    is_verified BOOLEAN DEFAULT FALSE                     -- Флаг подтверждения пользователя
);

CREATE TABLE wallets (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  -- Идентификатор кошелька
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,  -- Внешний ключ к таблице `users`
    wallet_address VARCHAR(255) NOT NULL,                -- Адрес кошелька
    balance NUMERIC DEFAULT 0                            -- Баланс кошелька
);

CREATE TABLE operations (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  -- Идентификатор операции
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,  -- Внешний ключ к таблице `users`
    operation_type VARCHAR(255),                         -- Тип операции
    amount NUMERIC,                                      -- Сумма операции
    operation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP   -- Дата и время операции
);


несколько update: 
на пароль, 
UPDATE users
SET password = 'new_password'  -- Новый пароль пользователя
WHERE id = 1;                  -- Уникальный идентификатор пользователя

на почту,
UPDATE users
SET email = 'new_email@example.com'  -- Новая почта пользователя
WHERE id = 1;                        -- Уникальный идентификатор пользователя

для личной информации
UPDATE users
SET first_name = 'NewFirstName',     -- Новое имя пользователя
    last_name = 'NewLastName',       -- Новая фамилия пользователя
    country = 'New Country'          -- Новая страна пользователя
WHERE id = 1;                        -- Уникальный идентификатор пользователя

удаление: 
DELETE FROM users
WHERE id = 1;  -- Уникальный идентификатор пользователя



## Wallets
id: bigint, primary key, serial, not null
address: varchar, unique - сюда индекс
project_id: bigint, foreign key(projects.id), on delete cascade, on update cascade
user_id: bigint, foreign key(users.id), on delete cascade, on update cascade
network_id: bigint, bigint, foreign key(networks.id), on delete cascade, on update cascade

### Запросы 
создание для пользователя
создание для проекта
получение всех

CREATE TABLE wallets (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,   -- Уникальный идентификатор кошелька
    address VARCHAR(255) UNIQUE,                          -- Адрес кошелька (уникальный)
    project_id BIGINT REFERENCES projects(id)             -- Внешний ключ к таблице `projects`
        ON DELETE CASCADE 
        ON UPDATE CASCADE,
    user_id BIGINT REFERENCES users(id)                   -- Внешний ключ к таблице `users`
        ON DELETE CASCADE 
        ON UPDATE CASCADE,
    network_id BIGINT REFERENCES networks(id)             -- Внешний ключ к таблице `networks`
        ON DELETE CASCADE 
        ON UPDATE CASCADE
);

Создание кошелька для пользователя:
INSERT INTO wallets (address, project_id, user_id, network_id)
VALUES ('user_wallet_address', project_id_value, user_id_value, network_id_value);

Создание кошелька для проекта:
INSERT INTO wallets (address, project_id, user_id, network_id)
VALUES ('project_wallet_address', project_id_value, user_id_value, network_id_value);

Получение всех кошельков:
SELECT * FROM wallets;
