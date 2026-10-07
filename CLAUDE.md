# IPA Translator (antes "IPA Keyboard")

Página web estática, en inglés, que convierte texto a transcripción fonética IPA y ofrece un teclado de símbolos IPA para escribir fonética a mano. Autor: m2aq. Idea original: Omar Gámez.

- Repositorio: https://github.com/m2aq/ipa_translator (público; antes se llamaba `ipa_keyboard`. GitHub redirige el repositorio viejo, pero la página vieja `m2aq.github.io/ipa_keyboard/` da 404)
- En línea (GitHub Pages, rama `main`, raíz): https://m2aq.github.io/ipa_translator/
- En las dos PC (trabajo y casa) la carpeta local se llama `ipa_translator`, igual que el repositorio y el proyecto (antes se llamaba `ipa-keyboard`).

## Cómo trabajar (dos computadoras: trabajo y casa)

GitHub es el único puente entre las PC. Claude no recuerda nada de una máquina a otra; este archivo es su contexto.

1. **Al empezar a trabajar:** verificar carpeta, rama, `git status -sb` y `git remote -v`; luego `git fetch` y, si no hay cambios locales ni divergencias, `git pull --ff-only`. Si hay cualquier discrepancia, detenerse y avisar. Nunca editar sin traer lo último.
2. **Al terminar:** dejar los cambios listos y avisar si algo queda sin subir. Solo se hace `git add`, `git commit` y `git push` cuando el usuario diga "súbelo". Lo que no se sube no existe en la otra PC.
3. Si el remoto apunta a `ipa_keyboard`, corregirlo: `git remote set-url origin https://github.com/m2aq/ipa_translator.git`.
4. Publicar = push a `main`; GitHub Pages tarda 1-2 min en actualizar. WhatsApp cachea la vista previa de los enlaces: subir primero, compartir después.
5. Este repo es PÚBLICO y GitHub Pages sirve todos sus archivos por URL (incluido este). No poner aquí contraseñas, claves, correos ni datos personales.

## Archivos

| Archivo | Qué es |
|---|---|
| `index.html` | Toda la app (HTML + CSS + JS en un solo archivo, sin build ni dependencias de npm) |
| `cmudict.js` | CMU Pronouncing Dictionary en un string JS (`window.CMU_RAW`), ~3.4 MB, ~125 mil palabras. Generado del `cmudict.dict` oficial quitando variantes `(2)`. Se carga con `<script>` dinámico para que funcione también abriendo el archivo directo |
| `LICENSE-cmudict.txt` | Licencia BSD de CMU (obligatoria; copiada del repo cmusphinx/cmudict) |
| `preview.jpg` | Logo grande: pantalla de carga + imagen de vista previa para WhatsApp/redes (`og:image`, 1200x630) |
| `title.jpg` | Imagen del título (wordmark) que se usa como `<h1>` en la página |
| `apple-touch-icon.png` | Icono para "Añadir a pantalla de inicio" en iPhone |
| `README.md` | Descripción pública y créditos |

Las imágenes de logo las genera el usuario con otra herramienta de imágenes (prompts) y las entrega ya listas; no se rehacen con código salvo que él lo pida. Los generadores de imagen suelen fallar con símbolos IPA (ɚ, ð, ˌ): verificarlos visualmente.

## Funciones de la página

1. **Spanish → English → IPA:** campo "Spanish" (Translate). Traduce al inglés y luego convierte. Botón Listen lee el español (voz `es-MX`).
   - Traducción principal: modelo local en el navegador, Transformers.js (`@huggingface/transformers@3` desde cdn.jsdelivr.net) con `Xenova/opus-mt-es-en`. Descarga ~115 MB la primera vez; luego queda en caché del navegador.
   - Respaldo: API pública MyMemory (`api.mymemory.translated.net`). OJO: ahí el texto sale a un servicio externo. Trata mayúsculas raro; el código prueba minúsculas primero.
2. **English → IPA:** campo "English" (Convert). Usa el CMU dict, inglés americano. Botón Listen lee el inglés (`en-US`).
3. **Phonetics:** cuadro de texto con el resultado, editable. Botones: Copy (con respaldo `execCommand` y mensaje "Press Ctrl+C"), Wrap in / /, ⌫, Clear, Key sound on/off (se guarda en localStorage).
4. **Teclado IPA:** vocales, diptongos, consonantes y marcas (ˈ ˌ ː . /). Cada tecla muestra una palabra de ejemplo; al tocarla inserta el símbolo y, si Key sound está activo, lee la palabra de ejemplo con Web Speech API. Las marcas no suenan.
5. **Pantalla de carga (splash):** muestra `preview.jpg` grande con barra de progreso mientras precarga el diccionario y el modelo de traducción; botón "Skip" aparece a los 4 s. Existe por una razón funcional (la primera visita descarga ~115 MB), no como adorno: no quitarla ni acortarla sin reemplazar esa precarga. Esto es una excepción deliberada a la regla global de evitar pantallas de carga.
6. Sección "Phonetics" es `position: sticky` arriba para que se vea al bajar por el teclado.

