# Arrays, Slices e Maps

Go oferece três estruturas de dados fundamentais para coleções: **arrays** (tamanho fixo), **slices** (arrays dinâmicos) e **maps** (tabelas hash). Na prática, slices e maps são usados na grande maioria dos casos por serem mais flexíveis.

---

## Arrays

Um array em Go tem **tamanho fixo** definido em tempo de compilação. O tamanho faz parte do tipo — `[3]int` e `[5]int` são tipos diferentes.

### Declaração e Inicialização

```go
// Declaração com zero values (todos os elementos são 0)
var numeros [5]int
fmt.Println(numeros) // [0 0 0 0 0]

// Declaração com valores iniciais
primos := [5]int{2, 3, 5, 7, 11}
fmt.Println(primos) // [2 3 5 7 11]

// Go conta automaticamente o tamanho com ...
vogais := [...]string{"a", "e", "i", "o", "u"}
fmt.Println(len(vogais)) // 5

// Inicialização por índice
semana := [7]string{0: "Dom", 6: "Sáb"}
// Demais posições serão "" (zero value de string)
```

### Acessando e Modificando Elementos

```go
frutas := [3]string{"maçã", "banana", "laranja"}

fmt.Println(frutas[0]) // maçã
frutas[1] = "morango"
fmt.Println(frutas) // [maçã morango laranja]
```

### Iterando com `for range`

```go
notas := [5]float64{7.5, 8.0, 9.5, 6.0, 8.5}
soma := 0.0

for _, nota := range notas {
    soma += nota
}

media := soma / float64(len(notas))
fmt.Printf("Média: %.2f\n", media) // Média: 7.90
```

> [!NOTE]
> Arrays em Go são **copiados por valor** ao serem atribuídos ou passados para funções. Para trabalhar com referências, use slices ou ponteiros.

---

## Slices

Slices são o tipo de coleção mais usado em Go. Eles são **dinâmicos** (crescem automaticamente), mais flexíveis que arrays e implementados como uma **visão** sobre um array subjacente.

### Anatomia de um Slice

Um slice é uma estrutura com três campos internos:

```
┌─────────────────────────────────────────┐
│  Slice                                  │
│  ┌──────────┐ ┌────────┐ ┌──────────┐  │
│  │ Ponteiro │ │ Length │ │ Capacity │  │
│  └──────────┘ └────────┘ └──────────┘  │
└─────────────────────────────────────────┘
```

- **Pointer**: aponta para o array subjacente
- **Length (len)**: número de elementos no slice
- **Capacity (cap)**: número de elementos disponíveis no array subjacente

### Declaração e Inicialização

```go
// Slice literal
frutas := []string{"maçã", "banana", "laranja"}
fmt.Println(frutas)       // [maçã banana laranja]
fmt.Println(len(frutas))  // 3
fmt.Println(cap(frutas))  // 3

// Slice vazio (nil)
var s []int
fmt.Println(s == nil)     // true

// Usando make(tipo, length, capacity)
numeros := make([]int, 5, 10)
fmt.Println(len(numeros)) // 5
fmt.Println(cap(numeros)) // 10
```

### Operações Fundamentais

```go
s := []int{1, 2, 3, 4, 5}

// Fatiamento (slicing) — [inicio:fim]
fmt.Println(s[1:3])  // [2 3]   (índices 1 e 2)
fmt.Println(s[:3])   // [1 2 3] (do início até 2)
fmt.Println(s[2:])   // [3 4 5] (do índice 2 até o fim)
fmt.Println(s[:])    // [1 2 3 4 5] (slice completo)
```

### `append` — Adicionando Elementos

```go
s := []int{1, 2, 3}

s = append(s, 4)          // Adiciona um elemento
s = append(s, 5, 6, 7)    // Adiciona múltiplos

// Expandindo outro slice com ...
outros := []int{8, 9, 10}
s = append(s, outros...)

fmt.Println(s) // [1 2 3 4 5 6 7 8 9 10]
```

> [!WARNING]
> Quando a capacidade de um slice é excedida, Go cria um **novo array subjacente** com o dobro da capacidade e copia os dados. Por isso, sempre reatribua o resultado do `append`: `s = append(s, ...)`.

### `copy` — Copiando Slices

```go
origem := []int{1, 2, 3, 4, 5}
destino := make([]int, 3)

n := copy(destino, origem)
fmt.Println(destino) // [1 2 3]
fmt.Println(n)       // 3 (elementos copiados)
```

