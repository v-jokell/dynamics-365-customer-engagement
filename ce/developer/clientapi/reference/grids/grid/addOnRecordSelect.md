---
title: "addOnRecordSelect (Client API reference) in Dynamics 365 for Customer Engagement| MicrosoftDocs"
ms.date: 11/10/2017
ms.service: crm-online
ms.topic: reference
applies_to: Dynamics 365 for Customer Engagement (online)
ms.assetid: 4bfd7f48-5db8-45ea-816f-9d590fbdf60d
author: KumarVivek
ms.author: kvivek
manager: amyla
search.audienceType: 
  - developer
search.app: 
  - D365CE
---
# addOnRecordSelect (Client API reference)

[!INCLUDE[./includes/addOnRecordSelect-description.md](./includes/addOnRecordSelect-description.md)]

## Grid types supported

Read-only and editable grids

## Syntax

`gridContext.getGrid().addOnRecordSelect(myFunction);`

|Name|Type|Required|Description|
|--|--|--|--|
|myFunction|function reference|Yes|The function to be executed when you select record (row) in a grid. The function will be added to the bottom of the event handler pipeline. The execution context is automatically passed as the first parameter to the function. See [execution context](../../../clientapi-execution-context.md) for more information.

## Remarks

To get the `gridContext`, see [Getting the grid context](../../grids.md#bkmk_gridcontext).

