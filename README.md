# Руководство по использованию Wildberries API MCP сервера

## Содержание
1. [Введение](#введение)
2. [Установка и запуск](#установка-и-запуск)
3. [Доступные инструменты API](#доступные-инструменты-api)
4. [Примеры использования](#примеры-использования)
5. [Типичные сценарии использования](#типичные-сценарии-использования)
6. [Получение токена API](#получение-токена-api)
7. [Устранение неполадок](#устранение-неполадок)

## Введение

Wildberries API MCP сервер представляет собой промежуточный сервис, который упрощает взаимодействие с API Wildberries. Он предоставляет унифицированный интерфейс для доступа к данным аналитики, статистике продвижения и другой информации из Wildberries API.

MCP сервер выполняет следующие функции:
- Упрощает обращение к различным эндпоинтам API Wildberries
- Обрабатывает ошибки и ограничения частоты запросов
- Унифицирует формат ответов
- Обеспечивает централизованную аутентификацию

## Установка и запуск

### Необходимые предварительные требования
- Node.js (версия 14 или выше)
- npm или yarn
- Docker и Docker Compose (опционально, для контейнеризации)
- Токен API Wildberries с соответствующими разрешениями

### Способ 1: Прямая установка через Node.js

```bash
# Клонирование репозитория
git clone https://github.com/yourusername/wb-api-mcp-server.git
cd wb-api-mcp-server

# Установка зависимостей
npm install

# Запуск сервера
npm start
```

Сервер запустится на порту 3000 по умолчанию. Вы можете указать другой порт, установив переменную окружения `PORT`:

```bash
PORT=8080 npm start
```

### Способ 2: Использование Docker

```bash
# Создание Docker-образа
docker build -t wb-api-mcp-server .

# Запуск Docker-контейнера
docker run -p 3000:3000 -d --name wb-api-mcp wb-api-mcp-server
```

### Способ 3: Использование Docker Compose

```bash
# Запуск сервера с Docker Compose
docker-compose up -d

# Остановка сервера
docker-compose down
```

### Проверка установки

Вы можете проверить, что сервер работает правильно, отправив запрос к эндпоинту проверки работоспособности:

```bash
curl http://localhost:3000/health
```

Вы должны получить ответ, подобный следующему:

```json
{
  "status": "ok",
  "timestamp": "2023-05-21T12:34:56.789Z"
}
```

## Доступные инструменты API

MCP сервер предоставляет следующие группы эндпоинтов:

### 1. Статистика продвижения (Promotion Statistics)

- **POST /api/adv/fullstats** - Статистика рекламных кампаний
- **GET /api/adv/auto/stat-words** - Статистика автоматической кампании по кластерам ключевых фраз
- **GET /api/adv/stat/words** - Статистика кампаний по ключевым фразам
- **GET /api/adv/stats/keywords** - Статистика по ключевым словам для автоматических кампаний
- **POST /api/adv/stats** - Статистика медийных кампаний

### 2. Воронка продаж (Sales Funnel)

- **POST /api/nm-report/detail** - Получение статистики карточек товаров за период
- **POST /api/nm-report/detail/history** - Получение статистики карточек товаров по дням
- **POST /api/nm-report/grouped/history** - Получение статистики карточек товаров, сгруппированных по категориям, брендам и тегам

### 3. Поисковые запросы (Search Queries)

- **POST /api/search-report/report** - Получение данных основного отчета по поисковым запросам
- **POST /api/search-report/table/groups** - Получение пагинации по группам для поисковых запросов
- **POST /api/search-report/table/details** - Получение пагинации по товарам внутри группы
- **POST /api/search-report/product/search-texts** - Получение поисковых текстов по товару
- **POST /api/search-report/product/orders** - Получение заказов и позиций по поисковым текстам товара

### 4. Отчет по остаткам (Stocks Report)

- **POST /api/stocks-report/products/groups** - Получение данных по группам товаров для отчета по остаткам
- **POST /api/stocks-report/products/products** - Получение данных по товарам для отчета по остаткам
- **POST /api/stocks-report/products/sizes** - Получение данных по размерам для отчета по остаткам
- **POST /api/stocks-report/offices** - Получение данных по складам для отчета по остаткам

### 5. CSV-отчеты продавца (Seller Analytics CSV)

- **POST /api/nm-report/downloads** - Создание CSV-отчета
- **GET /api/nm-report/downloads** - Получение списка отчетов
- **POST /api/nm-report/downloads/retry** - Повторная генерация отчета
- **GET /api/nm-report/downloads/file/:downloadId** - Получение файла отчета

## Примеры использования

### Получение статистики рекламных кампаний

```javascript
// Использование fetch
const response = await fetch('http://localhost:3000/api/adv/fullstats', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'api-key': 'ВАШ_ТОКЕН_WILDBERRIES_API'
  },
  body: JSON.stringify([
    {
      "id": 8960367,
      "dates": [
        "2024-04-07",
        "2024-04-06"
      ]
    }
  ])
});

const data = await response.json();
console.log(data);
```

### Получение статистики карточек товаров

```javascript
// Использование axios
const axios = require('axios');

const response = await axios.post('http://localhost:3000/api/nm-report/detail', {
  "brandNames": ["ВашБренд"],
  "objectIDs": [358],
  "tagIDs": [123],
  "nmIDs": [1234567],
  "timezone": "Europe/Moscow",
  "period": {
    "begin": "2024-04-01 00:00:00",
    "end": "2024-04-15 23:59:59"
  },
  "orderBy": {
    "field": "ordersSumRub",
    "mode": "asc"
  },
  "page": 1
}, {
  headers: {
    'api-key': 'ВАШ_ТОКЕН_WILDBERRIES_API'
  }
});

console.log(response.data);
```

## Типичные сценарии использования

### 1. Мониторинг эффективности рекламных кампаний

**Сценарий:** Вы хотите регулярно отслеживать эффективность ваших рекламных кампаний и анализировать ключевые метрики.

**Решение с использованием MCP:**
1. Настройте ежедневную задачу, которая запрашивает статистику по всем активным кампаниям.
2. Сохраняйте полученные данные в базу данных для исторического анализа.
3. Создайте дашборд, отображающий ключевые метрики (CTR, конверсии, затраты).

**Пример кода:**
```javascript
// Получение статистики кампаний
const campaigns = [123456, 789012]; // ID ваших кампаний
const dates = [getDateString(new Date())]; // Сегодняшняя дата

// Формирование запроса
const requestData = campaigns.map(id => ({
  id: id,
  dates: dates
}));

// Отправка запроса к MCP серверу
const campaignStats = await fetchFromMcp('/api/adv/fullstats', 'POST', requestData);

// Сохранение данных и генерация отчета
saveToDatabaseAndGenerateReport(campaignStats);
```

### 2. Анализ воронки продаж товаров

**Сценарий:** Вы хотите анализировать, как пользователи взаимодействуют с вашими товарами от просмотра карточки до покупки.

**Решение с использованием MCP:**
1. Запрашивайте детальную статистику по товарам за выбранный период.
2. Анализируйте конверсии на каждом этапе (просмотр → добавление в корзину → заказ → выкуп).
3. Выявляйте товары с низкими конверсиями для оптимизации.

**Пример кода:**
```javascript
// Получение статистики воронки продаж
const response = await fetchFromMcp('/api/nm-report/detail', 'POST', {
  "nmIDs": [/* ваши номенклатуры */],
  "timezone": "Europe/Moscow",
  "period": {
    "begin": "2024-04-01 00:00:00",
    "end": "2024-04-30 23:59:59"
  },
  "page": 1
});

// Анализ конверсий
const products = response.data.cards;
const lowConversionProducts = products.filter(product => {
  const stats = product.statistics.selectedPeriod;
  return stats.conversions.addToCartPercent < 5 || 
         stats.conversions.cartToOrderPercent < 20 ||
         stats.conversions.buyoutsPercent < 80;
});

// Генерация отчета по проблемным товарам
generateLowConversionReport(lowConversionProducts);
```

### 3. Оптимизация поисковой видимости

**Сценарий:** Вы хотите улучшить видимость ваших товаров в поиске Wildberries.

**Решение с использованием MCP:**
1. Запрашивайте отчеты по поисковым запросам для своих товаров.
2. Анализируйте, по каким запросам ваши товары имеют хорошие позиции, а по каким - плохие.
3. Оптимизируйте карточки товаров для улучшения позиций.

**Пример кода:**
```javascript
// Получение отчета по поисковым запросам
const searchReport = await fetchFromMcp('/api/search-report/report', 'POST', {
  "currentPeriod": {
    "start": "2024-04-01",
    "end": "2024-04-30"
  },
  "positionCluster": "all",
  "orderBy": {
    "field": "avgPosition",
    "mode": "desc"
  },
  "limit": 100,
  "offset": 0
});

// Получение поисковых текстов для конкретного товара
const searchTexts = await fetchFromMcp('/api/search-report/product/search-texts', 'POST', {
  "currentPeriod": {
    "start": "2024-04-01",
    "end": "2024-04-30"
  },
  "nmIds": [1234567],
  "topOrderBy": "openCard",
  "limit": 20
});

// Анализ результатов и формирование рекомендаций
analyzeSearchPositionsAndGenerateRecommendations(searchTexts);
```

### 4. Управление запасами на основе аналитики

**Сценарий:** Вы хотите оптимизировать уровень запасов товаров на складах на основе данных о продажах.

**Решение с использованием MCP:**
1. Регулярно запрашивайте отчеты по остаткам и продажам.
2. Рассчитывайте оптимальный уровень запасов на основе скорости продаж.
3. Выявляйте товары с избыточными или недостаточными запасами.

**Пример кода:**
```javascript
// Получение отчета по остаткам
const stocksReport = await fetchFromMcp('/api/stocks-report/products/products', 'POST', {
  "nmIDs": [/* ваши номенклатуры */],
  "currentPeriod": {
    "start": "2024-04-01",
    "end": "2024-04-30"
  },
  "stockType": "",
  "skipDeletedNm": true,
  "orderBy": {
    "field": "avgOrders",
    "mode": "desc"
  },
  "offset": 0
});

// Анализ скорости продаж и остатков
const stockOptimizationReport = stocksReport.data.items.map(item => {
  const dailySales = item.metrics.avgOrders;
  const currentStock = item.metrics.stockCount;
  const daysOfSupply = currentStock / dailySales;
  
  return {
    nmId: item.nmID,
    name: item.name,
    dailySales,
    currentStock,
    daysOfSupply,
    stockStatus: daysOfSupply < 7 ? 'LOW' : daysOfSupply > 30 ? 'HIGH' : 'OPTIMAL'
  };
});

// Генерация рекомендаций по управлению запасами
generateStockManagementRecommendations(stockOptimizationReport);
```

### 5. Генерация и анализ расширенных CSV-отчетов

**Сценарий:** Вы хотите получить детальные данные для глубокого анализа в Excel или другом инструменте.

**Решение с использованием MCP:**
1. Создайте задачу на генерацию CSV-отчета через MCP.
2. Дождитесь завершения генерации и загрузите отчет.
3. Импортируйте данные в аналитические инструменты для анализа.

**Пример кода:**
```javascript
// Создание задачи на генерацию отчета
const reportId = generateUUID();
const createReportResponse = await fetchFromMcp('/api/nm-report/downloads', 'POST', {
  "id": reportId,
  "reportType": "DETAIL_HISTORY_REPORT",
  "userReportName": "Аналитика по товарам за апрель",
  "params": {
    "nmIDs": [/* ваши номенклатуры */],
    "startDate": "2024-04-01",
    "endDate": "2024-04-30",
    "timezone": "Europe/Moscow",
    "aggregationLevel": "day",
    "skipDeletedNm": false
  }
});

// Проверка статуса генерации (через некоторое время)
setTimeout(async () => {
  const reportStatusResponse = await fetchFromMcp('/api/nm-report/downloads', 'GET', {
    'filter[downloadIds]': [reportId]
  });
  
  const reportStatus = reportStatusResponse.data[0].status;
  
  if (reportStatus === 'SUCCESS') {
    // Загрузка отчета
    downloadReport(reportId);
  } else if (reportStatus === 'FAILED') {
    // Повторная попытка генерации
    retryReport(reportId);
  }
}, 60000); // Проверка через 1 минуту
```

## Получение токена API

Для работы с API Wildberries через MCP сервер вам потребуется токен API. Вот как его получить:

1. **Войдите в личный кабинет продавца Wildberries**

   Перейдите на [seller.wildberries.ru](https://seller.wildberries.ru/) и авторизуйтесь.

2. **Перейдите в раздел настроек API**

   После входа в систему перейдите в раздел "Настройки" (обычно доступен из меню или профиля).

3. **Перейдите в раздел управления API**

   Найдите раздел "API" или "Доступ к API" или "Интеграция".

4. **Создайте новый токен API**

   - Нажмите "Создать новый токен" или аналогичную кнопку
   - Выберите необходимые права доступа для токена:
     - Для MCP сервера WB API вам потребуются:
       - Разрешение категории **Аналитика** для воронки продаж и поисковых запросов
       - Разрешение категории **Продвижение** для статистики рекламы
   - Укажите имя для токена (для вашего удобства)
   - При необходимости установите срок действия (или оставьте постоянным)

5. **Сгенерируйте и сохраните токен**

   После заполнения необходимой информации нажмите "Сгенерировать" или "Создать" для генерации токена API.

   **ВАЖНО**: Обязательно скопируйте и надежно сохраните ваш токен! Полный токен будет показан только один раз в целях безопасности.

## Устранение неполадок

### Частые проблемы

1. **Отказ в соединении**: Убедитесь, что сервер запущен и порт доступен.
2. **Ошибки аутентификации**: Проверьте, что ваш токен API Wildberries действителен и имеет необходимые разрешения.
3. **Ограничение частоты запросов**: Сервер обрабатывает ограничения частоты запросов API Wildberries, но вам может потребоваться подождать, если вы превысили допустимое количество запросов.

### Просмотр логов

При запуске с Docker или Docker Compose логи хранятся в директории `logs`, которая подключена как том.

Для просмотра логов в работающем Docker-контейнере:

```bash
docker logs wb-api-mcp
```

### Коды ошибок

- **401** - Ошибка аутентификации (проверьте ваш токен API)
- **429** - Превышение лимита запросов (подождите некоторое время)
- **400** - Неверный запрос (проверьте параметры запроса)
- **403** - Доступ запрещен (проверьте разрешения вашего токена)
