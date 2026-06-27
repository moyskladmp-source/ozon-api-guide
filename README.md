# Ozon Seller API — Методичка для разработчиков

> Основана на официальной документации: https://docs.ozon.ru/api/seller/

---

## Содержание

1. [Начало работы](#начало-работы)
2. [Авторизация](#авторизация)
3. [Товары](#товары)
4. [Цены и остатки](#цены-и-остатки)
5. [Аналитика](#аналитика)
6. [Заказы FBO](#заказы-fbo)
7. [Заказы FBS и rFBS](#заказы-fbs-и-rfbs)
8. [Поставки FBO](#поставки-fbo)
9. [Отчёты](#отчёты)
10. [Финансы](#финансы)
11. [Возвраты](#возвраты)
12. [Акции](#акции)
13. [Ограничения и ошибки](#ограничения-и-ошибки)
14. [Полезные паттерны](#полезные-паттерны)

---

## Начало работы

### Базовый URL

```
https://api-seller.ozon.ru
```

> ⚠️ **Важно:** С 16.05.2025 Seller API работает только **back-to-back** — прямые запросы из браузера заблокированы. Все запросы должны идти через бэкенд.

### Заголовки запроса

Все запросы требуют двух обязательных заголовков:

```http
Client-Id: ВАШ_CLIENT_ID
Api-Key: ВАШ_API_KEY
Content-Type: application/json
```

Ключи хранятся в личном кабинете: **Настройки → API ключи**.

### Пример базового запроса (Node.js)

```javascript
const axios = require('axios');

const headers = {
  'Client-Id': process.env.OZON_CLIENT_ID,
  'Api-Key': process.env.OZON_API_KEY,
  'Content-Type': 'application/json'
};

async function ozonApi(endpoint, body = {}) {
  const res = await axios.post(`https://api-seller.ozon.ru${endpoint}`, body, { headers });
  return res.data;
}
```

---

## Авторизация

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/roles` | Получить список ролей и методов по API-ключу |

Также доступна авторизация через **OAuth-токен** (нужна для Ozon Доставки и некоторых Premium-методов).

---

## Товары

### Категории и атрибуты

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/description-category/tree` | Дерево категорий и типов товаров |
| POST | `/v1/description-category/attribute` | Список характеристик категории |
| POST | `/v1/description-category/attribute/values` | Справочник значений характеристики |
| POST | `/v1/description-category/attribute/values/search` | Поиск по справочным значениям |

> Создание товаров доступно **только в категориях последнего уровня**.

### Создание и обновление товаров

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v3/product/import` | Создать или обновить товар |
| POST | `/v1/product/import/info` | Узнать статус создания/обновления |
| POST | `/v1/product/import-by-sku` | Создать товар по SKU (копия карточки) |
| POST | `/v1/product/attributes/update` | Обновить характеристики товара |
| POST | `/v1/product/pictures/import` | Загрузить или обновить изображения |
| POST | `/v1/product/update/offer-id` | Изменить артикулы товаров |

### Получение информации о товарах

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v3/product/list` | Список всех товаров |
| POST | `/v3/product/info/list` | Информация о товарах по идентификаторам (фото, статус) |
| POST | `/v4/product/info/attributes` | Характеристики товара |
| POST | `/v1/product/info/description` | Описание товара |
| POST | `/v4/product/info/limit` | Лимиты на ассортимент и создание |
| POST | `/v1/product/rating-by-sku` | Контент-рейтинг товаров |
| POST | `/v2/product/pictures/info` | Изображения товаров |
| POST | `/v1/product/info/subscription` | Количество подписчиков на товар |
| POST | `/v1/product/related-sku/get` | Получить связанные SKU |
| POST | `/v1/product/info/wrong-volume` | Товары с некорректными ОВХ |
| POST | `/v1/product/placement-zone/info` | Зоны размещения по SKU перед поставкой |

### Архив и удаление

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/product/archive` | Перенести товар в архив |
| POST | `/v1/product/unarchive` | Вернуть товар из архива |
| POST | `/v2/products/delete` | Удалить товар без SKU из архива |

### Штрихкоды

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/barcode/add` | Привязать штрихкод к товару |
| POST | `/v1/barcode/generate` | Создать штрихкод для товара |

### Уценённые товары

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/product/info/discounted` | Информация об уценке по SKU |
| POST | `/v1/product/update/discount` | Установить скидку на уценённый товар |

---

## Цены и остатки

### Цены

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/product/import/prices` | Обновить цену (не чаще 10 раз/час на товар) |
| POST | `/v5/product/info/prices` | Получить информацию о цене |
| POST | `/v1/product/action/timer/update` | Обновить таймер актуальности минимальной цены |
| POST | `/v1/product/action/timer/status` | Получить статус таймера |

### Остатки FBO

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v2/products/stocks` | Обновить количество товаров на складах |
| POST | `/v4/product/info/stocks` | Информация о количестве (FBS, rFBS, FBP) |
| POST | `/v2/analytics/stock_on_warehouses` | ⚠️ Устаревает → переходить на `/v1/analytics/stocks` |

### Остатки FBS

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v2/product/info/stocks-by-warehouse/fbs` | Остатки на складах продавца |
| POST | `/v1/product/info/warehouse/stocks` | Остатки на складе FBS и rFBS |

### Стратегии ценообразования

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/pricing-strategy/list` | Список стратегий |
| POST | `/v1/pricing-strategy/competitors/list` | Список конкурентов |

---

## Аналитика

> 💡 Для получения полных данных нужна подписка **Premium Plus** или **Premium Pro**.
> Без подписки: данные только за последние 3 месяца, ограниченные метрики.

### Основные эндпоинты

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/analytics/data` | **Главный аналитический метод** — данные по метрикам за период |
| POST | `/v1/analytics/stocks` | Аналитика по остаткам на складах |
| POST | `/v1/analytics/turnover/stocks` | Оборачиваемость — сколько дней хватит остатка |
| POST | `/v1/analytics/product-queries` | Поисковые запросы по товарам (Premium) |
| POST | `/v1/analytics/product-queries/details` | Детализация запросов по товару (Premium) |

### `/v1/analytics/data` — подробнее

Самый мощный метод. Позволяет получить продажи, показы, конверсию в разрезе любого периода.

**Группировки (dimension) — доступны всем:**
- `sku` — по товару
- `spu` — по объединённой карточке
- `day` — по дням
- `week` — по неделям
- `month` — по месяцам

**Группировки — только Premium Plus:**
- `year`, `category1`, `category2`, `brand`, `modelID`, `descriptionType`

**Метрики — доступны всем:**
- `revenue` — заказано на сумму
- `ordered_units` — заказано товаров

**Метрики — только Premium Plus:**
- `hits_view_search` / `hits_view_pdp` / `hits_view` — показы
- `hits_tocart_search` / `hits_tocart_pdp` / `hits_tocart` — в корзину
- `session_view_search` / `session_view_pdp` / `session_view` — сессии
- `conv_tocart_search` / `conv_tocart_pdp` / `conv_tocart` — конверсии
- `returns` — возвраты
- `cancellations` — отмены
- `delivered_units` — доставлено
- `position_category` — позиция в поиске

**Пример запроса — продажи по месяцам:**
```javascript
const data = await ozonApi('/v1/analytics/data', {
  date_from: '2025-01-01',
  date_to: '2026-06-01',
  dimension: ['sku', 'month'],
  metrics: ['ordered_units', 'revenue'],
  limit: 1000
});
```

**Пример запроса — продажи по конкретному товару:**
```javascript
const data = await ozonApi('/v1/analytics/data', {
  date_from: '2025-01-01',
  date_to: '2026-06-01',
  dimension: ['month'],
  filters: [{ key: 'sku', op: 'EQ', value: '123456789' }],
  metrics: ['ordered_units'],
  limit: 12
});
```

> ⚠️ Лимит: не более **1 запроса в минуту** для `/v1/analytics/data`.

### Получение остатков по кластерам

```javascript
// Склады → кластеры
const clusters = await ozonApi('/v1/cluster/list', {});
// Структура: clusters[].logistic_clusters[].warehouses[]

// Остатки по складам
const stocks = await ozonApi('/v2/analytics/stock_on_warehouses', {
  limit: 1000,
  warehouse_type: 'ALL'
});
```

---

## Заказы FBO

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v2/posting/fbo/list` | Список отправлений FBO |
| POST | `/v2/posting/fbo/get` | Детали отправления FBO |

---

## Заказы FBS и rFBS

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v3/posting/fbs/get` | Детали FBS-отправления |
| POST | `/v3/posting/fbs/list` | Список FBS-отправлений |
| POST | `/v3/posting/fbs/unfulfilled/list` | Невыполненные FBS-отправления |

---

## Поставки FBO

Подробнее о поставках FBO — в разделе **"Создание и управление заявками на поставку FBO"** в документации.

### Кластеры складов

```javascript
// Список кластеров и складов
const res = await ozonApi('/v1/cluster/list', {});
// res.clusters[].name — название кластера
// res.clusters[].logistic_clusters[].warehouses[].name — склады
```

---

## Отчёты

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/v1/report/list` | Список сформированных отчётов |
| POST | `/v1/report/info` | Информация о конкретном отчёте |

---

## Финансы

> Некоторые методы требуют Premium-подписку.

Раздел: **FinanceAPI** — финансовые отчёты, транзакции, выплаты.

---

## Возвраты

| Раздел | Описание |
|--------|----------|
| ReturnsAPI | Возвраты FBO и FBS |
| RFBSReturnsAPI | Возвраты rFBS |
| ReturnAPI | Возвратные отгрузки |
| CancellationAPI | Отмены заказов |

---

## Акции

### Акции Ozon

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/v1/actions` | Список акций |
| POST | `/v1/actions/candidates` | Товары, доступные для акции |
| POST | `/v1/actions/products` | Товары, участвующие в акции |
| POST | `/v1/actions/products/activate` | Добавить товар в акцию |
| POST | `/v1/actions/products/deactivate` | Удалить товар из акции |

### Акции продавца

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/v1/seller-actions/list` | Список акций продавца |
| POST | `/v1/seller-actions/create/discount` | Создать скидку |
| POST | `/v1/seller-actions/create/installment` | Создать рассрочку |
| POST | `/v1/seller-actions/products/add` | Добавить товары в акцию |

---

## Ограничения и ошибки

### Rate limits (лимиты запросов)

| Эндпоинт | Лимит |
|----------|-------|
| `/v1/analytics/data` | 1 запрос в минуту |
| `/v1/product/import/prices` | 10 обновлений цены в час на товар |
| `/v1/product/attributes/update` | Лимит в минуту и в сутки |
| Большинство методов | ~10 RPS (при превышении — ошибка 429) |

### Типичные ошибки

| Код / Текст | Причина | Решение |
|-------------|---------|---------|
| `429` | Превышен лимит запросов | Добавить паузы между запросами |
| `Circle is open` | Слишком много одинаковых/ошибочных запросов | Выждать, исправить запрос |
| `OFFER_ID_ALREADY_EXISTS` | Артикул уже используется | Проверить уникальность offer_id |
| `autoarchive_restore_failed` | Превышен лимит восстановления из архива | Обратиться в поддержку |
| `InvalidArgument` | Неверные параметры запроса | Проверить тело запроса |

---

## Полезные паттерны

### Пагинация через cursor

Многие методы возвращают данные постранично. Паттерн:

```javascript
async function fetchAll(endpoint, body) {
  let cursor = '';
  const result = [];
  
  while (true) {
    const res = await ozonApi(endpoint, { ...body, cursor, limit: 1000 });
    result.push(...(res.items || res.result || []));
    
    if (!res.cursor || res.items?.length < 1000) break;
    cursor = res.cursor;
  }
  
  return result;
}
```

### Пагинация через offset

```javascript
async function fetchAllByOffset(endpoint, body) {
  const result = [];
  let offset = 0;
  const limit = 1000;

  while (true) {
    const res = await ozonApi(endpoint, { ...body, offset, limit });
    const items = res.result?.items || res.items || [];
    result.push(...items);
    
    if (items.length < limit) break;
    offset += limit;
  }

  return result;
}
```

### Батчинг запросов

```javascript
// Разбить массив на чанки по N элементов
function chunks(arr, size) {
  const result = [];
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  return result;
}

// Пример: получить фото для 300 товаров (лимит 100 за раз)
const images = {};
for (const chunk of chunks(offerIds, 100)) {
  const res = await ozonApi('/v3/product/info/list', { offer_id: chunk });
  for (const item of res.items || []) {
    images[item.offer_id] = item.primary_image?.[0] || item.images?.[0] || null;
  }
}
```

### Прокси для изображений Ozon

Ozon блокирует прямые запросы к своим CDN-изображениям из браузера (hotlink protection). Решение — серверный прокси:

```javascript
// Бэкенд (Express)
app.get('/img-proxy', async (req, res) => {
  const url = req.query.url;
  if (!url || !url.startsWith('https://cdn1.ozon.ru')) {
    return res.status(400).send('Bad url');
  }
  const r = await axios.get(url, {
    responseType: 'stream',
    headers: { 'Referer': 'https://www.ozon.ru/' }
  });
  res.set('Content-Type', r.headers['content-type']);
  r.data.pipe(res);
});

// Фронтенд
<img src="/img-proxy?url=https://cdn1.ozon.ru/..." />
```

### Маппинг складов → кластеры

```javascript
const clusterData = await ozonApi('/v1/cluster/list', {});
const whToCluster = {};

for (const cl of clusterData.clusters || []) {
  for (const lc of cl.logistic_clusters || []) {  // ⚠️ не cl.warehouses!
    for (const wh of lc.warehouses || []) {
      whToCluster[wh.name] = cl.name;
      // нормализованный ключ для сравнения
      whToCluster[wh.name.toUpperCase().replace(/[_\-\s]/g, '')] = cl.name;
    }
  }
}
```

### Сезонность по истории продаж

```javascript
// Получить продажи по месяцам за последние 12 месяцев
const sales = await ozonApi('/v1/analytics/data', {
  date_from: '2025-01-01',
  date_to: '2026-06-01',
  dimension: ['sku', 'month'],
  metrics: ['ordered_units'],
  limit: 1000
});

// Посчитать коэффициент сезонности по месяцу
const monthlyTotals = {};
for (const row of sales.result.data) {
  const month = row.dimensions.find(d => /^\d{4}-\d{2}$/.test(d.id))?.id;
  const units = row.metrics[0];
  if (month) monthlyTotals[month] = (monthlyTotals[month] || 0) + units;
}
const avg = Object.values(monthlyTotals).reduce((a, b) => a + b, 0) / 12;
const seasonCoeff = {};
for (const [m, v] of Object.entries(monthlyTotals)) {
  seasonCoeff[m.slice(5)] = v / avg; // '01' → 0.8, '12' → 1.4 и т.д.
}
```

---

## Premium-методы

Раздел **Premium** в документации содержит расширенные методы для аналитики, чатов с покупателями и финансов. Доступны при наличии подписки **Premium**, **Premium Plus** или **Premium Pro**.

---

## Пуш-уведомления

Ozon умеет присылать уведомления на ваш сервер о новых заказах, отменах и других событиях. Настраивается в личном кабинете. Ваш сервис должен отвечать по стандартам REST API.

---

## Полезные ссылки

- 📖 [Официальная документация](https://docs.ozon.ru/api/seller/)
- 🔑 [Управление API-ключами](https://seller.ozon.ru/app/settings/api-keys)
- 📊 [Аналитика в кабинете](https://seller.ozon.ru/app/analytics/charts)
- 💬 [Платформа для разработчиков Ozon for dev](https://dev.ozon.ru/)
