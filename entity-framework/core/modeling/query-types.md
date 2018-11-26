---
title: 查询类型 - EF Core
author: anpete
ms.date: 02/26/2018
ms.assetid: 9F4450C5-1A3F-4BB6-AC19-9FAC64292AAD
uid: core/modeling/query-types
ms.openlocfilehash: 3328082dbc62aa80eb5fb29d2e57df1eef248d1f
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489487"
---
# <a name="query-types"></a>查询类型
> [!NOTE]
> 此功能是 EF Core 2.1 中的新增功能

除了实体类型以外， EF Core 模型可以包含 _查询类型_， 这可用于实现针对 *没有映射到实体类型的数据* 的数据库查询。

## <a name="compare-query-types-to-entity-types"></a>比较查询类型与实体类型

查询类型与实体类型的相同之处：

- 可以在 `OnModelCreating` 或通过派生 _DbContext_ 上的 "set" 属性添加到模型。
- 支持许多相同的映射功能，如继承映射和导航属性。 在关系型存储种，他们可以通过 fluent API 方法或数据注释配置目标数据库对象和列。

查询类型与实体类型的不同之处：

- 不需要定义键（Key，唯一标识）。
- 永远不会被 _DbContext_ 进行更改跟踪，因此永远不会在数据库种插入、 更新或删除数据。
- 永远不会由约定发现。
- 仅支持一部分导航映射功能-具体而言：
  - 它们可能永远不会作为关系的主体端。
  - 它们仅可包含指向的实体的引用导航属性。
  - 实体不能包含查询类型的导航属性。
- 在 _ModelBuilder_ 中通过 `Query` 方法而不是 `Entity` 方法使用（进行配置）。
- 通过 _DbContext_ 类型的 `DbQuery<T>` 而非 `DbSet<T>` 属性进行映射。
- 通过 `ToView` 方法，而非 `ToTable` 映射到数据库对象。
- 可以映射到 _定义查询_ - 定义查询是在模型中声明的第二次查询可作为查询类型的数据源。

## <a name="usage-scenarios"></a>使用场景

下面是一些查询类型主要使用场景：

- 作为即席`FromSql()`查询的返回类型。
- 映射到数据库视图。
- 映射到没有定义主键的表。
- 映射到模型中定义的查询。

## <a name="mapping-to-database-objects"></a>映射到数据库对象

查询类型映射到数据库对象是通过 `ToView` fluent API 来实现的。 从 EF Core 的角度来看，此方法中指定的数据库对象是 _视图_，这意味着它将被视为只读查询源且不能作为更新、 插入或删除操作的目标。 但是，这并不意味着该数据库对象必须是实际的数据库视图 - 它可以是被视为只读的一个数据库表。 相反，实体类型，EF Core 假定 `ToTable` 方法中指定的数据库对象被视为 _表_、 意味着它不仅可以用作查询源，还可以更新、删除和插入。 事实上，只要数据库上的视图被配置为可更新， 在 `ToTable` 中指定数据库视图名称，所有内容理应能够正常工作。

## <a name="example"></a>示例

下面的示例演示如何使用查询类型来查询数据库视图。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFrameworkCore/tree/master/samples/QueryTypes)。

首先，我们定义一个简单的博客和文章模型：

[!code-csharp[Main](../../../efcore-repo/samples/QueryTypes/Program.cs#Entities)]

接下来，我们定义一个简单的数据库视图，这样就可以查询每个博客下的帖子数：

[!code-csharp[Main](../../../efcore-repo/samples/QueryTypes/Program.cs#View)]

接下来，我们定义一个类来保存数据库视图的结果：

[!code-csharp[Main](../../../efcore-repo/samples/QueryTypes/Program.cs#QueryType)]

接下来，我们在 _OnModelCreating_ 中使用 `modelBuilder.Query<T>` API配置查询类型。
我们使用标准的 fluent 配置 Api 来配置查询类型的映射：

[!code-csharp[Main](../../../efcore-repo/samples/QueryTypes/Program.cs#Configuration)]

最后，我们可以采用标准方式来查询数据库视图：

[!code-csharp[Main](../../../efcore-repo/samples/QueryTypes/Program.cs#Query)]

> [!TIP]
> 请注意我们还定义了上下文级别查询属性 (DbQuery) 作为针对此类型查询的根。
