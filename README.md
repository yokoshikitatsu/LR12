#  Лабораторная работа 12: Paging

**Тема:** Реализация пагинации данных, Paging Library, работа с большими списками данных.

---

##  Цели работы

- Настроить Paging 3 в проекте (`paging-runtime`, `paging-compose`)
- Реализовать `PagingSource` для загрузки страниц через API из ЛР_11
- Собрать поток данных с помощью `Pager` и отобразить список в Compose через `LazyPagingItems`
- Обработать состояния загрузки (`refresh`, `append`): индикаторы и повтор при ошибке

---

##  Теоретическая часть

### Зачем нужна пагинация

Если загружать весь список с сервера сразу (например, тысячи записей), приложение долго ждёт ответа, тратит много памяти и может завершиться с ошибкой. Пагинация — загрузка данных порциями (страницами): сначала показываем 20 элементов, при прокрутке вниз подгружаем следующие 20. 

**Paging 3** — библиотека Jetpack, которая управляет загрузкой страниц и отдаёт данные в UI (в том числе в Compose через `LazyColumn`).

### Основные понятия Paging 3

| Компонент | Описание |
|-----------|----------|
| **PagingSource** | Класс, который умеет загрузить одну страницу: по ключу (например, номер страницы) выполняет запрос и возвращает `LoadResult.Page` (данные + ключи предыдущей/следующей страницы) или `LoadResult.Error`. Paging 3 сам вызывает `load()`, когда пользователь приближается к концу списка. |
| **Pager** | По конфигу (размер страницы) и фабрике `PagingSource` строит поток данных `Flow<PagingData<T>>`. |
| **PagingData / LazyPagingItems** | В Compose мы подписываемся на поток через `collectAsLazyPagingItems()` и получаем объект с `itemCount`, доступом по индексу и состоянием загрузки (`refresh`, `append`). Список рисуем в `LazyColumn` с помощью `items()`. |

---
##  Выполнение заданий

### Задание 2.1. Зависимости Paging

Добавлены зависимости в `app/build.gradle.kts`:

<img width="456" height="47" alt="image" src="https://github.com/user-attachments/assets/64eccc93-398d-4052-a9ca-9acdd79828ed" />


## Задание 2.2. PagingSource для API

Создан класс `PostPagingSource`, наследующий `PagingSource<Int, Post>`. В методе `load()` по `params.key` (номер страницы) вызывается API из ЛР_11: `api.getPosts(page = page, limit = params.loadSize)`. Ответ — `List<Post>`; оборачивается в `LoadResult.Page`.

**Файл:** `data/remote/PostPagingSource.kt`

<img width="848" height="679" alt="image" src="https://github.com/user-attachments/assets/42028fcf-8765-471d-b0e9-bc8bf24c39f2" />

## Задание 2.3. Класс с потоком PagingData (без ViewModel)

Создан объект `PostPagingProvider`, который предоставляет метод, возвращающий `Flow<PagingData<Post>>`. Внутри — `Pager` с `PagingConfig` и фабрикой `PostPagingSource`. Этот `Flow` собирается в Composable.

**Файл:** `data/PostPagingProvider.kt`
<img width="526" height="401" alt="image" src="https://github.com/user-attachments/assets/feb1cbdf-5914-412c-bdee-d413dedd941b" />

**Зачем:** UI не должен знать про PagingSource и Pager — только подписывается на поток и отображает элементы.

---

## Задание 2.4. Отображение в Compose и сбор потока

В `Composable` вызывается `PostPagingProvider.getPostsFlow().collectAsLazyPagingItems()` и рисуется список постов в `LazyColumn`.

**Файл:** `ui/screens/PostListPagingScreen.kt`
<img width="808" height="889" alt="image" src="https://github.com/user-attachments/assets/932fad22-5875-4d56-a7ef-35f265d17d27" />
<img width="779" height="791" alt="image" src="https://github.com/user-attachments/assets/fd8a15cc-7b51-4e73-9226-f3187e5e0682" />

**Зачем:** `collectAsLazyPagingItems()` подписывается на поток и при появлении новых страниц обновляет список; при этом элементы по индексу могут быть временно `null` (ещё не загружены) — показывается индикатор. Ключ `key` помогает Compose корректно перерисовывать только изменившиеся элементы.

---

## Задание 2.5. Состояния загрузки и ошибок

У `LazyPagingItems` есть свойство `loadState` (`refresh`, `append`, `prepend`):

- В начале списка обрабатывается `loadState.refresh`: при `Loading` показывается полноэкранный индикатор, при `Error` — сообщение и кнопка «Повторить» (вызов `posts.retry()`).
- В конце списка при `loadState.append is Loading` показывается индикатор под списком, при `append is Error` — сообщение и повторить.

---
##  Скриншоты и демонстрация

### Логи постраничной загрузки
<img width="1725" height="228" alt="image" src="https://github.com/user-attachments/assets/ff5640d8-b65c-4455-a0ad-e3a8c6ac0eae" />
---

### Демонстрации
<img width="393" height="869" alt="image" src="https://github.com/user-attachments/assets/39909624-cddb-47b5-99cf-27c4b63af593" />

<img width="395" height="870" alt="image" src="https://github.com/user-attachments/assets/22eaa921-cbdb-4628-acdd-e45a8fba0ac9" />

