---
title: "Configurar la recuperación ante desastres para máquinas virtuales de VMware locales en Azure con Azure Site Recovery | Microsoft Docs"
description: "Aprenda a configurar la recuperación ante desastres para máquinas virtuales de VMware locales en Azure con el servicio Azure Site Recovery."
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/11/2017
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 5810ff908d48fc4ff742d734e7c2457fdfe8cb03
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/11/2017
---
# <a name="set-up-disaster-recovery-to-azure-for-on-premises-vmware-vms"></a>Configurar la recuperación ante desastres para máquinas virtuales de VMware locales en Azure

Este tutorial muestra cómo configurar la recuperación ante desastres para máquinas virtuales de VMware locales con Windows en Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Especificar el origen y destino de la replicación.
> * Configurar el entorno de replicación de origen, incluidos los componentes de Site Recovery local y el entorno de replicación de destino.
> * Creación de una directiva de replicación
> * Habilitación de la replicación para una máquina virtual

Este es el tercer tutorial de una serie. En este tutorial se da por hecho que ya ha realizado las tareas de los tutoriales anteriores:

1. [Preparación de Azure](tutorial-prepare-azure.md)
2. [Preparación de VMware local](tutorial-prepare-on-premises-vmware.md)

Antes de empezar, es útil [revisar la arquitectura](concepts-vmware-to-azure-architecture.md) del escenario de recuperación ante desastres.


## <a name="select-a-replication-goal"></a>Selección de un objetivo de replicación

1. En **Almacenes de Recovery Services**, haga clic en el nombre del almacén y **ContosoVMVault**.
2. En **Introducción**, haga clic en Site Recovery. Haga clic en **Preparar infraestructura**.
3. En **Objetivo de protección** > **¿Dónde se encuentran sus máquinas?**, seleccione **Local**.
4. En **¿Dónde quiere replicar las máquinas?**, seleccione **En Azure**.
5. En **¿Las máquinas están virtualizadas?**, seleccione **Yes, with VMware vSphere Hypervisor** (Sí, con VMware vSphere Hypervisor). y, a continuación, haga clic en **Aceptar**.

## <a name="set-up-the-source-environment"></a>Configuración del entorno de origen

Para configurar el entorno de origen, descargue el archivo de instalación unificada de Site Recovery. Ejecute el programa de instalación para instalar los componentes de Site Recovery local, registrar los servidores de VMware en el almacén y detectar las máquinas virtuales locales.

### <a name="verify-on-premises-site-recovery-requirements"></a>Comprobación de los requisitos de Site Recovery local

Es necesaria una máquina de VMware local, única y de alta disponibilidad para hospedar los componentes de Site Recovery local. Los componentes incluyen el servidor de configuración, el servidor de procesos y el servidor de destino maestro.

- El servidor de configuración coordina la comunicación entre el entorno local y Azure, además de administrar la replicación de datos.
- El servidor de procesos actúa como puerta de enlace de replicación. Recibe datos de replicación, los optimiza con almacenamiento en caché, compresión y cifrado y los envía al almacenamiento de Azure. El servidor de procesos también instala Mobility Service en las máquinas virtuales que quiere replicar, además de realizar la detección automática de las máquinas virtuales de VMware locales.
- El servidor de destino maestro controla los datos de replicación durante la conmutación por recuperación desde Azure.

La máquina virtual debería cumplir los siguientes requisitos:

| **Requisito** | **Detalles** |
|-----------------|-------------|
| Número de núcleos de la CPU| 8 |
| RAM | 12 GB |
| Número de discos | 3: disco del sistema operativo, disco de memoria caché del servidor de procesos, unidad de retención (para la conmutación por recuperación) |
| Espacio libre en el disco (caché del servidor de procesos) | 600 GB |
| Espacio libre en el disco (disco de retención) | 600 GB |
| Versión del sistema operativo | Windows Server 2012 R2 |
| Configuración regional del sistema operativo | Español (es-es) |
| Versión de VMware vSphere PowerCLI | [PowerCLI 6.0](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1 "PowerCLI 6.0") |
| Roles de Windows Server | No habilite estos roles: Active Directory Domain Services, Internet Information Services, Hyper-V |
| Tipo de NIC | VMXNET3 |
| Tipo de dirección IP | estática |
| Puertos | 443 (orquestación del canal de control)<br/>9443 (Transporte de datos)|

