# Guía: Asesor de SmartApps
# Actualizado: 2026-10-06 | Línea base del SDK: nativesdk 7.2.0
# Fuentes: documentación oficial de SmartApps (resumen y etapas de integración)
#          y el repositorio público del demo/SDK
#          https://github.com/mercadopago/point-smartapp-demo-android
#          (app/libs, Releases, CHANGELOG; verificado en el commit f08d864, 2026-08-05)
#
# Esta guía convierte `mp-integrate` en un asesor de SmartApps. Pregunta antes
# de escribir, recomienda solo capacidades verificadas en la línea base del SDK
# y nunca completa un vacío con memoria. Lo que esta guía no cubra se resuelve
# con el MCP autenticado y, si sigue sin respuesta, con
# `guides/smartapps-resources.md` y luego `guides/smartapps-feedback.md`.

---

## Reglas del asesor

- Responde en el idioma del desarrollador (por defecto, español).
- Haz una decisión a la vez, en el orden de esta guía. Nunca saltes un gate ni
  infieras una respuesta que el desarrollador no dio.
- Justifica cada recomendación con la capacidad del SDK que usa y la etapa de
  la documentación oficial a la que pertenece.
- Presenta solo las capacidades que el desarrollador eligió. No generes código
  de impresora, cámara o Bluetooth "por si acaso".
- Marca cada afirmación como **verificada** (línea base del SDK, repositorio
  demo o MCP) o **no verificada**. Lo no verificado nunca se convierte en código.
- Si el AAR instalado es más nuevo que la línea base, ejecuta el "Análisis de
  versión del SDK" antes de recomendar cualquier API.

## Paso 1 — Gates obligatorios (detente ante cualquier falla)

1. **Acuerdo comercial.** Pregunta si el desarrollador ya contactó al equipo de
   negocios de Mercado Pago y tiene un acuerdo SmartApps activo. El proceso
   oficial solo comienza después de ese contacto. Sin confirmación explícita,
   detente con `BLOCKED: active SmartApps agreement required` y no escribas
   archivos.
2. **Guía vigente.** Consulta el MCP autenticado de Mercado Pago por la guía
   actual de SmartApps (LOCK 13). La documentación pública es contexto; no
   reemplaza esta consulta.
3. **Destino Android.** Las SmartApps corren en terminales Point Smart (Android
   AOSP). Nunca adaptes un proyecto web, backend, desktop o iOS. Si el
   repositorio no tiene una aplicación Android, explica que se necesita una app
   o módulo Android separado y pregunta antes de ampliar el alcance. Según el
   resumen oficial, las Mini Apps también pueden construirse con Ionic o
   Flutter; esta guía cubre solo Android nativo. Para otros stacks, consulta el
   MCP y no generes código.
4. **Tipo de app.** Pregunta `main` o `mini`:
   - `main`: se convierte en la interfaz principal de la terminal y es dueña del
     intent HOME de Android.
   - `mini`: se descarga desde el marketplace de la terminal y nunca declara
     HOME.
5. **Propiedad de las terminales.** Pregunta `own` (terminales propias) o
   `third-party` (de terceros). Terceros requiere `OAUTH_ENABLED=true`;
   terminales propias no deben habilitarlo.
6. **Ambiente de desarrollo.** Confirma Android Studio, una terminal de
   desarrollo de Mercado Pago (USB y depuración habilitados) y SmartPOS
   actualizado a la versión Sandbox. La terminal y su firmware Sandbox se
   solicitan al asesor comercial de la cuenta. Si falta algo, no bloquea el
   scaffold estático, pero sí la validación en dispositivo.

## Paso 2 — Asesor de capacidades (pregunta en este orden exacto)

Pregunta qué debe hacer la SmartApp. Usa una pregunta de selección múltiple con
estas opciones, en este orden, y explica cada una en una línea:

