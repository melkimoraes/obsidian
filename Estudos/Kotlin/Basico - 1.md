#### **1. Conceitos Iniciais de Kotlin (1-2 semanas)**

- **Configuração do Ambiente**
    - Instalar o IntelliJ IDEA.
    - Criar um projeto Kotlin simples.
    - Executar o primeiro programa: `println("Hello, Kotlin!")`.
- **Sintaxe Básica**
    - Declaração de variáveis:
        
        ```kotlin
        val nome = "João" // Imutável (como `final` em Java)
        var idade = 30    // Mutável
        ```
        
    - Tipos básicos (`Int`, `String`, `Double`, etc.) e inferência de tipo.
- **Funções**
    - Declaração de funções:
        
        ```kotlin
        fun soma(a: Int, b: Int): Int = a + b
        ```
        
    - Funções de uma linha (expressões).
- **Estruturas de Controle**
    - Uso do `if` como expressão:
        
        ```kotlin
        val maior = if (a > b) a else b
        ```
        
    - `when` como substituto do `switch`:
        
        ```kotlin
        when (dia) {
            1 -> println("Segunda-feira")
            2 -> println("Terça-feira")
            else -> println("Outro dia")
        }
        ```

Tipos
	- String, Char, Double, Boolean

Collections
- Lists - lista ordenada
```kotlin
//pra criar lista listOf() porem o listOf é só pra readonly! entao tem o MutableList
fun main() { 
    // Read only list
    val readOnlyShapes = listOf("triangle", "square", "circle")
    println(readOnlyShapes)
    // [triangle, square, circle]

    // Mutable list with explicit type declaration
    val shapes: MutableList<String> = mutableListOf("triangle", "square", "circle")
    println(shapes)
    // [triangle, square, circle]
}
```

acessando itens da lista:
```kotlin
val readOnlyShapes = listOf("triangle", "square", "circle")
println("The first item in the list is: ${readOnlyShapes[0]}")
// The first item in the list is: triangle

val readOnlyShapes = listOf("triangle", "square", "circle")
println("The first item in the list is: ${readOnlyShapes.first()}")
// The first item in the list is: triangle

val readOnlyShapes = listOf("triangle", "square", "circle")
println("This list has ${readOnlyShapes.count()} items")
// This list has 3 items

val readOnlyShapes = listOf("triangle", "square", "circle")
println("circle" in readOnlyShapes)
// true

val shapes: MutableList<String> = mutableListOf("triangle", "square", "circle")
// Add "pentagon" to the list
shapes.add("pentagon") 
println(shapes)  
// [triangle, square, circle, pentagon]

// Remove the first "pentagon" from the list
shapes.remove("pentagon") 
println(shapes)  
// [triangle, square, circle]
```
- Sets - lista desordenada → métodos iniciais iguais do List!
```kotlin
// Read-only set
val readOnlyFruit = setOf("apple", "banana", "cherry", "cherry")
// Mutable set with explicit type declaration
val fruit: MutableSet<String> = mutableSetOf("apple", "banana", "cherry", "cherry")

println(readOnlyFruit)
// [apple, banana, cherry]
```
- Maps - chave valor