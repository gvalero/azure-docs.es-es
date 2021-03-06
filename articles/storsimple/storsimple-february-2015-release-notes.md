---
title: "Notas de la versión de StorSimple 8000, actualización 0.3 | Microsoft Docs"
description: "Describe las nuevas características y soluciones, problemas abiertos y soluciones alternativas de la versión de Microsoft Azure StorSimple de febrero de 2015 (Actualización 0.3)."
services: storsimple
documentationcenter: NA
author: SharS
manager: carmonm
editor: 
ms.assetid: b01bfd04-f9f8-45f4-ade8-95ac2b979e6a
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: v-sharos
ms.openlocfilehash: 87a0a59d0c980baa87aa5d7f5f0e8098884b5ec9
ms.sourcegitcommit: 0930aabc3ede63240f60c2c61baa88ac6576c508
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/07/2017
---
# <a name="storsimple-8000-series-update-03-release-notes---february-2015"></a>Notas de la versión 0.3 de la actualización de la serie StorSimple 8000 - febrero de 2015
> [!NOTE]
> El portal clásico para StorSimple está en desuso. Los administradores de dispositivos StorSimple realizarán la transición automáticamente al nuevo Azure Portal según la programación de puesta en desuso. Recibirá un correo electrónico y una notificación del portal en los que se avisa de este paso. Este documento también se retirará pronto. Si tiene alguna pregunta sobre este paso, vea las [preguntas frecuentes de la migración a Azure Portal](storsimple-8000-move-azure-portal-faq.md).

## <a name="overview"></a>Información general
Las siguientes notas de la versión identifican los problemas críticos pendientes en la actualización 0.3 de la serie StorSimple 8000 publicada en febrero de 2015. También contienen una lista de las actualizaciones de software y firmware de StorSimple incluidas en esta versión. Esta es la tercera versión que aparece después de que la versión de lanzamiento de la serie StorSimple 8000 Series se pusiera a disposición general en julio de 2014.

Esta actualización no cambia la versión de software del dispositivo de la actualización de enero. Sigue siendo la versión 6.3.9600.17312. Puede confirmar que la actualización está instalada mediante la comprobación de la fecha de la **Última actualización** . Si la fecha es el 10/02/2015 o posterior, la actualización está correctamente instalada.  

Se recomienda buscar y aplicar las actualizaciones disponibles inmediatamente después de instalar el dispositivo StorSimple. También puede activar las actualizaciones automáticas para descargar e instalar actualizaciones de alta prioridad de Microsoft en cuanto se publiquen. Para obtener más información, consulte [Actualizar dispositivo StorSimple](storsimple-update-device.md).  

Revise la información contenida en las notas de la versión antes de implementar la actualización de la solución de StorSimple.  

> [!IMPORTANT]
> * Para instalar la actualización de febrero, use el servicio de Administrador de StorSimple y no Windows PowerShell para StorSimple.   
> * Tarda aproximadamente una hora en instalar esta actualización. Sin embargo, si va a instalar actualizaciones acumulativas, el proceso puede tardar unas 3 horas en completarse.  
> * La versión de febrero de StorSimple no contiene actualizaciones para el dispositivo virtual de StorSimple. Puede aplicar cualquier actualización disponible de Windows en el dispositivo virtual, incluidas revisiones de seguridad recientes, pero no verá un cambio de versión en el dispositivo virtual.  
> 
> 

Asegúrese de que se cumplen los siguientes requisitos previos antes de actualizar el dispositivo StorSimple.  

