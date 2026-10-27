# Tablero de Almacén — instrucciones

Tablero para la reunión diaria, la planificación de despacho y la logística. Un
solo archivo, `index.html`, sin servidor y sin nada que instalar. (Para leer Excel
de JD baja un lector la primera vez, con internet; el CSV no necesita nada.)

La idea es **cargar lo menos posible**: casi todo sale del archivo de JD. A mano
quedan las horas y la energía del cierre del día, pesar el relleno de algunas cajas
de vez en cuando, y corregir el stock después de un conteo. El registro de packing
por orden está **pendiente**: mientras tanto se estima.

---

## 1. Abrirlo y moverse

Doble clic en `index.html`. Arriba están las páginas, y adentro de cada una sus
secciones:

| Página | Secciones | Para qué |
|---|---|---|
| **Hoy** | — | La portada: pedidos del día, *Para atender* (lo único que pide acción, con link a donde se resuelve), cajas y relleno usados, pasillos. |
| **Órdenes** | Recorrido · Historial de cargas · Estados de JD | Las SJ de 535 a 580. |
| **Flujo** | Circuito y personal · Compromisos de salida | La del líder de despacho. |
| **Insumos** | Qué pedir · Stock · Insumos y reglas | Sugerencias de pedido, stock y la configuración. |
| **Costos** | Resumen · Relleno · Variables e indicadores | El mes en plata, el tablero de relleno y qué se mide. |
| **Mensual** | Precisión de stock · Operación · Atrasos | La foto del mes. |
| **Cargar** | Archivo de JD · Cierre del día · Medir relleno · Packing · Copia de seguridad | Todo lo que se carga, en un solo lugar. |

Cada pantalla tiene su dirección: `#insumos/stock`, `#carga/relleno`,
`#costos/relleno`… Sirve para dejar accesos directos en el celular. Volver a una
página abre la última sección que se miró.

---

## 2. El archivo de JD (lo único que hay que cargar seguido)

1. En JD, abrí la pantalla de líneas de órdenes de venta (por ejemplo *Trabajar con
   órdenes de venta*, P4210/P42101) filtrando de **535 a 580**.
2. Exportá la grilla a **CSV** o **Excel**.
3. **Soltá el archivo en cualquier página del tablero**, o tocá *Cargar archivo de
   JD*. Abajo aparece un aviso con lo que cambió.

Con cada archivo el tablero, solo:

- ubica cada orden en su etapa y registra lo que avanzó desde la carga anterior;
- **descuenta del stock** lo que llevaron las SJ que llegaron a embalado (cajas,
  relleno, film, pallets), estimado con las reglas y las mediciones;
- cuenta pickeados, embalados y despachados del día, que van a *Hoy* y se
  precargan en el cierre del día;
- recalcula qué insumos hay que pedir.

Conviene cargarlo **varias veces al día** (al arrancar, al mediodía, antes del
cierre): el "hace cuánto está en la etapa" sólo es tan preciso como la frecuencia de
carga. *Hoy* avisa cuando el archivo tiene más de 3 horas.

**Deshacer la última carga** (en Cargar → Archivo de JD) vuelve todo a como estaba,
incluido el stock que esa carga descontó.

Pruebas: en `ejemplos/` hay `jde-ordenes-1.csv` y `jde-ordenes-2.csv`. Cargalos en ese
orden.

### Columnas

Sólo dos son obligatorias: **número de orden** y **siguiente estado**. Las demás
suman: tipo de orden, último estado, línea, artículo, descripción, cantidad, unidad,
fecha promesa, cliente, sucursal. Se reconocen solas en español o inglés. Si alguna
no aparece, en *Columnas del archivo* se elige a mano una vez y queda recordada.

Lee CSV con `;`, `,` o tabulación, en UTF-8, UTF-16 o Windows-1252; números con coma
o punto decimal; y fechas dd/mm/aaaa, aaaa-mm-dd, juliano de JD (`126295`) o celda de
fecha de Excel.

### Las etapas

| Siguiente estado | Etapa | Qué pasa |
|---|---|---|
| 535 | Stock y despiece | Se controla el stock. Si hay, va a despiece; si no, hay que pedir lo que falta. |
| 545 | Sugerencia de picking | |
| 550 | Pickeado | |
| 555 | Embalado | Al llegar acá se descuentan las cajas y el relleno del stock. |
| 580 | **Enviada — fin** | La orden sale del recorrido. |

Se leen del *siguiente estado* de cada línea. Una orden está donde está su **línea
más atrasada**; las líneas canceladas (último estado 980) no cuentan.

**Sin stock**: si el archivo trae la cantidad en espera (backorder) de cada línea,
las SJ que esperan mercadería se marcan *sin stock*, tienen su filtro y aparecen en
*Para atender*. Si en JD esa columna se llama de otra forma, elegila una vez en
*Columnas del archivo*.

Los códigos se cambian en *Órdenes → Estados de JD*. Ahí también se pueden **agregar
etapas**, por ejemplo si el despiece tiene su propio estado.

### Opciones

