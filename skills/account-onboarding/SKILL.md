---
name: account-onboarding
description: Онбординг нового рекламного аккаунта. Создаёт конфигурацию для оптимизации.
---

# Account Onboarding

Интерактивный помощник для настройки нового Facebook Ads аккаунта.

---

## Твои задачи

1. Провести пользователя через сбор информации (~20 вопросов)
2. Валидировать данные на каждом этапе
3. Показать сводку и запросить подтверждение
4. Создать конфигурационные файлы
5. Проверить подключение к API

---

## Что создаётся в результате

```
.claude/ads-agent/config/
├── ad_accounts.md          ← Добавляется запись аккаунта
└── briefs/
    └── {account_name}.md   ← Создаётся новый бриф
```

---

## Workflow (8 шагов)

### Шаг 0: Приветствие

Выведи сообщение:

```
👋 Привет! Это онбординг нового рекламного аккаунта.

Я задам ~20 вопросов о бизнесе клиента:
• Базовая информация (4)
• Бизнес-параметры (5)
• Цель рекламы (1)
• Технические ID (2-5 в зависимости от цели)
• Направления/кампании (по 5 на каждое)

В конце создам:
- Запись в ad_accounts.md
- Бриф briefs/{name}.md

Готов начать?
```

Дождись подтверждения.

---

### Шаг 1: Базовая информация (4 вопроса)

**ОБЯЗАТЕЛЬНЫЕ** — все 4.

```
1. Название бизнеса:
   > (например: MyDent, FitLife, TechStore)

2. Facebook Ads Account ID:
   > act_XXXXXXXXX
   (найти в Business Manager → Ad Accounts)

3. Часовой пояс:
   > UTC+5 (Алматы) / UTC+3 (Москва) / UTC+6 (Астана) / другой

4. Ниша бизнеса:
   > E-commerce / Услуги / SaaS / Образование / Медицина / другая
```

**Валидация:**
- Account ID должен начинаться с `act_`
- Часовой пояс — формат UTC+X

**Дефолт:** Валюта всегда USD.

---

### Шаг 2: Бизнес-параметры (5 вопросов)

**ОБЯЗАТЕЛЬНЫЕ:** 1-3. **ОПЦИОНАЛЬНЫЕ:** 4-5.

```
1. Что продаёте / какие услуги:
   > (краткое описание основных продуктов/услуг)

2. Регион работы:
   > Казахстан / Россия / СНГ / Весь мир / конкретные города

3. Целевая аудитория:
   > Возраст: 25-45
   > Пол: все / жен / муж
   > Интересы: (ключевые)

4. Боли клиентов (для копирайтинга) [опционально]:
   > Какие проблемы решаете? Что волнует клиентов?

5. Конкурентные преимущества [опционально]:
   > Почему должны выбрать вас?
```

**После получения региона — найди geo ID:**

1. Прочитай справочник `config/knowledge/geo_locations.md`
2. Найди указанные города/страны в таблицах
3. Если город НЕ найден в справочнике — вызови MCP:
   ```
   search_geo_locations(query="{город}", location_types=["city"])
   ```
4. Сохрани для брифа:
   - `city_key` — ID города
   - `region_id` — ID региона (если есть)
   - `country_code` — код страны
   - `radius` — радиус по умолчанию (25 km для городов KZ, 40 km для RU)

---

### Шаг 3: Цель рекламы (1 вопрос)

**ОБЯЗАТЕЛЬНЫЙ.**

```
Выбери цель рекламы:

1. WhatsApp — сообщения в WhatsApp (оптимизация на начало переписки)
2. Instagram Трафик — клики на профиль Instagram
3. Site Leads — лиды на сайте (с Pixel)
4. Lead-формы — мгновенные формы Facebook/Instagram

> (1 / 2 / 3 / 4)
```

**Технические параметры по целям:**

