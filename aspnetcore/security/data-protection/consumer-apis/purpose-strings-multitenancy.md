---
title: "Назначение строками в ASP.NET Core"
author: rick-anderson
description: 
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 9d18c287-e0e6-4541-b09c-7fed45c902d9
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/consumer-apis/purpose-strings-multitenancy
ms.openlocfilehash: dd87d8bcaf0056b322908e9a3ef75678f603e1e6
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/11/2017
---
# <a name="purpose-hierarchy-and-multi-tenancy-in-aspnet-core"></a>Цель иерархию и Многопользовательские приложения в ASP.NET Core

Поскольку IDataProtector неявно IDataProtectionProvider, можно создать цепочку целей. В этом смысле поставщика. CreateProtector ([«purpose1», «purpose2»]) является эквивалентом поставщика. CreateProtector("purpose1"). CreateProtector("purpose2").

Это обеспечивает некоторые интересные иерархические связи в системе защиты данных. В предыдущем примере для [Contoso.Messaging.SecureMessage](purpose-strings.md#data-protection-contoso-purpose), SecureMessage компонента можно вызвать поставщик. CreateProtector("Contoso.Messaging.SecureMessage") после переднего плана и результат в поле закрытого _myProvider кэша. Будущие предохранители можно создать путем вызова метода _myProvider.CreateProtector ("пользователь: имя пользователя»), и эти предохранители ключа, которые будут использоваться для защиты отдельных сообщений.

Это может также отражается. Рассмотрим один логический приложение какие узлы нескольких клиентов (CMS кажется разумным) и каждый клиент можно настроить свою собственную систему управления проверки подлинности и состояние. Зонтик приложение имеет один основной поставщик, и вызывает поставщика. CreateProtector («клиента 1») и поставщика. CreateProtector («клиента (2)») для предоставления каждого клиента собственную изолированной срез системы защиты данных. Клиенты затем создать собственные отдельных предохранители, исходя из своих собственных потребностей, но независимо от того, насколько сильно они пытаются выполнить их не удается создать предохранителей, которые конфликтуют с любым другим клиентом в системе. Графически она представлена ниже.

![Несколько целей аренды](purpose-strings-multitenancy/_static/purposes-multi-tenancy.png)

>[!WARNING]
> Это предполагает рамках элементов управления приложения, какие интерфейсы API доступны для отдельных клиентов и клиентов, которое не может выполнить произвольный код на сервере. Если клиент может выполняться произвольный код, они могут выполнять отражение закрытых прервать гарантии изоляции или может только читать материала основного непосредственно и наследовать все подразделы, что пожелает.

Система защиты данных использует своего рода Многопользовательские приложения в конфигурации по умолчанию out of box. По умолчанию материала основного хранится в папке профиля пользователя учетной записи процесса рабочего (или в реестре, для удостоверения пула приложений IIS). Но фактически довольно часто, использование одной учетной записи для выполнения нескольких приложений и таким образом эти приложения появятся общий доступ к хозяину материала. Чтобы устранить эту проблему, система защиты данных автоматически вставляет идентификатор уникальными на уровне приложений как первый элемент в цепочке общей цели. Этой цели неявное служит для [изолировать отдельные приложения](../configuration/overview.md#data-protection-configuration-per-app-isolation) друг от друга, эффективно рассматривая каждое приложение как уникальный клиента в системе и процесс создания предохранителя выглядит идентична на рисунке выше.
