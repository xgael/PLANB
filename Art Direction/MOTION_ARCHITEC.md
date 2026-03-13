# Motion Architecture — Masterclass de Animacion para Interfaces Web

> Este documento es un sistema completo de motion design para interfaces digitales. Define la fisica, las reglas, los patrones y las razones detras de cada decision de animacion. Es independiente de framework — los principios aplican a CSS, GSAP, Framer Motion, Webflow o cualquier motor de animacion.

---

## 1. Filosofia: Por Que Existe el Movimiento

El movimiento en una interfaz no es decoracion. Es **informacion temporal** — comunica relaciones de causa-efecto que el layout estatico no puede expresar.

Tres funciones fundamentales del motion:

### 1.1 Orientacion Espacial

El movimiento responde la pregunta: **"De donde viene esto y a donde va?"**

Cuando un modal aparece con `scale(0.96) → scale(1)`, el usuario percibe que "crece" desde el centro de la pantalla. Cuando desaparece con `opacity → 0`, "se disuelve" en su lugar. Sin movimiento, el modal simplemente apareceria y desapareceria — el cerebro no tiene un modelo mental de donde existe ese elemento cuando no esta visible.

### 1.2 Continuidad Narrativa

El movimiento conecta estados discretos en una historia continua.

Un boton que cambia de "Enviar" a "Enviando..." con una transicion de 150ms comunica que **el mismo objeto cambio de estado**. Sin transicion, el usuario podria interpretar que un elemento fue reemplazado por otro. La transicion confirma identidad: es el mismo boton, solo cambio su estado.

### 1.3 Jerarquia de Atencion

El movimiento controla a donde mira el ojo.

El ojo humano es un detector de movimiento. Si mueves un elemento, capturas la atencion involuntariamente. Esto es una herramienta poderosa pero peligrosa:
- **Bien usado**: una notificacion que entra con slide atrae la atencion sin interrumpir
- **Mal usado**: un badge con `animate-ping` perpetuo roba atencion de la tarea principal

**Regla fundamental**: cada animacion debe poder responder a la pregunta **"Que informacion comunica este movimiento que no existe sin el?"** Si la respuesta es "nada, solo se ve bonito", la animacion sobra.

---

## 2. Fisica del Movimiento

### 2.1 Por Que Importa la Fisica

Los humanos viven en un mundo con masa, gravedad y friccion. Cuando un objeto digital se mueve de manera que contradice las leyes fisicas intuitivas, el cerebro lo registra como "falso" o "barato" aunque no pueda articular por que.

Un menu que se abre linealmente (velocidad constante) se siente robotico porque ningun objeto real se mueve asi — todo acelera desde el reposo y desacelera antes de detenerse. Un modal que rebota excesivamente se siente como un juguete porque la interfaz es una herramienta, no un parque de diversiones.

### 2.2 El Modelo de Velocidad

Todo movimiento tiene tres fases:

```
Inicio (aceleracion)  →  Movimiento (velocidad maxima)  →  Fin (desaceleracion)
```

La relacion entre estas tres fases es lo que define el **caracter** de la animacion:

| Perfil | Aceleracion | Desaceleracion | Sensacion | Uso |
|--------|-------------|----------------|-----------|-----|
| **ease-out** | Rapida | Lenta | Natural, llegada suave | Elementos que ENTRAN |
| **ease-in** | Lenta | Rapida | Acumulacion, partida | Elementos que SALEN |
| **ease-in-out** | Lenta | Lenta | Elegante, deliberado | Cambios de estado in-situ |
| **linear** | Constante | Constante | Mecanico, continuo | Progress bars, loaders |

### 2.3 Por Que ease-out para Entradas

Cuando un elemento aparece en pantalla, necesita **establecer su presencia rapidamente** y luego **asentarse**. El ease-out logra esto: arranca rapido (el usuario ve que algo llego) y desacelera al final (se "acomoda" en su posicion).

Si usaras ease-in para una entrada, el elemento empezaria lento — el usuario ya estaria mirando a otro lado cuando finalmente llega. Si usaras ease-in-out, la entrada seria dramatica e innecesariamente ceremoniosa.

### 2.4 Por Que ease-in para Salidas

Cuando un elemento se va, necesita **irse rapido al final** para liberar espacio visual. El ease-in acumula velocidad: empieza con un movimiento sutil ("me estoy yendo") y termina rapido ("ya me fui"). Esto minimiza el tiempo que el elemento ocupa espacio visual sin ser relevante.

### 2.5 Spring Physics (Fisica de Resorte)

Los springs son el gold standard del motion moderno porque reproducen la fisica real mejor que cualquier curva bezier.

Un spring se define por tres parametros:

```
stiffness  → Que tan rapido llega al destino (rigidez del resorte)
damping    → Que tan rapido se estabiliza (friccion)
mass       → Que tan pesado se siente el objeto
```

**Categorias de springs:**

| Nombre | Stiffness | Damping | Caracter | Uso |
|--------|-----------|---------|----------|-----|
| **Snappy** | 400-500 | 30-35 | Rapido, casi sin overshoot | Botones, toggles, tabs |
| **Responsive** | 200-300 | 20-25 | Ligero bounce | Cards, paneles |
| **Gentle** | 100-150 | 15-20 | Suave, bounce visible | Modales, sheets |
| **Bouncy** | 300-400 | 10-15 | Elastico, playful | Onboarding, empty states |

**Equivalencias CSS aproximadas:**

```css
/* Snappy spring → */
cubic-bezier(0.25, 1, 0.5, 1)

/* Responsive spring → */
cubic-bezier(0.16, 1, 0.3, 1)

/* Gentle spring → */
cubic-bezier(0.34, 1.56, 0.64, 1)    /* con overshoot */

/* CSS nativo (Chrome 116+) → */
transition-timing-function: linear(
  0, 0.009, 0.035 2.1%, 0.141, 0.281 6.7%, 0.723 12.9%,
  0.938 16.7%, 1.017, 1.077, 1.121, 1.149 24.3%,
  1.159, 1.163, 1.161, 1.154 29.9%, 1.129 32.8%,
  1.051 39.6%, 1.017 43.1%, 0.991, 0.977 51%,
  0.974 53.8%, 0.975 57.1%, 0.997 69.8%, 1.003 76.9%, 1
);
```

**Regla de springs**: usar springs solo cuando hay un cambio de posicion o escala visible. No usar springs para cambios de color u opacidad — se ven artificiales.

---

## 3. Duraciones — El Sistema Temporal

### 3.1 La Escala de Duracion

La duracion NO es arbitraria. Existe una escala basada en la percepcion humana:

```
Instantaneo       →     0ms        No se percibe cambio
Imperceptible     →  50-80ms       Se siente "instantaneo" pero con suavidad
Rapido            → 100-150ms      Feedback de interaccion (hover, click)
Estandar          → 200-300ms      Transiciones de layout, entradas de elementos
Deliberado        → 400-500ms      Cambios importantes, page transitions
Dramatico         → 600-1000ms     Secuencias de carga, reveals cinematicos
```

### 3.2 Regla de Seleccion de Duracion

La duracion correcta depende de **dos factores**:

**Factor 1: Distancia visual**
Cuanto mas lejos se mueve un elemento, mas tiempo necesita. Un tooltip que aparece en su lugar necesita 100ms. Un panel que se desliza 300px necesita 300ms. Un page transition full-screen necesita 500ms.

Formula aproximada:
```
duracion (ms) = distancia (px) × 0.8 + 100
```

Esto no es exacto — es una guia. Siempre ajustar por sensacion.

**Factor 2: Frecuencia de uso**
Los elementos que el usuario activa frecuentemente necesitan duraciones mas cortas. Si un dropdown se abre 50 veces al dia, 100ms. Si un modal se abre 3 veces al dia, 250ms. Si un onboarding ocurre una vez, 600ms.

