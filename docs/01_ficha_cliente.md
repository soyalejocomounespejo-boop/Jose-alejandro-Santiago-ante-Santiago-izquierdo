# Ficha del cliente — Seguimiento 1

**Tienda:** Se Te Cayó el Calzón
**Tipo de tienda:** productos para el bienestar íntimo y la salud sexual
**Aplicación:** programa de consola en C++, datos en memoria (sin base de datos ni interfaz gráfica)

## 1. Tipo de tienda

Tienda que vende lubricantes, preservativos, accesorios para adultos, aceites de masaje y productos de cuidado íntimo. Maneja un catálogo pequeño, con precios distintos y existencias limitadas, y cobra IVA del 19 % en todas sus ventas. Acepta tres formas de pago: efectivo, tarjeta y transferencia.

El cliente es ficticio: un propietario o propietaria que atiende el mostrador y administra el inventario.

## 2. Necesidad que resuelve la aplicación

Hoy el negocio llevaría el control de forma manual, lo que genera tres problemas:

| Problema | Consecuencia |
|---|---|
| No se conoce con exactitud el stock de cada producto | Se vende lo que no hay o se repone tarde |
| El IVA, el descuento por forma de pago y el cargo por transferencia se calculan a mano | Errores de cobro |
| No hay un registro ordenado de lo vendido | No se sabe cuánto se vendió ni qué productos rotan |

La aplicación centraliza el catálogo, calcula automáticamente el valor a cobrar, descuenta el stock en cada venta, avisa cuando quedan pocas unidades y consolida las ventas en un reporte.

## 3. ¿Quiénes la usarán?

| Usuario | Perfil | Qué hace | Funciones |
|---|---|---|---|
| Vendedor/a | Atiende al público; conocimientos básicos de computador | Consulta disponibilidad, registra ventas y cobra | F2, F3, F4, F5 |
| Propietario/a | Administra el negocio y decide las compras | Registra productos, revisa inventario y alertas, consulta el reporte | F1, F2, F5, F6 |

La versión 0 no tiene inicio de sesión ni roles: ambos perfiles usan el mismo menú de consola, y en una tienda pequeña pueden ser la misma persona. Los clientes finales no interactúan con el programa.

## 4. Funciones mínimas

| # | Función | Descripción |
|---|---|---|
| F1 | Registrar productos | Código, nombre, precio, stock y tipo de producto |
| F2 | Consultar el inventario | Productos con su precio y su stock actual |
| F3 | Registrar una venta | Una venta puede incluir varios productos y cantidades |
| F4 | Aplicar descuentos o impuestos | IVA y descuentos según la forma de pago |
| F5 | Controlar el stock | Descontar lo vendido y avisar cuando quede por debajo del mínimo |
| F6 | Reporte de ventas | Ventas realizadas y total vendido |

## 5. Reglas de negocio

- IVA del 19 %, calculado sobre el subtotal menos el descuento.
- Efectivo: sin descuento.
- Tarjeta: 3 % de descuento sobre el subtotal.
- Transferencia: 3 % de descuento y un cargo fijo de $800.
- Total = subtotal − descuento + IVA + cargo (si aplica).
- Se alerta cuando el stock de un producto es igual o inferior a 5 unidades.
- No se puede vender más de lo que hay en existencia, contando lo ya agregado a la misma venta.
- Cada venta recibe un número consecutivo.

## 6. Historias de usuario

| # | Función | Historia |
|---|---|---|
| HU1 | F1 | **Como** propietario, **quiero** registrar productos con código, nombre, tipo, precio y stock inicial, **para** tener el catálogo disponible para vender. |
| HU2 | F2 | **Como** vendedor, **quiero** consultar el inventario con precios y existencias, **para** saber qué puedo ofrecer y qué debo reponer. |
| HU3 | F3 | **Como** vendedor, **quiero** registrar una venta con varios productos y cantidades, **para** cobrar al cliente en una sola transacción. |
| HU4 | F3 | **Como** vendedor, **quiero** que el sistema no me deje vender más unidades de las que hay, **para** no comprometer productos que no tengo. |
| HU5 | F4 | **Como** vendedor, **quiero** que el sistema calcule el descuento según la forma de pago y el IVA, **para** cobrar el valor correcto sin hacer cuentas a mano. |
| HU6 | F5 | **Como** propietario, **quiero** que el stock se descuente al vender y que el sistema me avise cuando queden pocas unidades, **para** reponer a tiempo. |
| HU7 | F6 | **Como** propietario, **quiero** ver todas las ventas realizadas y el total vendido, **para** conocer cómo va el negocio. |

### Criterios de aceptación

**HU1 — Registrar productos**
- El código no puede estar vacío ni repetirse.
- Nombre y tipo no pueden estar vacíos; el stock no puede ser negativo; el precio no puede ser negativo.
- Al terminar se confirma que el producto quedó registrado.

**HU2 — Consultar inventario**
- Se muestra código, nombre, tipo, precio y stock de cada producto.
- Si no hay productos, se informa que no hay registrados.

**HU3 — Registrar una venta**
- Se pueden agregar varios productos hasta escribir FIN o responder que no se agregan más.
- Si un producto se agrega dos veces, se suman las cantidades en un solo renglón.
- La cantidad debe ser mayor que cero.
- Si no se agrega ningún producto, la venta se cancela.

**HU4 — Control de existencias en la venta**
- No se acepta una cantidad que, sumada a la ya agregada, supere el stock disponible.

**HU5 — Descuentos e impuestos**
- Efectivo, tarjeta y transferencia aplican las reglas de la sección 5.
- Al cerrar la venta se muestra subtotal, descuento, IVA, cargo (si aplica) y total.

**HU6 — Control de stock**
- Al registrar la venta, el stock baja según lo vendido.
- Si el stock resultante es igual o inferior a 5, se muestra una alerta con el producto y las unidades restantes.

**HU7 — Reporte de ventas**
- Por cada venta se muestra número, forma de pago, detalle de productos, subtotal, descuento, IVA y total.
- Al final se muestra la cantidad de ventas y el total vendido.
- Si no hay ventas, se informa que todavía no se han registrado.

## 7. Trazabilidad historias ↔ código de la versión 0

| Historia | Dónde se implementa en `v0/tienda_v0.cpp` |
|---|---|
| HU1 | `registrarProducto()` (líneas 96–132) |
| HU2 | `consultarInventario()` (líneas 148–190) |
| HU3 y HU4 | `registrarVenta()` (líneas 210–293) |
| HU5 | `registrarVenta()` (líneas 299–325) y `obtenerNombreFormaPago()` (líneas 199–208) |
| HU6 | `registrarVenta()` (líneas 327–333 y 349–357) y `consultarInventario()` (líneas 176–189) |
| HU7 | `reporteVentas()` (líneas 360–393) |

## 8. Fuera del alcance

Persistencia en archivos o base de datos, usuarios y roles, devoluciones y anulación de ventas, facturación electrónica, interfaz gráfica y venta en línea.
