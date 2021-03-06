---
title: 'Control de errores y excepciones: Azure Logic Apps | Microsoft Docs'
description: Patrones para el control de errores y excepciones en Azure Logic Apps
services: logic-apps
documentationcenter: .net,nodejs,java
author: jeffhollan
manager: anneta
editor: 
ms.assetid: e50ab2f2-1fdc-4d2a-be40-995a6cc5a0d4
ms.service: logic-apps
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 10/18/2016
ms.author: LADocs; jehollan
ms.openlocfilehash: 4eb6f743479886374692eadcf218b77b4bfcc933
ms.sourcegitcommit: 62eaa376437687de4ef2e325ac3d7e195d158f9f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/22/2017
---
# <a name="handle-errors-and-exceptions-in-azure-logic-apps"></a>Control de errores y excepciones en Azure Logic Apps

Azure Logic Apps proporciona completas herramientas y patrones para garantizar la solidez de sus integraciones y su resistencia frente a los errores. Cualquier arquitectura de integración presenta el reto de asegurarse de controlar correctamente el tiempo de inactividad o los problemas con sistemas dependientes. Logic Apps convierte el control de errores en una experiencia de primera clase al proporcionarle las herramientas que necesita para actuar ante las excepciones y los errores en los flujos de trabajo.

## <a name="retry-policies"></a>Directivas de reintentos

Una directiva de reintentos es el tipo más básico de control de errores y excepciones. Si una solicitud inicial ha agotado el tiempo de espera o ha producido error (toda solicitud que tenga como resultado una respuesta 429 o 5xx), esta directiva define si se debe reintentar la acción y cómo hacerlo. Existen tres tipos de directivas de reintentos: `exponential`, `fixed` y `none`. Si no se proporciona ninguna directiva de reintentos en la definición del flujo de trabajo, se utiliza la directiva predeterminada. Puede configurar directivas de reintentos en las **entradas** para una determinada acción o desencadenador, si se puede volver a reintentar. De forma similar, las directivas de reintentos de Diseñador de aplicaciones lógicas pueden configurarse (si procede) en la **configuración** de un determinado bloque.

Para obtener información acerca de las limitaciones de las directivas de reintentos, vea [Límites y configuración de Logic Apps](../logic-apps/logic-apps-limits-and-config.md) y, para obtener más información acerca de la sintaxis admitida, consulte la [sección acerca de la directiva de reintentos de Desencadenadores y acciones para flujos de trabajo][retryPolicyMSDN].

### <a name="exponential-interval"></a>Intervalo exponencial
El tipo de directiva `exponential` volverá a intentar una solicitud con error después de un intervalo de tiempo aleatorio desde un intervalo exponencialmente creciente. Se garantiza que cada reintento se enviará a un intervalo aleatorio mayor que **minimumInterval** y menor que **maximumInterval**. Se generará una variable aleatoria uniforme en el intervalo siguiente para cada reintento hasta, e incluido, **count**:
<table>
<tr><th> Rango de variable aleatoria </th></tr>
<tr><td>

| Número de reintentos | Intervalo mínimo | Intervalo mínimo |
| ------------ |  ------------ |  ------------ |
| 1 | Max(0, **minimumInterval**) | Min(interval, **maximumInterval**) |
| 2 | Max(interval, **minimumInterval**) | Min(2 * interval, **maximumInterval**) |
| 3 | Max(2*interval, **minimumInterval**) | Min(4 * interval, **maximumInterval**) |
| 4 | Max(4 * interval, **minimumInterval**) | Min(8 * interval, **maximumInterval**) |
| ... |

</td></tr></table>

Para las directivas de tipo `exponential`, son necesarios **interval** y **count**, mientras que **minimumInterval** y **maximumInterval** pueden proporcionarse opcionalmente para invalidar los valores predeterminados de PT5S y PT1D respectivamente.

