# 06 — Guía de contratos HTTP

[Índice](README.md) · [Frontend](07-frontend-and-demo.md)

Esta guía se deriva de controllers/DTOs actuales. No sustituye un OpenAPI público estable ni implica que los servicios privados estén desplegados. Las bases locales habituales son Spring `http://localhost:8080` y Go `http://localhost:9090`.

## 1. Autenticación

| Método | Ruta Spring | Contrato / resultado |
|---|---|---|
| POST | `/api/auth/register` | Registro validado, crea CLIENT y devuelve auth; 201 |
| POST | `/api/auth/login` | Credenciales válidas, devuelve auth; 200 |
| POST | `/api/auth/refresh` | Refresh opaco en Authorization Bearer; devuelve ambos tokens; 200 |
| GET | `/api/users/me` | Usuario autenticado |
| PATCH | `/api/users/me` | Actualización permitida de usuario |

`AuthResponse` contiene `accessToken`, `refreshToken`, `user`. No se incluyen ejemplos de tokens reales ni se expone password en una respuesta pública.

La exclusión de JWT en auth permite login/refresh sin access vigente; no permite fabricar refresh ni eludir su validación persistida.

## 2. Perfiles y publicaciones

| Área | Endpoints representativos |
|---|---|
| Cliente | POST/GET `/api/client/profile`; POST addresses; PUT/DELETE `/api/client/addresses/{addressId}`; PATCH active |
| Profesional | POST/GET/PUT `/api/worker/profile`; CRUD de direcciones según controller |
| Consulta profesional pública | GET `/api/worker/public`, `/public/{workerId}`, `/public/nearby`, `/public/top-rated` |
| Administración profesional | GET `/api/worker/admin/pending`; PATCH `/admin/{workerId}/approve`, `/reject`, `/suspend` |
| Publicaciones propias | POST/GET `/api/worker/services` y `/my`; PUT/DELETE `/{offeringId}`; PATCH `/{offeringId}/activate` |
| Publicaciones públicas | GET `/api/worker/services/public/{offeringId}`, `/public/category/{category}`, `/public/featured`, `/public/featured/category/{category}` |
| Destacar publicación | PATCH `/api/worker/services/admin/{offeringId}/feature` y `/unfeature` |

Los endpoints administrativos de worker/offering declaran ADMIN. Los públicos filtran disponibilidad/verificación en su lógica; el significado público proviene de SecurityConfig y del service, no de una etiqueta en Swagger.

## 3. Requests y propuestas

| Método | Ruta | Actor / propósito |
|---|---|---|
| POST | `/api/requests` | Cliente publica necesidad |
| GET | `/api/requests/my` | Solicitudes del cliente |
| GET | `/api/requests/open` | Oportunidades abiertas según reglas/filtros |
| GET | `/api/requests/worker/my` | Trabajo/propuestas del profesional |
| GET | `/api/requests/{requestId}` | Detalle con información de dominio y eventos |
| POST | `/api/requests/{requestId}/proposals` | Worker verificado, categoría correspondiente |
| PATCH | `/api/requests/{requestId}/proposals/{proposalId}/select` | Cliente dueño selecciona |
| PATCH | `/api/requests/{requestId}/start` | Worker seleccionado, request MATCHED, Payment HELD |
| PATCH | `/api/requests/{requestId}/complete` | Cliente dueño, IN_PROGRESS, Payment HELD |
| PATCH | `/api/requests/{requestId}/cancel` | Cliente dueño, OPEN |
| GET/POST | `/api/requests/{requestId}/comments` | Aclaraciones públicas de la solicitud; reglas de estado |

Retirar propuesta tiene lógica en el service, pero no se encontró una ruta de retiro en el controller actual de requests. No se agrega un endpoint inventado a esta guía porque una pantalla/demo pueda simularlo.

## 4. Contratación estructurada

Base: `/api/requests/{requestId}/contract`.

| Método | Sufijo | Regla principal |
|---|---|---|
| GET | vacío | Participante válido; summary de quotes/agreement/visit |
| POST | `/quotes` | Profesional seleccionado/verificado; sin agreement previo |
| POST | `/quotes/{quoteId}/accept` | Cliente dueño; quote SENT y perteneciente a contratación |
| POST | `/quotes/{quoteId}/reject` | Cliente dueño; mismo binding |
| POST | `/visit` | Worker seleccionado, antes de cotizar |
| POST | `/visit/complete` | Visita programada, fecha cumplida, gratuita |

