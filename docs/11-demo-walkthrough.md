# 11 — Recorrido reproducible de la demo

[Índice](README.md) · [Archivo de las 27 capturas de demo](12-demo-visual-archive.md) · [Abrir demo](https://youngnicojava.github.io/fixy-demo-frontend/)

Este recorrido explica cómo explorar Fixy sin el backend privado. Todos los usuarios, direcciones, contratos y pagos son ficticios y se almacenan localmente en el navegador. No uses datos personales ni contraseñas reales. Los IDs creados durante una sesión pueden cambiar: no son enlaces públicos compartidos ni registros del backend.

## Preparar la sesión

Usá la barra superior para **Entrar como cliente** o **Entrar como trabajador**. El cliente de ejemplo es José María Paz y el profesional es Luca Prodan. Cambiar de rol permite recorrer ambas perspectivas; no representa una elevación de permisos del sistema privado.

[Login](https://youngnicojava.github.io/fixy-demo-frontend/#/login) y [registro](https://youngnicojava.github.io/fixy-demo-frontend/#/register) muestran los formularios reales del frontend adaptados al entorno local. Las credenciales públicas de ejemplo aparecen en el aviso de login. La autenticación de la demo no demuestra JWT, refresh ni seguridad del backend; esos mecanismos se explican en [seguridad](04-security-and-auth.md).

## Navegación y dirección

1. Abrí el icono de cuenta del navbar: **Panel**, **Mis solicitudes** y **Pagos** son accesos al contexto del cliente.
2. Abrí **Mis direcciones**: se muestra la dirección ficticia activa y las acciones disponibles.
3. **Agregar dirección** abre el modal con tipo Casa/Trabajo/Otro, buscador, piso y departamento. El área de ubicación indica explícitamente que no usa mapas ni geolocalización externa.
4. Podés cerrar el modal sin guardar. No hace falta proporcionar una ubicación real para explorar la demo.

En esta revisión se abrió el menú de cuenta y se utilizó **Panel** para llegar al dashboard del cliente. No se declara probado el buscador del navbar: aparece deshabilitado en las vistas capturadas.

## Publicar una necesidad

Desde el dashboard del cliente, **Nueva solicitud** conduce al formulario:

1. Título del problema.
2. Categoría del oficio.
3. Descripción del alcance inicial.
4. **Continuar** abre la revisión previa a publicación.
5. **Publicar solicitud** crea una solicitud local en estado `OPEN`.

La publicación del profesional es otro concepto: [detalle de un servicio](https://youngnicojava.github.io/fixy-demo-frontend/#/services/11) muestra referencia comercial, precio orientativo, descripción y profesional. **Crear solicitud para este servicio** expresa el paso hacia una necesidad/contratación; la offering no es un Payment.

## De la propuesta al acuerdo

1. Cambiá a trabajador y abrí la nueva solicitud.
2. En **Tu participación**, escribí cómo ayudarías. Elegí **Evaluar y enviar presupuesto** para demostrar la negociación posterior.
3. Enviá la propuesta.
4. Volvé a cliente, abrí esa misma solicitud y **Seleccionar** al profesional.
5. La solicitud pasa a `MATCHED`; el chat queda como herramienta de coordinación.
6. Como trabajador, **Enviar presupuesto** abre mano de obra, materiales y alcance. El total se calcula a partir de los importes.
7. Como cliente, la tarjeta permite **Aceptar presupuesto** o **Rechazar**.
8. Aceptarlo crea un acuerdo y habilita el CTA de pago.

El chat no determina el importe. La tarjeta de presupuesto y el acuerdo permiten ver qué se contrató y cuánto cuesta.

## Pago simulado e inicio

**Pagar contratación** abre un diálogo que informa: no se cobrará dinero ni se abrirá Mercado Pago. **Confirmar simulación** representa `HELD` exclusivamente en el estado local de la demo.

Después, el profesional puede **Iniciar servicio** y la solicitud pasa a `IN_PROGRESS`. El historial muestra propuesta, selección, presupuesto, aceptación, confirmación simulada e inicio. El dinero ficticio permanece protegido.

## Completion, protección y review

En el sistema, el cliente utiliza **Confirmar finalización** y una contratación completada puede seguir con Payment `HELD` durante la protección. Completion y liquidación son decisiones distintas; ver [lifecycle](02-domain-and-lifecycle.md) y [Payment](05-payment-and-events.md).

**Límite de la captura del 10/10/2026:** la solicitud nueva llegó visualmente hasta `IN_PROGRESS`. Al intentar confirmar la finalización, el diálogo nativo `window.confirm` bloqueó el navegador automatizado. No se certificó esa transición ni se creó una review nueva. No se modificó el frontend para obtener una captura.

Para mostrar el resultado final se abrió el [fixture completado](https://youngnicojava.github.io/fixy-demo-frontend/#/requests/105). Su pantalla tiene una review ficticia de 5/5 y un estado financiero liquidado precargado. **No es evidencia de settlement ejecutado ni continuación de la nueva solicitud.**

## Evidencia de esta sesión

| Etapa | Resultado observado | Alcance |
|---|---|---|
| Navbar → Panel | Dashboard cliente visible | Navegación real de la demo |
| Dirección | Menú y modal abiertos | No se guardó una dirección nueva |
| Login / registro | Formularios capturados | Sin cuenta nueva; no valida auth del backend |
| Solicitud publicada | `OPEN`, necesidad y ubicación visibles | Nueva entidad local |
| Propuesta | Pendiente y visible para el cliente | Envío local |
| Selección | `MATCHED`, Luca Prodan seleccionado | Transición local |
| Presupuesto | Mano de obra 45.000 + materiales 10.000 | ARS 55.000 ficticios |
| Aceptación | Acuerdo registrado | Snapshot local |
| Pago | Aviso explícito y estado protegido | Simulado, sin Checkout Pro |
| Inicio | `IN_PROGRESS` y evento de inicio | Transición local |
| Completion nueva | No verificada | Diálogo nativo bloqueó automatización |
| Completion + review | Fixture 105 renderizado | Datos precargados, no transición nueva |

La sesión creó una solicitud local titulada **Demo portfolio · Reparar pérdida de agua** (ID local observado 200) y no alteró bases privadas ni llamó a Mercado Pago. Ese ID existe sólo en el almacenamiento de esta sesión: un visitante debe crear su propia solicitud o utilizar los fixtures.

Las capturas conservan los avisos de demo, los estados vacíos y el mensaje de ausencia de imágenes de la offering. No se rellenaron esos espacios con contenido inventado.