```
Frecuencia alta   → 80-150ms     (hover, toggles, tabs, tooltips)
Frecuencia media  → 150-300ms    (dropdowns, modales, paneles)
Frecuencia baja   → 300-600ms    (page transitions, onboarding, reveals)
Frecuencia unica  → 600-1200ms   (splash screens, landing page hero)
```

### 3.3 Duraciones Especificas por Componente

| Componente | Entrada | Salida | Razon |
|-----------|---------|--------|-------|
| **Tooltip** | 100ms | 80ms | Alta frecuencia, no debe estorbar |
| **Dropdown** | 150ms | 100ms | Frecuente, feedback inmediato |
| **Toast** | 250ms | 200ms | Debe captar atencion sin asustar |
| **Modal** | 250ms | 200ms | Cambio contextual importante |
| **Sheet/Panel** | 300ms | 250ms | Movimiento largo (slide) |
| **Page transition** | 400-500ms | 300ms | Cambio de contexto completo |
| **Load sequence** | 600-1200ms total | — | Primera impresion, se tolera mas lento |

**Regla de asimetria**: la entrada siempre es ligeramente mas lenta que la salida. El usuario necesita tiempo para percibir que algo aparecio, pero quiere que lo que se va se vaya rapido. Ratio tipico: entrada/salida = 1.2:1

---

## 4. Easing Library — Catalogo Completo de Curvas

### 4.1 Las 7 Curvas Esenciales

Cada curva tiene un nombre, un proposito y una restriccion de uso. No se mezclan arbitrariamente.

```css
/* 1. Standard — Interacciones basicas */
--ease-standard: cubic-bezier(0.4, 0, 0.2, 1);
/* Descripcion: Versatil, derivada de Material Design.
   Uso: hover, color changes, opacity.
   NO usar para: entradas/salidas de elementos. */

/* 2. Decelerate (Ease-out) — Entradas */
--ease-decelerate: cubic-bezier(0.0, 0, 0.2, 1);
/* Descripcion: Arranca rapido, frena suave.
   Uso: elementos que aparecen en pantalla.
   NO usar para: salidas o loops. */

/* 3. Accelerate (Ease-in) — Salidas */
--ease-accelerate: cubic-bezier(0.4, 0, 1, 1);
/* Descripcion: Arranca lento, se acelera.
   Uso: elementos que desaparecen.
   NO usar para: entradas o hover. */

/* 4. Sharp — Microinteracciones */
--ease-sharp: cubic-bezier(0.4, 0, 0.6, 1);
/* Descripcion: Rapida, casi lineal con suavizado.
   Uso: toggles, checkboxes, micro-feedback.
   NO usar para: transiciones largas (>200ms). */

/* 5. Expressive — Movimientos prominentes */
--ease-expressive: cubic-bezier(0.16, 1, 0.3, 1);
/* Descripcion: Entrada explosiva, asentamiento suave.
   Uso: modales, sheets, paneles grandes.
   NO usar para: elementos pequenos o frecuentes. */

/* 6. Spring-like — Rebote controlado */
--ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
/* Descripcion: Overshoot sutil, como un resorte amortiguado.
   Uso: elementos que "aterrizan" (scale, position).
   NO usar para: opacidad o color (el bounce no tiene sentido visual). */

/* 7. Linear — Progreso constante */
--ease-linear: linear;
/* Descripcion: Velocidad constante.
   Uso: UNICAMENTE progress bars, spinners, marquees.
   NO usar para: ningun otro tipo de transicion. */
```

### 4.2 Matriz de Seleccion

Para elegir la curva correcta, cruzar tipo de propiedad con tipo de accion:

| Propiedad | Entrada | Salida | Estado | Loop |
|-----------|---------|--------|--------|------|
| **opacity** | decelerate | accelerate | standard | — |
| **transform (position)** | expressive | accelerate | spring | — |
| **transform (scale)** | spring | accelerate | sharp | — |
| **background-color** | standard | standard | standard | — |
| **border-color** | standard | standard | standard | — |
| **width/height** | expressive | accelerate | standard | — |
| **progress** | — | — | — | linear |

### 4.3 Regla de Consistencia

Dentro de un mismo producto, cada tipo de componente usa SIEMPRE la misma curva. Si los modales usan `--ease-expressive` para entrar, TODOS los modales lo hacen. No mezclar curvas dentro de la misma categoria de componente.

---

## 5. Tipos de Animacion

Toda animacion web cae en una de cinco categorias. Cada una tiene reglas distintas.

### 5.1 Load Animations (Animaciones de Carga)

**Que son**: la primera impresion visual cuando la pagina o seccion carga. Son las animaciones que el usuario ve una sola vez por sesion.

**Anatomia de una secuencia de carga:**

```
Fase 1: Preloader       → Logo o indicador de que la pagina esta cargando
Fase 2: Page Reveal      → El preloader se retira, el contenido aparece
Fase 3: Hero Entrance    → El elemento principal entra (titulo, imagen)
Fase 4: Stagger Cascade  → Elementos secundarios entran en secuencia
Fase 5: Idle             → Todo esta en su lugar, la pagina es interactiva
```

**Reglas de load animations:**

1. **La secuencia completa no debe exceder 2.5 segundos** desde que el DOM esta listo. Mas de eso y el usuario percibe lentitud, no calidad.

2. **El contenido critico aparece primero**. La jerarquia de aparicion sigue la jerarquia de importancia:
   ```
   Navegacion → Hero/Titulo → Contenido principal → Elementos secundarios → Footer
   ```

3. **Stagger maximo: 5-7 elementos**. Si necesitas stagger en 15 cards, agrupa en filas de 3-4 y stagger las filas, no las cards individuales. El ojo no puede rastrear mas de 5-7 items en secuencia.

4. **El stagger delay entre items es 50-80ms**, nunca 200ms+. Delays largos entre items crean la sensacion de "cascada de dominó" que es entertaing la primera vez e insoportable la quinta.

**Implementacion CSS de load sequence:**

```css
/* Sistema de stagger con custom properties */
[data-load] {
  opacity: 0;
  transform: translateY(12px);
}

[data-load="visible"] {
  opacity: 1;
  transform: translateY(0);
  transition:
    opacity 400ms var(--ease-decelerate),
    transform 400ms var(--ease-expressive);
}

/* Stagger por indice */
[data-load-index="0"] { transition-delay: 0ms; }
[data-load-index="1"] { transition-delay: 60ms; }
[data-load-index="2"] { transition-delay: 120ms; }
[data-load-index="3"] { transition-delay: 180ms; }
[data-load-index="4"] { transition-delay: 240ms; }
```

**Implementacion JS de load sequence:**

```js
// Orquestador de carga
function orchestrateLoad() {
  const phases = [
    { selector: '[data-load="nav"]',     delay: 0 },
    { selector: '[data-load="hero"]',    delay: 100 },
    { selector: '[data-load="content"]', delay: 300 },
    { selector: '[data-load="cards"]',   delay: 500 },
  ];

  phases.forEach(({ selector, delay }) => {
    const elements = document.querySelectorAll(selector);
    elements.forEach((el, i) => {
      setTimeout(() => {
        el.setAttribute('data-load', 'visible');
      }, delay + (i * 60)); // 60ms stagger entre items del grupo
    });
  });
}

// Ejecutar cuando el DOM + fonts estan listos
document.fonts.ready.then(orchestrateLoad);
```

### 5.2 Scroll Animations (Animaciones de Scroll)

**Que son**: animaciones activadas por la posicion de scroll del usuario. El scroll se convierte en una linea de tiempo que el usuario controla.

**Dos paradigmas opuestos:**

**A) Scroll-Triggered (Activado por scroll)**
El scroll actua como un TRIGGER — dispara una animacion que corre de forma independiente.

```
Usuario scrollea → Elemento entra al viewport → Animacion se ejecuta (una vez)
```

Ejemplo: una card que entra con fade+slide cuando es visible.

