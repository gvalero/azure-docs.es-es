---
title: Copia de datos de Xero con Azure Data Factory (beta) | Microsoft Docs
description: "Obtenga información sobre cómo copiar datos desde Xero a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory."
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: spelluru
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/30/2017
ms.author: jingwang
ms.openlocfilehash: 9236198338fc66697942463e350a0f9732ea217b
ms.sourcegitcommit: c4cc4d76932b059f8c2657081577412e8f405478
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2018
---
# <a name="copy-data-from-xero-using-azure-data-factory-beta"></a>Copiar datos de Xero con Azure Data Factory (beta)

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de Xero. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

> [!NOTE]
> Este artículo se aplica a la versión 2 de Data Factory, que actualmente se encuentra en versión preliminar. Si usa la versión 1 del servicio Data Factory, que está disponible con carácter general, consulte [Actividad de copia en V1](v1/data-factory-data-movement-activities.md).

> [!IMPORTANT]
> Este conector está actualmente en versión beta. Puede probarlo y enviarnos sus comentarios. No debe usarse en entornos de producción.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede copiar datos de Xero en cualquier almacén de datos receptor admitido. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Azure Data Factory proporciona un controlador integrado para habilitar la conectividad. Por lo tanto, no es necesario instalar manualmente ningún controlador mediante este conector.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector Xero.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Xero:

| Propiedad | DESCRIPCIÓN | Requerido |
|:--- |:--- |:--- |
| Tipo | La propiedad type debe establecerse en: **Xero**. | Sí |
| host | Punto de conexión del servidor de Xero (es decir, api.xero.com).  | Sí |
| consumerKey | Clave de consumidor asociada a la aplicación de Xero. Puede elegir marcar este campo como SecureString para almacenarlo de forma segura en Data Factory, o almacenar la contraseña en Azure Key Vault y permitir que se extraiga la copia de la actividad desde allí al realizar la copia de datos. Obtenga más información sobre el [Almacenamiento de credenciales en Key Vault](store-credentials-in-key-vault.md). | Sí |
| privateKey | Clave privada del archivo .pem que se generó para la aplicación privada de Xero. Incluya todo el texto del archivo .pem, así como los finales de línea Unix (\n). Puede elegir marcar este campo como SecureString para almacenarlo de forma segura en Data Factory, o almacenar la contraseña en Azure Key Vault y permitir que se extraiga la copia de la actividad desde allí al realizar la copia de datos. Obtenga más información sobre el [Almacenamiento de credenciales en Key Vault](store-credentials-in-key-vault.md). | Sí |
| useEncryptedEndpoints | Especifica si los puntos de conexión de origen de datos se cifran mediante HTTPS. El valor predeterminado es true.  | Sin  |
| useHostVerification | Especifica si se requiere que el nombre de host del certificado del servidor coincida con el nombre de host del servidor al conectarse a través de SSL. El valor predeterminado es true.  | Sin  |
| usePeerVerification | Especifica si se debe verificar la identidad del servidor al conectarse a través de SSL. El valor predeterminado es true.  | Sin  |

**Ejemplo:**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "host" : "api.xero.com",
            "consumerKey": {
                 "type": "SecureString",
                 "value": "<consumerKey>"
            },
            "privateKey": {
                 "type": "SecureString",
                 "value": "<privateKey>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Xero.

Para copiar datos de Xero, establezca la propiedad type del conjunto de datos en **XeroObject**. No hay ninguna propiedad específica de tipo adicional en este tipo de conjunto de datos.

**Ejemplo**

```json
{
    "name": "XeroDataset",
    "properties": {
        "type": "XeroObject",
        "linkedServiceName": {
            "referenceName": "<Xero linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de Xero.

### <a name="xerosource-as-source"></a>XeroSource como origen

Para copiar datos de Xero, establezca el tipo de origen de la actividad de copia en **XeroSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | DESCRIPCIÓN | Requerido |
|:--- |:--- |:--- |
| Tipo | La propiedad type del origen de la actividad de copia debe establecerse en: **XeroSource** | Sí |
| query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM Contacts"`. | Sí |

**Ejemplo:**

```json
"activities":[
    {
        "name": "CopyFromXero",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Xero input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "XeroSource",
                "query": "SELECT * FROM Contacts"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>pasos siguientes
Para obtener una lista de los almacenes de datos que admite la actividad de copia, consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats).
