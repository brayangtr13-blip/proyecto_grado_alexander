# Control Store · Prototipo navegable (HTML + CSS)

Prototipo navegable del sistema **Control Store** para el **Autoservicio
Ayuelal**. Construido **solo con HTML y CSS**: no hay una sola línea de
JavaScript.

Autor: Brayan Alexander Torres Peña · SENA · Tecnología en ADSO

---

## Cómo abrirlo

Doble clic en `index.html`. No necesita servidor ni internet: las
tipografías están incluidas en `assets/fonts/`.

Si prefieres servidor:

```bash
python -m http.server 5513
```

---

## Estructura

```
08-Prototipo-HTML-CSS/
├── index.html                    Página pública
├── assets/
│   ├── css/
│   │   ├── estilos.css           Puerta única: importa las 7 capas
│   │   ├── 01-variables.css      Colores, fuentes y medidas
│   │   ├── 02-base.css           Reinicio y texto base
│   │   ├── 03-componentes.css    Botones, tablas, tarjetas, etiquetas
│   │   ├── 04-panel.css          Menú lateral, barra y contenido
│   │   ├── 05-acceso.css         Login y recuperación
│   │   ├── 06-landing.css        Página pública
│   │   └── 07-prototipo.css      Lo que antes hacía el JavaScript
│   ├── fonts/                    13 tipografías locales (.woff2)
│   └── img/                      Avatares
└── pages/
    ├── company/                  login · password_recovery
    ├── roles/                    admin · cashier
    └── modules/
        ├── inventory/            4 páginas
        ├── sales/                3 páginas
        ├── delivery/             2 páginas
        ├── users/                3 páginas
        └── reports/              1 página
```

**18 páginas · 200 enlaces internos · 0 rotos.**

---

## Arquitectura del CSS: capas (ITCSS)

Sin JavaScript no hay MVC — no hay nada que controle. El patrón
arquitectónico se traslada al CSS: **las hojas se ordenan en capas, de lo
más general a lo más específico.**

| Capa | Alcance | Regla |
|---|---|---|
| 01 Variables | Todo el sitio | Ningún color se escribe fuera de aquí |
| 02 Base | Etiquetas HTML | Sin clases |
| 03 Componentes | Piezas reutilizables | Sin posición ni márgenes externos |
| 04 Panel | Distribución | Coloca, no decora |
| 05–06 Páginas | Login y landing | Solo lo que no se repite |
| 07 Prototipo | Comportamiento | Reemplazos del JavaScript |

**Por qué en ese orden:** en CSS gana la última regla. Al ir de general a
particular, cada capa afina la anterior sin necesidad de `!important`.

`estilos.css` es la **única puerta**: cada página enlaza un solo archivo.

```html
<link rel="stylesheet" href="../../assets/css/estilos.css">
```

Cambiar el orden de las capas, agregar una nueva o cambiar todo por
Bootstrap se hace **en ese archivo**, no en las 18 páginas.

---

## Patrones de diseño aplicados

### Design tokens

`01-variables.css` es el único archivo donde se escribe un color. Todo lo
demás usa `var()`.

```css
--azul-menu: #16213d;
--fuente-titulo: "Bungee", sans-serif;
--radio: 16px;
```

Cambiar la identidad visual del sistema completo = cambiar un archivo.

### Contraste verificado

Cada pareja de color se midió con la fórmula de luminancia de la WCAG.
El mínimo es **4.5:1** para texto normal y **3:1** para texto grande.

Se corrigieron cinco colores que no llegaban:

| Token | Antes | Ahora | Dónde fallaba |
|---|---|---|---|
| `--texto-tenue` (claro) | `#969db0` | `#656a77` | Encabezados de tabla: 2.71 |
| `--texto-tenue` (oscuro) | `#6f7f9e` | `#808eaa` | Los mismos: 4.07 |
| `--aviso-texto` | `#a9670f` | `#93590d` | Etiqueta "pendiente": 3.97 |
| `--acento` | `#2f6fd0` | `#2c68c3` | Texto sobre el fondo: 4.49 |
| `--neutra-texto` | *(reusaba otro)* | pareja propia | Etiqueta de rol en oscuro: 3.77 |

Además se separó el acento en dos papeles:

```css
--acento: #2c68c3;          /* texto y enlaces */
--acento-solido: #2f6fd0;   /* fondo con texto blanco encima */
--sobre-acento: #ffffff;
```

> **Por qué dos:** en el tema oscuro el acento se aclara para leerse sobre
> fondo negro. Pero los contadores y los logos usan ese color **de fondo**
> con texto blanco: al aclararse, el blanco dejaba de leerse (2.84:1).
> Separando los dos papeles, cada uno se ajusta sin estorbar al otro.

**Resultado: 18 páginas, dos temas, cero fallos de contraste.**

### Tema oscuro por redefinición

El tema oscuro **no reescribe estilos**: redefine los mismos tokens.

```css
body:has(#interruptor-tema:checked) {
  --fondo: #0d1526;
  --texto: #e7edf9;
}
```

Son 30 líneas en vez de duplicar las 6 hojas.

### Formularios con un solo ritmo

Las cinco pantallas de captura comparten ancho y reparto:

```
fila 1:  campo corto + campo corto     279 + 279
fila 2:  campo largo                   574
fila 3:  campo corto + campo corto     279 + 279
fila 4:  campo largo                   574
```

El ancho sale de `--ancho-formulario`, no de un número suelto en cada
hoja. Y una regla evita el hueco cuando queda un campo impar al final:

