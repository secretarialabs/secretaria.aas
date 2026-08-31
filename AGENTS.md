# Secretaria

Secretaria es una plataforma SaaS para crear asistentes inteligentes
(orquestador de agentes).

## Arquitectura

Monorepo con submódulos Git.

- `web/` → Next.js 16 + React 19 + TypeScript
- `bff/` → NestJS 10 + Express + TypeScript + Supabase

## Principios

- TypeScript estricto.
- pnpm obligatorio.
- Node 23 obligatorio.
- Código limpio antes que optimización.
- Preferir composición sobre herencia.
- No duplicar lógica.
- Mantener arquitectura simple para el MVP.
- Todo texto que se visualice en el frontend debe estar en español. Los valores técnicos internos
  pueden permanecer en inglés, pero deben traducirse antes de mostrarse al usuario.

## Convenciones

- camelCase para variables.
- PascalCase para componentes y clases.
- ESLint obligatorio.
- Prettier obligatorio.
- Commits convencionales.
- Todo botón interactivo debe mostrar `cursor: pointer`. Los botones deshabilitados deben comunicar
  ese estado con `cursor: not-allowed`.

## Ciclo de vida de datos

Al crear una tabla o una nueva entidad persistente asociada a un usuario o espacio, la misma tarea
debe revisar el flujo de eliminación y reinicio de cuenta:

- definir si se elimina mediante `ON DELETE CASCADE` o de forma explícita en el backend;
- actualizar la función transaccional de limpieza mediante una migración nueva cuando corresponda;
- agregar sus archivos o recursos externos al servicio de limpieza si aplica;
- actualizar los textos de la Zona de peligro para informar al usuario qué datos se eliminan;
- agregar o actualizar pruebas que verifiquen que el reset no deja datos huérfanos.

No editar migraciones ya aplicadas para incorporar esta limpieza. Crear una migración nueva.

## Estructura

```
web/   → secretaria.web (Next.js)
bff/   → secretaria.bff (NestJS)
```

Cada subproyecto posee su propio `AGENTS.md` con reglas específicas.

## Cómo trabajar

Cuando una tarea involucre más de un proyecto:

1. leer primero este archivo
2. luego leer el `AGENTS.md` correspondiente
3. respetar las reglas de ambos

Si existe conflicto:

- prevalece el `AGENTS.md` del subproyecto.

## Canales de comunicación

SecretarIA tiene tres canales principales:

- Chat web
- Telegram
- WhatsApp

Toda funcionalidad relacionada con conversaciones, mensajes, comandos, archivos,
notificaciones, confirmaciones o respuestas debe diseñarse considerando los tres
canales, aunque alguno todavía no esté habilitado completamente.

Antes de implementar o modificar una funcionalidad:

1. Identificar si afecta a uno o más canales.
2. Evitar acoplar la lógica de negocio directamente a Chat, Telegram o WhatsApp.
3. Mantener la lógica compartida independiente del canal.
4. Implementar las diferencias mediante adapters, strategies o channel handlers.
5. Evaluar explícitamente el comportamiento esperado en:
   - Chat web
   - Telegram
   - WhatsApp
6. Incluir el canal de origen en los contratos y entidades cuando corresponda.
7. No asumir que una funcionalidad es exclusiva del Chat web salvo que el
   requerimiento lo indique explícitamente.

### Confirmaciones y aprobaciones conversacionales

Toda funcionalidad que requiera confirmación, aceptación, aprobación, rechazo,
cancelación o autorización para ejecutar una acción debe contemplar que:

- En Telegram y WhatsApp estas decisiones se realizan mediante la conversación.
- El canal debe explicar con claridad qué acción está pendiente y cuáles serán
  sus efectos antes de solicitar la confirmación.
- El usuario debe poder responder con lenguaje natural, por ejemplo: `Sí`,
  `Confirmo`, `Acepto`, `Enviar`, `Cancelar` o expresiones equivalentes.
- Las negaciones y cancelaciones deben evaluarse antes que las confirmaciones
  para evitar ejecuciones accidentales.
- Una respuesta de confirmación debe recuperar y ejecutar la acción pendiente;
  no debe interpretarse como una instrucción nueva ni perder su contexto.
- Si una operación contiene varias acciones con efectos diferentes, se debe
  consultar o verificar cada aprobación necesaria, salvo que el usuario las
  autorice explícitamente en el mismo mensaje.
- Los botones del Chat web pueden complementar el flujo, pero nunca deben ser el
  único mecanismo disponible para aprobar una acción en Telegram o WhatsApp.
- Antes de finalizar la implementación, verificar con tests el flujo completo:
  propuesta, preview, confirmación o rechazo por conversación y resultado final.

WhatsApp todavía no está habilitado completamente, pero sus necesidades deben
considerarse en contratos, arquitectura y decisiones que serían costosas de
modificar posteriormente.

Al finalizar una funcionalidad relacionada con comunicación, informar:

- Qué comportamiento es compartido.
- Qué diferencias existen por canal.
- Qué queda pendiente para los canales todavía no habilitados completamente.

### Módulos multicanal

Todo módulo de negocio nuevo debe diseñarse para operar desde los tres canales: Chat web, Telegram
y WhatsApp. Esto incluye crear, consultar, actualizar y eliminar entidades cuando la naturaleza del
módulo y los permisos del actor lo permitan.

Antes de considerar completo un módulo:

1. Definir casos de uso compartidos para CRUD, búsquedas y consultas en lenguaje natural.
2. Exponer esos casos de uso al agente sin duplicar reglas dentro de cada canal.
3. Resolver y autorizar al actor como propietario, integrante cuando el modelo lo soporte, cliente
   o desconocido antes de ejecutar acciones. Mientras el producto mantenga un único dueño por
   espacio, no agregar membresías solo para cumplir esta regla.
4. Aplicar confirmaciones conversacionales a operaciones sensibles, especialmente actualizaciones,
   eliminaciones, envíos y acciones con efectos externos.
5. Adaptar vocabulario, identificadores, campos y respuestas a la plantilla activa del espacio. Un
   rubro no debe heredar términos de otro, como patente o vehículo fuera de un taller.
6. Permitir que clientes externos realicen únicamente las consultas y acciones autorizadas para su
   identidad, por ejemplo consultar el estado de un vehículo, reparación, trámite u orden de
   servicio.
7. Implementar diferencias de presentación y transporte mediante adapters; los handlers de Chat
   web, Telegram y WhatsApp no deben contener lógica de negocio.
8. Agregar pruebas de paridad que ejecuten los mismos casos de uso desde los tres canales y pruebas
   negativas de permisos e aislamiento entre espacios.

Si un módulo no admite razonablemente alguna operación CRUD, documentar la excepción y el motivo en
su diseño y en la entrega final, en lugar de simular una operación que no corresponde.

## Menú Tareas

Antes de implementar o modificar una funcionalidad accionable para el usuario,
preguntar explícitamente si debe mostrarse en el menú `Tareas`.

Se considera accionable cuando genera alguno de estos elementos:

- pendientes o recordatorios;
- vencimientos o seguimientos;
- aprobaciones o confirmaciones;
- errores recuperables;
- acciones programadas o recurrentes;
- resultados que requieren una decisión posterior del usuario.

No es necesario consultarlo para cambios exclusivamente visuales, refactors
internos o navegación que no produzca trabajo pendiente.

Al finalizar una funcionalidad accionable, informar qué aparece en `Tareas`,
cómo se resuelve desde esa página y qué quedó deliberadamente fuera.
