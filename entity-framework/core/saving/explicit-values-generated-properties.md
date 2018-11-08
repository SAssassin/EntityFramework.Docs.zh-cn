---
title: 设置已生成属性的显式值 - EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 3f1993c2-cdf5-425b-bac2-a2665a20322b
uid: core/saving/explicit-values-generated-properties
ms.openlocfilehash: 00abef4d1208400ff68ced0a241b98b8dc9be5c0
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997848"
---
# <a name="setting-explicit-values-for-generated-properties"></a>设置生成属性显式值

生成的属性是在添加或者更新实体时（由 EF 或数据库生成）生成其值的属性。 有关详细信息，请参阅[生成的属性](../modeling/generated-properties.md)。

可能会出现希望为生成属性设置显式值而非使用其生成值的情况。

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/ExplicitValuesGenerateProperties/)。

## <a name="the-model"></a>模型

本文中使用的模型包含单个 `Employee` 实体。

[!code-csharp[Main](../../../samples/core/Saving/Saving/ExplicitValuesGenerateProperties/Employee.cs#Sample)]

## <a name="saving-an-explicit-value-during-add"></a>在添加期间保存显式值

`Employee.EmploymentStarted` 属性配置为由数据库为新实体生成值（使用默认值）。

[!code-csharp[Main](../../../samples/core/Saving/Saving/ExplicitValuesGenerateProperties/EmployeeContext.cs#EmploymentStarted)]

以下代码可将两个员工插入到数据库中。
* 对于第一个员工，没有为 `Employee.EmploymentStarted` 属性分配任何值，因此它将被设置为 `DateTime` 的 CLR 默认值。
* 对于第二个员工，我们已设置显式值为 `1-Jan-2000` 。

[!code-csharp[Main](../../../samples/core/Saving/Saving/ExplicitValuesGenerateProperties/Sample.cs#EmploymentStarted)]

输出显示了数据库已为第一个员工生成值，且显式值已用于第二个员工。

``` Console
1: John Doe, 1/26/2017 12:00:00 AM
2: Jane Doe, 1/1/2000 12:00:00 AM
```

### <a name="explicit-values-into-sql-server-identity-columns"></a>显式值用于 SQL Server IDENTITY 列

按照约定，`Employee.EmployeeId` 属性是存储生成的 `IDENTITY` 列。

对于大多数情况，上述方法将适用于键属性。 但是，若要将显式值插入到 SQL Server `IDENTITY` 列中，则必须在调用 `SaveChanges()` 之前手动启用 `IDENTITY_INSERT`。

> [!NOTE]  
> 我们在积压工作中有一个[功能请求](https://github.com/aspnet/EntityFramework/issues/703)，用来在 SQL Server 提供程序内自动执行此操作。

[!code-csharp[Main](../../../samples/core/Saving/Saving/ExplicitValuesGenerateProperties/Sample.cs#EmployeeId)]

输出显示了提供的 ID 已保存到数据库。

``` Console
100: John Doe
101: Jane Doe
```

## <a name="setting-an-explicit-value-during-update"></a>在更新期间设置显式值

`Employee.LastPayRaise` 属性配置为在更新期间由数据库生成值。

[!code-csharp[Main](../../../samples/core/Saving/Saving/ExplicitValuesGenerateProperties/EmployeeContext.cs#LastPayRaise)]

> [!NOTE]  
> 默认情况下，在更新期间如果你为生成属性指定显式值，EF Core 将引发异常。 若要避免此问题，必须对`AfterSaveBehavior`设置为较低级别的 `metadata API` （如上所示）。

> [!NOTE]  
> **EF Core 2.0 中的更改：** 在以前版本中，通过 `IsReadOnlyAfterSave` 标志控制保存后行为。 此标志已过时，将替换为 `AfterSaveBehavior`。

数据库中还存在触发器，以便在执行 `UPDATE` 操作期间为 `LastPayRaise` 列生成值。

[!code-sql[Main](../../../samples/core/Saving/Saving/ExplicitValuesGenerateProperties/employee_UPDATE.sql)]

以下代码用于增加数据库中两个员工的薪资。
* 对于第一个员工，没有为 `Employee.LastPayRaise` 属性分配任何值，因此仍将设置为 null。
* 对于第二个员工，设置显式值为一周前（使加薪在较早的日期开始生效）。

[!code-csharp[Main](../../../samples/core/Saving/Saving/ExplicitValuesGenerateProperties/Sample.cs#LastPayRaise)]

输出显示了数据库已为第一个员工生成值，且显式值已用于第二个员工。

``` Console
1: John Doe, 1/26/2017 12:00:00 AM
2: Jane Doe, 1/19/2017 12:00:00 AM
```
