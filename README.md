/*
    Proyecto final - Patrones de Software
    Version 0 (V0): Se Te Cayo el Calzon
    Esta version NO utiliza patrones de diseno.
    Datos almacenados en memoria.
*/

#include <iostream>
#include <iomanip>
#include <string>
#include <vector>
#include <limits>

using namespace std;

struct Producto {
    string codigo;
    string nombre;
    string tipo;
    double precio;
    int stock;
};

struct DetalleVenta {
    string codigoProducto;
    string nombreProducto;
    int cantidad;
    double precioUnitario;
    double subtotal;
};

struct Venta {
    int numero;
    vector<DetalleVenta> detalles;
    string formaPago;
    double subtotal;
    double descuento;
    double iva;
    double total;
};

vector<Producto> inventario;
vector<Venta> ventas;
int siguienteNumeroVenta = 1;

const double IVA = 0.19;
const int STOCK_MINIMO = 5;

void limpiarEntrada() {
    cin.clear();
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}

int leerEntero(const string& mensaje) {
    int valor;
    while (true) {
        cout << mensaje;
        if (cin >> valor) {
            limpiarEntrada();
            return valor;
        }
        cout << "Entrada no valida. Intente de nuevo.\n";
        limpiarEntrada();
    }
}

double leerDouble(const string& mensaje) {
    double valor;
    while (true) {
        cout << mensaje;
        if (cin >> valor && valor >= 0) {
            limpiarEntrada();
            return valor;
        }
        cout << "Ingrese un numero valido mayor o igual a cero.\n";
        limpiarEntrada();
    }
}

string leerTexto(const string& mensaje) {
    string texto;
    cout << mensaje;
    getline(cin, texto);
    return texto;
}

int buscarProductoPorCodigo(const string& codigo) {
    for (int i = 0; i < static_cast<int>(inventario.size()); i++) {
        if (inventario[i].codigo == codigo) {
            return i;
        }
    }
    return -1;
}

void registrarProducto() {
    cout << "\n========== REGISTRAR PRODUCTO ==========\n";

    string codigo = leerTexto("Codigo: ");
    if (codigo.empty()) {
        cout << "El codigo no puede estar vacio.\n";
        return;
    }

    if (buscarProductoPorCodigo(codigo) != -1) {
        cout << "Ya existe un producto con ese codigo.\n";
        return;
    }

    string nombre = leerTexto("Nombre del producto: ");
    string tipo = leerTexto(
        "Tipo (lubricante, preservativo, accesorio, cuidado intimo u otro): "
    );
    double precio = leerDouble("Precio: $");
    int stock = leerEntero("Cantidad inicial en stock: ");

    if (nombre.empty() || tipo.empty() || stock < 0) {
        cout << "Nombre, tipo y stock deben ser validos.\n";
        return;
    }

    Producto nuevo;
    nuevo.codigo = codigo;
    nuevo.nombre = nombre;
    nuevo.tipo = tipo;
    nuevo.precio = precio;
    nuevo.stock = stock;

    inventario.push_back(nuevo);

    cout << "Producto registrado correctamente.\n";
}

void cargarProductosDeEjemplo() {
    Producto p1 = {"P001", "Lubricante personal", "Cuidado intimo", 25000, 12};
    Producto p2 = {"P002", "Preservativos", "Proteccion", 18000, 20};
    Producto p3 = {"P003", "Aceite para masajes", "Bienestar", 32000, 8};
    Producto p4 = {"P004", "Accesorio para adultos", "Accesorios", 65000, 6};
    Producto p5 = {"P005", "Gel de cuidado intimo", "Cuidado intimo", 22000, 4};

    inventario.push_back(p1);
    inventario.push_back(p2);
    inventario.push_back(p3);
    inventario.push_back(p4);
    inventario.push_back(p5);
}

