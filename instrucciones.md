# Tablero de Almacén — instrucciones

Prototipo de tablero para la reunión diaria y para la planificación de despacho.
Un solo archivo, `index.html`, sin servidor, sin dependencias y sin nada que
instalar.

---

## 1. Abrirlo

Doble clic en `index.html`. Se abre en el navegador y funciona.

Tiene cuatro pestañas:

| Pestaña | Para qué |
|---|---|
| **Diaria** | La de la reunión de las 9. Movimiento del día y estado de cada pasillo. |
| **Flujo** | La del líder de despacho. Dónde está cada pallet, cuánta gente hace falta y qué viene atrasado. |
| **Mensual** | La foto del mes: precisión de stock, evolución, sectores, causas de ajustes y horas perdidas por atrasos. |
| **Carga del día** | El formulario del cierre de jornada. |

Se puede entrar directo a una pestaña con `#diaria`, `#flujo`, `#mensual` o
`#carga` al final de la dirección. Sirve para dejar un acceso directo en el
celular.

---

## 2. Subirlo a internet (gratis)

Hace falta para dos cosas: verlo desde el celular e instalarlo como app.

**Netlify Drop** es el camino más corto — sin cuenta, sin git, sin tarjeta:

1. Poné `index.html` solo en una carpeta (el nombre del archivo tiene que ser
   exactamente `index.html`).
2. Entrá a **app.netlify.com/drop** y arrastrá la carpeta.
3. Te devuelve una dirección tipo `algo-random.netlify.app`.

Creando una cuenta gratis podés renombrar el sitio y el enlace queda permanente.
Alternativas equivalentes: Cloudflare Pages o GitHub Pages.

> **Por qué hace falta HTTPS:** el celular sólo permite instalar como app las
> páginas servidas por HTTPS. Netlify lo trae puesto. Abriendo el archivo local
> esa opción no aparece.

### Instalarlo como app en el celular

Con la página abierta desde la dirección de Netlify:

- **Android (Chrome):** menú ⋮ → *Agregar a pantalla de inicio*
- **iPhone (Safari):** botón compartir → *Agregar a pantalla de inicio*

Queda con ícono propio y abre sin barra de navegador.

---

## 3. La vista Flujo

Es la pantalla de trabajo del líder de despacho. Se carga y se mira en el día,
no al cierre. Todo lo que se toca ahí queda guardado solo.

### 3.1 El circuito

Sigue el recorrido real, el mismo que se registra en JD Edwards:

```
armado → [R] → guardado → [pasillo] → picking → [expo] → packing → [salida] → camión
```

Las cuatro tarjetas son las **colas**: dónde está parado el pallet y qué está
esperando.

| Cola | Qué significa | Le falta |
|---|---|---|
| **En R** | Armado ya lo dejó en la ubicación lógica R. | guardado, picking, packing y despacho |
| **En pasillo** | Guardado le asignó ubicación real y lo subió a JD. | picking, packing y despacho |
| **En expo** | Pickeado y dejado en expo con su número de orden. | packing y despacho |
| **Salida camiones** | Embalado, esperando al transporte. | despacho |

El número de pallets de cada cola **se edita en la tarjeta misma**. Debajo se ve
cuántas horas pide la próxima tarea sobre esos pallets.

### 3.2 Plan de horas del turno

Los **puestos** no son lo mismo que las colas: el packing son dos, local y
exportación, porque se refuerzan por separado según el volumen del día.

Cada puesto tiene por delante todo lo que esté parado desde su cola hacia atrás
— el picking también va a tener que pickear lo que todavía está en R. De ahí sale
la columna **Personas**: las horas hombre que pide ese puesto dividido el largo
del turno. Al lado se carga cuánta gente hay asignada y el tablero avisa si falta.

Arriba de la tabla hay tres cosas que se configuran:

| Campo | Qué hace |
|---|---|
| **Turno de X a Y** | La ventana horaria. Sólo estas horas cuentan, y de lunes a viernes. |
| **Exportación %** | Qué parte del circuito va a exportación. Reparte el trabajo entre los dos puestos de packing. |
| **h/pallet** | El estándar de cada puesto. Editable fila por fila. |

