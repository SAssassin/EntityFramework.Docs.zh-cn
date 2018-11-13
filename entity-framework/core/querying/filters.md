---
title: 全局查询过滤器 - EF Core
author: anpete
ms.date: 11/03/2017
uid: core/querying/filters
ms.openlocfilehash: 73efe62262cf45cc1841d7a86cf59249cf07c5ea
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42996660"
---
# <a name="global-query-filters"></a>全局查询过滤器

全局查询过滤器是应用于元数据模型（通常为 *OnModelCreating*）中的实体类型的 LINQ 查询谓词（通常传递给 LINQ *Where* 查询操作的布尔表达式）。 此类过滤器自动应用于所涉及实体类型（包括间接引用的实体类型，比如使用 Include 或直接由导航属性引用的实体类型）的所有 LINQ 查询。 此功能的一些常见应用如下：

* **软删除** - 定义了“IsDeleted”属性的实体类型。
* **多租户** - 定义了“TenantId”属性的实体类型。

## <a name="example"></a>示例

下面的示例展示了在一个简单的博客模型中如何使用全局查询过滤器实现软删除和多租户查询。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFrameworkCore/tree/master/samples/QueryFilters)。

首先，定义实体：

[!code-csharp[Main](../../../efcore-repo/samples/QueryFilters/Program.cs#Entities)]

请注意 _Blog_ 实体上 __tenantId_ 字段的声明。 这会用于将每个 _Blog_ 实例与特定租户相关联。 同时在 _Post_ 实体类型上定义了 _IsDeleted_ 属性。 这会用于跟踪一个 _Post_ 实例是否已“软删除”。 也就是说，实例只是被标记为已删除，而非真正删除了基础数据。

接下来，使用 ```HasQueryFilter``` API 在 _OnModelCreating_ 中配置查询过滤器。

[!code-csharp[Main](../../../efcore-repo/samples/QueryFilters/Program.cs#Configuration)]

传递给 _HasQueryFilter_ 调用的谓词表达式将立即自动应用于这些类型的任何 LINQ 查询。

> [!TIP]
> 请注意 DbContext 实例级别字段的使用：```_tenantId``` 用于设置当前租户。 模型级过滤器将使用正确的上下文实例（即执行查询的实例）中的值。

## <a name="disabling-filters"></a>禁用过滤器

可使用 ```IgnoreQueryFilters()``` 对各个 LINQ 查询禁用过滤器。

[!code-csharp[Main](../../../efcore-repo/samples/QueryFilters/Program.cs#IgnoreFilters)]

## <a name="limitations"></a>限制

全局查询过滤器具有以下限制：

* 过滤器不能包含对导航属性的引用。
* 仅可为继承层次结构中的根实体类型定义过滤器。
