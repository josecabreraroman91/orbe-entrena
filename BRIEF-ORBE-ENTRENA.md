# Brief — Orbe Entrena v10

Trabajás sobre `index.html` del repo `orbe-entrena` (un solo archivo, sin frameworks).

**Regla de oro: NO rediseñar de cero.** Todo lo funcional se PORTA desde el sistema que ya
funciona en Academia DG (repo `academia-dg`, archivos `cal-unif.html` y `alumnos-unif.html`).
Ese sistema está probado con 800+ alumnos y 13 profesores. Si algo ya existe ahí, se copia
el comportamiento, no se inventa uno nuevo.

**No tocar** la lógica de Firebase que ya anda: login con Google, resolución de academia por
`/miembros`, reglas de seguridad, suscripción de prueba de 30 días, importación/exportación CSV.
Esas partes están cerradas y probadas.

---

## PARTE 1 — Portar el modelo de datos de DG (prioridad alta)

### 1.1 Ficha del alumno

Hoy Orbe Entrena guarda: `nombre, nombreNorm, tel, sede, nivel, activo, nota, ts, por`.

En DG (`alumnos-unif.html`, líneas ~170-196 y ~519-526) la ficha real es:

| Campo | Control | Valores | Nota |
|---|---|---|---|
| `nombre` | texto | — | ya existe |
| `cat` | texto | `P`, `1` a `8`, admite decimales (`5.75`) | **1 es la más alta, 8 la más baja.** NO usar 7ma/6ta/5ta. Hoy se llama `nivel` |
| `lado` | select | `DRV` (drive) · `REV` (revés) · `AMB` (ambos) | **falta, agregar** |
| `tipo` | select | `G` (grupal) · `D` (dúo) · `IND` (individual) | **falta, agregar** |
| `precio` | texto numérico | precio por clase en Gs. | **falta, agregar.** Con separador de miles automático al tipear (en DG es `attachMiles()`) |
| `tel` | texto | — | ya existe |
| `nomHor` | texto | nombre corto | **falta.** Es el que se muestra en el calendario |
| `nomWpp` | texto | diminutivo | **falta.** Para mensajes de WhatsApp |
| `sede` | select | sedes de la academia | ya existe, se mantiene |

Mantener `nombreNorm`, `ts`, `por` y las validaciones que ya existen (las reglas de Firebase
las exigen). Al renombrar `nivel` → `cat`, seguir leyendo `nivel` si el dato viejo existe,
para no romper los alumnos ya cargados.

**El precio vive en el alumno**, cada uno con el suyo, editable. NO hacer una lista de precios
compartida por tipo de clase.

### 1.2 Celda del calendario

Hoy: `{ nombre, mod, ts, por }` con `MODOS = ['', 'G', 'D', 'IND', 'BLOQUE']`.

En DG (`cal-unif.html`, líneas ~392-430 y ~670+) el alumno dentro de la celda es:

    { nombre, nivel, lado, mod, nota }

Cambios:
- Agregar `nivel`, `lado` y `nota` a lo que se guarda en la celda.
- `mod` suma la opción `COACH` (clase propia del profe). Mantener `BLOQUE` que ya usa Entrena.
- Ejemplo real de DG: `{nombre:'Isaura Rivaldi', nivel:'7', lado:'REV', mod:'D', nota:''}`

### 1.3 Autocompletado al asignar en el calendario (lo más importante)

Esto es lo que hace práctico al sistema y hoy falta. En DG está en `cal-unif.html`:

1. Un `<datalist id="alumnosList">` que se llena con todos los nombres de alumnos
   (función `fillAlumnosDatalist()`, línea ~3989). El input del editor de celda usa
   `list="alumnosList"`.
2. Al escribir, `autocompletarDesdeNube()` (línea ~2968) busca el alumno por nombre
   normalizado y **completa solo tres campos: `nivel`, `lado` y `mod`**. Si el alumno no
   existe, no toca nada.
3. Hay un botón **fijo** "+ Crear alumno nuevo" que **guarda solo el nombre**, con el texto
   de ayuda: "Guarda solo el nombre. Los demás datos se completan luego en Alumnos."
   El botón está siempre visible, no aparece ni se esconde solo.

Portar los tres puntos. La búsqueda tiene que ignorar tildes y ñ (Orbe Entrena ya tiene
esa normalización, reutilizarla).

### 1.4 Precio fuera del calendario

El precio NO se muestra en ninguna celda del calendario ni en la vista del profe.
Vive en la ficha del alumno y nada más.

---

