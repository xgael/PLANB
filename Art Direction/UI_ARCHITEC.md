# UI Architecture: AI Society Pro — Component Specification

> Documento de referencia para mantener la consistencia visual en todos los componentes de la interfaz. Cada valor aqui listado es intencional. Si un componente no aparece aqui, se construye a partir de los tokens base.

---

## 1. Design Tokens

### 1.1 Superficies

```
--surface-0:    #09090B       fondo principal (body, areas de contenido)
--surface-1:    #111113       cards, sidebar, modales, dropdowns
--surface-2:    #1A1A1F       hover sobre surface-1, inputs focus
--surface-3:    #232329       bordes activos, input focus ring
```

### 1.2 Texto

```
--text-primary:     #FAFAFA       titulos, valores, texto principal
--text-secondary:   #A1A1AA       labels, descripciones, nav items
--text-tertiary:    #52525B       metadata, timestamps, placeholders activos
--text-ghost:       #3F3F46       placeholders, texto disabled, hints
```

### 1.3 Bordes

```
--border-subtle:    #FFFFFF06     separadores entre secciones, strokes de cards
--border-default:   #FFFFFF08     sidebar, header separators
--border-input:     #FFFFFF0A     bordes de inputs, kbd hints
--border-strong:    #FFFFFF12     bordes en hover o focus
```

### 1.4 Estado

```
--success:    #22C55E     operativo, completado, online
--warning:    #EAB308     alerta, rate limit, atencion
--error:      #EF4444     error, caido, fallo critico
--info:       #3B82F6     informacion, enlace, referencia
```

**Regla**: los colores de estado se aplican SOLO al texto o a un dot indicador. Nunca como fondo de badge, nunca como borde de card, nunca como glow.

### 1.5 Acento de marca

```
--brand:      #764FF9     UNICAMENTE en logotipo y maximo UN elemento por pantalla
```

### 1.6 Tipografia

**Familia**: `Inter` (sustituye a Sohne para web; mantener tracking y pesos equivalentes)

| Rol | Peso | Tamano | Tracking | Color |
|-----|------|--------|----------|-------|
| Display / Hero numbers | 600 | 42px | -2px | --text-primary |
| Titulo de pagina | 600 | 22px | -0.5px | --text-primary |
| Titulo de seccion | 500 | 14px | 0 | --text-primary |
| Body / Nav items | 400 | 13px | 0 | --text-secondary |
| Meta / Timestamps | 400 | 12px | 0 | --text-ghost |
| Labels uppercase | 500 | 11px | 1px | --text-tertiary |
| KPI subtitulo | 400 | 12px | 0 | --text-ghost |
| Unidades junto a KPI | 400 | 16px | 0 | --text-tertiary |
| Kbd / Hints | 500 | 10-11px | 0 | --text-ghost |

**Reglas tipograficas**:
- Maximo 3 pesos por vista (400, 500, 600)
- Las MAYUSCULAS se usan solo en labels de categoria (AGENTES, LATENCIA, ERRORES). Nunca en titulos
- Tracking negativo en numeros grandes (-2px en 42px) para crear densidad sin peso decorativo
- Nunca usar Montserrat, Poppins, o cualquier alternativa redondeada

### 1.7 Espaciado

```
--space-xs:     4px
--space-sm:     8px
--space-md:     12px
--space-lg:     16px
--space-xl:     24px
--space-2xl:    32px
```

### 1.8 Bordes redondeados

```
--radius-none:    0px       tabs, algunos elementos inline
--radius-sm:      2px       barras de chart, indicadores
--radius-md:      6px       nav items, badges, kbd
--radius-lg:      8px       cards, inputs, modales, charts
```

**Regla**: NUNCA usar border-radius mayor a 8px en componentes funcionales. El 16px, 20px, rounded-xl, rounded-full — se eliminan.

---

## 2. Botones

### 2.1 Boton Primario

```css
background:     #FAFAFA
color:          #09090B
font-size:      13px
font-weight:    500
padding:        8px 16px
border-radius:  6px
border:         none
```

**Hover**: `background: #E4E4E7`
**Active**: `background: #D4D4D8`
**Disabled**: `background: #3F3F46; color: #52525B; cursor: not-allowed`
**Transicion**: `background-color 150ms ease`

