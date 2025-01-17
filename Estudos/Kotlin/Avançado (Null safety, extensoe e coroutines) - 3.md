

- **Null Safety**
    - Trabalhar com valores nulos: (se nao tiver esse “?” vai dar erro como no java caso de um nullpointer)
        
        ```kotlin
        var nome: String? = null
        println(nome?.length) // Safe call -> aqui nao vai dar nullpointer!

		//outro exemplo:
		fun lengthString(maybeString: String?): Int? = maybeString?.length

		fun main() { 
		    val nullString: String? = null
		    println(lengthString(nullString))
		    // null
		}

		//também tem a opção de usar o elvis operator no caso de null:
		fun main() {
		    val nullString: String? = null
		    println(nullString?.length ?: 0)
		    // 0
		}
        ```
		
- **Extensões**
    - Criar funções de extensão:
         São funções adicionadas a uma classe, mas que não alteram a classe original. Elas utilizam o receptor (`this`) para acessar as propriedades e métodos da classe onde são declaradas.
        ```kotlin
        fun String.espelhar(): String = this.reversed()
        println("Kotlin".espelhar()) // "niltoK"

		// OUTRO EXEMPLO - Função de extensão para a classe String
		fun String.isPalindrome(): Boolean {
		    val normalized = this.replace(" ", "").lowercase()
		    return normalized == normalized.reversed()
		}
		
		fun main() {
		    val text = "A man a plan a canal Panama"
		    println(text.isPalindrome()) // true
		}

		// OUTRO EXEMPLO - Propriedade de extensão para a classe List<Int>
		val List<Int>.sumEven: Int
		    get() = this.filter { it % 2 == 0 }.sum()
		
		fun main() {
		    val numbers = listOf(1, 2, 3, 4, 5, 6)
		    println(numbers.sumEven) // 12 (soma de 2 + 4 + 6)
		}


        ```
        - A função `isPalindrome` foi adicionada à classe `String`.
		- Agora, qualquer `String` pode chamar essa função diretamente.

	- **Extensões não modificam a classe original.** Elas apenas adicionam funcionalidades no contexto onde são usadas.
	- **Não podem acessar membros privados ou protegidos da classe original.**
	- **Podem ser usadas em tipos nulos.** Você pode tratar `null` diretamente na extensão.

- **Coroutines (Introdução)**
    - Conhecer o básico sobre programação assíncrona:

	###### **Conceitos Básicos**

1. **Coroutines são leves:** Diferente de threads, elas têm baixo custo para serem criadas e executadas. Você pode ter milhares de coroutines rodando em uma aplicação sem o impacto de performance que threads poderiam causar.
    
2. **Suspend Function:** Uma função marcada com `suspend` pode ser pausada e retomada sem bloquear a thread. Isso é o que torna as coroutines poderosas.
    
3. **Builders de Coroutine:** Funções como `launch` e `async` são usadas para criar coroutines. Elas definem o "contexto" onde a coroutine será executada.


    ```kotlin
        import kotlinx.coroutines.*

fun main() = runBlocking {
    println("Início: ${Thread.currentThread().name}")

    launch {
        delay(1000L) // Pausa a execução por 1 segundo
        println("Coroutine executada: ${Thread.currentThread().name}")
    }

    println("Fim: ${Thread.currentThread().name}")
}        
```

Saida disso : 
Início: main 
Fim: main 
Coroutine executada: main

###### **Explicação:**

1. A função `runBlocking` cria uma Coroutine que bloqueia a thread principal até que todo o código dentro dela seja executado.
2. A função `launch` inicia uma nova Coroutine que é executada de forma assíncrona.
3. `delay(1000L)` pausa a Coroutine atual por 1 segundo sem bloquear a thread.



##### **Builders de Coroutines**

### 1. **`launch`**

Inicia uma Coroutine e **não retorna um resultado**.

kotlin

CopiarEditar

`launch {     println("Tarefa com launch") }`

### 2. **`async`**

Inicia uma Coroutine e **retorna um resultado** através de um `Deferred`.

kotlin

CopiarEditar

`val result = async {     delay(1000L)     "Resultado da coroutine" } println(result.await()) // "Resultado da coroutine"`

---

## **Scopes de Coroutine**

Coroutines devem ser executadas dentro de um **Coroutine Scope**, que gerencia sua execução. Exemplos de scopes:

1. **`GlobalScope`**
    
    - Coroutines vivem enquanto o aplicativo estiver rodando.
    - Não recomendado para produção, pois não permite cancelamento fácil.
    
    kotlin
    
    CopiarEditar
    
    `GlobalScope.launch {     println("Executando no GlobalScope") }`
    
2. **`runBlocking`**
    
    - Bloqueia a thread principal até que a execução termine.
    - Usado para testes ou exemplos simples.
    
    kotlin
    
    CopiarEditar
    
    `runBlocking {     println("Bloqueando a thread principal") }`
    
3. **`CoroutineScope`**
    
    - Escopo que pode ser controlado e cancelado, ideal para produção.
    
    kotlin
    
    CopiarEditar
    
    `class MyActivity : CoroutineScope {     private val job = Job()     override val coroutineContext = Dispatchers.Main + job      fun fetchData() {         launch {             val data = async(Dispatchers.IO) { getDataFromNetwork() }             println(data.await())         }     }      fun onDestroy() {         job.cancel() // Cancela todas as coroutines ao destruir o escopo     } }`
    

---

## **Dispatchers**

Coroutines podem ser executadas em diferentes **dispatchers**, que determinam em qual thread o código será executado:

1. **`Dispatchers.Main`**
    
    - Usado para atualizar a interface do usuário (somente em Android).
2. **`Dispatchers.IO`**
    
    - Otimizado para operações de E/S, como acessar arquivos ou fazer chamadas de rede.
3. **`Dispatchers.Default`**
    
    - Usado para tarefas pesadas, como cálculos.
4. **`Dispatchers.Unconfined`**
    
    - Não está confinado a nenhuma thread específica.

**Exemplo:**

kotlin

CopiarEditar

`launch(Dispatchers.IO) {     val result = getDataFromDatabase()     withContext(Dispatchers.Main) {         updateUI(result)     } }`

---

## **Cancelando Coroutines**

Você pode cancelar coroutines para liberar recursos. Para isso, use um `Job`.

kotlin

CopiarEditar

`val job = launch {     repeat(1000) { i ->         println("Executando $i...")         delay(500L)     } }  delay(2000L) // Deixa a coroutine rodar por 2 segundos job.cancel() // Cancela a coroutine println("Coroutine cancelada")`

---

## **Funções Comuns**

- **`delay(timeMillis: Long)`**
    - Pausa a Coroutine sem bloquear a thread.
- **`withContext(context: CoroutineContext)`**
    - Muda o contexto da Coroutine de forma temporária.

**Exemplo:**

kotlin

CopiarEditar

`val result = withContext(Dispatchers.IO) {     // Código pesado aqui     "Resultado" }`

---

Coroutines são uma ferramenta poderosa para escrever código assíncrono e escalável de forma simples e intuitiva. Se precisar de ajuda em casos mais específicos (como em Android ou integração com APIs), posso detalhar!