- **Tipos de orden**: `SJ` por defecto. Varios separados por coma, o vacío para todos.
- **"Trae todas las órdenes abiertas"**: la que deja de venir se da por salida.
  Destildalo si vas a cargar un listado parcial.

---

## 3. Insumos

### Los que hay

| Tipo | Cuáles |
|---|---|
| Caja | 32x24x12 · 32x24x24 · 9100 · 9200 · 9300 · S25 chica · S25 grande · 60x40x40 |
| Pallet | 60x40x40 · 77x68x65 · 77x68x80 · 120x80x80 |
| Relleno | Ranpak compactado · Ranpak liso · Relleno reciclado · Cartón procesado |
| Otro | Film stretch |

Cada relleno tiene un **origen**: *se compra*, *se compra, reciclado* o *se hace en
casa (reciclado)*. Lo hecho en casa (el cartón procesado) no tiene stock que
controlar ni entra nunca en una sugerencia de pedido; lo reciclado suma en el
tablero de relleno.

Stocks, mínimos, lotes, proveedores y costos son **de ejemplo**: se cargan los reales
en *Insumos → Insumos y reglas*.

### Cuánto lleva cada SJ

1. Cada línea se **clasifica** según su código y descripción, con los patrones de
   *Clases de artículo*. Las dos que trae (*Grande* / *Chico*, por diámetro DN) son de
   ejemplo: hay que armarlas con los artículos reales.
2. Las **reglas de consumo** dicen qué caja lleva cada clase y cuántas unidades entran
   por caja. Lo que trae: *Grande* → 1 caja 60x40x40 por unidad; *Chico* → 1 caja
   32x24x12 cada 2 unidades; *Sin clasificar* → 1 caja 32x24x24 por unidad; 1 pallet
   120x80x80 por orden; 1 rollo de film cada 8 órdenes.
3. El **relleno va por caja**: tantos kilos de cada relleno por cada caja de un tipo.
   Esas reglas usan el **promedio de lo medido** (ver abajo); mientras no haya
   mediciones, el valor que tiene la regla.

Una caja no se comparte entre órdenes: se redondea para arriba orden por orden.

### Qué pedir

```
a pedir = (lo que piden las SJ que no llegaron a embalado + mínimo) − stock − lo que ya viene
```

redondeado al **lote** del proveedor. Aparece en *Insumos → Qué pedir*, agrupado por
proveedor, con la fecha estimada de llegada. *Copiar pedido* deja el texto listo para
mandar; *Registrar como pedido* lo pasa a *en camino*; *Recibido* lo suma al stock.

### El stock

Baja solo cuando las SJ llegan a embalado (555). Como es una estimación, después de un conteo
físico se corrige en *Insumos → Stock*.

---

## 4. Relleno

### Medir

En **Cargar → Medir relleno**: elegí la caja y el relleno, anotá los kilos y Enter. El
campo queda listo para la próxima caja. Con unas pocas mediciones por combinación
alcanza; se pueden sumar más cuando se quiera. Pesar una combinación de caja y relleno
que no tenía regla la crea sola.

### El tablero

En **Costos → Relleno**, para el mes elegido:

- **Relleno reciclado**: kilos de cartón procesado y relleno reciclado usados.
- **Por caja**: kilos de relleno por caja, en promedio.
- **Reciclado**: qué parte del relleno fue reciclado.
- **Ahorro estimado**: los kilos reciclados al precio del relleno comprado que más se
  usa.

Más el detalle por tipo, los kilos reciclados por día y la tabla de cuánto lleva cada
caja, con de dónde sale cada valor (medido o de la regla).

---

## 5. Packing — pendiente

Todavía no se registra qué caja y cuánto relleno lleva cada SJ. Hasta que se resuelva,
el consumo de cada orden se **estima** con las reglas y las mediciones. *Cargar →
Packing* muestra lo estimado hoy, orden por orden: ahí va a ir el registro real, que
reemplazará a la estimación.

---

## 6. Cierre del día y costos

En **Cargar → Cierre del día**:

- Pedidos armados, embalados y despachados ya vienen **en azul**, sacados de JD.
  Revisalos y corregí sólo si hace falta. Lo que se tipea no se pisa.
- Cajas, relleno, pallets y film aparecen calculados (*de JD*); no se tipean.
- Quedan para cargar las horas hombre, la energía y lo que no sale de JD.

Las **variables** (*Costos → Variables e indicadores*) son cualquier cosa que se mida y
tenga costo. Las que agrupan insumos se calculan solas; las otras se cargan en el
cierre. Cada jornada guarda el precio del día.

Los **indicadores** son una cosa dividida otra sobre el mes: costo por orden salida,
relleno por caja, reciclado por caja, embalaje por orden, horas hombre por pedido…

---

## 7. Copia de seguridad

Todo vive en el navegador del equipo. En **Cargar → Copia de seguridad** se baja un
archivo con todo y se puede restaurar en otra PC o en el celular. Restaurar
**reemplaza** lo que había en ese equipo.

---

## 8. Subirlo a internet (gratis, con Vercel)

Hace falta para dos cosas: verlo desde el celular e instalarlo como app.

