# 09 — Fixy conectado: frontend real y backend

[Índice](README.md) · [Demo pública independiente](https://youngnicojava.github.io/fixy-demo-frontend/) · [Archivo visual de la demo](12-demo-visual-archive.md)

Estas **19 capturas auténticas** corresponden al frontend privado Angular en `localhost:4200`, conectado a Spring en `:8080` y PostgreSQL histórico local. Payment Service se inició en `:9090`. Se obtuvieron el **10/10/2026**, con las cuentas de desarrollo autorizadas.

Los nombres, ubicaciones y solicitudes visibles son datos de prueba cuya publicación fue autorizada por el propietario. No se incluyen contraseñas, emails de acceso, tokens, firmas, headers privados ni IDs de Mercado Pago. Los formularios de autenticación se capturaron vacíos.

## Alcance de la evidencia

- Es el producto conectado, no la demo que sustituye HTTP por almacenamiento local.
- El ciclo se ilustra con solicitudes históricas en distintos estados. No es una nueva ejecución completa de una misma contratación.
- No se creó Payment/preference, no se realizó checkout, no se enviaron webhooks ni se ejecutaron reconciliaciones/payouts.
- El formulario de nueva solicitud se completó hasta revisión previa; **no se publicó**.
- No se alteró código ni diseño para obtener las imágenes. Se acotaron vistas al contenido útil, evitando divulgar los datos de contacto del footer o una dirección de domicilio precisa del menú.
- Las fotos de algunas publicaciones ya eran placeholders declarados por su autor. La pantalla es real; las fotos no demuestran trabajos ejecutados.

## Índice visual

1. [Autenticación](#autenticación)
2. [Navbar y direcciones](#navbar-y-direcciones)
3. [Dashboards](#dashboards)
4. [Publicación y solicitud](#publicación-y-solicitud)
5. [ServiceRequest: lifecycle persistido](#servicerequest-lifecycle-persistido)
6. [Pagos y chat](#pagos-y-chat)
7. [Swagger real](#swagger-real)

## Autenticación


### Login del frontend conectado

![Login del frontend conectado](../assets/screenshots-real/auth-login.jpg)

Formulario de ingreso real. Se iniciaron sesiones de cliente y worker para las vistas siguientes; las credenciales no aparecen en la imagen. La captura del formulario no demuestra por sí sola rotación de refresh.


### Registro

![Registro](../assets/screenshots-real/auth-register.jpg)

Primer paso con nombre, apellido, email y contraseña. No se creó una cuenta nueva ni se enviaron documentos.


## Navbar y direcciones


### Portada real

![Portada real](../assets/screenshots-real/home.jpg)

Navbar y portada del frontend privado. Sin barra de demo ni sustitución local de API. Se conserva el producto tal como está renderizado.


### Menú del navbar

![Menú del navbar](../assets/screenshots-real/navbar-worker.jpg)

Menú abierto bajo la identidad del worker. Panel profesional se utilizó para acceder al dashboard. El buscador del navbar aparece deshabilitado: no se declara validada esa búsqueda.


### Address modal con Leaflet

![Address modal con Leaflet](../assets/screenshots-real/address-modal.jpg)

Tipo de dirección, buscador, piso/departamento y mapa Leaflet con tiles OpenStreetMap. Formulario vacío y ubicación inicial del mapa. No se solicitó geolocalización ni se guardó una dirección.


## Dashboards


### Dashboard cliente

![Dashboard cliente](../assets/screenshots-real/client-dashboard.jpg)

Actividad y solicitudes obtenidas del backend. Las cards muestran estado y profesional seleccionado; Nueva solicitud conduce al formulario real y Pagos mantiene el contexto de contratación.


### Dashboard profesional

![Dashboard profesional](../assets/screenshots-real/worker-dashboard.jpg)

Verificación, reputación, servicio en curso, publicaciones y propuestas existentes. El aviso contextual describe presupuesto y pago protegido como pasos previos al inicio en el flujo actual.


## Publicación y solicitud


### Detalle de publicación profesional

![Detalle de publicación profesional](../assets/screenshots-real/service-offering-detail.jpg)

Se abrió desde la imagen de un servicio visible en home. Incluye galería, precio orientativo, profesional, descripción y opiniones. La offering no equivale a un Payment; las fotos del anuncio son placeholders declarados en sus datos.


### Crear solicitud

![Crear solicitud](../assets/screenshots-real/request-create.jpg)

Título, categoría y descripción introducidos como borrador para mostrar la interacción. No se persistió una nueva solicitud.


### Publish: revisión previa

![Publish: revisión previa](../assets/screenshots-real/request-publish-review.jpg)

Resumen real y CTA Publicar solicitud. Se detuvo aquí para conservar los datos históricos: no se pulsó publicar.


## ServiceRequest: lifecycle persistido


### OPEN: necesidad publicada

![OPEN: necesidad publicada](../assets/screenshots-real/request-open.jpg)

Solicitud existente, comentarios, propuestas e historial. Es un registro histórico abierto; la captura no certifica su scheduler de expiración ni renueva su vigencia.


### MATCHED: presupuesto y acuerdo

![MATCHED: presupuesto y acuerdo](../assets/screenshots-real/request-matched-agreement.jpg)

Desplegables abiertos con presupuesto desglosado, snapshot del acuerdo y protección. El pago del fixture sigue pendiente y no se ejecutó el CTA. El fixture financiero de anomalía permanece intacto; esta imagen no autoriza repetir su checkout.


### IN_PROGRESS histórico

![IN_PROGRESS histórico](../assets/screenshots-real/request-in-progress.jpg)

Solicitud previa a la evolución actual: trabajo en progreso mientras el pago sigue pendiente. Se conserva esa inconsistencia histórica, sin corregirla para obtener una captura. No representa el happy path vigente, que exige confirmación financiera antes de iniciar.


### COMPLETED + HELD + review

![COMPLETED + HELD + review](../assets/screenshots-real/request-completed-review.jpg)

Golden Path histórico con presupuesto aceptado, acuerdo, pago protegido, review 5/5 e historial persistido. El historial registra confirmación por reconciliación, inicio, finalización y valoración. COMPLETED y SETTLED son decisiones distintas. No se ejecutaron esas transiciones de nuevo.


## Pagos y chat


### Pagos pendientes

![Pagos pendientes](../assets/screenshots-real/payments-pending.jpg)

Listado real de operaciones por contratación, sin checkout combinado. No se pulsó Pagar en ninguna card.


### Pagos protegidos

![Pagos protegidos](../assets/screenshots-real/payments-protected.jpg)

La pestaña Protegidos carga el Golden Path histórico desde Spring. No es simulación local de dinero. La fecha de protección ya pasó pero el estado observado sigue HELD; no se declara settlement automático PASS.


### Chat con contexto comercial

![Chat con contexto comercial](../assets/screenshots-real/conversation-contract.jpg)

Como worker, header y listado identifican al cliente. El acuerdo y el pago protegido aparecen en el chat. La conversación seleccionada está vacía: no se inventaron mensajes ni se envió contenido para llenar la pantalla.


## Swagger real


### OpenAPI servido por Spring

![OpenAPI servido por Spring](../assets/screenshots-real/backend-swagger.jpg)

Documento cargado desde /v3/api-docs en la aplicación activa. No se autorizó Swagger con tokens ni se ejecutó Try it out.


### API de contratación y lifecycle

![API de contratación y lifecycle](../assets/screenshots-real/backend-contract-api.jpg)

Endpoints reales de inicio/finalización y controller de visitas, presupuestos y acuerdo. La imagen proviene de Swagger activo, no de una interfaz reconstruida.


## Entorno y validación

| Comprobación | Resultado |
|---|---|
| Angular /home, puerto 4200 | HTTP 200 |
| Spring /v3/api-docs, puerto 8080 | HTTP 200 |
| Payment /health, puerto 9090 | HTTP 200 |
| PostgreSQL local de Fixy, puerto 5433 | 28 tablas; conexión de Spring correcta |
| Sesiones cliente y worker | Vistas por rol cargadas |
| Leaflet/OpenStreetMap | Mapa visible en address modal |
| Contrataciones | Estados leídos de registros existentes |
| DB schema | Arranque con ddl-auto=validate, sin actualización DDL |
| Liquidación automática | Desactivada sólo en el proceso de captura |
| Consola | Dos avisos ResizeObserver loop completed with undelivered notifications observados; no se declara consola limpia |

No se modificaron estados de dominio ni se ejecutaron suites privadas nuevas para este cambio documental. Iniciar sesión utiliza el mecanismo normal de autenticación y puede registrar tokens de sesión; no se afirma que toda la DB haya sido inmutable.

El proceso de captura conservó HELD sin ejecutar release. La revisión visual no acredita settlement productivo ni webhook real firmado PASS. [Testing y evidencia](08-testing-and-evidence.md).

El [manifest](evidence-manifest.json) registra hashes SHA-256, rutas y origen de las imágenes, separando frontend conectado, Swagger y la demo archivada.
