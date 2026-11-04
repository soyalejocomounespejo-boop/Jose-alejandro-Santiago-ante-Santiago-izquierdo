/*
    Proyecto final - Patrones de Software
    Version 1 (V1): Se Te Cayo el Calzon
    Misma tienda de la V0, redisenada con patrones de diseno:
      - CREACIONAL:     Factory Method (CreadorPago y sus creadores concretos)
      - ESTRUCTURAL:    Facade         (Caja)
      - COMPORTAMIENTO: Strategy       (PoliticaPago y sus politicas)
    Datos almacenados en memoria.
*/

#include <algorithm>
#include <cctype>
#include <iomanip>
#include <iostream>
#include <limits>
#include <memory>
#include <string>
#include <vector>

using namespace std;

// ============================================================
// MODELO
// ============================================================

struct Producto {
    static constexpr int STOCK_MINIMO = 5;

    string codigo;
    string nombre;
    string tipo;
    double precio;
    int stock;

    bool hayStock(int cantidad) const { return cantidad <= stock; }
    void descontar(int cantidad) { stock -= cantidad; }
    bool stockBajo() const { return stock <= STOCK_MINIMO; }
};

struct DetalleVenta {
    string codigoProducto;
    string nombreProducto;
    int cantidad;
    double precioUnitario;
    double subtotal;
};

struct Venta {
    int numero = 0;
    vector<DetalleVenta> detalles;
    string formaPago;
    double subtotal = 0;
    double descuento = 0;
    double iva = 0;
    double cargo = 0;  // en la V0 este dato se perdia
    double total = 0;
};

struct ItemCarrito {
    string codigo;
    int cantidad;
};

// ============================================================
// COMPORTAMIENTO: Strategy
// Cada forma de pago es una politica intercambiable que sabe
// calcular su propio descuento y cargo.
// ============================================================

const double DESCUENTO_ELECTRONICO = 0.03;
const double CARGO_TRANSFERENCIA = 800;

class PoliticaPago {
public:
    virtual ~PoliticaPago() = default;
    virtual string nombre() const = 0;
    virtual string descripcion() const = 0;
    virtual double descuento(double subtotal) const = 0;
    virtual double cargo() const = 0;
};

class PagoEfectivo : public PoliticaPago {
public:
    string nombre() const override { return "Efectivo"; }
    string descripcion() const override { return "Efectivo (sin descuento)"; }
    double descuento(double) const override { return 0; }
    double cargo() const override { return 0; }
};

class PagoTarjeta : public PoliticaPago {
public:
    string nombre() const override { return "Tarjeta"; }
    string descripcion() const override { return "Tarjeta (3% de descuento)"; }
    double descuento(double subtotal) const override {
        return subtotal * DESCUENTO_ELECTRONICO;
    }
    double cargo() const override { return 0; }
};

class PagoTransferencia : public PoliticaPago {
public:
    string nombre() const override { return "Transferencia"; }
    string descripcion() const override {
        return "Transferencia (3% de descuento y cargo fijo de $800)";
    }
    double descuento(double subtotal) const override {
        return subtotal * DESCUENTO_ELECTRONICO;
    }
    double cargo() const override { return CARGO_TRANSFERENCIA; }
};

// ============================================================
// CREACIONAL: Factory Method
// CreadorPago declara el metodo fabrica; cada creador concreto
// decide que politica de pago instanciar.
// ============================================================

class CreadorPago {
public:
    virtual ~CreadorPago() = default;
    virtual unique_ptr<PoliticaPago> crearPolitica() const = 0;
};

class CreadorEfectivo : public CreadorPago {
public:
    unique_ptr<PoliticaPago> crearPolitica() const override {
        return make_unique<PagoEfectivo>();
    }
};

class CreadorTarjeta : public CreadorPago {
public:
    unique_ptr<PoliticaPago> crearPolitica() const override {
        return make_unique<PagoTarjeta>();
    }
};