No sombra. No glow. No gradiente. No transform scale.

### 2.2 Boton Secundario

```css
background:     transparent
color:          #A1A1AA
font-size:      13px
font-weight:    500
padding:        8px 16px
border-radius:  6px
border:         1px solid #FFFFFF0A
```

**Hover**: `background: #FFFFFF08; color: #FAFAFA`
**Active**: `background: #FFFFFF0D`
**Disabled**: `color: #3F3F46; border-color: #FFFFFF06`

### 2.3 Boton Ghost / Texto

```css
background:     transparent
color:          #A1A1AA
font-size:      13px
font-weight:    400
padding:        8px 12px
border-radius:  6px
border:         none
```

**Hover**: `color: #FAFAFA; background: #FFFFFF06`
**Active**: `color: #FAFAFA`

### 2.4 Boton de Icono

```css
width:          32px
height:         32px
border-radius:  6px
background:     transparent
color:          #52525B   (icono)
display:        flex
align-items:    center
justify-content: center
```

**Hover**: `background: #FFFFFF08; color: #A1A1AA`
**Icono**: 16px, stroke-width 1.5px

### 2.5 Boton Destructivo

```css
background:     transparent
color:          #EF4444
font-size:      13px
font-weight:    500
padding:        8px 16px
border-radius:  6px
border:         1px solid #EF444433
```

**Hover**: `background: #EF444415; color: #F87171`

---

## 3. Cards

### 3.1 Card Base

```css
background:     #111113
border-radius:  8px
border:         1px solid #FFFFFF06
padding:        24px
```

**Hover** (si es interactiva): `background: #1A1A1F`
**Transicion**: `background-color 150ms ease`

Sin sombra. Sin backdrop-blur. Sin gradiente de borde.

### 3.2 Stat Card / KPI

Estructura vertical:

```
[LABEL]         11px, 500, uppercase, tracking 1px, --text-tertiary
[VALOR]         42px, 600, tracking -2px, --text-primary
  [UNIDAD]      16px, 400, --text-tertiary (inline si aplica, ej: "41 ms")
[SUBTITULO]     12px, 400, --text-ghost (opcional, ej: "0 activos")
```

```css
background:     #111113
border-radius:  8px
border:         1px solid #FFFFFF06
padding:        24px
gap:            8px (vertical)
```

**Reglas**:
- El numero es el protagonista. Sin icono decorativo al lado
- Si se necesita indicador de tendencia: flecha inline de 12px, no un icono en cuadrado
- Sin fondo de icono 44x44 con color. Solo el numero
- Maximo 4 KPIs por fila. Nunca 6 apretados

### 3.3 Card de Lista (Activity, Quick Actions)

```css
background:     #111113
border-radius:  8px
border:         1px solid #FFFFFF06
```

**Header de la card**:
```css
padding:        16px 24px
/* Titulo y accion secundaria en extremos */
```
- Titulo: 14px, 500, --text-primary
- Accion ("Ver todo"): 12px, 400, --text-tertiary

**Filas individuales**:
```css
padding:        12px 24px
border-top:     1px solid #FFFFFF06
justify-content: space-between
align-items:    center
```
- Texto principal: 13px, 400, --text-primary
- Texto meta: 11px, 400, --text-tertiary
- Timestamp: 12px, 400, --text-ghost

**Reglas**:
- Sin iconos en contenedores redondeados por fila
- Si una fila tiene estado (warning), el texto principal usa --warning, no un badge aparte
- El gap entre titulo y meta dentro de una fila es 2px

---

## 4. Inputs

### 4.1 Input de Texto

```css
background:     #111113
border:         1px solid #FFFFFF0A
border-radius:  8px
padding:        10px 14px
font-size:      13px
font-family:    Inter
color:          #FAFAFA
```

**Placeholder**: `color: #3F3F46`
**Hover**: `border-color: #FFFFFF12`
**Focus**: `border-color: #FFFFFF20; outline: none; background: #1A1A1F`
**Disabled**: `background: #0D0D0F; color: #3F3F46; border-color: #FFFFFF06`
**Error**: `border-color: #EF4444`

