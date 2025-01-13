#### **2. Orientação a Objetos no Kotlin (2-3 semanas)**

- **Classes e Objetos**
    - Criar classes:
        
        ```kotlin
        class Pessoa(val nome: String, var idade: Int)
        ```
        
    - Instanciar objetos:
        
        ```kotlin
        val pessoa = Pessoa("João", 25)
        ```
        
- **Propriedades**
    - Getters e setters automáticos.
    - Uso de `val` e `var` em classes.
⭐minhas anotações
- Use `val` para propriedades imutáveis (apenas `getter`).
- Use `var` para propriedades mutáveis (tem `getter` e `setter`).
- Getters e setters padrão são gerados automaticamente, mas você pode customizá-los se necessário.
- Bloco `init` pode ser usado para inicializações personalizadas.
```kotlin
class Pessoa(nome: String, idade: Int) {
    val saudacao: String

    init {
        saudacao = "Olá, meu nome é $nome e tenho $idade anos."
    }
}

fun main() {
    val pessoa = Pessoa("João", 25)
    println(pessoa.saudacao) // Saída: Olá, meu nome é João e tenho 25 anos.
}

```
 - Get pra setar uma variavel como se fosse um campo calculado:
```kotlin
class Retangulo(val largura: Int, val altura: Int) {
    val area: Int
        get() = largura * altura
}

fun main() {
    val retangulo = Retangulo(5, 10)
    println(retangulo.area) // Saída: 50
}

```
- Data Class: 
> As `data class` são usadas para classes que apenas armazenam dados. O Kotlin automaticamente gera métodos como `toString()`, `equals()`, e `hashCode()`.
```kotlin
// Data class para representar dados simples
data class Usuario(val nome: String, val email: String)

fun main() {
    val usuario1 = Usuario("Alice", "alice@email.com")
    val usuario2 = Usuario("Bob", "bob@email.com")

    println(usuario1) // Saída: Usuario(nome=Alice, email=alice@email.com)

    // Comparação de objetos
    val usuario3 = Usuario("Alice", "alice@email.com")
    println(usuario1 == usuario3) // Saída: true (compara os valores)
}

```


- **Herança**
    - Modificador `open` para permitir herança:
        
        ```kotlin
        open class Animal(val nome: String)
        class Cachorro(nome: String): Animal(nome)
        ```
                
- **Singletons com `object`**
    - Criar singletons:
        
        ```kotlin
        object Configuracao {
            val url = "http://meusite.com"
        }
        ```

 - Objetos Anônimos

Se você precisa de uma instância rápida sem criar uma classe, use `object` diretamente.
```kotlin
fun main() {
    val objetoAnonimo = object {
        val id = 123
        fun mostrarId() {
            println("ID do objeto: $id")
        }
    }

    objetoAnonimo.mostrarId() // Saída: ID do objeto: 123
}

```
