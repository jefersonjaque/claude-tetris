# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Descripción general

Tetris en JavaScript vanilla con HTML5 Canvas. No hay `package.json`, bundler, transpilador, linter ni tests: no existen comandos de build, lint ni pruebas. El proyecto consta solo de `index.html`, `style.css` y `game.js`. El README y la interfaz están en español, así que mantén ese idioma en textos de UI y documentación.

## Ejecutar el juego

- Abrir `index.html` directamente en el navegador (en Windows: `start index.html`), o
- Servir la carpeta con un servidor estático, por ejemplo `python -m http.server 8000`, y abrir `http://localhost:8000`.

Para verificar cambios hay que probar el juego manualmente en el navegador.

## Arquitectura

Toda la lógica vive en `game.js` (script clásico con `'use strict'`, sin módulos). Estado global mutable declarado en una sola línea (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) que `init()` reinicia por completo; el botón de reinicio simplemente vuelve a llamar a `init()`.

Puntos que requieren leer varias partes para entenderse:

- **Piezas y colores comparten índice.** `PIECES[i]` y `COLORS[i]` usan el mismo índice 1–7, y las celdas del tablero guardan ese índice (0 = vacía). Los valores dentro de cada matriz de `PIECES` ya son el índice de la pieza, no `1`. Al añadir una pieza hay que actualizar ambas tablas y el `Math.random() * 7` de `randomPiece()`.
- **Ciclo de una pieza:** `loop` (por gravedad) o `softDrop`/`hardDrop` (por teclado) → `lockPiece()` → `merge()` + `clearLines()` + `spawn()`. `spawn()` promueve `next` a `current`, genera una nueva `next` y llama a `endGame()` si la pieza recién aparecida ya colisiona.
- **Rotación:** `rotateCW` produce una matriz nueva; `tryRotate` la prueba con desplazamientos horizontales `[0, -1, 1, -2, 2]` (wall kicks simples) y descarta el giro si ninguno es válido.
- **Dificultad:** en `clearLines()` el nivel es `floor(lines / 10) + 1` y `dropInterval = max(100, 1000 − (level − 1) × 90)`. El valor inicial `1000` está duplicado en `init()` y en esa fórmula; si lo cambias, cambia ambos.
- **Puntuación:** `LINE_SCORES[cleared] * level`, más 1 punto por fila en soft drop y 2 por celda en hard drop.
- **Renderizado:** `draw()` repinta todo cada frame (rejilla, tablero, pieza fantasma con alpha 0.2 vía `ghostY()`, pieza actual). La vista previa usa un segundo canvas (`#next-canvas`, 120×120) dibujado solo desde `spawn()` con `drawNext()`.
- **Acoplamiento con el DOM:** `game.js` obtiene elementos por id (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`). Si renombras un id en `index.html`, actualiza `game.js`. Las dimensiones del canvas `#board` (300×600) deben ser `COLS × BLOCK` por `ROWS × BLOCK`; si cambias esas constantes, ajusta `width`/`height` en `index.html`.
- **Pausa y fin de juego** reutilizan el mismo `#overlay` (clase `hidden` para ocultarlo) cambiando el texto de `#overlay-title` y `#overlay-score`.

## Peculiaridades conocidas

- `togglePause()` al reanudar no vuelve a ocultar el overlay (solo `init()` lo hace); un cambio en la pausa debe tenerlo en cuenta.
- `endGame()` se invoca desde dentro de `loop` (vía `lockPiece` → `spawn`), pero `loop` termina llamando a `requestAnimationFrame` de nuevo, por lo que el bucle sigue ejecutándose tras el Game Over.
- La lista de controles del panel en `index.html` no menciona la tecla `X` (rotar), que sí existe en `game.js`.
