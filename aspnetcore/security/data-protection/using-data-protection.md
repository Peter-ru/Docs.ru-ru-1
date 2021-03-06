---
title: "Приступая к работе с API защиты данных"
author: rick-anderson
description: 
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 39b7a73c-29d4-4137-b311-49529adcf3cb
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/using-data-protection
ms.openlocfilehash: 2a381acf5faa7071fe4a22641037fcee11f6d362
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/11/2017
---
# <a name="getting-started-with-the-data-protection-apis"></a>Приступая к работе с API защиты данных

<a name=security-data-protection-getting-started></a>

В его простейшей защиту данных состоит из следующих шагов:

1. Создание данных предохранитель из поставщик защиты данных.

2. Вызовите метод защитить с данными, которые необходимо защитить.

3. Вызовите метод снять защиту с данными, которые вы хотите преобразовать обратно в обычный текст.

Большинство платформ, таких как ASP.NET или SignalR уже настройки системы защиты данных и добавить его в контейнер службы, который доступен через внедрения зависимостей. Следующий пример демонстрирует настройку контейнер службы для внедрения зависимостей и регистрации стека защиты данных, получения поставщик защиты данных через DI, создания предохранителя и защиту и снятие защиты данных

[!code-csharp[Main](../../security/data-protection/using-data-protection/samples/protectunprotect.cs?highlight=26,34,35,36,37,38,39,40)]

При создании предохранителя необходимо указать один или несколько [строки цели](consumer-apis/purpose-strings.md). Строка цели обеспечивает изоляцию между потребителей, например предохранителя, созданные с целью строку «зеленый» не сможет снять защиту данных, предоставленных предохранитель с целью «фиолетовый».

>[!TIP]
> Экземпляры IDataProtectionProvider и IDataProtector являются потокобезопасными для нескольких клиентов. Предполагается, что после компонент получает ссылку на IDataProtector через вызов CreateProtector, он будет использовать эту ссылку для несколько вызовов Protect и Unprotect.
>
>Вызов Unprotect вызовет CryptographicException, если защищенный полезных данных не может быть проверена или расшифровать. Некоторые компоненты нужна возможность пропускать ошибки во время снятия защиты операций; компонент, который считывает файлы cookie проверки подлинности может обработать эту ошибку и обрабатывать запрос, как если бы объекты cookie не на всех, а не ошибкой запроса сразу. В частности, компонентами, которые хотите такого поведения следует перехватывать CryptographicException вместо подавление всех исключений.
