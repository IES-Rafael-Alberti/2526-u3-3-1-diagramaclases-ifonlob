# Solución: Ejercicio 3 – Sistema de Gestión de Pedidos E-Commerce

## Análisis del Problema

### Identificación de Clases

1. **Cliente**
    - Representa a la persona que compra en la tienda.
    - Atributos: nombre, direccionEnvio, telefono, correo.
2. **Cuenta**
    - Monedero electrónico de un cliente.
    - Atributos: numero, saldo.
    - Métodos: recargarSaldo(), comprar().
3. **TarjetaCredito**
    - Medio de recarga y pago.
    - Atributos: numero, saldo, fechaVencimiento.
4. **Producto**
    - Artículo del inventario.
    - Atributos: codigo, nombre, descripcion, stock, pvp.
5. **Pedido** (abstracta)
    - Pedido genérico, heredan los tipos.
    - Atributos: estado, costo, numero.
    - Subclases:
        - **PedidoSimple**
            - Métodos: agregarProducto(), calcularCosto().
        - **PedidoCompuesto**
            - Métodos: agregarPedido(), calcularCostoTotal().
6. **LineaPedido**
    - Detalle de producto y cantidad de un pedido.
    - Atributos: cantidad, precioProducto.
7. **ProcesadorCobros** (Singleton)
    - Clase única para cobrar pedidos.
    - Métodos: realizarRevision(estado).
8. **estadoPedido** (enum)
    - PENDIENTE, COBRADO, DISTRIBUCION, CONFIRMADO.

## Análisis de Relaciones

### 1. Cliente–Cuenta (Composición)

- Un Cliente puede tener una o varias Cuentas.
- Justificación: El cliente puede eliminarse con todas sus cuentas.


### 2. Cuenta–Pedido

- Una Cuenta realiza varios Pedidos.
- Restricción: Solo si la Cuenta tiene saldo suficiente y es del Cliente dueño del Pedido.


### 3. Cuenta–Tarjeta

- Una Cuenta está asociada a una única Tarjeta de Crédito.


### 4. Pedido–PedidoSimple/PedidoCompuesto (Herencia, Composite)

- PedidoCompuesto *contiene* 2 o más Pedidos (simula arbol jerárquico de pedidos).


### 5. PedidoSimple–Cuenta (Pago)

- PedidoSimple se paga siempre con una única Cuenta.


### 6. Pedido–LineaPedido–Producto (Clase de asociación)

- LíneaPedido vincula Pedido y Producto, incluyendo cantidad, precio en el momento.
- Restricción: Solo si producto stock > 0.

## Tabla de Roles y Cardinalidades

| Relación | Clase Origen | Rol Origen | Card.Origen | Clase Destino | Rol Destino | Card.Destino |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| Composición | Cliente | propietario | 1 | Cuenta |  | 0..* |
| Asociación | Cuenta |  | 1 | TarjetaCredito |  | 1 |
| Asociación | Cuenta |  | 1 | Pedido |  | 0..* |
| Generalización | Pedido |  | 1 | PedidoSimple |  | 0..* |
| Generalización | Pedido |  | 1 | PedidoCompuesto |  | 0..* |
| Composición (Composite) | PedidoCompuesto | contiene | 0..1 | Pedido | contenido | 2..* |
| Asociación | PedidoSimple | paga | 0..* | Cuenta |  | 1 |
| Agregación | Pedido | posee | 1 | LineaPedido |  | 0..* |
| Asociación | LineaPedido | pertenece | 1 | Producto |  | 1 |


## Decisiones de Diseño

### ¿Por qué PedidoCompuesto?

Permite estructurar jerarquías de pedidos, esencial para pedidos agrupados, envíos parciales, o gestión avanzada.

### ¿Por qué clase LineaPedido?

Encapsula producto, cantidad y precio en el momento del pedido. Es una clase de asociación entre Producto y Pedido.

### ¿Restricciones en compras y stock?

Obliga que el pedido solo se realice si:

- La cuenta tiene saldo suficiente.
- El producto pedido tiene stock disponible.
- El cliente solo paga con cuentas propias.


### ¿Singleton en ProcesadorCobros?

Solo debe haber un encargado de cobrar pedidos, controlando las transacciones y el pase de estado.

## Diagrama de Clases PlantUML

```plantuml
@startuml

class Cliente{
    - nombre: String
    - direccionEnvio : String
    - telefono : Int
    - correo : String
}

class Producto{ 
    - codigo : Int
    - stock : Int
    - nombre : String
    - descripcion : String
    - pvp : Double
}

class Cuenta{
    - numero : Int
    - saldo : Double
    + recargarSaldo() : void
    + comprar() : void
}

class TarjetaCredito{
    - numero : Int
    - saldo : Double
    - fechaVencimiento : Date
}

class LineaPedido{
    - cantidad : Int
    - precioProducto : Double
}

class Pedido{
    - estado : estadoPedido
    - costo : Int
    - numero : Int
}

class PedidoSimple{
    + agregarProducto(producto : Producto,cantidad : Int) : void
    + calcularCosto() : Double
}

class PedidoCompuesto{
    + agregarPedido(pedido : Pedido) : void
    + calcularCostoTotal() : Double
}

enum estadoPedido{
    PENDIENTE
    COBRADO
    DISTRIBUCION
    CONFIRMADO
}

class ProcesadorCobros <<singleton>>{
    realizarRevision(estado : estadoPedido) : void 
}

Cliente "1" *-- "0..*" Cuenta : propietario
Cuenta "1" -- "0..*" Pedido : realiza {restriccion : La cuenta sólo puede realizar un pedido si CUENTA.saldo > PEDIDO.costo y la cuenta con la que se paga el pedido es de su propiedad.}
Cuenta "1" -- "1" TarjetaCredito : usa
Pedido <|-- PedidoSimple 
Pedido <|-- PedidoCompuesto
PedidoSimple "0..*" -- "1" Cuenta : paga
PedidoCompuesto "0..1" *-- "2..*" Pedido : contiene
Pedido "1" -- "0..*" LineaPedido : posee
LineaPedido "1" -- "1" Producto : pertenece {restricción: Sólo se puede pedir un producto si PRODUCTO.stock > 0}

note right of PedidoSimple
{restricción : Máximo 20 productos por Pedido}
end note

@enduml
```


## Conceptos Clave de UML Aplicados

1. **Composite**: PedidoCompuesto puede contener otros pedidos (simples o compuestos).
2. **Singleton**: ProcesadorCobros único para toda la gestión de cobros.
3. **Clase de Asociación**: LineaPedido une Pedido y Producto incluyendo atributos extra.
4. **Restricciones de negocio**: En pedidos, en stock, en saldos.
5. **Generalización**: Pedido es la clase base común.
6. **Encapsulación**: Todos los atributos privados, métodos públicos.