void consultarInventario() {
    cout << "\n================ INVENTARIO ================\n";

    if (inventario.empty()) {
        cout << "No hay productos registrados.\n";
        return;
    }

    cout << left
         << setw(10) << "Codigo"
         << setw(30) << "Nombre"
         << setw(20) << "Tipo"
         << right
         << setw(14) << "Precio"
         << setw(10) << "Stock" << '\n';

    cout << string(84, '-') << '\n';

    for (const Producto& producto : inventario) {
        cout << left
             << setw(10) << producto.codigo
             << setw(30) << producto.nombre
             << setw(20) << producto.tipo
             << right
             << setw(14) << fixed << setprecision(2) << producto.precio
             << setw(10) << producto.stock << '\n';
    }

    cout << "\nProductos con stock igual o inferior al minimo:\n";
    bool hayAlertas = false;

    for (const Producto& producto : inventario) {
        if (producto.stock <= STOCK_MINIMO) {
            cout << "- " << producto.nombre
                 << " (stock: " << producto.stock << ")\n";
            hayAlertas = true;
        }
    }

    if (!hayAlertas) {
        cout << "No hay alertas de stock.\n";
    }
}

void mostrarFormasDePago() {
    cout << "\nFormas de pago:\n";
    cout << "1. Efectivo (sin descuento)\n";
    cout << "2. Tarjeta (3% de descuento)\n";
    cout << "3. Transferencia (3% de descuento y cargo fijo de $800)\n";
}

string obtenerNombreFormaPago(int opcion) {
    if (opcion == 1) {
        return "Efectivo";
    } else if (opcion == 2) {
        return "Tarjeta";
    } else if (opcion == 3) {
        return "Transferencia";
    }
    return "No definida";
}

void registrarVenta() {
    cout << "\n============= REGISTRAR VENTA =============\n";

    if (inventario.empty()) {
        cout << "No hay productos para vender.\n";
        return;
    }

    Venta nuevaVenta;
    nuevaVenta.numero = siguienteNumeroVenta;
    nuevaVenta.subtotal = 0;
    nuevaVenta.descuento = 0;
    nuevaVenta.iva = 0;
    nuevaVenta.total = 0;

    while (true) {
        consultarInventario();

        string codigo = leerTexto(
            "\nCodigo del producto a agregar (FIN para terminar): "
        );

        if (codigo == "FIN" || codigo == "fin" || codigo == "Fin") {
            break;
        }

        int indice = buscarProductoPorCodigo(codigo);
        if (indice == -1) {
            cout << "No se encontro un producto con ese codigo.\n";
            continue;
        }

        int cantidad = leerEntero("Cantidad que desea vender: ");
        if (cantidad <= 0) {
            cout << "La cantidad debe ser mayor que cero.\n";
            continue;
        }

        int cantidadYaAgregada = 0;
        for (const DetalleVenta& detalle : nuevaVenta.detalles) {
            if (detalle.codigoProducto == codigo) {
                cantidadYaAgregada = detalle.cantidad;
                break;
            }
        }

        if (cantidad + cantidadYaAgregada > inventario[indice].stock) {
            cout << "No hay suficientes unidades en inventario para esa cantidad total.\n";
            continue;
        }

        bool yaAgregado = false;
        for (DetalleVenta& detalle : nuevaVenta.detalles) {
            if (detalle.codigoProducto == codigo) {
                detalle.cantidad += cantidad;
                detalle.subtotal =
                    detalle.cantidad * detalle.precioUnitario;
                yaAgregado = true;
                break;
            }
        }

        if (!yaAgregado) {
            DetalleVenta detalle;
            detalle.codigoProducto = inventario[indice].codigo;
            detalle.nombreProducto = inventario[indice].nombre;
            detalle.cantidad = cantidad;
            detalle.precioUnitario = inventario[indice].precio;
            detalle.subtotal = detalle.precioUnitario * cantidad;
            nuevaVenta.detalles.push_back(detalle);
        }

        cout << "Producto agregado a la venta.\n";

        string continuar = leerTexto("Agregar otro producto? (s/n): ");
        if (continuar != "s" && continuar != "S") {
            break;
        }
    }

    if (nuevaVenta.detalles.empty()) {
        cout << "La venta se cancelo porque no se agregaron productos.\n";
        return;
    }

    for (const DetalleVenta& detalle : nuevaVenta.detalles) {
        nuevaVenta.subtotal += detalle.subtotal;
    }

    mostrarFormasDePago();
    int opcionPago;

    do {
        opcionPago = leerEntero("Seleccione la forma de pago: ");
        if (opcionPago < 1 || opcionPago > 3) {
            cout << "Seleccione una opcion entre 1 y 3.\n";
        }
    } while (opcionPago < 1 || opcionPago > 3);

    nuevaVenta.formaPago = obtenerNombreFormaPago(opcionPago);

    // Calculo directo de descuentos, sin utilizar patrones.
    if (opcionPago == 2) {
        nuevaVenta.descuento = nuevaVenta.subtotal * 0.03;
    } else if (opcionPago == 3) {
        nuevaVenta.descuento = nuevaVenta.subtotal * 0.03;
    }

    double cargoTransferencia = 0;
    if (opcionPago == 3) {
        cargoTransferencia = 800;
    }

    double baseImponible = nuevaVenta.subtotal - nuevaVenta.descuento;
    nuevaVenta.iva = baseImponible * IVA;
    nuevaVenta.total = baseImponible + nuevaVenta.iva + cargoTransferencia;

    // Actualizar el inventario luego de calcular la venta.
    for (const DetalleVenta& detalle : nuevaVenta.detalles) {
        int indice = buscarProductoPorCodigo(detalle.codigoProducto);
        if (indice != -1) {
            inventario[indice].stock -= detalle.cantidad;
        }
    }

    ventas.push_back(nuevaVenta);
    siguienteNumeroVenta++;

    cout << "\nVenta registrada correctamente.\n";
    cout << "Numero de venta: " << nuevaVenta.numero << '\n';
    cout << "Subtotal: $" << fixed << setprecision(2)
         << nuevaVenta.subtotal << '\n';
    cout << "Descuento: $" << nuevaVenta.descuento << '\n';
    cout << "IVA (19%): $" << nuevaVenta.iva << '\n';
    if (cargoTransferencia > 0) {
        cout << "Cargo por transferencia: $" << cargoTransferencia << '\n';
    }
    cout << "TOTAL: $" << nuevaVenta.total << '\n';

    for (const DetalleVenta& detalle : nuevaVenta.detalles) {
        int indice = buscarProductoPorCodigo(detalle.codigoProducto);
        if (indice != -1 && inventario[indice].stock <= STOCK_MINIMO) {
            cout << "ALERTA: stock bajo para "
                 << inventario[indice].nombre
                 << ". Unidades restantes: "
                 << inventario[indice].stock << '\n';
        }
    }
}

