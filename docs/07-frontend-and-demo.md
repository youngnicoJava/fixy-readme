# 07 — Frontend Angular y demo interactiva

[Índice](README.md) · [Testing](08-testing-and-evidence.md) · [Galería](09-visual-tour.md)

## 1. Stack y runtime reales

El frontend conectado declara Angular 22 y TypeScript 6, RxJS, Angular Material/CDK, SCSS e iconos. Usa componentes standalone y rutas `loadComponent` para carga diferida. Signals representan estado reactivo; computed evita duplicar derivaciones donde se utiliza. Observables siguen siendo útiles para HTTP y coordinación.

La implementación actual conserva `provideZoneChangeDetection()` y `zone.js` como polyfill. OnPush se utiliza donde está declarado; no se documenta una garantía de que todos los componentes ya cumplan ese patrón.

Hay dependencias/build de servidor Angular y Express. Sin embargo, la ruta server comodín usa `RenderMode.Client`, y no se observa configuración global de client hydration en app.config. La presencia de `@angular/ssr` no demuestra renderizado SSR/hydration real de todas las pantallas.

La demo usa build estático. Su hosting GitHub Pages no ejecuta servidor Angular ni backend.

## 2. Organización funcional

| Área | Pantallas / propósito |
|---|---|
| auth | Login, registro y modal de acceso |
| dashboard | Vista cliente y vista profesional |
| requests | Publicar, listar oportunidades, propuestas y detalle |
| contracting / ContractPanel | Quotes, agreement, visita, pago y aftercare |
| conversations | Layout, lista, conversación y widget |
| payments | Contrataciones independientes y acciones financieras |
| worker / service-offerings | Publicaciones, detalle y perfil profesional |
| profile | Perfil, direcciones y onboarding |
| admin | Revisión de profesionales pendientes |
| shared | Formularios, hints, cards y componentes reutilizados |
| core | Servicios, guards, interceptor, constantes y configuración |

Los modelos frontend no son entidades JPA. Deben respetar DTOs, enums y nullability sin asumir que worker profile ID es User ID o que agreement existe desde que hay propuesta.

## 3. RequestDetail como centro

El detalle muestra necesidad, estado, profesional seleccionado, contratación, conversaciones/comentarios y timeline. ContractPanel reutiliza presupuesto/acuerdo/pago, en lugar de construir otra lógica financiera cada vez que aparece un CTA.

La jerarquía busca que el usuario entienda trabajo, profesional, importe y próximo paso. Los detalles secundarios se agrupan en secciones/disclosures; revisar presupuesto no requiere interpretar mensajes de chat.

Los controles dependen de rol y estado. La validación real del backend sigue siendo obligatoria: ocultar “Iniciar” no impide un PATCH manual.

Aftercare usa consultas opcionales para review/dispute. `204` representa ausencia válida; errores de autorización/request/backend conservan su semántica. No se utiliza catch-all para convertir cualquier fallo en “todavía no hay review”.

## 4. Pagos sin carrito

`/payments` muestra cada contratación por separado. Un agreement aceptado puede aparecer como pendiente sin Payment creado. El backend entrega `ContractPaymentItem` con title, workerName, total y Payment opcional.

Secciones de UI: pendientes, protegidos, finalizados y reembolsados. Los estados subyacentes deben mapearse a PENDING/HELD/RELEASED/REFUNDED y al handling de FAILED; “finalizado” financiero no es sinónimo del estado COMPLETED del trabajo.

RequestDetail, chat y listado de Pagos utilizan el mismo servicio/API de Payment. No hay checkout multicontratación ni total agregado para cobrar varios requests.

El retorno de Mercado Pago vuelve a consultar el backend. El query de retorno sirve para contexto; no confirma HELD por sí mismo.

## 5. Identidad y lifecycle del chat

La lista, widget y header usan `counterpart` del DTO. Su cálculo ocurre desde participantes auténticos de la conversación, nunca desde el último sender. Avatar y nombre deben corresponder a esa misma identidad; foto ausente tiene fallback visual.

El chat conectado carga/envía mensajes por HTTP. No se declara WebSocket, typing indicators, presencia ni sincronización automática entre dispositivos que el código no implementa.

Al cambiar de conversación, el componente invalida trabajo pendiente del contexto anterior. Usa `takeUntilDestroyed` para lifecycle y un notifier de cambio de contexto en operaciones de carga/envío/borrado. Esto evita que una respuesta de un chat anterior pise el actual.

Una tarjeta comercial del chat obtiene presupuesto/acuerdo de contracting. No almacena la aceptación como un Message TEXT ni extrae precios de texto libre.

## 6. Onboarding contextual

Hints/versiones locales y guías integradas explican acciones relevantes sin tours bloqueantes. Ejemplos: primer presupuesto, negociación, pago protegido y dashboard de profesional habilitado.

En la demo, la guía depende de pantalla, rol y lifecycle; se puede cerrar, reabrir y conservar dismissal. Explica qué acción realizar, pero no ejecuta el dominio por el usuario ni elige presupuesto/pago automáticamente.

La pantalla Info explica la separación entre simulación y plataforma conectada y enlaza este repositorio técnico. Navbar y barra de demo mantienen visible la identidad de demostración.

