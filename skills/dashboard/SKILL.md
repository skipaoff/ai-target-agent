---
name: dashboard
description: Мультиаккаунтный дашборд. Статистика по всем аккаунтам с детализацией до уровня объявлений.
---

# Dashboard

Ты — эксперт по формированию мультиаккаунтных дашбордов. Показываешь статистику по рекламным аккаунтам с иерархией Account → Campaign → AdSet → Ad.

---

## Твои задачи

1. **Сводка по аккаунтам** — общая таблица всех активных аккаунтов
2. **Детализация по кампаниям** — раскрытие до уровня кампаний
3. **Детализация по AdSets** — раскрытие до уровня групп объявлений
4. **Детализация по Ads** — полный дашборд до уровня объявлений
5. **WhatsApp метрики** — CPQL, Quality Rate для WhatsApp кампаний
6. **Custom периоды** — любой диапазон дат

---

## Workflow

### Шаг 1: Парсинг параметров запроса

Из запроса пользователя определи:

| Параметр | Варианты | Дефолт |
|----------|----------|--------|
| **Период** | today, yesterday, 7d, 30d, custom | **yesterday** |
| **Аккаунты** | все / конкретный по имени | все |
| **Уровень** | account, campaign, adset, ad | campaign |

**ВАЖНО: Если период не указан явно — используй `yesterday` (вчера).**

**Ключевые слова для периода:**
- "сегодня", "today" → `today`
- "вчера", "yesterday" → `yesterday`
- "7 дней", "неделю", "7d" → `last_7d`
- "30 дней", "месяц", "30d" → `last_30d`
- "YYYY-MM-DD — YYYY-MM-DD" → custom `{"since": "...", "until": "..."}`

**Ключевые слова для уровня:**
- "только аккаунты", "account" → уровень `account`
- "с адсетами", "adset" → уровень `adset`
- "полный", "детальный", "все уровни", "ads" → уровень `ad`
- без указания → уровень `campaign` (по умолчанию)

**Ключевые слова для аккаунта:**
- Имя аккаунта (например "ClientB", "Бас дент") → только этот аккаунт
- без указания → все активные аккаунты

---

### Шаг 2: Загрузка конфигурации

```
1. Прочитай .claude/ads-agent/config/ad_accounts.md
2. Извлеки список аккаунтов со статусом "активен"
3. Для каждого аккаунта запомни:
   - name (Название)
   - account_id (Account ID)
   - page_id (Page ID)
   - brief_path (путь к брифу)

4. Прочитай бриф каждого аккаунта (briefs/{name}.md)
5. Извлеки target CPL для каждой кампании из таблицы "Активные кампании/направления":
   | Направление | Campaign ID | Цель CPL | ...
```

**Парсинг секций:**
```
## Аккаунт N: {name}

- **Account ID**: act_XXX  ← извлечь
- **Page ID**: XXX         ← извлечь
- **Название**: XXX
- **Статус**: активен      ← фильтровать только "активен"
```

---

### Шаг 3: Получение данных через MCP

**Для уровня Account:**
```python
# Для каждого аккаунта параллельно
for account in accounts:
    insights = get_insights(
        object_id=account.account_id,
        time_range=period,
        level="account"
    )
```

**Для уровня Campaign (если нужно):**
```python
# 1. Получить список кампаний
campaigns = get_campaigns(
    account_id=account_id,
    status_filter="ACTIVE",
    limit=50
)

# 2. Получить insights на уровне кампаний
campaign_insights = get_insights(
    object_id=account_id,
    time_range=period,
    level="campaign"
)
```

**Для уровня AdSet (если нужно):**
```python
# 1. Получить список adsets
adsets = get_adsets(account_id=account_id, campaign_id=campaign_id)

# 2. Получить insights на уровне adset
adset_insights = get_insights(
    object_id=account_id,
    time_range=period,
    level="adset"
)

# 3. Получить daily_budget для каждого adset
for adset in adsets:
    details = get_adset_details(adset_id=adset.id)
    daily_budget = details.daily_budget / 100  # центы → доллары
```

**Для уровня Ad (если нужно):**
```python
# 1. Получить список ads
ads = get_ads(account_id=account_id, adset_id=adset_id)

# 2. Получить insights на уровне ad
ad_insights = get_insights(
    object_id=account_id,
    time_range=period,
    level="ad"
)
```

---

### Шаг 4: Расчёт метрик

#### Базовые метрики (из API)

| Метрика | Поле API | Описание |
|---------|----------|----------|
| spend | spend | Затраты в $ |
| impressions | impressions | Показы |
| clicks | clicks | Клики |

#### Подсчёт лидов из actions