| Nombre del elemento | Obligatorio | Tipo | Descripción |
| ------------ | -------- | ---- | ----------- |
| type | Sí | String | `exponential` |
| count | Sí | Entero | número de reintentos, debe estar comprendido entre 1 y 90  |
| interval | Sí | String | intervalo de reintentos en [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations); debe estar entre PT5S y PT1D |
| minimumInterval | No| String | intervalo mínimo de reintentos en [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations); debe estar entre PT5S y **interval** |
| maximumInterval | No| String | intervalo mínimo de reintentos en [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations); debe estar entre **interval** y PT1D |

### <a name="fixed-interval"></a>Intervalo fijo

El tipo de directiva `fixed` volverá a intentar una solicitud con error después de esperar el intervalo proporcionado antes de enviar la solicitud siguiente.

| Nombre del elemento | Obligatorio | Tipo | Descripción |
| ------------ | -------- | ---- | ----------- |
| type | Sí | String | `fixed`|
| count | Sí | Entero | número de reintentos; debe estar comprendido entre 1 y 90 |
| interval | Sí | String | intervalo de reintentos en [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations); debe estar entre PT5S y PT1D |

### <a name="none"></a>Ninguna
El tipo de directiva `none` no volverá a intentar una solicitud con error.

| Nombre del elemento | Obligatorio | Tipo | Descripción |
| ------------ | -------- | ---- | ----------- |
| type | Sí | String | `none`|

### <a name="default"></a>Valor predeterminado
Si no se especificó ninguna directiva de reintentos, se usará la directiva predeterminada. La directiva predeterminada es una directiva de intervalo exponencial que enviará hasta 4 reintentos, en intervalos que aumentan exponecialmente 7,5 segundos y que están limitados entre 5 y 45 segundos. Esta directiva predeterminada (que se usa cuando **retryPolicy** no se define) es equivalente a la directiva de esta definición de flujo de trabajo HTTP de ejemplo:

```json
"HTTP":
{
    "inputs": {
        "method": "GET",
        "uri": "http://myAPIendpoint/api/action",
        "retryPolicy" : {
            "type": "exponential",
            "count": 4,
            "interval": "PT7.5S",
            "minimumInterval": "PT5S",
            "maximumInterval": "PT45S"
        }
    },
    "runAfter": {},
    "type": "Http"
}
```

## <a name="catch-failures-with-the-runafter-property"></a>Detección de errores con la propiedad RunAfter

Cada acción de aplicación lógica declara qué acciones deben finalizar antes de que se inicie la acción, algo parecido a ordenar los pasos del flujo de trabajo. En la definición de la acción, esta ordenación se conoce como la propiedad `runAfter`. Esta propiedad es un objeto que describe qué acciones y estados de acciones ejecutan la acción. De forma predeterminada, todas las acciones que se agregan mediante el Diseñador de aplicación lógica se establecen en `runAfter` respecto al paso anterior, si el paso anterior era `Succeeded`. Sin embargo, este valor se puede personalizar para que active acciones cuando las acciones anteriores sean `Failed`, `Skipped` o un posible conjunto de estos valores. Si desea agregar un elemento a un tema de Service Bus designado después de que una acción específica `Insert_Row` produzca un error, podría usar la siguiente configuración `runAfter`:

```json
"Send_message": {
    "inputs": {
        "body": {
            "ContentData": "@{encodeBase64(body('Insert_Row'))}",
            "ContentType": "{ \"content-type\" : \"application/json\" }"
        },
        "host": {
            "api": {
                "runtimeUrl": "https://logic-apis-westus.azure-apim.net/apim/servicebus"
            },
            "connection": {
                "name": "@parameters('$connections')['servicebus']['connectionId']"
            }
        },
        "method": "post",
        "path": "/@{encodeURIComponent('failures')}/messages"
    },
    "runAfter": {
        "Insert_Row": [
            "Failed"
        ]
    }
}
```

Observe que la propiedad `runAfter` está establecida para desencadenarse si el estado de la acción `Insert_Row` es `Failed`. Para ejecutar la acción si el estado de la acción es `Succeeded`, `Failed` o `Skipped`, use esta sintaxis:

```json
"runAfter": {
        "Insert_Row": [
            "Failed", "Succeeded", "Skipped"
        ]
    }
```

