# 4E Tools --- DATABASE v2.3

## Extensions

Первая Supabase migration должна включать:

``` sql
CREATE EXTENSION IF NOT EXISTS citext;
```

`citext` нужен для регистронезависимого `profiles.login`.

## Auth

Supabase `auth.users` отвечает за идентификацию и пароль.
`public.profiles.id` --- 1:1 FK к `auth.users.id`.

UI использует `login + password`. Auth-слой нормализует login и
преобразует его в технический email `<login>@4e.local`. Email
confirmation отключён. Self-signup в мобильном клиенте отсутствует.
Создание auth-аккаунта выполняется только доверенным административным
серверным каналом. Service-role key никогда не находится в мобильном
приложении.

## Таблицы

### brigades

-   id uuid PK
-   name citext NOT NULL UNIQUE
-   active boolean NOT NULL default true
-   created_at timestamptz NOT NULL default now()

### profiles

-   id uuid PK FK -\> auth.users.id
-   login citext NOT NULL UNIQUE
-   full_name text NOT NULL
-   role enum `employee|brigadier|admin`
-   position text NULL
-   brigade_id uuid NULL FK -\> brigades.id
-   active boolean NOT NULL default true
-   created_at timestamptz NOT NULL default now()
-   updated_at timestamptz NOT NULL default now()

### warehouses

-   id uuid PK
-   name citext NOT NULL UNIQUE
-   active boolean NOT NULL default true
-   created_at timestamptz NOT NULL default now()

### categories

-   id uuid PK
-   name citext NOT NULL UNIQUE
-   sort_order int
-   active boolean NOT NULL default true

### tool_models

-   id uuid PK
-   category_id uuid NOT NULL FK -\> categories.id
-   name text NOT NULL
-   inventory_group integer NOT NULL UNIQUE
-   image_url text NULL
-   active boolean NOT NULL default true
-   CHECK (inventory_group \> 0)

`inventory_group` не присылается мобильным клиентом. Модель создаётся
через `create_tool_model` по D-036.

### tools

-   id uuid PK
-   model_id uuid NOT NULL FK -\> tool_models.id
-   slot integer NOT NULL
-   status enum `AVAILABLE|ASSIGNED|DAMAGED|REPAIR|WRITTEN_OFF`
-   warehouse_id uuid NULL FK -\> warehouses.id
-   assigned_user_id uuid NULL FK -\> profiles.id
-   condition_note text NULL
-   serial_number text NULL
-   purchase_date date NULL
-   note text NULL
-   created_at timestamptz NOT NULL default now()
-   updated_at timestamptz NOT NULL default now()
-   UNIQUE(model_id, slot)
-   CHECK (slot \> 0)

Отображаемый `inventory_number` вычисляется как
`<tool_models.inventory_group>.<tools.slot>`. Клиент не хранит и не
задаёт этот номер как источник истины.

### Инварианты tools

-   AVAILABLE: `warehouse_id IS NOT NULL`, `assigned_user_id IS NULL`.
-   ASSIGNED: `assigned_user_id IS NOT NULL`, `warehouse_id IS NULL`.
-   DAMAGED: не выдаётся.
-   REPAIR: не выдаётся; фиктивный склад «Сервис» не используется.
-   WRITTEN_OFF: `warehouse_id IS NULL`, `assigned_user_id IS NULL`, не
    выдаётся.
-   `slot > 0`.

CHECK constraints в миграциях должны отражать допустимые комбинации
статуса, склада и назначенного сотрудника.

### movements

-   id uuid PK
-   tool_id uuid NOT NULL FK -\> tools.id
-   actor_user_id uuid NOT NULL FK -\> profiles.id
-   assigned_user_id uuid NULL FK -\> profiles.id
-   action enum
    `TAKE|RETURN|MARK_DAMAGED|SEND_REPAIR|RETURN_REPAIR|WRITE_OFF`
-   from_warehouse_id uuid NULL FK -\> warehouses.id
-   to_warehouse_id uuid NULL FK -\> warehouses.id
-   condition text NULL
-   comment text NULL
-   request_id uuid NOT NULL
-   created_at timestamptz NOT NULL default now()
-   UNIQUE(actor_user_id, request_id)

`request_id` обязателен для всех mutating RPC, включая
create/repair/write-off.

### rpc_idempotency

-   id uuid PK
-   actor_user_id uuid NOT NULL FK -\> profiles.id
-   request_id uuid NOT NULL
-   operation text NOT NULL
-   result_ref jsonb NULL
-   created_at timestamptz NOT NULL default now()
-   UNIQUE(actor_user_id, request_id)

Назначение: единая идемпотентность всех mutating RPC. RPC сначала
проверяет эту таблицу; после успешной доменной операции сохраняет
operation/result_ref в той же транзакции. Повторный вызов с тем же
`(actor_user_id, request_id)` не повторяет mutation и возвращает
сохранённый результат.