```js
// IntersectionObserver — el metodo nativo y performante
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('animate-in');
      observer.unobserve(entry.target); // Solo una vez
    }
  });
}, {
  threshold: 0.2,        // 20% del elemento visible
  rootMargin: '0px 0px -50px 0px'  // Trigger 50px antes del borde inferior
});

document.querySelectorAll('[data-scroll-reveal]')
  .forEach(el => observer.observe(el));
```

```css
[data-scroll-reveal] {
  opacity: 0;
  transform: translateY(20px);
  transition:
    opacity 500ms var(--ease-decelerate),
    transform 500ms var(--ease-expressive);
}

[data-scroll-reveal].animate-in {
  opacity: 1;
  transform: translateY(0);
}
```

**B) Scroll-Linked (Vinculado al scroll)**
El scroll ES la linea de tiempo — la animacion avanza y retrocede segun la posicion del scroll.

```
Usuario scrollea abajo → Animacion progresa
Usuario scrollea arriba → Animacion retrocede
```

Ejemplo: parallax, progress bars de lectura, transformaciones continuas.

```css
/* CSS Scroll-Driven Animations (nativo, sin JS) */
@keyframes fade-slide {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.scroll-linked-element {
  animation: fade-slide linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}

/* Parallax con scroll-timeline */
.parallax-layer {
  animation: parallax-shift linear both;
  animation-timeline: scroll();
}

@keyframes parallax-shift {
  from { transform: translateY(0); }
  to   { transform: translateY(-100px); }
}
```

**Reglas de scroll animations:**

1. **Scroll-triggered para contenido, scroll-linked para decoracion.** Las cards de contenido usan triggered (se ven una vez y quedan). Los fondos de parallax usan linked (son ambientales).

2. **Offset de 20-30% del viewport**. El elemento empieza a animarse cuando el 20-30% ya es visible, no cuando el pixel superior toca el borde inferior. Esto evita que el usuario vea "elementos apareciendo desde la nada" — cuando nota el elemento, ya esta parcialmente visible.

3. **Solo un tipo de reveal por pagina**. Si las cards entran con fade-up, TODAS las cards entran con fade-up. No mezclar fade-left, fade-right, fade-up, scale-in en la misma pagina. La consistencia es mas importante que la variedad.

4. **Parallax maximo: 2 capas**. Foreground y background. Tres o mas capas de parallax crean nausea en usuarios sensibles y problemas de performance serios.

5. **Nunca secuestrar el scroll (scroll hijacking)**. El scroll natural del navegador debe funcionar siempre. Las animaciones vinculadas al scroll ACOMPANAN al scroll, no lo reemplazan. Scroll snapping agresivo, velocidades de scroll personalizadas, o secciones que "consumen" scroll sin mover la pagina son hostiles para el usuario.

### 5.3 Interaction Animations (Micro-interacciones)

**Que son**: feedback inmediato a una accion del usuario — hover, click, focus, drag.

**Principio: Estimulo → Respuesta inmediata**

El tiempo entre la accion del usuario y el feedback visual no debe exceder **100ms**. Mas de eso y el cerebro desconecta la causa del efecto.

**Catalogo de micro-interacciones:**

```css
/* Hover — cambio de estado visual */
.interactive-element {
  transition: background-color 150ms var(--ease-standard),
              color 150ms var(--ease-standard),
              border-color 150ms var(--ease-standard);
}

/* Press/Active — respuesta tactil */
.pressable:active {
  transform: scale(0.97);
  transition: transform 80ms var(--ease-sharp);
}
.pressable:not(:active) {
  transition: transform 250ms var(--ease-spring);
}

/* Focus — accesibilidad + orientacion */
.focusable:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
  transition: outline-offset 100ms var(--ease-standard);
}

/* Toggle — cambio de estado binario */
.toggle-thumb {
  transition: transform 200ms var(--ease-spring);
}
.toggle-thumb[data-state="on"] {
  transform: translateX(16px);
}

/* Expand/Collapse — revelacion de contenido */
.collapsible-content {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 250ms var(--ease-expressive);
}
.collapsible-content[data-open="true"] {
  grid-template-rows: 1fr;
}
.collapsible-content > div {
  overflow: hidden;
}
```

**Reglas de micro-interacciones:**

1. **Hover no es obligatorio**. En mobile no existe hover. Si el hover comunica informacion critica (como "esto es clickeable"), esa informacion debe existir sin hover — a traves de color, cursor, o affordance visual.

2. **Active/Press SIEMPRE acompana a hover**. Si un elemento tiene hover, tambien debe tener un estado :active diferente. Sin el, el click se siente "muerto".

3. **Solo animar propiedades compositas**. Las propiedades que el navegador anima en la GPU (transform, opacity) son gratis. Las que requieren layout (width, height, padding, margin) o paint (background-color, box-shadow, border) son costosas. Preferir siempre:
   - `transform: scale()` sobre `width/height`
   - `transform: translate()` sobre `top/left/margin`
   - `opacity` sobre `visibility/display`

4. **Nunca animar en hover si el hover se activa frecuentemente**. Si un nav tiene 8 items y el usuario pasa el mouse por todos, 8 animaciones simultaneas son ruido visual. En ese caso, la transicion debe ser instantanea (50-80ms) no cinematica (300ms+).

### 5.4 State Transitions (Transiciones de Estado)

**Que son**: cambios de un estado funcional a otro — loading → success, collapsed → expanded, default → error.

**Los estados de un componente forman una maquina de estados:**

```
             ┌──────────┐
    entrada  │  DEFAULT │  hover/focus
    ────────→│          │◄──────────┐
             └────┬─────┘           │
                  │ click           │ blur
                  ▼                 │
             ┌──────────┐          │
             │ LOADING  │──────────┘ (si cancel)
             └────┬─────┘
                  │ response
          ┌───────┴───────┐
          ▼               ▼
     ┌─────────┐    ┌─────────┐
     │ SUCCESS │    │  ERROR  │
     └────┬────┘    └────┬────┘
          │ 2-3s         │ dismiss
          ▼              ▼
     ┌──────────┐   ┌──────────┐
     │  RESET   │   │  DEFAULT │
     └──────────┘   └──────────┘
```

**Reglas de state transitions:**

1. **Cada transicion de estado DEBE tener movimiento**. El cambio abrupto de un boton de "Enviar" a "Cargando..." sin transicion rompe la percepcion de identidad del objeto.

2. **Los estados de error necesitan mas atencion que los de exito**. Error: borde que cambia de color + shake sutil (2-3px, 300ms). Exito: checkmark fade-in (200ms). El error requiere accion; el exito no.

3. **Los loading states deben ser predecibles**. Un spinner es mejor que una barra de progreso falsa. Si no sabes cuanto tardara, no finja progreso.

```css
/* Transicion de estado en boton */
.button {
  transition:
    background-color 150ms var(--ease-standard),
    color 150ms var(--ease-standard),
    transform 200ms var(--ease-spring);
}

/* Loading: desactivar interaccion, indicar proceso */
.button[data-state="loading"] {
  pointer-events: none;
  opacity: 0.7;
}

/* Success: feedback positivo breve */
.button[data-state="success"] {
  background-color: var(--success);
  color: white;
}

/* Error shake */
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%      { transform: translateX(-3px); }
  40%      { transform: translateX(3px); }
  60%      { transform: translateX(-2px); }
  80%      { transform: translateX(2px); }
}

.input[data-state="error"] {
  border-color: var(--error);
  animation: shake 300ms var(--ease-sharp);
}
```

### 5.5 Page Transitions (Transiciones de Pagina)

**Que son**: el movimiento entre vistas o rutas completas. Son las transiciones de mayor escala.

**Patron basico: Exit → Enter**

```
Pagina actual sale  (300ms)  →  Nueva pagina entra  (400ms)
                               └── con stagger si tiene multiples secciones
```

**La View Transitions API (nativa):**

