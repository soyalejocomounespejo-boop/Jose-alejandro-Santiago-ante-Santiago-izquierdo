# Diagrama de secuencia — registrar una venta

Escenario: el vendedor registra una venta pagada por **transferencia**, con la versión 1 (`v1/main.cpp`). Cada mensaje indica la línea del código que lo produce. Los patrones se identifican por color: 🟩 Facade (`Caja`) · 🟦 Factory Method (`CreadorTransferencia`) · 🟧 Strategy (`PagoTransferencia`).

```mermaid
sequenceDiagram
    autonumber
    actor V as Vendedor
    participant C as :Consola
    box rgb(209,231,221) Facade
        participant K as :Caja
    end
    participant P as :Producto
    box rgb(207,226,255) Factory Method
        participant F as :CreadorTransferencia
    end
    box rgb(255,229,180) Strategy
        participant S as :PagoTransferencia
    end

    V->>C: elige 3. Registrar venta
    V->>C: codigo del producto y cantidad
    C->>K: buscar(codigo) · L387
    K-->>C: Producto
    C->>P: hayStock(cantidad + yaAgregada) · L403
    P-->>C: true
    C->>K: formasDePago() · L422
    K-->>C: descripciones de las formas de pago
    V->>C: elige forma de pago (3. Transferencia)
    C->>K: registrarVenta(carrito, opcion) · L437
    activate K
    K->>F: crearPolitica() · L197
    F->>S: crea
    F-->>K: PoliticaPago
    loop por cada producto del carrito
        K->>P: descontar(cantidad) · L215
    end
    K->>S: nombre() · L218
    K->>S: descuento(subtotal) · L219
    K->>S: cargo() · L220
    K->>K: ventas.push_back(venta) · L226
    K-->>C: Venta
    deactivate K
    C->>C: imprimirResumen(venta) · L441
    C->>K: alertasDeVenta(venta) · L443
    K->>P: stockBajo() · L242
    K-->>C: productos con stock bajo
    C-->>V: recibo y alertas de stock
```