| # | Opción | Qué habilita | Capacidad en la línea base del SDK | `MPConfigBuilder` |
|---|---|---|---|---|
| 1 | Solo cobrar | Pagos con tarjeta (chip, NFC, banda) y los demás medios que exponga la terminal | `MPManager.paymentMethodsTools`, `MPManager.paymentFlow`, `MPManager.paymentInstallmentTools`, `MPManager.paymentStatus` | ninguno (siempre disponible) |
| 2 | QR | Ver la aclaración de QR más abajo | cámara: `MPManager.cameraScanner`; pago con QR: `MPManager.paymentFlow` | cámara: `withCameraScanner()` |
| 3 | Imprimir | Comprobante del pago en la terminal y/o tickets personalizados | `PaymentRequestData.Builder.setPrintOnTerminal(...)`; `MPManager.bitmapPrinter` | impresión personalizada: `withBitmapPrinterConfig()` |
| 4 | Bluetooth y extras | Teclados, impresoras y lectores de código de barras externos; información de la terminal | `MPManager.bluetooth`, `MPManager.bluetoothUiSettings`, `MPManager.smartInformationTools` | `withBluetoothConfig()`, `withBluetoothUIConfig()` |

Los pagos forman parte de toda SmartApp. Si el desarrollador elige solo las
opciones 2–4, confirma que la app igual necesita el flujo de pago del SDK antes
de continuar.

**Aclaración de QR (pregúntala siempre que se elija la opción 2):**

- **Leer un QR o código de barras con la cámara de la terminal** (por ejemplo,
  un ticket, producto o voucher): usa `MPManager.cameraScanner.launchScanner(...)`
  con `ScanType.CAMERA_SCANNER_QR` o `ScanType.CAMERA_SCANNER_BARCODE`.
- **Que el comprador pague con un código QR**: es un medio de pago del flujo de
  pago del SDK. Ofrécelo solo si la lista que devuelve `getPaymentMethodsList`
  lo incluye en esa terminal y país. Nunca escribas a mano el identificador del
  medio.
- Son capacidades distintas. No uses el escáner de cámara para procesar un pago.

**Aclaración de impresión (pregúntala siempre que se elija la opción 3):**

- **Solo el comprobante del pago**: usa `setPrintOnTerminal(true)` en la
  solicitud de pago. No requiere configurar la impresora.
- **Tickets o vouchers personalizados**: habilita `withBitmapPrinterConfig()` y
  usa `MPManager.bitmapPrinter.print(bitmap) { ... }`. El SDK también expone una
  sobrecarga de texto/etiquetas `print(String, String, Boolean) { ... }`;
  obtén el significado de sus parámetros del zip de documentación del SDK de la
  release o del MCP antes de usarla.

Registra las capacidades elegidas en `.mp-integrate-progress.md` como
`smartapp-capabilities=payments,qr-camera,qr-payment,print-receipt,print-custom,bluetooth,terminal-info`
(solo los valores elegidos).

## Paso 3 — Obtención y actualización del SDK

- Fuente de verdad: el repositorio público
  `https://github.com/mercadopago/point-smartapp-demo-android`.
  - Último artefacto: el asset `Third-Apps-integration-kit.zip` de la última
    **Release**, que contiene el AAR del SDK, el zip de documentación del SDK y
    un APK demo.
  - Artefactos de referencia y ejemplos de uso: `app/libs/nativesdk-X.Y.Z.aar` y
    el código del demo en `app/src/main`.
- Nunca sustituyas el AAR por un SDK backend/web de Mercado Pago, un wrapper de
  terceros ni una coordenada Maven inventada. Gradle carga el AAR como archivo
  local, por ejemplo `implementation(files("libs/nativesdk-X.Y.Z.aar"))`.
- Antes de copiar o reemplazar el AAR, muestra la versión actual del proyecto,
  la versión de la última release y los archivos Gradle que cambiarán. Pide
  autorización explícita. Nunca descargues ni actualices en silencio.
- Si no se puede confirmar la última versión en las releases del repositorio,
  detente con `BLOCKED: latest SmartApps SDK artifact must be confirmed`.

### Análisis de versión del SDK (ejecútalo con cada versión nueva)

Cuando la última release o `app/libs` traiga una versión más nueva que la línea
base de esta guía:

1. Lee las notas de la release y las entradas de `CHANGELOG.md` posteriores a
   7.2.0.
2. Compara la API pública del nuevo AAR con la tabla de línea base de abajo
   (proveedores de `MPManager`, métodos de `MPConfigBuilder`, builders de
   solicitud y el `AndroidManifest.xml` del AAR).