No box-shadow en focus. No ring azul/purple.

### 4.2 Label de Input

```css
font-size:      12px
font-weight:    500
color:          #A1A1AA
margin-bottom:  6px
```

### 4.3 Helper Text / Error Text

```css
font-size:      11px
font-weight:    400
margin-top:     4px
```
- Normal: `color: #52525B`
- Error: `color: #EF4444`

### 4.4 Textarea

Mismas reglas que Input de Texto pero con:
```css
min-height:     100px
resize:         vertical
```

### 4.5 Select / Dropdown Trigger

```css
background:     #111113
border:         1px solid #FFFFFF0A
border-radius:  8px
padding:        10px 14px
font-size:      13px
color:          #FAFAFA
/* Icono chevron-down a la derecha, 16px, --text-tertiary */
```

**Hover**: `border-color: #FFFFFF12`
**Open**: `border-color: #FFFFFF20; background: #1A1A1F`

### 4.6 Search Input (Command Style)

```css
background:     transparent
border:         1px solid #FFFFFF0A
border-radius:  6px
padding:        4px 10px
font-size:      12px
color:          #3F3F46
/* Contiene "Search" + hint "cmd+K" */
```

Este componente es compacto y actua como trigger del command palette. No es un search bar prominente.

### 4.7 Checkbox

```css
width:          16px
height:         16px
border:         1px solid #FFFFFF12
border-radius:  4px
background:     transparent
```

**Checked**: `background: #FAFAFA; border-color: #FAFAFA`
**Check mark**: `color: #09090B` (icono check de 12px)
**Hover**: `border-color: #FFFFFF20`

### 4.8 Toggle / Switch

```css
width:          36px
height:         20px
border-radius:  10px
background:     #3F3F46        (off)
```

**On**: `background: #FAFAFA`
**Thumb**: circulo de 16px, `#09090B` (on) o `#52525B` (off)
**Transicion**: `background-color 150ms ease`

---

## 5. Modales

### 5.1 Overlay

```css
background:     #09090BCC     (80% opacity)
backdrop-filter: none          /* SIN blur */
```

### 5.2 Modal Container

```css
background:     #111113
border:         1px solid #FFFFFF08
border-radius:  8px
padding:        0
max-width:      480px         (sm) | 640px (md) | 800px (lg)
box-shadow:     0 25px 50px #00000080
```

### 5.3 Modal Header

```css
padding:        20px 24px
border-bottom:  1px solid #FFFFFF06
```
- Titulo: 16px, 600, --text-primary
- Boton cerrar: icono X de 16px, --text-tertiary, hover --text-primary

### 5.4 Modal Body

```css
padding:        24px
```

### 5.5 Modal Footer

```css
padding:        16px 24px
border-top:     1px solid #FFFFFF06
display:        flex
justify-content: flex-end
gap:            8px
```

**Reglas**:
- Sin animacion de entrada scale+fade. Solo fade de 150ms
- Sin backdrop-blur. El overlay es opaco
- Los modales destructivos no tienen borde rojo. El boton de accion es el indicador

---

## 6. DataTable

### 6.1 Contenedor

```css
background:     #111113
border:         1px solid #FFFFFF06
border-radius:  8px
overflow:       hidden
```

### 6.2 Header de Tabla

```css
background:     #0D0D0F
border-bottom:  1px solid #FFFFFF06
```

**Celda de header**:
```css
padding:        10px 24px
font-size:      11px
font-weight:    500
text-transform: uppercase
letter-spacing: 1px
color:          #52525B
```

### 6.3 Fila de Datos

```css
border-bottom:  1px solid #FFFFFF06
```

**Celda de datos**:
```css
padding:        12px 24px
font-size:      13px
color:          #FAFAFA       (dato principal)
                #A1A1AA       (dato secundario)
                #52525B       (dato terciario)
```

**Hover fila**: `background: #FFFFFF06`
**Seleccionada**: `background: #FFFFFF08`

### 6.4 Footer / Paginacion

```css
padding:        12px 24px
border-top:     1px solid #FFFFFF06
```
- Texto info: 12px, --text-tertiary ("Mostrando 1-10 de 234")
- Botones paginacion: estilo Ghost, 13px

