# Glass Architecture: AI Society Pro — Liquid Glass Implementation Guide

> Guia de arquitectura para implementar el lenguaje de diseno Liquid Glass (iOS 26) en AI Society Pro. Este documento traduce los principios de Apple a CSS/Tailwind para una aplicacion web dark-theme. Complementa a `UI_ARCHITEC.md` — los tokens base de color, tipografia y espaciado siguen vigentes.

---

## 1. Principio Fundamental

**Liquid Glass se aplica UNICAMENTE a la capa de navegacion que flota sobre el contenido.**

```
SI  → Tab bars, nav bars, sidebars, toolbars
SI  → Sheets, modales, command palette
SI  → Context menus, dropdowns, popovers
SI  → Cards de widget, acciones flotantes
NO  → Listas de datos, tablas, contenido de lectura
NO  → Inputs dentro de formularios
NO  → Alertas criticas o acciones destructivas
NO  → Glass sobre glass (nunca apilar dos superficies glass)
```

La razon: el glass crea jerarquia entre controles y contenido. Si el contenido tambien es glass, la jerarquia colapsa y todo se vuelve ilegible.

---

## 2. El Material

### 2.1 Que es Liquid Glass

Una superficie translucida que:
- **Refracta** el contenido detras (distorsion sutil)
- **Refleja** luz especular que responde al movimiento
- **Se adapta** cromaticamente al fondo (muestrea wallpaper/contenido circundante)
- **Responde** a la interaccion con micro-animaciones

### 2.2 Variantes del Material

| Variante | Transparencia | Uso | Blur |
|----------|--------------|-----|------|
| **Regular** | Media (frosted) | Navegacion, toolbars, cards | Alto (~12-16px) |
| **Clear** | Alta (cristalino) | Sobre media/imagenes, overlays | Bajo (~4-6px) |
| **Solid** (fallback) | Ninguna | Reduce Transparency, datos densos | Ninguno |

### 2.3 Regla de Tinting

El tint NO es decorativo. Se usa SOLO para comunicar significado semantico:
- Accion primaria → tint sutil del color de marca
- Estado de exito → tint verde
- Alerta → tint ambar

Si el tint no comunica nada, no se usa. El glass es neutro por defecto.

---

## 3. Design Tokens — Glass Layer

Estos tokens extienden los de `UI_ARCHITEC.md`. Solo aplican a elementos de la capa glass.

### 3.1 Fondos Glass (Dark Theme)

```css
/* Regular Glass — navegacion, sidebar, cards elevadas */
--glass-regular-bg:       rgba(17, 17, 19, 0.72);       /* #111113 al 72% */
--glass-regular-blur:     16px;
--glass-regular-saturate: 180%;

/* Clear Glass — sobre media, overlays sutiles */
--glass-clear-bg:         rgba(9, 9, 11, 0.40);          /* #09090B al 40% */
--glass-clear-blur:       6px;
--glass-clear-saturate:   150%;

/* Elevated Glass — modales, command palette, sheets */
--glass-elevated-bg:      rgba(17, 17, 19, 0.85);        /* #111113 al 85% */
--glass-elevated-blur:    24px;
--glass-elevated-saturate: 180%;
```

### 3.2 Bordes Glass

```css
--glass-border:           rgba(255, 255, 255, 0.12);     /* #FFFFFF1F — mas visible que solid */
--glass-border-subtle:    rgba(255, 255, 255, 0.08);     /* #FFFFFF14 — separadores internos */
--glass-border-strong:    rgba(255, 255, 255, 0.18);     /* #FFFFFF2E — hover, focus */
```

### 3.3 Sombras Glass

```css
/* Sombra exterior — profundidad flotante */
--glass-shadow:           0 8px 32px rgba(0, 0, 0, 0.35);

/* Sombra elevada — modales, sheets */
--glass-shadow-elevated:  0 25px 50px rgba(0, 0, 0, 0.50);

/* Highlight interior — brillo sutil en borde superior */
--glass-highlight:        inset 0 1px 0 rgba(255, 255, 255, 0.06);
```

