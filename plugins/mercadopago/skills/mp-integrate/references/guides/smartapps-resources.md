# Guía: Recursos externos de SmartApps
# Actualizado: 2026-10-06
#
# Fuentes de consulta del asesor de SmartApps (`guides/smartapps.md`) cuando la
# guía incluida y el MCP autenticado no responden una pregunta. Solo se listan
# recursos públicos. No agregues recursos internos, privados ni propios de una
# empresa a este archivo.

---

## Orden de consulta

Consulta las fuentes en este orden y detente en la primera que responda con
evidencia verificable. Respeta el límite del plugin de un WebFetch por
interacción; prioriza el MCP y los archivos locales por sobre las consultas web.

| # | Fuente | Úsala para | Acceso |
|---|---|---|---|
| 1 | MCP de Mercado Pago `search_documentation` | Guía vigente de SmartApps, restricciones, disponibilidad por país, homologación | MCP autenticado (bajo demanda) |
| 2 | `developers/llms.txt` oficial del país | Índice de las páginas oficiales vigentes | `https://www.mercadopago.com.{country}/developers/llms.txt` (un WebFetch) |
| 3 | Documentación oficial de SmartApps | Etapas de integración, funcionalidades de la terminal (Bluetooth, cámara, impresión, pantalla completa, información de la terminal), restricciones, seguridad | `https://www.mercadopago.com.{country}/developers/{lang}/docs/smartapps/overview` |
| 4 | Releases del SDK | Último `Third-Apps-integration-kit.zip` (AAR, zip de documentación del SDK, APK demo) y notas de versión | `https://github.com/mercadopago/point-smartapp-demo-android/releases` |
| 5 | Repositorio demo | Ejemplo de uso de cada capacidad del SDK, `CHANGELOG.md`, configuración de Gradle y manifest | `https://github.com/mercadopago/point-smartapp-demo-android` |
| 6 | Zip de documentación del SDK | Referencia de métodos y clases de la versión exacta del SDK en uso | Dentro del kit de la release; léelo localmente después de que el desarrollador lo descargue |
| 7 | Issues del repositorio demo | Problemas conocidos reportados por otros integradores | `https://github.com/mercadopago/point-smartapp-demo-android/issues` |

## Skills relacionados del plugin

Usa estos skills del plugin en lugar de responder desde la guía de SmartApps:

| Necesidad | Skill |
|---|---|
| Webhooks o notificaciones del backend que concilia los pagos de la SmartApp | `mp-webhooks` |
| Usuarios, credenciales o tarjetas de prueba | `mp-test-setup` |
| Revisión de calidad o preparación para homologación de una integración existente | `mp-review` |

## Reglas

- Cita la fuente usada en cada respuesta que salga de este archivo.
- No trates foros, blogs ni repositorios de terceros como fuentes oficiales. Si
  son la única evidencia, marca la respuesta como **no verificada** y no la
  conviertas en código.
- No descargues artefactos sin autorización explícita (ver las reglas de
  obtención del SDK en `guides/smartapps.md`).
- Si ninguna fuente responde la pregunta, sigue `guides/smartapps-feedback.md`.
