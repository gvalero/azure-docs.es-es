---
title: "Implementación del servicio StorSimple Manager | Microsoft Docs"
description: "Aquí encontrará información sobre cómo crear y eliminar el servicio StorSimple Manager en el Portal de Azure clásico, así como una descripción acerca de cómo administrar la clave de registro de servicio."
services: storsimple
documentationcenter: 
author: alkohli
manager: timlt
editor: 
ms.assetid: bc1d5650-275c-42ed-bc77-cdb596f85943
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/03/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ee22a31e9c0ec23d9b042dc894cafe0fc346e742
ms.sourcegitcommit: 0930aabc3ede63240f60c2c61baa88ac6576c508
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/07/2017
---
# <a name="deploy-the-storsimple-manager-service-in-the-azure-classic-portal"></a>Implementación del servicio StorSimple Manager en el Portal de Azure clásico
> [!NOTE]
> El portal clásico para StorSimple está en desuso. Los administradores de dispositivos StorSimple realizarán la transición automáticamente al nuevo Azure Portal según la programación de puesta en desuso. Recibirá un correo electrónico y una notificación del portal en los que se avisa de este paso. Este documento también se retirará pronto. Para ver la versión de este artículo para el nuevo Azure Portal, vaya a [Implementación del servicio StorSimple Manager en el Portal de Azure clásico](storsimple-8000-manage-service.md). Si tiene alguna pregunta sobre este paso, consulte las [preguntas frecuentes de la migración a Azure Portal](storsimple-8000-move-azure-portal-faq.md).


## <a name="overview"></a>Información general
El servicio StorSimple Manager se ejecuta en Microsoft Azure y se conecta a varios dispositivos StorSimple. Después de crear el servicio, puede usarlo para administrar los dispositivos desde el Portal de Microsoft Azure clásico, ejecutándolo en un explorador. Esto permite supervisar todos los dispositivos que están conectados al servicio StorSimple Manager desde una única ubicación central, reduciendo la carga administrativa.

La página de aterrizaje de StorSimple Manager enumera todos los servicios de StorSimple Manager que puede usar para administrar los dispositivos de almacenamiento de StorSimple. Para cada servicio StorSimple Manager, se presenta la siguiente información en la página de StorSimple Manager:

* **Nombre** : el nombre asignado al servicio StorSimple Manager cuando se creó. **El nombre del servicio no se puede cambiar una vez creado el servicio. Esto también se aplica en el caso de otras entidades como dispositivos, volúmenes, contenedores de volúmenes y directivas de copia de seguridad del Portal de Azure clásico.**
* **Estado**: estado del servicio, que puede ser **Activo**, **Creando** o **En línea**.
* **Ubicación** : ubicación geográfica en la que se implementará el dispositivo StorSimple.
* **Suscripción** : suscripción de facturación asociada a su servicio.

Las tareas comunes que se pueden realizar a través de la página de StorSimple Manager son:

* Crear un servicio
* Eliminar un servicio
* Obtener la clave de registro del servicio
* Volver a generar la clave de registro de servicio

Este tutorial describe cómo realizar cada una de estas tareas.

## <a name="create-a-service"></a>Crear un servicio
Utilice la opción **Creación rápida** para crear un servicio de StorSimple Manager si desea implementar el dispositivo StorSimple. Para crear un servicio, debe tener:

* Una suscripción con un contrato Enterprise
* Una cuenta de Almacenamiento de Microsoft Azure activa.
* La información de facturación que se usa para la administración de acceso

También puede generar una cuenta de almacenamiento predeterminada cuando se crea el servicio.

Un único servicio puede administrar varios dispositivos. Sin embargo, un dispositivo no puede abarcar varios servicios. Una gran empresa puede tener varias instancias de servicio para trabajar con distintas suscripciones, organizaciones o incluso las ubicaciones de implementación. Tenga en cuenta que necesita instancias separadas del servicio StorSimple Manager para administrar matrices virtuales de StorSimple y dispositivos de la serie StorSimple 8000.

> [!IMPORTANT] 
> Si tiene un servicio sin usar creado (no se realizaron operaciones en este recurso) antes de agosto de 2016, no puede administrarse a través de Azure Portal o el Portal de Azure clásico. Le recomendamos que cree un nuevo servicio en Azure Portal.

