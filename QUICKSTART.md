# QUICKSTART — Полная установка с нуля

Пошаговая инструкция настройки AI Ads Agent. Если ты никогда не открывал терминал — иди по фазам последовательно, не пропускай.

**Время:** 3–5 часов (большая часть — ожидание публикации Meta App, 1–3 дня).

---

## Фаза 0 — Подготовка компьютера (Mac)

### 0.1 Проверь macOS 12 или новее
🍎 → «Об этом Mac» → версия системы.

### 0.2 Установи Homebrew
Открой Terminal (Spotlight → «Terminal»). Вставь команду с https://brew.sh и нажми Enter. Подожди 3–5 минут.

### 0.3 Установи Node.js
```bash
brew install node
node --version    # должен быть v18 или выше
```

### 0.4 Установи Python и uv
```bash
brew install python
pip install uv
uv --version
```

### 0.5 Установи Git
```bash
brew install git
git --version
```

### 0.6 (Опционально) VS Code
Скачай с https://code.visualstudio.com — пригодится для редактирования брифов.

✅ Чек-лист: `node`, `python`, `uv`, `git` — все команды отвечают версией.

---

## Фаза 1 — Claude Code

### 1.1 Регистрация
Зайди на https://console.anthropic.com и создай аккаунт.

### 1.2 Пополни баланс
Минимум **$5**. Средняя сессия с агентом стоит $0.10–0.50.

### 1.3 Создай API-ключ
console.anthropic.com → API Keys → Create Key. Формат: `sk-ant-api03-...`

⚠️ **Сохрани ключ сразу** — он показывается только один раз.

### 1.4 Установи Claude Code
```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

### 1.5 Авторизуйся
```bash
claude
```
Введи API-ключ когда попросит.

✅ Чек-лист: `claude --version` отвечает, авторизация прошла.

---

## Фаза 2 — Файлы агента

### 2.1 Клонируй репозиторий
```bash
cd ~/Desktop
git clone https://github.com/skipaoff/ai-target-agent.git
cd ai-target-agent
```

### 2.2 Проверь структуру
```bash
ls
```
Должны быть: `CLAUDE.md`, `config/`, `skills/`, `docs/`, `examples/`, `README.md`, `QUICKSTART.md`.

✅ Чек-лист: папка скачана, все файлы на месте.

---

## Фаза 3 — Meta App + System User Token (самая сложная)

### 3.1 Зайди на developers.facebook.com
Войди под своим Facebook-аккаунтом.

### 3.2 Создай App
My Apps → **Create App**
- Тип: **Business** (не Consumer!)
- Название: например `My Ads Agent`
- Привяжи Business Manager

### 3.3 Добавь Marketing API
Add Product → **Marketing API** → Set Up.

### 3.4 Запроси разрешения
App Review → Permissions → добавь:
- `ads_management`
- `ads_read`
- `pages_manage_ads`

### 3.5 ⚠️ ОПУБЛИКУЙ APP — критически важно
App Settings → Basic → переключи **Status: Live**.

Без этого нельзя создавать Ad Creatives через API. Публикация занимает 1–3 дня.

### 3.6 System User Token
business.facebook.com → Business Settings → System Users → **Add**:
- Имя: `AI Agent`
- Роль: **Admin**

Затем **Generate New Token**:
- Выбери созданный App
- Разрешения: `ads_management`, `ads_read`, `pages_manage_ads`

⚠️ **Скопируй токен сразу** — он не показывается повторно.
✅ System User Token **не протухает** (в отличие от User Access Token, который умирает через 60 дней).

### 3.7 Найди Ad Account ID
Business Manager → Accounts → Ad Accounts. Формат: `act_XXXXXXXXXX`.

✅ Чек-лист: App создан и Live, System User Token получен, Ad Account ID известен.

---

## Фаза 4 — MCP сервер

### 4.1 Скачай MCP сервер
```bash
cd ~
git clone https://github.com/pipeboard-co/meta-ads-mcp.git
cd meta-ads-mcp
```

### 4.2 Установи зависимости
```bash
uv sync
```

### 4.3 Проверь
```bash
uv run meta-ads-mcp --help
```

### 4.4 Узнай полный путь
```bash
pwd
```
Скопируй результат — это абсолютный путь к папке meta-ads-mcp.

### 4.5 Создай `.mcp.json` в папке агента
```bash
cd ~/Desktop/ai-ads-agent
cp .mcp.json.example .mcp.json
```

Открой `.mcp.json` в редакторе и заполни:
- `META_APP_ID` → App ID из Facebook Developer (Basic Settings)
- `META_APP_SECRET` → App Secret оттуда же
- `META_ACCESS_TOKEN` → System User Token из шага 3.6
- Путь `/ABSOLUTE/PATH/TO/meta-ads-mcp` → результат `pwd` из шага 4.4

✅ Чек-лист: MCP сервер установлен, `.mcp.json` заполнен реальными данными.

---

## Фаза 5 — Первый запуск

### 5.1 Запусти Claude Code в папке агента
```bash
cd ~/Desktop/ai-ads-agent
claude
```

### 5.2 Проверь подключение MCP
Внутри Claude напиши:
```
/mcp
```
Должно показать: `meta-ads: connected ✓`

### 5.3 Первый тест
Напиши агенту:
```
Покажи список моих рекламных аккаунтов
```

Если видишь свой кабинет — всё работает. 🎉

### Типичные ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| MCP disconnected | Неправильный путь к meta-ads-mcp | Проверь `pwd`, вставь точный путь в `.mcp.json` |
| Token invalid | Неправильный токен | Перегенерируй System User Token |
| Permission denied | App в Development | Опубликуй App (фаза 3.5) |
| Command not found: uv | uv не установлен | `pip install uv` |
| Command not found: claude | Claude Code не в PATH | Перезапусти терминал или `npm install -g @anthropic-ai/claude-code` |

✅ Чек-лист: MCP connected, аккаунты видны.

---

## Фаза 6 — Настрой под себя

### 6.1 Добавь свой аккаунт в `config/ad_accounts.md`
Открой файл, заполни блок-шаблон своими данными (Account ID, Page ID, валюта, часовой пояс).

### 6.2 Создай бриф
```bash
cp config/briefs/_template.md config/briefs/myproject.md
```
Открой `myproject.md` и заполни: ниша, продукт, аудитория, целевой CPL, правила работы.

### 6.3 Проверь
```
Прочитай бриф моего проекта и скажи, что ты знаешь
```

✅ Чек-лист: Аккаунт в `ad_accounts.md`, бриф заполнен.

---

## Фаза 7 — Первая реальная работа

### 7.1 Анализ кабинета
```
/ads-optimizer myproject
```

### 7.2 Первый отчёт
```
/ads-reporter myproject
```
HTML-файл появится в папке `reports/`.

### 7.3 Первая оптимизация
Агент предложит действия — подтверди «Да» или отклони.

⚠️ **Агент никогда не выполняет изменения в кабинете без подтверждения.**

### 7.4 Первый запуск кампании
```
/campaign-manager
```
Агент задаст вопросы → построит структуру → дождётся подтверждения.

---

## Поддержка

Telegram: [@dreamm_assistant](https://t.me/dreamm_assistant)
