---
title: 创建和删除 Api-EF Core
author: bricelam
ms.author: bricelam
ms.date: 11/10/2017
ms.openlocfilehash: 336f6fd655603a2474a58dfef377e121d9b04c3a
ms.sourcegitcommit: a088421ecac4f5dc5213208170490181ae2f5f0f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/08/2018
ms.locfileid: "51285634"
---
# <a name="create-and-drop-apis"></a>创建和删除 Api

EnsureCreated 和 EnsureDeleted 方法提供[迁移](migrations/index.md)的轻量级替代以管理数据库架构。 当数据是临时的或者数据库架构变化后可以丢弃数据时，这是非常有用的。 例如在原型制作时，测试，或本地缓存的情况下。

某些提供程序 （尤其是非关系的） 不支持迁移。 此时，EnsureCreated 通常是初始化数据库架构的最简单方法。

> [!WARNING]
> EnsureCreated 和迁移不能很好地协同工作。 如果您使用的迁移，则不要使用 EnsureCreated 来初始化架构。

从 EnsureCreated 到迁移的过渡不是无缝体验。 达成此目的的最简单方法是删除数据库，然后重新使用迁移创建该数据库。 如果你希望在将来使用迁移，最好一开始就使用迁移，而不是 EnsureCreated 。

## <a name="ensuredeleted"></a>EnsureDeleted

如果数据库存在，EnsureDeleted 方法将确保其被删除。 如果你没有相应权限，则将引发异常。

``` csharp
// Drop the database if it exists
dbContext.Database.EnsureDeleted();
```

## <a name="ensurecreated"></a>EnsureCreated

如果数据库不存在，EnsureCreated 将创建该数据库，并初始化该数据库的架构。 如果存在任何表 （包括另一个 DbContext 类的表），则不会进行架构初始化。

``` csharp
// Create the database if it doesn't exist
dbContext.Database.EnsureCreated();
```

> [!TIP]
> 此外提供了这些方法的异步版本。

## <a name="sql-script"></a>SQL 脚本

若要使用的 EnsureCreated SQL，您可以使用 GenerateCreateScript 方法。

``` csharp
var sql = dbContext.Database.GenerateCreateScript();
```

## <a name="multiple-dbcontext-classes"></a>多个 DbContext 类

在数据库中没有表时，才能使用 EnsureCreated。 如果有必要，也可以编写自己的检查以查看架构是否需要进行初始化，并可通过 IRelationalDatabaseCreator 服务来初始化架构。

``` csharp
// TODO: Check whether the schema needs to be initialized

// Initialize the schema for this DbContext
var databaseCreator = dbContext.GetService<IRelationalDatabaseCreator>();
databaseCreator.CreateTables();
```