class CreadorTransferencia : public CreadorPago {
public:
    unique_ptr<PoliticaPago> crearPolitica() const override {
        return make_unique<PagoTransferencia>();
    }
};

// ============================================================
// ESTRUCTURAL: Facade
// Caja oculta el proceso de venta (validar, crear la politica de
// pago, calcular, descontar stock, guardar) detras de un metodo.
// ============================================================

class Caja {
public:
    static constexpr double IVA = 0.19;

    void registrarMedioPago(unique_ptr<CreadorPago> creador) {
        creadores.push_back(move(creador));
    }

    const vector<Producto>& inventario() const { return productos; }
    const vector<Venta>& historial() const { return ventas; }

    const Producto* buscar(const string& codigo) const {
        for (const Producto& p : productos) {
            if (p.codigo == codigo) return &p;
        }
        return nullptr;
    }

    string validarCodigo(const string& codigo) const {
        if (codigo.empty()) return "El codigo no puede estar vacio.";
        if (buscar(codigo) != nullptr) return "Ya existe un producto con ese codigo.";
        return "";
    }

    // Devuelve el mensaje de error, o "" si el producto quedo registrado.
    string registrarProducto(const Producto& nuevo) {
        string error = validarCodigo(nuevo.codigo);
        if (!error.empty()) return error;
        if (nuevo.nombre.empty() || nuevo.tipo.empty() || nuevo.stock < 0) {
            return "Nombre, tipo y stock deben ser validos.";
        }
        productos.push_back(nuevo);
        return "";
    }

    vector<string> formasDePago() const {
        vector<string> descripciones;
        for (const auto& creador : creadores) {
            descripciones.push_back(creador->crearPolitica()->descripcion());
        }
        return descripciones;
    }

    // Unico metodo que necesita el cliente para vender.
    Venta registrarVenta(const vector<ItemCarrito>& carrito, int opcionPago) {
        unique_ptr<PoliticaPago> politica =
            creadores.at(opcionPago - 1)->crearPolitica();  // Factory Method

        Venta venta;
        venta.numero = siguienteNumeroVenta++;

        for (const ItemCarrito& item : carrito) {
            Producto* producto = localizar(item.codigo);
            if (producto == nullptr) continue;

            DetalleVenta detalle;
            detalle.codigoProducto = producto->codigo;
            detalle.nombreProducto = producto->nombre;
            detalle.cantidad = item.cantidad;
            detalle.precioUnitario = producto->precio;
            detalle.subtotal = producto->precio * item.cantidad;

            venta.detalles.push_back(detalle);
            venta.subtotal += detalle.subtotal;
            producto->descontar(item.cantidad);
        }

        venta.formaPago = politica->nombre();               // Strategy
        venta.descuento = politica->descuento(venta.subtotal);
        venta.cargo = politica->cargo();

        double base = venta.subtotal - venta.descuento;
        venta.iva = base * IVA;
        venta.total = base + venta.iva + venta.cargo;

        ventas.push_back(venta);
        return venta;
    }

    vector<const Producto*> productosConStockBajo() const {
        vector<const Producto*> bajos;
        for (const Producto& p : productos) {
            if (p.stockBajo()) bajos.push_back(&p);
        }
        return bajos;
    }

    vector<const Producto*> alertasDeVenta(const Venta& venta) const {
        vector<const Producto*> alertas;
        for (const DetalleVenta& detalle : venta.detalles) {
            const Producto* p = buscar(detalle.codigoProducto);
            if (p != nullptr && p->stockBajo()) alertas.push_back(p);
        }
        return alertas;
    }

private:
    vector<Producto> productos;
    vector<Venta> ventas;
    vector<unique_ptr<CreadorPago>> creadores;
    int siguienteNumeroVenta = 1;

    Producto* localizar(const string& codigo) {
        for (Producto& p : productos) {
            if (p.codigo == codigo) return &p;
        }
        return nullptr;
    }
};

// ============================================================
// CONSOLA (entrada y salida; no contiene reglas de negocio)
// ============================================================

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

