# Laboratorio: acceso, arquitectura y seguridad

Fecha de documentación: **2026-11-08**.

## Estado y alcance

Este documento conserva información proporcionada por el usuario y hallazgos de una auditoría anterior. **No constituye una nueva auditoría ni acredita endurecimiento aplicado.** Los hallazgos históricos no se han revalidado y pueden cambiar. La fecha de documentación no se presenta como fecha de ejecución de aquella auditoría.

La decisión actual es mantener un laboratorio de aprendizaje **sin Docker ni sandbox**. Se describen riesgos y una posible migración futura; no se modifica configuración, permisos, usuarios, servicios ni red. No se realizan instalaciones, pruebas de elevación ni otras pruebas de seguridad.

## Arquitectura confirmada por el usuario

```text
Mac del usuario
  ├─ Cliente SSH ── Internet ── Security Group de AWS ── SSH en EC2
  │                            permite SSH solo desde
  │                            la IP pública del usuario
  │
  └─ Navegador: localhost:18789
          │
          └─ Túnel SSH hacia EC2
                 └─ Gateway: 127.0.0.1:18789 (loopback)
                        └─ Laboratorio sin Docker ni sandbox
```

Según el usuario, no se usa Tailscale ni proxy inverso y el puerto del Gateway no se publica en Internet. La restricción del Security Group es una declaración del usuario, **no una comprobación de AWS**. El túnel mantiene el acceso humano al Dashboard sin publicar el Gateway.

## Evidencia, procedencia e impacto

| Evidencia o decisión conservada                                                                                                        | Procedencia y límites                                                                                                                             | Impacto o interpretación                                                                                                                                     |
| -------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| EC2 accedida mediante SSH desde un Mac; Security Group permite SSH solo desde la IP pública del usuario.                               | Confirmado por el usuario; reglas de AWS no auditadas.                                                                                            | Restricción declarada del acceso de red; no prueba independiente de exposición efectiva.                                                                     |
| Dashboard en localhost:18789 mediante túnel SSH hacia Gateway 127.0.0.1:18789; sin Tailscale, proxy inverso ni puerto Gateway público. | Confirmado por el usuario.                                                                                                                        | Arquitectura de acceso que se propone conservar; no demuestra por sí sola aislamiento de las herramientas locales.                                           |
| Laboratorio sin Docker ni sandbox para aprendizaje.                                                                                    | Decisión actual del usuario al documentar.                                                                                                        | Se acepta conscientemente la ausencia de aislamiento por sandbox; las instrucciones de texto no son una barrera técnica.                                     |
| Ubuntu 24.04.5 en VM AWS; cuenta ubuntu, UID 1000, grupos sudo y lxd.                                                                  | Auditoría anterior, no revalidada.                                                                                                                | La identidad y pertenencia a grupos requieren revisión autorizada antes de una migración; lxd es un grupo privilegiado que no debe heredar la futura cuenta. |
| sudo -n -l reportó (ALL) NOPASSWD: ALL; no se ejecutó root.                                                                            | Auditoría anterior; se consultó la política, no se ensayó elevación.                                                                              | Riesgo alto de elevación desde la cuenta de ejecución. Deshabilitar tools.elevated no elimina los privilegios que el sistema concede a esa cuenta.           |
| exec: perfil full, host auto, seguridad full, ask off, fallback deny; comandos inocuos ejecutados sin aprobación.                      | Auditoría anterior. El preview principal no incluía overrides /exec; no se afirma cobertura de todas las variantes.                               | Superficie de ejecución amplia en lo observado. fallback deny no debe interpretarse como aprobación obligatoria ni como aislamiento.                         |
| Sandbox: runtime direct, mode off.                                                                                                     | Auditoría anterior, no revalidada.                                                                                                                | Ejecución directa, sin barrera de sandbox en aquella observación.                                                                                            |
| tools.fs.workspaceOnly sin configurar; lectura fuera del workspace demostrada con /etc/os-release y /usr/lib/os-release.               | Auditoría anterior; alcance de prueba limitado a esos archivos.                                                                                   | El workspace no era una frontera de lectura demostrada. No se infiere acceso universal ni lectura de secretos.                                               |
| Permisos de escritura sobre el home y la configuración de OpenClaw observados por metadatos.                                           | Auditoría anterior; no se ensayaron escrituras.                                                                                                   | Posible capacidad de alterar archivos propios y configuración; no equivale a modificación realizada ni a comprobación exhaustiva de acceso efectivo.         |
| Gateway en loopback con autenticación por token; directorio de estado 0700 y configuración 0600.                                       | Auditoría anterior, no revalidada; sin valores de credenciales.                                                                                   | Controles de acceso observados, no garantías frente a procesos que comparten la identidad propietaria ni frente a elevación.                                 |
| Auditoría: 0 critical, 1 warn por trustedProxies vacío y 1 info.                                                                       | Resultado histórico, no certificado del estado actual.                                                                                            | Sin proxy inverso, esa advertencia no justifica por sí misma cambiar trustedProxies. El conteo no descarta otros riesgos.                                    |
| Escucha SSH en wildcard.                                                                                                               | Auditoría anterior; Security Group descrito por el usuario, no auditado.                                                                          | Una escucha wildcard no demuestra exposición a Internet: deben distinguirse escucha local y controles de red.                                                |
| Dos dispositivos administrativos emparejados; cero nodos y cero canales.                                                               | Auditoría anterior; se omiten identificadores.                                                                                                    | Existía una superficie administrativa emparejada; no se verifican ahora vigencia ni permisos de esos dispositivos.                                           |
| Navegador habilitado; sesiones autenticadas no inspeccionadas.                                                                         | Auditoría anterior.                                                                                                                               | No se conoce el alcance de acceso a servicios o credenciales del navegador. No se afirma que existieran sesiones autenticadas concretas.                     |
| Visibilidad de sesiones sin configurar; documentación local consultada entonces describía default all; agentToAgent habilitado.        | Auditoría anterior y documentación local consultada previamente; no se probaron historiales ni se releyeron esas referencias para este documento. | Posible alcance amplio entre sesiones o agentes que requiere validación futura autorizada. No se acredita acceso efectivo a conversaciones privadas.         |