> [!TIP]
> Las acciones que se ejecutan y completan correctamente después de que una acción anterior haya producido error se marcan como `Succeeded`. Este comportamiento significa que, si detecta correctamente todos los errores de un flujo de trabajo, la propia ejecución se marca como `Succeeded`.

## <a name="scopes-and-results-to-evaluate-actions"></a>Ámbitos y resultados para evaluar acciones

Del mismo modo que puede establecer la propiedad RunAfter para acciones individuales, también puede agrupar acciones dentro de un [ámbito](../logic-apps/logic-apps-loops-and-scopes.md), que actúa como agrupación lógica de acciones. Los ámbitos son útiles para organizar las acciones de aplicación lógica y para realizar evaluaciones agregadas sobre el estado de un ámbito. El ámbito en sí recibe un estado una vez que hayan terminado todas las acciones en un ámbito. El estado del ámbito se determina con los mismos criterios que una ejecución. Si la acción final en una rama de la ejecución es `Failed` o `Aborted`, el estado es `Failed`.

Para activar acciones específicas para los errores que se produjeran dentro del ámbito, puede usar `runAfter` con un ámbito que esté marcado como `Failed`. Si *cualquier* acción en el ámbito produce un error, la ejecución después de que un ámbito produzca un error permite crear una única acción para detectar errores.

### <a name="getting-the-context-of-failures-with-results"></a>Obtención del contexto de errores con resultados

Aunque la detección de errores desde un ámbito es útil, quizás también necesite el contexto como ayuda para comprender exactamente qué acciones han producido un error y los errores o códigos de estado que se hayan devuelto. La función del flujo de trabajo `@result()` proporciona contexto sobre el resultado de todas las acciones en un ámbito.

`@result()` toma un único parámetro, un nombre de ámbito, y devuelve una matriz de todos los resultados de acción desde dentro de ese ámbito. Estos objetos de acción incluyen los mismos atributos que el objeto `@actions()`, como la hora de inicio de la acción, la hora de finalización de la acción, el estado de la acción, las entradas de acción, los id. de correlación de acción y la salidas de acción. Para enviar el contexto de las acciones que produjeron un error dentro de un ámbito, puede emparejar fácilmente una función `@result()` con una propiedad `runAfter`.

Para ejecutar una acción *para cada una* de las acciones de un ámbito con el estado `Failed`, filtre la matriz de resultados para las acciones que produjeron un error; puede emparejar `@result()` con una acción **[Filtrar matriz](../connectors/connectors-native-query.md)** y un bucle **[ForEach](../logic-apps/logic-apps-loops-and-scopes.md)**. Puede tomar la matriz de resultados filtrada y realizar una acción para cada error mediante el bucle **ForEach** . En este ejemplo, seguido de una explicación detallada, se envía una solicitud HTTP POST con el cuerpo de respuesta de todas las acciones que produjeron un error dentro del ámbito `My_Scope`.

```json
"Filter_array": {
    "inputs": {
        "from": "@result('My_Scope')",
        "where": "@equals(item()['status'], 'Failed')"
    },
    "runAfter": {
        "My_Scope": [
            "Failed"
        ]
    },
    "type": "Query"
},
"For_each": {
    "actions": {
        "Log_Exception": {
            "inputs": {
                "body": "@item()['outputs']['body']",
                "method": "POST",
                "headers": {
                    "x-failed-action-name": "@item()['name']",
                    "x-failed-tracking-id": "@item()['clientTrackingId']"
                },
                "uri": "http://requestb.in/"
            },
            "runAfter": {},
            "type": "Http"
        }
    },
    "foreach": "@body('Filter_array')",
    "runAfter": {
        "Filter_array": [
            "Succeeded"
        ]
    },
    "type": "Foreach"
}
```

Este es un tutorial detallado que describe lo que sucede:

1. Para obtener el resultado de todas las acciones dentro de `My_Scope`, la acción **Filtrar matriz** filtra `@result('My_Scope')`.

2. La condición para **Filtrar matriz** es cualquier elemento `@result()` que tenga el estado `Failed`. Esta condición filtra la matriz con los resultados de todas las acciones de `My_Scope` a una matriz con solo los resultados de acción que hayan producido un error.