bool esFin(string texto) {
    transform(texto.begin(), texto.end(), texto.begin(),
              [](unsigned char c) { return tolower(c); });
    return texto == "fin";
}

void imprimirInventario(const Caja& caja) {
    cout << "\n================ INVENTARIO ================\n";

    if (caja.inventario().empty()) {
        cout << "No hay productos registrados.\n";
        return;
    }

    cout << left << setw(10) << "Codigo" << setw(30) << "Nombre"
         << setw(20) << "Tipo" << right << setw(14) << "Precio"
         << setw(10) << "Stock" << '\n';
    cout << string(84, '-') << '\n';

    for (const Producto& p : caja.inventario()) {
        cout << left << setw(10) << p.codigo << setw(30) << p.nombre
             << setw(20) << p.tipo << right << setw(14) << p.precio
             << setw(10) << p.stock << '\n';
    }

    cout << "\nProductos con stock igual o inferior al minimo:\n";
    vector<const Producto*> bajos = caja.productosConStockBajo();
    for (const Producto* p : bajos) {
        cout << "- " << p->nombre << " (stock: " << p->stock << ")\n";
    }
    if (bajos.empty()) cout << "No hay alertas de stock.\n";
}

void imprimirResumen(const Venta& venta) {
    cout << "Subtotal: $" << venta.subtotal << '\n';
    cout << "Descuento: $" << venta.descuento << '\n';
    cout << "IVA (19%): $" << venta.iva << '\n';
    if (venta.cargo > 0) {
        cout << "Cargo adicional (" << venta.formaPago << "): $"
             << venta.cargo << '\n';
    }
    cout << "TOTAL: $" << venta.total << '\n';
}

void pantallaRegistrarProducto(Caja& caja) {
    cout << "\n========== REGISTRAR PRODUCTO ==========\n";

    string codigo = leerTexto("Codigo: ");
    string error = caja.validarCodigo(codigo);
    if (!error.empty()) {
        cout << error << '\n';
        return;
    }

    Producto nuevo;
    nuevo.codigo = codigo;
    nuevo.nombre = leerTexto("Nombre del producto: ");
    nuevo.tipo = leerTexto(
        "Tipo (lubricante, preservativo, accesorio, cuidado intimo u otro): ");
    nuevo.precio = leerDouble("Precio: $");
    nuevo.stock = leerEntero("Cantidad inicial en stock: ");

    error = caja.registrarProducto(nuevo);
    cout << (error.empty() ? "Producto registrado correctamente." : error)
         << '\n';
}

void pantallaVenta(Caja& caja) {
    cout << "\n============= REGISTRAR VENTA =============\n";

    if (caja.inventario().empty()) {
        cout << "No hay productos para vender.\n";
        return;
    }

    vector<ItemCarrito> carrito;

    while (true) {
        imprimirInventario(caja);

        string codigo = leerTexto(
            "\nCodigo del producto a agregar (FIN para terminar): ");
        if (esFin(codigo)) break;

        const Producto* producto = caja.buscar(codigo);
        if (producto == nullptr) {
            cout << "No se encontro un producto con ese codigo.\n";
            continue;
        }

        int cantidad = leerEntero("Cantidad que desea vender: ");
        if (cantidad <= 0) {
            cout << "La cantidad debe ser mayor que cero.\n";
            continue;
        }

        auto existente = find_if(carrito.begin(), carrito.end(),
                                 [&](const ItemCarrito& i) { return i.codigo == codigo; });
        int yaAgregada = (existente != carrito.end()) ? existente->cantidad : 0;

        if (!producto->hayStock(cantidad + yaAgregada)) {
            cout << "No hay suficientes unidades en inventario para esa cantidad total.\n";
            continue;
        }

        if (existente != carrito.end()) existente->cantidad += cantidad;
        else carrito.push_back({codigo, cantidad});

        cout << "Producto agregado a la venta.\n";

        string continuar = leerTexto("Agregar otro producto? (s/n): ");
        if (continuar != "s" && continuar != "S") break;
    }

    if (carrito.empty()) {
        cout << "La venta se cancelo porque no se agregaron productos.\n";
        return;
    }

    vector<string> formas = caja.formasDePago();
    int cantidadFormas = static_cast<int>(formas.size());
    cout << "\nFormas de pago:\n";
    for (int i = 0; i < cantidadFormas; i++) {
        cout << (i + 1) << ". " << formas[i] << '\n';
    }

    int opcion;
    do {
        opcion = leerEntero("Seleccione la forma de pago: ");
        if (opcion < 1 || opcion > cantidadFormas) {
            cout << "Seleccione una opcion entre 1 y " << cantidadFormas << ".\n";
        }
    } while (opcion < 1 || opcion > cantidadFormas);

    Venta venta = caja.registrarVenta(carrito, opcion);

    cout << "\nVenta registrada correctamente.\n";
    cout << "Numero de venta: " << venta.numero << '\n';
    imprimirResumen(venta);

    for (const Producto* p : caja.alertasDeVenta(venta)) {
        cout << "ALERTA: stock bajo para " << p->nombre
             << ". Unidades restantes: " << p->stock << '\n';
    }
}