## Conversión ARPAbet → IPA (lógica propia, sin librería)

- Tabla `ARPA` en `index.html`. EH→ɛ, ER→ɝ/ɚ (acentuado/átono), AH→ʌ/ə (acentuado/átono), R→r, IY→i, UW→u, OW→oʊ.
- Palabras de UNA sola sílaba no llevan marca de acento (`hi` → /haɪ/, no /ˈhaɪ/). En 2+ sílabas: acento primario ˈ y secundario ˌ colocados con máximo ataque silábico (set `ONSETS`).
- Palabras que no están en el diccionario se dejan tal cual con `*` y se listan abajo del campo. Ejemplo: "Adonai" no está.
- Limitaciones conocidas: una sola pronunciación por palabra (heterónimos como read/lead), solo inglés americano, la heurística de silabeo puede fallar en palabras largas.
- El teclado ofrece también símbolos británicos (əʊ, ɪə, eə, ʊə, ɒ, ɑː…) aunque el conversor produce americano.

## Diseño (decisiones YA cerradas por el usuario, no revertir)

- Paleta editorial: fondo crema `#f4f0e8`, tinta `#161513`, gris `#7a746a`, líneas `#d8d1c3`, acento verde `#2f6b3a`, tarjetas `#fbf8f2`. Fuente Noto Sans (buen soporte IPA). El título es imagen (`title.jpg`, tipografía Fraunces en el logo).
- Página y textos en INGLÉS (solo el placeholder del campo español va en español).
- En pantallas táctiles NO se llama `focus()` al textarea al tocar símbolos/botones, para que no salga el teclado del iPhone y tape el teclado IPA (variable `touch`).
- Móvil (<700 px): teclas compactas, filas de botones que ocupan el ancho, sección Phonetics compacta.
- El sonido en las teclas fue pedido DESPUÉS (en la sesión de casa); al inicio el usuario había dicho "no quiero sonido". Hoy está activo por defecto y se puede apagar.
- Créditos CMU: una línea muy pequeña y tenue (opacity .55) justo encima del footer, con enlace a la licencia.

### Footer (sello m2aq) — formato aprobado para ESTE proyecto
- Banda negra con borde superior sutil, tipografía mono, 3 columnas: izquierda vacía · centro · derecha.
- Centro: `Original idea by Omar Gámez` │ (línea vertical fina) `Developed by m2aq ●`. Ambos en blanco puro y mismo tamaño (1.1rem). El "2" de m2aq en verde `#4ADE80`. El LED verde (8 px, glow, parpadeo 2 s) va AL FINAL, después de m2aq (no a la izquierda).
- Derecha: `© <año actual>` solo, sin repetir m2aq.
- Sin tagline central y SIN enlace a GitHub ni a ningún perfil del usuario (regla estricta: nunca enlazar su GitHub).
- En móvil el centro se apila (idea arriba, Developed by abajo) y se quita la línea vertical.
- Acento en "Gámez": va siempre con acento aunque el texto esté en inglés.

## Créditos y licencias

- Idea original: Omar Gámez. Desarrollo: m2aq.
- CMU Pronouncing Dictionary © 1993-2015 Carnegie Mellon University, licencia BSD: debe conservarse `LICENSE-cmudict.txt` y la línea de créditos en la página. No mezclar datos de Wiktionary (CC BY-SA) sin pensarlo.
- Modelo `Xenova/opus-mt-es-en` (Helsinki-NLP) cargado por Transformers.js desde CDN: es una dependencia externa en tiempo de ejecución.

## Estado del proyecto

- Probado en iPhone real: funciona todo (audio, cuadro Phonetics fijo, icono en pantalla de inicio y vista previa de WhatsApp).

## Limitaciones conocidas

- El traductor local falla con modismos (por ejemplo "me cae gordo"); para eso haría falta un traductor en la nube.
- Safari puede borrar la caché del modelo (~115 MB) si no se abre el sitio en una semana.

## Ideas pendientes / futuro

- Modo chat donde cada mensaje se muestre con su transcripción IPA (necesitaría backend en tiempo real; hoy el sitio es 100 % estático).
- Mejor manejo de heterónimos por contexto, y palabras fuera del diccionario (modelo grafema→fonema de respaldo).
- Acento británico como opción.
- Atajos con el teclado normal (th → θ) descartados por chocar con escribir inglés; si se retoma, usar prefijo o modo aparte.
- Mantener este archivo al día cuando cambie una decisión.
