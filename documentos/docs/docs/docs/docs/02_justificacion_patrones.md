# Justificación de los patrones — Seguimiento 2

Patrones elegidos para pasar de la versión 0 a la versión 1 de **Se Te Cayó el Calzón**, uno por cada grupo:

| Grupo | Patrón | Antipatrón de la V0 que corrige (ver `02_antipatrones.md`) |
|---|---|---|
| Creacional | Factory Method | Condicionales por forma de pago al decidir qué calcular (antipatrones 3 y 8) |
| Estructural | Facade (clase `Caja`) | Función que lo hace todo y mezcla de interfaz con lógica (antipatrones 1 y 6) |
| Comportamiento | Strategy | Descuentos duplicados, números mágicos y cargo perdido (antipatrones 3, 4, 5 y 9) |

> Combinación registrada: **Factory Method + Facade + Strategy**. Debe confirmarse con el docente que ningún otro equipo la haya registrado antes (regla 3.1 de la guía).

## 1. Strategy (comportamiento): formas de pago intercambiables

**Problema en la V0.** El descuento y el cargo se calculan con `if / else if` según el número de la forma de pago (`v0/tienda_v0.cpp:311-321`). El 3 % está escrito dos veces (líneas 313 y 315), el `800` aparece suelto (línea 320) y el cargo se pierde al guardar la venta (antipatrón 9).

**Solución.** Una interfaz `PoliticaPago` con `nombre()`, `descuento(subtotal)` y `cargo()`. Cada forma de pago es una clase: `PagoEfectivo`, `PagoTarjeta` y `PagoTransferencia`. El 3 % y los $800 viven como constantes dentro de su clase, y el cargo forma parte del resultado de la venta.

**Qué pasa si se retira.** Vuelven los `if / else if` y el descuento duplicado; agregar una forma de pago exige modificar el cálculo de la venta.

## 2. Factory Method (creacional): crear la política de pago elegida

**Problema en la V0.** La opción del menú (1, 2 o 3) se traduce a una forma de pago con condicionales en dos sitios (líneas 199-208 y 311-321), y los valores 1, 2 y 3 están repartidos en el código (antipatrón 5).

**Solución.** Una clase `FabricaPago` con el método `crear(opcion)` que devuelve un `PoliticaPago`. Es el único lugar que conoce qué clase concreta corresponde a cada opción; la venta solo conoce la interfaz.

**Qué pasa si se retira.** La creación de `PagoEfectivo`, `PagoTarjeta` y `PagoTransferencia` volvería a hacerse con condicionales dentro del flujo de venta.

## 3. Facade (estructural): la clase `Caja`

**Problema en la V0.** `registrarVenta()` tiene 149 líneas (210-358): pide datos, valida, calcula, descuenta stock e imprime. No se puede reutilizar la lógica sin la consola.

**Solución.** Una clase `Caja` que expone un método `registrarVenta(...)` y oculta la coordinación del proceso: validar existencias, pedir la política a la fábrica, calcular el cobro, descontar el stock y guardar la venta. La consola solo recoge datos, llama a `Caja` y muestra el resultado.

**Qué pasa si se retira.** La orquestación vuelve al menú y a funciones largas que mezclan entrada/salida con reglas de negocio.

## 4. Por qué los tres se integran en el flujo «registrar una venta»

Se cumple la regla de integración de la guía (3.1):

1. **Comparten una abstracción.** `FabricaPago` produce `PoliticaPago` y `Caja` la consume; `PagoEfectivo`, `PagoTarjeta` y `PagoTransferencia` son las implementaciones de esa misma interfaz.
2. **El objeto que produce uno lo usa otro.** Factory Method crea la estrategia → Strategy calcula el cobro → Facade (`Caja`) coordina ambas dentro del mismo método de venta.
3. **Al retirar cualquiera, reaparecen los problemas.**

| Si se retira… | Reaparece |
|---|---|
| Strategy | Condicionales y descuento duplicado en el cálculo del cobro |
| Factory Method | Condicionales para decidir qué política usar |
| Facade | Función larga que mezcla consola y reglas de negocio |

Secuencia del flujo en la V1: `main` → `Caja.registrarVenta(...)` → `FabricaPago.crear(opcion)` → `PoliticaPago.descuento()/cargo()` → se descuenta el stock y se guarda la venta.

## 5. Patrones considerados y descartados

| Patrón | Motivo |
|---|---|
| Observer (alerta de stock) | Resolvería la alerta duplicada, pero se prefirió limitar el diseño a un patrón por grupo y mantener el programa dentro de las 250-400 líneas. |
| Decorator (IVA y descuentos) | Se superpone con Strategy: cada forma de pago aplica una sola regla y no hay descuentos acumulables. |
| Singleton (inventario único) | Reintroduce estado global; la guía lo señala como posible antipatrón. |
| Builder (armar la venta) | Aportaría poco en un programa de consola de este tamaño. |

## 6. Cambio en vivo previsto para la sustentación

Con este diseño, agregar una nueva forma de pago (por ejemplo, `PagoContraentrega`) se resuelve con una clase nueva que implementa `PoliticaPago` y una línea en `FabricaPago`, sin modificar `Caja` ni las demás políticas.