| Цель | Campaign Objective | Optimization Goal | Destination Type |
|------|-------------------|-------------------|------------------|
| WhatsApp | OUTCOME_ENGAGEMENT | CONVERSATIONS | WHATSAPP |
| Instagram Трафик | OUTCOME_TRAFFIC | LINK_CLICKS | — |
| Site Leads | OUTCOME_LEADS | OFFSITE_CONVERSIONS | WEBSITE |
| Lead-формы | OUTCOME_LEADS | LEAD_GENERATION | ON_AD |

---

### Шаг 4: Технические ID (динамический)

В зависимости от цели (из Шага 3) собери нужные технические данные.

**ВСЕГДА ОБЯЗАТЕЛЬНЫЕ:**

```
1. Facebook Page ID:
   > (число, например: 123456789012345)
   (найти в Business Suite → Страницы → О странице)

2. Есть ли Instagram аккаунт? (да/нет)
   Если да:
   > Instagram Account ID: (число)
   (найти в Business Suite → Instagram аккаунты)
```

**В ЗАВИСИМОСТИ ОТ ЦЕЛИ:**

**Если цель = "WhatsApp":**
```
3. WhatsApp Business номер телефона:
   > +7XXXXXXXXXX (формат с кодом страны)
   (должен быть подключен к Business Manager)
```

**Если цель = "Lead-формы":**
```
3. Lead Form ID [если уже есть]:
   > (число) или "создать новую"
   (найти в Meta Business Suite → Формы)

4. Какие поля в форме нужны:
   > Имя / Телефон / Email / Город / другие
```

**Если цель = "Site Leads":**
```
3. URL сайта/лендинга:
   > https://example.com

4. Facebook Pixel ID:
   > (число, например: 123456789012345)
   (найти в Events Manager → Data Sources)

5. Есть ли настроенные события? (да/нет)
   Если да, какие:
   > Purchase / Lead / AddToCart / ViewContent / другие
```

**Если цель = "Instagram Трафик":**
```
(Дополнительных ID не требуется, только Page ID и Instagram ID)
```

**Валидация:**
- Page ID — только цифры, 15-16 знаков
- Instagram ID — только цифры
- WhatsApp — формат +XXXXXXXXXXX (E.164)
- Lead Form ID — только цифры или "создать новую"
- Pixel ID — только цифры, 15-16 знаков
- URL — валидный URL с https://

---

### Шаг 5: Направления/кампании (интерактивно)

```
Сколько направлений/кампаний в аккаунте?
> (1-10)
```

Для КАЖДОГО направления спроси:

```
Направление {N}:
1. Название: (например: Импланты, Виниры, Курсы)
2. Целевой CPL: $X (стоимость лида в USD)
3. Дневной бюджет: $X
4. Приоритет: высокий / средний / низкий
5. Campaign ID [если уже есть]: или "создать новую"
```

**Валидация:**
- CPL должен быть числом > 0
- Бюджет должен быть числом >= 5

---

### Шаг 6: ROAS (только для e-commerce)

**Спрашивать только если ниша = E-commerce.**

```
Целевой ROAS:

ROAS (Return on Ad Spend) — сколько долларов выручки приносит $1 рекламных расходов.

Пример расчёта:
• Потратили на рекламу: $100
• Получили выручки: $300
• ROAS = $300 / $100 = 3x

Формула: ROAS = Выручка / Расходы на рекламу

Типичные значения:
• 2x — минимально приемлемый
• 3x-4x — хороший результат
• 5x+ — отличный результат

> Целевой ROAS: Xх (например: 3x) или "не отслеживаю"
```

---

### Шаг 7: Валидация

Выведи ВСЮ собранную информацию в структурированном виде:

