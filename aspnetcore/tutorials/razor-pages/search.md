---
title: "Добавление поиска на страницы Razor ASP.NET Core"
author: rick-anderson
description: "Инструкции по добавлению поиска на страницы Razor ASP.NET Core"
keywords: "ASP.NET Core,поиск,страницы Razor"
ms.author: riande
manager: wpickett
ms.date: 08/07/2017
ms.topic: get-started-article
ms.technology: aspnet
ms.prod: asp.net-core
uid: tutorials/razor-pages/search
ms.openlocfilehash: f70f1e9b0e085f5aa90fcca499526588662c3cfd
ms.sourcegitcommit: ffac7e195bd7f99364f3aab45e491eeaf8f173b0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/12/2017
---
# <a name="adding-search-to-an-aspnet-core-mvc-app"></a>Добавление поиска в приложение ASP.NET Core MVC

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

В этом документе на страницу Index добавляется возможность поиска фильмов по *жанру* или *названию*.

Обновите метод `OnGetAsync` страницы Index, добавив следующий код:

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

В первой строке метода `OnGetAsync` создается запрос [LINQ](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/linq/) для выбора фильмов:

```csharp
 var movies = from m in _context.Movie
              select m;
```

Этот запрос определяется *только* в этой точке и **не** выполняется для базы данных.

Если параметр `searchString` содержит строку, запрос фильмов изменяется для фильтрации по строке поиска:

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

Код `s => s.Title.Contains()` представляет собой [лямбда-выражение](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions). Лямбда-выражения используются в запросах [LINQ](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/linq/) на основе методов в качестве аргументов стандартных методов операторов запроса, таких как метод [Where](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) или `Contains` (используется в предшествующем коде). Запросы LINQ не выполняются в том случае, если они определяются или изменяются путем вызова метода (например, `Where`, `Contains` или `OrderBy`). Вместо этого выполнение запроса откладывается. Это означает, что вычисление выражения откладывается до тех пор, пока не будет выполнена итерация его реализованного значения или не будет вызван метод `ToListAsync`. Дополнительные сведения см. в разделе [Выполнение запроса](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/ef/language-reference/query-execution).

**Примечание.** Метод [Contains](http://msdn.microsoft.com/library/bb155125.aspx) выполняется в базе данных, а не в коде C#. Регистр символов запроса учитывается в зависимости от параметров базы данных и сортировки. В SQL Server метод `Contains` сопоставляется с [SQL LIKE](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/like-transact-sql), в котором регистр символов не учитывается. В SQLite при параметрах сортировки по умолчанию регистр символов учитывается.

Перейдите на страницу Movies и добавьте строку запроса, такую как `?searchString=Ghost`, к URL-адресу (например, `http://localhost:5000/Movies?searchString=Ghost`). Отображаются отфильтрованные фильмы.

![Представление Index](search/_static/ghost.png)

Если на страницу Index добавлен следующий шаблон маршрута, строку поиска можно передать в виде сегмента URL-адреса (например, `http://localhost:5000/Movies/ghost`).

```cshtml
@page "{searchString?}"
```

Предыдущее ограничение маршрута разрешало поиск названия в виде данных маршрута (сегмент URL-адреса) вместо значения строки запроса.  Символ `?` в `"{searchString?}"` означает, что этот параметр является необязательным.

![Представление Index, в URL-адрес которого добавлено слово ghost, возвращает два фильма: Ghostbusters и Ghostbusters 2](search/_static/g2.png)

Тем не менее пользователи вряд ли будут изменять URL-адрес для поиска фильмов. На этом шаге добавляется пользовательский интерфейс для поиска фильмов. Если было добавлено ограничение маршрута `"{searchString?}"`, удалите его.

Откройте файл *Pages/Movies/Index.cshtml* и добавьте разметку `<form>`, которая выделена в следующем коде:

[!code-cshtml[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index2.cshtml?highlight=14-19&range=1-22)]

Тег HTML `<form>` использует [вспомогательную функцию тега Form](xref:mvc/views/working-with-forms#the-form-tag-helper). При отправке формы строка фильтра отправляется на страницу *Pages/Movies/Index*. Сохраните изменения и проверьте работу фильтра.

![Представление Index со словом ghost в текстовом поле фильтра по названию](search/_static/filter.png)

## <a name="search-by-genre"></a>Поиск по жанру

Добавьте следующие выделенные свойства в файл *Pages/Movies/Index.cshtml.cs*:

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=11-)]

Свойство `SelectList Genres` содержит список жанров. В этом списке пользователь может выбрать жанр фильма.

Свойство `MovieGenre` содержит конкретный жанр, выбранный пользователем, например "Western" (Вестерн).

Обновите метод `OnGetAsync`, используя следующий код:

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

Следующий код определяет запрос LINQ, который извлекает все жанры из базы данных.

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

Список жанров `SelectList` создается путем проецирования отдельных жанров.

<!-- BUG in OPS
Tag snippet_selectlist's start line '75' should be less than end line '29' when resolving "[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]"

There is no start line.

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]
-->

```csharp
Genres = new SelectList(await genreQuery.Distinct().ToListAsync());
```

### <a name="adding-search-by-genre"></a>Добавление поиска по жанру

Обновите файл *Index.cshtml* следующим образом:

[!code-cshtml[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

Проверьте работу приложения, выполнив поиск по жанру, по названию фильма и по обоим этим параметрам.

>[!div class="step-by-step"]
[Предыдущая статья — "Обновление страниц"](xref:tutorials/razor-pages/da1)
[Следующая статья — "Добавление нового поля"](xref:tutorials/razor-pages/new-field)