El repositorio ya está listo para **Vercel**: es un sitio estático, no tiene nada que
compilar, y `vercel.json` le dice a Vercel cómo servirlo.

1. Entrá a **vercel.com** con tu cuenta de GitHub.
2. *Add New… → Project* e importá el repositorio **Spirax-sarco-mvp**.
3. Dejá todo como viene: *Framework Preset* **Other**, sin *Build Command*, sin
   *Install Command*. Tocá **Deploy**.
4. Te devuelve una dirección tipo `spirax-sarco-mvp.vercel.app`. En *Settings →
   Domains* se le puede cambiar el nombre o poner un dominio propio.

Desde ahí, **cada push a `main` se publica solo** en un minuto.

> **Los datos no se mudan solos.** Lo cargado vive en el navegador y queda atado a
> la dirección de la página. Si ya venías usando la de GitHub Pages, antes de pasarte
> bajá una copia en *Cargar → Copia de seguridad* y restaurala en la dirección de
> Vercel.

> **Por qué hace falta HTTPS:** el celular sólo permite instalar como app las
> páginas servidas por HTTPS. Vercel lo trae puesto. Abriendo el archivo local esa
> opción no aparece.

### Instalarlo como app en el celular

Con la página abierta desde la dirección de Vercel:

- **Android (Chrome):** menú ⋮ → *Agregar a pantalla de inicio*
- **iPhone (Safari):** botón compartir → *Agregar a pantalla de inicio*

Queda con ícono propio y abre sin barra de navegador.

---

## 9. La vista Flujo

Es la pantalla de trabajo del líder de despacho. Se carga y se mira en el día,
no al cierre. Todo lo que se toca ahí queda guardado solo.

### 9.1 El circuito

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

### 9.2 Plan de horas del turno

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

### 9.3 Compromisos de salida y el atraso

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

### 9.4 Dónde termina todo esto

En **Mensual → Atrasos**, sumadas por área las horas hombre que costaron los
atrasos del mes en curso. Es el número para llevar a una reunión: *"armado nos
generó 7 horas hombre de atraso este mes"*.

---

## 10. Cambiar los datos

Todos los números de ejemplo están juntos en un solo bloque, al principio del
`<script>`, bajo el comentario `/* ===== DATOS ===== */`. No hay que tocar nada más.

### Hoy

Los números del día salen de JD. Lo único de ejemplo son los pasillos.

### Estado de los pasillos

`estado` acepta `"ok"` (verde), `"leve"` (amarillo) o `"grave"` (rojo).
Cada desvío lleva el producto, la cantidad y si falta o sobra:

```js
{ cod:"D", nom:"Repuestos", estado:"grave", desvios:[
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
| `DIAS` / `DESPACHO` | Pallets despachados por día. |
| `PICK_CAL` / `PICK_VEL` | Precisión y líneas por hora del picking. |

Las horas de atraso del mensual no se cargan ahí: salen solas de los compromisos
de la vista Flujo.

Si cambiás mucho un rango de valores, puede que haya que ajustar `yMin` / `yMax`
del gráfico correspondiente, más abajo en `pintarCharts()`.

---

## 11. Exportar a PDF

Botón **Exportar PDF** arriba a la derecha → elegir *Guardar como PDF*.

Sale con encabezado propio y fecha de emisión, sin barra de navegación ni botones.
Si el tablero está en modo oscuro, lo pasa a claro solo antes de imprimir. Los
campos editables se imprimen como texto, sin cajas.

Exporta **la pestaña que estés viendo**: para el reporte mensual, primero abrí
Mensual y después exportá.

---

## 12. Lo que hay que saber

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

## 13. Decisiones pendientes

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
7. **¿El despiece tiene estado propio en JD?** Hoy queda dentro de 535 (stock y
   despiece). Si tiene un código, se agrega como etapa en *Órdenes → Estados de JD*.
   Y confirmar cómo se llama en el exporte la columna de cantidad en espera, que es
   la que marca las SJ sin stock.
8. **Clasificar los artículos.** Las clases *Grande* / *Chico* y sus patrones
   (`DN50`, `DN15`…) son de ejemplo. Lo ideal es usar un dato que ya traiga JD
   (código de familia, categoría de artículo o peso) y agregarlo como columna al
   exportar; mientras tanto, los patrones por código o descripción alcanzan.
9. **Cuánto insumo lleva cada cosa.** Cajas por unidad, kg de relleno, rollos de
   film: hay que medirlo una semana para que las sugerencias de pedido sean
   confiables.
10. **Qué caja lleva cada artículo.** Ya están las 8 cajas, los 4 pallets y los 4
    rellenos reales, pero las reglas que los asignan (Grande → 60x40x40, Chico →
    32x24x12, 1 pallet 120x80x80 por orden) son de ejemplo. Falta armar las clases
    con los artículos reales y decir qué caja y qué pallet lleva cada una.
11. **Packing por orden.** Queda pendiente el registro de qué caja y cuánto relleno
    lleva cada SJ. Cuando esté, reemplaza a la estimación orden por orden.
