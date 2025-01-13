#### **2. Orientação a Objetos no Kotlin (2-3 semanas)**

- **Classes e Objetos**
    - Criar classes:
        
        ```kotlin
        class Pessoa(val nome: String, var idade: Int)
        ```
     → no kotlin nao precisa declarar as variaveis se elas ja estiverem no construtor, claro que se nao tiver no construtor ai sim declarar no corpo da classe
     
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
        ⭐ vou dar mais um exemplo ali embaixo.
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

#### **Singleton (`object`)**

Os singletons são usados quando você precisa de uma única instância global em todo o seu aplicativo. Um caso comum é para gerenciar configurações ou fornecer serviços centralizados, como um logger ou uma conexão de banco de dados.

```kotlin
// Singleton para gerenciar configurações globais
object Configuracao {
    var modoEscuro: Boolean = false

    fun exibirConfiguracoes() {
        println("Modo Escuro: $modoEscuro")
    }
}

fun main() {
    Configuracao.modoEscuro = true
    Configuracao.exibirConfiguracoes() // Saída: Modo Escuro: true
}

```



---

#### **Objeto Anônimo**

Objetos anônimos são úteis quando você precisa de uma instância única e rápida de uma classe, geralmente para implementações de interfaces ou classes abstratas "na hora". Um exemplo clássico é o uso em listeners ou callbacks.

```kotlin
// Implementação rápida de uma interface
interface ClickListener {
    fun onClick()
}

fun configurarBotao(listener: ClickListener) {
    println("Botão configurado.")
    listener.onClick()
}

fun main() {
    configurarBotao(object : ClickListener {
        override fun onClick() {
            println("Botão clicado!")
        }
    })
    // Saída:
    // Botão configurado.
    // Botão clicado!
}

```

---

### **Modificadores de Acesso no Kotlin**

Os modificadores de acesso controlam a visibilidade de classes, propriedades, funções, etc.

|Modificador|Descrição|
|---|---|
|`public`|Visível para todos (padrão, mesmo que não seja explicitado).|
|`private`|Visível apenas dentro do escopo em que foi declarado (classe ou arquivo).|
|`protected`|Igual ao `private`, mas permite acesso em classes derivadas.|
|`internal`|Visível apenas dentro do mesmo módulo (um módulo é geralmente um projeto ou biblioteca).|


---

### **Interface**

- As interfaces definem contratos que as classes devem implementar.
- Diferente de classes abstratas, uma interface não pode armazenar estado, mas pode conter **funções com implementação padrão**.

```kotlin
interface Veiculo {
    val tipo: String
    fun mover() // Método abstrato
    fun parar() {
        println("O veículo parou.") // Método com implementação padrão
    }
}

class Carro : Veiculo {
    override val tipo: String = "Carro"
    override fun mover() {
        println("O carro está se movendo.")
    }
}

fun main() {
    val carro = Carro()
    println(carro.tipo) // Saída: Carro
    carro.mover()       // Saída: O carro está se movendo.
    carro.parar()       // Saída: O veículo parou.
}

```

---

### **Classes Abstratas**

- As classes abstratas podem conter propriedades e métodos com ou sem implementação.
- Diferente de interfaces, elas podem armazenar estado e conter um construtor.

```kotlin
abstract class Animal(val nome: String) {
    abstract fun emitirSom() // Método abstrato

    fun dormir() {
        println("$nome está dormindo.") // Método com implementação
    }
}

class Cachorro(nome: String) : Animal(nome) {
    override fun emitirSom() {
        println("$nome está latindo: Au au!")
    }
}

fun main() {
    val cachorro = Cachorro("Rex")
    cachorro.emitirSom() // Saída: Rex está latindo: Au au!
    cachorro.dormir()    // Saída: Rex está dormindo.
}

```

---

### **Comparação: Interface vs Classe Abstrata**

|Característica|Interface|Classe Abstrata|
|---|---|---|
|Construtor|Não possui|Pode possuir|
|Implementação padrão|Sim, desde que não dependa de estado|Sim|
|Estado (propriedades com valores)|Não pode ter|Pode armazenar valores|
|Herança múltipla|Sim, uma classe pode implementar várias|Não, apenas uma classe base abstrata|
