# Система учёта — Конструкторское бюро

⚠️ **Внимание: NDA (Соглашение о неразглашении)**

Данный репозиторий является портфолио-визиткой. Исходный код проекта является закрытым в связи с коммерческой тайной работодателя. В этом описании представлена архитектура проекта, используемый стек технологий, а также скриншоты пользовательского интерфейса (с использованием тестовых и обезличенных данных) исключительно для демонстрации моих компетенций как инженера-разработчика.

---

Корпоративное веб-приложение для учёта производственных дефектов, создания актов диагностики/утилизации/ОТК, отслеживания эффективности сотрудников и внутренних задач.

**Версия:** 1.2  
**Платформа:** Streamlit (Python)  
**База данных:** SQLite  
**Назначение:** учёт дефектов, генерация актов, аналитика эффективности, внутренние задачи и почта.

---

## Технологии

| Компонент | Библиотека |
|-----------|-----------|
| UI | Streamlit 1.35+ |
| База данных | sqlite3 |
| Работа с данными | pandas, openpyxl |
| DOCX | python-docx, docxtpl |
| Telegram API | requests |
| Аутентификация | bcrypt, hashlib |
| Окружение | python-dotenv |
| Логирование | logging (RotatingFileHandler) |
| Десктоп | pywebview, pywin32 |
| Сборка | PyInstaller |
| Установка | Inno Setup |

---

## Быстрый старт

```bash
pip install -r requirements.txt
streamlit run app.py
```

---

## Структура проекта

```text
app.py                      # Точка входа
config.py                   # Конфигурация (пути, .env)
database.py                 # Инициализация БД, утилиты (safe_execute, hash/verify)
auth.py                     # Аутентификация (cookies, bcrypt, rate-limit)
theme.py                    # Тёмная тема "Night Witch"
ui_helpers.py               # Общие UI-компоненты (кэшированные запросы, фильтры)
docx_utils.py               # Генерация DOCX из шаблонов
telegram_utils.py           # Отправка в Telegram
taskmesh_client.py          # HTTP-клиент TaskMesh (задачи/чаты)
notifications.py            # Push-уведомления в браузере
archive_parser.py           # Парсинг существующих DOCX-архивов
logging_config.py           # Централизованное логирование
launcher.py                 # Десктоп-загрузчик

tabs/                        # Вкладки интерфейса
  ├── tab_acts.py     📝 Создание актов
  ├── tab_inspect.py  📋 Акт осмотра ОТК
  ├── tab_mail.py     📌 Задачи и почта
  ├── tab_stat.py     📊 Статистика
  ├── tab_arch.py     📂 Архив
  ├── tab_imp.py      📥 Импорт актов
  ├── tab_eff.py      ⚡ Эффективность
  ├── tab_dash.py     🏠 Дашборд руководителя
  └── tab_admin.py    🔐 Управление доступом

АКТ_ОСМОТРА.docx            # Шаблон акта осмотра
АКТ_ОТК.docx                # Шаблон акта ОТК
АКТ_УТИЛИЗАЦИИ.docx         # Шаблон акта утилизации

Archive_Acts/               # Сгенерированные акты
Эффективность/              # Excel-шаблоны эффективности
DB_Archive_*/               # Архивы БД
logs/                       # Логи приложения

config.json                 # Telegram-конфиг (легаси)
.env                        # Переменные окружения
requirements.txt            # Зависимости
launcher_config.json        # Конфиг загрузчика

logo.png / icon.png / logo.ico
OTK_Groznye_Pticy.exe       # Собранный exe
installer.iss               # Inno Setup скрипт
*.bat                       # Скрипты сборки/установки
```

---

## Схема работы приложения (Mermaid)