Además: 
- Asegúrese de que el reloj del sistema de la máquina virtual esté sincronizado con un servidor horario. La hora debe estar sincronizada en 15 minutos. Si es mayor, el programa de instalación produce un error.
el programa de instalación produce un error.
- Asegúrese de que la máquina virtual del servidor de configuración pueda acceder a estas direcciones URL:

    [!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]
    
- Asegúrese de que las reglas de firewall basadas en direcciones IP permitan la comunicación con Azure.
    - Permita los [intervalos IP del centro de datos de Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653), el puerto 443 (HTTPS) y el puerto 9443 (replicación de datos).
    - Permita los intervalos de direcciones IP correspondientes a la región de Azure de su suscripción y del oeste de EE. UU. (se usan para Access Control y para Identity Management).


### <a name="download-the-site-recovery-unified-setup-file"></a>Descarga del archivo de instalación unificada de Site Recovery

1. En el almacén **Preparar infraestructura**, haga clic en **Origen**.
1. En **Preparar origen**, haga clic en **+Servidor de configuración**.
2. En **Agregar servidor**, compruebe que aparezca **Servidor de configuración** en **Tipo de servidor**.
3. Descargue el archivo de instalación unificada de Site Recovery.
4. Descargue la clave de registro del almacén. Se le solicitará cuando ejecute la instalación unificada. La clave será válida durante cinco días a partir del momento en que se genera.

   ![Configurar origen](./media/tutorial-vmware-to-azure/source-settings.png)

### <a name="set-up-the-configuration-server"></a>Configuración del servidor de configuración

1. Ejecute el archivo de instalación unificada.
2. En **Before You Begin** (Antes de comenzar), seleccione **Install the configuration server and process server** (Instalar el servidor de configuración y el servidor de procesos) y luego haga clic en **Siguiente**.

3. En **Third-Party Software License** (Licencia de software de terceros), haga clic en **Acepto** para descargar e instalar MySQL y luego haga clic en **Siguiente**.

4. En **Registro**, seleccione la clave de registro que descargó del almacén.

5. En **Configuración de Internet**, especifique cómo se conecta el proveedor que se ejecuta en el servidor de configuración a Azure Site Recovery a través de Internet.

   - Si quiere conectarse con el proxy configurado actualmente en la máquina, seleccione **Conectar con Azure Site Recovery con un servidor proxy**.
   - Si quiere que el proveedor se conecte directamente, seleccione **Conectar directamente con Azure Site Recovery sin un servidor proxy**.
   - Si el proxy existente requiere autenticación, o si quiere usar un proxy personalizado para la conexión del proveedor, seleccione **Connect with custom proxy settings** (Conectarse con una configuración de proxy personalizada) y especifique la dirección, el puerto y las credenciales.

   ![Firewall](./media/tutorial-vmware-to-azure/combined-wiz4.png)

6. En **Comprobación de requisitos previos**, el programa de instalación ejecuta una comprobación para asegurarse de que se pueda ejecutar la instalación. Si aparece una advertencia sobre la **comprobación de la sincronización de hora global**, compruebe que la hora del reloj del sistema (configuración de **fecha y hora**) es la misma que la de la zona horaria.

   ![Requisitos previos](./media/tutorial-vmware-to-azure/combined-wiz5.png)

7. En **MySQL Configuration** (Configuración de MySQL), cree credenciales para iniciar sesión en la instancia de servidor MySQL que se va a instalar.

8. En **Detalles del entorno**, seleccione **Sí** para proteger las máquinas virtuales de VMware. El programa de instalación comprueba que PowerCLI 6.0 esté instalado.

9. En **Ubicación de instalación**, seleccione dónde quiere instalar los archivos binarios y almacenar la memoria caché. La unidad que seleccione debe tener al menos 5 GB de espacio disponible en disco, pero se recomienda usar una unidad de memoria caché con 600 GB o más de espacio libre.