### Slices 2D (Matriz)

```go
// Criando uma matriz 3x3
matriz := [][]int{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
}

for _, linha := range matriz {
    for _, valor := range linha {
        fmt.Printf("%d ", valor)
    }
    fmt.Println()
}
```

**Saída:**
```
1 2 3 
4 5 6 
7 8 9 
```

### Deletando Elementos de um Slice

Go não tem função `remove` nativa. O idioma correto é:

```go
s := []int{1, 2, 3, 4, 5}
i := 2  // Índice a remover

// Remove o elemento no índice i
s = append(s[:i], s[i+1:]...)
fmt.Println(s) // [1 2 4 5]
```

---

## Maps

Maps são coleções de **pares chave-valor** (tabelas hash). As chaves devem ser de um tipo **comparável** (int, string, bool, etc.).

### Declaração e Inicialização

```go
// Map literal
capitais := map[string]string{
    "Brasil":     "Brasília",
    "Argentina":  "Buenos Aires",
    "Chile":      "Santiago",
    "Peru":       "Lima",
}

// Com make
idades := make(map[string]int)

// Map vazio declarado (nil — cuidado!)
var m map[string]int // nil — não use sem inicializar!
```

> [!WARNING]
> Um map `nil` **entra em pânico** se você tentar inserir um elemento. Sempre inicialize com `make` ou um map literal antes de escrever nele.

### Inserindo e Acessando Valores

```go
idades := make(map[string]int)

// Inserindo
idades["Alice"] = 30
idades["Bob"] = 25
idades["Carol"] = 35

// Lendo
fmt.Println(idades["Alice"]) // 30

// Chave inexistente retorna zero value
fmt.Println(idades["Zé"])    // 0 (zero value de int)
```

### Verificando se uma Chave Existe

```go
capitais := map[string]string{
    "Brasil": "Brasília",
}

// Idioma Go: dois valores de retorno
valor, existe := capitais["Brasil"]
if existe {
    fmt.Println("Capital:", valor) // Capital: Brasília
}

_, existe = capitais["Portugal"]
if !existe {
    fmt.Println("País não encontrado")
}
```

### Deletando um Elemento

```go
delete(capitais, "Brasil")
fmt.Println(capitais) // map[]
```

### Iterando sobre um Map

```go
estoque := map[string]int{
    "maçã":    50,
    "banana":  30,
    "laranja": 20,
}

for produto, quantidade := range estoque {
    fmt.Printf("%-10s → %d unidades\n", produto, quantidade)
}
```

> [!NOTE]
> A **ordem de iteração de um map é aleatória** em Go. Não confie em uma ordem específica. Se precisar de ordem, use um slice de chaves e ordene com `sort`.

### Map com Slice como Valor

```go
// Agrupando alunos por turma
turmas := map[string][]string{
    "A": {"Alice", "Bob", "Carol"},
    "B": {"David", "Eva"},
}

turmas["A"] = append(turmas["A"], "Frank")
fmt.Println(turmas["A"]) // [Alice Bob Carol Frank]
```

---

## Comparação Rápida

| Característica | Array | Slice | Map |
|---------------|-------|-------|-----|
| Tamanho | Fixo | Dinâmico | Dinâmico |
| Tipo de chave | Índice (int) | Índice (int) | Qualquer comparável |
| Zero value | `[N]T{}` | `nil` | `nil` |
| Copiado por valor? | ✅ Sim | ❌ Não (referência) | ❌ Não (referência) |
| Ordenado? | ✅ Sim | ✅ Sim | ❌ Não |

---

## Exemplo Completo — Contagem de Palavras

```go
package main

import (
    "fmt"
    "strings"
)

func contarPalavras(texto string) map[string]int {
    contagem := make(map[string]int)
    palavras := strings.Fields(texto) // divide por espaços

    for _, palavra := range palavras {
        palavra = strings.ToLower(palavra)
        contagem[palavra]++
    }

    return contagem
}

func main() {
    texto := "Go é rápido Go é simples Go é poderoso"
    contagem := contarPalavras(texto)

    for palavra, freq := range contagem {
        fmt.Printf("%-10s → %d\n", palavra, freq)
    }
}
```

**Saída (ordem pode variar):**
```
go         → 3
é          → 3
rápido     → 1
simples    → 1
poderoso   → 1
```