```mermaid
flowchart TB
    subgraph User["👤 Пользователь"]
        Browser["Браузер (Streamlit UI)"]
    end

    subgraph App["📦 Приложение (Python)"]
        direction TB
        Entry["app.py<br/>Точка входа"]
        
        subgraph Auth["🔐 Аутентификация"]
            Login["auth.py<br/>Login / Register"]
            Cookies["Cookie-based сессия"]
            RateLimit["Rate Limiting<br/>(5 попыток)"]
        end

        subgraph Tabs["📑 Вкладки интерфейса"]
            Acts["tab_acts.py<br/>Создание актов"]
            Inspect["tab_inspect.py<br/>Акт осмотра ОТК"]
            Mail["tab_mail.py<br/>Задачи и почта"]
            Stat["tab_stat.py<br/>Статистика"]
            Arch["tab_arch.py<br/>Архив"]
            Imp["tab_imp.py<br/>Импорт"]
            Eff["tab_eff.py<br/>Эффективность"]
            Dash["tab_dash.py<br/>Дашборд"]
            Admin["tab_admin.py<br/>Управление"]
        end

        subgraph Utils["🔧 Утилиты"]
            UI["ui_helpers.py<br/>Общие компоненты"]
            Docx["docx_utils.py<br/>Генерация DOCX"]
            Telegram["telegram_utils.py<br/>Отправка в Telegram"]
            TaskMesh["taskmesh_client.py<br/>TaskMesh API"]
            Notif["notifications.py<br/>Push-уведомления"]
            Parser["archive_parser.py<br/>Парсинг DOCX"]
        end
    end

    subgraph Storage["💾 Хранилище"]
        DB[("factory_defects.db<br/>SQLite<br/>14 таблиц")]
        DocxFiles["📁 Archive_Acts/<br/>*.docx"]
        Templates["📄 Шаблоны DOCX"]
        Logs["📝 logs/app.log"]
    end

    subgraph External["🌐 Внешние системы"]
        TG[("Telegram Bot API")]
        TM[("TaskMesh Server<br/>localhost:8080")]
    end

    subgraph Config["⚙️ Конфигурация"]
        Env[".env"]
        TGConfig["config.json"]
    end

    %% Связи
    Browser -->|HTTP| Entry
    Entry -->|setup_cookies| Login
    Login --> Cookies
    Cookies --> RateLimit
    RateLimit -->|st.session_state| Tabs

    Entry -->|init_db| DB
    Entry -->|inject_theme| Theme
    Entry -->|render_notifications| Notif

    Tabs -->|safe_execute| DB
    Tabs -->|generate_diag_act| Docx
    Tabs -->|send_to_telegram| Telegram
    Tabs -->|taskmesh_*| TaskMesh
    Tabs -->|get_unread_count| UI

    Docx -->|читает| Templates
    Docx -->|сохраняет| DocxFiles

    Telegram -->|отправляет документы| TG
    TaskMesh -->|REST API| TM

    Parser -->|читает| DocxFiles
    Parser -->|записывает| DB

    Notif -->|WebSocket Poll| DB
    Notif -->|Browser Notification| Browser

    UI -->|кэшированные запросы| DB

    Config -->|dotenv| Entry
    Config -->|load_tg_config| Telegram

    Logs -.->|записывают| LogFile
```

## Данные

### База данных (SQLite)

14 таблиц: `departments`, `users`, `employees`, `defect_logs`, `defect_items`, `util_logs`, `util_items`, `inspection_logs`, `operations`, `work_log`, `messages`, `room_keys`, `tasks`, `task_comments`.

Связи:
- `defect_logs` → `defect_items` (один ко многим)
- `defect_logs` → `inspection_logs` (связь по `parent_log_id`)
- `util_logs` → `util_items` (один ко многим)
- Все логи привязаны к `departments` и `employees`

---

## Пользователи и роли

| Роль | Доступ |
|------|--------|
| `superadmin` | Полный доступ, включая управление пользователями |
| `manager` | Всё, кроме админ-панели |
| `employee` | Всё, кроме админ-панели и ОТК-проверки |
| `otk` / `otk_vedma` | Полный доступ + вкладка актов осмотра ОТК |

---

## Функциональность

### Акты (3 типа)

| Акт | Шаблон | Описание |
|-----|--------|----------|
| **Осмотра (диагностика)** | `АКТ_ОСМОТРА.docx` | Фиксация дефекта: артикул, наименование, ответственный, решение |
| **ОТК** | `АКТ_ОТК.docx` | Проверка ОТК: вердикт (проверено/отремонтировано/брак/утиль/готово) |
| **Утилизации** | `АКТ_УТИЛИЗАЦИИ.docx` | Списание: причина, заключение |

### Внутренние модули
- **Задачи** — доска задач с приоритетами, статусами, дедлайнами, комментариями
- **Почта** — внутренние сообщения между отделами, привязка к актам
- **Статистика** — аналитика дефектов, утилизации, графики, генерация отчётов DOCX
- **Эффективность** — учёт операций, нормо-часы, расчёт KPI, отчёты в Telegram
- **Дашборд руководителя** — сотрудники, сводка по отделу, ежедневный отчёт, учёт ключей
- **Импорт** — ручной ввод актов, пакетный импорт CSV/Excel/из архива DOCX
- **Архив** — просмотр/скачивание/отправка сгенерированных актов

---

## Пользовательский интерфейс
<img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/9639b838-b944-4e20-bf80-11919f9d2f44" />
<img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/300682c6-93ab-43f3-b4c9-654716254c84" />