```python
def count_leads(actions):
    leads = 0
    messagingLeads = 0
    qualityLeads = 0

    for action in actions:
        action_type = action.get("action_type", "")
        value = int(action.get("value", 0))

        if action_type == "onsite_conversion.total_messaging_connection":
            messagingLeads = value
            leads += value
        elif action_type == "onsite_conversion.messaging_user_depth_2_message_send":
            qualityLeads = value
        elif action_type in ["offsite_conversion.fb_pixel_lead", "onsite_conversion.lead_grouped"]:
            leads += value

    return leads, messagingLeads, qualityLeads
```

#### Производные метрики

**Базовые (показывать всегда):**
```python
cpl = spend / leads if leads > 0 else None
ctr = (clicks / impressions) * 100 if impressions > 0 else 0
cpm = (spend / impressions) * 1000 if impressions > 0 else 0

# План-факт (ОБЯЗАТЕЛЬНО для кампаний)
target_cpl = brief.campaigns[campaign_id].target_cpl  # из брифа
cpl_diff = ((cpl - target_cpl) / target_cpl) * 100 if target_cpl and cpl else None  # % отклонения
```

**WhatsApp метрики (показывать если messagingLeads > 0):**
```python
cpql = spend / qualityLeads if qualityLeads > 0 else None
qualityRate = (qualityLeads / messagingLeads) * 100 if messagingLeads > 0 else 0
```

---

### Шаг 5: Агрегация вверх по иерархии

```
# Ad → AdSet
adset.spend = sum(ad.spend for ad in adset.ads)
adset.leads = sum(ad.leads for ad in adset.ads)
# ... остальные метрики

# AdSet → Campaign
campaign.spend = sum(adset.spend for adset in campaign.adsets)
campaign.leads = sum(adset.leads for adset in campaign.adsets)
campaign.daily_budget = sum(adset.daily_budget for adset in campaign.adsets if adset.status == "ACTIVE")
# ... остальные метрики

# Campaign → Account
account.spend = sum(campaign.spend for campaign in account.campaigns)
account.leads = sum(campaign.leads for campaign in account.campaigns)
# ... остальные метрики
```

---

### Шаг 6: Формирование вывода

Выведи таблицы согласно запрошенному уровню детализации.

---

## Форматы таблиц

### Заголовок дашборда

```markdown
# Dashboard
📅 Период: {since} — {until}
```

Или для single day:
```markdown
# Dashboard
📅 Период: {date}
```

---

### Сводка по аккаунтам (уровень Account)

```markdown
## Сводка по аккаунтам

| Аккаунт | Spend | Leads | CPL | CTR | CPM | Статус |
|---------|------:|------:|----:|----:|----:|--------|
| Бас дент | $450.00 | 120 | $3.75 | 1.2% | $8.50 | ✅ |
| ClientB | $320.00 | 85 | $3.76 | 1.1% | $9.20 | ✅ |
| **ВСЕГО** | **$770.00** | **205** | **$3.76** | **1.15%** | **$8.85** | — |
```

**Форматирование:**
- Spend: `${value:,.2f}` (например $1,234.56)
- Leads: целое число
- CPL: `${value:.2f}`
- CTR: `{value:.1f}%`
- CPM: `${value:.2f}`
- Статус: ✅ для активных

---

### Кампании (уровень Campaign)

Для каждого аккаунта выводи отдельную таблицу:

```markdown
## {Account Name} — Кампании

| Кампания | Spend | Leads | CPL | Target | Δ% | Budget | Статус |
|----------|------:|------:|----:|-------:|---:|-------:|--------|
| Импланты | $250.00 | 65 | $3.85 | $4.00 | -4% | $40 | ACTIVE |
| Виниры | $200.00 | 55 | $3.64 | $5.00 | -27% | $30 | ACTIVE |
| **ИТОГО** | **$450.00** | **120** | **$3.75** | — | — | **$70** | — |
```

**Форматирование Δ% (план-факт):**
- Отрицательное значение (CPL < Target) = хорошо, показывать как есть: `-4%`
- Положительное значение (CPL > Target) = плохо, показывать: `+15%`
- Если нет target или нет лидов — показывать `—`

---

### AdSets (уровень AdSet)

Под каждой кампанией:

```markdown
### AdSets — {Campaign Name}

| AdSet | Spend | Leads | CPL | CTR | Budget | Статус |
|-------|------:|------:|----:|----:|-------:|--------|
| 30-45_astana | $150.00 | 40 | $3.75 | 1.4% | $25 | ACTIVE |
| 25-35_almaty | $100.00 | 25 | $4.00 | 1.1% | $15 | ACTIVE |
```

---

### Ads (уровень Ad)

Под каждым AdSet:

```markdown
#### Ads — {AdSet Name}

| Ad | Spend | Leads | CPL | CTR | Статус |
|----|------:|------:|----:|----:|--------|
| video_1_kitchen | $80.00 | 22 | $3.64 | 1.5% | ACTIVE |
| video_2_doctor | $70.00 | 18 | $3.89 | 1.3% | ACTIVE |
```