### 3.4 Specular Highlight

```css
/* Pseudo-elemento ::after para el brillo especular */
--glass-specular-bg:      rgba(255, 255, 255, 0.04);
--glass-specular-opacity: 0.5;
--glass-specular-blur:    1px;
```

---

## 4. CSS Base — Clases Reutilizables

### 4.1 Glass Regular

```css
.glass {
  background: var(--glass-regular-bg);
  backdrop-filter: blur(var(--glass-regular-blur)) saturate(var(--glass-regular-saturate));
  -webkit-backdrop-filter: blur(var(--glass-regular-blur)) saturate(var(--glass-regular-saturate));
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  box-shadow:
    var(--glass-shadow),
    var(--glass-highlight);
  position: relative;
}

/* Specular highlight layer */
.glass::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: inherit;
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.06) 0%,
    transparent 40%
  );
  pointer-events: none;
}
```

### 4.2 Glass Clear

```css
.glass-clear {
  background: var(--glass-clear-bg);
  backdrop-filter: blur(var(--glass-clear-blur)) saturate(var(--glass-clear-saturate));
  -webkit-backdrop-filter: blur(var(--glass-clear-blur)) saturate(var(--glass-clear-saturate));
  border: 1px solid var(--glass-border-subtle);
  border-radius: 16px;
  box-shadow: var(--glass-shadow);
  position: relative;
}
```

### 4.3 Glass Elevated

```css
.glass-elevated {
  background: var(--glass-elevated-bg);
  backdrop-filter: blur(var(--glass-elevated-blur)) saturate(var(--glass-elevated-saturate));
  -webkit-backdrop-filter: blur(var(--glass-elevated-blur)) saturate(var(--glass-elevated-saturate));
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  box-shadow:
    var(--glass-shadow-elevated),
    var(--glass-highlight);
  position: relative;
}
```

### 4.4 Glass Tinted

```css
/* Tint semantico — se anade como modificador */
.glass-tint-brand  { background: rgba(118, 79, 249, 0.08); }  /* #764FF9 */
.glass-tint-success { background: rgba(34, 197, 94, 0.08); }   /* #22C55E */
.glass-tint-warning { background: rgba(234, 179, 8, 0.08); }   /* #EAB308 */
.glass-tint-error   { background: rgba(239, 68, 68, 0.08); }   /* #EF4444 */

/* El tint se aplica SOBRE el glass base, no lo reemplaza.
   Ejemplo: class="glass glass-tint-brand" */
```

---

## 5. Tailwind CSS Implementation

### 5.1 Tailwind Config — Tokens Personalizados

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        glass: {
          regular: 'rgba(17, 17, 19, 0.72)',
          clear: 'rgba(9, 9, 11, 0.40)',
          elevated: 'rgba(17, 17, 19, 0.85)',
          border: 'rgba(255, 255, 255, 0.12)',
          'border-subtle': 'rgba(255, 255, 255, 0.08)',
          'border-strong': 'rgba(255, 255, 255, 0.18)',
          specular: 'rgba(255, 255, 255, 0.04)',
        }
      },
      backdropBlur: {
        glass: '16px',
        'glass-clear': '6px',
        'glass-elevated': '24px',
      },
      backdropSaturate: {
        glass: '1.8',
      },
      boxShadow: {
        glass: '0 8px 32px rgba(0, 0, 0, 0.35), inset 0 1px 0 rgba(255, 255, 255, 0.06)',
        'glass-elevated': '0 25px 50px rgba(0, 0, 0, 0.50), inset 0 1px 0 rgba(255, 255, 255, 0.06)',
      },
      borderRadius: {
        glass: '16px',
        'glass-sm': '12px',
        'glass-pill': '9999px',
      }
    }
  }
}
```

### 5.2 Clases Tailwind por Componente

```html
<!-- Glass Regular -->
<div class="bg-glass-regular backdrop-blur-glass backdrop-saturate-glass
            border border-glass-border rounded-glass shadow-glass relative">

