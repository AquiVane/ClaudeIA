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

## Novedades 05/09 (tercera vuelta) — CONFIRMADO: los 2 puntos pendientes de arriba, más decisión de arquitectura clave

- ✅ **Vaneh confirmó los 2 puntos pendientes**: nada de scraper de IG/TikTok, y DNI queda para cuando haya usuarios externos.
- **Volumen real de uso aclarado por Vaneh**: cada admin (ella, Ger) va a generar **6-7 videos/día, 30 días/mes** (~180-210 videos/mes por persona), de 40seg-1min, esa cifra diaria ya incluye reintentos/correcciones. Esto cambió la cuenta de costos por completo respecto a lo estimado en la vuelta anterior (que asumía uso ocasional).
- **⚠️ DECISIÓN DE ARQUITECTURA: generación propia sobre GPU alquilada, NO un servicio pago-por-uso de terceros (HeyGen/D-ID/Synthesia/Tavus).** Motivo, con números reales (investigados 05/09, van a cambiar con el tiempo — revalidar antes de comprometerse):
  - Servicio de terceros (HeyGen, el más barato de los grandes): **$1-4 por minuto de video generado**. A este volumen (~400 min/mes combinados entre Vaneh y Ger) sale **$400-1.600/mes**. Inviable.
  - GPU alquilada (Runpod, RTX 4090 community cloud: $0,34/hora) + modelo open source propio: los modelos que existen hoy para esto (Hallo2, LivePortrait, MuseTalk, OmniHuman — ninguno corre en tiempo real, tardan 3-10x la duración del video en procesar) dan un costo de **2-6 centavos de dólar por video**. A este volumen: **~$10-70/mes combinado entre las dos**. 20 a 100 veces más barato.
  - Son estimaciones (no se probó ningún modelo todavía), pero la diferencia de orden de magnitud es tan grande que la decisión no depende de afinar el número.
  - **Pendiente crítico antes de elegir el modelo concreto**: revisar la licencia de cada uno (SadTalker/LivePortrait/MuseTalk/Hallo2/etc.) — varios tienen licencia "solo investigación/no comercial", y esto eventualmente va a ser un servicio pago a terceros, así que hay que elegir uno cuya licencia lo permita (o aceptar el riesgo conscientemente).
  - Usar el modelo elegido vía **Runpod Serverless** (o equivalente) para pagar solo por segundo de cómputo real usado, sin costo cuando está inactivo — coincide con el pedido de Vaneh de que sea "pago por uso" aunque el que lo construya seamos nosotros.
- **✅ Previsualización antes de generar el video completo — requisito aceptado.** Antes de correr la generación completa (cara/voz final), generar un preview rápido y barato (pocos frames o los primeros 2-3 segundos, menos pasos/calidad) para que la persona vea que no salió deforme antes de gastar el costo completo. Reduce directamente el volumen de "correcciones" que ya está contado en las 6-7 generaciones/día.
- **Fuente de fotos/video vs. análisis de contenido — son DOS features distintas, no una:**
  1. **Fotos/video para el perfil (la cara)**: siguen siendo archivos descargados por la persona desde su propio celular/cuenta (no capturas de pantalla — pierden calidad, y la calidad de imagen importa para el parecido). Queda pendiente redactar un instructivo corto de cómo bajar tus propias fotos/videos de Instagram/TikTok, con nota de revisarlo cada tanto porque las apps cambian de interfaz.
  2. **Capturas de pantalla del perfil completo + estadísticas de los últimos 90 días** (full page, imagen o PDF): idea nueva de Vaneh, 100% legal (la persona capturando su propia pantalla, sin automatización). No es para entrenar la cara — es para que la IA entienda qué tipo de contenido/formato/tono ya le funciona a esa persona y lo tenga en cuenta al generar guiones/videos. Feature separada, a diseñar.
  - **Fase 2, no bloquea el arranque**: evaluar las APIs oficiales de Instagram (Graph API de Meta) y TikTok — ambas permiten en modo desarrollo/sandbox que un puñado de cuentas designadas a mano (exactamente el caso: Vaneh y Ger sobre sus propias cuentas) accedan a sus propios datos/estadísticas sin pasar por la revisión pública de la app. Si funciona como se espera, podría reemplazar las capturas manuales de estadísticas con datos reales vía API, sin curro legal. Requiere integrar OAuth de cada plataforma — no es parte del MVP.
- **Automatización de límites según plan pagado (para cuando haya usuarios externos pagos)**:
  - Suba de plan de un usuario: vía webhook del medio de pago (Mercado Pago, ya usado en el resto del ecosistema COSMART) — al confirmarse el pago se actualiza el límite en la base al toque, sin intervención manual. Si el usuario está generando algo en ese momento, no se corta.
  - Infra propia (Cloudflare): con tarjeta cargada y plan pago activado, el excedente sobre el nivel gratis se cobra automático todos los meses — no hay paso de "factura y pagás a mano" que pueda frenar el servicio.
- **Cantidad y variedad de fotos/video/audio — versión ampliada, para usar como copy de onboarding en la app**:
  - Fotos (30-50): importa la **variedad de condiciones** (ángulo: frente/3-4/perfil; luz: natural/interior/contraluz; expresión: neutra/sonriendo/hablando; distancia: primer plano/medio cuerpo/cuerpo entero) mucho más que la cantidad — 25 fotos variadas enseñan más que 50 casi iguales.
  - Video (3-5 clips, 15-60seg): aporta variedad de **movimiento** (gestos, giros de cabeza, parpadeo) que una foto fija no puede dar — es la parte que más ayuda a evitar el efecto "cara rara" que Vaneh reportó en otras herramientas.
  - Audio (5-10 min): variedad de **tono/emoción** (no monótono) para que la voz clonada suene natural en guiones de distinto ánimo.

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