```
📋 СВОДКА ОНБОРДИНГА

## Базовая информация
- Название: {business_name}
- Account ID: {account_id}
- Валюта: USD
- Часовой пояс: {timezone}
- Ниша: {niche}

## Бизнес
- Продукт: {product}
- Регион: {region}
- ЦА: {target_audience}
- Боли: {pains}
- Преимущества: {advantages}

## Цель рекламы
- Цель: {goal} ({objective} / {optimization_goal})
- ROAS: {roas} (если e-commerce)

## Технические ID
- Page ID: {page_id}
- Instagram ID: {instagram_id} (или "нет")
- WhatsApp: {whatsapp_phone} (если WhatsApp)
- Lead Form ID: {lead_form_id} (если Lead-формы)
- Pixel ID: {pixel_id} (если Site Leads)
- Сайт: {website_url} (если Site Leads)

## Направления ({count})
| Название | Целевой CPL | Бюджет | Приоритет | Campaign ID |
|----------|-------------|--------|-----------|-------------|
{directions_rows}

---

Всё верно? (да / нет, изменить)
```

Если "нет" — спроси что изменить и обнови данные.

---

### Шаг 8: Генерация файлов

После подтверждения создай файлы:

#### 1. Добавь запись в ad_accounts.md

Прочитай `.claude/ads-agent/config/ad_accounts.md`, найди последний номер аккаунта и добавь в конец (перед "Пример формата"):

```markdown
---

## Аккаунт {N}: {business_name}

- **Account ID**: {account_id}
- **Page ID**: {page_id}
- **Instagram ID**: {instagram_id}
- **Название**: {business_name}
- **Бриф**: [briefs/{filename}.md](briefs/{filename}.md)
- **Статус**: активен
- **Валюта**: USD
- **Часовой пояс**: {timezone}
- **Цель рекламы**: {goal}
- **Заметки**: {niche}. {short_description}

---
```

#### 2. Создай briefs/{filename}.md

```markdown
# {business_name} - Бриф

> Создан через /account-onboarding {date}

---

## Бизнес

- **Ниша**: {niche}
- **Продукт**: {product}
- **Регион**: {region}
- **Целевая аудитория**: {target_audience}
- **Сайт/Landing**: {website}

### Для копирайтинга
- **Боли клиентов**: {pains}
- **Преимущества**: {advantages}

---

## Цель рекламы

- **Цель**: {goal}
- **Campaign Objective**: {objective}
- **Optimization Goal**: {optimization_goal}
- **Destination Type**: {destination_type}
- **Целевой ROAS**: {roas}

---

## Технические ID (для API)

### Обязательные
- **Facebook Page ID**: `{page_id}`
- **Instagram Account ID**: `{instagram_id}` <!-- или "нет" -->

### В зависимости от цели

<!-- Заполняется в зависимости от goal -->

#### Для WhatsApp:
- **WhatsApp номер**: `{whatsapp_phone}`

#### Для Lead-форм:
- **Lead Form ID**: `{lead_form_id}`
- **Поля формы**: {form_fields}

#### Для Site Leads:
- **URL сайта**: {website_url}
- **Pixel ID**: `{pixel_id}`
- **Настроенные события**: {pixel_events}

### Гео-локации

| Город/Страна | Тип | Key/Code | Region ID | Радиус |
|--------------|-----|----------|-----------|--------|
| {city_name} | city | {city_key} | {region_id} | {radius} km |
| {country_name} | country | {country_code} | — | — |

---

## Правила оптимизации (дефолты)

### Пороговые значения
- **Минимум impressions для решений**: 1000
- **Минимум конверсий для решений**: 3
- **Минимум дней до агрессивных действий**: 2
- **Ad-eater порог**: CPL > target × 3

### Бюджетные лимиты
- **Максимальное увеличение за раз**: 30%
- **Максимальное уменьшение за раз**: 50%

### Временные ограничения
- **Часовой пояс**: {timezone}
- **Не создавать adsets после**: 18:00

### CPL диапазоны
- **Normal**: target × 1.0-1.5
- **High**: target × 1.5-2.0
- **Ad-eater**: target × 3.0+

---

## Активные кампании/направления

| Название | Campaign ID | Цель CPL | Дневной бюджет | Приоритет | Статус |
|----------|-------------|----------|----------------|-----------|--------|
{directions_rows}

---

## Заметки

- Создан: {date}
- Онбординг провёл: Claude (skill: account-onboarding)

---

## История изменений

| Дата | Изменение |
|------|-----------|
| {date} | Создан бриф через /account-onboarding |
```

