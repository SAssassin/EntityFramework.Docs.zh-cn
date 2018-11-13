---
title: 未连接的实体 - EF Core
author: ajcvickers
ms.author: avickers
ms.date: 10/27/2016
ms.assetid: 2533b195-d357-4056-b0e0-8698971bc3b0
uid: core/saving/disconnected-entities
ms.openlocfilehash: 51367d2619b1943c300f8954123f70b909ad96e7
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994394"
---
# <a name="disconnected-entities"></a>未连接的实体

DbContext 实例将自动跟踪从数据库返回的实体。 调用 SaveChanges() 方法时，系统能够检测到对这些实体所做的更改并根据需要更新数据库。 有关详细信息，请参阅[基本保存](basic.md)和[关联数据](related-data.md)。

但是，有时实体是用一个上下文实例查询的，然后使用另一个上下文实例来保存。这通常在“未连接”的情况下发生，比如 Web 应用程序，在一个客户端请求中由服务器端查询实体，发送到客户端，经客户端修改，再由另一个请求发送回服务器端保存。

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Disconnected/)。

> [!TIP]
> EF Core 只能跟踪具有给定主键值的任何实体的一个实例。 避免使其成为问题的最佳方法是为每个工作单元使用短生存期的上下文，使上下文一开始是空的、向其附加实体、保存这些实体，然后释放并丢弃上下文。

## <a name="identifying-new-entities"></a>标识新实体

### <a name="client-identifies-new-entities"></a>客户端标识新实体

由客户端告知服务器端实体是新增的还是已有的，最简单的情况下只需处理“在何时告知”。

本文的其余部分介绍了需要以其他某种方式确定是插入还是更新的情况。

### <a name="with-auto-generated-keys"></a>使用自动生成的键

自动生成的键的值通常用于确定是需要插入实体还是需要更新实体。 如果未设置键（即其值仍为 CLR 默认值 null、零等），则实体必定是新的，需要插入。 另一方面，如果已设置键的值，则必定是之前已保存的实体，现在需要更新。 换言之，如果键具有值，则是先前查询出来，而后发送到客户端，现在返回以进行更新的实体。

实体类型已知时，可以轻松检查未设置的键：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#IsItNewSimple)]

但是，EF 还有一种适用于任何实体类型和键类型的内置方法可用于检查键的设置状况：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#IsItNewGeneral)]

> [!TIP]  
> 即使实体处于“Added”状态，一旦由上下文跟踪，就会设置键。 这有助于遍历实体（关系）图并确定对每个实体做什么，如使用 TrackGraph API 时。 在执行任何调用以跟踪实体 _之前_，应仅按照此处显示的方式使用键值。

### <a name="with-other-keys"></a>使用其他键

未自动生成键值时，需要使用其他某种机制来确定新实体。 有以下两种常规方法：
 * 查询实体
 * 从客户端传递标志

若要查询实体，只需使用 Find 方法：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#IsItNewQuery)]

这超出了本文档的范围，无法展示用于从客户端传递标志的完整代码。 在 Web 应用中，通常表示为不同操作发出不同请求，或传递请求中的某些状态，然后在控制器中进行提取。

## <a name="saving-single-entities"></a>保存单个实体

如果知道是需要插入还是需要更新，则可以相应地使用 Add 或 Update：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#InsertAndUpdateSingleEntity)]

但是，如果实体使用自动生成的键值，则 Update 方法可以用于以下两种情况：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#InsertOrUpdateSingleEntity)]

Update 方法通常将实体标记为更新，而不是插入。 但是，如果实体具有自动生成的键且未设置任何键值，则实体会自动标记为插入。

> [!TIP]  
> EF Core 2.0 中已引入此行为。 对于早期版本，始终需要显式选择 Add 或 Update。

如果实体不使用自动生成的键，则应用程序必须确定是应插入实体还是应更新实体：例如：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#InsertOrUpdateSingleEntityWithFind)]

步骤如下：
* 如果 Find 返回 null，则数据库尚未包含具有此 ID 的博客，因此对 Add 的调用会将其标记为插入。
* 如果 Find 返回实体，则该实体存在于数据库中，且上下文现在正在跟踪现有实体
  * 然后，使用 SetValues 将此实体上所有属性的值设置为来自客户端的值。
  * SetValues 调用根据需要会将实体标记为进行更新。

> [!TIP]  
> SetValues 仅将被跟踪实体中发生变化的属性标记为已修改。 这意味着当发送更新到数据库时，只会更新实际发生更改的列。 （如果未发生更改，则完全不会发送任何更新。）

## <a name="working-with-graphs"></a>使用实体图

### <a name="identity-resolution"></a>标识解析

如上所述，EF Core 只能跟踪具有给定主键值的任何实体的一个实例。 使用实体图时，理想情况下应创建实体图，以便保留此固定对象，且上下文应仅用于一个工作单元。 如果实体图中包含重复项，则必须先处理该实体图，然后再将其发送到 EF 以合并多个实例。 如果实例值和关系发生冲突，则以上操作可能并不轻松，因此应尽快在应用程序管道中完成重复项合并以避免解析冲突。

### <a name="all-newall-existing-entities"></a>所有新实体/所有现有实体

对于实体图使用的一个示例是插入或更新博客及其关联文章集合。 如果所有实体应插入实体图，或应更新所有这些实体，则该过程与上述单个实体的过程相同。 例如，博客和文章的实体图创建如下：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#CreateBlogAndPosts)]

可按如下方式插入：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#InsertGraph)]

对 Add 的调用会将博客和所有文章标记为插入。

同样，如果实体图中的所有实体都需要更新，则可以使用 Update：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#UpdateGraph)]

博客及其所有文章将标记为更新。

### <a name="mix-of-new-and-existing-entities"></a>新实体和现有实体的组合

即使实体图包含需要插入的实体和需要更新的实体的组合，通过自动生成的键，Update 也可同时用于插入和更新：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#InsertOrUpdateGraph)]

Update 会将实体图中，如博客或文章，尚未设置键值的任何实体标记为插入，而其他所有实体会标记为更新。

如前所述，不使用自动生成的键时，需要查询和一些处理过程，如下：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#InsertOrUpdateGraphWithFind)]

## <a name="handling-deletes"></a>处理删除

由于实体不存在通常表示应删除该实体，因此删除可能很难处理。 解决此问题的一种方法是使用“软删除”，以便将该实体标记为已删除，而不是实际删除。 然后，删除将变得与更新相同。 可以使用[查询筛选器](xref:core/querying/filters)实现软删除。

对于真删除，常见模式是通过查询的扩展来演示实体图真正不同的部分。例如:

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#InsertUpdateOrDeleteGraphWithFind)]

## <a name="trackgraph"></a>TrackGraph

在内部，Add、Attach 和 Update 使用实体图遍历，为每个实体就是否应将其标记为 Added（若要插入）、Modified（若要更新）、Unchanged（不执行任何操作）或 Deleted（若要删除）作出决定。 此机制是通过 TrackGraph API 公开的。 例如，假设当客户端发送回实体实体图时，会在每个实体上设置某些标志，指示应如何处理实体。 然后，TrackGraph 可用于处理此标志：

[!code-csharp[Main](../../../samples/core/Saving/Saving/Disconnected/Sample.cs#TrackGraph)]

为了简化示例，标志仅作为实体的一部分显示。 通常，标志是 DTO 或包含在请求中的其他某个状态的一部分。
