---
title: 'Ejemplo de script de CLI de Azure: detalles sobre Azure Redis Cache | Microsoft Docs'
description: 'Ejemplo de script de CLI de Azure: detalles sobre Azure Redis Cache'
services: redis-cache
documentationcenter: 
author: steved0x
manager: douge
editor: 
tags: azure-service-management
ms.assetid: 155924e6-00d5-4a8c-ba99-5189f300464a
ms.service: cache-redis
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 08/30/2017
ms.author: sdanie
ms.openlocfilehash: 0927508117bda67b62a188f89a2d2e151fe02a5a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/11/2017
---
# <a name="get-details-of-an-azure-redis-cache"></a>Obtenga detalles sobre una instancia de Azure Redis Cache

En este escenario, obtendrá información sobre cómo recuperar los detalles de una instancia de Azure Redis Cache, incluido su estado de aprovisionamiento.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/redis-cache/show-cache/show-cache.sh "Azure Redis Cache")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los comandos siguientes para recuperar los detalles de una instancia de Azure Redis Cache. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Comando | Notas |
|---|---|
| [az redis show](https://docs.microsoft.com/cli/azure/redis#az_redis_show) | Recupera detalles de una instancia de Azure Redis Cache. |


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure/overview).

Encontrará más ejemplos de scripts de CLI de Azure Redis Cache en la [documentación de Azure Redis Cache](../cli-samples.md).