<!-- Glass Clear -->
<div class="bg-glass-clear backdrop-blur-glass-clear
            border border-glass-border-subtle rounded-glass relative">

<!-- Glass Elevated (modales) -->
<div class="bg-glass-elevated backdrop-blur-glass-elevated backdrop-saturate-glass
            border border-glass-border rounded-glass shadow-glass-elevated relative">
```

### 5.3 Plugin Custom para Specular Highlight

```js
// Tailwind plugin
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function({ addUtilities }) {
  addUtilities({
    '.glass-shine': {
      '&::after': {
        content: '""',
        position: 'absolute',
        inset: '0',
        borderRadius: 'inherit',
        background: 'linear-gradient(180deg, rgba(255,255,255,0.06) 0%, transparent 40%)',
        pointerEvents: 'none',
      }
    }
  })
})
```

---

## 6. Componentes — Especificacion Glass

### 6.1 Sidebar

```css
.sidebar-glass {
  width: 220px;
  background: var(--glass-regular-bg);
  backdrop-filter: blur(20px) saturate(180%);
  border-right: 1px solid var(--glass-border-subtle);
  /* Sin border-radius — pegado al borde */
}
```

**Comportamiento**: la sidebar refracta el contenido principal detras, dando percepcion de profundidad. El nav item activo usa un fondo glass sutil adicional:

```css
.nav-item-active {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 8px;
}
```

Los nav items inactivos son transparentes. Sin fondo glass.

### 6.2 Header / Navigation Bar

```css
.navbar-glass {
  height: 48px;
  background: var(--glass-regular-bg);
  backdrop-filter: blur(16px) saturate(180%);
  border-bottom: 1px solid var(--glass-border-subtle);
  /* position: sticky; top: 0; z-index: 40; */
}
```

**Regla iOS 26**: en scroll, el navbar puede volverse mas transparente o compactarse. En nuestra implementacion web:

```css
/* Al hacer scroll, reducir opacidad del fondo */
.navbar-glass.scrolled {
  background: rgba(17, 17, 19, 0.55);
}
```

### 6.3 Tab Bar

```css
.tabbar-glass {
  background: var(--glass-regular-bg);
  backdrop-filter: blur(16px) saturate(180%);
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  padding: 4px;
  box-shadow: var(--glass-shadow);
}

.tab-item {
  padding: 8px 16px;
  border-radius: 12px;    /* concentrico con el contenedor de 16px */
  color: var(--text-secondary);
  font-size: 13px;
  transition: background-color 150ms ease, color 150ms ease;
}

.tab-item.active {
  background: rgba(255, 255, 255, 0.12);
  color: var(--text-primary);
}
```

**Concentric corners**: el radio del tab item (12px) es menor que el del contenedor (16px), creando alineacion concentrica con 4px de padding visual.

### 6.4 Cards Elevadas

```css
.card-glass {
  background: var(--glass-regular-bg);
  backdrop-filter: blur(12px) saturate(180%);
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  padding: 24px;
  box-shadow: var(--glass-shadow);
}
```

**Cuando usar glass en cards**:
- Cards de resumen/widget que flotan sobre un fondo con contenido (imagenes, gradientes)
- Cards de accion rapida o acceso directo
- Cards de notificacion/toast

**Cuando NO usar glass en cards**:
- Cards de datos (stat cards con numeros) → usar solido `#111113`
- Cards dentro de listas densas → usar solido
- Cards que contienen tablas o formularios → usar solido

### 6.5 Modales / Sheets