```css
.campos-fila > .campo:last-child:nth-child(odd) {
  grid-column: 1 / -1;
}
```

> **Por qué el formulario no ocupa toda la pantalla:** un campo tan ancho
> como el monitor obliga al ojo a saltar de un extremo a otro. Los 620px
> son una decisión, no un sobrante.

### Componentes con modificadores

Una clase base y variantes con `--`:

```html
<button class="boton boton--azul boton--chico">
<span class="etiqueta etiqueta--exito">
```

La base define la forma; el modificador solo cambia lo que difiere.

### Técnica de cajas

Cada página sigue la misma jerarquía:

```html
<div class="contenedor">     <!-- ancho y separación -->
  <section class="tarjeta">  <!-- caja con nombre -->
    ...
  </section>
</div>
```

**Regla:** una sola caja manda la distribución. Si el padre y el hijo
declaran rejilla, el espacio se parte dos veces y media pantalla queda
vacía. En el acceso, la rejilla vive únicamente en `.contenedor.acceso`;
`body` solo cede el alto.

También se unificaron las reglas que estaban escritas dos veces
(`.acceso-marca`, `.acceso-formulario` / `.acceso-panel`,
`.acceso-caja` / `.panel-caja` y su *media query*): ahora comparten
selector en vez de repetirse.

---

## Comportamiento sin JavaScript

Tres cosas que normalmente exigen JS se resolvieron con CSS:

| Función | Cómo |
|---|---|
| Tema claro / oscuro | `<input type="checkbox">` oculto + `body:has(...)` |
| Ventana de cerrar sesión | `.modal:target` — la abre el ancla `#modal-salida` |
| Pasos de la recuperación | `.caja-paso:target` con enlaces entre pasos |

Todo está en `07-prototipo.css`.

### Y el acceso navega de verdad

Sin JavaScript el botón **Entrar** no puede validar nada, así que es un
enlace directo al tablero. Los tres usuarios de prueba también: cada uno
entra al panel que le corresponde por su rol.

| Enlace | Lleva a |
|---|---|
| Entrar · Acceder con Google | `roles/admin.html` |
| admin@controlstore.com | `roles/admin.html` |
| juan.perez@controlstore.com | `roles/cashier.html` |
| luis.p@controlstore.com | `modules/delivery/delivery_read.html` |

> El tema oscuro **no se recuerda** al cambiar de página: guardar la
> preferencia exige `localStorage`, y eso es JavaScript.

---

## Lo que este prototipo no hace

Es una maqueta navegable. Los datos están escritos en el HTML.

- **El buscador no filtra.** Es el campo, sin la lógica.
- **Los formularios no validan ni guardan.**
- **El login no verifica nada** — cualquier página se abre directo.
- **Bloquear un usuario o quitar del carrito** no responde.
- **Imprimir el reporte** no está conectado.

Nada de eso es un error: son exactamente las funciones que aporta la
siguiente capa.

---

## Cómo se le agrega JavaScript después

El HTML ya está preparado. Cada punto donde entra la lógica tiene un
`id` o una clase estable, y **no hay un solo `onclick` en el HTML**.

| Qué conectar | Dónde engancha |
|---|---|
| Buscador de tablas | `#buscador-global` |
| Filas de una tabla | `.tabla tbody` |
| Envío de formularios | `#formulario-usuario`, `#formulario-linea`, … |
| Mensajes de resultado | `#resultado`, `#mensaje-venta` |
| Tema y sesión | `#interruptor-tema`, `#modal-salida` |

El orden recomendado:

1. **JavaScript propio** — buscador, validación y tablas dinámicas.
2. **PHP + MySQL** — reemplazar los datos escritos en el HTML por
   consultas. Las 12 tablas ya existen en `02-Base-de-datos/`.
3. **Bootstrap o React** — solo si hace falta. Como los colores y medidas
   viven en `01-variables.css`, se mapean a las variables del framework
   sin reescribir las páginas.

> **Por qué en ese orden:** cada paso deja el anterior funcionando. Si el
> framework no llega, el prototipo sigue siendo entregable.

---

## Sangría del HTML

Las tablas y las tarjetas se generaban desde JavaScript en una sola línea
—una llegaba a **2.320 caracteres**—. Se les dio sangría con una regla
que no cambia el dibujo:

> Un elemento se abre en varias líneas **solo si el navegador descarta
> los espacios que eso agrega**: cuando adentro hay elementos de bloque,
> cuando el contenedor es `flex` o `grid`, o cuando es un `<svg>`.

Si adentro solo hay texto y etiquetas en línea (`<a>`, `<span>`), la
línea se deja entera: partirla metería un espacio que sí se ve.

Línea más larga: **2.320 → 458** caracteres.

---

## Verificación

Comparado página por página contra `07-Frontend-MVC` midiendo la
**geometría real** de cada elemento: posición y tamaño en píxeles del
menú, la barra, el contenedor, el enlace activo, el contador, cada
indicador, la tabla, cada encabezado y cada fila, además de los colores,
la tipografía y el texto de las celdas.

**Las 15 páginas del panel dieron la misma huella.** La interfaz no cambió.

> Medir el `textContent` crudo no sirve: una tabla escrita en una línea y
> la misma con sangría dan textos distintos aunque el navegador las
> dibuje idénticas. Por eso la comparación es por píxeles.

- 18 páginas · 0 archivos `.js` · 0 etiquetas `<script>` · 0 `onclick`
- 200 enlaces internos verificados · 0 rotos