Ejemplo de presupuesto, con datos ficticios:

```json
{
  "laborAmount": 40000,
  "materialsAmount": 15000,
  "scope": "Reemplazo de tramo de cañería y prueba de estanqueidad"
}
```

Ambos importes son requeridos, no negativos, con máximo dos decimales y diez dígitos enteros. El total debe ser positivo y lo calcula el servidor. Scope es obligatorio, máximo 2000 caracteres.

`VisitInput` contiene `price`, `scheduledAt` y `notes`; fecha futura al programar, precio no negativo y notas de hasta 1000 caracteres. Precio positivo no convierte la visita en pagada: su flujo financiero sigue pendiente.

`ContractSummary` contiene `serviceRequestId`, `title`, `clientId`, `workerUserId`, `workerName`, `quotes`, `agreement`, `visit`. Agreement y visit pueden ser null; quotes es colección y puede estar vacía. `workerId` en quote/agreement refiere a perfil profesional; `workerUserId` del summary refiere a User. No son intercambiables.

## 5. Payments públicos de Spring

| Método | Ruta | Contrato |
|---|---|---|
| POST | `/api/payments` | CLIENT; body sólo identifica serviceRequestId; 201 |
| GET | `/api/payments/my` | CLIENT; contrataciones y Payment opcional |
| GET | `/api/payments/request/{serviceRequestId}` | CLIENT/WORKER y participante; 200 o 204 |
| POST | `/api/payments/{paymentId}/synchronize` | CLIENT dueño; inspección/recuperación validada |
| POST | `/api/payments/{paymentId}/reconcile` | CLIENT dueño; body providerPaymentId |
| POST | `/api/payments/{serviceRequestId}/release` | CLIENT dueño; elegibilidad y solicitud, 202; integración externa incompleta |

**Atención al ID de release:** esa ruta recibe ServiceRequest ID, no Payment ID. Reconcile/synchronize sí reciben Payment ID.

Ejemplo de creación ficticio:

```json
{ "serviceRequestId": 9001 }
```

Respuesta representativa una vez confirmada; números ficticios, no fixture privado:

```json
{
  "id": 7001,
  "serviceRequestId": 9001,
  "amount": 55000,
  "feeAmount": 4950,
  "status": "HELD",
  "checkoutUrl": null,
  "agreementId": 8001,
  "protectionEndsAt": null,
  "heldAt": "2026-10-10T12:00:00Z",
  "settledAt": null
}
```

`protectionEndsAt` es null antes de completion. Checkout URL sólo se expone para PENDING. `settledAt` corresponde a release confirmado. La UI no debe sustituir null por una fecha inventada.

`GET /my` puede incluir agreement aceptado con `payment: null`, para mostrar pendiente de pago antes de crear una operación. No se crea Payment por consultar el listado.

## 6. Conversaciones y mensajes

| Método | Ruta | Significado |
|---|---|---|
| GET | `/api/conversations/my` | Summaries visibles al participante |
| GET | `/api/conversations/requests/{requestId}` | Recupera conversación existente de contratación |
| POST | `/api/conversations/offerings/{offeringId}` | Crea/recupera contacto desde anuncio disponible |
| GET | `/api/conversations/{conversationId}` | Detalle autorizado |
| POST | `/api/conversations/{conversationId}/close` | Cierre bajo reglas |
| POST | `/api/conversations/{conversationId}/reopen` | Request en dispute; no reapertura general de offering |
| GET/POST | `/api/messages/conversations/{conversationId}` | Lectura/envío para participantes |
| DELETE | `/api/messages/{messageId}` | Borrado lógico por sender propietario |

La selección de propuesta crea la conversación de request. El GET por request del controller no debe interpretarse como creación libre antes del match aunque exista un método get-or-create en el service.

Summary/detail incluyen `counterpart` con `userId`, `name`, `surname`, `profilePhotoUrl`. La foto de worker se incluye cuando corresponde; el cliente no tiene una foto inventada si el modelo no la expone. `lastMessage.senderId/senderName` identifica emisor del mensaje y no reemplaza counterpart.