```css
/* Definir que elementos son "el mismo" entre paginas */
.page-header {
  view-transition-name: header;
}

.page-hero-image {
  view-transition-name: hero;
}

/* Controlar la animacion de transicion */
::view-transition-old(header) {
  animation: fade-out 250ms var(--ease-accelerate);
}

::view-transition-new(header) {
  animation: fade-in 350ms var(--ease-decelerate);
}

/* El hero image hace morphing entre posiciones */
::view-transition-old(hero),
::view-transition-new(hero) {
  animation-duration: 400ms;
  animation-timing-function: var(--ease-expressive);
}
```

```js
// Trigger programatico
document.startViewTransition(() => {
  // Actualizar el DOM aqui
  updatePageContent(newContent);
});
```

**Reglas de page transitions:**

1. **La nueva pagina NUNCA empieza vacia**. El header/nav debe ser persistente o tener un placeholder inmediato. Una pantalla en blanco, aunque sea por 100ms, mata la percepcion de velocidad.

2. **Cross-fade como default seguro**. Si no tienes tiempo para disenar transiciones elaboradas, un crossfade de 300ms entre paginas es mejor que nada. No es glamoroso pero es funcional.

3. **Shared element transitions solo para max 2 elementos**. Si intentas animar 5 elementos simultaneamente entre paginas, el resultado es caos visual. Animar solo el elemento focal (imagen, titulo) y dejar que el resto haga fade.

4. **La transicion de salida es mas simple que la de entrada**. Salida: fade-out global (200-300ms). Entrada: stagger de secciones (400-600ms total). El usuario no necesita "despedirse" de la pagina anterior.

---

## 6. Orchestracion — El Arte de la Secuencia

### 6.1 Que es Orchestracion

Es la disciplina de coordinar multiples animaciones para que se perciban como un movimiento unico y coherente. No es "varias animaciones corriendo al mismo tiempo" — es un sistema coreografico.

### 6.2 Stagger (Cascada)

El patron mas comun de orchestracion. Los elementos de un grupo aparecen secuencialmente con un delay fijo entre cada uno.

**Formula del stagger optimo:**

```
delay_entre_items = duracion_individual / (n_items × 0.8)

Ejemplo: 5 cards con animacion de 400ms
delay = 400 / (5 × 0.8) = 100ms → redondeamos a 80ms

Card 1: delay 0ms
Card 2: delay 80ms
Card 3: delay 160ms
Card 4: delay 240ms
Card 5: delay 320ms

Total de la secuencia: 320ms + 400ms = 720ms
```

**Restriccion critica**: el total de la secuencia (ultimo delay + duracion de la animacion) no debe exceder **1200ms** para contenido funcional. Si el stagger produce una secuencia de 3 segundos, hay demasiados items — agrupa en bloques.

### 6.3 Direccionalidad del Stagger

El stagger tiene una direccion que comunica significado:

| Direccion | Patron | Cuando usar |
|-----------|--------|-------------|
| **Arriba→Abajo** | Items aparecen de arriba hacia abajo | Listas verticales, feeds. El mas comun y natural |
| **Izquierda→Derecha** | Items aparecen de izquierda a derecha | Grids horizontales, tabs, navigation |
| **Centro→Afuera** | El centro aparece primero, luego los lados | Heroes, elementos focales, expansion radial |
| **Afuera→Centro** | Los lados aparecen primero, luego el centro | Contenedores que "se llenan" |

**Regla**: la direccion del stagger debe coincidir con la direccion de lectura o la logica del layout. Un grid de cards se stagger izquierda→derecha, fila por fila (como se lee). Un dropdown se stagger arriba→abajo (como se navega).

### 6.4 Layered Orchestration (Por Capas)

Las secuencias complejas se organizan en capas jerarquicas:

```
Capa 1 — Estructura:    Sidebar, Header, contenedor principal (0ms)
Capa 2 — Contenido:     Titulo, subtitulo, CTA (200ms)
Capa 3 — Datos:         Cards, charts, tablas (400ms)
Capa 4 — Decoracion:    Badges, indicadores, metadata (600ms)
```

Cada capa tiene su propio stagger interno. Las capas se ejecutan secuencialmente.

**Regla de capas**: maximo 4 capas. Mas de 4 y la secuencia se siente interminable. Si necesitas mas capas, es que tienes demasiada informacion en la vista.

### 6.5 Overlap (Superposicion)

Los buenos secuenciadores no esperan a que una animacion termine para empezar la siguiente. Las animaciones se solapan:

```
Titulo:    |████████████|
Subtitulo:      |████████████|          ← empieza al 40% del titulo
Cards:               |████████████|     ← empieza al 40% del subtitulo
```

**Formula de overlap**: el siguiente item empieza cuando el anterior ha completado el 30-50% de su animacion. Esto crea flujo sin esperas.

```css
/* Overlap con delays calculados */
.title    { transition-delay: 0ms; }      /* duracion: 400ms */
.subtitle { transition-delay: 160ms; }    /* 40% de 400ms */
.cards    { transition-delay: 320ms; }    /* 160 + 40% de 400ms */
```

---

## 7. Propiedades Animables — Que Mover y Que No

### 7.1 Clasificacion por Rendimiento

```
TIER 1 — Compositing only (GPU, 0 layout cost):
  ✓ transform (translate, scale, rotate)
  ✓ opacity
  ✓ filter (blur, brightness — con moderacion)
  ✓ clip-path (con precaucion)

TIER 2 — Paint only (repinta, no recalcula layout):
  △ background-color
  △ color
  △ border-color
  △ box-shadow
  △ outline

TIER 3 — Layout (recalcula posiciones de otros elementos):
  ✗ width, height
  ✗ padding, margin
  ✗ top, left, right, bottom
  ✗ font-size
  ✗ border-width
  ✗ gap
```

**Regla estricta**: NUNCA animar propiedades de Tier 3 directamente. Siempre hay una alternativa de Tier 1:

| En vez de animar... | Animar... |
|---------------------|-----------|
| `width` | `transform: scaleX()` |
| `height` | `transform: scaleY()` o `grid-template-rows` |
| `top/left` | `transform: translate()` |
| `padding` | `transform: scale()` con `transform-origin` |
| `font-size` | `transform: scale()` (si es decorativo) |
| `gap` | no animar — cambiar en step |

### 7.2 Combinaciones de Propiedades

Las animaciones convincentes combinan 2-3 propiedades. Mas de 3 es ruido.

**Combinaciones efectivas:**

```css
/* Entrada standard: posicion + opacidad */
.enter {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Aparicion con escala: escala + opacidad */
.pop-in {
  from { opacity: 0; transform: scale(0.95); }
  to   { opacity: 1; transform: scale(1); }
}

/* Slide con presencia: posicion + opacidad + escala sutil */
.slide-enter {
  from { opacity: 0; transform: translateX(-20px) scale(0.98); }
  to   { opacity: 1; transform: translateX(0) scale(1); }
}
```

**Combinaciones prohibidas:**

```css
/* NUNCA: blur + scale + translate + opacity + color */
/* Es un circo visual. El ojo no sabe que rastrear. */

/* NUNCA: box-shadow animada con color */
/* Performance destructiva, ademas parece un juego de Flash. */

/* NUNCA: backdrop-filter animada (cambiar blur en runtime) */
/* Fuerza re-render del filtro cada frame. El GPU se ahoga. */
```

### 7.3 Regla del Transform Compuesto

Cuando combinas transforms, el orden importa y la relacion entre ellos debe tener sentido fisico:

```css
/* CORRECTO: translate primero, luego scale */
transform: translateY(12px) scale(0.96);
/* El objeto esta "abajo y un poco encogido" — fisicamente coherente */

/* INCORRECTO: rotate + scale + translate sin logica */
transform: rotate(5deg) scale(1.1) translateX(20px);
/* El ojo no sabe que esta pasando — tres movimientos inconexos */
```

---

## 8. Patrones Recurrentes — Recetario

### 8.1 Reveal (Aparicion)

El patron mas comun. Un elemento pasa de invisible a visible.

**Variantes:**