## 7. Motor local de la demo

La copia pública conserva componentes y estilo del producto, con un backend simulado dentro del navegador:

- `DemoEngine`: operaciones y reglas locales.
- Estado/fixtures: relaciones ficticias coherentes.
- Store Angular: integración reactiva y persistencia.
- Interceptor: responde localmente; nunca hace fallback HTTP al backend real.

El motor serializa escrituras y hace rollback del estado si falla validación/persistencia. Esto permite demostrar concurrencia/idempotencia local con tests, pero no reemplaza locks PostgreSQL ni consistencia distribuida.

Storage versionado mantiene sesión, solicitudes, presupuestos, acuerdos, mensajes y reseñas. Cada origen/navegador tiene su estado. No hay sincronización entre dispositivos/pestañas; cambiar rol recarga para evitar componentes con contexto anterior.

### Fixtures iniciales

| Request de demo | Propósito |
|---|---|
| 101 / 102 | Oportunidades abiertas para iniciar recorrido |
| 103 | Profesional seleccionado, agreement y Payment pendiente |
| 104 | Trabajo en progreso, pago protegido |
| 105 | Contratación completada, fixture financiero finalizado y review |

Son datos ficticios, no copias de los pagos privados. El fixture finalizado no prueba que la demo haya realizado settlement.

## 8. Recorrido reproducible sin backend

1. Abrir la demo y entrar como cliente.
2. Publicar una necesidad o usar una solicitud abierta.
3. Cambiar a trabajador y enviar propuesta para categoría habilitada.
4. Cambiar a cliente y seleccionar profesional.
5. Para precio a evaluar, worker envía quote con mano de obra/materiales; cliente acepta.
6. Cliente elige Pagar y confirma **simulación local**.
7. Worker inicia; cliente completa; Payment conserva HELD/protección.
8. Cliente crea review; perfil del worker muestra el nuevo rating.
9. Recargar para comprobar persistencia; restablecer para repetir.

Las guías e Info explican los límites durante el recorrido. No se pide documento, tarjeta ni datos de una persona real.

## 9. Límites deliberados de demo

No existen cobro real, provider lookup, webhook, Kafka, PostgreSQL, OAuth, correo, payout, refund ni release automático. Las operaciones financieras son ficticias.

Mapas, geocodificación y captcha se reemplazan por controles locales; no se publican claves HERE/Turnstile ni se solicita geolocalización real. Tener Leaflet en las dependencias heredadas no obliga a mostrar un mapa que aparenta datos reales.

La visita paga continúa bloqueada. El onboarding puede crear PENDING, pero no inventa aprobación de admin; hay workers ficticios ya verificados para explorar.

Algunas pantallas informativas/edición del producto todavía no están implementadas. La demo explica ese límite, no crea funcionalidades ajenas al código fuente. Imágenes ingresadas localmente están sujetas a cuota de storage y límite por archivo.

## 10. Deployment real de la demo

**GitHub Pages, mediante GitHub Actions.** URL: [youngnicojava.github.io/fixy-demo-frontend](https://youngnicojava.github.io/fixy-demo-frontend/).

El workflow instala dependencias con Bun, usa Node compatible, ejecuta typecheck, lint acotado y tests, compila la configuración Pages y publica el artefacto estático. El build usa base del repositorio y rutas hash, como `/#/requests/103`, para soportar navegación sin rewrites de servidor.

La copia también tiene configuración Docker/Nginx y Vercel. Eso no significa que el deployment público observado se haya hecho con Vercel.

### Ejecutar el repositorio público

```sh
git clone https://github.com/youngnicoJava/fixy-demo-frontend.git
cd fixy-demo-frontend
bun install --frozen-lockfile
bun run start -- --port 4300
```

Requisitos declarados: Bun 1.3.14 y Node compatible con Angular 22 según engines del repo. Alternativa Docker Compose disponible en ese repositorio. No requiere `.env` con credenciales de Fixy.

## 11. Validación visual de esta publicación

Se abrieron home, detalle con agreement, Pagos, conversación e interfaz profesional de la demo publicada. Se comprobaron lista/header con interlocutor, separación financiera y avisos demo. Las lecturas de consola de chat, Pagos y dashboard capturados no mostraron errores.

No se ejecutó checkout, no se alteró Payment privado y no se declara un nuevo E2E completo por haber obtenido imágenes. No se hizo una auditoría de responsive exhaustiva en esta publicación; la revisión móvil previa se identifica en la evidencia histórica.

## Evidencia visual del producto conectado

La [galería principal](09-visual-tour.md) usa el frontend privado conectado a Spring/PostgreSQL, con Payment Service activo. Incluye dashboards, mapa Leaflet/OpenStreetMap, formularios, publicación, lifecycle histórico, pagos, chat y Swagger. Se obtuvieron imágenes nuevas sin ejecutar operaciones financieras ni completar otra contratación.

La galería de la demo descrita arriba permanece en el [archivo separado](12-demo-visual-archive.md). Los avisos de consola limpia de aquella observación no se extrapolan al producto conectado: en la captura local se observaron dos notificaciones ResizeObserver.
