---
title: "Migración de máquinas virtuales de AWS a Azure con Azure Site Recovery | Microsoft Docs"
description: "En este artículo se describe cómo migrar máquinas virtuales que se están ejecutando en Amazon Web Services (AWS) en Azure con Azure Site Recovery."
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: carmonm
editor: 
ms.assetid: ddb412fd-32a8-4afa-9e39-738b11b91118
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 11/01/2017
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 814d8ee4952dd08707849eadc1e4e97ab6087da0
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/11/2017
---
# <a name="migrate-amazon-web-services-aws-vms-to-azure"></a>Migración de máquinas virtuales de Amazon Web Services (AWS) a Azure

En este tutorial se enseña cómo migrar máquinas virtuales (VM) de Amazon Web Services (AWS) a máquinas virtuales de Azure con Site Recovery. Al migrar instancias de EC2 a Azure, se tratan las máquinas virtuales como si fuesen físicas en los equipos locales. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Preparar recursos de Azure
> * Preparar las instancias de AWS EC2 para la migración
> * Implementar un servidor de configuración
> * Habilitar la replicación para máquinas virtuales
> * Probar la conmutación por error para asegurarse de que todo funciona
> * Ejecutar una conmutación por error única en Azure

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.


## <a name="prepare-azure-resources"></a>Preparar recursos de Azure 

Debe tener algunos recursos preparados en Azure para las instancias de EC2 migradas que se van a usar. Tales incluyen una cuenta de almacenamiento, un almacén y una red virtual.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Las imágenes de máquinas replicadas se conservan en Azure Storage. Las máquinas virtuales de Azure se crean desde el almacenamiento cuando se realiza la conmutación por error desde el entorno local en Azure.

