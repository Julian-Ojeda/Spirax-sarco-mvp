# Tablero de Almacén

Tablero diario para la reunión de las 9:00 del almacén: movimiento del día, estado
de los pasillos, carga de trabajo de despacho y seguimiento mensual de la precisión
de stock.

**Página:** https://julian-ojeda.github.io/Spirax-sarco-mvp/

Un solo archivo (`index.html`), sin servidor, sin dependencias y sin nada que
instalar. Funciona en la PC y se puede agregar a la pantalla de inicio del celular
como app.

## Vistas

| Pestaña | Para qué |
|---|---|
| **Diaria** | Pedidos armados, embalados y despachados; pallets y bultos por destino; estado de cada pasillo con el detalle de qué producto está fuera de lugar. |
| **Flujo** | La del líder de despacho: dónde está parado cada pallet del circuito, cuántas horas hombre falta meterle, cuánta gente hace falta en cada puesto y qué compromisos vienen atrasados. |
| **Mensual** | Precisión de stock, su evolución, KPI por sector, pallets por día, causas de los ajustes y horas hombre perdidas por atrasos. |
| **Carga del día** | Formulario del cierre de jornada. |

## El circuito de despacho

La vista **Flujo** sigue el recorrido real, el mismo que se registra en JD Edwards:

```
armado → [R] → guardado → [pasillo] → picking → [expo] → packing → [salida] → camión
```

Entre corchetes van las **colas**: dónde está parado el pallet y qué está
esperando. El packing se reparte en dos puestos, local y exportación.

De ahí salen las dos cosas que el líder necesita:

- **Cuánta gente poner en cada puesto hoy**, a partir de los pallets parados en
  cada cola y el estándar de horas por pallet.
- **Cuánto atraso llevamos, en horas hombre.** Para cada compromiso de salida:
  el trabajo que pide el pallet menos las horas hábiles que quedan hasta que sale
  el camión. Si nos entregan 3 pallets 12 minutos antes de la salida, son 3,4 h de
  trabajo contra 0,2 h disponibles: **3,2 horas hombre de atraso**, anotadas
  contra el área que entregó tarde.

## Cómo se usa y cómo se cambian los datos

Ver **[instrucciones.md](instrucciones.md)**.

## Estado

Prototipo. Los números que trae son de ejemplo: realistas pero inventados. Sirven
para discutir qué se mide y cómo se presenta, no para tomar decisiones.

El estándar de 1,12 h por pallet está repartido entre los cuatro puestos con una
estimación que **hay que confirmar** — ver las decisiones pendientes al final de
[instrucciones.md](instrucciones.md).

Los datos que se cargan quedan en el navegador del dispositivo (localStorage), no
se envían a ningún servidor.