```css
/* Overlay con dimming — SIN blur en el overlay */
.modal-overlay {
  background: rgba(0, 0, 0, 0.60);
  /* El dimming es necesario para .clear glass */
}

/* Modal container */
.modal-glass {
  background: var(--glass-elevated-bg);
  backdrop-filter: blur(24px) saturate(180%);
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  box-shadow: var(--glass-shadow-elevated);
  max-width: 560px;
}

/* Separadores internos */
.modal-glass .modal-header {
  padding: 20px 24px;
  border-bottom: 1px solid var(--glass-border-subtle);
}

.modal-glass .modal-footer {
  padding: 16px 24px;
  border-top: 1px solid var(--glass-border-subtle);
}
```

### 6.6 Command Palette (cmd+K)

```css
.command-palette {
  background: var(--glass-elevated-bg);
  backdrop-filter: blur(24px) saturate(180%);
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  width: 560px;
  max-height: 400px;
  box-shadow: var(--glass-shadow-elevated);
  overflow: hidden;
}

.command-palette .search-input {
  padding: 16px 20px;
  background: transparent;
  border-bottom: 1px solid var(--glass-border-subtle);
  font-size: 15px;
  color: var(--text-primary);
}

.command-palette .result-item {
  padding: 10px 20px;
  transition: background-color 150ms ease;
}

.command-palette .result-item:hover,
.command-palette .result-item.selected {
  background: rgba(255, 255, 255, 0.08);
}
```

### 6.7 Context Menu / Dropdown

```css
.dropdown-glass {
  background: var(--glass-elevated-bg);
  backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid var(--glass-border);
  border-radius: 12px;
  padding: 4px;
  box-shadow: var(--glass-shadow-elevated);
  min-width: 200px;
}

.dropdown-item {
  padding: 8px 12px;
  border-radius: 8px;     /* concentrico: 8px item dentro de 12px container */
  font-size: 13px;
  color: var(--text-secondary);
  gap: 8px;
  transition: background-color 150ms ease;
}

.dropdown-item:hover {
  background: rgba(255, 255, 255, 0.08);
  color: var(--text-primary);
}

.dropdown-separator {
  height: 1px;
  background: var(--glass-border-subtle);
  margin: 4px 0;
}
```

### 6.8 Tooltip

```css
.tooltip-glass {
  background: var(--glass-regular-bg);
  backdrop-filter: blur(12px) saturate(180%);
  border: 1px solid var(--glass-border-subtle);
  border-radius: 8px;
  padding: 6px 10px;
  font-size: 12px;
  color: var(--text-primary);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.30);
}
```

### 6.9 Toast / Notificacion

```css
.toast-glass {
  background: var(--glass-regular-bg);
  backdrop-filter: blur(16px) saturate(180%);
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  padding: 12px 16px;
  box-shadow: var(--glass-shadow);
  min-width: 320px;
}

/* Indicador de estado — barra lateral */
.toast-glass::before {
  content: '';
  position: absolute;
  left: 0;
  top: 12px;
  bottom: 12px;
  width: 3px;
  border-radius: 2px;
  background: var(--toast-color); /* success/warning/error */
}
```

### 6.10 Botones Glass

```css
/* Boton glass primario — para acciones en contextos glass */
.btn-glass {
  background: rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 12px;
  padding: 8px 16px;
  color: var(--text-primary);
  font-size: 13px;
  font-weight: 500;
  transition: background-color 150ms ease;
}

.btn-glass:hover {
  background: rgba(255, 255, 255, 0.18);
}

.btn-glass:active {
  background: rgba(255, 255, 255, 0.24);
  transform: scale(0.98);
  transition: transform 100ms ease;
}

/* Boton glass pill — tab bar items, segmented controls */
.btn-glass-pill {
  background: transparent;
  border: none;
  border-radius: 9999px;
  padding: 6px 14px;
  color: var(--text-secondary);
  font-size: 13px;
  transition: all 150ms ease;
}

.btn-glass-pill.active {
  background: rgba(255, 255, 255, 0.12);
  color: var(--text-primary);
}
```

