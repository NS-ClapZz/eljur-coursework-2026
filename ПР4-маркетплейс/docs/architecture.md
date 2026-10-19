# Архитектура ShopSystem

## Почему приложение разделено на проекты

Решение разделено на доменную модель, сценарии приложения, инфраструктуру, API и WPF-клиент. Для практической работы API использует EF Core и SQLite; отдельная консольная версия из исходного учебного проекта сохранена.

```text
ShopSystem.Desktop ──HTTP──> ShopSystem.Api ──> SQLite
ShopSystem.ConsoleApp ──> ShopSystem.Application ──> ShopSystem.Core
          │                         ▲
          └──> ShopSystem.Infrastructure ──────────┘
```

- `ShopSystem.Core` содержит сущности каталога и заказов. Он не зависит от других проектов решения.
- `ShopSystem.Application` содержит сценарии работы с товарами и заказами, а также интерфейс репозитория. Он зависит только от `Core`.
- `ShopSystem.Infrastructure` реализует хранение данных в памяти. Его можно заменить реализацией для базы данных, не меняя сервисы приложения.
- `ShopSystem.ConsoleApp` является composition root: создаёт реализации зависимостей и отвечает за взаимодействие с пользователем.
- `ShopSystem.Application.Tests` проверяет сценарии приложения независимо от консоли и реальной инфраструктуры.
- `ShopSystem.Api` предоставляет каталог, учётные записи, корзину, заказы и административные операции. Роли проверяются авторизационной политикой, пароль сохраняется как хэш.
- `ShopSystem.Desktop` — WPF-клиент каталога, регистрации/входа, корзины и оформления заказа.
- SQLite-модель разделяет пользователей, категории, товары, ПВЗ, строки корзины, заказы и строки заказа. В строке заказа хранится снимок цены и названия на дату покупки.

Для такого размера решения дальнейшее дробление на отдельные проекты `Catalog`, `Orders` и другие было бы избыточным. Внутри проектов эти области уже выражены папками и пространствами имён.

## Распределение исходных файлов

| Исходный файл | Новое расположение | Решение |
|---|---|---|
| `Program.cs` | `src/ShopSystem.ConsoleApp/Program.cs` | Точка входа |
| `Product.cs` | `src/ShopSystem.Core/Catalog/Product.cs` | Модель каталога |
| `Order.cs` | `src/ShopSystem.Core/Orders/Order.cs` | Модель заказа |
| `ProductService.cs` | `src/ShopSystem.Application/Catalog/ProductService.cs` | Сценарии каталога |
| `OrderService.cs` | `src/ShopSystem.Application/Orders/OrderService.cs` | Сценарии заказов |
| `Database.cs` | `src/ShopSystem.Infrastructure/Persistence/InMemoryShopRepository.cs` | Реализация хранилища вынесена за интерфейс |
| `Tests.cs` | `tests/ShopSystem.Application.Tests/*Tests.cs` | Переписан как автоматические xUnit-тесты |
| `appsettings.json` | `src/ShopSystem.ConsoleApp/appsettings.example.json` | Секреты удалены, оставлен безопасный шаблон |
| `architecture.png` | `docs/architecture.png` | Архитектурная документация |
| `notes.txt` | `docs/development-notes.md` | Заметки преобразованы в документацию |

