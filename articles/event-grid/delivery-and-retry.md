---
title: Entrega y reintento de entrega de Azure Event Grid
description: "Describe cómo Azure Event Grid entrega eventos y cómo administra los mensajes no entregados."
services: event-grid
author: djrosanova
manager: timlt
ms.service: event-grid
ms.topic: article
ms.date: 01/10/2018
ms.author: darosa
ms.openlocfilehash: fe9089334deceb38186add56ce3fb1d6ecc20363
ms.sourcegitcommit: 71fa59e97b01b65f25bcae318d834358fea5224a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2018
---
# <a name="event-grid-message-delivery-and-retry"></a>Entrega y reintento de entrega de mensajes de Event Grid 

En este artículo se describe cómo Azure Event Grid administra los eventos cuando no se confirma la entrega.

Event Grid ofrece entrega duradera. Entrega cada mensaje por lo menos una vez en cada suscripción. Los eventos se envían inmediatamente al webhook registrado de cada suscripción. Si un webhook no acusa recibo de un evento en los 60 segundos siguientes al primer intento, Event Grid reintenta la entrega del evento.

## <a name="message-delivery-status"></a>Estado de entrega de mensajes

Event Grid usa códigos de respuesta HTTP para acusar recibo de eventos. 

### <a name="success-codes"></a>Códigos de éxito

Los siguientes códigos de respuesta HTTP response indican que un evento se ha entregado correctamente en el webhook. Event Grid considera la entrega finalizada.

- 200 OK
- 202 - Aceptado

### <a name="failure-codes"></a>Códigos de error

Los siguientes códigos de respuesta HTTP indican que el intento de entrega de un evento no se ha realizado. Event Grid intenta enviar de nuevo el evento. 

- 400 - Solicitud incorrecta
- 401 No autorizado
- 404 No encontrado
- 408 - Tiempo de espera de solicitud
- 414 - URI de solicitud demasiado largo
- Error de servidor interno 500
- Servicio no disponible 503
- Tiempo de espera de puerta de enlace 504

Cualquier otro código de respuesta o la falta de respuesta indica un error. Event Grid reintenta la entrega. 

## <a name="retry-intervals"></a>Intervalos de reintento

Event Grid usa una directiva de reintentos de retroceso exponencial para la entrega de eventos. Si el webhook no responde o devuelve un código de error, Event Grid reintenta la entrega según la siguiente programación:

1. 10 segundos
2. 30 segundos
3. 1 minuto
4. 5 minutos
5. 10 minutos
6. 30 minutos
7. 1 hora

Event Grid agrega una pequeña selección aleatoria a todos los intervalos de reintento.

## <a name="retry-duration"></a>Duración de reintentos

Durante la versión preliminar, Azure Event Grid expira todos los eventos que no se hayan entregado en dos horas.

## <a name="next-steps"></a>pasos siguientes

* Para ver el estado de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Para obtener una introducción a Event Grid, vea [Acerca de Event Grid](overview.md).
* Para comenzar a usar rápidamente Event Grid, vea [Creación y enrutamiento de eventos personalizados con Azure Event Grid](custom-event-quickstart.md).