---

### Шаг 9: Тест подключения

После создания файлов проверь доступ к API:

```python
# Проверка доступа
campaigns = get_campaigns(account_id="{account_id}", limit=5)
```

Выведи результат:

```
✅ Онбординг завершён!

Созданные файлы:
1. .claude/ads-agent/config/ad_accounts.md — запись добавлена
2. .claude/ads-agent/config/briefs/{filename}.md — бриф создан

API проверка:
- Доступ к аккаунту: ✅
- Найдено кампаний: {count}

Теперь можно использовать:
- /ads-optimizer — оптимизация
- /ads-reporter — отчёты
- /campaign-manager — управление кампаниями
```

Если API вернул ошибку:

```
⚠️ Файлы созданы, но API недоступен!

Ошибка: {error_message}

Проверьте:
1. Account ID правильный?
2. MCP токен имеет доступ к аккаунту?
3. Аккаунт активен в Business Manager?

Файлы сохранены, можно исправить Account ID в briefs/{filename}.md
```

---

## Дефолтные значения

```yaml
currency: USD

optimization_rules:
  min_impressions: 1000
  min_conversions: 3
  min_days: 2
  max_budget_increase: 30
  max_budget_decrease: 50
  no_create_after: "18:00"
  ad_eater_multiplier: 3

cpl_ranges:
  normal_multiplier: 1.5
  high_multiplier: 2.0
  ad_eater_multiplier: 3.0

budget_defaults:
  adset_min: 5
```

---

## Цели и их параметры

| Цель | objective | optimization_goal | destination_type | billing_event |
|------|-----------|-------------------|------------------|---------------|
| WhatsApp | OUTCOME_ENGAGEMENT | CONVERSATIONS | WHATSAPP | IMPRESSIONS |
| Instagram Трафик | OUTCOME_TRAFFIC | LINK_CLICKS | — | IMPRESSIONS |
| Site Leads | OUTCOME_LEADS | OFFSITE_CONVERSIONS | WEBSITE | IMPRESSIONS |
| Lead-формы | OUTCOME_LEADS | LEAD_GENERATION | ON_AD | IMPRESSIONS |

---

## Валидация данных

| Поле | Проверка |
|------|----------|
| account_id | Начинается с `act_`, только цифры после |
| page_id | Только цифры, 15-16 знаков |
| instagram_id | Только цифры или "нет" |
| whatsapp_phone | Формат +XXXXXXXXXXX (E.164) |
| lead_form_id | Только цифры или "создать новую" |
| pixel_id | Только цифры, 15-16 знаков |
| website_url | Валидный URL с https:// |
| timezone | Формат UTC+X или UTC-X |
| target_cpl | Число > 0 |
| roas | Число > 1 или "не отслеживаю" |
| budget | Числа > 0, min >= 5 |

---

## Генерация filename

```
filename = business_name.lower()
           .replace(" ", "_")
           .replace("-", "_")
           .replace(/[^a-z0-9_]/g, "")

Примеры:
- "MyDent" → "mydent"
- "FitLife Studio" → "fitlife_studio"
- "ИП Иванов" → "ip_ivanov" (транслит)
```

---

## После онбординга

Чек-лист готовности:
- [ ] Бриф создан в `briefs/{name}.md`
- [ ] Аккаунт добавлен в `ad_accounts.md`
- [ ] API доступ проверен
- [ ] Готов к `/ads-optimizer`

---

## Частые ошибки

| Ошибка | Решение |
|--------|---------|
| Account ID без `act_` | Добавь префикс `act_` |
| Ошибка API доступа | Проверь токен в MCP, добавь аккаунт в Business Manager |
| Файл уже существует | Спроси: обновить или создать с суффиксом? |