1. En el menú [Azure Portal](https://portal.azure.com), haga clic en **Nuevo** -> **Storage** -> **Cuenta de Storage**.
2. Escriba un nombre para la cuenta de almacenamiento. En estos tutoriales, usamos el nombre **awsmigrated2017**. El nombre debe ser único en Azure, tener entre 3 y 24 caracteres, y usar solo números y letras minúsculas.
3. Mantenga los valores predeterminados de **Modelo de implementación**, **Tipo de cuenta**, **Rendimiento** y **Se requiere transferencia segura**.
5. Seleccione el valor predeterminado **RA-GRS** en **Replicación**.
6. Seleccione la suscripción que quiere usar para este tutorial.
7. En **Grupo de recursos**, seleccione **Crear nuevo**. En este ejemplo se usa el nombre **migrationRG**.
8. Seleccione **Europa Occidental** como ubicación. 
9. Haga clic en **Crear** para crear la cuenta de almacenamiento.

### <a name="create-a-vault"></a>Creación de un almacén

1. En [Azure Portal](https://portal.azure.com), en el panel de navegación izquierdo, haga clic en **Más servicios**, y busque y seleccione **Almacenes de Recovery Services**.
2. En la página Almacenes de Recovery Services, haga clic en **+ Agregar** en la parte superior izquierda de la página.
3. En **Nombre**, escriba *myVault*. 
4. En **Suscripción**, seleccione la suscripción adecuada.
4. En **Grupo de recursos**, seleccione **Usar existente** y *migrationRG*. 
5. En **Ubicación**, seleccione *Europa Occidental*. 
5. Para acceder rápidamente al nuevo almacén desde el panel, seleccione **Anclar al panel**.
7. Cuando haya terminado, haga clic en **Crear**.

El nuevo almacén aparecerá en **Panel** > **Todos los recursos** y en la página principal de **Almacenes de Recovery Services**.

### <a name="set-up-an-azure-network"></a>Configurar una red de Azure

Cuando se crean máquinas virtuales de Azure después de la migración (conmutación por error), se unen a esta red.

1. En [Azure Portal](https://portal.azure.com), haga clic en **Nuevo** > **Redes** >
   **Red virtual**
3. En **Nombre**, escriba *myMigrationNetwork*.
4. Deje el valor predeterminado de **Espacio de direcciones**.
5. En **Suscripción**, seleccione la suscripción adecuada.
6. En **Grupo de recursos**, seleccione **Usar existente** y elija *migrationRG* en el menú desplegable.
7. En **Ubicación**, seleccione **Europa Occidental**. 
8. Deje los valores predeterminados de **Subred**, tanto de **Nombre** como de **Intervalo IP**.
9. Deje **Puntos de conexión de servicio** deshabilitado.
10. Cuando haya terminado, haga clic en **Crear**.


## <a name="prepare-the-ec2-instances"></a>Preparar las instancias de EC2

Necesita una o varias máquinas virtuales que quiera migrar. Estas instancias de EC2 deben ejecutar la versión de 64 bits de Windows Server 2008 R2 SP1 o posterior, Windows Server 2012, Windows Server 2012 R2 o Red Hat Enterprise Linux 6.7 (solo instancias de HVM virtualizadas). El servidor debe tener solo los controladores Citrix PV o AWS PV. No se admiten instancias que ejecuten controladores RedHat PV.

Mobility Service debe instalarse en cada máquina que quiera replicar. Site Recovery instala este servicio automáticamente cuando se habilita la replicación para la máquina virtual. Para la instalación automática, debe preparar una cuenta en las instancias de EC2, que Site Recovery usará para acceder a la máquina virtual.

Puede usar una cuenta local o de dominio. En el caso de las máquinas virtuales Linux, debe ser la cuenta raíz en el servidor Linux de origen. En el caso de las máquinas virtuales Windows, si no usa una cuenta de dominio, deshabilite el control Acceso de usuarios remotos en la máquina local:

  - En el Registro, en **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**, agregue la entrada DWORD **LocalAccountTokenFilterPolicy** y establezca el valor en 1.
    
También necesita una instancia de EC2 aparte, que puede usar como el servidor de configuración de Site Recovery. Esta instancia debe ejecutar Windows Server 2012 R2. 
    

## <a name="prepare-the-infrastructure"></a>Preparación de la infraestructura 

En la página del portal del almacén, seleccione **Site Recovery** en la sección **Introducción** y, a continuación, haga clic en **Preparar infraestructura**.

### <a name="1-protection-goal"></a>1 Objetivo de protección

Seleccione los valores siguientes en la página **Objetivo de protección**:
   
|    |  | 
|---------|-----------|
| ¿Dónde se encuentran sus máquinas? | **Local**|
| ¿Dónde quiere replicar las máquinas? |**En Azure**|
| ¿Las máquinas están virtualizadas? | **No virtualizado/Otro**|

Cuando haya terminado, haga clic en **Aceptar** para continuar con la siguiente sección.

### <a name="2-source-prepare"></a>2 Preparación del origen 

En el panel **Preparar origen**, haga clic en **+Servidor de configuración**. 

1. Use una instancia de EC2 que ejecute Windows Server 2012 R2 para crear un servidor de configuración y registrarlo en el almacén de recuperación.

2. Configure el proxy en la máquina virtual de la instancia de EC2 que usa como el servidor de configuración para que pueda acceder a [Direcciones URL del servicio](site-recovery-support-matrix-to-azure.md).

3. Descargue el programa [Instalación unificada de Microsoft Azure Site Recovery](http://aka.ms/unifiedinstaller_wus). Puede descargarlo en la máquina local y, a continuación, copiarlo en la máquina virtual que usa como el servidor de configuración.

4. Haga clic en el botón **Descargar** para descargar la clave de registro del almacén. Copie el archivo descargado en la máquina virtual que usa como el servidor de configuración.

5. En la máquina virtual, haga clic en el instalador descargado **Instalación unificada de Microsoft Azure Site Recovery** y seleccione **Ejecutar como administrador**. 

    1. En **Antes de comenzar**, seleccione **Install the configuration server and process server** (Instalar el servidor de configuración y el servidor de procesos) y luego haga clic en **Siguiente**.
    2. En **Third-Party Software License** (Licencia de software de terceros), haga clic en **I Accept the third party license agreement** (Acepto el contrato de licencia de terceros) y haga clic en **Siguiente**.
    3. En **Registro**, haga clic en Examinar y navegue hasta la ubicación donde colocó el archivo de clave de registro del almacén y, a continuación, haga clic en **Siguiente**.
    4. En **Configuración de Internet**, seleccione **Conectar directamente con Azure Site Recovery sin un servidor proxy.** y haga clic en **Siguiente**.
    5. En la página **Comprobación de requisitos previos**, se ejecutan comprobaciones de varios elementos. Cuando haya terminado, haga clic en **Siguiente**. 
    6. En **Configuración de MySQL**, proporcione las contraseñas necesarias y, a continuación, haga clic en **Siguiente**.
    7. En **Detalles del entorno**, seleccione **No**, ya que no es necesario proteger máquinas de VMware y, a continuación, haga clic en **Siguiente**.
    8. En **Ubicación de instalación**, haga clic en **Siguiente** para aceptar el valor predeterminado.
    9. En **Selección de red**, haga clic en **Siguiente** para aceptar el valor predeterminado.
    10. En **Resumen**, haga clic en **Instalar**.
    11. **Progreso de la instalación** muestra información acerca del punto del proceso de instalación en que se encuentra. Cuando haya terminado, haga clic en **Finalizar**. Se muestra un elemento emergente sobre la necesidad de un posible reinicio. Haga clic en **Aceptar**. También se muestra un elemento emergente sobre la frase de contraseña de conexión del servidor de configuración. Copie la frase de contraseña en el Portapapeles y guárdela en algún lugar seguro.
    
6. En la máquina virtual, ejecute **cspsconfigtool.exe** para crear una o varias cuentas de administración en el servidor de configuración. Asegúrese de que las cuentas de administración tienen permisos de administrador en las instancias de EC2 que quiere migrar. 

Cuando termine de configurar el servidor de configuración, vuelva al portal y seleccione el servidor que acaba de crear como **Servidor de configuración** y, a continuación, haga clic en *Aceptar** para continuar en el paso 3 Preparación del destino.

### <a name="3-target-prepare"></a>3 Preparación del destino 

En esta sección, debe introducir información sobre los recursos que creó en la sección [Preparar recursos de Azure](#prepare-azure-resources) anteriormente en este tutorial.

1. En **Suscripción**, seleccione la suscripción de Azure que usó para el tutorial [Preparación de Azure](tutorial-prepare-azure.md).
2. Seleccione **Resource Manager** como modelo de implementación.
3. Site Recovery comprueba que tiene una o más redes y cuentas de Azure Storage compatibles. Deberían ser los recursos que creó en la sección [Preparar recursos de Azure](#prepare-azure-resources) anteriormente en este tutorial.
4. Cuando haya terminado, haga clic en **Aceptar**.


### <a name="4-replication-settings-prepare"></a>4 Preparación de la configuración de replicación 

Debe crear una directiva de replicación para poder habilitar la replicación

1. Haga clic en **+ Replicate and Associate** (+ Replicar y asociar).
2. En **Nombre**, escriba **myReplicationPolicy**.
3. Deje el resto de la configuración en los valores predeterminados y haga clic en **Aceptar** para crear la directiva. La nueva directiva se asocia automáticamente al servidor de configuración. 

### <a name="5-deployment-planning-select"></a>5 Selección de la planificación de la implementación 

En **¿Completó el planeamiento de implementación?**, seleccione **Lo haré más tarde** en la lista desplegable y, a continuación, haga clic en **Aceptar**.

Cuando termine las 5 secciones de **Preparar infraestructura**, haga clic en **Aceptar**.


## <a name="enable-replication"></a>Habilitar replicación

Habilite la replicación para cada una de las máquinas virtuales que quiera migrar. Cuando se habilita la replicación, Site Recovery instala Mobility Service automáticamente. 

1. Abra el [Azure Portal](htts://portal.azure.com).
1. En la página del almacén, en **Introducción**, haga clic en **Site Recovery**.
2. En **Para máquinas locales y máquinas virtuales de Azure**, haga clic en **Paso 1: Replicar la aplicación**. Complete las páginas del asistente con la información siguiente y haga clic en **Aceptar** en cada página cuando finalice:
    - 1 Configuración del origen
      
    |  |  |
    |-----|-----|
    | Origen: | **local**|
    | Ubicación de origen:| nombre de la instancia de EC2 del servidor de configuración.|
    |Tipo de máquina: | **máquinas físicas**|
    | Servidor de proceso: | seleccione el servidor de configuración en la lista desplegable.|
    
    - 2 Configuración del destino
        
    |  |  |
    |-----|-----|
    | Destino: | deje el valor predeterminado.|
    | Suscripción: | elija la suscripción que ha usado.|
    | Grupo de recursos posterior a la conmutación por error:| debe ser el grupo de recursos que se creó en la sección [Preparar recursos de Azure](#prepare-azure-resources).|
    | Modelo de implementación posterior a la conmutación por error: | elija **Resource Manager**.|
    | Cuenta de almacenamiento: | elija la cuenta de almacenamiento que se creó en la sección [Preparar recursos de Azure](#prepare-azure-resources).|
    | Red de Azure: | elija **Configurar ahora para las máquinas seleccionadas**.|
    | Red de Azure posterior a la conmutación por error: | elija la red que se creó en la sección [Preparar recursos de Azure](#prepare-azure-resources).|
    | Subred: | elija el **valor predeterminado** de la lista desplegable.|
    
    - 3 Selección de máquinas físicas
        
        Haga clic en **+ Máquina física** y, a continuación, rellene los campos **Nombre**, **Dirección IP** y **Tipo de SO** de la instancia de EC2 que quiere migrar. Luego, haga clic en **Aceptar**.
        
    - 4 Propiedades de configuración de propiedades
        
        Seleccione la cuenta que creó en el servidor de configuración de la lista desplegable y haga clic en **Aceptar**.
        
    - 5 Configuración de replicación de la configuración de opciones de replicación
    
        Asegúrese de que la directiva de replicación seleccionada en la lista desplegable es **myReplicationPolicy** y, a continuación, haga clic en **Aceptar**.
        
3. Cuando se complete el asistente, haga clic en **Habilitar replicación**.
        

Puede realizar un seguimiento del progreso del trabajo **Habilitar la protección** en **Supervisión e informes** > **Trabajos** > **Trabajos de Site Recovery**. La máquina estará preparada para la conmutación por error después de que finalice el trabajo **Finalizar la protección** .        
        
Cuando se habilita la replicación para una máquina virtual, los cambios pueden tardar 15 minutos o más en aplicarse y aparecer en el portal.

## <a name="run-a-test-failover"></a>Ejecución de una conmutación por error de prueba

Cuando se ejecuta una conmutación por error de prueba, ocurre lo siguiente:

1. Se ejecuta una comprobación de requisitos previos para garantizar que se dan todas las condiciones necesarias para la conmutación por error.
2. La conmutación por error procesa los datos, de modo que se pueda crear una máquina virtual de Azure. Si se selecciona el último punto de recuperación, se crea un punto de recuperación a partir de los datos.
3. Se crea una VM de Azure mediante los datos procesados en el paso anterior.

En el portal, ejecute la conmutación por error de prueba de la manera siguiente:

1. En la página de su almacén, vaya a **Elementos protegidos** > **Elementos replicados**> haga clic en la máquina virtual > **+ Conmutación por error de prueba**.

2. Seleccione un punto de recuperación para usarlo en la conmutación por error:
    - **Procesado más recientemente**: conmuta por error la VM en el último punto de recuperación procesado por Site Recovery. Se muestra la marca de tiempo. Con esta opción, no se empleó tiempo en el procesamiento de datos, por lo que se proporciona un objetivo de tiempo de recuperación (RTO) bajo.
    - **Más reciente coherente con la aplicación**: esta opción conmuta por error todas las VM en el punto de recuperación más reciente coherente con la aplicación. Se muestra la marca de tiempo.
    - **Personalizado**: seleccione un punto de recuperación.
3. En **Probar conmutación por error**, seleccione la red de Azure de destino a la que se conectarán las máquinas virtuales de Azure después de la conmutación por error. Debe ser la red que se creó en la sección [Preparar recursos de Azure](#prepare-azure-resources).
4. Haga clic en **Aceptar** para iniciar la conmutación por error. Para realizar el seguimiento del progreso, haga clic en la máquina virtual para abrir sus propiedades. También puede hacer clic en el trabajo **Conmutación por error de prueba** en la página del almacén en **Supervisión e informes** > **Trabajos** >
   **Trabajos de Site Recovery**.
5. Una vez finalizada la conmutación por error, la VM de Azure de réplica aparece en Azure Portal > **Virtual Machines**. Compruebe que la VM tiene el tamaño adecuado, que está conectada a la red correspondiente y que se está ejecutando.
6. Ahora debería poder conectarse a la VM replicada en Azure.
7. Para eliminar máquinas virtuales de Azure que se crearon durante la conmutación por error de prueba, haga clic en **Limpiar conmutación por error de prueba** en el plan de recuperación. En **Notas**, registre y guarde las observaciones asociadas a la conmutación por error de prueba.

En algunos escenarios, la conmutación por error requiere un procesamiento adicional que tarda aproximadamente de ocho a diez minutos en completarse. 


## <a name="migrate-to-azure"></a>Migración a Azure

Ejecute una conmutación por error real para las instancias de EC2 a fin de migrarlas a máquinas virtuales de Azure. 

1. En **Elementos protegidos** > **Elementos replicados**, haga clic en las instancias de AWS > **Conmutación por error**.
2. En **Conmutación por error**, seleccione un **Punto de recuperación** en el que realizar la conmutación por error. Seleccione el punto de recuperación más reciente.
3. Seleccione **Shut down machine before beginning failover** (Apagar la máquina antes de comenzar la conmutación por error) si desea que Site Recovery intente apagar las máquinas virtuales de origen antes de desencadenar la conmutación por error. La conmutación por error continúa aunque se produzca un error de cierre. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.
4. Compruebe que la máquina virtual aparece en **Elementos replicados**. 
5. Haga clic con el botón derecho en cada máquina virtual > **Completar la migración**. Con esta acción se completa el proceso de migración, y se detienen la replicación de la máquina virtual de AWS y la facturación de Site Recovery para la VM.

    ![Completar migración](./media/tutorial-migrate-aws-to-azure/complete-migration.png)

> [!WARNING]
> **No cancele una conmutación por error en curso**: antes de iniciar la conmutación por error, se detiene la replicación de la máquina virtual. Si se cancela una conmutación por error en curso, la conmutación por error se detiene, pero no se replica la máquina virtual de nuevo.  


    

## <a name="next-steps"></a>Pasos siguientes

En este tema, ha aprendido cómo migrar instancias de EC2 de AWS en máquinas virtuales de Azure. Para más información acerca de las máquinas virtuales de Azure, continúe con los tutoriales de máquinas virtuales Windows.

> [!div class="nextstepaction"]
> [Tutoriales de máquinas virtuales Windows de Azure](../virtual-machines/windows/tutorial-manage-vm.md)
