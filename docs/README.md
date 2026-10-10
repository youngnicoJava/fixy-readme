# Documentación técnica de Fixy

[Volver al README](../README.md) · [Demo interactiva](https://youngnicojava.github.io/fixy-demo-frontend/)

Esta documentación describe tres proyectos privados que cooperan: frontend Angular, backend Spring Java/Kotlin y adaptador Payment en Go. La demo pública permite explorar la UX sin ejecutar esos servicios.

## Ruta de lectura

1. [Arquitectura](01-architecture.md): responsabilidades y comunicación entre procesos.
2. [Dominio](02-domain-and-lifecycle.md): qué se contrata y cómo cambian los estados.
3. [Backend](03-backend-engineering.md): implementación, integridad y concurrencia.
4. [Seguridad](04-security-and-auth.md): sesión, autorización y fronteras internas.
5. [Payment](05-payment-and-events.md): checkout, confirmación, recuperación y Kafka.
6. [API](06-api-contracts.md): contratos HTTP y ejemplos explicativos.
7. [Frontend y demo](07-frontend-and-demo.md): composición de pantallas y simulación.
8. [Testing](08-testing-and-evidence.md): qué evidencia existe y qué falta demostrar.
9. [Galería conectada](09-visual-tour.md): frontend privado activo y Swagger real.
10. [Decisiones y backlog](10-decisions-and-roadmap.md): alcance deliberado y trabajo pendiente.
11. [Recorrido de la demo](11-demo-walkthrough.md): pasos de la aplicación pública independiente.
12. [Archivo visual de la demo](12-demo-visual-archive.md): material anterior separado del frontend conectado.

## Cómo interpretar las afirmaciones

| Etiqueta | Significado |
|---|---|
| Implementado | Existe una ruta de código inspeccionada; no implica despliegue productivo |
| Test de regresión | Hay test que protege una regla; revisar si usa mocks o DB de test |
| Evidencia histórica PASS | Existe resultado E2E guardado; no se reejecutó para escribir estos docs |
| Verificado visualmente | Pantalla abierta y capturada en navegador para esta publicación |
| Pendiente | Falta implementación o evidencia necesaria para declararlo completo |
| Propuesto | Recomendación; no se presenta como comportamiento actual |

La fuente principal es el código actual revisado el **10/10/2026**. Se contrastó con los informes internos de pruebas; se publican únicamente conclusiones sanitizadas. No se copian dumps, credenciales, logs completos, documentos de identidad, direcciones reales ni IDs de pagos de las cuentas de prueba.

Los nombres de clases y endpoints explican decisiones técnicas. Este repositorio no contiene una copia del código privado ni un SDK público contractual/versionado. Los ejemplos JSON son ilustraciones del contrato, con identidades ficticias; no son registros del E2E privado.
