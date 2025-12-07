# Solución: Ejercicio 6 – Sistema de Gestión de Biblioteca

## Análisis del Problema

### Identificación de Clases

1. **Autor**
    - Atributos: nombre, fechaNacimiento
    - Método derivado: calcularEdad()
2. **Libro**
    - Atributos: titulo, tipo (enum libroTipo), estado (enum libroEstado), isbn
    - Método: cambiarEstado(nuevoEstado)
3. **Prestamo**
    - Atributo: fechaPrestamo
    - Método derivado: calcularFechaLimite()
4. **Multa**
    - Atributos: fechaInicio, diasMulta
5. **Socio**
    - Atributos: nombre, dni, fechaInscripcion, librosPrestados (o método), multaActiva (o método)
    - Métodos: prestarLibro(), devolverLibro(), calcularTiempoRestanteMulta()
6. **Enum libroTipo**
    - NOVELA, POESIA, CUENTO
7. **Enum libroEstado**
    - DISPONIBLE, PRESTADO, RETRASADO


## Análisis de Relaciones

1. **Libro–Autor (N:M por narrativa, aquí 1 a 1..*)**
    - Un libro tiene uno o más autores (coautoría posible).
    - Asociación: Libro "1" -- "1..*" Autor
2. **Socio–Préstamo–Libro (N:M)**
    - Un socio puede tener 0..3 préstamos activos.
    - Cada préstamo vincula un libro con un socio.
    - Asociación: Socio "1" -- "0..3" Prestamo, Prestamo "1" -- "1" Libro
3. **Socio–Multa (1:N)**
    - Un socio puede tener varias multas, pero solo una activa.
    - Asociación: Socio "1" -- "0..*" Multa


## Tabla de Roles y Cardinalidades

| Relación | Clase Origen | Rol Origen | Card.Origen | Clase Destino | Rol Destino | Card.Destino |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| Asociación | Libro | escrito por | 1 | Autor |  | 1..* |
| Asociación | Socio | tiene | 1 | Prestamo |  | 0..3 |
| Asociación | Prestamo | sobre | 1 | Libro |  | 1 |
| Asociación | Socio |  | 1 | Multa |  | 0..* |



## Decisiones de Diseño

1. **Enumeraciones**
    - Se usan enums para limitar los valores posibles de tipo de libro y estado.
2. **Clase Prestamo**
    - Relaciona un socio y un libro, añadiendo cuándo empezó el préstamo y cuándo debe terminar.
3. **Clase Multa**
    - Permite gestionar sanciones y restricciones, desacoplando la lógica de multa del socio.
4. **Restricciones y reglas de negocio**
    - Límite de 3 préstamos activos gestionado por cardinalidad y el método prestarLibro.
    - Un socio solo puede prestar si no tiene multa activa y no supera máximo de libros.
    - Cambio de estado del libro y cálculo de multas tras devolución.

## Diagrama de Clases PlantUML

```plantuml
@startuml

enum libroTipo{
    NOVELA
    POESIA
    CUENTO
}

enum libroEstado{
    DISPONIBLE
    PRESTADO
    RETRASADO
}

class Autor{
    - nombre : String
    - fechaNacimiento : Date
    + calcularEdad() : Int {derived}

}

class Libro{
    - titulo : String
    - tipo : libroTipo
    - estado : libroEstado
    - isbn : String
    + cambiarEstado(nuevoEstado : libroEstado) : void

}


class Prestamo{
    - fechaPrestamo : Date
    + calcularFechaLimite() : Date {derived}
}

class Multa {
    - fechaInicio : Date
    - diasMulta : Int
}

class Socio{ 
    - nombre : String
    - dni : String
    - fechaInscripcion : Date
    - librosPrestados() : Int
    - multaActiva() : Boolean
    + devolverLibro(prestamo : Prestamo) : void
    + calcularTiempoRestanteMulta(fechaInicio : Multa) : Int {derived}
    + prestarLibro(libro : Libro) : void {restriccion : Un socio sólo puede encargar un libro si SOCIO.multaActiva = False y como máximo 3}
}

Libro "1..*" -- "1" Autor : escrito por

Socio "1" -- "0..3" Prestamo : tiene

Prestamo "1" -- "1" Libro : sobre


note right of Prestamo
El método calcularFechaLimite() calcula la fecha límite de la siguiente forma: fechaLimite = fechaPrestamo + 30
end note
note right of Socio
- El método calcularTiempoRestanteMulta() calcula el tiempo restante de la multa a partir de MULTA.fechaInicio
- Si el valor de retorno de calcularTiempoRestanteMulta(), multaActiva pasará a valer True hasta que el tiempo restante de la multa sea 0.
@enduml

```
## Código en Kotlin
```kotlin
enum class LibroTipo {
    NOVELA,
    POESIA,
    CUENTO
}

enum class LibroEstado {
    DISPONIBLE,
    PRESTADO,
    RETRASADO
}

class Autor(
    private val nombre: String,
    private val fechaNacimiento: java.util.Date
) {

    private val libros: MutableList<Libro> = mutableListOf()

    fun calcularEdad(): Int {
        return 0
    }
}

class Libro(
    private val titulo: String,
    private val tipo: LibroTipo,
    private var estado: LibroEstado,
    private val isbn: String
    private var autor: Autor
) {

    fun cambiarEstado(nuevoEstado: LibroEstado) {
    }
}

class Prestamo(
    private val fechaPrestamo: java.util.Date,
    private val libro: Libro,
    private val socio: Socio
) {
    fun calcularFechaLimite(): java.util.Date {
        return java.util.Date()
    }
}

class Multa(
    private val fechaInicio: java.util.Date,
    private val diasMulta: Int
)

class Socio(
    private val nombre: String,
    private val dni: String,
    private val fechaInscripcion: java.util.Date
) {
    private val prestamos: MutableList<Prestamo> = mutableListOf()

    private fun librosPrestados(): Int {
        return 0
    }

    private fun multaActiva(): Boolean {
        return false
    }

    fun devolverLibro(prestamo: Prestamo) {
    }

    fun calcularTiempoRestanteMulta(multa: Multa): Int {
        return 0
    }

    fun prestarLibro(libro: Libro) {
    }
}
```