Realice los siguientes pasos para crear un servicio.

[!INCLUDE [storsimple-create-new-service](../../includes/storsimple-create-new-service.md)]

## <a name="delete-a-service"></a>Eliminar un servicio
Antes de eliminar un servicio, asegúrese de que no lo esté utilizando ningún dispositivo conectado. Si se está utilizando el servicio, desactive los dispositivos conectados. La operación de desactivación eliminará la conexión entre el dispositivo y el servicio, pero conservará los datos del dispositivo en la nube.

> [!IMPORTANT] 
> Después de eliminar un servicio, no se puede revertir la operación. Cualquier dispositivo que estaba utilizando el servicio deberá restablecerse a los valores de fábrica para que pueda ser usado con otro servicio. En este escenario, se perderán los datos locales del dispositivo, así como la configuración.

Realice los siguientes pasos para eliminar un servicio.

### <a name="to-delete-a-service"></a>Para eliminar un servicio
1. En la página **Servicio StorSimple Manager** , haga clic en el servicio que desee eliminar.
2. En la parte inferior de la página, haga clic en **Eliminar** .
3. Haga clic en **Sí** en la notificación de confirmación. Es posible que el servicio tarde unos minutos en eliminarse.

## <a name="get-the-service-registration-key"></a>Obtener la clave de registro del servicio
Después de haber creado correctamente un servicio, deberá registrar el dispositivo StorSimple con el servicio. Para registrar el primer dispositivo StorSimple, necesitará la clave de registro del servicio. Para registrar dispositivos adicionales con un servicio existente StorSimple, necesitará la clave de registro y la clave de cifrado de datos de servicio (que se genera en el primer dispositivo durante el registro). Para obtener más información acerca de la clave de cifrado de datos de servicio, consulte [Seguridad de StorSimple](storsimple-security.md). Puede obtener la clave de registro accediendo a la **Clave de registro** on the **Servicios** .

Realice los pasos siguientes para obtener la clave de registro del servicio.

[!INCLUDE [storsimple-get-service-registration-key](../../includes/storsimple-get-service-registration-key.md)]

Mantenga la clave de registro del servicio en una ubicación segura. Necesitará esta clave, así como la clave de cifrado de datos de servicio para registrar dispositivos adicionales con este servicio. Después de obtener la clave de registro del servicio, deberá configurar el dispositivo a través de la interfaz de Windows PowerShell para StorSimple.

Para obtener más información sobre cómo usar esta clave de registro, consulte [Paso 3: Configurar y registrar el dispositivo a través de Windows PowerShell para StorSimple](storsimple-deployment-walkthrough.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple).

## <a name="regenerate-the-service-registration-key"></a>Volver a generar la clave de registro de servicio
Será necesario volver a generar una clave de registro del servicio si es necesario para realizar la rotación de claves o si ha cambiado la lista de administradores de servicios. Cuando se regenera la clave, la nueva clave se utiliza solo para registrar dispositivos posteriores. Los dispositivos que ya se han registrado no se ven afectados por este proceso.

Realice los pasos siguientes para volver a generar una clave de registro de servicio.

### <a name="to-regenerate-the-service-registration-key"></a>Para volver a generar la clave de registro de servicio
1. En la página **Servicio StorSimple Manager**, haga clic en **Clave de registro**.
2. En el cuadro de diálogo **Clave de registro del servicio**, haga clic en **Regenerar**.
3. Aparecerá un mensaje de confirmación. Haga clic en **Aceptar** para continuar con la regeneración.
4. Aparecerá una nueva clave de registro de servicio.
5. Copie esta clave y guárdela para registrar los dispositivos nuevos con este servicio.
6. Haga clic en el icono de verificación  ![Icono de marca de verificación](./media/storsimple-manage-service/HCS_CheckIcon.png) para cerrar este cuadro de diálogo.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el [proceso de implementación de StorSimple](storsimple-deployment-walkthrough-u2.md).
* Obtenga más información sobre cómo [administrar su cuenta de almacenamiento de StorSimple](storsimple-manage-storage-accounts.md).
* Obtenga más información sobre cómo [usar el servicio StorSimple Manager para administrar su dispositivo StorSimple](storsimple-manager-service-administration.md).
