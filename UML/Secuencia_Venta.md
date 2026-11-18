# Diagrama de secuencia — registrar una venta

Escenario: el vendedor registra una venta pagada por **transferencia**, con la versión 1 (`v1/main.cpp`). Cada mensaje indica la línea del código que lo produce (por ejemplo `L437`). Los patrones aparecen entre paréntesis junto a su clase: `Caja` (Facade), `CreadorTransferencia` (Factory Method) y `PagoTransferencia` (Strategy).

```mermaid
sequenceDiagram
    actor Vendedor
    participant Consola
    participant Caja as Caja (Facade)
    participant Producto
    participant Creador as CreadorTransferencia (Factory Method)
    participant Pago as PagoTransferencia (Strategy)

    Vendedor->>Consola: 1. elige Registrar venta
    Vendedor->>Consola: 2. codigo del producto y cantidad
    Consola->>Caja: 3. buscar(codigo) - L387
    Caja-->>Consola: 4. Producto
    Consola->>Producto: 5. hayStock(cantidad + yaAgregada) - L403
    Producto-->>Consola: 6. true
    Consola->>Caja: 7. formasDePago() - L422
    Caja-->>Consola: 8. descripciones de las formas de pago
    Vendedor->>Consola: 9. elige forma de pago (Transferencia)
    Consola->>Caja: 10. registrarVenta(carrito, opcion) - L437
    Caja->>Creador: 11. crearPolitica() - L197
    Creador->>Pago: 12. crea
    Creador-->>Caja: 13. PoliticaPago
    loop por cada producto del carrito
        Caja->>Producto: 14. descontar(cantidad) - L215
    end
    Caja->>Pago: 15. nombre() - L218
    Caja->>Pago: 16. descuento(subtotal) - L219
    Caja->>Pago: 17. cargo() - L220
    Caja->>Caja: 18. ventas.push_back(venta) - L226
    Caja-->>Consola: 19. Venta
    Consola->>Consola: 20. imprimirResumen(venta) - L441
    Consola->>Caja: 21. alertasDeVenta(venta) - L443
    Caja->>Producto: 22. stockBajo() - L242
    Caja-->>Consola: 23. productos con stock bajo
    Consola-->>Vendedor: 24. recibo y alertas de stock
```