## Индексы

-   tools(model_id,status)
-   tools(warehouse_id,status)
-   tools(assigned_user_id,status)
-   profiles(brigade_id)
-   movements(tool_id,created_at DESC)
-   movements(actor_user_id,created_at DESC)
-   movements(assigned_user_id,created_at DESC)

## Сериализация нумерации

Для атомарного присвоения последовательных номеров используется
транзакционная advisory-lock схема PostgreSQL:

``` sql
SELECT pg_advisory_xact_lock(hashtext('<scope>'));
```

Scopes: - для `tool_models.inventory_group`:
`tool_models.inventory_group`; - для `tools.slot` конкретной модели:
`tool:<model_id>.slot`.

После получения lock RPC вычисляет `MAX(...)+1` внутри той же
транзакции. Нельзя заменять это на `SELECT MAX(...) FOR UPDATE`: такой
запрос не защищает от параллельного INSERT.

## RPC create_tool_model

Только admin.

Вход: - category_id - name - image_url nullable - request_id

Транзакция: 1. проверить admin profile; 2. проверить idempotency по
request_id; 3. сериализовать присвоение следующего номера модели через
блокировку выделенного ресурса нумерации; 4. вычислить
`next inventory_group = MAX(inventory_group)+1`; 5. INSERT
`tool_models`; 6. COMMIT; 7. вернуть созданную модель.

Клиент не передаёт `inventory_group`.

## RPC create_tool

Только admin.

Вход: - model_id - initial_warehouse_id - nullable serial_number -
nullable purchase_date - nullable note - request_id

Транзакция: 1. проверить admin profile; 2. проверить idempotency; 3.
заблокировать строку выбранного `tool_models`; 4. определить
`next slot = MAX(slot)+1` для этой модели; 5. INSERT `tools` как
AVAILABLE; 6. COMMIT; 7. вернуть tool и display inventory number.

Клиент не задаёт slot/inventory_number.

## RPC take_tool

Вход: `tool_id`, `request_id`.

1.  active auth/profile;
2.  idempotency;
3.  `SELECT tool FOR UPDATE`;
4.  `status=AVAILABLE` и warehouse active;
5.  UPDATE → ASSIGNED, warehouse_id=NULL, assigned_user_id=auth.uid();
6.  INSERT movement TAKE с исходным складом;
7.  COMMIT.

Два параллельных TAKE одного экземпляра не могут оба завершиться
успешно.

## RPC return_tool

Вход: `tool_id`, `warehouse_id`, `condition`, `comment`, `request_id`.

1.  active user + active warehouse;
2.  idempotency;
3.  `SELECT tool FOR UPDATE`;
4.  employee возвращает только свой ASSIGNED tool;
5.  исправен → AVAILABLE + warehouse;
6.  повреждён → DAMAGED + warehouse;
7.  assigned_user_id=NULL;
8.  INSERT RETURN;
9.  COMMIT.

## Admin RPC

### send_to_repair(tool_id, comment, request_id)

Admin only. Row lock + idempotency. Допустимое исходное состояние
проверяется. → REPAIR. INSERT SEND_REPAIR.

### return_from_repair(tool_id, warehouse_id, comment, request_id)

Admin only. Row lock + idempotency. REPAIR → AVAILABLE на активном
складе. INSERT RETURN_REPAIR.

### write_off(tool_id, comment, request_id)

Admin only. Row lock + idempotency. → WRITTEN_OFF. `warehouse_id=NULL`,
`assigned_user_id=NULL`. INSERT WRITE_OFF.

Все mutating RPC атомарны и идемпотентны через `rpc_idempotency` по
D-038.

## RLS --- минимальный контракт

-   anonymous: никаких доменных данных.
-   authenticated + active profile: чтение активного каталога и
    необходимых складов.
-   employee:
    -   видит свои текущие назначения:
        `tools.assigned_user_id = auth.uid()`;
    -   видит movements, где `actor_user_id = auth.uid()` OR
        `assigned_user_id = auth.uid()`;
    -   видит каталог/доступность, необходимую для поиска инструмента.
-   brigadier:
    -   всё employee;
    -   текущие назначения и разрешённую историю пользователей, чьи
        `profiles.brigade_id` совпадают с brigade_id бригадира (D-034).
-   admin:
    -   административное чтение через проверяемую сервером роль.
-   прямые клиентские INSERT/UPDATE критического состояния `tools` и
    `tool_models` запрещены; mutations выполняются через RPC.
-   `movements` обычным клиентом не редактируются и не удаляются.
-   service-role key отсутствует в mobile bundle.

## Cache

По D-037 persistent TanStack Query cache не входит в MVP. Server-state
cache живёт только в памяти процесса.

Точные SQL policies, CHECK constraints, grants, функции и migrations
создаются в Supabase-этапе и покрываются integration tests.