void pantallaReporte(const Caja& caja) {
    cout << "\n================ REPORTE DE VENTAS ================\n";

    if (caja.historial().empty()) {
        cout << "Todavia no se han registrado ventas.\n";
        return;
    }

    double totalGeneral = 0;
    for (const Venta& venta : caja.historial()) {
        cout << "\nVenta numero: " << venta.numero << '\n';
        cout << "Forma de pago: " << venta.formaPago << '\n';
        for (const DetalleVenta& d : venta.detalles) {
            cout << "  " << d.nombreProducto << " | Cantidad: " << d.cantidad
                 << " | Precio unitario: $" << d.precioUnitario
                 << " | Subtotal: $" << d.subtotal << '\n';
        }
        imprimirResumen(venta);
        totalGeneral += venta.total;
    }

    cout << "\nCantidad de ventas realizadas: " << caja.historial().size() << '\n';
    cout << "Total vendido: $" << totalGeneral << '\n';
}

void mostrarMenu() {
    cout << "\n============================================\n";
    cout << "     SE TE CAYO EL CALZON - V1\n";
    cout << "============================================\n";
    cout << "1. Registrar producto\n";
    cout << "2. Consultar inventario\n";
    cout << "3. Registrar venta\n";
    cout << "4. Reporte de ventas\n";
    cout << "5. Salir\n";
    cout << "============================================\n";
}

void cargarProductosDeEjemplo(Caja& caja) {
    caja.registrarProducto({"P001", "Lubricante personal", "Cuidado intimo", 25000, 12});
    caja.registrarProducto({"P002", "Preservativos", "Proteccion", 18000, 20});
    caja.registrarProducto({"P003", "Aceite para masajes", "Bienestar", 32000, 8});
    caja.registrarProducto({"P004", "Accesorio para adultos", "Accesorios", 65000, 6});
    caja.registrarProducto({"P005", "Gel de cuidado intimo", "Cuidado intimo", 22000, 4});
}

int main() {
    cout << fixed << setprecision(2);

    Caja caja;
    caja.registrarMedioPago(make_unique<CreadorEfectivo>());
    caja.registrarMedioPago(make_unique<CreadorTarjeta>());
    caja.registrarMedioPago(make_unique<CreadorTransferencia>());
    cargarProductosDeEjemplo(caja);

    int opcion;
    do {
        mostrarMenu();
        opcion = leerEntero("Seleccione una opcion: ");

        switch (opcion) {
            case 1: pantallaRegistrarProducto(caja); break;
            case 2: imprimirInventario(caja); break;
            case 3: pantallaVenta(caja); break;
            case 4: pantallaReporte(caja); break;
            case 5:
                cout << "Gracias por comprar en Se Te Cayo el Calzon. Vuelva pronto.\n";
                break;
            default:
                cout << "Opcion no valida. Intente de nuevo.\n";
        }
    } while (opcion != 5);

    return 0;
}
