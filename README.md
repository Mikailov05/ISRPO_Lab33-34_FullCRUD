
# NotesApp - Полноценный CRUD с базой данных

**Студент:** Микаилов Ахмед

**Лабораторная работа №33-34**

## Описание проекта

NotesApp — это RESTful API для управления заметками с категориями. Приложение позволяет создавать, читать, обновлять и удалять заметки, а также управлять категориями, к которым они относятся.

### Основные возможности:

- Полноценный CRUD для категорий и заметок
- Фильтрация, поиск, сортировка и пагинация заметок
- Валидация входных данных через Data Annotations
- Единый формат ответов API
- Связь один-ко-многим между категориями и заметками

### Технологии:

- ASP.NET Core Web API
- Entity Framework Core
- SQLite
- Swagger / OpenAPI

## Структура проекта

```
NotesApp/
├── Controllers/
│   ├── CategoriesController.cs
│   └── NotesController.cs
├── Data/
│   └── AppDbContext.cs
├── Helpers/
│   └── ApiResponse.cs
├── Models/
│   ├── Category.cs
│   ├── Note.cs
│   └── DTOs/
│       ├── CategoryDtos.cs
│       ├── NoteFilterDto.cs
│       └── NoteDto.cs
├── Repositories/
│   ├── ICategoryRepository.cs
│   ├── CategoryRepository.cs
│   ├── INoteRepository.cs
│   └── NoteRepository.cs
├── Program.cs
├── appsettings.json
└── requests.http
```

## Маршруты API

### Категории

| Метод | URL | Описание | Коды ответа |
|---|---|---|---|
| GET | /api/categories | Все категории с кол-вом заметок | 200 |
| GET | /api/categories/{id} | Одна категория | 200, 404 |
| GET | /api/categories/{id}/notes | Категория с заметками | 200, 404 |
| POST | /api/categories | Создать категорию | 201, 400 |
| PUT | /api/categories/{id} | Обновить категорию | 200, 400, 404 |
| DELETE | /api/categories/{id} | Удалить категорию | 204, 400, 404 |

### Заметки

| Метод | URL | Описание | Коды ответа |
|---|---|---|---|
| GET | /api/notes | Все заметки с фильтрами | 200 |
| GET | /api/notes/{id} | Одна заметка | 200, 404 |
| POST | /api/notes | Создать заметку | 201, 400 |
| PUT | /api/notes/{id} | Обновить заметку | 200, 400, 404 |
| PATCH | /api/notes/{id}/pin | Закрепить / открепить | 200, 404 |
| PATCH | /api/notes/{id}/archive | Архивировать / восстановить | 200, 404 |
| DELETE | /api/notes/{id} | Удалить заметку | 204, 404 |

### Параметры фильтрации для GET /api/notes

| Параметр | Тип | Описание | Значение по умолчанию |
|---|---|---|---|
| categoryId | int? | Фильтр по категории | null |
| isPinned | bool? | Только закреплённые | null |
| archived | bool | Включать архивные | false |
| search | string? | Поиск по заголовку и содержимому | null |
| minPriority | int? | Минимальный приоритет (1-5) | null |
| sortBy | string | Поле сортировки | "createdAt" |
| descending | bool | Направление сортировки | true |
| page | int | Номер страницы | 1 |
| pageSize | int | Размер страницы | 10 |

## Паттерн Repository

### Что такое Repository?

Repository — это паттерн проектирования, который изолирует логику работы с данными от логики контроллера.

### Архитектура без Repository:

```
Контроллер → напрямую работает с DbContext → База данных
```

### Архитектура с Repository:

```
Контроллер → Repository → DbContext → База данных
```

### Зачем нужен Repository?

| Проблема без Repository | Решение с Repository |
|---|---|
| Логика запросов к БД размазана по контроллерам | Все запросы в одном месте |
| Трудно тестировать контроллер отдельно от БД | Можно подменить репозиторий на тестовый |
| При смене БД нужно менять все контроллеры | Меняется только репозиторий |
| Дублирование одинаковых запросов | Переиспользуемые методы |

### Пример из реального мира

Представьте склад (база данных) и кладовщика (Repository). Продавцы (контроллеры) не ходят на склад сами — они отправляют запрос кладовщику. Кладовщик знает, где всё лежит и как правильно работать со складом.

### Реализация в проекте

```csharp
// Интерфейс — контракт, который описывает, что умеет репозиторий
public interface ICategoryRepository
{
    Task<IEnumerable<CategoryResponseDto>> GetAllAsync();
    Task<Category?> GetByIdAsync(int id);
    Task<Category> CreateAsync(Category category);
    // ...
}

// Реализация — конкретная работа с базой данных
public class CategoryRepository : ICategoryRepository
{
    private readonly AppDbContext _db;
    
    public async Task<Category?> GetByIdAsync(int id) =>
        await _db.Categories.FindAsync(id);
    // ...
}
```

### Преимущества в нашем проекте

1. **Тестируемость** — можно заменить реальный репозиторий на mock-объект
2. **Поддержка** — вся логика запросов в одном месте
3. **Переиспользование** — методы вроде `ExistsAsync()` используются в нескольких контроллерах
4. **Разделение ответственности** — контроллеры занимаются только HTTP-логикой (маршруты, статусы), а не построением SQL-запросов
```