10. En **Selección de red**, especifique el agente de escucha (adaptador de red y puerto SSL) en el que el servidor de configuración enviará y recibirá los datos de replicación. El puerto 9443 es el que se usa de forma predeterminada para enviar y recibir el tráfico de replicación, pero puede modificar este número de puerto para adecuarlo a los requisitos de su entorno. También se abre el puerto 443, que se usa para coordinar las operaciones de replicación. No use el puerto 443 para enviar o recibir tráfico de replicación.

11. En **Resumen**, revise la información y haga clic en **Instalar**. El programa de instalación instala el servidor de configuración y con él registra el servicio Azure Site Recovery.

    ![Resumen](./media/tutorial-vmware-to-azure/combined-wiz10.png)

    Se genera una frase de contraseña cuando finaliza la instalación. La necesitará al habilitar la replicación, así que cópiela y manténgala en una ubicación segura. El servidor se muestra en el panel **Configuración** > **Servidores** del almacén.

### <a name="configure-automatic-discovery"></a>Configuración de la detección automática

Para detectar máquinas virtuales, el servidor de configuración tiene que conectarse a servidores de VMware locales. Para los fines de este tutorial, agregue el servidor vCenter, o hosts de vSphere, con una cuenta que tenga privilegios de administrador en el servidor. Creó esta cuenta en el [tutorial anterior](tutorial-prepare-on-premises-vmware.md). 

Para agregar la cuenta:

1. En la máquina virtual del servidor de configuración, inicie **CSPSConfigtool.exe**. Esta herramienta está disponible como acceso directo en el escritorio y se encuentra en la carpeta *ubicación de instalación*\home\svsystems\bin.

2. Haga clic en **Administrar cuentas** > **Agregar cuenta**.

   ![Agregar cuenta](./media/tutorial-vmware-to-azure/credentials1.png)

3. En **Detalles de la cuenta**, agregue la cuenta que se utilizará para la detección automática.

   ![Detalles](./media/tutorial-vmware-to-azure/credentials2.png)

Para agregar el servidor de VMware:

