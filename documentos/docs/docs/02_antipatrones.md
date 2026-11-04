# Informe de antipatrones y olores de código — versión 0

**Archivo analizado:** `v0/tienda_v0.cpp` (sin modificar desde el Seguimiento 1)

Cada hallazgo indica el archivo y la línea donde aparece y el problema que causa. Los errores encontrados se registran aquí y no se corrigen en la V0.

## Resumen

| # | Antipatrón / olor | Ubicación | Gravedad |
|---|---|---|---|
| 1 | Función que lo hace todo (God Function) | `v0/tienda_v0.cpp:210-358` | Alta |
| 2 | Estado global | `v0/tienda_v0.cpp:42-44` | Alta |
| 3 | Condicionales encadenados por forma de pago | `v0/tienda_v0.cpp:199-208` y `:311-321` | Alta |
| 4 | Código duplicado | `:312-316`, `:249-270`, `:176-185` y `:349-357`, `:340-347` y `:382-385` | Alta |
| 5 | Números mágicos | `:313`, `:315`, `:320`, `:164`, `:232` | Media |
| 6 | Mezcla de interfaz y lógica de negocio | `:210-358` | Alta |
| 7 | Modelo de dominio anémico | `v0/tienda_v0.cpp:16-40` | Media |
| 8 | Obsesión por los primitivos | `:35`, `:19`, `:300` | Media |
| 9 | Dato calculado que se pierde (error) | `:318-325`, `:344-346`, `:382-385` | Alta |
| 10 | Valor de retorno especial (-1) | `:87-94` y sus usos | Baja |
| 11 | Comparación frágil de texto | `:232` y `:285` | Baja |

---

## 1. Función que lo hace todo (God Function)

- **Dónde:** `registrarVenta()`, líneas 210 a 358 (149 líneas, más de un tercio del programa).
- **Qué hace:** en una sola función se piden datos, se validan, se arma el detalle, se suman repetidos, se elige la forma de pago, se calculan descuento e IVA, se descuenta el stock, se imprime el recibo y se lanzan las alertas.
- **Problema:** cualquier cambio (una forma de pago nueva, otro formato de recibo) obliga a editar esta función, y es difícil de leer y de probar por partes. Viola el principio de responsabilidad única.

## 2. Estado global

- **Dónde:** líneas 42 a 44: `inventario`, `ventas` y `siguienteNumeroVenta`.
- **Problema:** cualquier función puede leerlos y modificarlos sin control (por ejemplo, líneas 129, 331 y 335-336). El resultado de una función depende del orden de las llamadas y no se puede probar aislada.

## 3. Condicionales encadenados por forma de pago

- **Dónde:** `obtenerNombreFormaPago()` (líneas 199-208) y el cálculo de descuento y cargo (líneas 311-321).
- **Problema:** cada forma de pago es una rama `if / else if`. Agregar Nequi, contraentrega u otra obliga a modificar código que ya funciona en al menos tres lugares (líneas 192-197, 199-208 y 311-321) y a ajustar la validación `opcionPago > 3` (líneas 304 y 307). Viola el principio abierto/cerrado.

## 4. Código duplicado

| Qué se repite | Líneas |
|---|---|
| Descuento del 3 % para tarjeta y para transferencia | 313 y 315 |
| Recorrido del detalle para ubicar un producto ya agregado | 249-254 y 262-270 |
| Alerta de stock bajo | 176-185 (inventario) y 349-357 (venta) |
| Impresión de subtotal, descuento, IVA y total | 340-347 (venta) y 382-385 (reporte) |
| Búsqueda del producto por código al descontar y al alertar | 329 y 350 |

- **Problema:** un cambio de regla, por ejemplo el mínimo de stock o el porcentaje de descuento, debe hacerse en varios lugares y es fácil olvidar uno.

## 5. Números mágicos

- **Dónde:** `0.03` (líneas 313 y 315), `800` (línea 320), `84` (línea 164), el texto `"FIN"` (línea 232) y las opciones `1`, `2` y `3` (líneas 200-205, 304, 307, 312, 314 y 319).
- **Problema:** el valor no tiene nombre que explique su significado. Cambiar el descuento o el cargo obliga a buscarlos a mano. Solo `IVA` y `STOCK_MINIMO` (líneas 46-47) están bien definidos como constantes.

## 6. Mezcla de interfaz y lógica de negocio

- **Dónde:** `registrarVenta()` mezcla `cin`/`cout` con el cálculo de totales y la actualización de stock. Además llama a `consultarInventario()` en cada vuelta del ciclo (línea 226), que imprime todo el inventario.
- **Problema:** no se puede reutilizar la lógica con otra interfaz ni probarla sin la consola.

## 7. Modelo de dominio anémico

- **Dónde:** `Producto`, `DetalleVenta` y `Venta` (líneas 16 a 40).
- **Problema:** son estructuras sin comportamiento. Las reglas (calcular el subtotal de un renglón, descontar stock, calcular totales) viven en funciones externas, por lo que los objetos no protegen sus propios datos: el stock puede quedar negativo si alguien lo modifica directamente.

## 8. Obsesión por los primitivos

- **Dónde:** `tipo` como `string` (línea 19), `formaPago` como `string` (línea 35) y la forma de pago elegida como `int` (línea 300).
- **Problema:** se pierde significado y validación. Un valor mal escrito no lo detecta el compilador, y el tipo de producto no participa en ninguna regla.

## 9. Dato calculado que se pierde (error de la V0)

- **Dónde:** `cargoTransferencia` es una variable local (línea 318) que se suma al total (línea 325) y se imprime en la venta (líneas 344-346), pero `Venta` (líneas 32-40) no tiene dónde guardarlo y `reporteVentas()` no lo muestra (líneas 382-385).
- **Problema comprobado al ejecutar:** en una venta por transferencia de 2 unidades de `P001` el reporte muestra Subtotal 50000.00, Descuento 1500.00, IVA 9215.00 y Total 58515.00. La cuenta 50000 − 1500 + 9215 da 57715.00: faltan los 800 del cargo.
- **Cómo se corrige en la V1:** el cargo pasa a formar parte del cálculo de cobro y se guarda en la venta.

## 10. Valor de retorno especial (-1)

- **Dónde:** `buscarProductoPorCodigo()` devuelve -1 si no encuentra el producto (línea 93) y cada llamada debe compararlo (líneas 105, 237 y 330).
- **Problema:** olvidar la comprobación produce un acceso fuera de rango.

## 11. Comparación frágil de texto

- **Dónde:** línea 232 compara `"FIN"`, `"fin"` y `"Fin"` por separado; línea 285 compara `"s"` y `"S"`.
- **Problema:** `FiN` o `fIN` no se reconocerían como fin de la lista.

---

## Antipatrones que la V1 resolverá con patrones

| Antipatrón | Patrón que lo corrige |
|---|---|
| 3, 4 (descuento), 5 (0.03 y 800), 8 (forma de pago), 9 | Strategy |
| 3 (creación según la opción elegida), 8 | Factory Method |
| 1, 6 | Facade |

La justificación está en `03_justificacion_patrones.md`. Los antipatrones 2, 7 y 10 quedan identificados; su corrección no es el objetivo de los tres patrones elegidos.
