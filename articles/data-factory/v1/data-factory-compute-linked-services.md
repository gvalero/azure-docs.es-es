---
title: Entornos de proceso compatibles con Azure Data Factory | Microsoft Docs
description: "Obtenga información sobre los entornos de proceso que puede usar en las canalizaciones de Azure Data Factory (como Azure HDInsight) para transformar o procesar datos."
services: data-factory
documentationcenter: 
author: sharonlo101
manager: jhubbard
editor: monicar
ms.assetid: 6877a7e8-1a58-4cfb-bbd3-252ac72e4145
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/01/2017
ms.author: shlo
robots: noindex
ms.openlocfilehash: 7733ea111de896ab0f825c85b89be25ebafdbd85
ms.sourcegitcommit: 48fce90a4ec357d2fb89183141610789003993d2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/12/2018
---
# <a name="compute-environments-supported-by-azure-data-factory"></a>Entornos de proceso compatibles con Azure Data Factory
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory, que está disponible con carácter general. Si usa la versión 2 del servicio Data Factory, que se encuentra en versión preliminar, vea [Compute linked services in V2](../compute-linked-services.md) (Servicios vinculados de proceso en V2).

En este artículo se explican distintos entornos de procesos que se pueden usar para procesar o transformar datos. También se proporcionan detalles acerca de las distintas configuraciones (a petición frente traiga su propia) admitidas por la Factoría de datos al configurar servicios vinculados que vinculan estos entornos de procesos a una Factoría de datos de Azure.

En la tabla siguiente se proporciona una lista de entornos de proceso compatibles con Data Factory y las actividades que se pueden ejecutar en ellos. 

