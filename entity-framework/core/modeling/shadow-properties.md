---
title: 隐藏属性-EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 75369266-d2b9-4416-b118-ed238f81f599
uid: core/modeling/shadow-properties
ms.openlocfilehash: b7b7b10642564dfa3dbc05755188b5b5c63e0d03
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993797"
---
# <a name="shadow-properties"></a>隐藏属性

隐藏属性是指在 .NET 实体类中未定义但在 EF Core 模型中有定义的属性。 这些属性的值和状态是完全通过更改跟踪器来维护的。

数据库中若有数据不应被所映射的实体类型公开时，隐藏属性非常有用。 它们通常用于外键属性，其中两个实体之间的关系在数据库中通过外键的值来表示，但此关系在实体类型之间是用导航属性来管理的。

隐藏属性的值可以通过 `ChangeTracker` API 来获取或更改。

``` csharp
   context.Entry(myBlog).Property("LastUpdated").CurrentValue = DateTime.Now;
```

可以通过 LINQ 查询中的静态方法 `EF.Property` 来引用隐藏属性。

``` csharp
var blogs = context.Blogs
    .OrderBy(b => EF.Property<DateTime>(b, "LastUpdated"));
```

## <a name="conventions"></a>约定

当发现一种关系，但在依赖实体类中没找到对应的外键属性时，可以通过约定创建隐藏属性。 在这种情况下，将引入隐藏的外键属性。 隐藏的外键属性将被命名为 `<导航属性名称><主体实体的键属性名称>` （用于命名依赖实体中指向主体实体的导航属性）。 如果**主体实体的键属性名称**包含**导航属性名称**，则名称只是 `<主体实体的键属性名称>` 。 如果依赖实体上没有任何导航属性，则用**主体类型名称**代替（在命名方式中**导航属性名称**的位置）。

例如，以下代码会将隐藏属性 `BlogId` 引入到 `Post` 实体。

<!-- [!code-csharp[Main](samples/core/Modeling/Conventions/Samples/ShadowForeignKey.cs)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public Blog Blog { get; set; }
}
```

## <a name="data-annotations"></a>数据注释

隐藏属性**不能**使用数据注释来创建。

## <a name="fluent-api"></a>Fluent API

可以使用 Fluent API 配置隐藏属性。 一旦调用了字符串重载的 `Property` 方法，就可以继续链式调用其他配置方法，和普通属性一样。

如果提供给 `Property` 方法的名称与现有属性 （现有隐藏属性或一个在实体类中定义的非隐藏属性） 的名称相匹配，则代码将优先配置现有属性，而不是引入新的隐藏属性。

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/ShadowProperty.cs?highlight=7,8)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .Property<DateTime>("LastUpdated");
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}
```