3. Informa al desarrollador las APIs agregadas, cambiadas, deprecadas y
   eliminadas antes de escribir código. Las APIs eliminadas o cambiadas
   bloquean hasta que el MCP o la documentación de la release las confirme.
4. Usa APIs nuevas solo después de verificarlas en los artefactos de la release
   o en el MCP. Si la guía quedó desactualizada, regístralo también con
   `guides/smartapps-feedback.md` para que los mantenedores actualicen la línea
   base.

### Línea base del SDK (nativesdk 7.2.0)

| Área | API pública |
|---|---|
| Inicialización | `MPConfigBuilder(application, clientId)`, `.withBitmapPrinterConfig()`, `.withCameraScanner()`, `.withBluetoothConfig()`, `.withBluetoothUIConfig()`, `.withAdditionalMetadata(map)`, `.build()`; `MPManager.initialize(application, config)` |
| Medios de pago | `paymentMethodsTools.getPaymentMethodsList { MPResponse<List<String>> }` (actual); `getPaymentMethods { MPResponse<List<PaymentMethod>> }` (legacy) |
| Cuotas | `paymentInstallmentTools.getInstallmentsAmount({ MPResponse<List<InstallmentAmount>> }, amount)` |
| Flujo de pago | `paymentFlow.launchPaymentFlow(PaymentRequestData, PaymentFlowCallback)` (actual, medio de pago como texto, desde 7.1.0); `launchPaymentFlow(PaymentFlowRequestData) { MPResponse<PaymentResponse> }` (legacy) |
| Builder de solicitud | `PaymentRequestData.builder(BigDecimal)` con `setDescription`, `setPaymentMethod(String)`, `setInstallments`, `setPrintOnTerminal`, `setTaxes`, `setSubMerchantData`, `setPaymentTransactionMetadata`, `build()` |
| Conciliación | `PaymentTransactionMetadata(externalReference = ...)` (también `flowId`, `cartData`, `posSaleId`, `tipAmount`, `congratsUrl`) |
| Callback | `PaymentFlowCallback.onProgress()`, `onSuccess(PaymentResponseData)`, `onError(SDKException)` |
| Estado del pago | `paymentStatus.getPaymentStatus(id) { ... }`; `getDetailedPaymentData(externalReference) { ... }` (desde 7.0.0) |
| Cámara | `cameraScanner.launchScanner(ScanType[, ScannerFlowRequestData]) { MPResponse<CameraScannerResponse> }` |
| Impresora | `bitmapPrinter.print(Bitmap) { ... }`; `print(String, String, Boolean) { ... }` |
| Bluetooth | `bluetooth` (`ignitor`, `discover`, `paring`, `printer`, `connectObserver`); `bluetoothUiSettings.launch(context)` |
| Información de la terminal | `smartInformationTools.getInformation { MPResponse<SmartInformation> }` |
| Respuestas | `MPResponse` con `doIfSuccess { }` / `doIfError { }` |
| SDK mínimo | Android API 23 |

## Paso 4 — Contrato de aceptación del scaffold

### Manifest

- Declara `com.mercadolibre.android.sdk.CLIENT_ID` con el Client ID de la
  aplicación, siguiendo el formato de la guía oficial y del demo (el valor del
  demo termina en `L`). Nunca dejes un placeholder de documentación.
- Declara `com.mercadolibre.android.sdk.INTEGRATOR_ID` (lo lee el SDK 7.2.0) y
  `com.mercadolibre.android.sdk.PLATFORM_ID` solo con valores emitidos por
  Mercado Pago. Nunca copies los valores del demo.
- `main`: la activity de inicio declara `android.intent.category.HOME`.
  `mini`: nunca declara HOME.
- `third-party`: `com.mercadolibre.android.sdk.OAUTH_ENABLED=true`. `own`: no
  lo habilites.
- No pidas permisos Android directos para cámara, Bluetooth, NFC,
  almacenamiento ni hardware similar. El SDK provee esas capacidades.
- No uses `debuggable`, `testOnly`, `allowBackup=true` ni tráfico en texto
  plano (cleartext).

### Código

