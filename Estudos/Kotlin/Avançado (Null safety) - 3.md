

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
        ```
        
- **Extensões**
    - Criar funções de extensão:
        
        ```kotlin
        fun String.espelhar(): String = this.reversed()
        println("Kotlin".espelhar()) // "niltoK"
        ```
        
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
        
