# 4E Tools --- TASKS v2.4

## 0 --- Foundation

-   [ ] Инициализировать Expo + TypeScript.
-   [ ] Настроить Expo Router.
-   [ ] Включить TypeScript strict.
-   [ ] Настроить ESLint/Prettier.
-   [ ] Добавить scripts: `typecheck`, `lint`, `test`.
-   [ ] Создать `app/`, `src/`, `supabase/`, `assets/branding/`, `AI/`.
-   [ ] Создать `src/strings/` для централизованных русских строк.
-   [ ] Добавить утверждённые брендовые ассеты в `assets/branding/`.
-   [ ] Создать `.env.example`.
-   [ ] В `.env.example` разрешены только `EXPO_PUBLIC_SUPABASE_URL` и
    `EXPO_PUBLIC_SUPABASE_ANON_KEY`.
-   [ ] Убедиться, что service-role key не используется в mobile client.
-   [ ] Настроить `.gitignore`.
-   [ ] Первый commit.

## 1 --- Design system

-   [ ] Design tokens.
-   [ ] Dark/Light.
-   [ ] Theme Context + AsyncStorage.
-   [ ] Button/Input/Card/Screen.
-   [ ] Loading/Error/Empty.
-   [ ] Bottom navigation.
-   [ ] Не добавлять persistent server-state cache (D-037).

## 2 --- Supabase

-   [ ] Migration: `CREATE EXTENSION IF NOT EXISTS citext`.
-   [ ] Auth + profiles.
-   [ ] brigades, warehouses, categories, tool_models, tools, movements.
-   [ ] UNIQUE/CHECK constraints.
-   [ ] Индексы.
-   [ ] RLS.
-   [ ] `create_tool_model` по D-036.
-   [ ] `create_tool` по D-032.
-   [ ] `take_tool`.
-   [ ] `return_tool`.
-   [ ] `send_to_repair`.
-   [ ] `return_from_repair`.
-   [ ] `write_off`.
-   [ ] `request_id` idempotency для всех mutating RPC.
-   [ ] RPC/RLS integration tests.
-   [ ] Seed/dev data strategy.
-   [ ] Отключить email confirmation для закрытого корпоративного auth.
-   [ ] Подтвердить отсутствие client self-signup.

## 3 --- Auth

-   [ ] Splash LOCKED: logo + session restore.
-   [ ] Login LOCKED: login/password; technical email скрыт.
-   [ ] Нормализация регистронезависимого login.
-   [ ] Неверный login/password → утверждённый error text из UX.md.
-   [ ] Inactive account → утверждённый error text.
-   [ ] No network → утверждённый error text.
-   [ ] Service unavailable → утверждённый error text.
-   [ ] «Забыли пароль?» → «Обратитесь к администратору».
-   [ ] Settings без авторизации: только «О приложении» + версия.
-   [ ] Profile + role.
-   [ ] Session restore.
-   [ ] Logout.

## 4 --- Locked home

-   [ ] Реализовать UX.md без редизайна.
-   [ ] Search.
-   [ ] My Tools.
-   [ ] Categories.
-   [ ] Loading/error/empty.
-   [ ] Dark/Light parity.

## 5 --- Catalog

-   [ ] Category → models.
-   [ ] Model → instances/warehouses.
-   [ ] Instance details.
-   [ ] Status display.

## 6 --- TAKE

-   [ ] Confirmation.
-   [ ] RPC + request_id.
-   [ ] Loading/double-tap protection.
-   [ ] Conflict refresh/error.
-   [ ] Offline blocked with clear message.

## 7 --- RETURN

-   [ ] My Tools.
-   [ ] Warehouse.
-   [ ] condition + damaged comment.
-   [ ] RPC + request_id.
-   [ ] Offline blocked.
-   [ ] Success/error states.

## 8 --- History / Brigade

-   [ ] Employee history.
-   [ ] Tool history.
-   [ ] Brigade current-membership view по D-034.
-   [ ] Admin history.

## 9 --- Admin

-   [ ] Mobile dashboard.
-   [ ] Create model через `create_tool_model` (D-036).
-   [ ] Create physical tool через `create_tool` (D-032).
-   [ ] Employees/brigades.
-   [ ] Warehouses.
-   [ ] Damage/repair RPC flows.
-   [ ] Write-off RPC flow.
-   [ ] serial_number / purchase_date / note.

## 10 --- Android QA

-   [ ] Unit/component tests.
-   [ ] TAKE concurrency.
-   [ ] create_tool_model concurrency.
-   [ ] create_tool concurrency.
-   [ ] RETURN authorization.
-   [ ] Repair/write-off authorization/idempotency.
-   [ ] RLS role tests.
-   [ ] Dark/Light.
-   [ ] Android device sizes.
-   [ ] Test build.
-   [ ] Real-device field test.

## Post-MVP

-   [ ] Persistent TanStack Query cache.
-   [ ] Richer offline read cache.
-   [ ] QR.
-   [ ] LOST workflow.
-   [ ] Due dates / «Просроченные».
-   [ ] iOS.