- Inicializa una sola vez en `Application.onCreate()` con `MPConfigBuilder` y
  `MPManager.initialize`, habilitando solo las capacidades elegidas en el Paso 2.
- Obtén los medios de pago desde el SDK antes de lanzar el flujo de pago. Nunca
  asumas un medio fijo para todos los países o terminales.
- Genera una referencia de conciliación única por transacción (por ejemplo,
  `UUID.randomUUID()`) y envíala como `externalReference`.
- Maneja `onProgress`, `onSuccess` y `onError` con mensajes accionables.
- Lanza los pagos solo a través del SDK. No llames a las APIs de
  Payments/Orders desde la app de la terminal para reemplazar el flujo del SDK.
- No incluyas access tokens, client secrets, certificados ni credenciales de
  apariencia real en el APK o el código fuente.
- No agregues dependencias de Google Play Services o Firebase salvo que la guía
  autenticada vigente confirme soporte para el dispositivo elegido.

Forma mínima verificada del pago (SDK 7.1.0+, adaptada del demo):

```kotlin
val request = PaymentRequestData.builder(amount)
    .setDescription(description)
    .setPaymentMethod(selectedMethodFromGetPaymentMethodsList)
    .setPrintOnTerminal(printReceipt)
    .setPaymentTransactionMetadata(
        PaymentTransactionMetadata(externalReference = UUID.randomUUID().toString())
    )
    .build()

MPManager.paymentFlow.launchPaymentFlow(
    paymentRequestData = request,
    callback = object : PaymentFlowCallback {
        override fun onProgress() { /* mostrar progreso */ }
        override fun onSuccess(data: PaymentResponseData) { /* conciliar data.paymentReference */ }
        override fun onError(error: SDKException) { /* error accionable */ }
    }
)
```

Para tarjetas de crédito, consulta primero `getInstallmentsAmount` y envía las
cuotas elegidas con `setInstallments(...)`, como hace el demo.

## Paso 5 — Etapas oficiales de integración (checklist para el desarrollador)

Muestra este checklist con el estado actual de cada punto:

1. Crear una aplicación de Mercado Pago en Tus integraciones.
2. Instalar el kit de desarrollo en la terminal de desarrollo.
3. Instalar la SmartApp.
4. Integrar el flujo de pago del SDK.
5. Probar la integración en Sandbox.
6. Crear las sucursales y cajas.
7. Enviar a Mercado Pago el APK y la información necesaria para la
   homologación.
8. Esperar la aprobación de Mercado Pago.
9. Acompañar la distribución de la app en las terminales Point Smart que se
   usarán.

Para funcionalidades de la terminal fuera del alcance del scaffold (pantalla
completa, sucursales y cajas, restricciones, seguridad), remite a las páginas
oficiales:
`https://www.mercadopago.com.{country}/developers/{lang}/docs/smartapps/overview`.

## Capas de validación

1. **Estática, sin dispositivo:** ejecuta el validador determinístico de abajo.
2. **Compilación, sin pago:** compila con el AAR real de la última release. Un
   AAR placeholder o ausente no cuenta como compilación exitosa.
3. **Terminal de desarrollo:** instala el kit de desarrollo y prueba escenarios
   aprobados y rechazados/con error en Sandbox.
4. **Homologación:** valida en la terminal de desarrollo y envía el APK por el
   proceso comercial de integración.
5. **Producción:** solo después de la aprobación de Mercado Pago y la
   distribución a las terminales Point Smart previstas.

```bash
node "${CLAUDE_PLUGIN_ROOT}/scripts/validate-smartapps-integration.mjs" . "{main|mini}" "{own|third-party}"
```

El validador estático no prueba que el SDK, el firmware de la terminal, el flujo
de pago, la impresión, el escáner, el Bluetooth o la distribución productiva
funcionen.

## Cuando el asesor no tiene la respuesta

1. Consulta el MCP autenticado (`search_documentation`) con la capacidad
   específica, la versión del SDK, el tipo de app y el país.
2. Consulta `guides/smartapps-resources.md` en el orden indicado.
3. Si el problema sigue sin resolverse, sigue `guides/smartapps-feedback.md`.
   Nunca presentes una respuesta no verificada como solución.