## Riesgos y límites de interpretación

- **Riesgo principal histórico:** ejecución amplia bajo una cuenta con sudo sin contraseña. La protección de red del Gateway y las aprobaciones de herramientas son controles distintos de los privilegios del proceso en Linux.
- Sin sandbox, las reglas escritas y el comportamiento esperado del asistente no impiden técnicamente todas las acciones que permite la identidad del sistema.
- Los permisos 0700/0600 limitan acceso de otras identidades ordinarias, pero no aíslan secretos de procesos ejecutados por su propietario.
- No se realizó una auditoría nueva de AWS, firewall, SSH, actualizaciones, dispositivos, navegador, historiales, secretos, sockets, permisos efectivos ni configuración completa.
- No se ejecutó root en la auditoría descrita. En esta tarea solo se documentan los datos suministrados: no se ejecutan sudo, pruebas de elevación ni pruebas de escritura fuera del documento.
- El alcance histórico y sus resultados no deben convertirse en afirmaciones permanentes sobre el sistema actual.

## Propuesta futura: identidad Linux dedicada

**Propuesta pendiente de autorización e implementación; no aplicada.**

1. Separar la identidad humana administradora de una cuenta Linux dedicada a OpenClaw. La cuenta dedicada no tendría sudo, pertenencia a sudo/docker/lxd ni acceso a sockets privilegiados. Revisar también mecanismos equivalentes de privilegio y accesos indirectos en una fase autorizada.
2. Asignar workspace, configuración y credenciales a la cuenta dedicada con propiedad y acceso mínimos necesarios. Mantener las credenciales fuera del workspace y evitar que respaldos o registros introduzcan copias allí. El acceso de mantenimiento humano debe ser separado y controlado.
3. Limitar el filesystem accesible y los comandos disponibles como defensa adicional, distinguiendo restricciones de herramientas de controles efectivos del sistema operativo. No presentar esos límites como sandbox si no proporcionan ese aislamiento.
4. Ejecutar el servicio con la cuenta dedicada y endurecimiento compatible, restringiendo escrituras y acceso a los directorios estrictamente necesarios. Evaluar compatibilidad antes de aplicar restricciones: no se propone una receta universal ni cambios al servicio actual.
5. Conservar Gateway en loopback y el acceso humano mediante túnel SSH; no introducir un puerto público ni un proxy como requisito de la migración.

**Quitar sudo no equivale a disponer de sandbox.** La cuenta dedicada seguirá pudiendo acceder a los secretos que le pertenezcan o que necesite usar, y podrá tener acceso a la red según los controles aplicables. Una sesión de navegador autenticada también puede conferir capacidades. El aislamiento futuro deberá definirse explícitamente si se desea una frontera más fuerte.

## Migración futura por etapas autorizadas

Cada etapa requiere autorización específica antes de sus acciones. Este documento no autoriza ni inicia ninguna de ellas.