```css
/* Fade Up — el default universal */
@keyframes fade-up {
  from { opacity: 0; transform: translateY(16px); }
  to   { opacity: 1; transform: translateY(0); }
}
/* Usar en: cards, secciones, contenido general */
/* Duracion: 400-500ms | Easing: decelerate/expressive */

/* Fade In (sin movimiento) — contenido que "aparece" */
@keyframes fade-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}
/* Usar en: imagenes, overlays, transiciones de estado */
/* Duracion: 200-300ms | Easing: decelerate */

/* Scale In — elementos que "nacen" */
@keyframes scale-in {
  from { opacity: 0; transform: scale(0.9); }
  to   { opacity: 1; transform: scale(1); }
}
/* Usar en: modales, tooltips, popovers, command palette */
/* Duracion: 200-250ms | Easing: expressive/spring */

/* Slide In — elementos que vienen de un borde */
@keyframes slide-in-right {
  from { opacity: 0; transform: translateX(100%); }
  to   { opacity: 1; transform: translateX(0); }
}
/* Usar en: panels laterales, sheets, drawers */
/* Duracion: 300-400ms | Easing: expressive */
```

**Regla critica**: el offset de translate NUNCA debe ser mayor al 5% del viewport o 30px (lo que sea menor). Offsets de 100px+ crean la sensacion de "elementos volando desde fuera de pantalla" que se siente amateur. La excepcion es slide-in-right/left para paneles que realmente vienen del borde.

### 8.2 Dismiss (Salida)

La version invertida del reveal, pero NO simetrica.

```css
/* Fade Down — salida standard */
@keyframes fade-down {
  from { opacity: 1; transform: translateY(0); }
  to   { opacity: 0; transform: translateY(8px); }
}
/* Offset de salida es MENOR que el de entrada (8px vs 16px)
   porque la salida debe ser rapida y discreta */
/* Duracion: 200-300ms | Easing: accelerate */

/* Scale Out — elementos que "se disuelven" */
@keyframes scale-out {
  from { opacity: 1; transform: scale(1); }
  to   { opacity: 0; transform: scale(0.95); }
}
/* Duracion: 150-200ms | Easing: accelerate */
```

**Regla**: las salidas son 20-30% mas rapidas que las entradas y usan offsets mas pequenos. El usuario no necesita "ver" como se va un elemento — necesita que desaparezca rapido para que el nuevo contenido tome su lugar.

### 8.3 Morph (Transformacion)

Un elemento cambia de forma o posicion manteniendo su identidad.

```css
/* Layout morph con FLIP technique */
.morphing-element {
  transition: transform 350ms var(--ease-expressive);
}
```

```js
// FLIP: First, Last, Invert, Play
function flip(element, callback) {
  // FIRST: capturar posicion actual
  const first = element.getBoundingClientRect();

  // Ejecutar el cambio de DOM
  callback();

  // LAST: capturar posicion nueva
  const last = element.getBoundingClientRect();

  // INVERT: calcular el delta y posicionar donde estaba
  const deltaX = first.left - last.left;
  const deltaY = first.top - last.top;
  const deltaW = first.width / last.width;
  const deltaH = first.height / last.height;

  element.style.transform = `translate(${deltaX}px, ${deltaY}px) scale(${deltaW}, ${deltaH})`;
  element.style.transformOrigin = 'top left';

  // PLAY: animar hacia la posicion nueva
  requestAnimationFrame(() => {
    element.style.transition = 'transform 350ms cubic-bezier(0.16, 1, 0.3, 1)';
    element.style.transform = 'none';

    element.addEventListener('transitionend', () => {
      element.style.transition = '';
      element.style.transformOrigin = '';
    }, { once: true });
  });
}
```

**Cuando usar morph:**
- Cards que se expanden a detalle (grid → full view)
- Elementos de lista que se reordenan
- Imagenes que transicionan entre vistas (thumbnail → full)

**Cuando NO usar morph:**
- Entre elementos que no comparten identidad visual
- Cuando hay mas de 2 elementos morphing simultaneamente
- Sobre textos largos (el scale distorsiona la tipografia)

### 8.4 Pulse / Attention (Captura de Atencion)

Animaciones que dicen "mira aqui" sin intervencion del usuario.

```css
/* Pulse sutil — notificacion nueva, cambio de dato */
@keyframes subtle-pulse {
  0%   { opacity: 1; }
  50%  { opacity: 0.5; }
  100% { opacity: 1; }
}
/* Duracion: 1 ciclo de 600ms, NO loop infinito */
/* Usar: cuando un numero cambia, cuando hay nuevo contenido */

/* Number change — el valor crece momentaneamente */
@keyframes number-bump {
  0%   { transform: scale(1); }
  40%  { transform: scale(1.05); }
  100% { transform: scale(1); }
}
/* Duracion: 400ms con ease-spring */
/* Usar: cuando un KPI se actualiza en tiempo real */
```

**Regla absoluta**: NUNCA usar animaciones en loop infinito para captar atencion. Un elemento pulsando eternamente se convierte en ruido visual que el usuario aprende a ignorar y que le causa fatiga. Si necesitas atencion persistente, usa color o posicion estatica, no movimiento.

La unica excepcion son indicadores de proceso activo (loading spinners, progress bars).

---

## 9. Scroll Storytelling — Narrar con el Scroll

### 9.1 El Scroll como Timeline

El scroll es la unica herramienta de interaccion que TODO usuario entiende. No requiere aprendizaje. Por eso es el vehiculo perfecto para narrativa digital.

El truco es que el scroll no es "pasar de largo" — es **revelar**. Cada pixel de scroll debe sentirse como avanzar una historia.

### 9.2 Ritmo de la Pagina

Una pagina bien animada tiene ritmo, como una pieza musical:

```
INTRO (Hero)          → Forte     → Impacto visual fuerte, animacion grande
DESARROLLO (Content)  → Piano     → Reveals suaves, ritmo constante
CLIMAX (CTA/Feature)  → Forte     → Animacion destacada, parallax o reveal especial
CODA (Footer)         → Pianissimo → Minimo movimiento, cierre limpio
```

**No todas las secciones merecen animacion.** Si cada seccion tiene un reveal elaborado, ninguno destaca. La regla es:

```
Seccion hero:               Animacion compleja (load sequence, parallax)
Secciones de contenido:     Reveal sutil (fade-up al scrollear)
Seccion de conversion/CTA:  Animacion destacada (escala, parallax sutil, reveal especial)
Footer:                     Sin animacion de scroll
```

### 9.3 Velocidad de Revelacion

El ritmo al que los elementos aparecen cuando el usuario scrollea debe ser proporcional a la velocidad de lectura:

**Scroll lento** (lectura): los reveals deben activarse antes de que el usuario los busque — el usuario lee, los elementos ya estan visibles.

**Scroll rapido** (exploracion): los reveals deben ser rapidos y no bloquear — el usuario escanea, no quiere esperar 800ms por cada seccion.

```css
/* Solucion: duraciones cortas + triggers tempranos */
[data-scroll-reveal] {
  opacity: 0;
  transform: translateY(16px);
  transition:
    opacity 350ms var(--ease-decelerate),
    transform 350ms var(--ease-expressive);
}

/* Trigger cuando el 15% del elemento es visible
   (en vez del tipico 50%) para anticiparse al scroll */
```

### 9.4 Parallax — Cuando Si y Cuando No

El parallax crea profundidad haciendo que las capas se muevan a diferentes velocidades. Es cinematico pero peligroso.

**Cuando usar parallax:**
- Landing pages donde la historia visual es el producto
- Imagenes de hero con texto superpuesto
- Fondos decorativos que refuerzan atmosfera

**Cuando NO usar parallax:**
- Dashboards o aplicaciones de datos (distrae de la funcion)
- Sobre textos largos (dificulta la lectura)
- En mobile (performance y usabilidad)
- Cuando ya hay scroll-triggered reveals (sobrecarga)

**Parallax correcto:**

```css
/* Factor de parallax: 0.1-0.3 del scroll normal */
/* Mas de 0.3 y se siente como un videojuego */

.parallax-bg {
  animation: parallax linear both;
  animation-timeline: scroll();
}

@keyframes parallax {
  from { transform: translateY(0); }
  to   { transform: translateY(-60px); }   /* sutil, max 60-100px */
}
```