---

## 7. Concentric Corners — Regla de Radios

Apple introduce el concepto de **esquinas concentricas**: los elementos hijos tienen un radio menor que su contenedor, creando alineacion optica perfecta.

```
Formula: radio_hijo = radio_padre - padding

Contenedor: border-radius 16px, padding 4px
  → Hijo:   border-radius 12px

Contenedor: border-radius 12px, padding 4px
  → Hijo:   border-radius 8px

Contenedor: border-radius 16px, padding 8px
  → Hijo:   border-radius 8px
```

### Tabla de Radios Concentricos

| Componente | Radio | Contexto |
|-----------|-------|----------|
| Modal / Sheet | 16px | Contenedor principal |
| Card glass | 16px | Contenedor de contenido |
| Tab bar | 16px | Contenedor de tabs |
| Tab item | 12px | Dentro de tab bar (4px padding) |
| Dropdown menu | 12px | Contenedor de items |
| Dropdown item | 8px | Dentro de dropdown (4px padding) |
| Boton glass | 12px | Independiente |
| Boton pill | 9999px | Independiente (capsule) |
| Tooltip | 8px | Elemento pequeno |
| Input dentro de modal | 8px | Dentro de card/modal |

---

## 8. Interacciones y Animaciones

### 8.1 Transiciones Permitidas

```css
/* Base para todos los elementos glass */
transition: background-color 150ms ease,
            border-color 150ms ease,
            box-shadow 150ms ease,
            opacity 150ms ease;
```

### 8.2 Interactive Press (iOS-style)

Los elementos interactivos glass responden al toque con un efecto fisico:

```css
.glass-interactive:active {
  transform: scale(0.97);
  transition: transform 80ms ease-out;
}

.glass-interactive:not(:active) {
  transition: transform 300ms cubic-bezier(0.34, 1.56, 0.64, 1); /* bounce back */
}
```

### 8.3 Aparicion de Elementos Glass

```css
/* Modales, sheets, command palette */
@keyframes glass-appear {
  from {
    opacity: 0;
    transform: scale(0.96) translateY(8px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.glass-enter {
  animation: glass-appear 200ms cubic-bezier(0.16, 1, 0.3, 1);
}
```

### 8.4 Shimmer (Highlight en Interaccion)

Efecto sutil de brillo que recorre la superficie al interactuar:

```css
@keyframes glass-shimmer {
  0%   { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

.glass-shimmer:active::after {
  background: linear-gradient(
    90deg,
    transparent 0%,
    rgba(255, 255, 255, 0.08) 50%,
    transparent 100%
  );
  background-size: 200% 100%;
  animation: glass-shimmer 600ms ease;
}
```

### 8.5 Animaciones PROHIBIDAS

- `animate-ping` en cualquier elemento
- Glow pulsante (`box-shadow` animado con color)
- Blur animado (cambiar el valor de `backdrop-filter` blur en runtime — performance destructivo)
- Transiciones mayores a 300ms en elementos de UI funcional

---

## 9. Jerarquia de Profundidad (Z-Layers)

```
z-0    Contenido (listas, datos, charts)         → Solido, sin glass
z-10   Cards elevadas, widgets                    → glass regular
z-20   Sidebar, navbar (sticky)                   → glass regular
z-30   Dropdowns, popovers, tooltips              → glass elevated
z-40   Sheets, modales                            → glass elevated + overlay dim
z-50   Command palette                            → glass elevated + overlay dim
z-60   Toasts, notificaciones                     → glass regular (flotan sobre todo)
```

**Regla critica**: NUNCA colocar un elemento glass directamente sobre otro elemento glass. El glass no puede muestrear otro glass — el resultado visual es ruido. Si un modal glass abre un dropdown, el dropdown debe tener `--glass-elevated-bg` con opacidad suficiente para ser autonomo.

---

## 10. Accesibilidad

### 10.1 Reduce Transparency (Fallback Solido)