### 6.5 Celda con Estado

- Operativo: texto en #22C55E + dot de 6px
- Error/Caido: texto en #EF4444 + dot de 6px
- Sin badges de fondo coloreado
- Sin iconos en cuadrados redondeados

### 6.6 Fila Expandible

```css
/* Contenido expandido */
background:     #0D0D0F
padding:        16px 24px
border-bottom:  1px solid #FFFFFF06
```

---

## 7. Charts

### 7.1 Contenedor de Chart

```css
background:     #111113
border:         1px solid #FFFFFF06
border-radius:  8px
min-height:     320px
```

### 7.2 Header de Chart

```css
padding:        20px 24px 12px 24px
```
- Titulo: 14px, 500, --text-primary
- Subtitulo: 12px, 400, --text-tertiary
- Legend items: 11px, --text-tertiary con dot de 6px del color de la serie

### 7.3 Paleta de Datos (monocromatica)

```
Serie 1:    #FAFAFA      (principal, blanco)
Serie 2:    #A1A1AA      (secundaria, gris medio)
Serie 3:    #52525B      (terciaria, gris oscuro)
Serie 4:    #3F3F46      (cuaternaria, gris muy oscuro)
Fondo bar:  #1A1A1F      (background de progress bars)
```

**Regla**: NO usar el color de marca (#764FF9) ni ningun color saturado en charts funcionales. La data se diferencia por luminosidad, no por hue.

### 7.4 Bar Chart

```css
/* Barras */
border-radius:  2px 2px 0 0       (solo arriba)
gap:            6px                 (entre barras)
```

- Eje Y: oculto o muy sutil (11px, --text-ghost)
- Eje X: 11px, --text-tertiary
- Grid lines: #FFFFFF06 horizontal, sin verticales
- Sin animaciones de entrada en las barras

### 7.5 Progress Bar (horizontal)

```css
/* Track */
height:         6px
border-radius:  3px
background:     #1A1A1F

/* Fill */
border-radius:  3px
background:     #FAFAFA | #A1A1AA | #52525B | #3F3F46  (segun serie)
```

- Label a la izquierda: 11px, --text-tertiary, width fijo (64px)
- Gap entre label y barra: 12px
- Gap entre filas: 16px

### 7.6 Line Chart

```css
stroke-width:   1.5px
fill:           none
```
- Linea: color de la serie (ver paleta)
- Dot en hover: 4px circulo, fill del color de la serie
- Area debajo: gradiente vertical del color al transparente, opacity 0.1

### 7.7 Tooltips de Chart

```css
background:     #1A1A1F
border:         1px solid #FFFFFF0A
border-radius:  6px
padding:        8px 12px
font-size:      12px
color:          #FAFAFA
box-shadow:     0 4px 12px #00000040
```

---

## 8. Navigation

### 8.1 Sidebar

```css
width:          220px
background:     #111113
border-right:   1px solid #FFFFFF08
```

**Logo area**:
```css
height:         56px
padding:        0 20px
border-bottom:  1px solid #FFFFFF08
align-items:    center
```
- Nombre app: 15px, 600, --text-primary, tracking -0.3px
- Tag "Pro": 10px, 500, --text-tertiary, tracking 0.5px

### 8.2 Nav Item (Default)

```css
height:         36px
padding:        0 12px
border-radius:  6px
gap:            10px
align-items:    center
```
- Icono: 16px, --text-tertiary (#52525B), Lucide, stroke 1.5px
- Texto: 13px, 400, --text-secondary (#A1A1AA)

**Hover**: `background: #FFFFFF06; color (texto): #FAFAFA`

### 8.3 Nav Item (Activo)

```css
background:     #FFFFFF08
```
- Indicador: rectangulo de 2px width, 16px height, --text-primary, border-radius 1px
- Texto: 13px, 500, --text-primary (#FAFAFA)
- Sin icono (reemplazado por la barra indicadora)

**Regla**: el item activo se distingue por la barra de 2px + texto blanco. No por fondo coloreado, no por borde purple, no por icono mas grande.

### 8.4 User Section (Sidebar Footer)

```css
padding:        12px 16px
border-top:     1px solid #FFFFFF08
```
- Nombre: 13px, 500, --text-secondary
- Icono logout: 14px, --text-ghost, a la derecha

Sin avatar. Sin rol debajo del nombre. Sin gradiente de fondo.

### 8.5 Header Bar

```css
height:         48px
padding:        0 32px
border-bottom:  1px solid #FFFFFF06
justify-content: space-between
align-items:    center
```
- Breadcrumb/titulo: 13px, 500, --text-primary
- Acciones derecha: icono bell 16px (--text-tertiary), status dot 6px (--success), search trigger

---

## 9. Badges y Status

### 9.1 Status Dot

```css
width:          6px
height:         6px
border-radius:  50%
```
- Online/Operativo: `background: #22C55E`
- Warning: `background: #EAB308`
- Error/Caido: `background: #EF4444`
- Info: `background: #3B82F6`
- Neutral/Offline: `background: #52525B`

**Regla**: SIN animate-ping, SIN glow, SIN ring pulsante. Es un punto estatico.

### 9.2 Badge de Texto

```css
font-size:      11px
font-weight:    500
padding:        2px 8px
border-radius:  4px
background:     transparent
```
- Success: `color: #22C55E`
- Warning: `color: #EAB308`
- Error: `color: #EF4444`

Sin fondo coloreado. Sin borde de color. Solo texto del color de estado.

Si necesitas diferenciacion visual adicional, usa un fondo sutil:
```css
background:     #22C55E0D    (success, 5% opacity)
```

### 9.3 Counter Badge (Notificaciones)

```css
min-width:      18px
height:         18px
border-radius:  9px
background:     #FAFAFA
color:          #09090B
font-size:      10px
font-weight:    600
text-align:     center
padding:        0 5px
```

---

## 10. Tabs

### 10.1 Tab Bar

```css
border-bottom:  1px solid #FFFFFF06
gap:            0
```

### 10.2 Tab Item

```css
padding:        10px 16px
font-size:      13px
font-weight:    400
color:          #52525B
border-bottom:  2px solid transparent
```

**Hover**: `color: #A1A1AA`
**Activo**: `color: #FAFAFA; font-weight: 500; border-bottom-color: #FAFAFA`

Sin fondo redondeado. Sin badge de color. El indicador es un underline de 2px blanco.

---

## 11. Tooltips y Popovers

### 11.1 Tooltip

```css
background:     #1A1A1F
border:         1px solid #FFFFFF0A
border-radius:  6px
padding:        6px 10px
font-size:      12px
color:          #FAFAFA
box-shadow:     0 4px 12px #00000040
max-width:      240px
```

**Regla**: sin flecha/arrow. Aparece con fade de 100ms. Sin delay excesivo.

### 11.2 Popover / Dropdown Menu

```css
background:     #111113
border:         1px solid #FFFFFF08
border-radius:  8px
padding:        4px
box-shadow:     0 8px 24px #00000060
min-width:      180px
```

**Menu Item**:
```css
padding:        8px 12px
border-radius:  4px
font-size:      13px
color:          #A1A1AA
gap:            8px             (icono + texto)
```
- Icono: 16px, --text-tertiary
- Hover: `background: #FFFFFF08; color: #FAFAFA`
- Destructivo: `color: #EF4444`
- Separator: `height: 1px; background: #FFFFFF06; margin: 4px 0`

---

## 12. Command Palette (cmd+K)

```css
/* Overlay */
background:     #09090BCC

/* Container */
background:     #111113
border:         1px solid #FFFFFF08
border-radius:  8px
width:          560px
max-height:     400px
box-shadow:     0 25px 50px #00000080
```

**Search Input**:
```css
padding:        16px 20px
border-bottom:  1px solid #FFFFFF06
font-size:      15px
color:          #FAFAFA
background:     transparent
```
- Icono search: 18px, --text-tertiary
- Placeholder: --text-ghost

**Result Item**:
```css
padding:        10px 20px
font-size:      13px
color:          #A1A1AA
gap:            10px
```
- Icono: 16px, --text-tertiary
- Hover / Seleccionado: `background: #FFFFFF08; color: #FAFAFA`
- Shortcut hint (derecha): 11px, --text-ghost

---

## 13. Toast / Notificaciones

```css
background:     #111113
border:         1px solid #FFFFFF08
border-radius:  8px
padding:        12px 16px
box-shadow:     0 8px 24px #00000060
min-width:      320px
max-width:      420px
gap:            12px
```

- Titulo: 13px, 500, --text-primary
- Descripcion: 12px, 400, --text-secondary
- Indicador izquierdo: barra de 3px width, height completo, color de estado
- Dismiss: icono X de 14px, --text-ghost

Sin icono en cuadrado coloreado. El color de estado se comunica con la barra lateral.

---

## 14. Empty States

```css
padding:        48px 32px
text-align:     center
```

- Icono: 32px, --text-ghost (sin contenedor, sin fondo circular)
- Titulo: 16px, 500, --text-primary
- Descripcion: 13px, 400, --text-tertiary
- CTA: boton secundario, centrado

---

## 15. Skeleton / Loading

```css
background:     #1A1A1F
border-radius:  4px
```

Animacion: pulse sutil entre `#1A1A1F` y `#232329`, duracion 1.5s, ease-in-out. Sin shimmer horizontal brillante.

---

## 16. Sidebar Colapsado

```css
width:          48px
padding:        0
```

- Solo iconos de 16px centrados horizontalmente
- Tooltip con nombre del item al hover
- Logo se reduce a iniciales o icono

---

## 17. Reglas Generales de Interaccion

### Transiciones permitidas
```css
transition: background-color 150ms ease;
transition: color 150ms ease;
transition: border-color 150ms ease;
transition: opacity 150ms ease;
```

### Transiciones PROHIBIDAS
- `transform: scale()` en hover de cards o botones
- `box-shadow` animado (glow effects)
- `backdrop-filter` de cualquier tipo
- Duraciones mayores a 200ms en elementos de UI
- `animate-ping`, `animate-pulse` en elementos funcionales (excepto skeleton)

### Focus visible
```css
outline:        2px solid #FAFAFA40
outline-offset: 2px
```

Sin focus ring azul por defecto del navegador. Sin ring purple.

---

## 18. Iconografia

- **Libreria**: Lucide Icons
- **Tamano default**: 16px
- **Stroke width**: 1.5px
- **Color default**: --text-tertiary (#52525B)
- **Hover color**: --text-secondary (#A1A1AA)
- **Tamano maximo en UI**: 18px (headers, acciones principales)
- **Tamano en empty states**: 32px

**Reglas**:
- Los iconos van SOLOS, sin contenedor cuadrado con fondo
- Si un icono no aporta informacion que el texto no tenga, se elimina
- En nav items: el icono es opcional. Si se usa, esta alineado con el texto a 10px de gap
- En stat cards: NO hay icono. El numero ES el indicador visual

---

## 19. Resumen de Anatomia por Componente

```
Card genérica:          surface-1 + border-subtle + radius-lg + padding 24px
Card de lista:          surface-1 + border-subtle + radius-lg + filas con border-top
Stat card:              surface-1 + border-subtle + radius-lg + padding 24px + gap 8px
Input:                  surface-1 + border-input + radius-lg + padding 10px 14px
Boton primario:         text-primary bg + text inverted + radius-md + padding 8px 16px
Boton secundario:       transparent + border-input + radius-md + padding 8px 16px
Modal:                  surface-1 + border-default + radius-lg + shadow fuerte
Dropdown:               surface-1 + border-default + radius-lg + shadow media
Tooltip:                surface-2 + border-input + radius-md + shadow sutil
Tab:                    border-bottom 2px + text-primary cuando activo
Nav item:               radius-md + 36px height + icono 16px + texto 13px
Status:                 dot 6px (sin glow) + texto color estado
Chart:                  surface-1 + border-subtle + radius-lg + min-height 320px
DataTable:              surface-1 + border-subtle + radius-lg + header surface-0
```

---

## 20. Mantra de Implementacion

> "Ante la duda, quita. Si el componente funciona sin el elemento, el elemento sobra."

Cada propiedad CSS tiene un proposito. Si no puedes explicar por que esta ahi, no deberia estar. La interfaz de AI Society Pro es un instrumento, no una exhibicion.