## 7. Reviews y disputes

| Método | Ruta | Semántica |
|---|---|---|
| GET | `/api/reviews/requests/{requestId}` | Review singular opcional; participante; 200/204 |
| POST | `/api/reviews/requests/{requestId}` | Cliente dueño; COMPLETED; una review; 201 |
| GET | `/api/reviews/workers/{workerProfileId}` | Colección de reviews; vacía si no hay |
| GET | `/api/reviews/workers/{workerProfileId}/rating` | Promedio, cero si no hay reviews |
| GET | `/api/disputes/requests/{requestId}` | Dispute singular opcional; participante; 200/204 |
| POST | `/api/disputes/requests/{requestId}` | Abrir reclamo bajo reglas de protección |
| GET | `/api/disputes/{disputeId}` | Detalle autorizado |
| GET | `/api/disputes/my` | Reclamos abiertos por usuario |
| GET/POST | `/api/disputes/{disputeId}/evidence` | Evidencia de participante; agregar exige dispute activa |

Hay operaciones administrativas/cancelación en DisputeService que no se presentan como rutas públicas si el controller inspeccionado no las expone. La existencia de lógica interna no demuestra UI completa de gestión de reclamos.

## 8. Tráfico interno y recursos auxiliares

| Servicio | Ruta | Frontera |
|---|---|---|
| Spring | POST `/api/payments/internal/events` | X-Internal-Secret; aplicar evento bajo validaciones |
| Spring | POST `/api/payouts/internal/{payoutId}/complete` | Callback interno de retiro; módulo parcial |
| Go | POST `/internal/mp/preferences` | Secreto interno + idempotency key |
| Go | POST `/internal/mp/payments/inspect` | Consulta externa read-only |
| Go | POST `/internal/mp/payments/reconcile` | Validación externa, Spring aplica estado |
| Go | POST `/internal/mp/payments/{providerPaymentId}/release` | Actualmente 501, no implementado |
| Go | POST `/internal/mp/oauth/exchange` | Cuenta vinculada OAuth |
| Go | POST `/internal/mp/payouts` | Integración provisional; no validada como payout productivo |
| Go | POST `/webhooks/mercadopago` | Público con firma HMAC |
| Go | GET `/health` | Salud básica del proceso, no certifica todas las dependencias |
| Spring | GET `/api/location/search`, `/geocode`, `/reverse-geocode` | HERE |

Spring tiene además `/api/payment-accounts/link`, `/me`, `/api/worker-earnings/me`, `/me/balance` y `/api/payouts`. Su presencia no acredita cuenta OAuth, retiro ni settlement listos para uso real.

## 9. Semántica HTTP

| Código | Lectura correcta en estos flujos |
|---|---|
| 200 / 201 | Lectura/operación exitosa según endpoint |
| 202 | Solicitud aceptada, no confirma liberación del dinero |
| 204 | Recurso asociado opcional todavía ausente |
| 400 | DTO inválido o condición de negocio representada como bad request |
| 401 | Autenticación/firma inválida según frontera |
| 403 | Usuario no autorizado / no participante / no habilitado |
| 404 | Request/recurso solicitado no encontrado; no ocultarlo genéricamente |
| 409 | Conflicto de estado, duplicado, acuerdo incoherente o anomalía externa |
| 429 | Rate limiting |
| 501 | Operación externa no implementada |
| 502 / 503 | Proveedor/transport/configuración no disponible según capa |

No todos los errores comparten body JSON. El frontend debe tipar ausencias válidas sin convertir cualquier error en null. Los endpoints de colección no usan 204 como sinónimo genérico de lista vacía.

## 10. Swagger/OpenAPI

El backend incorpora springdoc; las rutas locales habituales son `/swagger-ui/index.html` y `/v3/api-docs`. Se permiten en la configuración de seguridad inspeccionada.

No se publica un Swagger inventado ni se exporta automáticamente un OpenAPI desde código sin ejecutar la aplicación como si fuera respuesta real. Para esta revisión el servicio local no estuvo disponible; se documentan controllers/DTOs y queda pendiente incorporar una captura auténtica cuando esté activo.
