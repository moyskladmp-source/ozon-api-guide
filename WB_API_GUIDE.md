# Wildberries Seller API — Методичка для разработчиков

> Основана на официальной документации: https://dev.wildberries.ru/
> Swagger: https://dev.wildberries.ru/swagger

---

## Содержание

1. [Начало работы](#начало-работы)
2. [Авторизация и токены](#авторизация-и-токены)
3. [Работа с товарами](#работа-с-товарами)
4. [Цены и скидки](#цены-и-скидки)
5. [Склады и остатки FBS](#склады-и-остатки-fbs)
6. [Поставки FBW](#поставки-fbw)
7. [Заказы FBS](#заказы-fbs)
8. [Заказы DBW / DBS / Самовывоз](#заказы-dbw--dbs--самовывоз)
9. [Аналитика и данные](#аналитика-и-данные)
10. [Отчёты](#отчёты)
11. [Маркетинг и продвижение](#маркетинг-и-продвижение)
12. [Общение с покупателями](#общение-с-покупателями)
13. [Финансы и документы](#финансы-и-документы)
14. [Тарифы](#тарифы)
15. [Лимиты запросов](#лимиты-запросов)
16. [Ошибки](#ошибки)
17. [Полезные паттерны](#полезные-паттерны)

---

## Начало работы

### Базовые URL

WB API использует разные домены для разных категорий методов:

```
https://content-api.wildberries.ru   — товары, карточки, медиа
https://marketplace-api.wildberries.ru — заказы, склады, остатки
https://statistics-api.wildberries.ru  — статистика и отчёты
https://advert-api.wildberries.ru      — реклама и продвижение
https://feedbacks-api.wildberries.ru   — отзывы и вопросы
https://seller-analytics-api.wildberries.ru — аналитика
https://supplies-api.wildberries.ru    — поставки FBW
https://common-api.wildberries.ru      — общее (тарифы, информация)
https://documents-api.wildberries.ru   — документы и финансы
```

### Заголовок авторизации

Все запросы требуют заголовок:

```http
Authorization: Bearer ВАШ_ТОКЕН
Content-Type: application/json
```

### Пример базового запроса (Node.js)

```javascript
const axios = require('axios');

const TOKEN = process.env.WB_TOKEN;

async function wbApi(baseUrl, endpoint, params = {}, method = 'GET') {
  const config = {
    method,
    url: `${baseUrl}${endpoint}`,
    headers: {
      'Authorization': `Bearer ${TOKEN}`,
      'Content-Type': 'application/json'
    }
  };
  
  if (method === 'GET') config.params = params;
  else config.data = params;
  
  const res = await axios(config);
  return res.data;
}
```

---

## Авторизация и токены

### Типы токенов

| Тип | Описание | Лимиты |
|-----|----------|--------|
| **Персональный** | Для личного использования | 300 запросов/мин |
| **Базовый** | Для интеграций без сервисного соглашения | 150 запросов/мин |
| **Сервисный** | Для авторизованных партнёров WB | 300 запросов/мин |
| **Тестовый** | Для разработки в песочнице | 150 запросов/мин |

### Как создать токен

1. Личный кабинет → **Интеграции по API**
2. Нажать **+ Создать токен**
3. Выбрать категории доступа (только нужные!)
4. Выбрать уровень: **Чтение и запись** или **Только чтение**

> ⚠️ Выбирайте только те категории, с которыми планируете работать. Если токен утечёт — ущерб будет минимальным.

### Категории токенов

- **Контент** — товары, карточки, медиа
- **Маркетплейс** — заказы, склады, остатки
- **Статистика** — отчёты и аналитика
- **Реклама** — продвижение и кампании
- **Отзывы** — вопросы и отзывы покупателей
- **Поставки** — поставки FBW
- **Аналитика** — воронка продаж, история остатков

---

## Работа с товарами

> Токен: **Контент** | Лимит: 100 запросов/мин

### Категории и характеристики

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/content/v2/object/all` | Список всех предметов (категорий) |
| GET | `/content/v2/object/charcs/{subjectId}` | Характеристики предмета |
| GET | `/content/v2/directory/brands` | Справочник брендов |
| GET | `/content/v2/directory/colors` | Справочник цветов |
| GET | `/content/v2/directory/seasons` | Справочник сезонов |
| GET | `/content/v2/directory/vat` | Справочник ставок НДС |

### Создание и редактирование карточек

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/content/v2/cards/upload` | Создать карточки товаров |
| POST | `/content/v2/cards/update` | Редактировать карточки |
| POST | `/content/v2/cards/upload/add` | Добавить товар к существующей карточке |
| POST | `/content/v2/cards/moveNm` | Перенести товар в другую карточку |
| POST | `/content/v2/cards/delete/trash` | Перенести карточки в корзину |
| POST | `/content/v2/cards/recover` | Восстановить карточки из корзины |

### Получение карточек

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/content/v2/get/cards/list` | Список карточек с фильтрами |
| POST | `/content/v2/cards/error/list` | Несозданные карточки с ошибками |

### Медиафайлы

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/content/v3/media/save` | Загрузить медиафайлы в карточку |
| POST | `/content/v3/media/file` | Загрузить файл напрямую |

### Ярлыки (теги)

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/content/v2/tags` | Список ярлыков |
| POST | `/content/v2/tag` | Создать ярлык |
| PATCH | `/content/v2/tag/{id}` | Изменить ярлык |
| DELETE | `/content/v2/tag/{id}` | Удалить ярлык |
| POST | `/content/v2/tags/goods/link` | Привязать ярлык к товарам |

---

## Цены и скидки

> Токен: **Маркетплейс** | Лимит: 300 запросов/мин

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v2/list/goods/filter` | Информация о ценах товаров |
| POST | `/api/v2/upload/task` | Установить цены и скидки |
| GET | `/api/v2/history/goods/task` | История изменений цен |
| GET | `/api/v2/buffer/goods/filter` | Товары в буфере цен |
| POST | `/api/v2/buffer/goods/accept` | Принять изменения цен из буфера |
| POST | `/api/v2/buffer/goods/reject` | Отклонить изменения цен |

### Пример установки цены

```javascript
await wbApi('https://discounts-prices-api.wildberries.ru', '/api/v2/upload/task', {
  data: [
    {
      nmID: 123456789,    // артикул WB
      price: 1500,        // цена в рублях
      discount: 20        // скидка в %
    }
  ]
}, 'POST');
```

---

## Склады и остатки FBS

> Токен: **Маркетплейс** | Лимит: 300 запросов/мин

### Склады продавца

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v3/warehouses` | Список складов продавца |
| POST | `/api/v3/warehouses` | Создать склад |
| PUT | `/api/v3/warehouses/{warehouseId}` | Изменить склад |
| DELETE | `/api/v3/warehouses/{warehouseId}` | Удалить склад |

### Остатки на складах продавца

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v3/stocks/{warehouseId}` | Остатки на складе |
| PUT | `/api/v3/stocks/{warehouseId}` | Обновить остатки |
| DELETE | `/api/v3/stocks/{warehouseId}` | Удалить остатки |

### Пример обновления остатков

```javascript
await wbApi('https://marketplace-api.wildberries.ru', `/api/v3/stocks/${warehouseId}`, {
  stocks: [
    { sku: '1234567890123', amount: 10 },
    { sku: '9876543210987', amount: 5 }
  ]
}, 'PUT');
```

---

## Поставки FBW

> Токен: **Поставки** | FBW = "Fulfillment by Wildberries" (склад WB)

### Информация для формирования поставок

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/api/v1/acceptance/options` | Доступные склады и типы упаковки |
| GET | `/api/v1/acceptance/coefficients` | Коэффициенты приёмки по складам |
| GET | `/api/v1/warehouses` | Список складов WB |
| GET | `/api/v1/transit/directions` | Транзитные направления |

### Информация о поставках

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/supplies` | Список поставок |
| GET | `/api/v1/supplies/{supplyId}` | Информация о поставке |
| GET | `/api/v1/supplies/{supplyId}/orders` | Заказы в поставке |
| GET | `/api/v1/supplies/{supplyId}/trbx` | Грузоместа в поставке |
| POST | `/api/v1/supplies` | Создать поставку |
| PATCH | `/api/v1/supplies/{supplyId}/close` | Закрыть поставку |
| DELETE | `/api/v1/supplies/{supplyId}` | Удалить поставку |

### Пример получения коэффициентов приёмки

```javascript
// Коэффициент = стоимость приёмки. Чем ниже — тем выгоднее
const coefficients = await wbApi(
  'https://supplies-api.wildberries.ru',
  '/api/v1/acceptance/coefficients'
);
// Возвращает: warehouseID, warehouseName, coefficient, date
```

---

## Заказы FBS

> Токен: **Маркетплейс** | FBS = "Fulfillment by Seller" (склад продавца)

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v3/orders` | Список сборочных заданий |
| GET | `/api/v3/orders/new` | Новые заказы |
| POST | `/api/v3/orders/stickers` | Этикетки для заказов |
| PUT | `/api/v3/orders/{orderId}/deliver` | Передать в доставку |
| PATCH | `/api/v3/orders/{orderId}/cancel` | Отменить заказ |
| GET | `/api/v3/supplies` | Список поставок FBS |
| POST | `/api/v3/supplies` | Создать поставку FBS |
| PATCH | `/api/v3/supplies/{supplyId}/orders/{orderId}` | Добавить заказ в поставку |
| GET | `/api/v3/passes/offices` | Список ПВЗ для пропусков |

---

## Заказы DBW / DBS / Самовывоз

| Схема | Описание |
|-------|----------|
| **DBW** | Delivery by Wildberries — доставка силами WB от склада продавца |
| **DBS** | Delivery by Seller — доставка силами продавца |
| **Самовывоз** | Покупатель забирает сам |

Для каждой схемы — аналогичный набор методов для получения сборочных заданий, обновления статусов и маркировки.

---

## Аналитика и данные

> Токен: **Аналитика**

### Воронка продаж

Статистика по карточкам: показы, переходы, корзина, заказы.

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| POST | `/api/v2/nm-report/detail` | Детальная статистика по артикулам |
| POST | `/api/v2/nm-report/detail/history` | История по артикулам (до года, подписка Джем) |
| POST | `/api/v2/nm-report/grouped` | Сгруппированная статистика |
| POST | `/api/v2/nm-report/grouped/history` | История сгруппированной статистики |

**Параметры агрегации:**
- `day` — по дням (по умолчанию)
- `week` — по неделям

**Пример запроса:**

```javascript
const stats = await wbApi(
  'https://seller-analytics-api.wildberries.ru',
  '/api/v2/nm-report/detail/history',
  {
    selectedPeriod: {
      begin: '2025-01-01 00:00:00',
      end: '2025-12-31 23:59:59'
    },
    nmIDs: [123456789, 987654321],
    aggregationLevel: 'week'
  },
  'POST'
);
```

**Лимиты (воронка):**

| Тип токена | Лимит |
|-----------|-------|
| Персональный / Сервисный | 3 запроса/мин |
| Базовый | 2 запроса/час |

### Поисковые запросы по товарам

```
GET /api/v1/analytics/keyword-stats
```

Показывает, по каким поисковым запросам покупатели находили ваши товары.

### История остатков

```
GET /api/v1/analytics/warehouse-remains
```

История изменения остатков на складах WB.

### Оценка товара

```
POST /api/v1/analytics/goods-realization
```

Метрики реализации товаров: продажи, возвраты, доля.

---

## Отчёты

> Токен: **Статистика**

### Основные отчёты

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v5/supplier/reportDetailByPeriod` | Детализированный отчёт о продажах |
| GET | `/api/v1/supplier/sales` | Продажи |
| GET | `/api/v1/supplier/orders` | Заказы |
| GET | `/api/v1/supplier/stocks` | Остатки |
| GET | `/api/v1/supplier/incomes` | Поставки (приходы) |

### Специальные отчёты

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/analytics/acceptance-report` | Операции при приёмке |
| GET | `/api/v1/analytics/antifraud-details` | Удержания |
| GET | `/api/v1/paid-storage` | Платное хранение |
| GET | `/api/v1/analytics/sales-region` | Продажи по регионам |
| GET | `/api/v1/analytics/brand-share` | Доля бренда в продажах |
| GET | `/api/v1/analytics/excise-report` | Товары с обязательной маркировкой |
| GET | `/api/v1/analytics/goods-return` | Возвраты и перемещение товаров |

### Пример получения продаж за период

```javascript
const sales = await wbApi(
  'https://statistics-api.wildberries.ru',
  '/api/v1/supplier/sales',
  {
    dateFrom: '2025-01-01',
    flag: 0  // 0 = все, 1 = только новые с последнего запроса
  }
);
```

> ⚠️ Отчёты по статистике могут содержать большой объём данных. Используйте параметр `dateFrom` для ограничения периода.

---

## Маркетинг и продвижение

> Токен: **Реклама**

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/adv/v1/promotion/count` | Количество кампаний |
| GET | `/adv/v1/promotion/adverts` | Список кампаний |
| GET | `/adv/v2/promotion/adverts` | Список кампаний v2 |
| POST | `/adv/v1/promotion/count` | Кампании по статусу |
| GET | `/adv/v1/stat/all` | Статистика по всем кампаниям |
| GET | `/adv/v2/fullstat` | Полная статистика кампании |
| POST | `/adv/v0/start` | Запустить кампанию |
| POST | `/adv/v0/pause` | Поставить кампанию на паузу |
| POST | `/adv/v0/stop` | Остановить кампанию |
| POST | `/adv/v1/cpm` | Установить ставку CPM |
| GET | `/adv/v1/auctions` | Аукционы (ставки конкурентов) |
| GET | `/adv/v1/calendar/promo` | Календарь акций WB |

---

## Общение с покупателями

> Токен: **Отзывы**

### Отзывы

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/feedbacks` | Список отзывов |
| PATCH | `/api/v1/feedbacks` | Ответить на отзыв |
| GET | `/api/v1/feedbacks/count/unanswered` | Количество отзывов без ответа |
| GET | `/api/v1/feedbacks/products/rating/top` | Топ товаров по рейтингу |
| POST | `/api/v1/feedbacks/actions/report` | Пожаловаться на отзыв |

### Вопросы

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/questions` | Список вопросов |
| PATCH | `/api/v1/questions` | Ответить на вопрос |
| GET | `/api/v1/questions/count/unanswered` | Количество вопросов без ответа |

### Чаты

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/chats` | Список чатов |
| GET | `/api/v1/chats/messages` | Сообщения чата |
| POST | `/api/v1/chats/messages` | Отправить сообщение |

### Закреплённые отзывы

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/feedbacks/pinned` | Список закреплённых отзывов |
| PUT | `/api/v1/feedbacks/pin` | Закрепить отзыв |
| DELETE | `/api/v1/feedbacks/pin` | Открепить отзыв |

---

## Финансы и документы

> Токен для финансов отдельный — **Документы**

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/tariffs/box` | Тарифы на доставку коробов |
| GET | `/api/v1/tariffs/pallet` | Тарифы на доставку палет |
| GET | `/api/v1/tariffs/return` | Тарифы на возврат |
| GET | `/api/v1/accounting/balance` | Баланс счёта продавца |
| GET | `/api/v1/documents/financial-report` | Финансовый отчёт |
| GET | `/api/v1/documents/edo` | Документы ЭДО |

---

## Тарифы

> Токен: **Маркетплейс**

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/v1/tariffs/box` | Стоимость доставки коробов на склады WB |
| GET | `/api/v1/tariffs/pallet` | Стоимость доставки палет |
| GET | `/api/v1/tariffs/return` | Стоимость возврата |
| GET | `/api/v1/tariffs/transfer` | Стоимость перемещения |

---

## Лимиты запросов

WB использует алгоритм **token bucket** для равномерного распределения нагрузки.

### Категория Маркетплейс (заказы, склады, остатки)

| Тип токена | Лимит | Интервал | Всплеск |
|-----------|-------|----------|---------|
| Персональный | 300 зап/мин | 200 мс | 20 |
| Сервисный | 300 зап/мин | 200 мс | 20 |
| Базовый | 150 зап/мин | 200 мс | 10 |
| Тестовый | 150 зап/мин | 200 мс | 10 |

### Категория Контент (товары, карточки)

| Период | Лимит | Интервал | Всплеск |
|--------|-------|----------|---------|
| 1 мин | 100 запросов | 600 мс | 5 |

### Категория Аналитика (воронка продаж)

| Тип токена | Лимит |
|-----------|-------|
| Персональный / Сервисный | 3 запроса/мин |
| Базовый | 2 запроса/час |

> ⚠️ При превышении лимита возвращается ошибка **429 Too Many Requests**.

---

## Ошибки

| Код | Описание | Что делать |
|-----|----------|-----------|
| `400` | Неправильный запрос | Проверить тело/параметры запроса |
| `401` | Не авторизован | Проверить токен |
| `403` | Доступ запрещён | Проверить категорию токена |
| `404` | Метод не найден | Проверить URL и версию API |
| `429` | Превышен лимит | Добавить паузы, уменьшить частоту |
| `500` | Ошибка сервера | Повторить позже |

**Формат ошибки:**
```json
{
  "title": "path not found",
  "detail": "Please consult the https://dev.wildberries.ru/openapi/api-information",
  "status": 404,
  "statusText": "Not Found",
  "timestamp": "2025-04-24T07:25:28Z"
}
```

> Обращайте внимание на поле `detail` в ошибках 404 и 429 — там часто есть полезная подсказка.

---

## Полезные паттерны

### Retry при ошибке 429

```javascript
async function wbApiWithRetry(baseUrl, endpoint, params, method = 'GET', retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await wbApi(baseUrl, endpoint, params, method);
    } catch (e) {
      if (e.response?.status === 429 && i < retries - 1) {
        await new Promise(r => setTimeout(r, 2000 * (i + 1))); // 2s, 4s, 6s
        continue;
      }
      throw e;
    }
  }
}
```

### Пагинация через cursor

```javascript
async function fetchAllOrders(warehouseId) {
  let cursor = undefined;
  const orders = [];

  while (true) {
    const res = await wbApi(
      'https://marketplace-api.wildberries.ru',
      '/api/v3/orders',
      { limit: 1000, next: cursor }
    );
    orders.push(...(res.orders || []));

    if (!res.cursor?.hasMore) break;
    cursor = res.cursor.id;
  }

  return orders;
}
```

### Получить остатки на всех складах WB

```javascript
async function getWbStocks() {
  // 1. Получить список всех складов WB
  const warehouses = await wbApi(
    'https://supplies-api.wildberries.ru',
    '/api/v1/warehouses'
  );

  // 2. Получить коэффициенты приёмки
  const coefficients = await wbApi(
    'https://supplies-api.wildberries.ru',
    '/api/v1/acceptance/coefficients'
  );

  return { warehouses, coefficients };
}
```

### Получить все продажи за период (статистика)

```javascript
async function getSalesByPeriod(dateFrom) {
  const sales = await wbApi(
    'https://statistics-api.wildberries.ru',
    '/api/v1/supplier/sales',
    { dateFrom, flag: 0 }
  );
  return sales;
}
// dateFrom формат: '2025-01-01'
```

### Ответить на все непрочитанные отзывы

```javascript
async function replyToFeedbacks(replyText) {
  const feedbacks = await wbApi(
    'https://feedbacks-api.wildberries.ru',
    '/api/v1/feedbacks',
    { isAnswered: false, take: 100, skip: 0 }
  );

  for (const fb of feedbacks.data?.feedbacks || []) {
    await wbApi(
      'https://feedbacks-api.wildberries.ru',
      '/api/v1/feedbacks',
      { id: fb.id, text: replyText },
      'PATCH'
    );
    await new Promise(r => setTimeout(r, 300)); // пауза между запросами
  }
}
```

### Аналитика воронки продаж по месяцам

```javascript
async function getFunnelByMonth(nmIDs, year) {
  const data = await wbApi(
    'https://seller-analytics-api.wildberries.ru',
    '/api/v2/nm-report/detail/history',
    {
      selectedPeriod: {
        begin: `${year}-01-01 00:00:00`,
        end: `${year}-12-31 23:59:59`
      },
      nmIDs,
      aggregationLevel: 'week' // 'day' или 'week'
    },
    'POST'
  );
  return data;
}
```

---

## Особенности WB API vs Ozon API

| | Wildberries | Ozon |
|--|------------|------|
| Базовый URL | Несколько доменов по категориям | Один домен `api-seller.ozon.ru` |
| Авторизация | Bearer токен в заголовке | Client-Id + Api-Key |
| Аналитика | Воронка + история + поисковые запросы | `/v1/analytics/data` с метриками |
| Поставки | FBW (склад WB) | FBO (склад Ozon) |
| Склад продавца | FBS | FBS |
| Лимиты | Token bucket, разные по категориям | Фиксированные лимиты по эндпоинтам |
| Документация | dev.wildberries.ru | docs.ozon.ru/api/seller |
| Sandbox | Есть (dev.wildberries.ru/sandbox) | Есть |

---

## Полезные ссылки

- 📖 [Документация WB API](https://dev.wildberries.ru/)
- 🔧 [Swagger](https://dev.wildberries.ru/swagger)
- 🧪 [Песочница](https://dev.wildberries.ru/sandbox)
- 📊 [Статус API](https://dev.wildberries.ru/wb-status)
- 💬 [Сообщество разработчиков](https://dev.wildberries.ru/forum)
- 📚 [База знаний](https://dev.wildberries.ru/knowledge-base)
- 📰 [Журнал изменений](https://dev.wildberries.ru/release-notes)
- 🏢 [Личный кабинет продавца](https://seller.wildberries.ru/)
