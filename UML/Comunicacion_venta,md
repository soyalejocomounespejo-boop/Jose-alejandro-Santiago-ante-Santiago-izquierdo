# Diagrama de comunicación — registrar una venta

Mismo escenario que el diagrama de secuencia: venta pagada por **transferencia** en la versión 1 (`v1/main.cpp`). Los mensajes van numerados (1, 5, 5.1, 5.1.1…) según su orden y su anidamiento. Los patrones se identifican por color: 🟩 Facade (`Caja`) · 🟦 Factory Method (`CreadorTransferencia`) · 🟧 Strategy (`PagoTransferencia`).

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 90, "rankSpacing": 170, "curve": "basis"}}}%%
flowchart LR
    V(["Vendedor"])
    C[":Consola"]
    K[":Caja<br/>Facade"]
    P[":Producto"]
    F[":CreadorTransferencia<br/>Factory Method"]
    S[":PagoTransferencia<br/>Strategy"]

    V -->|"1: elige registrar venta y forma de pago"| C
    C -->|"2: buscar(codigo)"| K
    C -->|"3: hayStock(cantidad + yaAgregada)"| P
    C -->|"4: formasDePago()"| K
    C -->|"5: registrarVenta(carrito, opcion)"| K
    K -->|"5.1: crearPolitica()"| F
    F -->|"5.1.1: crea"| S
    K -->|"5.2: descontar(cantidad)"| P
    K -->|"5.3: nombre()"| S
    K -->|"5.4: descuento(subtotal)"| S
    K -->|"5.5: cargo()"| S
    C -->|"6: imprimirResumen(venta)"| C
    C -->|"7: alertasDeVenta(venta)"| K
    K -->|"7.1: stockBajo()"| P

    style V fill:#fff,stroke:#555,color:#000
    style C fill:#eceaff,stroke:#7b61c9,color:#000
    style P fill:#eceaff,stroke:#7b61c9,color:#000
    style K fill:#d1e7dd,stroke:#198754,stroke-width:2px,color:#000
    style F fill:#cfe2ff,stroke:#0d6efd,stroke-width:2px,color:#000
    style S fill:#ffe5b4,stroke:#fd7e14,stroke-width:2px,color:#000
```
