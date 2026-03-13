# Art Direction: AI Society Pro — UI Redesign

## Filosofia Visual

La interfaz actual sufre de **ruido decorativo**: gradientes purple que compiten entre si, glows innecesarios, backdrop-blur en cada superficie, y una jerarquia visual plana donde todo brilla igual. Nada respira.

La nueva direccion es **tipografia como arquitectura**. El texto manda. Las superficies sirven. El color aparece solo cuando tiene algo que decir.

---

## 1. Tipografia

### Typeface: Sohne (Klim Type Foundry)

Sohne es una neo-grotesca que hereda la neutralidad de Akzidenz-Grotesk pero con la precision optica de una fuente disenada para pantalla. No decora. Comunica.

**Escala tipografica:**

| Uso | Peso | Tamano | Tracking |
|-----|------|--------|----------|
| Display / Hero numbers | Sohne Halbfett (600) | 48–64px | -0.02em |
| Titulos de seccion | Sohne Kraftig (500) | 18–22px | -0.01em |
| Labels / Navigation | Sohne Buch (400) | 14px | 0.00em |
| Subtitulos / Meta | Sohne Leicht (300) | 12–13px | 0.01em |
| Stat values (KPIs) | Sohne Halbfett (600) | 36–48px | -0.03em |
| Monospace / Datos tecnicos | Sohne Mono Buch | 13px | 0.00em |

**Reglas:**
- Nunca mas de 3 pesos en una misma vista
- Los numeros grandes son el punto focal. No necesitan iconos gigantes al lado
- El tracking negativo en display crea densidad visual sin peso decorativo
- Las mayusculas se usan solo en labels de categoria (AGENTES, LATENCIA), nunca en titulos

---

## 2. Color

### Paleta: Monocromatica con acento funcional

La paleta actual es monocolor purple. Todo es purple. Si todo es importante, nada lo es.

**Sistema de color nuevo:**

```
Background:
  --surface-0:    #09090B     (fondo principal, casi negro)
  --surface-1:    #111113     (cards, sidebar)
  --surface-2:    #1A1A1F     (hover, elevacion sutil)
  --surface-3:    #232329     (bordes activos, inputs)

Texto:
  --text-primary:   #FAFAFA     (titulos, valores)
  --text-secondary: #A1A1AA     (labels, descripciones)
  --text-tertiary:  #52525B     (metadata, timestamps)
  --text-ghost:     #3F3F46     (placeholders, disabled)

Acento — Solo cuando importa:
  --accent:         #FFFFFF     (accion primaria, estado activo)
  --accent-muted:   #A1A1AA     (accion secundaria)

Estado:
  --success:   #22C55E    (sin glow, sin ping animation)
  --warning:   #EAB308
  --error:     #EF4444
  --info:      #3B82F6
```