| Entorno de procesos                      | actividades                               |
| ---------------------------------------- | ---------------------------------------- |
| [Clúster de HDInsight a petición](#azure-hdinsight-on-demand-linked-service) o [clúster HDInsight propio](#azure-hdinsight-linked-service) | [DotNet](data-factory-use-custom-activities.md), [Hive](data-factory-hive-activity.md), [Pig](data-factory-pig-activity.md), [MapReduce](data-factory-map-reduce.md), [Hadoop Streaming](data-factory-hadoop-streaming-activity.md) |
| [Azure Batch](#azure-batch-linked-service) | [DotNet](data-factory-use-custom-activities.md) |
| [Azure Machine Learning](#azure-machine-learning-linked-service) | [Actividades de Machine Learning: ejecución de Batch y recurso de actualización](data-factory-azure-ml-batch-execution-activity.md) |
| [Análisis con Azure Data Lake](#azure-data-lake-analytics-linked-service) | [U-SQL de análisis con Data Lake](data-factory-usql-activity.md) |
| [Azure SQL](#azure-sql-linked-service), [Azure SQL Data Warehouse](#azure-sql-data-warehouse-linked-service), [SQL Server](#sql-server-linked-service) | [Procedimiento almacenado](data-factory-stored-proc-activity.md) |

## <a name="supported-hdinsight-versions-in-azure-data-factory"></a>Versiones admitidas de HDInsight en Azure Data Factory
HDInsight de Azure es compatible con varias versiones de clústeres de Hadoop que se pueden implementar en cualquier momento. Cada versión crea una versión específica de la distribución HortonWorks Data Platform (HDP) y un conjunto de componentes que están incluidos en esa distribución. Microsoft sigue actualizando la lista de versiones admitidas de HDInsight para proporcionar las últimas correcciones y componentes de ecosistema de Hadoop. Para más información, consulte [Versiones compatibles de HDInsight](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions).

> [!IMPORTANT]
> La versión 3.3 de HDInsight basado en Linux se retiró el 31 de julio de 2017. A los clientes de servicios vinculados de HDInsight a petición de Data Factory v1 se les dio de plazo hasta el 15 de diciembre de 2017 para probar y actualizar a una versión posterior de HDInsight. HDInsight basado en Windows se retirará el 31 de julio de 2018.
>
> 

**¿Qué sucede una vez transcurrida la fecha de retirada?** 

Después del 15 de diciembre de 2017:

- Ya no podrá crear clústeres de la versión 3.3 (o versiones anteriores) de HDInsight basado en Linux mediante el servicio vinculado de HDInsight a petición en Azure Data Factory v1. 
- Si no se especifican las [propiedades osType y/o Version](https://docs.microsoft.com/azure/data-factory/v1/data-factory-compute-linked-services#azure-hdinsight-on-demand-linked-service) explícitamente en las definiciones JSON existentes de servicios vinculados de HDInsight a petición en Azure Data Factory v1, el valor predeterminado se cambiará de **Version=3.1, osType=Windows** a **Version=[última versión predeterminada de HDI](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#hadoop-components-available-with-different-hdinsight-versions), osType=Linux**.

Después del 31 de julio de 2018:

- Ya no podrá crear ninguna versión de los clústeres de HDInsight basado en Linux mediante los servicios vinculados de HDInsight a petición en Azure Data Factory v1. 

 **Acciones recomendadas** 

- Actualice la [propiedad osType and/or Version](https://docs.microsoft.com/azure/data-factory/v1/data-factory-compute-linked-services#azure-hdinsight-on-demand-linked-service) de las definiciones afectadas de servicios vinculados de HDInsight a petición en Azure Data Factory v1 a versiones más nuevas de HDInsight basado en Linux (HDInsight 3.6) para asegurarse de que puede usar los componentes y correcciones más recientes del ecosistema de Hadoop. 
- Antes del 15 de diciembre de 2017, pruebe las actividades de streaming de Hive, Pig, MapReduce y Hadoop de Azure Data Factory V1 que hacen referencia al servicio vinculado afectado para asegurarse de que son compatibles con el nuevo valor predeterminado de *osType* and/or *Version* (Version=3.6, osType=Linux) o con los valores explícitos de versión y tipo de sistema operativo de HDInsight a los que va a actualizar. Para aprender más sobre la compatibilidad, revise las páginas web de documentación [Migración desde un clúster de HDInsight basado en Windows a un clúster basado en Linux](https://docs.microsoft.com/azure/hdinsight/hdinsight-migrate-from-windows-to-linux) y [¿Cuáles son los componentes y versiones de Hadoop disponibles con HDInsight?](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#hortonworks-release-notes-associated-with-hdinsight-versions) 
- Establezca explícitamente el tipo de sistema operativo de Windows antes del 15 de diciembre de 2017, si quiere seguir usando los servicios vinculados de HDInsight a petición de Azure Data Factory v1 para crear clústeres de HDInsight basados en Windows. Sin embargo, aún se recomienda migrar a clústeres de HDInsight basado en Linux antes del 31 de julio de 2018. 
- Si va a usar un servicio vinculado de HDInsight a petición para ejecutar la actividad personalizada de DotNet de Azure Data Factory v1, actualice la definición JSON de actividad personalizada de DotNet para usar en su lugar un servicio vinculado de Azure Batch. Aprenda más en la página web de documentación [Uso de actividades personalizadas en una canalización de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/v1/data-factory-use-custom-activities). 

>[!Note]
>Para los clientes que usan su servicio vinculado existente Bring Your Own Cluster (BYOC) de HDInsight en Azure Data Factory v1 o para aquellos que usan BYOC y un servicio vinculado de HDInsight a petición en Azure Data Factory v2, ya está aplicada la versión más reciente de la directiva de compatibilidad de los clústeres de Azure HDInsight, así que no se requiere ninguna acción. 
>
> 


## <a name="on-demand-compute-environment"></a>Entorno de procesos a petición
En este tipo de configuración, el entorno de procesos está totalmente administrado por el servicio Azure Data Factory. El servicio Factoría de datos lo crea automáticamente antes de que se envíe un trabajo para procesar los datos y que se quite cuando finalice el trabajo. Puede crear un servicio vinculado para el entorno de procesos a petición, configurarlo y controlar la configuración granular para la ejecución del trabajo, la administración del clúster y las acciones de arranque.

> [!NOTE]
> La configuración a petición solo se admite actualmente para los clústeres de HDInsight de Azure.
>
> 

## <a name="azure-hdinsight-on-demand-linked-service"></a>Servicio vinculado a petición de HDInsight de Azure
El servicio Azure Data Factory crea automáticamente un clúster de HDInsight basado en Windows/Linux a petición para procesar los datos. El clúster se crea en la misma región que la cuenta de almacenamiento (propiedad linkedServiceName en JSON) asociada al clúster.

Tenga en cuenta los siguientes puntos **importantes** acerca del servicio vinculado de HDInsight a petición:

* No se ve el clúster de HDInsight a petición creado en su suscripción de Azure. El servicio Azure Data Factory administra el clúster de HDInsight a petición en su nombre.
* Los registros de trabajos que se ejecutan en un clúster de HDInsight a petición se copian en la cuenta de almacenamiento asociada al clúster de HDInsight. Puede tener acceso a estos registros desde el Portal de Azure en la hoja **Detalles de la ejecución de actividad** . Consulte el artículo [Supervisión y administración de canalizaciones](data-factory-monitor-manage-pipelines.md) para obtener más información.
* Se le cobrará solo por el tiempo en el que el clúster de HDInsight esté en ejecución y realizando trabajos.

> [!IMPORTANT]
> El aprovisionamiento bajo demanda de un clúster de Azure HDInsight suele tardar **20 minutos** o más.
>
> 

### <a name="example"></a>Ejemplo
En el siguiente JSON se define un servicio vinculado de HDInsight a petición basado en Linux. El servicio Data Factory crea automáticamente un clúster de HDInsight **basado en Linux** al procesar un segmento de datos. 

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.6",
            "osType": "Linux",
            "clusterSize": 1,
            "timeToLive": "00:05:00",            
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

> [!IMPORTANT]
> El clúster de HDInsight crea un **contenedor predeterminado** en el almacenamiento de blobs que especificó en JSON (**linkedServiceName**). HDInsight no elimina este contenedor cuando se elimina el clúster. Este comportamiento es así por diseño. Con el servicio vinculado de HDInsight a petición se crea un clúster de HDInsight cada vez tenga que procesarse un segmento, a menos que haya un clúster existente activo (**timeToLive**), que se elimina cuando finaliza el procesamiento. 
>
> A medida que se procesen más segmentos, verá numerosos contenedores en su Almacenamiento de blobs de Azure. Si no los necesita para solucionar problemas de trabajos, puede eliminarlos para reducir el costo de almacenamiento. Los nombres de estos contenedores siguen un patrón: `adf**yourdatafactoryname**-**linkedservicename**-datetimestamp`. Use herramientas como el [Explorador de Microsoft Storage](http://storageexplorer.com/) para eliminar contenedores de Azure Blob Storage.
>
> 

### <a name="properties"></a>Properties (Propiedades)
| Propiedad                     | DESCRIPCIÓN                              | Requerido |
| ---------------------------- | ---------------------------------------- | -------- |
| Tipo                         | La propiedad type se debe establecer en **HDInsightOnDemand**. | Sí      |
| clusterSize                  | Número de nodos de datos o trabajo del clúster El clúster de HDInsight se crea con dos nodos principales junto con el número de nodos de trabajo que haya especificado para esta propiedad. Los nodos son de tamaño Standard_D3 con 4 núcleos, por lo que un clúster de nodos de 4 trabajos necesitará 24 núcleos (4\*4 = 16 para nodos de trabajo, más 2\*4 = 8 para nodos principales). Consulte [Creación de clústeres de Hadoop basados en Linux en HDInsight](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md) para más información acerca del nivel Standard_D3. | Sí      |
| timeToLive                   | El tiempo de inactividad permitido para el clúster de HDInsight a petición. Especifica cuánto tiempo permanece activo el clúster de HDInsight a petición después de la finalización de una ejecución de actividad si no hay ningún otro trabajo activo en el clúster.<br/><br/>Por ejemplo, si una ejecución de actividad tarda 6 minutos y timetolive está establecido en 5 minutos, el clúster permanece activo durante 5 minutos después de los 6 minutos de procesamiento de la ejecución de actividad. Si se ejecuta otra actividad con un margen de 6 minutos, la procesa el mismo clúster.<br/><br/>Crear un clúster de HDInsight a petición es una operación costosa (podría tardar un poco); use esta configuración si es necesario para mejorar el rendimiento de una factoría de datos mediante la reutilización de un clúster de HDInsight a petición.<br/><br/>Si establece el valor de timetolive en 0, el clúster se elimina en cuanto se completa la ejecución de la actividad. Por el contrario, si establece un valor alto, el clúster puede permanecer inactivo innecesariamente, lo que conlleva unos costos elevados. Por lo tanto, es importante que establezca el valor adecuado en función de sus necesidades.<br/><br/>Varias canalizaciones pueden compartir la instancia del clúster de HDInsight a petición si el valor de la propiedad timetolive está correctamente configurado. | Sí      |
| version                      | Versión del clúster de HDInsight, consulte las [versiones de HDInsight compatibles](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#supported-hdinsight-versions) para obtener las versiones de HDInsight que se permiten. Si no se especifica, usa la [versión predeterminada más reciente de HDI](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#hadoop-components-available-with-different-hdinsight-versions). | Sin        |
| linkedServiceName            | El servicio vinculado de Azure Storage que usará el clúster a petición para almacenar y procesar datos. El clúster de HDInsight se crea en la misma región que esta cuenta de Azure Storage.<p>Actualmente, no se puede crear un clúster de HDInsight a petición que utilice una instancia de Azure Data Lake Store como almacenamiento. Si desea almacenar los datos de resultados del procesamiento de HDInsight en una instancia de Azure Data Lake Store, utilice una actividad de copia para copiar los datos desde Azure Blob Storage a Azure Data Lake Store. </p> | Sí      |
| additionalLinkedServiceNames | Especifica cuentas de almacenamiento adicionales para el servicio vinculado de HDInsight, de forma que el servicio Factoría de datos pueda registrarlas en su nombre. Estas cuentas de almacenamiento deben estar en la misma región que el clúster de HDInsight, que se crea en la misma región que la cuenta de almacenamiento especificada por linkedServiceName. | Sin        |
| osType                       | Tipo de sistema operativo. Los valores permitidos son: Linux y Windows. Si no se especifica, Linux se utiliza de forma predeterminada.  <br/>Recomendamos usar clústeres de HDInsight basados en Linux, ya que a partir del 31 de julio de 2018 se retirará HDInsight de Windows. | Sin        |
| hcatalogLinkedServiceName    | Nombre del servicio vinculado de SQL de Azure que apunta a la base de datos de HCatalog. El clúster de HDInsight a petición se creará usando la base de datos SQL de Azure como metaalmacén. | Sin        |

#### <a name="additionallinkedservicenames-json-example"></a>Ejemplo JSON de additionalLinkedServiceNames

```json
"additionalLinkedServiceNames": [
    "otherLinkedServiceName1",
    "otherLinkedServiceName2"
  ]
```

### <a name="advanced-properties"></a>Propiedades avanzadas
También puede especificar las siguientes propiedades para la configuración granular del clúster de HDInsight a petición.

| Propiedad               | DESCRIPCIÓN                              | Requerido |
| :--------------------- | :--------------------------------------- | :------- |
| coreConfiguration      | Especifica los parámetros de configuración Core (como en core-site.xml) para crear el clúster de HDInsight. | Sin        |
| hBaseConfiguration     | Especifica los parámetros de configuración HBase (como en hbase-site.xml) para el clúster de HDInsight. | Sin        |
| hdfsConfiguration      | Especifica los parámetros de configuración HDFS (hdfs-site.xml) para el clúster de HDInsight. | Sin        |
| hiveConfiguration      | Especifica los parámetros de configuración Hive (hive-site.xml) para el clúster de HDInsight. | Sin        |
| mapReduceConfiguration | Especifica los parámetros de configuración MapReduce (mapred-site.xml) para el clúster de HDInsight. | Sin        |
| oozieConfiguration     | Especifica los parámetros de configuración Oozie (oozie-site.xml) para el clúster de HDInsight. | Sin        |
| stormConfiguration     | Especifica los parámetros de configuración Storm (storm-site.xml) para el clúster de HDInsight. | Sin        |
| yarnConfiguration      | Especifica los parámetros de configuración Yarn (yarn-site.xml) para el clúster de HDInsight. | Sin        |

#### <a name="example--on-demand-hdinsight-cluster-configuration-with-advanced-properties"></a>Ejemplo: configuración del clúster de HDInsight a petición con propiedades avanzadas

```json
{
  "name": " HDInsightOnDemandLinkedService",
  "properties": {
    "type": "HDInsightOnDemand",
    "typeProperties": {
      "version": "3.6",
      "osType": "Linux",
      "clusterSize": 16,
      "timeToLive": "01:30:00",
      "linkedServiceName": "adfods1",
      "coreConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "hiveConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "mapReduceConfiguration": {
        "mapreduce.reduce.java.opts": "-Xmx4000m",
        "mapreduce.map.java.opts": "-Xmx4000m",
        "mapreduce.map.memory.mb": "5000",
        "mapreduce.reduce.memory.mb": "5000",
        "mapreduce.job.reduce.slowstart.completedmaps": "0.8"
      },
      "yarnConfiguration": {
        "yarn.app.mapreduce.am.resource.mb": "5000",
        "mapreduce.map.memory.mb": "5000"
      },
      "additionalLinkedServiceNames": [
        "datafeeds",
        "adobedatafeed"
      ]
    }
  }
}
```

### <a name="node-sizes"></a>Tamaño de nodo
Puede especificar los tamaños de los nodos principal, de datos y de zookeeper con las siguientes propiedades: 

| Propiedad          | DESCRIPCIÓN                              | Requerido |
| :---------------- | :--------------------------------------- | :------- |
| headNodeSize      | Especifica el tamaño del nodo principal. El valor predeterminado es: Standard_D3 Consulte la sección **Especificación de tamaños de nodos** para más información. | Sin        |
| dataNodeSize      | Especifica el tamaño del nodo de datos. El valor predeterminado es: Standard_D3 | Sin        |
| zookeeperNodeSize | Especifica el tamaño del nodo de Zoo Keeper. El valor predeterminado es: Standard_D3 | Sin        |

#### <a name="specifying-node-sizes"></a>Especificación de tamaños de nodo
Consulte el artículo [Tamaños de máquinas virtuales](../../virtual-machines/linux/sizes.md) para conocer los valores de cadena que debe especificar para las propiedades mencionadas anteriormente. Los valores deben ser conformes a los **CMDLET y API** a los que se hace referencia en el artículo. Como puede ver en el artículo, el nodo de datos de tamaño grande (valor predeterminado) tiene 7 GB de memoria, que es posible que no sea lo suficientemente bueno para su escenario. 

Si quiere crear nodos de trabajo y principales de tamaño D4, especifique **Standard_D4** para el valor de las propiedades headNodeSize y dataNodeSize. 

```json
"headNodeSize": "Standard_D4",    
"dataNodeSize": "Standard_D4",
```

Si especifica un valor incorrecto para estas propiedades, puede recibir el siguiente **error:** Error al crear el clúster. Excepción: No se puede completar la operación de creación del clúster. Error en la operación con el código '400'. El clúster generó el estado: 'Error'. Mensaje: 'PreClusterCreationValidationFailure'. Si recibe este error, asegúrese de que está usando el nombre de **CMDLET y API** de la tabla del artículo [Tamaños de las máquinas virtuales](../../virtual-machines/linux/sizes.md).  

> [!NOTE]
> Actualmente Azure Data Factory no es compatible con clústeres de HDInsight que usan Azure Data Lake Store como almacén principal. Use Azure Storage como almacén principal para los clústeres de HDInsight. 
>
> 


## <a name="bring-your-own-compute-environment"></a>Traer su propio entorno de procesos
En este tipo de configuración, los usuarios pueden registrar un entorno de procesos existente como un servicio vinculado en la Factoría de datos. El usuario administra el entorno de procesos y el servicio Factoría de datos lo usa para ejecutar las actividades.

Este tipo de configuración se admite para los entornos de procesos siguientes:

* HDInsight de Azure
* Azure Batch
* Azure Machine Learning
* Análisis con Azure Data Lake
* Azure SQL DB, Azure SQL DW, SQL Server

## <a name="azure-hdinsight-linked-service"></a>Servicio vinculado de HDInsight de Azure
Puede crear un servicio vinculado de HDInsight de Azure para registrar su propio clúster de HDInsight con la Factoría de datos.

### <a name="example"></a>Ejemplo

```json
{
  "name": "HDInsightLinkedService",
  "properties": {
    "type": "HDInsight",
    "typeProperties": {
      "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
      "userName": "admin",
      "password": "<password>",
      "linkedServiceName": "MyHDInsightStoragelinkedService"
    }
  }
}
```

### <a name="properties"></a>Properties (Propiedades)
| Propiedad          | DESCRIPCIÓN                              | Requerido |
| ----------------- | ---------------------------------------- | -------- |
| Tipo              | La propiedad type se debe establecer en **HDInsight**. | Sí      |
| clusterUri        | El URI del clúster de HDInsight.        | Sí      |
| Nombre de usuario          | Especifique el nombre de usuario que se usará para conectarse a un clúster de HDInsight existente. | Sí      |
| contraseña          | Especifique la contraseña para la cuenta de usuario.   | Sí      |
| linkedServiceName | Nombre del servicio vinculado para Azure Storage que hace referencia al almacenamiento Azure Blob Storage que usa el clúster de HDInsight. <p>Actualmente, no se puede especificar un servicio vinculado de Azure Data Lake Store para esta propiedad. Puede acceder a los datos de Azure Data Lake Store desde scripts de Hive o Pig si el clúster de HDInsight tiene acceso a Data Lake Store. </p> | Sí      |

## <a name="azure-batch-linked-service"></a>Servicio vinculado de Azure Batch
Puede crear un servicio vinculado de Azure Batch para registrar un grupo de lotes de máquinas virtuales (VM) en una factoría de datos. Puede ejecutar actividades personalizadas .NET con Azure Batch o Azure HDInsight.

Consulte los temas siguientes si no está familiarizado con el servicio Azure Batch:

* [Aspectos básicos de Azure Batch](../../batch/batch-technical-overview.md) para información general del servicio Azure Batch.
* Cmdlet [New-AzureBatchAccount](https://msdn.microsoft.com/library/mt125880.aspx) para crear una cuenta de Azure Batch, o [Azure Portal](../../batch/batch-account-create-portal.md) para crear la cuenta de Azure Batch con Azure Portal. Consulte el tema [Uso de Azure PowerShell para administrar la cuenta de Azure Batch](http://blogs.technet.com/b/windowshpc/archive/2014/10/28/using-azure-powershell-to-manage-azure-batch-account.aspx) para obtener instrucciones detalladas sobre cómo usar este cmdlet.
* [New-AzureBatchPool](https://msdn.microsoft.com/library/mt125936.aspx) para crear un grupo de Lote de Azure.

### <a name="example"></a>Ejemplo

```json
{
  "name": "AzureBatchLinkedService",
  "properties": {
    "type": "AzureBatch",
    "typeProperties": {
      "accountName": "<Azure Batch account name>",
      "accessKey": "<Azure Batch account key>",
      "poolName": "<Azure Batch pool name>",
      "linkedServiceName": "<Specify associated storage linked service reference here>"
    }
  }
}
```

Agregue "**.\<nombre de región\>**" al nombre de la cuenta de Batch para la propiedad **accountName**. Ejemplo:

```json
"accountName": "mybatchaccount.eastus"
```

Otra opción es proporcionar el punto de conexión batchUri tal como se muestra en el siguiente ejemplo:

```json
"accountName": "adfteam",
"batchUri": "https://eastus.batch.azure.com",
```

### <a name="properties"></a>Properties (Propiedades)
| Propiedad          | DESCRIPCIÓN                              | Requerido |
| ----------------- | ---------------------------------------- | -------- |
| Tipo              | La propiedad type se debe establecer en **AzureBatch**. | Sí      |
| accountName       | Nombre de la cuenta de Azure Batch.         | Sí      |
| accessKey         | Clave de acceso de la cuenta de Azure Batch.  | Sí      |
| poolName          | Nombre del grupo de máquinas virtuales.    | Sí      |
| linkedServiceName | Nombre del servicio vinculado de Azure Storage asociado a este servicio vinculado de Azure Batch. Este servicio vinculado se usa para los archivos de almacenamiento provisional necesarios para ejecutar la actividad y almacenar los registros de ejecución de la actividad. | Sí      |

## <a name="azure-machine-learning-linked-service"></a>Servicio vinculado de Azure Machine Learning
Un servicio vinculado de Azure Machine Learning se crea para registrar un punto de conexión de puntuación por lotes de Machine Learning en una factoría de datos.

### <a name="example"></a>Ejemplo

```json
{
  "name": "AzureMLLinkedService",
  "properties": {
    "type": "AzureML",
    "typeProperties": {
      "mlEndpoint": "https://[batch scoring endpoint]/jobs",
      "apiKey": "<apikey>"
    }
  }
}
```

### <a name="properties"></a>Properties (Propiedades)
| Propiedad   | DESCRIPCIÓN                              | Requerido |
| ---------- | ---------------------------------------- | -------- |
| type       | La propiedad type se debe establecer en: **AzureML**. | Sí      |
| mlEndpoint | La dirección URL de puntuación por lotes.                   | Sí      |
| apiKey     | La API del modelo de área de trabajo publicado.     | Sí      |

## <a name="azure-data-lake-analytics-linked-service"></a>Servicio vinculado con el Análisis con Azure Data Lake
Cree un servicio vinculado de **Azure Data Lake Analytics** para vincular un servicio de proceso de Azure Data Lake Analytics a una instancia de Azure Data Factory. La actividad de U-SQL de Data Lake Analytics de la canalización hace referencia a este servicio vinculado. 

En la siguiente tabla se ofrecen descripciones de las propiedades genéricas que se usan en la definición de JSON. Puede elegir entre la autenticación de la entidad de servicio y la autenticación de credenciales de usuario.

| Propiedad                 | DESCRIPCIÓN                              | Requerido                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| **type**                 | La propiedad type se debe establecer en: **AzureDataLakeAnalytics**. | Sí                                      |
| **accountName**          | Nombre de la cuenta de Análisis de Azure Data Lake  | Sí                                      |
| **dataLakeAnalyticsUri** | Identificador URI de Análisis de Azure Data Lake.           | Sin                                        |
| **subscriptionId**       | Identificador de suscripción de Azure                    | No (si no se especifica, se usa la suscripción de Data Factory). |
| **resourceGroupName**    | Nombre del grupo de recursos de Azure.                | No (si no se especifica, se usa el grupo de recursos de la factoría de datos). |

### <a name="service-principal-authentication-recommended"></a>Autenticación de la entidad de servicio (recomendada)
Para usar la autenticación de la entidad de servicio, registre una entidad de aplicación en Azure Active Directory (AAD) y concédale acceso a Data Lake Store. Consulte [Autenticación entre servicios](../../data-lake-store/data-lake-store-authenticate-using-active-directory.md) para ver los pasos detallados. Anote los siguientes valores; los usará para definir el servicio vinculado:
* Identificador de aplicación
* Clave de la aplicación 
* Id. de inquilino

Para usar la autenticación de la entidad de servicio, especifique las siguientes propiedades:

| Propiedad                | DESCRIPCIÓN                              | Requerido |
| :---------------------- | :--------------------------------------- | :------- |
| **servicePrincipalId**  | Especifique el id. de cliente de la aplicación.     | Sí      |
| **servicePrincipalKey** | Especifique la clave de la aplicación.           | Sí      |
| **tenant**              | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí      |

**Ejemplo: autenticación de la entidad de servicio**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>Autenticación de credenciales de usuario
También puede utilizar la autenticación de credenciales de usuario para Data Lake Analytics mediante la especificación de las propiedades siguientes:

| Propiedad          | DESCRIPCIÓN                              | Requerido |
| :---------------- | :--------------------------------------- | :------- |
| **authorization** | Haga clic en el botón **Autorizar** de Data Factory Editor y escriba sus credenciales, que asignan la dirección URL de autorización generada automáticamente a esta propiedad. | Sí      |
| **sessionId**     | Id. de sesión de OAuth de la sesión de autorización de OAuth. Cada id. de sesión es único y solo se puede usar una vez. Esta configuración se genera automáticamente al usar Data Factory Editor. | Sí      |

**Ejemplo: autenticación de credenciales de usuario**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>Expiración del token
El código de autorización que se generó al hacer clic en el botón **Autorizar** expira poco tiempo después. Consulte la tabla siguiente para conocer el momento en que expiran los distintos tipos de cuentas de usuario. Puede ver el siguiente mensaje de error cuando el **token de autenticación expira**: Error de operación de credencial: invalid_grant - AADSTS70002: error al validar las credenciales. AADSTS70008: la concesión de acceso proporcionada expiró o se revocó. Id. de seguimiento: d18629e8-af88-43c5-88e3-d8419eb1fca1 Id. de correlación: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Marca de tiempo: 2015-12-15 21:09:31Z

| Tipo de usuario                                | Expira después de                            |
| :--------------------------------------- | :--------------------------------------- |
| Cuentas de usuario NO administradas por Azure Active Directory (@hotmail.com, @live.com, etc.) | 12 horas                                 |
| Cuentas de usuario administradas por Azure Active Directory (AAD) | 14 días después de la ejecución del último segmento. <br/><br/>Noventa días, si un segmento basado en el servicio vinculado basado en OAuth se ejecuta al menos una vez cada catorce días. |

Para evitar o resolver este error, vuelva a dar la autorización con el botón **Autorizar** cuando el **token expire** e implemente de nuevo el servicio vinculado. También puede generar valores para las propiedades **sessionId** y **authorization** mediante programación, para lo que usará el código siguiente:

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

Para más información sobre las clases de Data Factory que se usan en el código, consulte los temas [AzureDataLakeStoreLinkedService Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx), [AzureDataLakeAnalyticsLinkedService Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx) y [AuthorizationSessionGetResponse Class](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx). Agregue una referencia a Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll para la clase WindowsFormsWebAuthenticationDialog. 

## <a name="azure-sql-linked-service"></a>Servicio vinculado SQL de Azure
Cree un servicio vinculado de Azure SQL y úselo con la [actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para invocar un procedimiento almacenado desde una canalización de Factoría de datos. Vea el artículo [Conector SQL de Azure](data-factory-azure-sql-connector.md#linked-service-properties) para más información sobre este servicio vinculado.

## <a name="azure-sql-data-warehouse-linked-service"></a>Servicio vinculado de Azure SQL Data Warehouse
Cree un servicio vinculado de SQL Data Warehouse y úselo con la [actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para invocar un procedimiento almacenado desde una canalización de Data Factory. Consulte el artículo sobre el [conector de Azure SQL Data Warehouse](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties) para más información acerca de este servicio vinculado.

## <a name="sql-server-linked-service"></a>Servicio vinculado de SQL Server
Cree un servicio vinculado de SQL Server y úselo con la [actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para invocar un procedimiento almacenado desde una canalización de Data Factory. Consulte el artículo sobre el [conector de SQL Server](data-factory-sqlserver-connector.md#linked-service-properties) para más información acerca de este servicio vinculado.

