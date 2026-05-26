# Estruturas de Controle

As estruturas de controle determinam o **fluxo de execução** de um programa. Go possui um conjunto enxuto e poderoso: `if`, `for` e `switch`. Notavelmente, **Go não tem `while`** — o `for` cobre todos os casos de repetição.

---

## `if` / `else if` / `else`

A estrutura condicional em Go é parecida com C, mas **sem parênteses** ao redor da condição.

```go
package main

import "fmt"

func main() {
    temperatura := 28

    if temperatura > 35 {
        fmt.Println("Muito quente!")
    } else if temperatura > 25 {
        fmt.Println("Agradável.")
    } else if temperatura > 15 {
        fmt.Println("Um pouco frio.")
    } else {
        fmt.Println("Frio!")
    }
}
```

**Saída:**
```
Agradável.
```

> [!WARNING]
> As **chaves `{}` são obrigatórias** em Go, mesmo que o bloco tenha apenas uma linha. Ao contrário de C ou Java, não é possível omiti-las.

---

### `if` com Instrução de Inicialização

Go permite declarar uma variável **dentro do próprio `if`**. Ela fica disponível apenas no escopo do bloco condicional.

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // numero só existe dentro do bloco if/else
    if numero, err := strconv.Atoi("42"); err == nil {
        fmt.Printf("Número convertido: %d\n", numero)
    } else {
        fmt.Printf("Erro na conversão: %v\n", err)
    }
}
```

> [!TIP]
> Esse padrão é **muito comum em Go** para tratar erros retornados por funções sem poluir o escopo externo com variáveis desnecessárias.

---

## `for` — O Único Loop de Go

Go tem **apenas um tipo de loop**: o `for`. Mas ele pode ser usado de três formas diferentes.

### 1. `for` Clássico (estilo C)

```go
for inicialização; condição; pós-iteração {
    // corpo
}
```

```go
for i := 0; i < 5; i++ {
    fmt.Printf("Iteração %d\n", i)
}
```

**Saída:**
```
Iteração 0
Iteração 1
Iteração 2
Iteração 3
Iteração 4
```

---

### 2. `for` como `while`

Omitindo a inicialização e o pós-iteração:

```go
contador := 1

for contador <= 5 {
    fmt.Println(contador)
    contador++
}
```

---

### 3. `for` Infinito

```go
for {
    fmt.Println("Rodando para sempre...")
    // Use break para sair
    break
}
```

---

### 4. `for range` — Iterando sobre coleções

O `for range` é a forma idiomática de iterar sobre strings, arrays, slices, maps e channels.

```go
// Iterando sobre um slice
frutas := []string{"maçã", "banana", "laranja"}

for indice, fruta := range frutas {
    fmt.Printf("[%d] %s\n", indice, fruta)
}
```

**Saída:**
```
[0] maçã
[1] banana
[2] laranja
```

```go
// Ignorando o índice com _
for _, fruta := range frutas {
    fmt.Println(fruta)
}

// Iterando sobre um map
capitais := map[string]string{
    "Brasil":    "Brasília",
    "Argentina": "Buenos Aires",
    "Chile":     "Santiago",
}

for pais, capital := range capitais {
    fmt.Printf("%s → %s\n", pais, capital)
}

// Iterando sobre uma string (caractere por caractere)
for i, char := range "Go!" {
    fmt.Printf("Posição %d: %c\n", i, char)
}
```

---

### Controle de Loop: `break` e `continue`

```go
for i := 0; i < 10; i++ {
    if i == 3 {
        continue  // Pula para a próxima iteração
    }
    if i == 7 {
        break     // Sai do loop completamente
    }
    fmt.Println(i)
}
// Saída: 0 1 2 4 5 6
```

### Labels em Loops Aninhados

```go
externo:
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        if j == 1 {
            break externo  // Sai do loop externo
        }
        fmt.Printf("i=%d j=%d\n", i, j)
    }
}
// Saída: i=0 j=0
```

> [!WARNING]
> Use labels com moderação. O uso excessivo pode tornar o código difícil de ler e manter.

---

## `switch`

O `switch` em Go é mais poderoso que em outras linguagens:

- **Não precisa de `break`** entre os cases (o comportamento padrão já é parar após o match)
- Pode comparar **qualquer tipo de dado**, não apenas inteiros
- Pode ter **múltiplos valores** em um mesmo case
- Pode ser usado **sem expressão** (como um `if-else` encadeado)

### `switch` Básico

```go
dia := "quarta"

switch dia {
case "segunda", "terça", "quarta", "quinta", "sexta":
    fmt.Println("Dia útil")
case "sábado", "domingo":
    fmt.Println("Fim de semana!")
default:
    fmt.Println("Dia inválido")
}
```

---

### `switch` com Expressão Inicializadora

```go
switch hora := 14; {
case hora < 12:
    fmt.Println("Bom dia!")
case hora < 18:
    fmt.Println("Boa tarde!")
default:
    fmt.Println("Boa noite!")
}
```

---

### `switch` sem Expressão (substituto do `if-else`)

```go
nota := 85

switch {
case nota >= 90:
    fmt.Println("A — Excelente")
case nota >= 80:
    fmt.Println("B — Bom")
case nota >= 70:
    fmt.Println("C — Regular")
default:
    fmt.Println("D — Insuficiente")
}
```

---

### `fallthrough` — Forçando a queda entre cases

Use `fallthrough` para executar o próximo case mesmo sem corresponder à condição:

```go
x := 1

switch x {
case 1:
    fmt.Println("Case 1")
    fallthrough
case 2:
    fmt.Println("Case 2 (executado pelo fallthrough)")
case 3:
    fmt.Println("Case 3 (não executado)")
}
```

**Saída:**
```
Case 1
Case 2 (executado pelo fallthrough)
```

> [!WARNING]
> `fallthrough` em Go é **incondicional** — ele sempre executa o próximo case, independentemente da condição. Use com cautela.

---

### `switch` em Tipos (Type Switch)

Muito usado para verificar o tipo dinâmico de uma interface:

```go
func verificarTipo(v interface{}) {
    switch tipo := v.(type) {
    case int:
        fmt.Printf("Inteiro: %d\n", tipo)
    case string:
        fmt.Printf("String: %s\n", tipo)
    case bool:
        fmt.Printf("Boolean: %t\n", tipo)
    default:
        fmt.Printf("Tipo desconhecido: %T\n", tipo)
    }
}

func main() {
    verificarTipo(42)
    verificarTipo("hello")
    verificarTipo(true)
    verificarTipo(3.14)
}
```

**Saída:**
```
Inteiro: 42
String: hello
Boolean: true
Tipo desconhecido: float64
```

---

## Comparação de Estruturas de Controle

| Estrutura | Uso principal |
|-----------|--------------|
| `if / else` | Decisões simples com condições booleanas |
| `for` clássico | Repetição com contador |
| `for` como `while` | Repetição enquanto condição é verdadeira |
| `for range` | Iteração sobre coleções |
| `switch` | Múltiplas comparações sobre um valor |
| Type switch | Verificação de tipo dinâmico |

---

## Exemplo Completo — FizzBuzz

Um clássico exercício que combina `for` e condicionais:

```go
package main

import "fmt"

func main() {
    for i := 1; i <= 30; i++ {
        switch {
        case i%15 == 0:
            fmt.Println("FizzBuzz")
        case i%3 == 0:
            fmt.Println("Fizz")
        case i%5 == 0:
            fmt.Println("Buzz")
        default:
            fmt.Println(i)
        }
    }
}
```
