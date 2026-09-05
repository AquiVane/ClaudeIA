# HANDOFF — IA de COSMART (ia.cosmart.com.ar)

Actualizado: 2026-09-05. Todavía no hay código — este archivo tiene el pedido original de Vaneh más una segunda vuelta de requisitos/decisiones de la misma fecha. Leer todo antes de tocar nada.

## Novedades 05/09 (segunda vuelta, antes de codear nada)

Vaneh pidió que "todo sea gratis" y tiene ya dominio + GitHub + Cloudflare (donde vive el resto de los workers de COSMART). Se le dio esta guía, pendiente de que confirme los dos puntos marcados abajo antes de empezar a construir:

- **Realidad de costos**: Cloudflare (Workers, D1, R2) cubre gratis casi todo — hosting, DB de usuarios/perfiles, storage de fotos/video/audio (R2: 10GB/mes gratis y **sin costo de egress**, clave para servir video). Lo único que Cloudflare NO tiene es GPU para el paso de generación en sí (face-swap/lip-sync/voice-clone) — Workers AI no trae ese tipo de modelo. Para eso hace falta un servicio externo tipo Replicate/fal.ai (pago por uso, centavos por video) o alquiler de GPU (Runpod/Vast.ai) si el volumen crece. Para 2 usuarios (Vaneh + Ger) el costo real esperado es de un par de dólares por mes en el peor caso, no estrictamente $0 — se lo aclaramos así para que no la sorprenda después.
- **⚠️ Rascado de Instagram/TikTok — recomendación: NO construir un scraper, pendiente de confirmación de Vaneh.** Aunque la idea de "solo su propio perfil, detectable porque ya subió 50-100 fotos" es una buena idea de control antifraude, scrapear de forma automatizada viola los ToS de Instagram/TikTok igual sea perfil propio o ajeno — para 2 usuarios no vale la pena el riesgo legal ni el mantenimiento (se rompe cada vez que la plataforma cambia el HTML). Alternativa propuesta: subida manual de archivos que cada uno ya puede descargar de su propia cuenta (Instagram/TikTok lo permiten nativamente) — mismo resultado, sin scraper. **Si Vaneh insiste en el scraper, hay que retomar esta conversación antes de construir nada ahí.**
- **Cantidad de fotos/video/audio recomendada** (para lograr parecido fiel, no una cara genérica): 30-50 fotos variadas (ángulos/luz/expresión distintos, más importa la variedad que la cantidad), 3-5 clips de video de 15-60 seg hablando de frente (esto es lo que más ayuda a no "cambiar la cara", captura gestos/movimiento que una foto no tiene), 5-10 min de audio limpio para clonar voz.
- **DNI + términos de uso (antifraude/legal)**: idea validada por Vaneh, pero **se recomienda implementarlo recién cuando haya usuarios externos**, no para Vaneh/Ger ahora (ellos se conocen, no hace falta la fricción). Cuando se implemente: el DNI es dato personal sensible bajo ley argentina de protección de datos, va a necesitar manejo más cuidadoso que una foto común (cifrado, acceso restringido) — dejarlo diseñado pero no es parte del MVP.
- **Modelo de usuarios confirmado**: rol **admin** (Vaneh como creadora, Ger) = storage y generaciones sin límite. Rol **user** (futuros registros externos, van a pagar) = con límites. Cada persona con su login/sesión propia, perfiles nunca compartidos.
- **Cómo pensar los límites/planes futuros**: el storage de entrada (fotos/audio/video) NO es el cuello de botella (R2 gratis alcanza de sobra, un perfil pesa ~500MB-1GB). El costo real que escala es la **generación (GPU)** — por eso los planes pagos futuros deberían armarse alrededor de "cuántos videos podés generar por mes" (+ retención de los generados, ej. se borran a los 30 días si no se bajaron), no de gigas de storage. Precio pensado para ser muy barato y masivo, no premium.

**Antes de empezar a construir código, falta que Vaneh confirme**: (1) subida manual en vez de scraper de RRSS, (2) dejar DNI/verificación para más adelante. Con eso confirmado, el siguiente paso lógico es: login de 2 usuarios (Vaneh/Ger) + subida de fotos/video/audio a R2.

## Qué es esto

Una IA propia de COSMART para generar videos a partir de fotos, audios y guiones — pensada para resolver un problema puntual de Vaneh: no tiene tiempo para crear contenido, y las herramientas que probó o la hacen mal (le cambian la cara) o son inaccesibles en precio. Ella la describe como su proyecto más grande desde que empezó, con expectativa fuerte de impacto económico.

Va a alojarse en **`ia.cosmart.com.ar`**.

## Uso previsto: privado, no público

Por ahora la usan solo **Vaneh y Germán (Ger)** — no un producto para el público. Cada uno tendría su propio perfil dentro de la herramienta.

## Requisitos tal como los planteó Vaneh (05/09)

- Cada persona (Vaneh, Ger) crea su propio perfil, subiéndole a la IA:
  - Una **biblioteca de fotos** — la cifra que tiró Vaneh como referencia es ~50 imágenes por persona — para que el resultado sea **fiel a la cara real**, no una aproximación genérica. Esto es explícitamente el punto que hoy le falla a las alternativas que probó.
  - **Links de videos** que la IA pueda extraer ("rascar") de redes sociales y de cualquier web.
  - **Audios y guiones** propios, para generar el video final (la persona hablando ese guion, con su cara y voz).
- Motivación explícita: "todas andan como el tuje y ninguna me lee bien" — la calidad/lectura de guion de las herramientas que probó no le sirve.

## Alternativas que Vaneh ya evaluó (contexto competitivo)

- **Social Boots**: según ella es la que mejor funciona hoy, pero "a mí personalmente me cambia la cara" — falla justo en el punto de fidelidad que más le importa.
- Una herramienta que menciona como "See Dance" (nombre tal cual lo dijo, sin confirmar cuál es exactamente — podría ser una referencia a Sora o similar) y "los chinos hicieron la suya": la entiende cara / inaccesible en precio.

## Estado actual

Solo idea/planificación, recién anotada. Sin decisiones técnicas tomadas, sin código, sin scaffolding de proyecto todavía.

## Pendiente (arranque del proyecto, todo abierto)

- Definir arquitectura técnica: ¿API de terceros existentes para face-swap/lip-sync/voice-clone/text-to-video, fine-tuning sobre un modelo abierto, o desarrollo propio? No decidido, hay que scopearlo con Vaneh.
- El "rascado" de videos de RRSS y webs de terceros que pide Vaneh tiene implicancias legales/de términos de servicio reales (scraping de redes sociales ajenas) — evaluarlo con cuidado antes de construir nada ahí, no asumir que es tan simple como bajar un video de una URL.
- Definir dónde y cómo se guardan de forma privada las fotos/videos/audios de cada perfil — son datos biométricos/de imagen real de Vaneh y Germán, no contenido de cliente cualquiera.
- Definir stack, hosting y cómo se conecta (si corresponde) con el resto del ecosistema COSMART (`training`, `cosmart-workers`, `hub`).
- Nada de esto está confirmado — es el primer volcado de la idea, a desarrollar en próximas sesiones.