Cuando el usuario activa "Reduce Transparency" o el navegador no soporta `backdrop-filter`:

```css
@supports not (backdrop-filter: blur(1px)) {
  .glass,
  .glass-clear,
  .glass-elevated {
    background: #111113;           /* Solido, como UI_ARCHITEC.md */
    backdrop-filter: none;
    border: 1px solid #FFFFFF0A;   /* Borde ligeramente mas visible */
  }
}

/* Preferencia de usuario */
@media (prefers-reduced-transparency: reduce) {
  .glass,
  .glass-clear,
  .glass-elevated {
    background: #111113;
    backdrop-filter: none;
  }
}
```

### 10.2 Reduce Motion

```css
@media (prefers-reduced-motion: reduce) {
  .glass-enter {
    animation: none;
    opacity: 1;
  }

  .glass-interactive:active {
    transform: none;
  }

  .glass-shimmer:active::after {
    animation: none;
  }
}
```

### 10.3 Contraste Aumentado

```css
@media (prefers-contrast: more) {
  .glass,
  .glass-clear,
  .glass-elevated {
    background: rgba(17, 17, 19, 0.92);      /* Casi opaco */
    border-color: rgba(255, 255, 255, 0.25);  /* Borde mas fuerte */
  }
}
```

### 10.4 Legibilidad