1. Abra [Azure Portal](https://portal.azure.com) y haga clic en **Todos los recursos**.
2. Haga clic en el almacén de Recovery Service denominado **ContosoVMVault**.
3. Haga clic en **Site Recovery** > **Preparar infraestructura** > **Origen**.
4. Seleccione **+vCenter** para conectarse a un servidor vCenter o un host ESXi de vSphere.
5. En **Agregar servidor vCenter**, especifique un nombre descriptivo para el servidor. Luego, especifique la dirección IP o FQDN.
6. Deje el puerto establecido en 443, a menos que los servidores de VMware escuchen las solicitudes en otro puerto.
7. Seleccione la cuenta que se va a usar para conectarse al servidor. Haga clic en **Aceptar**.

Site Recovery se conecta a los servidores de VMware con la configuración especificada y detecta máquinas virtuales.

> [!NOTE]
> El nombre de la cuenta podría tardar 15 minutos o más en aparecer en el portal. Para que se actualice inmediatamente, haga clic en **Servidores de configuración** > ***nombre de servidor*** > **Actualizar servidor**.

## <a name="set-up-the-target-environment"></a>Configuración del entorno de destino

Seleccione y compruebe los recursos de destino.

1. Haga clic en **Preparar infraestructura** > **Destino** y seleccione la suscripción de Azure que quiere usar.
2. Especifique si el modelo de implementación de destino está basado en Resource Manager o si es el clásico.
3. Site Recovery comprueba que tiene una o más cuentas de almacenamiento y redes compatibles.

   ![Destino](./media/tutorial-vmware-to-azure/storage-network.png)

## <a name="create-a-replication-policy"></a>Creación de una directiva de replicación

1. Abra [Azure Portal](https://portal.azure.com) y haga clic en **Todos los recursos**.
2. Haga clic en el almacén de Recovery Service denominado **ContosoVMVault**.
3. Para crear una directiva de replicación, haga clic en **Infraestructura de Site Recovery** > **Directivas de replicación** > **+Directiva de replicación**.
4. En **Crear la directiva de replicación**, especifique un nombre de directiva **VMwareRepPolicy**.
5. En **Umbral de RPO**, use el valor predeterminado de 60 minutos. Este valor define la frecuencia de creación de puntos de recuperación. Se genera una alerta cuando la replicación continua supera este límite.
6. En **Retención de punto de recuperación**, use el valor predeterminado de 24 horas como duración del período de retención de cada punto de recuperación. En este tutorial se selecciona 72 horas. Las máquinas virtuales replicadas se pueden recuperar a cualquier momento de un período.
7. En **Frecuencia de instantánea coherente con la aplicación**, use el valor predeterminado de 60 minutos como frecuencia de creación de instantáneas coherentes con la aplicación. Haga clic en **Aceptar** para crear la directiva.

   ![Directiva](./media/tutorial-vmware-to-azure/replication-policy.png)

La directiva se asocia automáticamente al servidor de configuración. De forma predeterminada, se crea automáticamente una directiva correspondiente para la conmutación por recuperación. Por ejemplo, si la directiva de replicación es **rep-policy**, la directiva de conmutación por recuperación será **rep-policy-failback**. Esta directiva no se usa hasta que se inicie una conmutación por recuperación desde Azure.

## <a name="enable-replication"></a>Habilitar replicación

Site Recovery instala Mobility Service cuando se habilita la replicación para una máquina virtual. Los cambios pueden tardar 15 minutos o más en aplicarse y aparecer en el portal.

Habilite la replicación como se indica a continuación:

1. Haga clic en **Replicar la aplicación** > **Origen**.
2. En **Origen**, seleccione el servidor de configuración.
3. En **Tipo de máquina**, seleccione **Máquinas virtuales**.
4. En **vCenter/vSphere Hypervisor**, seleccione el servidor vCenter que administra el host de vSphere, o bien seleccione el host.
5. Seleccione el servidor de procesos (servidor de configuración). Luego haga clic en **Aceptar**.
6. En **Destino**, seleccione la suscripción y el grupo de recursos donde desee crear las máquinas virtuales conmutadas por error. Elija el modelo de implementación que desee usar en Azure (clásico o de Resource Manager) para las máquinas virtuales conmutadas por error.
7. Seleccione la cuenta de Azure Storage que desea usar para los datos de replicación.
8. Seleccione la red y la subred de Azure a la que se conectarán las máquinas virtuales de Azure cuando se creen después de la conmutación por error.
9. Seleccione la opción **Configurar ahora para las máquinas seleccionadas** con el fin de aplicar la configuración de red a todas las máquinas que seleccione para su protección. Seleccione **Configurar más tarde** para seleccionar la red de Azure por máquina.
10. En **Máquinas virtuales** > **Seleccionar máquinas virtuales**, haga clic en cada máquina que desea replicar y selecciónela. Solo puede seleccionar aquellas máquinas en las que se pueda habilitar la replicación. y, a continuación, haga clic en **Aceptar**.
11. En **Propiedades** > **Configurar propiedades**, seleccione la cuenta que va a usar el servidor de procesos para instalar automáticamente Mobility Service en el equipo.
12. En **Configuración de la replicación** > **Establecer configuración de replicación**, compruebe que se ha seleccionado la directiva de replicación correcta.
13. Haga clic en **Enable Replication**.

Puede hacer un seguimiento del progreso del trabajo **Habilitar protección** en **Configuración** > **Trabajos** > **Trabajos de Site Recovery**. La máquina estará preparada para la conmutación por error después de que finalice el trabajo **Finalizar la protección** .

Para supervisar las máquinas virtuales que agregue, puede comprobar la última hora de detección de máquinas virtuales en **Servidores de configuración**
> **Último contacto**. Para agregar máquinas virtuales sin esperar a la detección programada, resalte el servidor de configuración (no haga clic en él) y haga clic en **Actualizar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtención de detalles de recuperación ante desastres](site-recovery-test-failover-to-azure.md)
