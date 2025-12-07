# Solución: Ejercicio 4 – Sistema de Gestión Empresarial

## Análisis del Problema

### Identificación de Clases

1. **Persona** (abstracta)
    - Datos comunes: nombre, fechaNacimiento.
    - Método derivado: calcularEdad().
2. **Empleado** (abstracta)
    - Hereda de Persona, añade sueldoBruto.
    - Método derivado: calcularSueldo().
    - **EmpleadoNormal** (subclase de Empleado)
    - **EmpleadoResponsable** (subclase de Empleado)
        - Atributo: tipo (SUPERVISOR/JEFE/GERENTE).
3. **Cliente** (hereda de Persona)
    - Añade númeroTelefono.
4. **Empresa**
    - CIF, nombre corporativo, dirección fiscal.
5. **Enum tipoEmpleado**
    - SUPERVISOR, JEFE, GERENTE.


## Análisis de Relaciones

### 1. Herencia

- Persona es supertipo para Empleado y Cliente.
- Empleado es supertipo para EmpleadoNormal y EmpleadoResponsable.


### 2. Asociación reflexiva (jerarquía empleados)

- Un EmpleadoResponsable se encarga de 0..* Empleados.
    - Refleja jerarquía interna y cadena de mando.


### 3. Asociación Cliente–Empresa

- Un Cliente pertenece a 1..* Empresas.
- Una Empresa tiene 0..* Clientes.
- Restricción: todo Cliente debe estar vinculado, como mínimo, a una Empresa.


## Tabla de Roles y Cardinalidades

| Relación | Clase Origen | Rol Origen | Card.Origen | Clase Destino | Rol Destino | Card.Destino |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| Herencia | Persona |  |  | Cliente |  |  |
| Herencia | Persona |  |  | Empleado |  |  |
| Herencia | Empleado |  |  | EmpleadoNormal |  |  |
| Herencia | Empleado |  |  | EmpleadoResponsable |  |  |
| Reflexiva responsable | EmpleadoResponsable | se encarga | 1 | Empleado | subordinado | 0..* |
| Asociación comercial | Cliente | pertenece | 0..* | Empresa |  | 1..* |



## Decisiones de Diseño

### ¿Por qué Persona es abstracta?

Permite factorizar los datos comunes y el método derivado de edad, evitando duplicación en Cliente y Empleado.

### ¿Por qué EmpleadoResponsable y EmpleadoNormal?

Refleja la diferenciación jerárquica y permite modelar las cadenas de mando.

### ¿Por qué reflexiva sólo en EmpleadoResponsable?

Según el enunciado, sólo los responsables tienen subordinados. Para escenarios con jerarquías más complejas, se usaría la asociación reflexiva directa en Empleado.

### ¿Por qué Cliente pertenece a al menos una Empresa?

La empresa no trabaja con particulares, así que todo cliente debe estar asociado a una o más empresas.


## Diagrama de Clases PlantUML

```plantuml
@startuml

enum tipoEmpleado{
    SUPERVISOR
    JEFE
    GERENTE
}

abstract class Persona{
    - nombre : String
    - fechaNacimiento : Date
    + calcularEdad() : Int {derived}
}

abstract class Empleado{
    - sueldoBruto : Double
    + calcularSueldo() : Double {derived}
}

class Cliente{
    - numeroTelefono : String
}

class Empresa{
    - cif : String
    - nombreCorporativo : String
    - direccionFiscal : String
}

class EmpleadoNormal{
}

class EmpleadoResponsable{
    - tipo : tipoEmpleado
}

Cliente "0..*" -- "1..*" Empresa : pertenece

Persona <|-- Cliente
Persona <|-- Empleado

Empleado <|-- EmpleadoNormal
Empleado <|-- EmpleadoResponsable

EmpleadoResponsable "1" -- "0..*" Empleado : se encarga

note right of Persona
El método calcularEdad() calcula la edad a partir de fechaNacimiento.
end note

note right of Empleado
El método calcularSueldo() utiliza la fórmula : sueldoNeto = sueldoBruto * 0.78 aunque puede depender del rango.
end note
@enduml
```

## Código Kotlin

```kotlin
enum class TipoEmpleado {
    SUPERVISOR,
    JEFE,
    GERENTE
}

abstract class Persona(
    private val nombre: String,
    private val fechaNacimiento: java.util.Date
) {
    abstract fun calcularEdad(): Int
}

abstract class Empleado(
    private val sueldoBruto: Double,
) : Persona(
    nombre = "",
    fechaNacimiento = java.util.Date()
) {
    abstract fun calcularSueldo(): Double
}

class Cliente(
    private val numeroTelefono: String,
    nombre: String,
    fechaNacimiento: java.util.Date
) : Persona(nombre, fechaNacimiento) {

    private val empresas: MutableList<Empresa> = mutableListOf()
}

class Empresa(
    private val cif: String,
    private val nombreCorporativo: String,
    private val direccionFiscal: String
) {
    private val clientes: MutableList<Cliente> = mutableListOf()
}

class EmpleadoNormal(
    sueldoBruto: Double,
    nombre: String,
    fechaNacimiento: java.util.Date
) : Empleado(sueldoBruto) {
}

class EmpleadoResponsable(
    private val tipo: TipoEmpleado,
    sueldoBruto: Double,
    nombre: String,
    fechaNacimiento: java.util.Date
) : Empleado(sueldoBruto) {
    private val empleadosACargo: MutableList<Empleado> = mutableListOf()
}






```