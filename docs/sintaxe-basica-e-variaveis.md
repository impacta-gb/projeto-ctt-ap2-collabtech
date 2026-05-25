# Sintaxe Básica e Variáveis

Go possui uma sintaxe limpa e minimalista, intencionalmente projetada para ser fácil de ler e escrever. Nesta seção, veremos os fundamentos da linguagem: como declarar variáveis, os tipos de dados disponíveis e as regras básicas de estrutura de um programa Go.

---

## Estrutura de um Arquivo Go

Todo arquivo Go pertence a um **pacote** (`package`). O pacote especial `main` define o ponto de entrada de um programa executável.

```go
package main  // Declaração do pacote

import "fmt"  // Importação de pacotes

// Função principal — ponto de entrada do programa
func main() {
    fmt.Println("Go é incrível!")
}
```

### Regras importantes

- O nome do pacote vem sempre na **primeira linha**
- As importações ficam logo após a declaração do pacote
- O Go usa **chaves `{}`** para delimitar blocos de código
- **Não há ponto e vírgula** ao final das linhas (o compilador os insere automaticamente)

> [!WARNING]
> Em Go, **importações não utilizadas causam erro de compilação**. Se você importar um pacote e não usá-lo, o programa não compilará.

---

## Comentários

```go
// Comentário de linha única

/*
   Comentário
   de múltiplas linhas
*/

// godoc: comentários acima de funções e tipos
// são usados para gerar documentação automática
func Saudar(nome string) string {
    return "Olá, " + nome
}
```

---

## Declaração de Variáveis

Go oferece **três formas principais** de declarar variáveis:

### 1. Declaração explícita com `var`

```go
var nome string = "Gopher"
var idade int = 5
var ativo bool = true
```

### 2. Declaração com inferência de tipo

```go
var nome = "Gopher"   // Go infere que é string
var contador = 10     // Go infere que é int
```

### 3. Declaração curta com `:=` (mais comum)

```go
nome := "Gopher"
idade := 5
preco := 29.99
```

> [!NOTE]
> O operador `:=` só pode ser usado **dentro de funções**. Para variáveis globais (nível de pacote), use `var`.

### Declaração múltipla

```go
// Múltiplas variáveis na mesma linha
var x, y, z int = 1, 2, 3

// Bloco var
var (
    nome    string  = "Go"
    versao  int     = 22
    estavel bool    = true
)
```

---

## Tipos de Dados Primitivos

### Tipos Numéricos

| Tipo | Tamanho | Faixa de valores |
|------|---------|-----------------|
| `int8` | 8 bits | -128 a 127 |
| `int16` | 16 bits | -32.768 a 32.767 |
| `int32` | 32 bits | -2.147.483.648 a 2.147.483.647 |
| `int64` | 64 bits | -9.2×10¹⁸ a 9.2×10¹⁸ |
| `int` | 32 ou 64 bits (depende da plataforma) | — |
| `uint` | sem sinal | 0 a 4.294.967.295 (32-bit) |
| `float32` | 32 bits | precisão simples |
| `float64` | 64 bits | precisão dupla |
| `complex64` | 64 bits | números complexos |
| `complex128` | 128 bits | números complexos |

```go
var inteiro int = 42
var flutuante float64 = 3.14159
var complexo complex128 = 3 + 4i
```

### Tipo String

Strings em Go são sequências de bytes imutáveis codificadas em UTF-8.

```go
var saudacao string = "Olá, Mundo!"
nome := "Gopher"

// Concatenação
nomeCompleto := "Go " + "Lang"

// String multilinha com backtick (raw string)
texto := `Esta é uma string
que ocupa múltiplas
linhas sem escapes`

// Comprimento em bytes
fmt.Println(len("Go")) // 2
```

### Tipo Boolean

```go
var ativo bool = true
var inativo bool = false

resultado := 10 > 5  // true
```

### Tipo Rune e Byte

```go
var letra rune = 'A'  // alias para int32, representa um caractere Unicode
var b byte = 'z'      // alias para uint8
```

---

## Zero Values (Valores Padrão)