1. **Plan y alcance:** acordar capacidades necesarias, límites, responsable humano y ventana de intervención; validar entonces el estado pertinente sin ampliar la auditoría por defecto. Identificar dependencias de identidad, memoria, delegación y herramientas.
2. **Respaldo y recuperación:** preparar un respaldo seguro de los datos necesarios, preservando propiedad y permisos pertinentes, acceso restringido y posibilidad de restauración. Definir rollback antes de alterar el servicio. Nunca copiar secretos al workspace ni incluirlos en documentación o salidas.
3. **Preparación de la cuenta dedicada:** crearla y asignar solo accesos necesarios, con administrador humano separado. Planificar propiedad de los archivos sin transferencias masivas indiscriminadas ni concesión de privilegios heredados.
4. **Transición controlada:** migrar el servicio y los datos necesarios con tratamiento seguro de credenciales, preservando loopback y túnel. Aplicar endurecimiento compatible gradualmente, con puntos de retorno claros.
5. **Validación de continuidad y límites:** ejecutar únicamente las pruebas previamente autorizadas que correspondan a los criterios siguientes; registrar evidencias sin secretos. Mantener el acceso humano de recuperación durante toda la transición.
6. **Cierre o rollback:** aceptar el cambio solo tras verificar continuidad y controles; si falla, restaurar el estado respaldado conforme al plan aprobado y documentar el riesgo residual. Retirar accesos o recursos antiguos solo con autorización y después de comprobar que ya no son necesarios.

No se incluyen comandos de cambio o de elevación ejecutables. No se ha aplicado ninguna etapa.

## Criterios de aceptación FUTUROS — no ejecutados

| Criterio                                         | Evidencia futura esperada, sujeta a autorización                                                                                                                                             | Estado                   |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| Cuenta dedicada sin privilegios administrativos. | Identidad efectiva del proceso y del servicio; ausencia de sudo, grupos sudo/docker/lxd, sockets privilegiados y vías equivalentes relevantes. No exige ensayar elevación sin autorización.  | Pendiente; no ejecutado. |
| Control de escrituras y acceso mínimo.           | Propiedad y permisos apropiados de workspace, configuración y credenciales; prueba acotada y acordada de escritura permitida y denegada en ubicaciones de prueba, sin tocar datos sensibles. | Pendiente; no ejecutado. |
| Identidad y memoria conservadas.                 | Comprobar continuidad de la identidad del asistente y acceso a los archivos de memoria autorizados; no ampliar visibilidad ni reproducir secretos en resultados.                             | Pendiente; no ejecutado. |
| Delegación funcional y limitada.                 | Verificar delegación con datos sintéticos, alcance de herramientas y visibilidad de sesiones conforme a la política acordada; no explorar historiales privados.                              | Pendiente; no ejecutado. |
| Acceso humano preservado.                        | Administrador separado, SSH y Dashboard mediante túnel operativos; Gateway sigue en loopback y no se publica el puerto. Comprobación autorizada de controles de red si entra en el alcance.  | Pendiente; no ejecutado. |
| Servicio y herramientas necesarios operativos.   | Confirmar ejecución bajo la cuenta dedicada y compatibilidad de restricciones con las capacidades autorizadas, incluidos persistencia y reinicio si se aprueba probarlos.                    | Pendiente; no ejecutado. |
| Respaldo y rollback utilizables.                 | Evidencia acordada de recuperación segura, con permisos preservados y sin introducir secretos en el workspace; registrar responsable y punto de retorno.                                     | Pendiente; no ejecutado. |
| Riesgo residual explícito.                       | Aceptación informada de que sin sudo sigue sin haber sandbox, y de los accesos necesarios a secretos propios, red y posibles sesiones autenticadas.                                          | Pendiente; no ejecutado. |

## Referencias y trazabilidad

Fuentes de este registro: confirmaciones del usuario y resumen proporcionado de una auditoría anterior. Las siguientes rutas fueron consultadas anteriormente y se conservan **solo como referencias locales; no se han releído para esta documentación**:

- /home/ubuntu/.npm-global/lib/node_modules/openclaw/docs/cli/approvals.md
- /home/ubuntu/.npm-global/lib/node_modules/openclaw/docs/cli/sandbox.md
- /home/ubuntu/.npm-global/lib/node_modules/openclaw/docs/gateway/security/tool-permissions.md

El documento omite secretos, IP públicas reales, valores de tokens e identificadores de dispositivos. Las direcciones loopback incluidas describen la arquitectura local. Ninguna propuesta ni criterio pendiente constituye una afirmación de endurecimiento aplicado.