3. Realización de una acción **For Each** en las salidas **Filtered Array**. En este paso se realiza una acción *para cada* acción resultante en error que se filtrara antes.

    Si solo hay una acción en el ámbito que produjera error, las acciones en el elemento `foreach` solo se ejecutan una vez. 
    Muchas acciones con error causan una acción por cada error.

4. Envío de una solicitud HTTP POST en el cuero de respuesta del elemento `foreach`, o `@item()['outputs']['body']`. La forma del elemento `@result()` es la misma que la forma de `@actions()`, y se puede analizar del mismo modo.

5. Incluya dos encabezados personalizados con el nombre de la acción con errores `@item()['name']` y el id. de seguimiento de cliente de la ejecución con errores `@item()['clientTrackingId']`.

Como referencia, este es un ejemplo de un solo elemento `@result()`, que muestra las propiedades `name`, `body` y `clientTrackingId` que se analizan en el ejemplo anterior. Fuera de un elemento `foreach`, `@result()` devuelve una matriz de estos objetos.

```json
{
    "name": "Example_Action_That_Failed",
    "inputs": {
        "uri": "https://myfailedaction.azurewebsites.net",
        "method": "POST"
    },
    "outputs": {
        "statusCode": 404,
        "headers": {
            "Date": "Thu, 11 Aug 2016 03:18:18 GMT",
            "Server": "Microsoft-IIS/8.0",
            "X-Powered-By": "ASP.NET",
            "Content-Length": "68",
            "Content-Type": "application/json"
        },
        "body": {
            "code": "ResourceNotFound",
            "message": "/docs/folder-name/resource-name does not exist"
        }
    },
    "startTime": "2016-08-11T03:18:19.7755341Z",
    "endTime": "2016-08-11T03:18:20.2598835Z",
    "trackingId": "bdd82e28-ba2c-4160-a700-e3a8f1a38e22",
    "clientTrackingId": "08587307213861835591296330354",
    "code": "NotFound",
    "status": "Failed"
}
```

Para poner en práctica diferentes patrones de control de excepciones, puede usar las expresiones anteriores. Es posible que elija ejecutar una única acción de control de excepciones fuera del ámbito que acepte toda la matriz filtrada de errores y quite el elemento `foreach`. También puede incluir otras propiedades útiles de la respuesta `@result()` mostrada antes.

## <a name="azure-diagnostics-and-telemetry"></a>Diagnósticos de Azure y datos de telemetría

Los patrones anteriores son una manera excelente de controlar errores y excepciones dentro de una ejecución, pero también puede identificar y responder a los errores con independencia de la ejecución en sí. 
[Diagnósticos de Azure](../logic-apps/logic-apps-monitor-your-logic-apps.md) ofrece un método sencillo de enviar todos los eventos de flujo de trabajo (incluidos todos los estados de ejecución y acción) a una cuenta de Azure Storage o a un Centro de eventos de Azure. Para evaluar los estados de ejecución, puede supervisar los registros y las métricas, o publicarlos en la herramienta de supervisión que prefiera. Una posible opción es transmitir todos los eventos mediante Azure Event Hubs a [Stream Analytics](https://azure.microsoft.com/services/stream-analytics/). En Stream Analytics, puede escribir consultas en directo partiendo de anomalías, promedios o errores en los registros de diagnóstico. Con Stream Analytics es fácil enviar resultados a otros orígenes de datos, como colas, temas, SQL, Azure Cosmos DB y Power BI.

## <a name="next-steps"></a>Pasos siguientes

* [Vea cómo un cliente crea el control de errores con Azure Logic Apps](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)
* [Encuentre más escenarios y ejemplos de Logic Apps](../logic-apps/logic-apps-examples-and-scenarios.md)
* [Aprenda a crear implementaciones automatizadas para aplicaciones lógicas](../logic-apps/logic-apps-create-deploy-template.md)
* [Creación e implementación de aplicaciones lógicas con Visual Studio](logic-apps-deploy-from-vs.md)

<!-- References -->
[retryPolicyMSDN]: https://docs.microsoft.com/rest/api/logic/actions-and-triggers#Anchor_9