## PARTE 2 — Rediseño visual

### 2.1 Problema actual
Todas las secciones son tarjetas blancas apiladas una debajo de la otra sobre fondo crema,
con botones negros y sin color. Es prolijo pero plano y obliga a un scroll largo.

### 2.2 Secciones en pestañas
Reemplazar la pila vertical por una barra de pestañas y mostrar **una sección a la vez**.
Orden exacto:

1. **Calendario** (arranca en esta)
2. **Alumnos**
3. **Asistencia**
4. **Configuración**
5. **Datos** (importar/exportar)

La barra queda fija arriba (`position: sticky`) y con scroll horizontal en celular.
Cada pestaña con un ícono SVG inline (nada de emojis).

### 2.3 Paleta — "pádel" sobre la marca Orbe

    --court:      #0E6FE5   /* azul cancha: acento propio de Orbe Entrena */
    --court-deep: #0A4DA0   /* hover y textos sobre fondo claro */
    --court-soft: #E7F1FE   /* fondos suaves, foco de inputs */
    --ball:       #C8FF4D   /* lima Orbe: la pelota */
    --ball-deep:  #7E9A1F   /* texto sobre lima */
    --ball-soft:  #F1F7E0
    --ink:        #0B0E1A   /* tinta Orbe: barra superior y texto */
    --paper:      #FFFFFF
    --mist:       #F4F7FB   /* fondo de la app */
    --line:       #E4E9F1
    --muted:      #6B7686

Reglas:
- **Fondo claro, no oscuro.** Los profes marcan asistencia al sol al costado de la cancha;
  el fondo oscuro se lava con la luz.
- Azul cancha para acciones, foco y pestaña activa. Lima solo como chispa (badge de prueba,
  detalles), nunca como fondo grande.
- El azul cancha es el color de Orbe Entrena dentro de la línea (Tributa cian `#3DD7D0`,
  Tracker violeta `#7C6BFF`).

### 2.4 Tipografía
**Space Grotesk** en toda la app (es la tipografía de marca de Orbe), cargada desde Google Fonts.
Pesos 400/500/600. Jerarquía real: títulos de sección 18px/600, etiquetas de campo 12.5px
en `--muted`, texto normal 15px.

### 2.5 Barra superior
Fondo `--ink`, wordmark "Orbe Entrena" con el isotipo de la O y el punto en órbita
(el punto en lima, la O en blanco). Abajo una línea de 3px mitad azul cancha, mitad lima.
A la derecha el correo del usuario.

### 2.6 Franja de la prueba
Debajo de la barra, fondo `--court-soft`, texto `--court-deep`, con un badge lima que dice
"Prueba" y al lado los días restantes. Cuando vence, cambia a tono de alerta.

### 2.7 Detalles de UI
- Inputs: borde 1.5px `--line`, radio 10px; al enfocar borde `--court` + halo `--court-soft`.
- Botón principal: fondo `--court`, texto blanco, radio 11px.
- Filas de alumno: avatar circular con iniciales, nombre, teléfono y sede, y a la derecha
  chips de `cat`, `tipo` y `precio`.
- Celdas del calendario: la modalidad como etiqueta chica arriba, y cada alumno con su
  categoría en un cuadradito oscuro + el `nomHor`. Celda con gente = borde azul cancha.
- Nada de emojis: íconos SVG inline coherentes entre sí.
- Que funcione bien en celular (los profes lo usan desde el teléfono).

---

## Cómo verificar antes de dar por terminado

1. Abrir la app con Playwright y sacar capturas de las 5 secciones, en escritorio y en
   celular (390px de ancho). Revisar las capturas y corregir lo que se vea mal.
2. Probar el flujo completo: crear un alumno con todos los campos nuevos → abrir el
   calendario → escribir su nombre en una celda → confirmar que se autocompletan
   nivel, lado y modalidad.
3. Probar el botón "+ Crear alumno nuevo" desde el calendario (guarda solo el nombre).
4. Confirmar que el precio NO aparece en ninguna parte del calendario.
5. Confirmar que sigue andando lo de antes: login, cambio de día y sede, asistencia,
   exportar CSV.
6. Los alumnos ya cargados (con `nivel` viejo y sin los campos nuevos) tienen que seguir
   viéndose sin romper nada.

## Qué NO hacer

- No agregar librerías ni frameworks: sigue siendo un solo archivo HTML.
- No tocar las reglas de Firebase ni la lógica de suscripción.
- No inventar campos ni flujos que DG no tenga.
- No usar `localStorage` para datos de la academia (va todo a Firebase).