**Parallax incorrecto:**

```css
/* NUNCA: parallax de 500px que mueve el fondo drasticamente */
/* NUNCA: parallax en el texto principal */
/* NUNCA: 3+ capas de parallax */
/* NUNCA: parallax combinado con scroll-snapping */
```

---

## 10. Performance — Las Reglas Irrompibles

### 10.1 El Presupuesto de Frame

El navegador renderiza a 60fps (16.6ms por frame). Cada animacion que corre debe completar su trabajo en esos 16.6ms. Si no, el usuario percibe "jank" (tartamudeo).

```
Presupuesto por frame:
  Layout:           ~0-2ms (si solo animas compositing)
  Paint:            ~0-4ms (si animas color/shadow)
  Composite:        ~1-2ms (GPU handles this)
  JavaScript:       ~2-4ms (event handlers, RAF)
  ────────────────
  TOTAL disponible: ~16.6ms
  Margen seguro:    ~10ms (dejar headroom para el navegador)
```

### 10.2 Reglas de Performance

1. **will-change con moderacion**. Solo en elementos que REALMENTE van a animarse. `will-change: transform` reserva una capa de GPU por elemento — 20 elementos con will-change son 20 capas extra.

```css
/* CORRECTO: aplicar antes de animar, remover despues */
.about-to-animate {
  will-change: transform, opacity;
}
.done-animating {
  will-change: auto;
}

/* INCORRECTO: aplicar globalmente */
* { will-change: transform; }  /* GPU se queda sin memoria */
```

2. **IntersectionObserver para scroll animations, no scroll event**. El evento `scroll` se dispara 60+ veces por segundo. IntersectionObserver usa el hilo del compositor y no bloquea el main thread.

3. **`content-visibility: auto`** para secciones fuera del viewport. Esto le dice al navegador que no renderice lo que no se ve:

```css
.section-below-fold {
  content-visibility: auto;
  contain-intrinsic-size: auto 500px;
}
```

4. **backdrop-filter es el enemigo silencioso**. Cada superficie con `backdrop-filter: blur()` forza al GPU a samplear y procesar los pixeles detras. Limite estricto: maximo 3-4 superficies con backdrop-filter visibles al mismo tiempo.

5. **Detener animaciones fuera del viewport**. Si un elemento con animacion CSS en loop sale del viewport, deberia pausarse:

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    entry.target.style.animationPlayState =
      entry.isIntersecting ? 'running' : 'paused';
  });
});
```

6. **Batch DOM reads y writes**. Leer y escribir el DOM alternadamente causa "layout thrashing" — cada lectura fuerza un recalculo si hubo una escritura previa.

```js
// MAL — causa layout thrashing
elements.forEach(el => {
  const height = el.offsetHeight;    // READ
  el.style.height = height + 10;     // WRITE
  // El siguiente READ fuerza recalculo por el WRITE anterior
});

// BIEN — batch reads, luego batch writes
const heights = elements.map(el => el.offsetHeight);  // ALL READS
elements.forEach((el, i) => {
  el.style.transform = `translateY(${heights[i]}px)`;  // ALL WRITES
});
```

### 10.3 Performance por Dispositivo

```
Desktop moderno:     Sin restricciones (dentro de las reglas)
Laptop con bateria:  Reducir parallax, simplificar staggers
Tablet:              Eliminar parallax, reducir blur a 12px
Mobile:              Eliminar parallax, reducir backdrop-filter,
                     duraciones -20%, solo fade/opacity reveals
Low-end mobile:      Solo opacity transitions, sin transforms complejos
```

```css
/* Deteccion de capacidad */
@media (prefers-reduced-motion: no-preference) and (min-width: 768px) {
  /* Full animations */
}

@media (prefers-reduced-motion: no-preference) and (max-width: 767px) {
  /* Simplified animations */
  [data-scroll-reveal] {
    transform: none;  /* Solo opacity, sin translate */
  }
}
```

---

## 11. Accesibilidad — Motion Inclusivo

### 11.1 prefers-reduced-motion

No es opcional. Es un requisito legal en muchas jurisdicciones y un requisito etico en todas.

```css
@media (prefers-reduced-motion: reduce) {
  /* Eliminar TODA animacion decorativa */
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }

  /* Mantener cambios de estado instantaneos (necesarios para usabilidad) */
  /* El cambio de color en hover, el focus ring, etc. siguen funcionando
     porque son "instantaneos" con 0.01ms */
}
```

**Por que 0.01ms y no 0ms?**
Porque `0ms` puede romper scripts que escuchan `transitionend` o `animationend` — esos eventos nunca se disparan con duracion 0. Con `0.01ms`, los eventos SI se disparan pero el movimiento es imperceptible.

### 11.2 Que Conservar con Reduced Motion

No todo se elimina. Los cambios de estado funcional DEBEN seguir siendo visibles, solo sin movimiento:

- Hover: el cambio de color ocurre, pero sin transicion (instantaneo)
- Focus: el ring aparece sin animacion
- Toggle: el switch cambia de posicion sin slide (instantaneo)
- Modal: aparece sin animacion de entrada, pero SI aparece
- Toast: aparece sin slide, pero SI aparece

### 11.3 Vestibular Disorders (Trastornos Vestibulares)

Algunas animaciones causan nausea o mareo en personas con trastornos vestibulares:

**Triggers peligrosos:**
- Parallax (movimiento diferencial entre capas)
- Scale que afecta grandes areas de la pantalla
- Rotaciones
- Movimientos que cubren mas del 25% del viewport
- Auto-play videos con movimiento de camara

**Regla**: cualquier animacion que cubra mas del 25% del area visible debe respetar `prefers-reduced-motion`.

---

## 12. Combinaciones — Cuando Usar Que

### 12.1 Matriz de Decision por Tipo de Sitio

| Tipo de sitio | Load sequence | Scroll reveals | Parallax | Micro-interactions | Page transitions |
|---------------|:---:|:---:|:---:|:---:|:---:|
| **Landing/Marketing** | Completa | Si, prominentes | Si (1-2 capas) | Hover + Press | Opcional |
| **Portfolio/Agencia** | Teatral | Si, sutiles | Opcional | Hover elegante | Si, shared elements |
| **SaaS/Dashboard** | Rapida o ninguna | No | No | Si, funcionales | Fade simple |
| **E-commerce** | Minima | Sutiles | No | Hover en productos | No (velocidad > estilo) |
| **Blog/Editorial** | No | Fade-in suave | No | Links + share | No |
| **Documentacion** | No | No | No | Code copy, nav | No |

### 12.2 Combinaciones Seguras vs Peligrosas

**Combinaciones que funcionan:**

```
Load sequence + Scroll reveals
  → La load anima el above-the-fold, los scroll reveals animan el below
  → NO superponer: si un elemento tuvo load animation, NO tiene scroll reveal

Scroll reveals + Micro-interactions
  → Los reveals establecen presencia, las micro-interactions dan feedback
  → Son complementarios porque operan en momentos distintos

Page transitions + Load sequence
  → La transicion REEMPLAZA la load sequence en navegacion interna
  → La load sequence es solo para el primer page load
```

**Combinaciones peligrosas:**

```
Parallax + Scroll reveals
  → Demasiado movimiento simultaneo. El ojo no sabe a donde mirar.
  → Si necesitas ambos: parallax SOLO en el hero, reveals en el resto

Load sequence completa + Page transitions elaboradas
  → La pagina se siente lenta porque hay dos capas de "esperar"
  → Elegir uno: o tienes un load espectacular, o transiciones espectaculares

Scroll hijacking + Cualquier otra animacion
  → El scroll hijacking ya es desorientador. Anadirle reveals o parallax
     lo hace nauseabundo. Nunca combinar.
