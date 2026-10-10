# 08 — Testing y evidencia

[Índice](README.md) · [Galería](09-visual-tour.md)

## 1. Qué se demuestra y con qué

Un test con mock de Mercado Pago puede demostrar una regla local, pero no que un webhook atravesó un túnel real. Un screenshot de demo puede demostrar composición visual, pero no persistencia PostgreSQL. Un checkout approved no demuestra que el callback llegó a Fixy.

Esta documentación separa cuatro niveles:

1. Código actual inspeccionado.
2. Tests de regresión existentes.
3. Evidencia histórica guardada de una campaña E2E controlada.
4. Capturas/observación de navegador realizadas para esta publicación.

No se mezclan en un porcentaje de coverage ni se etiqueta como PASS un checkpoint que no se ejecutó.

## 2. Backend: suites existentes

| Suite | Comportamiento protegido |
|---|---|
| AuthControllerTest | Contrato de autenticación/refresh y errores |
| RefreshTokenConcurrencyTest | Rotación concurrente del token opaco |
| BusinessRulesRegressionTest | Worker verificado/categoría, duplicados, ownership, selección y completion con pago |
| ContractLifecycleTest | Presupuesto, revisión, aceptación, inmutabilidad y visita |
| AgreementPaymentTest | Agreement/payment, locks, checkout, protección, disputes, review, reconciliación y duplicados |
| OptionalAftercareHttpTest | 204 de ausencia y conservación de errores reales |
| ConversationCounterpartTest | Cliente/worker reciben al otro participante |
| CheckoutGatewayTest | Contrato de la frontera Spring → Go |
| CorsConfigTest | Orígenes configurables y formato permitido |
| PaymentKafkaListenerTest | Envelope/key, delegación transaccional y errores |

AgreementPaymentTest verifica, entre otras condiciones: no pagar sin quote aceptado; usar importe de agreement; actor autorizado; mismatch de importe; completion conservando HELD; early release bloqueado; dispute activa; callback idempotente; earning sólo al release confirmado; timeout sin perder identidad; leases concurrentes; reconciliación con ownership y anomaly sin efectos; mismo provider payment no financia dos contrataciones.

Las pruebas con H2 y gateways simulados no demuestran todos los detalles de locking/esquema de PostgreSQL productivo. Esa diferencia forma parte del plan de validación, no se oculta bajo “todos los tests pasan”.

## 3. Servicio Go: suites existentes

Hay tests para preferencias, recuperación, return URLs, inspección de provider payments, reconciliación, boundaries del proveedor, handler/diagnóstico de webhook y publisher Kafka.

Protegen mapeo de ARS/reference, rechazo de retorno local/host de webhook, recuperación de una preference coherente, errores/ambigüedad externa, HMAC, no producir efectos en modo diagnóstico y confirmación de publicación.

Los transports fakes y writers de test mantienen esas pruebas aisladas. No se presentan como cobros reales ni como evidencia de una configuración específica de cuenta Mercado Pago.

## 4. Frontend y demo

El frontend conectado contiene tests de interceptor auth, PaymentService, Pagos, ContractPanel, hints y counterpart, además de componentes auxiliares. Vitest es la infraestructura actual; no se describe Jest como runner activo porque el contexto antiguo lo mencionara.

La última validación guardada de la demo registra **12 archivos / 56 tests PASS**, typecheck y build PASS, y lint acotado PASS. El lint global conserva incidencias heredadas; no se lo etiqueta PASS. También hay advertencias de SCSS/templates registradas.

Los tests de demo cubren reglas locales y bloqueo del fallback al backend, persistencia, roles, contratación, quotes, pago simulado, completion/review/rating, visitas gratuitas, disputes, participantes y guías. No certifican seguridad bancaria ni backend productivo.

## 5. Golden Path histórico, evidencia sanitizada

Campaña guardada del 4–5 de octubre de 2026:

| Checkpoint | Resultado histórico |
|---|---|
| Contratación nueva mediante API normal | PASS |
| Propuesta y presupuesto aceptado / agreement | PASS |
| Checkout Pro TEST manual | PASS |
| Provider approved / accredited | PASS |
| Reconciliación autenticada con provider lookup real | PASS |
| Payment PENDING → HELD | PASS |
| Prevención de duplicación financiera | PASS |
| Inicio por worker seleccionado | PASS |
| Completion por cliente dueño | PASS |
| Payment después de completion | HELD, protección activa, sin settlement |
| Review única y rating actualizado | PASS |
| Persistencia/reload/UI del fixture | PASS en evidencia guardada |
| Webhook REAL firmado procesado end-to-end | **PENDIENTE — NO PASS** |
| Release / payout real | **No ejecutado / no validado** |