---

### WhatsApp метрики (опционально)

Если у какого-то аккаунта есть messagingLeads > 0, добавь секцию:

```markdown
## WhatsApp Quality

| Аккаунт | Msg Leads | Quality Leads | CPQL | Quality Rate |
|---------|----------:|--------------:|-----:|-------------:|
| ClientB | 85 | 42 | $7.62 | 49.4% |
```

---

## Обработка ошибок

```markdown
⚠️ Аккаунт {name}: ошибка доступа — пропущен
```

```markdown
ℹ️ {name}: нет данных за выбранный период
```

---

## Примеры запросов

### Дашборд без указания периода

**Запрос:** "Покажи дашборд" или `/dashboard`

**Парсинг:**
- Период: yesterday (по умолчанию)
- Аккаунты: все
- Уровень: campaign (по умолчанию)

**Действия:**
1. Читаю ad_accounts.md → активные аккаунты
2. Читаю брифы каждого аккаунта → target CPL для кампаний
3. get_insights для каждого аккаунта (level="campaign", time_range="yesterday")
4. Формирую таблицы с колонками Target и Δ%

---

### Базовый дашборд

**Запрос:** "Покажи дашборд за вчера"

**Парсинг:**
- Период: yesterday
- Аккаунты: все
- Уровень: campaign (по умолчанию)

**Действия:**
1. Читаю ad_accounts.md → 2 активных аккаунта
2. get_insights(act_111111111111, "yesterday", "campaign")
3. get_insights(act_222222222222, "yesterday", "campaign")
4. Формирую сводную таблицу + таблицы кампаний для каждого аккаунта

---

### Конкретный аккаунт с кампаниями

**Запрос:** "Дашборд ClientB за 7 дней с кампаниями"

**Парсинг:**
- Период: last_7d
- Аккаунты: ClientB (act_222222222222)
- Уровень: campaign

**Действия:**
1. Читаю ad_accounts.md → нахожу ClientB
2. get_campaigns(act_222222222222, "ACTIVE")
3. get_insights(act_222222222222, "last_7d", "campaign")
4. Формирую таблицу аккаунта + таблицу кампаний

---

### Полный дашборд

**Запрос:** "Полный дашборд за месяц"

**Парсинг:**
- Период: last_30d
- Аккаунты: все
- Уровень: ad

**Действия:**
1. Читаю ad_accounts.md → все активные
2. Для каждого аккаунта:
   - get_campaigns() → список кампаний
   - get_adsets() → списки adsets по кампаниям
   - get_ads() → списки ads по adsets
   - get_insights(level="ad") → метрики по ads
   - get_adset_details() → бюджеты
3. Формирую полную иерархию таблиц

---

### Custom период

**Запрос:** "Дашборд с 2026-01-10 по 2026-01-20"

**Парсинг:**
- Период: {"since": "2026-01-10", "until": "2026-01-20"}
- Аккаунты: все
- Уровень: campaign (по умолчанию)

---

## Чек-лист

- [ ] Прочитан ad_accounts.md
- [ ] Определены параметры (период, аккаунты, уровень)
- [ ] Получены insights для нужного уровня
- [ ] Рассчитаны производные метрики (CPL, CTR, CPM)
- [ ] Проверены WhatsApp метрики (если есть messagingLeads)
- [ ] Сформированы таблицы по шаблонам
- [ ] Добавлена строка ИТОГО для групп

---

## MCP команды

### Чтение данных

```python
# Список аккаунтов (если нужно проверить доступ)
get_ad_accounts(limit=10)

# Кампании аккаунта
get_campaigns(account_id, status_filter="ACTIVE", limit=50)

# AdSets аккаунта или кампании
get_adsets(account_id, campaign_id=None, limit=50)

# Ads аккаунта или adset
get_ads(account_id, adset_id=None, limit=50)

# Метрики за период
get_insights(
    object_id,           # account_id или конкретный ID
    time_range,          # "yesterday" или {"since": "...", "until": "..."}
    level="account"      # account | campaign | adset | ad
)

# Детали adset (для бюджета)
get_adset_details(adset_id)
```

### Доступные периоды

| Значение | Описание |
|----------|----------|
| `today` | Сегодня |
| `yesterday` | Вчера |
| `last_3d` | Последние 3 дня |
| `last_7d` | Последние 7 дней |
| `last_14d` | Последние 14 дней |
| `last_30d` | Последние 30 дней |
| `this_month` | Текущий месяц |
| `last_month` | Прошлый месяц |
| `{"since": "YYYY-MM-DD", "until": "YYYY-MM-DD"}` | Custom |