```

### 12.3 La Curva de "Demasiado"

```
  Percepcion de calidad
  ▲
  │           ╭──────╮
  │          ╱        ╲
  │         ╱          ╲
  │        ╱            ╲
  │       ╱              ╲
  │      ╱                ╲ ← aqui empiezan las quejas de rendimiento
  │     ╱                  ╲
  │    ╱                    ╲
  │───╱──────────────────────╲───→ Cantidad de animacion
      0   pocas  justas  muchas  exceso

El sweet spot: suficiente animacion para que la interfaz
se sienta viva, pero NO tanta como para que se sienta lenta
o distraiga de la tarea.
```

**Regla del 80/20**: el 80% de la percepcion de calidad viene del 20% de las animaciones. Esas son: entrada del hero, transiciones de estado (loading/success/error), y hover feedback. Si SOLO implementas esas tres, ya tienes el 80% del impacto.

---

## 13. Anti-patrones — Lo Que No Hacer (y Por Que)

### 13.1 animate-ping (Pulso Infinito)

```css
/* EL PEOR PATRON en UI funcional */
@keyframes ping {
  75%, 100% { transform: scale(2); opacity: 0; }
}
.animate-ping { animation: ping 1s cubic-bezier(0, 0, 0.2, 1) infinite; }
```

**Por que es malo:** crea un movimiento perpetuo que captura la atencion involuntariamente. El ojo humano es incapaz de ignorar movimiento periferico. Un badge con ping en el sidebar compite con el contenido principal por la atencion del usuario, cada segundo, para siempre. Es el equivalente visual de alguien gritando "HEY" infinitamente en una biblioteca.

**Alternativa:** un dot estatico de color. Si necesitas indicar "nuevo", cambia el color a --success o anade un counter. La informacion se comunica sin movimiento.

### 13.2 Glow Hover (Box-shadow Animado con Color)

```css
/* Destructivo para performance y estetica */
.card:hover { box-shadow: 0 0 30px rgba(118, 79, 249, 0.5); }
```

**Por que es malo:** las sombras con color saturado y blur grande son costosas de renderizar (cada frame recalcula el blur) y crean un look "gamer RGB" que contradice cualquier intento de sofisticacion. Ademas, el glow compite con el contenido de la card por atencion.

**Alternativa:** `background-color` cambia a un tono ligeramente mas claro. Sutil, performante, elegante.

### 13.3 Scroll Hijacking

```js
// Secuestrar el scroll natural
window.addEventListener('scroll', (e) => {
  e.preventDefault();
  // Scroll personalizado...
});
```

**Por que es malo:** rompe el modelo mental del usuario. El scroll es el control mas fundamental de la web — cuando no funciona como se espera, el usuario siente que perdio el control. Ademas, rompe accesibilidad (screen readers, keyboard navigation) y es hostile en mobile (donde el momentum scroll es parte de la UX del OS).

**Alternativa:** CSS `scroll-snap` para secciones con ancla, que mantiene el scroll natural pero "asiste" al usuario.

### 13.4 Blur Animado (backdrop-filter variable)

```css
/* Nunca animar el valor de blur */
@keyframes focus-effect {
  from { backdrop-filter: blur(0px); }
  to   { backdrop-filter: blur(16px); }
}
```

**Por que es malo:** cada frame de esa animacion fuerza al GPU a re-samplear y re-procesar toda el area detras del elemento. En 60fps, eso son 60 renders del filtro por segundo. En dispositivos no premium, esto causa lag visible y puede drenar bateria.

**Alternativa:** animar la opacidad de un pseudo-elemento que YA tiene blur aplicado estaticamente:

```css
.overlay::after {
  backdrop-filter: blur(16px);
  opacity: 0;
  transition: opacity 200ms var(--ease-decelerate);
}
.overlay.active::after {
  opacity: 1;
}
```

### 13.5 Delays Excesivos

```css
.card:nth-child(10) { animation-delay: 2000ms; }
```

**Por que es malo:** el usuario llego a la pagina hace 2 segundos y todavia hay elementos apareciendo. Esto se siente como un sitio lento, no como un sitio animado. La diferencia entre "animacion elaborada" y "lentitud" es puramente temporal — si pasa de 1.2 segundos, es lento.

### 13.6 Animaciones que Bloquean Interaccion

```css
.modal { animation: elaborate-entrance 800ms; pointer-events: none; }
.modal.animate-done { pointer-events: auto; }
```

**Por que es malo:** el usuario quiere interactuar pero el boton no responde porque la animacion no ha terminado. Esto es frustrante y crea la percepcion de que la interfaz esta rota.

**Regla**: los elementos deben ser interactivos DESDE QUE APARECEN, aunque la animacion no haya terminado. La animacion es visual, la interactividad es funcional — nunca subordinar la funcion a la forma.

### 13.7 Reveal Inconsistente

```css
/* Card 1 entra desde la izquierda */
.card-1 { animation: slide-left; }
/* Card 2 entra desde abajo */
.card-2 { animation: slide-up; }
/* Card 3 entra con scale */
.card-3 { animation: scale-in; }
```

**Por que es malo:** tres animaciones diferentes para tres cards del mismo tipo crean confusion. El cerebro busca patrones — si cada card se comporta diferente, no puede anticipar que hara la siguiente. Esto genera carga cognitiva innecesaria.

**Regla**: elementos del mismo tipo SIEMPRE usan la misma animacion. Cards = fade-up. Modales = scale-in. Toasts = slide-right. Sin excepciones.

---

## 14. Implementacion — Patrones por Tecnologia

### 14.1 CSS Puro (Vanilla)

Para proyectos simples o donde se quiere cero dependencias:

```css
/* Sistema de animacion con data attributes */

/* === Variables globales === */
:root {
  --ease-decelerate:  cubic-bezier(0.0, 0, 0.2, 1);
  --ease-accelerate:  cubic-bezier(0.4, 0, 1, 1);
  --ease-standard:    cubic-bezier(0.4, 0, 0.2, 1);
  --ease-expressive:  cubic-bezier(0.16, 1, 0.3, 1);
  --ease-spring:      cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-sharp:       cubic-bezier(0.4, 0, 0.6, 1);

  --duration-instant:  80ms;
  --duration-fast:     150ms;
  --duration-normal:   250ms;
  --duration-slow:     400ms;
  --duration-slower:   600ms;
}

/* === Reveal system === */
[data-reveal] {
  opacity: 0;
  transform: translateY(16px);
  transition:
    opacity var(--duration-slow) var(--ease-decelerate),
    transform var(--duration-slow) var(--ease-expressive);
}
[data-reveal].is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* === Stagger (CSS custom property approach) === */
[data-reveal][data-stagger] {
  transition-delay: calc(var(--stagger-index, 0) * 60ms);
}

/* === Interaction feedback === */
.interactive {
  transition:
    background-color var(--duration-fast) var(--ease-standard),
    color var(--duration-fast) var(--ease-standard),
    border-color var(--duration-fast) var(--ease-standard);
}

/* === Reduced motion === */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

```js
// Controlador de reveals con IntersectionObserver
class RevealController {
  constructor() {
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      { threshold: 0.15, rootMargin: '0px 0px -40px 0px' }
    );

    document.querySelectorAll('[data-reveal]').forEach((el, i) => {
      el.style.setProperty('--stagger-index', i);
      this.observer.observe(el);
    });
  }

  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-visible');
        this.observer.unobserve(entry.target);
      }
    });
  }
}

// Inicializar cuando fonts y DOM estan listos
document.fonts.ready.then(() => new RevealController());
```

### 14.2 GSAP (Proyectos Complejos)

Para secuencias cinematicas, scroll-linking avanzado, o cuando CSS no alcanza:

