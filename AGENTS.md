# Cómo escribir estos apuntes

Apuntes animados de ML en español, hechos a partir de los cuadernos de clase (en catalán, `.ipynb`). Cada cuaderno se convierte en **una página HTML autocontenida**. La referencia de estilo y código es `algebra_lineal.html`: antes de escribir una página nueva, léela entera y copia de ahí lo que necesites.

## Estructura del repo

- `index.html`: portada con una tarjeta por tema.
- `<tema>.html`: una página por cuaderno, nombre en `snake_case` sin tildes (`algebra_lineal.html`).
- Nada más: sin build, sin dependencias, sin carpetas de assets.

## Reglas duras

- **Un solo archivo por página.** CSS en `<style>`, JS en `<script>` al final del `<body>`, datos incrustados como constantes. Tiene que funcionar sin conexión abriendo el archivo con doble clic.
- **Sin librerías ni CDN.** Ni MathJax, ni D3, ni Plotly, ni fuentes externas. Gráficos en SVG generado con JS; fórmulas en HTML/Unicode.
- **Modo claro y oscuro.** Todos los colores de la interfaz salen de las variables CSS de `:root` y su bloque `prefers-color-scheme:dark`. Los colores de datos (`C.blue`, `C.orange`…) son los únicos fijos.
- **Los números tienen que ser los del cuaderno.** Cada resultado que aparezca (MSE, pesos, distancias…) se copia de las salidas del cuaderno o se recalcula y se comprueba que coincide. Nunca se inventa un número "de ejemplo" que parezca real. Si un ejemplo es inventado, se dice.
- **Español**, aunque el cuaderno esté en catalán. Nombres de variables del código se dejan como están (`entrena`, `historial`) para que el alumno los reconozca.

## Plantilla de una página

Copiar de `algebra_lineal.html`, en este orden:

1. `<head>` con el mismo bloque `:root` y el mismo CSS (quitar las clases que no se usen).
2. `<a class="back" href="index.html">← Volver a los apuntes</a>`
3. `<h1>` + `<p class="sub">` que nombra el cuaderno de origen y dice que los gráficos se mueven.
4. `<div class="big">` con **la frase** que el alumno tiene que entender al final.
5. `<nav class="toc">` con una pastilla por sección (`<a href="#s1">1 · Nombre</a>`) y `Resumen`.
6. Las secciones `<h2 id="sN">N · Título</h2>`, siguiendo el orden del cuaderno.
7. `<h2 id="res">` con una `table.sum` (Concepto · Qué es · Numpy · Número para comprobar) y otro `.big` de cierre.
8. Si el cuaderno tiene ejercicios: lista "lo que te tiene que salir" con los resultados esperados.
9. `<footer>` indicando el cuaderno de origen y que funciona sin conexión.

## Cómo se escribe cada sección

Patrón fijo, en este orden:

1. **`.idea`** — una o dos frases con `<b>Idea:</b>`. La intuición sin fórmulas.
2. **`.f`** — la fórmula, centrada, con `<sub>`, `<sup>` y `<span class="sum">` para Σ.
3. **`dl.leg`** — leyenda de **cada** símbolo de la fórmula anterior (`<dt>` símbolo, `<dd>` qué es, en palabras). Si un símbolo no se explica aquí ni en una sección anterior, falta.
4. Párrafo corto que introduce el widget y dice **qué hacer** ("Arrastra…", "Dale a Reproducir…").
5. **`<div class="w" id="wN">`** — el widget animado.
6. **`<pre>`** — el código numpy equivalente, con el resultado en un comentario (`# → 4.003748`).

Tono:

- Frases cortas, tuteo, segunda persona ("mira", "fíjate", "arrastra").
- Primero el porqué en palabras, luego la fórmula, luego el código.
- Analogías concretas (la sombra, la montaña con niebla, las calles de Manhattan).
- Resaltados en línea: `.hg` verde (bien/positivo), `.hr` rojo (mal/negativo), `.hy` amarillo (lo importante ahora), `.hb` azul (secundario).
- Trampas típicas en un `.idea` con `<b>Trampa típica:</b>`.
- Nada de relleno ("En esta sección veremos…"). Al grano.

## Widgets

Cada sección importante lleva al menos un widget. Todos siguen el mismo esqueleto:

```js
/* ================= 3a: descripción corta ================= */
(function(){
  const host=document.getElementById('w3a');
  const msg=H('div',{class:'msg'},host);          // frase que cambia en cada paso
  const P=new Plot(host,{x:[x0,x1],y:[y0,y1],label:'texto accesible'});
  const g=P.g();
  // ... dibujar con P.arrow, P.dot, P.line, P.poly, P.label
  const calc=H('div',{class:'calc'},host);         // la cuenta con números reales
  const sp=stepper(host,N,st=>{ /* dibujar el paso st */ },{interval:1700});
  sp.draw();
})();
```

Piezas que ya existen y se reutilizan (no reescribir):

- `E` / `H`: crear nodos SVG / HTML.
- `f(x, d)`: formatear números (quita `-0` y ceros sobrantes).
- `tween(obj, ms, fn, done)` + `ease`: animaciones con `requestAnimationFrame`.
- `Plot`: ejes con cuadrícula, conversión mundo↔píxel (`X`, `Y`, `P`, `toWorld`), y `arrow`, `label`, `dot`, `line`, `poly` con sus `set*`.
- `stepper(host, n, render, {interval})`: botones Anterior / Reproducir / Siguiente / ↺ y "Paso i de n".
- `compTable`: tabla animada componente a componente (norma, distancia, producto escalar).

Tipos de widget, de más a menos usado:

- **Paso a paso** (`stepper`): explica una cuenta o una construcción. Cada paso cambia `msg` y `calc`.
- **Deslizadores** (`.sl` con `<input type="range">` + `<output>`): para parámetros (pesos, η, matriz 2×2). Redibujar en `oninput`.
- **Arrastrar** (`pointerdown/move/up` + `P.toWorld`): para vectores o puntos que el alumno mueve.
- **Clic / hover** sobre puntos o celdas para ver su detalle.

Reglas de los widgets:

- El estado inicial ya se ve bien sin tocar nada.
- Siempre hay una línea `.calc` con la cuenta real, no solo el dibujo.
- Los cálculos se hacen en JS en la propia página con los datos incrustados (p. ej. `IRIS`), no se precalculan a mano.
- Un `.hint` debajo si la interacción no es obvia.
- Si algo diverge (η grande), se recorta el dibujo; nada de `NaN` en pantalla.

## Datos

- Iris va incrustado como `const IRIS={X:[...],y:[...]}` (copiarlo de `algebra_lineal.html`).
- Otros datos pequeños, igual. Datos aleatorios del cuaderno (`np.random.seed`) no se pueden reproducir en JS: se incrustan los valores o se usan datos inventados avisando de que lo son.

## Portada

Al crear una página, añadir su tarjeta en `index.html` copiando la existente: miniatura SVG de 120×120 que represente el tema, `num` (`02 · N secciones`), título, una frase, y `tags` con los conceptos.

## Antes de dar una página por terminada

- Abrirla en el navegador: sin errores en consola, en claro y en oscuro, y a 375 px de ancho.
- Probar cada widget: Reproducir hasta el final, ↺, y mover los deslizadores a los extremos.
- Comprobar que los números de la página coinciden con las salidas del cuaderno.
- Que el enlace de la portada y el de "Volver" funcionan.
