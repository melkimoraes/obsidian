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
        