### 3.3 Compromisos de salida y el atraso

Acá se anota cada pallet que ya tiene fecha y hora de salida comprometida. Por
cada uno el tablero calcula:

```
atraso (horas hombre) = trabajo que pide el pallet − horas hábiles que quedan
```

**Horas hábiles**, no horas de reloj: entre el lunes a la tarde y el martes a la
mañana no trabaja nadie y el pallet no avanza. Contar el tiempo de corrido daría
un número mentiroso.

Ejemplo, con el estándar de 1,12 h por pallet: si tienen que salir 3 pallets el
martes a las 14:00 y nos los entregan 12 minutos antes, son **3,4 h de trabajo
contra 0,2 h disponibles → 3,2 horas hombre de atraso**. Lo mismo si hoy es lunes
y todavía no nos los dieron: el tablero lo muestra en rojo desde ahora, sin
esperar a que se venza.

Cada compromiso lleva:

- **Etapa**: dónde está. Incluye *Sin entregar (armado)*, para lo que todavía no
  llegó a nuestras manos. Al moverlo de etapa, el trabajo que falta baja solo.
- **Responsable**: de quién depende si llega tarde — armado, coordinación de
  envíos, revisión de calidad, transporte, almacén.
- **Despachado**: congela el atraso con el valor que tenía al salir el camión. Si
  no se congelara, el reloj seguiría corriendo y el número se iría a cualquier lado.

El destino importa: un pallet de exportación usa el estándar del packing de expo
y uno nacional el del local. Si los dos estándares son distintos, cada compromiso
pide lo suyo.

### 3.4 Dónde termina todo esto

En **Mensual → Atrasos**, sumadas por área las horas hombre que costaron los
atrasos del mes en curso. Es el número para llevar a una reunión: *"armado nos
generó 7 horas hombre de atraso este mes"*.

---

## 4. Cambiar los datos

Todos los números de ejemplo están juntos en un solo bloque, al principio del
`<script>`, bajo el comentario `/* ===== DATOS ===== */`. No hay que tocar nada más.

### Vista diaria

```js
const DIA = {
  pickeados:   14,   // pedidos armados
  embalados:   12,
  despachados: 11,
  pallets: { exportacion: 6,  nacional: 3  },
  boxes:   { exportacion: 24, nacional: 41 }   // "boxes" = bultos
};
```

### Estado de los pasillos

`estado` acepta `"ok"` (verde), `"leve"` (amarillo) o `"grave"` (rojo).
Cada desvío lleva el producto, la cantidad y si falta o sobra:

```js
{ cod:"D", nom:"Repuestos", quien:"D.I.", estado:"grave", desvios:[
    { item:"Bridas DN50",          cant:3, tipo:"falta" },
    { item:"Kit reparación BM-34", cant:1, tipo:"sobra" } ] }
```

Un pasillo sin desvíos lleva `desvios:[]` y muestra "Todo en su ubicación".

### Vista flujo

Son dos listas distintas y conviene no confundirlas:

| Constante | Qué es |
|---|---|
| `ETAPAS` | Las **colas** del circuito: R, pasillo, expo y salida. Definen las tarjetas. |
| `TAREAS` | Los **puestos** a los que se asigna gente, con su `hpp` (horas por pallet). `cola` dice desde qué cola tienen trabajo y `dest` limita el puesto a un destino. |
| `MEZCLA_EXPO` | % del circuito que va a exportación. |
| `TURNO` | Ventana horaria por defecto. |
| `RESPONSABLES` | Las áreas que pueden figurar como causa de un atraso. |
| `COMP_DEMO` | Compromisos de ejemplo. `dias` son días hábiles desde hoy (negativo = pasado), así el ejemplo se entiende cualquier día que se abra. |

Los valores de `hpp`, el turno y la mezcla son sólo el punto de partida: lo que se
edite desde la vista pisa a estos y queda guardado.

