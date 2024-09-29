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
создать с заполнением полей - insert;
получить все - select;
удалить по id;

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
посмотреть все транзакции пользователя: выборка с джоином, но это скорее нужно будет из под таблицы с пользователем)
совершение операции: update - уменьшение кол-ва токенов у проекта, увеличение у пользователя

тут просто запрос на создание таблицы, нужно еще подумать

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
создание: create
просмотр всех: select 
retrieve: select ?не знаю, что это за запрос, в роутам на питоне он отображается
обновление: update по id - обязательные поля: amount, cost_per_token

наверное, сюда стоит добавить количество токенов, которые на разлоке, чтобы можно было менять через панель, для вывода средств

удаление по id
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
несколько update: на пароль, почту, личной информации
удаление: не знаю, понадобится ли

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