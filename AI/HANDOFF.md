# 4E Tools --- HANDOFF v2.4

## Phase

Specification v2.4 synchronized and ready for `create-expo-app`.
Application implementation has not started.

## Закрыто в v2.4

-   D-038: `create_tool_model` и `create_tool` получили реальную
    идемпотентность через `movements` + `CREATE_MODEL` / `CREATE_TOOL`.
-   Все mutating RPC используют единый
    `UNIQUE(actor_user_id, request_id)` контракт.
-   D-039: полный tool lifecycle/history доступен только admin.
-   Employee видит собственные movements.
-   Brigadier видит movements текущей бригады согласно D-034.
-   Нумерация конкретизирована через
    `pg_advisory_xact_lock(hashtext(scope))`.
-   Scope для модели: `tool_models.inventory_group`.
-   Scope для экземпляров: `tool:<model_id>.slot`.
-   Явно запрещено полагаться на `SELECT MAX(...) FOR UPDATE` для защиты
    от параллельных INSERT.
-   TASKS и UX синхронизированы до v2.4.
-   README добавлен как обязательная foundation-задача.

## Следующая задача

Создать foundation: 1. актуальный Expo starter + TypeScript; 2. Expo
Router; 3. strict TypeScript; 4. ESLint/Prettier; 5. scripts
`typecheck`, `lint`, `test`; 6. `.env.example` + `.gitignore`; 7.
`app/`, `src/`, `supabase/`, `assets/branding/`, `AI/`; 8. сохранить
этот пакет документации без изменения решений; 9. запустить проверки;
10. сделать первый commit; 11. обновить HANDOFF фактическими
результатами.

## Не делать на foundation этапе

-   не менять LOCKED UX;
-   не добавлять service-role в mobile client;
-   не добавлять persistent server-state cache;
-   не расширять RLS на полный tool history для employee/brigadier;
-   не менять семантику нумерации и идемпотентности.

## Проверки

Не запускались: application code ещё отсутствует.