* Asegúrese de que ambos controladores de dispositivo se están ejecutando antes de buscar actualizaciones. Si no se está ejecutando uno de los controladores, se producirá un error en la búsqueda. Para comprobar que los controladores están en buen estado, vaya a **Estado de hardware** en la página **Mantenimiento**. Si algún componente **Requiere atención**, póngase en contacto con el soporte técnico de Microsoft antes de continuar.
* Asegúrese de que las IP fijas del controlador 0 y el controlador 1 sean enrutables y puedan conectarse a Internet, ya que se usan para el mantenimiento de las actualizaciones del dispositivo. Puede usar el [cmdlet Test-Connection](https://technet.microsoft.com/library/hh849808.aspx) para hacer ping a una dirección conocida fuera de la red, como outlook.com, para comprobar que el controlador tiene conectividad a la red externa.
* Asegúrese de que los puertos 80 y 443 están disponibles en el dispositivo StorSimple para la comunicación saliente. Para obtener más información, consulte [Requisitos de red para el dispositivo StorSimple](storsimple-system-requirements.md#networking-requirements-for-your-storsimple-device).
* Si la versión del software de dispositivo es anterior a 6.3.9600.17312 (actualización de octubre de 2014), deshabilite los puertos Data 2 y Data 3, en caso de que estén habilitados, antes de iniciar la actualización. Deje los puertos Data 2 o Data 3 habilitados cuando la aplicación de la actualización puede hacer que el controlador del dispositivo entre en modo de recuperación. Tenga en cuenta que, al deshabilitar las interfaces de red, todos los volúmenes asociados se desconectarán y se interrumpirá la E/S mientras dure la actualización.  

## <a name="whats-new-in-the-february-release"></a>Novedades de la versión de febrero
Esta actualización contiene una corrección para el problema del restablecimiento de fábrica que se produjo en los dispositivos que se habían actualizado desde la versión GA a la versión de octubre de 2014. Para obtener más información, consulte [Problemas corregidos en esta versión](#issues-fixed-in-the-february-release).   

Esta actualización no contiene características o funcionalidades nuevas.  

## <a name="issues-fixed-in-the-february-release"></a>Problemas corregidos en la versión de febrero
En la tabla siguiente se describe el problema que se corrigió en esta actualización.  

| N.º | Característica | Problema | Se aplica a un dispositivo físico | Se aplica a un dispositivo virtual |
| --- | --- | --- | --- | --- |
| 1 |Restablecimiento de fábrica |Intenta realizar un restablecimiento de fábrica en un dispositivo que originalmente tenía instalada la versión GA (versión 6.3.9600.17215), pero se actualiza a la versión de octubre (versión 6.3.9600.17312). Se produce un error en el restablecimiento de fábrica y el dispositivo se vuelve inestable. |Sí |No |

## <a name="known-issues-in-the-february-release"></a>Problemas conocidos de la versión de febrero
En la tabla siguiente se proporciona un resumen de los problemas conocidos de esta versión.

| Nº | Característica | Problema | Comentarios/solución alternativa | Se aplica a un dispositivo físico | Se aplica a un dispositivo virtual |
| --- | --- | --- | --- | --- | --- |
| 1 |Restablecimiento de fábrica |En algunos casos, al realizar un restablecimiento de fábrica, el dispositivo StorSimple puede bloquearse y mostrar este mensaje: **Restablecimiento de fábrica en curso (fase 8)**. Esto sucede si presiona CTRL+C mientras el cmdlet está en curso. |No presione CTRL+C después de iniciar un restablecimiento de fábrica. Si ya está en este estado, póngase en contacto con el soporte técnico de Microsoft para conocer los pasos siguientes. |Sí |No |
| 2 |Cuórum de disco |En raras ocasiones, si se desconecta la mayoría de los discos en el revestimiento de EBOD de un dispositivo 8600 y no se produce un cuórum de disco, el bloque de almacenamiento se desconectará. Seguirá desconectado incluso si se vuelven a conectar los discos. |Necesitará reiniciar el dispositivo. Si el problema persiste, póngase en contacto con el soporte técnico de Microsoft para conocer los pasos siguientes. |Sí |No |
| 3 |Errores de instantánea en la nube |En raras ocasiones, una instantánea en la nube puede producir el error **Se ha alcanzado el límite máximo de copia de seguridad**. Esto ocurre si se superan los 255 clones en línea en el mismo dispositivo, procedentes del mismo volumen original eliminado. | |Sí |Sí |
| 4 |Identificador de controlador incorrecto |Cuando se realiza un reemplazo de controlador, el controlador 0 puede aparecer como controlador 1. Durante el reemplazo de controlador, cuando se carga la imagen desde el nodo del mismo nivel, el identificador de controlador puede mostrarse inicialmente como el identificador del controlador del mismo nivel. En raras ocasiones, este comportamiento también puede aparecer después del reinicio del sistema. |No se requiere ninguna acción del usuario. La situación se solucionará una vez completado el reemplazo del controlador. |Sí |No |
| 5 |Gráficos de supervisión de dispositivos |En el servicio de Administrador de StorSimple, los gráficos de supervisión de dispositivos no funcionan cuando está habilitada la autenticación básica o NTLM en la configuración del servidor proxy para el dispositivo. |Modifique la configuración de proxy web para el dispositivo registrado con el servicio de Administrador de StorSimple para que la autenticación se establezca en NONE. Para ello, ejecute el cmdlet Set-HcsWebProxy de Windows PowerShell para StorSimple. |Sí |Sí |
| 6 |Cuentas de almacenamiento |El uso del servicio de almacenamiento para eliminar la cuenta de almacenamiento es un escenario no admitido. Esto provocará una situación en la que no se pueden recuperar los datos de usuario. | |Sí |Sí |
| 7 |Conmutación por error del dispositivo |No se admiten varias conmutaciones por error de un contenedor de volúmenes del mismo dispositivo de origen a diferentes dispositivos de destino.    La conmutación por error de un único dispositivo inactivo a varios dispositivos hará que los contenedores de volúmenes del primer dispositivo conmutado por error pierdan la propiedad de los datos. Después de este tipo de conmutación por error, estos contenedores de volúmenes aparecerán o se comportarán de forma diferente cuando se visualicen en el Portal de Azure clásico. | |Sí |No |
| 8 |Instalación |Durante la instalación del adaptador de StorSimple para SharePoint, deberá proporcionar una dirección IP de dispositivo para que la instalación finalice correctamente. | |Sí |No |
| 9 |Proxy web |Si la configuración de proxy web tiene HTTPS como protocolo especificado, la comunicación de dispositivo a servicio se verá afectada y el dispositivo se desconectará. También se generarán paquetes de compatibilidad en el proceso, que consumen muchos recursos en el dispositivo. |Asegúrese de que la dirección URL del proxy web tiene HTTP como protocolo especificado. Obtenga más información sobre cómo [Configurar el proxy web para el dispositivo](storsimple-configure-web-proxy.md). |Sí |No |
| 10 |Proxy web |Si configura y habilita el proxy web en un dispositivo registrado, será necesario reiniciar el controlador activo en el dispositivo. | |Sí |No |
| 11 |Latencia alta de la nube y alta carga de trabajo de E/S |Cuando el dispositivo StorSimple encuentra una combinación de latencias muy altas de la nube (del orden de segundos) y alta carga de trabajo de E/S, los volúmenes del dispositivo pasan a un estado degradado y las operaciones de E/S pueden fallar con el error «el dispositivo no está listo». |Necesitará reiniciar los controladores de dispositivo de forma manual o realizar una conmutación por error del dispositivo para recuperarse de esta situación. |Sí |No |

## <a name="physical-device-updates-in-the-february-release"></a>Actualizaciones del dispositivo físico en la versión de febrero
Esta actualización corrige el problema del restablecimiento de fábrica que se produjo en los dispositivos que se habían actualizado desde la versión GA a la versión de octubre de 2014. No contiene otras actualizaciones en el dispositivo StorSimple.  

## <a name="serial-attached-scsi-sas-controller-and-firmware-updates-in-the-february-release"></a>Controlador SCSI conectado en serie (SAS) y actualizaciones de firmware en la versión de febrero
Esta versión no contiene las actualizaciones para el controlador SCSI conectado en serie (SAS) o el firmware. La actualización del controlador se introdujo en la versión de octubre de 2014.  

## <a name="virtual-device-updates-in-the-february-release"></a>Actualizaciones del dispositivo virtual en la versión de febrero
Esta versión no contiene actualizaciones para el dispositivo virtual. La aplicación de esta actualización no cambiará la versión de software de un dispositivo virtual.

