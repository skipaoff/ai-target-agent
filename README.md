# AI Ads Agent

AI-агент для управления Facebook/Instagram рекламой через Claude Code.

## Что это

Набор Claude Skills + конфигурация для автоматизации рекламы:

- Анализ и оптимизация кампаний с Health Score
- Создание кампаний под разные цели (WhatsApp, Lead Forms, Traffic)
- Отчёты по метрикам за любой период
- Анализ креативов с Risk Score
- Таргетинг — поиск аудиторий, Lookalike

## Что входит

### 12 Skills

| Skill | Команда | Что делает |
|---|---|---|
| Главный агент | `/ads-agent` | Точка входа, оркестрация |
| Оптимизатор | `/ads-optimizer` | Health Score, рекомендации по бюджетам |
| Отчёты | `/ads-reporter` | Метрики за today/3d/7d/30d |
| Дашборд | `/dashboard` | Статистика по всем аккаунтам |
| Кампании | `/campaign-manager` | Создание Campaign → AdSet → Ad |
| Таргетинг | `/targeting-expert` | Интересы, гео, Lookalike |
| Анализ креативов | `/creative-analyzer` | Risk Score, ad-eaters |
| Копирайтинг | `/creative-copywriter` | Тексты для рекламы |
| Генерация картинок | `/creative-image-generator` | Изображения через Gemini |
| Онбординг | `/account-onboarding` | Добавление нового аккаунта |
| Удаление | `/account-delete` | Удаление аккаунта из конфига |
| Нейминг | `/naming-rules` | Настройка правил именования |

### База знаний

- `config/knowledge/safety_rules.md` — правила безопасности
- `config/knowledge/metrics_glossary.md` — Health Score, формулы метрик
- `config/knowledge/fb_best_practices.md` — лучшие практики Facebook
- `config/knowledge/troubleshooting.md` — решение проблем
- `config/knowledge/geo_locations.md` — справочник гео-локаций

### Конфигурация

- `config/ad_accounts.md` — список рекламных аккаунтов
- `config/briefs/` — брифы по каждому аккаунту
- `config/creatives.md` — реестр креативов
- `config/naming_convention.md` — правила именования объявлений

## Установка

```bash
git clone https://github.com/skipaoff/ai-target-agent.git
cd ai-target-agent
cp .mcp.json.example .mcp.json
```

Заполни `.mcp.json` своими ключами:

- `META_APP_ID` — ID приложения из developers.facebook.com
- `META_APP_SECRET` — App Secret из Basic Settings
- `META_ACCESS_TOKEN` — System User Token (не User Access Token)

Запусти Claude Code в папке агента:

```bash
claude
```

Внутри Claude:

```
/mcp
```

Должно показать `meta-ads: connected ✓`. После этого:

```
/ads-agent
```

Полная инструкция по настройке Meta App, получению System User Token и подключению MCP сервера — в [QUICKSTART.md](QUICKSTART.md).

## Требования

- Claude Code — CLI или VS Code extension
- Anthropic API ключ
- MCP сервер `meta-ads-mcp` (устанавливается отдельно)
- Meta App в статусе Live (не Development)
- System User Token с правами `ads_management`, `ads_read`, `pages_manage_ads`

## Структура

```
ai-ads-agent/
├── README.md
├── QUICKSTART.md
├── CLAUDE.md
├── .mcp.json.example
├── .gitignore
├── skills/                # 12 Claude Skills
├── config/
│   ├── AGENT.md
│   ├── ad_accounts.md
│   ├── briefs/
│   │   └── _template.md
│   ├── creatives.md
│   ├── naming_convention.md
│   └── knowledge/
├── examples/              # Примеры брифов
├── docs/                  # Дополнительная документация
├── history/               # Логи действий (в .gitignore)
└── reports/               # HTML-отчёты (в .gitignore)
```

## Как работает

1. Пишешь `/ads-agent` или любой другой skill
2. Claude читает конфиги и брифы
3. Получает данные через MCP (Facebook API)
4. Анализирует и предлагает действия
5. После твоего подтверждения — выполняет

## Health Score

Система оценки эффективности AdSet (−100 до +100):

| Диапазон | Класс | Рекомендация |
|---|---|---|
| +25 и выше | very_good | Увеличить бюджет +20–30% |
| +5 до +24 | good | Держать или +10% |
| −5 до +4 | neutral | Мониторинг |
| −25 до −6 | slightly_bad | Снизить −20–50% |
| ниже −25 | bad | Пауза или −50% |

Компоненты: CPL Gap, Trends, CTR, CPM, Frequency.

## Технические нюансы

- Meta App обязательно опубликовано (Status: Live), иначе нельзя создавать Ad Creatives через API
- System User Token не протухает, в отличие от User Access Token
- В `.mcp.json` нужен полный абсолютный путь к `meta-ads-mcp` (получи через `pwd`)
- `.mcp.json` не коммитится — он в `.gitignore`
- Известный баг: `upload_ad_image` через MCP иногда падает, обход — `curl` напрямую

## Лицензия

Skills и конфигурация — MIT.
MCP сервер `meta-ads-mcp` использует BSL 1.1 (детали в репо MCP).

## Поддержка

Telegram: [@dreamm_assistant](https://t.me/dreamm_assistant)