```js
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

// Load sequence orquestada
function heroLoadSequence() {
  const tl = gsap.timeline({
    defaults: {
      ease: 'power3.out',      // equivale a --ease-expressive
      duration: 0.6,
    }
  });

  tl.from('[data-load="nav"]', {
    opacity: 0,
    y: -20,
    duration: 0.4,
  })
  .from('[data-load="hero-title"]', {
    opacity: 0,
    y: 30,
  }, '-=0.2')                     // overlap de 200ms
  .from('[data-load="hero-subtitle"]', {
    opacity: 0,
    y: 20,
    duration: 0.4,
  }, '-=0.3')
  .from('[data-load="hero-cta"]', {
    opacity: 0,
    scale: 0.95,
    duration: 0.3,
    ease: 'back.out(1.7)',        // spring-like
  }, '-=0.2');

  return tl;
}

// Scroll-triggered reveals
function setupScrollReveals() {
  gsap.utils.toArray('[data-scroll-reveal]').forEach(el => {
    gsap.from(el, {
      opacity: 0,
      y: 16,
      duration: 0.5,
      ease: 'power2.out',
      scrollTrigger: {
        trigger: el,
        start: 'top 85%',       // trigger cuando el 15% es visible
        once: true,
      }
    });
  });
}

// Scroll-linked parallax
function setupParallax() {
  gsap.utils.toArray('[data-parallax]').forEach(el => {
    const speed = parseFloat(el.dataset.parallax) || 0.2;

    gsap.to(el, {
      y: () => -100 * speed,
      ease: 'none',
      scrollTrigger: {
        trigger: el,
        start: 'top bottom',
        end: 'bottom top',
        scrub: true,
      }
    });
  });
}
```

### 14.3 Framer Motion / React (SPAs)

Para aplicaciones React con transiciones de estado ricas:

```jsx
import { motion, AnimatePresence } from 'framer-motion';

// Variantes reutilizables — centralizadas como el easing system
const variants = {
  fadeUp: {
    initial:  { opacity: 0, y: 16 },
    animate:  { opacity: 1, y: 0 },
    exit:     { opacity: 0, y: 8 },
  },
  scaleIn: {
    initial:  { opacity: 0, scale: 0.95 },
    animate:  { opacity: 1, scale: 1 },
    exit:     { opacity: 0, scale: 0.95 },
  },
  slideRight: {
    initial:  { opacity: 0, x: -20 },
    animate:  { opacity: 1, x: 0 },
    exit:     { opacity: 0, x: 20 },
  },
};

// Transiciones por tipo de componente
const transitions = {
  snappy:     { type: 'spring', stiffness: 500, damping: 30 },
  responsive: { type: 'spring', stiffness: 300, damping: 25 },
  gentle:     { type: 'spring', stiffness: 150, damping: 18 },
  fast:       { duration: 0.15, ease: [0.4, 0, 0.2, 1] },
  normal:     { duration: 0.25, ease: [0.16, 1, 0.3, 1] },
};

// Modal con animacion
function Modal({ isOpen, children }) {
  return (
    <AnimatePresence>
      {isOpen && (
        <>
          <motion.div
            className="overlay"
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            transition={{ duration: 0.2 }}
          />
          <motion.div
            className="modal"
            variants={variants.scaleIn}
            initial="initial"
            animate="animate"
            exit="exit"
            transition={transitions.gentle}
          >
            {children}
          </motion.div>
        </>
      )}
    </AnimatePresence>
  );
}

// Stagger list
function StaggerList({ items }) {
  return (
    <motion.ul
      initial="initial"
      animate="animate"
      variants={{
        animate: { transition: { staggerChildren: 0.06 } }
      }}
    >
      {items.map(item => (
        <motion.li
          key={item.id}
          variants={variants.fadeUp}
          transition={transitions.normal}
        >
          {item.content}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

### 14.4 Webflow Interactions

Para proyectos Webflow sin codigo:

```
Estructura de interactions en Webflow:

1. TRIGGERS
   ├── Page Load          → Load sequences
   ├── Scroll Into View   → Section reveals
   ├── Mouse Hover        → Card interactions
   ├── Mouse Click        → State changes
   └── Scroll (continuous)→ Parallax, progress

2. ACTIONS (por trigger)
   ├── Move (transform: translate)
   ├── Scale (transform: scale)
   ├── Rotate (transform: rotate)
   ├── Opacity (opacity)
   ├── Size (width/height — usar con precaucion)
   └── Background Color

3. TIMING
   ├── Duration: seguir la escala de este documento
   ├── Easing: Webflow ofrece ease, ease-in-out, ease-in, ease-out + custom bezier
   ├── Stagger: usar "Affect siblings" + delay entre items
   └── Sequence: "After previous" para overlap controlado

REGLAS WEBFLOW:
- Usar "Limit to once" en scroll-triggered reveals
- NO activar "Smoothing" en scroll-linked animations
  (introduce lag perceptible)
- Cada interaction class SOLO afecta una propiedad por accion
- Nombramiento: IX-[trigger]-[accion]-[componente]
  Ejemplo: IX-scroll-fadeup-card
```

---

## 15. Testing y Validacion

### 15.1 Checklist por Animacion

Antes de marcar una animacion como "lista", verificar:

```
□ Responde a prefers-reduced-motion
□ No excede 300ms en componentes de alta frecuencia
□ No excede 1200ms en secuencias de stagger
□ Solo anima propiedades de Tier 1 (transform, opacity)
□ No bloquea la interactividad del usuario
□ Se ve bien a 60fps en el dispositivo target mas lento
□ Tiene un proposito claro (orientacion, continuidad o jerarquia)
□ Es consistente con otros elementos del mismo tipo
□ No causa nausea (no cubre >25% del viewport en movimiento)
□ Funciona sin JavaScript (progressive enhancement)
```

### 15.2 Herramientas de Validacion

```
Chrome DevTools:
  → Performance tab: grabar 5s de scroll, verificar que no hay frames
    rojos (>16.6ms)
  → Animations tab: inspeccionar timing y easing de cada animacion
  → Rendering tab: activar "Paint flashing" para ver que se repinta
  → CSS Overview: verificar que no hay transitions en propiedades de layout

Firefox DevTools:
  → Animation Inspector: timeline visual de todas las animaciones activas
  → Accessibility Inspector: verificar comportamiento con reduced-motion

Emulacion:
  → Chrome > Rendering > Emulate prefers-reduced-motion: reduce
  → Throttle CPU a 4x slowdown para simular mobile
  → Throttle network a "Slow 3G" para verificar load sequence
```

---

## 16. Glosario

| Termino | Definicion |
|---------|-----------|
| **Easing** | La curva que define la aceleracion/desaceleracion de una animacion |
| **Stagger** | Delay incremental entre items de un grupo que animan secuencialmente |
| **Orchestration** | Coordinacion de multiples animaciones en una secuencia coherente |
| **FLIP** | First-Last-Invert-Play: tecnica para animar cambios de layout eficientemente |
| **Jank** | Tartamudeo visual causado por frames que exceden 16.6ms |
| **Compositing** | Capa de renderizado que el GPU maneja sin afectar layout o paint |
| **Spring** | Modelo fisico de animacion basado en resorte (stiffness, damping, mass) |
| **Scrub** | Vincular el progreso de una animacion a una variable externa (ej: scroll position) |
| **Parallax** | Efecto de profundidad donde capas se mueven a diferentes velocidades |
| **Reveal** | Animacion de entrada de un elemento que pasa de oculto a visible |
| **Morph** | Transicion donde un elemento cambia de forma/posicion manteniendo identidad |
| **Overshoot** | Cuando una animacion sobrepasa su valor final antes de regresar (bounce) |
| **Affordance** | Indicador visual de que un elemento es interactivo |
| **Viewport** | El area visible de la pagina en la pantalla del usuario |
| **Threshold** | El porcentaje de un elemento que debe ser visible para activar un trigger |

---

## 17. Mantra

> "El mejor motion design es invisible. El usuario no deberia notar las animaciones — deberia notar que la interfaz se siente natural, responsiva y viva. Si alguien dice 'que bonitas animaciones', probablemente son demasiadas. Si alguien dice 'esta interfaz se siente increible', las animaciones estan haciendo su trabajo."

La animacion no es un feature. Es oxigeno — necesario, vital, pero invisible cuando esta bien. Solo se nota cuando falta o cuando sobra.