- El texto sobre glass SIEMPRE usa `--text-primary` (#FAFAFA) o `--text-secondary` (#A1A1AA)
- Nunca `--text-tertiary` o `--text-ghost` sobre glass — contraste insuficiente
- Los iconos sobre glass son minimo `--text-secondary`
- Si el contraste es insuficiente con el fondo, aumentar la opacidad del `background` del glass

---

## 11. Donde Aplicar Glass vs Solido en AI Society Pro

### Glass (capa de navegacion)

| Componente | Variante | Justificacion |
|-----------|----------|---------------|
| Sidebar | Regular | Flota sobre el contenido, da contexto con refraccion |
| Header/Navbar | Regular | Sticky, content pasa debajo |
| Command Palette | Elevated | Overlay flotante de alta prioridad |
| Modales | Elevated | Dialogos que flotan sobre todo |
| Context menus | Elevated | Menus temporales |
| Toasts | Regular | Notificaciones flotantes |
| Tooltips | Regular | Informacion contextual |
| Quick Actions (floating) | Regular | Si se presenta como panel flotante |

### Solido (capa de contenido)

| Componente | Fondo | Justificacion |
|-----------|-------|---------------|
| Stat Cards / KPIs | #111113 | Datos — legibilidad maxima |
| Charts | #111113 | Visualizacion — precision |
| Activity List | #111113 | Lista densa de datos |
| System Health | #111113 | Estado critico — claridad |
| DataTables | #111113 | Datos tabulares — NO glass |
| Formularios / Inputs | #111113 | Entrada de datos — NO glass |
| Area de contenido principal | #09090B | Fondo base, sin efecto |

---

## 12. Performance

### 12.1 Reglas de Optimizacion

- `backdrop-filter` es GPU-intensive. Limitar a **maximo 3-4 superficies glass visibles simultaneamente**
- Usar `will-change: transform` en elementos glass con animaciones
- NO animar el valor de `blur()` — esto fuerza re-render del filtro cada frame
- Agrupar elementos glass cercanos (equivalente a `GlassEffectContainer` de SwiftUI) para compartir una unica region de sampling
- En mobile, reducir `blur` a 12px (de 16px) y eliminar `saturate` para mejor rendimiento

### 12.2 Fallback Progresivo

```css
/* Nivel 1: Full glass (desktop moderno) */
.glass { backdrop-filter: blur(16px) saturate(180%); background: rgba(17,17,19,0.72); }

/* Nivel 2: Glass simplificado (mobile, GPU limitada) */
@media (max-width: 768px) {
  .glass { backdrop-filter: blur(12px); background: rgba(17,17,19,0.80); }
}

/* Nivel 3: Sin glass (sin soporte o reduced transparency) */
@supports not (backdrop-filter: blur(1px)) {
  .glass { background: #111113; }
}
```

---

## 13. Resumen Visual de Valores

```
┌─────────────────────────┬──────────────────────────────┬───────────┐
│ Propiedad               │ Regular        │ Elevated              │
├─────────────────────────┼──────────────────────────────┼───────────┤
│ background opacity      │ 72%            │ 85%                   │
│ backdrop-filter blur    │ 16px           │ 24px                  │
│ backdrop saturate       │ 180%           │ 180%                  │
│ border                  │ #FFFFFF1F (12%)│ #FFFFFF1F (12%)       │
│ border-radius           │ 16px           │ 16px                  │
│ shadow                  │ 0 8px 32px     │ 0 25px 50px           │
│ specular gradient       │ 6% → 0%        │ 6% → 0%              │
│ inner highlight         │ inset 0 1px    │ inset 0 1px           │
│ z-layer                 │ 10-20          │ 30-50                 │
│ concentric child radius │ -4px del padre │ -4px del padre        │
└─────────────────────────┴──────────────────────────────┴───────────┘
```

---

## 14. Ejemplo Completo — Sidebar + Modal

```html
<!-- Sidebar Glass -->
<aside class="fixed left-0 top-0 h-screen w-[220px] z-20
              bg-glass-regular backdrop-blur-glass backdrop-saturate-glass
              border-r border-glass-border-subtle glass-shine">
  <!-- Nav items sobre superficie glass -->
  <nav class="p-3 flex flex-col gap-0.5">
    <a class="flex items-center gap-2.5 h-9 px-3 rounded-lg
              bg-white/[0.08] text-[#FAFAFA] text-[13px] font-medium">
      <span class="w-0.5 h-4 bg-[#FAFAFA] rounded-sm"></span>
      Dashboard
    </a>
    <a class="flex items-center gap-2.5 h-9 px-3 rounded-lg
              text-[#A1A1AA] text-[13px] hover:bg-white/[0.06] hover:text-[#FAFAFA]
              transition-colors duration-150">
      Proyectos
    </a>
  </nav>
</aside>

<!-- Modal Glass -->
<div class="fixed inset-0 z-40 flex items-center justify-center">
  <!-- Overlay dim -->
  <div class="absolute inset-0 bg-black/60"></div>
  <!-- Modal -->
  <div class="relative z-50 w-[480px]
              bg-glass-elevated backdrop-blur-glass-elevated backdrop-saturate-glass
              border border-glass-border rounded-glass shadow-glass-elevated
              glass-shine glass-enter">
    <div class="p-5 border-b border-glass-border-subtle">
      <h2 class="text-[16px] font-semibold text-[#FAFAFA]">Nuevo Proyecto</h2>
    </div>
    <div class="p-6">
      <!-- Inputs SOLIDOS dentro del modal glass -->
      <input class="w-full bg-[#111113] border border-white/[0.10] rounded-lg
                     px-3.5 py-2.5 text-[13px] text-[#FAFAFA]
                     placeholder:text-[#3F3F46] focus:border-white/[0.20]
                     focus:bg-[#1A1A1F] outline-none transition-colors duration-150" />
    </div>
    <div class="p-4 border-t border-glass-border-subtle flex justify-end gap-2">
      <button class="btn-glass">Crear</button>
    </div>
  </div>
</div>
```

---

## 15. Mantra

> "El glass es la ventana, no el contenido. Si no puedes ver a traves de el hacia algo que importa, no deberia ser glass."

Liquid Glass en AI Society Pro amplifica la sensacion de profundidad y modernidad sin sacrificar la legibilidad que establecimos en `UI_ARCHITEC.md`. La navegacion flota. El contenido es solido. Cada superficie sabe exactamente que es.
