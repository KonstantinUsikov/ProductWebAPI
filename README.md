# ProductWebAPI

REST Web-сервис на ASP.NET Core для управления базой данных из трёх таблиц: **Supplier** (поставщики), **Category** (категории) и **Product** (товары).
Поддерживает основные операции: `GET`, `POST`, `PUT`, `PATCH` (JSON Patch), `DELETE`.
API документирован с помощью Swashbuckle (Swagger UI).

## Технологии

- .NET 7 (`net7.0`), ASP.NET Core Web API (контроллеры)
- Entity Framework Core 7 + SQL Server
- Newtonsoft.Json (в том числе для `JsonPatchDocument`)
- Swashbuckle.AspNetCore 6.5: Swagger UI, собранный из XML-комментариев к контроллерам

## Структура проекта

```
ProductWebAPI.sln
WebAPI/
├── Controllers/
│   ├── CategoriesController.cs               # /api/categories
│   ├── ProductsController.cs                 # /api/products
│   └── SuppliersController.cs                # /api/suppliers
├── Models/
│   ├── Category.cs, Product.cs, Supplier.cs  # сущности EF Core
│   ├── *Request.cs, *Response.cs             # DTO для запросов и ответов
│   ├── ProductContext.cs                     # контекст базы данных
│   └── SeedData.cs                           # заполнение БД тестовыми данными
├── Migrations/                               # миграции EF Core
├── Properties/launchSettings.json            # профили запуска
├── appsettings.json                          # строка подключения к БД
└── Program.cs                                # настройка сервисов, Swagger и маршрутов
```

## Модель данных

| Сущность | Поле | Тип | Ограничения |
|---|---|---|---|
| **Supplier** | `id` | int | первичный ключ |
| | `name` | string | обязательное |
| | `city` | string | обязательное |
| **Category** | `id` | int | первичный ключ |
| | `name` | string | обязательное |
| **Product** | `id` | int | первичный ключ |
| | `name` | string | обязательное |
| | `price` | decimal(8, 2) | от 1 до 100000 |
| | `supplierId` | int | внешний ключ на `Supplier`, ≥ 1 |
| | `categoryId` | int | внешний ключ на `Category`, ≥ 1 |

При удалении поставщика или категории связанные с ними товары удаляются каскадно.

## Запуск

### Требования

- [.NET 7 SDK](https://dotnet.microsoft.com/download/dotnet/7.0)
- SQL Server LocalDB (устанавливается вместе с Visual Studio) или любой другой экземпляр SQL Server

### Настройка базы данных

Строка подключения задаётся в `WebAPI/appsettings.json`:

```json
"ConnectionStrings": {
  "ProductConnection": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=ProductDB"
}
```

Чтобы подключиться к другому серверу, измените это значение.

### Запуск приложения

```bash
git clone <url-репозитория>
cd ProductWebAPI
dotnet run --project WebAPI
```

Сервис будет доступен по адресу `http://localhost:5000`.

В окружении `Development` (используется по умолчанию в `launchSettings.json`) при старте приложения:
1. автоматически применяются миграции, так что вручную выполнять `dotnet ef database update` не нужно;
2. если таблицы пустые, они заполняются тестовыми данными: 3 категории, 5 поставщиков и 5 товаров.

## Документация API

- Swagger UI: http://localhost:5000/swagger
- Спецификация OpenAPI: http://localhost:5000/swagger/v1/swagger.json

## Эндпоинты

### Поставщики: `/api/suppliers`

| Метод | Маршрут | Описание |
|---|---|---|
| `GET` | `/api/suppliers` | Список всех поставщиков |
| `GET` | `/api/suppliers/{id}` | Поставщик по id |
| `POST` | `/api/suppliers` | Создать поставщика |
| `PUT` | `/api/suppliers` | Обновить поставщика (id передаётся в теле запроса) |
| `PATCH` | `/api/suppliers/{id}` | Частично обновить поставщика (JSON Patch) |
| `DELETE` | `/api/suppliers/{id}` | Удалить поставщика |

### Категории: `/api/categories`

| Метод | Маршрут | Описание |
|---|---|---|
| `GET` | `/api/categories` | Список всех категорий |
| `GET` | `/api/categories/{id}` | Категория по id |
| `POST` | `/api/categories` | Создать категорию |
| `PUT` | `/api/categories` | Обновить категорию (id передаётся в теле запроса) |
| `PATCH` | `/api/categories/{id}` | Частично обновить категорию (JSON Patch) |
| `DELETE` | `/api/categories/{id}` | Удалить категорию |

### Товары: `/api/products`

| Метод | Маршрут | Описание |
|---|---|---|
| `GET` | `/api/products` | Список всех товаров |
| `GET` | `/api/products/{id}` | Товар по id |
| `GET` | `/api/products/category/{id}` | Товары указанной категории |
| `GET` | `/api/products/supplier/{id}` | Товары указанного поставщика |
| `POST` | `/api/products` | Создать товар |
| `PUT` | `/api/products` | Обновить товар (id передаётся в теле запроса) |
| `PATCH` | `/api/products/{id}` | Частично обновить товар (JSON Patch) |
| `DELETE` | `/api/products/{id}` | Удалить товар |

### Коды ответов

- `200`: запрос выполнен успешно. `POST`, `PUT`, `PATCH` и `DELETE` возвращают созданный, изменённый или удалённый объект.
- `400`: некорректный id или тело запроса не прошло валидацию.
- `404`: объект с указанным id не найден. Для товаров этот код возвращается и тогда, когда не найден указанный поставщик или категория.
- `500`: некорректное тело `PATCH`-запроса.

## Примеры запросов

Создать товар:

```bash
curl -X POST http://localhost:5000/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Видеокарта Asus GeForce GT 1030", "price": 1.00, "supplierId": 1, "categoryId": 1}'
```

Ответ:

```json
{
  "id": 21,
  "name": "Видеокарта Asus GeForce GT 1030",
  "price": 1.00,
  "supplierId": 1,
  "categoryId": 1
}
```

Обновить поставщика целиком:

```bash
curl -X PUT http://localhost:5000/api/suppliers \
  -H "Content-Type: application/json" \
  -d '{"id": 2, "name": "newName", "city": "newCity"}'
```

Частично обновить товар (JSON Patch):

```bash
curl -X PATCH http://localhost:5000/api/products/2 \
  -H "Content-Type: application/json-patch+json" \
  -d '[{"op": "replace", "path": "name", "value": "newProductName"}, {"op": "replace", "path": "price", "value": "888"}]'
```

Удалить категорию:

```bash
curl -X DELETE http://localhost:5000/api/categories/2
```
