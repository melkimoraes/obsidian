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

checar se algo existe dentro da lista,map é igual:

```kotlin
val readOnlyFruit = setOf("apple", "banana", "cherry", "cherry")
println("banana" in readOnlyFruit)
// true
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
```kotlin
// Read-only map
val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
println(readOnlyJuiceMenu)
// {apple=100, kiwi=190, orange=100}

// Mutable map with explicit type declaration
val juiceMenu: MutableMap<String, Int> = mutableMapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
println(juiceMenu)
// {apple=100, kiwi=190, orange=100}
```

To access a value in a map, use the [indexed access operator](https://kotlinlang.org/docs/operator-overloading.html#indexed-access-operator) `[]` with its key:
```kotlin
// Read-only map
val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
println("The value of apple juice is: ${readOnlyJuiceMenu["apple"]}")
// The value of apple juice is: 100
```


Controle de condições:
- If → normal, só o ternario como eu vi, muda um pouco mas ok
- When → substitui o switch do java
```kotlin
val obj = "Hello"

when (obj) {
    // Checks whether obj equals to "1"
    "1" -> println("One")
    // Checks whether obj equals to "Hello"
    "Hello" -> println("Greeting")
    // Default statement
    else -> println("Unknown")     
}

// uma variavel com expressao com when:
val obj = "Hello"    

val result = when (obj) {
    // If obj equals "1", sets result to "one"
    "1" -> "One"
    // If obj equals "Hello", sets result to "Greeting"
    "Hello" -> "Greeting"
    // Sets result to "Unknown" if no previous condition is satisfied
    else -> "Unknown"
}
```
- For
```kotlin
for (number in 1..5) { 
    // number is the iterator and 1..5 is the range
    print(number)
}
// 12345

val cakes = listOf("carrot", "cheese", "chocolate")

for (cake in cakes) {
    println("Yummy, it's a $cake cake!")
}
// Yummy, it's a carrot cake!
// Yummy, it's a cheese cake!
// Yummy, it's a chocolate cake!
```
- While
```kotlin
var cakesEaten = 0
while (cakesEaten < 3) {
    println("Eat a cake")
    cakesEaten++
}
// Eat a cake
// Eat a cake
// Eat a cake

var cakesEaten = 0
var cakesBaked = 0
while (cakesEaten < 3) {
    println("Eat a cake")
    cakesEaten++
}
do {
    println("Bake a cake")
    cakesBaked++
} while (cakesBaked < cakesEaten)
// Eat a cake
// Eat a cake
// Eat a cake
// Bake a cake
// Bake a cake
// Bake a cake
```


Lambda expressions → de vez de fazer uma function usamos uma função anonima:
```kotlin
fun main() {
    val upperCaseString = { text: String -> text.uppercase() }
    println(upperCaseString("hello"))
    // HELLO
}
```