Las conclusiones provienen de reportes internos de API, persistencia y UI. Sus IDs reales, datos de cuenta y dumps no se publican. El fixture no se reejecutó ni se cambió para esta documentación.

Una simulación oficial posterior respondió 200 en diagnóstico separado. No reemplaza el checkpoint REAL ni transforma en PASS las entregas fallidas anteriores.

## 6. Caso negativo de anomalía

Se conservó aparte una contratación con varios provider payments TEST aprobados para la misma correlación. El estado local quedó PENDING y la recuperación se bloqueó con `409 MULTIPLE_PROVIDER_PAYMENTS`, sin earning/payout.

Esto demuestra la necesidad de separar “el proveedor aprobó algo” de “hay una operación inequívoca que puede aplicarse a esta contratación”. No se resolvió el caso escogiendo un ID, editando DB ni creando otro checkout.

El fixture sirve para regresión negativa. No se considera parte del Golden Path válido ni se publica como datos para reutilizar sus cobros.

## 7. Comprobaciones visuales: demo y sistema conectado

Fecha: **10/10/2026**.

**Primera captura: demo pública, antes de iniciar los servicios privados.**

- Demo pública abierta en navegador real: portada, RequestDetail con agreement, Pagos, chat y dashboard worker.
- Capturas guardadas con avisos de datos ficticios/sin backend.
- Lista y header del chat muestran al mismo interlocutor.
- Consola de vistas chat, Pagos y dashboard capturadas: sin errores observados.
- Spring `/v3/api-docs` y Go `/health` locales: no disponibles durante esta revisión; no se levantaron servicios ni se hicieron pagos para obtener material.
- No se generó captura ficticia de Swagger.
- Se revisaron enlaces locales, imágenes, fences y contenido público antes de publicar.

**Segunda captura: frontend privado conectado, con publicación autorizada de datos de prueba.**

- Se reutilizó Angular activo en :4200 y PostgreSQL histórico en :5433, con 28 tablas.
- Se iniciaron Spring :8080 con `ddl-auto=validate` y Payment Service :9090; Angular /home, Spring /v3/api-docs y Payment /health respondieron HTTP 200.
- Se ingresó con cuentas de cliente y worker mediante login normal. Los formularios de auth se capturaron vacíos, sin publicar credenciales.
- Se obtuvieron 19 capturas del producto conectado y Swagger auténtico, incluyendo mapa Leaflet/OpenStreetMap.
- Se consultaron estados históricos OPEN, MATCHED, IN_PROGRESS y COMPLETED, además de acuerdo, presupuesto, HELD y review. Se explica la inconsistencia histórica de la solicitud en progreso con pago pendiente; no se presenta como el happy path actual.
- No se publicaron solicitudes ni se ejecutaron pagos, preferences, reconciliación, webhook, review o payout. El borrador se detuvo en revisión de publicación.
- La liquidación automática se desactivó sólo para este proceso de captura. El Golden Path completado sigue HELD y no se considera settlement PASS.
- Se observaron dos notificaciones ResizeObserver en consola. No se declara ausencia total de errores.
- La [galería conectada](09-visual-tour.md) reemplaza a la demo como evidencia visual principal; la [demo anterior](12-demo-visual-archive.md) queda separada.

No se reejecutó la suite privada de backend/frontend/Go para un cambio exclusivamente documental. Las ejecuciones históricas están etiquetadas como tales. No se afirma un nuevo build privado PASS.

## 8. Comandos de referencia, no nuevas ejecuciones E2E

Estos comandos orientan la infraestructura existente; para repos privados requieren checkout, JDK y configuración local adecuados. No son instrucciones para probar cobros productivos.

```sh
# Backend Spring: compilar y tests
./mvnw test
./mvnw package

# Payment Go: tests aislados
# Ejecutar en el repositorio Payment, revisando configuración de tests.
go test ./...

# Demo pública
bun run typecheck
bun run test -- --watch=false
bun run build
```

Windows usa `mvnw.cmd`. No se requiere que un lector ejecute servicios privados para estudiar los diagramas. Las credenciales nunca se incluyen en comandos públicos.

## 9. Pendientes de validación

Prioridad: webhook REAL firmado completo; Kafka con broker real, retries/DLT y reentrega; recovery ante timeout/caída; locking e índices en PostgreSQL; responsive/accesibilidad completa; seguridad de logs y reverse proxy; configuración de release/payout por producto del proveedor.

Los próximos tests deben cubrir comportamiento e integridad, no sólo aumentar un contador de coverage. [Backlog y decisiones](10-decisions-and-roadmap.md).
