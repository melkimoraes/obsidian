

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

        ```
        - A função `isPalindrome` foi adicionada à classe `String`.
		- Agora, qualquer `String` pode chamar essa função diretamente.

- **Coroutines (Introdução)**
    - Conhecer o básico sobre programação assíncrona:
        
        ```kotlin
        import kotlinx.coroutines.*
        fun main() = runBlocking {
            launch {
                delay(1000L)
                println("Kotlin Coroutines!")
            }
        }
        ```
        
