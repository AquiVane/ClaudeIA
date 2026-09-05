# HANDOFF — IA de COSMART (ia.cosmart.com.ar)

Actualizado: 2026-09-05. Primera sesión de este repo — todavía no hay arquitectura ni código, esto documenta el pedido inicial de Vaneh tal cual lo planteó (05/09), para no perderlo ni tener que repreguntarlo.

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