**Reglas:**
- El color de acento de marca (#764FF9) se mantiene SOLO en el logotipo y en un unico elemento por pantalla (un boton primario, un indicador activo). No en iconos, no en badges, no en fondos de cards
- Los bordes son `#FFFFFF08` o `#FFFFFF06`. Apenas visibles. La estructura se lee por espaciado, no por lineas
- Cero gradientes decorativos en la UI funcional. Los gradientes se reservan para el marketing
- Cero box-shadow glow (shadow-[#764FF9]/25 desaparece por completo)
- Los estados de color (success, error) usan el color puro en texto, nunca como fondo de badge con borde

---

## 3. Superficies y Espacio

### Principio: El espacio es el lujo

Referencia directa: Teenage Engineering, Studio 99, Rejouice.

**Cards:**
- Sin border-radius exagerado. `border-radius: 8px` maximo, `0px` es aceptable
- Sin backdrop-blur. El fondo es solido: `--surface-1`
- Sin "glow on hover". El hover es un cambio de background a `--surface-2`, nada mas
- El padding interno es generoso: 24px minimo, 32px en cards principales
- Sin iconos decorativos flotando en esquinas. Si hay un icono, esta alineado con el texto

**Sidebar:**
- Fondo solido `--surface-1`, sin transparency ni blur
- Items de navegacion: texto puro. El item activo se marca con `--text-primary` y un indicador de 2px a la izquierda. Sin fondos coloreados, sin rounded-xl highlight
- Los iconos de navegacion son opcionales. Si se usan, son 16px en `--text-tertiary`
- Sin seccion de avatar elaborada. Nombre + rol en texto plano al fondo

**Stat Cards:**
- El numero es el protagonista. Grande (36-48px), peso 600
- La label va arriba en 11px uppercase `--text-tertiary`
- Sin icono de 44px en un cuadrado redondeado. Si se necesita un indicador visual, es un dot de 6px del color de estado
- Sin borde animado. Sin "pointer-events-none absolute blur"
- La card es un rectangulo limpio con buen padding

---

## 4. Layout

### Grid y Estructura

- El sidebar es de 240px. Sin boton de collapse visible siempre. Si colapsa, colapsa a iconos de 48px
- El header es minimo: breadcrumb a la izquierda, acciones a la derecha. Sin search bar prominente (la busqueda es un shortcut cmd+K que abre modal)
- El contenido principal usa un max-width de 1200px centrado, con padding lateral de 32px
- Las stats van en grid responsive, no forzadas a 6 columnas en una fila
- Los charts tienen mas altura (380px minimo) y menos decoracion en headers

### Jerarquia de secciones:

```
1. Hero / Welcome       → Se simplifica a una linea de texto: "Dashboard"
                           con fecha debajo. Sin icono sparkles, sin card envolvente
2. KPIs                 → Grid de 3-4 cards, numeros grandes, sin ruido
3. Grafico principal    → Ocupa ancho completo o 2/3, con altura generosa
4. Actividad reciente   → Lista limpia, sin iconos repetitivos
5. Acciones rapidas     → Se mueven a un command palette (cmd+K),
                           no ocupan espacio permanente en el dashboard
```

---

## 5. Iconografia

- Lucide icons a 16-18px maximo
- Peso de stroke: 1.5px (no 2px default)
- Color: `--text-tertiary` por defecto, `--text-secondary` en hover
- Los iconos NO van dentro de cuadrados redondeados con fondo. Van solos
- Si un icono no anade informacion que el texto no tenga, se elimina

---

## 6. Interacciones y Estados

**Hover:**
- Cards: `background-color` cambia de `--surface-1` a `--surface-2`. Nada mas
- Links/Nav: `color` cambia de `--text-tertiary` a `--text-primary`
- Sin transform scale, sin box-shadow glow, sin transition de 500ms

**Active/Selected:**
- Sidebar item activo: texto en `--text-primary`, barra vertical de 2px en blanco
- Tab activo: underline de 1px o texto en `--text-primary`

**Transiciones:**
- Solo `background-color` y `color`, con `150ms ease`
- Sin animate-ping en ningun elemento de la UI funcional
- Sin "pointer-events-none absolute" para efectos de glow

---

## 7. Lo que se elimina

| Antes | Despues |
|-------|---------|
| Gradiente purple en logo, iconos, badges | Color solido, gradiente solo en logo |
| backdrop-blur-xl en todo | Fondos solidos |
| shadow-[#764FF9]/25 (purple glow) | Sin shadow o shadow neutral sutil |
| border-radius: 16px en todo | 0-8px segun contexto |
| animate-ping en status dots | Dot estatico de color |
| Iconos en cuadrados 44px con fondo | Iconos inline sin contenedor |
| "Buenas noches, Edgar" card elaborada | Texto plano: "Dashboard" + fecha |
| Quick Actions grid permanente | Command palette (cmd+K) |
| 6 stat cards en una fila | 3-4 cards con mejor jerarquia |

---

## 8. Referentes

| Referente | Lo que tomamos |
|-----------|---------------|
| **Sohne / Klim Type** | Tipografia como sistema. Escala precisa. Tracking intencional |
| **Teenage Engineering** | Grid modular. Blanco y negro. La informacion es el diseno |
| **Studio 99** | Espacio generoso. Tipografia bold como elemento visual. Minimalismo real |
| **Rejouice** | Contraste extremo. Texto grande sobre negro. Confianza en la tipografia |
| **uix.vikram profile cards** | Dark cards limpias. Jerarquia de texto clara. Glassmorphism controlado |
| **Model Discovery cards** | Editorial. Datos estructurados. Whitespace como elemento activo |

---

## 9. Mantra

> "Si tienes que decorar la informacion para que se vea bien, la informacion no esta bien estructurada."

La UI de AI Society Pro debe sentirse como un **instrumento de precision**, no como un showcase de efectos CSS. Cada pixel ocupa espacio por una razon. Si no la tiene, se va.