void reporteVentas() {
    cout << "\n================ REPORTE DE VENTAS ================\n";

    if (ventas.empty()) {
        cout << "Todavia no se han registrado ventas.\n";
        return;
    }

    double totalGeneral = 0;

    for (const Venta& venta : ventas) {
        cout << "\nVenta numero: " << venta.numero << '\n';
        cout << "Forma de pago: " << venta.formaPago << '\n';

        for (const DetalleVenta& detalle : venta.detalles) {
            cout << "  " << detalle.nombreProducto
                 << " | Cantidad: " << detalle.cantidad
                 << " | Precio unitario: $" << fixed << setprecision(2)
                 << detalle.precioUnitario
                 << " | Subtotal: $" << detalle.subtotal << '\n';
        }

        cout << "Subtotal: $" << venta.subtotal << '\n';
        cout << "Descuento: $" << venta.descuento << '\n';
        cout << "IVA: $" << venta.iva << '\n';
        cout << "Total de la venta: $" << venta.total << '\n';

        totalGeneral += venta.total;
    }

    cout << "\nCantidad de ventas realizadas: " << ventas.size() << '\n';
    cout << "Total vendido: $" << fixed << setprecision(2)
         << totalGeneral << '\n';
}

void mostrarMenu() {
    cout << "\n============================================\n";
    cout << "     SE TE CAYO EL CALZON - V0\n";
    cout << "============================================\n";
    cout << "1. Registrar producto\n";
    cout << "2. Consultar inventario\n";
    cout << "3. Registrar venta\n";
    cout << "4. Reporte de ventas\n";
    cout << "5. Salir\n";
    cout << "============================================\n";
}

int main() {
    cargarProductosDeEjemplo();

    int opcion;
    do {
        mostrarMenu();
        opcion = leerEntero("Seleccione una opcion: ");

        switch (opcion) {
            case 1:
                registrarProducto();
                break;
            case 2:
                consultarInventario();
                break;
            case 3:
                registrarVenta();
                break;
            case 4:
                reporteVentas();
                break;
            case 5:
                cout << "Gracias por comprar en Se Te Cayo el Calzon. Vuelva pronto.\n";
                break;
            default:
                cout << "Opcion no valida. Intente de nuevo.\n";
        }
    } while (opcion != 5);

    return 0;
}