Em Go, toda variável declarada sem inicialização recebe um **valor zero** automaticamente:

| Tipo | Zero Value |
|------|-----------|
| `int`, `float64` | `0` |
| `bool` | `false` |
| `string` | `""` (string vazia) |
| ponteiros, slices, maps, channels, funções | `nil` |

```go
var numero int     // 0
var texto string   // ""
var ativo bool     // false

fmt.Println(numero, texto, ativo) // 0  false
```

---

## Constantes

Constantes são valores imutáveis definidos em tempo de compilação.

```go
const Pi = 3.14159
const NomeApp = "MeuApp"
const MaxRetries = 3

// Bloco de constantes
const (
    StatusOK    = 200
    StatusNotFound = 404
    StatusError    = 500
)
```

### `iota` — gerador de constantes sequenciais

```go
const (
    Domingo = iota  // 0
    Segunda         // 1
    Terça           // 2
    Quarta          // 3
    Quinta          // 4
    Sexta           // 5
    Sábado          // 6
)

fmt.Println(Segunda) // 1
fmt.Println(Sexta)   // 5
```

> [!TIP]
> O `iota` é muito útil para criar enumerações em Go, já que a linguagem não possui uma palavra-chave `enum` nativa.

---

## Conversão de Tipos

Go é **estritamente tipado** — não há conversões implícitas. Toda conversão deve ser explícita:

```go
var inteiro int = 42
var flutuante float64 = float64(inteiro)  // int → float64
var intNovo int = int(flutuante)          // float64 → int

// String ↔ int com o pacote strconv
import "strconv"

numero := 123
texto := strconv.Itoa(numero)         // int → string: "123"
valor, err := strconv.Atoi("456")     // string → int: 456
```

> [!WARNING]
> Converter `float64` para `int` **trunca** o valor decimal, sem arredondar:
> ```go
> int(3.99) // resultado: 3, não 4
> ```

---

## Entrada e Saída Básica

```go
package main

import "fmt"

func main() {
    // Saída
    fmt.Println("Hello!")           // com quebra de linha
    fmt.Print("Sem quebra ")        // sem quebra de linha
    fmt.Printf("Nome: %s\n", "Go") // formatado

    // Entrada
    var nome string
    fmt.Print("Digite seu nome: ")
    fmt.Scan(&nome)
    fmt.Printf("Olá, %s!\n", nome)
}
```

### Verbos de formatação (`fmt.Printf`)

| Verbo | Descrição | Exemplo |
|-------|-----------|---------|
| `%v` | Valor padrão | `fmt.Printf("%v", 42)` → `42` |
| `%T` | Tipo da variável | `fmt.Printf("%T", 42)` → `int` |
| `%d` | Inteiro decimal | `fmt.Printf("%d", 10)` → `10` |
| `%f` | Float | `fmt.Printf("%.2f", 3.14)` → `3.14` |
| `%s` | String | `fmt.Printf("%s", "Go")` → `Go` |
| `%t` | Boolean | `fmt.Printf("%t", true)` → `true` |
| `%b` | Binário | `fmt.Printf("%b", 5)` → `101` |
| `%x` | Hexadecimal | `fmt.Printf("%x", 255)` → `ff` |

---

## Exemplo Completo

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // Declarações variadas
    nome := "Gopher"
    idade := 5
    versao := 1.22
    ativo := true

    const MaxConexoes = 100

    // Exibindo com formatação
    fmt.Printf("Nome:    %s\n", nome)
    fmt.Printf("Idade:   %d anos\n", idade)
    fmt.Printf("Versão:  %.2f\n", versao)
    fmt.Printf("Ativo:   %t\n", ativo)
    fmt.Printf("Máx. Conexões: %d\n", MaxConexoes)

    // Conversão
    idadeTexto := strconv.Itoa(idade)
    fmt.Println("Idade como string: " + idadeTexto)
}
```

**Saída:**

```
Nome:    Gopher
Idade:   5 anos
Versão:  1.22
Ativo:   true
Máx. Conexões: 100
Idade como string: 5
```
