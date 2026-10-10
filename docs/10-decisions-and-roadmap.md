# 10 — Decisiones, límites y próximos pasos

[Índice](README.md) · [Volver al README](../README.md)

## 1. Decisiones de producto que ordenan el dominio

| Decisión actual | Motivo |
|---|---|
| Solicitud separada de anuncio | Una publicación comercial no es automáticamente contratación |
| Quote estructurado | El precio no depende de interpretar chat libre |
| Agreement snapshot | Mantener trazabilidad del importe y alcance aceptados |
| Derivar acuerdo desde propuesta definitiva | Evitar doble confirmación idéntica |
| Conservar enums históricos | Evolucionar sin migración destructiva innecesaria |
| No cobro automático por horas | Oficios requieren importe acordado antes del pago |
| Completion separado de release | Permitir trabajo finalizado con protección financiera activa |
| Protección capturada en acuerdo | Evitar que un cambio de default altere silenciosamente lo pactado |
| Dispute bloquea settlement | Evitar liberar fondos con reclamo activo |
| Review desde contratación completada | Vincular reputación con trabajo verificable |

Las decisiones representan implementación/reglas actuales; no significan que todas las políticas económicas futuras estén definidas.

## 2. Decisiones técnicas

**Core Spring modular + adapter Go.** Mantiene contratación y estado financiero local juntos, y aísla el contrato del proveedor. No crea un microservicio para cada entidad.

**HTTP para comandos, eventos para confirmación.** Checkout/recovery necesita una respuesta inmediata y sigue siendo HTTP. Kafka es transporte opcional de eventos, no reemplazo universal de integración.

**Reserva duradera antes de checkout.** Un timeout no debe originar un nuevo ID que facilite doble cobro. La operación se conserva para recuperación validada.

**Fail closed ante anomalía.** Múltiples pagos externos no se resuelven eligiendo uno ni aplicando HELD a ciegas. El sistema bloquea y requiere una decisión futura auditada.

**Snapshot y transacción local.** Locks y constraints protegen relaciones dentro de PostgreSQL. No se promete atomicidad con el proveedor ni exactly-once distribuido.

**Demo independiente.** Permite explicar el producto y mostrar el frontend sin publicar core privado, credenciales o infraestructura. Sus simulaciones no se presentan como operaciones reales.

## 3. Límites que requieren definición de negocio

- Cancelación después del match, antes/después de pagar y sus reglas de devolución.
- Refund parcial y operaciones posteriores a liquidación.
- Visita paga: contratación separada, accounting, cancelación y descuento posterior explícito.
- Seña y saldo: total acordado, propósito de cada pago, condición de inicio/completion y devoluciones.
- Adendas/revisiones después de aceptar un acuerdo.
- Políticas de protección por categoría/tipo de trabajo, sin inventar duraciones.
- Tratamiento excepcional de varios cobros aprobados para una referencia.
- Habilitación contractual/técnica del producto del proveedor para settlement y payout.

No se implementan automáticamente como “refactors”: cambian compromisos económicos y deben diseñarse antes de ejecutar dinero.

## 4. Backlog por impacto

| Prioridad | Trabajo | Evidencia de cierre esperada |
|---|---|---|
| P0 | Webhook REAL firmado end-to-end con configuración consistente | Entrega real, HMAC válido, lookup y transición persistida, sin simulación |
| P0 | Cerrar contrato externo de release/settlement/payout | Producto soportado y pruebas seguras, sin endpoint provisional |
| P1 | Control versionado del esquema y DB histórica | Migraciones no destructivas con rollback/backup y tests PostgreSQL |
| P1 | Kafka con infraestructura real y fallos | Reentrega, crash, DLT y recuperación autorizada sin efectos duplicados |
| P1 | Observabilidad financiera y recovery operacional | Correlación sanitizada, alertas PENDING/provider aprobado y anomalías |
| P1 | Endurecer seguridad operativa | Secret rotation, red interna, TLS/ACL de broker, proxy/IP y logs |
| P2 | Reducir queries de listados | Métricas antes/después en conversaciones y Pagos, sin N+1 excesivos |
| P2 | Uniformar errores/contratos | Taxonomía estable, nullability y tests API/UI |
| P2 | Completar UX/responsive/accesibilidad | Recorrido cliente/worker con teclado/móvil y sin estados confusos |
| P2 | Revisar límites transaccionales de módulos financieros parciales | Llamadas externas acotadas y recuperación sin perder operación |
| P3 | Adendas / visita paga / seña-saldo | Sólo después de política económica y modelo auditables |

Flyway y Kafka no son sustitutos de cerrar reglas de negocio. Flyway figura como evolución futura del schema; Kafka ya tiene implementación parcial de transporte que necesita evidencia operativa, no otra reescritura.

## 5. Deuda conocida que no se oculta

- `ddl-auto=update` no ofrece historial controlado de migraciones.
- No hay outbox/inbox general ni ledger contable completo.
- Rate limiting in-memory y secreto interno compartido requieren estrategia productiva.
- Refresh en localStorage no ofrece la protección de cookie HttpOnly.
- Angular aún usa Zone.js; infraestructura server no equivale a SSR/hydration completa.
- Chat por HTTP sin push automático de tiempo real.
- Lint global y advertencias visuales heredadas no resueltos completamente.
- Operaciones de service no siempre tienen endpoint/UI completo.
- Settlement externo no implementado y payout/OAuth no validados como solución productiva.
- Una simulación 200 no demuestra webhook REAL ni cierre del Golden Path financiero por esa vía.

## 6. Qué debe demostrar una futura publicación

Cada nueva afirmación de PASS debería identificar ejecución, fecha, entorno y fuente de evidencia sanitizada. Capturas del producto real, tests aislados y pruebas con proveedor cumplen roles diferentes.

Este repo permanecerá una explicación pública del sistema, no un dump del código privado ni una manera de compartir secretos. La licencia de documentación no cambia la decisión de mantener privado el core.