### Vista mensual

| Constante | Qué es |
|---|---|
| `SECTORES` | Los cinco KPI: recepción, pasillos, picking, packing, despacho. |
| `PASILLOS` | Precisión y cantidad de errores de cada pasillo en el mes. |
| `CAUSAS` | Motivos de los ajustes de inventario. |
| `MESES` / `PRECISION` | La serie de evolución de la precisión de stock. |
| `HITO_DUENO` | Índice del mes marcado con la línea vertical (0 = el primero). |
| `DIAS` / `DESPACHO` | Pallets despachados por día. |
| `PICK_CAL` / `PICK_VEL` | Precisión y líneas por hora del picking. |

Las horas de atraso del mensual no se cargan ahí: salen solas de los compromisos
de la vista Flujo.

Si cambiás mucho un rango de valores, puede que haya que ajustar `yMin` / `yMax`
del gráfico correspondiente, más abajo en `pintarCharts()`.

---

## 5. Exportar a PDF

Botón **Exportar PDF** arriba a la derecha → elegir *Guardar como PDF*.

Sale con encabezado propio y fecha de emisión, sin barra de navegación ni botones.
Si el tablero está en modo oscuro, lo pasa a claro solo antes de imprimir. Los
campos editables se imprimen como texto, sin cajas.

Exporta **la pestaña que estés viendo**: para el reporte mensual, primero abrí
Mensual y después exportá.

---

## 6. Lo que hay que saber

**Los datos viven en un solo navegador.** Lo que se carga —tanto el cierre de
jornada como el flujo y los compromisos— queda en el equipo donde se cargó
(localStorage). Si se carga en la PC, no aparece en el celular. Es la consecuencia
de no tener servidor.

Lo bueno de eso: **ningún dato de la empresa sale a internet**. El hosting sólo
sirve el HTML. Aunque la dirección sea pública, los números viven en el dispositivo.

**El atraso se mide contra el reloj.** Un compromiso abierto va cambiando de
número solo a lo largo del día, y por eso se recalcula cada minuto mientras la
pestaña está a la vista. El valor se congela recién al marcarlo como despachado.

**Los números que trae son de ejemplo.** Son realistas pero inventados. Sirven para
discutir qué se mide y cómo se muestra, no para tomar decisiones.

---

## 7. Decisiones pendientes

1. **Confirmar el reparto de la 1,12 h por pallet.** El total es el que usa hoy el
   líder de despacho, pero cómo se divide entre los cuatro puestos es una
   estimación puesta acá: guardado 0,25 · picking 0,50 · packing 0,25 · despacho
   0,12. El total no cambia si se reparte distinto, pero **sí cambia a quién le
   falta gente** y cuánto atraso muestra un pallet parado en el medio del
   circuito. Vale medir una semana con cronómetro.
2. **¿El packing de exportación tarda más que el local?** Arrancan iguales en 0,25.
   Si el de expo lleva más (cajón, documentación, pallet tratado), cargarlo
   distinto cambia lo que pide cada pedido de exportación.
3. **Confirmar la mezcla de exportación.** Está en 50%. Es lo único que reparte el
   trabajo entre los dos puestos de packing cuando se miran los pallets del
   circuito en bloque.
4. **¿Backend o no?** Es la única forma de que los datos se compartan entre
   dispositivos, de que el líder y la reunión vean lo mismo y de tener histórico
   real de atrasos. Todo lo demás depende de esto.
5. **Confirmar cómo se mide la precisión de stock.** El tablero asume *absoluta por
   ubicación* (cada ubicación con diferencia cuenta como error, sin compensar
   sobrantes con faltantes). Si hoy se mide en neto o por unidades, el número real
   va a ser más bajo al cambiar el criterio — conviene saberlo antes de comprometer
   la curva hacia arriba.
6. **Cuántos días de carga aguanta el equipo.** Si llenar el formulario o mantener
   los compromisos lleva más de dos o tres minutos, se abandona. Vale la pena
   medirlo la primera semana y recortar campos si hace falta.
